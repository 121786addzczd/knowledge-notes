# NATインスタンス変更後にECSタスクからS3へ接続できない場合の対応

## 背景

NATインスタンスとして使用していた `t2.micro` が、AWS側のキャパシティ不足により起動できなかったため、インスタンスタイプを `t3.micro` に変更して起動した。

変更後、GitHub Actionsから `ecspresso run` を実行したところ、ECSタスクがAmazon S3上の環境変数ファイルを取得できず、起動に失敗した。

本事象は、`t3.micro` というインスタンスタイプ自体が原因ではない。NATインスタンスの実際のネットワークインターフェース名と、iptablesのNATルールに設定されたインターフェース名が一致していなかったことが原因だった。

## GitHub Actionsで発生したエラー

GitHub ActionsからDBマイグレーション用のECSタスクを起動した際、次のエラーが出力された。

```text
ResourceInitializationError: failed to download env files:
The task can't download the environment variable files from Amazon S3.
There is a connection issue between the task and Amazon S3.
operation error S3: HeadBucket, request canceled, context deadline exceeded
```

`HTTP StatusCode: 0` や `context deadline exceeded` が出ていたため、IAM権限ではなく、ECSタスクからS3までのネットワーク疎通を中心に調査した。

IAM権限が不足している場合は、一般的に `AccessDenied` やHTTPステータス `403` などが出力される。

## 構成上の前提

- ECSタスクはプライベートサブネットで実行される
- プライベートサブネットのデフォルトルートはNATインスタンスを向いている
- S3 Gateway VPC Endpointは使用していない
- そのため、ECSタスクがS3へ接続するにはNATインスタンスが正常にパケットを転送する必要がある

## NATインスタンスへの接続方法

調査および修正コマンドは、AWS Systems ManagerのSession Managerを使用してNATインスタンス上で実行した。

Session Managerは、EC2へSSH接続する場合と異なり、インターネット向けにSSHの22番ポートを開放したり、SSH秘密鍵を配布・管理したりする必要がない。そのため、必要なIAM権限や監査設定を行ったうえで、原則としてSession Managerの利用を推奨する。

SSHでも同じ確認は可能だが、インバウンドルールの開放や秘密鍵の管理が必要になるため、特別な理由がない限り推奨しない。

## 調査手順

### 1. IPフォワーディングを確認する

```bash
sysctl net.ipv4.ip_forward
```

今回の結果は次のとおりで、IPフォワーディングは有効だった。

```text
net.ipv4.ip_forward = 1
```

値が `0` の場合、NATインスタンスは受信したパケットを転送できない。一時的に有効化する場合は次を実行する。

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

### 2. iptablesのNATルールを確認する

```bash
sudo iptables -t nat -S POSTROUTING
```

今回の結果は次のとおりだった。

```text
-P POSTROUTING ACCEPT
-A POSTROUTING -o enX0 -j MASQUERADE
```

`MASQUERADE` は、プライベートサブネットから来た通信の送信元IPアドレスを、NATインスタンスのIPアドレスへ変換するための設定である。

### 3. 実際に外向き通信で使用されるNIC名を確認する

```bash
ip route get 1.1.1.1
ip -br address
```

今回の結果は次のとおりだった。

```text
1.1.1.1 via 10.0.0.1 dev ens5 src 10.0.33.108 uid 1001

lo    UNKNOWN  127.0.0.1/8 ::1/128
ens5  UP       10.0.33.108/18 fe80::46e:1cff:fee2:df6d/64
```

実際のNIC名は `ens5` だったが、iptablesのNATルールでは `enX0` が指定されていた。

```text
実際のNIC名:             ens5
NATルールが指定したNIC名: enX0
```

iptablesでは、存在しないインターフェース名を指定したルールも登録できる。そのため、設定時にエラーは発生しないが、そのインターフェースをパケットが通らず、NAT変換が実行されない状態になっていた。

## 修正手順

誤った `enX0` のルールを削除し、実際のNICである `ens5` を指定したルールを追加した。

```bash
sudo iptables -t nat -D POSTROUTING -o enX0 -j MASQUERADE
sudo iptables -t nat -A POSTROUTING -o ens5 -j MASQUERADE
```

修正後の設定を確認する。

```bash
sudo iptables -t nat -S POSTROUTING
```

次のように `ens5` が表示されれば、NIC名の修正は完了である。

```text
-P POSTROUTING ACCEPT
-A POSTROUTING -o ens5 -j MASQUERADE
```

> [!IMPORTANT]
> NIC名は環境によって異なる。上記の `ens5` をそのまま使用するのではなく、必ず `ip route get 1.1.1.1` などで対象インスタンスのNIC名を確認すること。

## 動作確認

修正後、GitHub Actionsを再実行し、`ecspresso run` からDBマイグレーション用ECSタスクを起動した。

ECSタスクはS3上の環境変数ファイルを取得でき、正常に起動・終了した。GitHub Actionsの後続処理も正常に実行された。

必要に応じて、ECSタスク実行中にiptablesのカウンターを確認する。

```bash
sudo iptables -t nat -L POSTROUTING -n -v
```

`ens5` の `MASQUERADE` 行にある `pkts` と `bytes` が増加すれば、通信がこのNATルールを通過していることを確認できる。

## 復旧しない場合の追加確認

NIC名を修正しても復旧しない場合は、次の項目を確認する。

```bash
# NATインスタンス自身からS3へ接続できるか確認する
curl -I --connect-timeout 10 https://s3.ap-northeast-1.amazonaws.com

# FORWARDチェーンでパケットが破棄されていないか確認する
sudo iptables -L FORWARD -n -v

# nftables側のルールを確認する
sudo nft list ruleset
```

あわせて、AWS側でも次を確認する。

- NATインスタンスのSource/Destination Checkが無効になっていること
- プライベートサブネットのデフォルトルートがNATインスタンスを向いていること
- NATインスタンスがパブリックサブネットにあり、Internet Gatewayへのルートを持つこと
- セキュリティグループとネットワークACLで必要な通信が許可されていること

## 再発防止

今回のiptablesコマンドによる修正は、設定の保存方法によってはEC2の再起動や再作成で失われる可能性がある。次のいずれかで設定を永続化する。

- systemdや起動スクリプトでIPフォワーディングとiptablesルールを設定する
- iptablesの永続化機能を使用し、再起動後にルールを復元する
- User Dataなどで起動時に実際のデフォルトルートのNIC名を取得し、NATルールを設定する

固定したNIC名を設定ファイルに直接書く場合は、インスタンスやAMIの変更によってNIC名が変わる可能性を考慮する。

また、今回のようにS3への接続が目的であれば、プライベートサブネットのルートテーブルへS3 Gateway VPC Endpointを関連付ける方法も有効である。S3向け通信がNATインスタンスに依存しなくなるため、NATインスタンス停止時の影響を減らせる。

## まとめ

1. GitHub Actionsの `ecspresso run` で、ECSタスクがS3の環境変数ファイルを取得できないことを検知した
2. Session ManagerでNATインスタンスへ接続した
3. IPフォワーディングが有効であることを確認した
4. iptablesのNATルールが `enX0` を指定していることを確認した
5. 実際の外向きNICが `ens5` であることを確認した
6. `MASQUERADE` ルールのNIC名を `enX0` から `ens5` へ修正した
7. GitHub Actionsから `ecspresso run` を再実行し、ECSタスクが正常に完了することを確認した

`t2.micro` から `t3.micro` への変更自体がS3接続エラーを発生させたわけではなく、変更後のインスタンスに適用されたNATルールと実際のNIC名の不一致が直接の原因だった。
