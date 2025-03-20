# 【Java】オブジェクトの中身が見れない？ObjectMapperを使ったデバッグ方法
JavaでAPIのDTOやモデルクラスのデバッグを行っていると、こんな表示に悩まされたことはありませんか？
```shell
model.dto.userInfoForRes@748735cd
```
「え？何この文字列…オブジェクトの中身が見えないんだけど・・・」

実はこれはJavaの標準的なオブジェクト表示で、オブジェクトのクラス名とハッシュコードを出しているだけなんです。

## なぜ中身が見えないのか？
JavaでSystem.out.println()を使ってオブジェクトを表示すると、内部的にはtoString()メソッドが呼ばれています。

```java
System.out.println(userInfo);
```
ですが、`toString()`をオーバーライドしていないクラスだと、JavaのObjectクラスのデフォルト実装が使われてしまいます。

その結果、`クラス名@ハッシュコード`の形式でしか表示されず、中身がまったく見えません。

## エクリプスのブレークポイントでのデバッグについて
Java開発では通常、エクリプスのブレークポイントを設定して、デバッグビューでオブジェクトの中身を確認する方法も一般的です。

ただし、今回はあくまで `printデバッグ` を中心に解説します。
IDEのデバッガに頼らず、コンソール上で素早く確認したい場合に便利な方法です。

## 解決方法(ObjectMapperを使ってJSON形式で出力する)
そんな時に便利なのが JacksonのObjectMapper！

### 手順
```java
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;

ObjectMapper objectMapper = new ObjectMapper();
try {
    String json = objectMapper.writeValueAsString(userInfo);
    System.out.println("JSON: " + json);
} catch (JsonProcessingException e) {
    e.printStackTrace();
}
```

### 出力例
```shell
JSON: {"name":"Morpheus","job":"Leader","id": "199","createdAt":"2020-02-20T11:00:28.107Z","contactdetails":{"phone”:”8439743294793","email":"test@abc.com"}}
```
これでオブジェクトの全プロパティの値が一目瞭然！

### 整形して見やすくする
さらに整形表示にしたい場合は writerWithDefaultPrettyPrinter() を使います。

```java
String prettyJson = objectMapper.writerWithDefaultPrettyPrinter().writeValueAsString(userInfo);
System.out.println(prettyJson);
```

### 出力例
```json
{
  "name": "Morpheus",
  "job": "Leader",
  "id": "199",
  "createdAt": "2020-02-20T11:00:28.107Z",
  "contactdetails": {
    "phone”:”8439743294793",
    "email":"test@abc.com"
  }
}
```


## まとめ
Javaでオブジェクトの中身が見えないときは、ObjectMapperを使ってJSON形式で出力すれば一発解決できます。
特に、API開発などでレスポンス確認やデバッグを頻繁に行う場合は、とても便利な方法です。

さらに発展編として
- toString()をオーバーライドして表示
- Lombokの@ToStringアノテーション活用
- ログフレームワーク（SLF4J, Logbackなど）と組み合わせたデバッグログ出力

などがあるようです？
