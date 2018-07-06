# 1. イントロダクション

Akka HTTPは *akka-actor* と *akka-stream* 上に実装されたサーバー/クライアントサイドのHTTPスタックです。Akka HTTPはWebフレームワークではなく、HTTPベースのサービスを提供/利用するためのより一般的なツールキットです。ブラウザのインタラクションももちろんAkka HTTPの提供するスコープに含まれますが、そこが一番の目的というわけではありません。

Akka HTTPはむしろオープンな設計に従っており、何度も「同じことをする」ためにいくつか異なるAPIのレベルを提供しています。
あなたのアプリケーションに最適な抽象化レベルを選ぶことができます。
これはもしあなたが高レベルのAPIを使って何かを達成するのにつまづいたとしても、より柔軟だが多くのアプリケーションコードを記述する必要があるかもしれない、低レベルのAPIを使って同じことを成し遂げるチャンスがあるということです。

## Akka HTTPの哲学

Akka HTTPはアプリケーションのコアよりもインテグレーション層を作るツールを提供する、という明確な目的をもってドライブされています。

フレームワークという言葉を専門用語として考えましょう。フレームワークは、あなたに、アプリケーションを構築するための「フレーム」を与えるものです。フレームワークには多くの規定事項が定められていて、素早く開始して結果を出せるように支援機構を含む基盤を提供します。ある意味で、フレームワークはアプリケーションに命を吹き込むための「肉付け」の下にある、骨組みのようなものです。このようなフレームワークは、アプリケーション開発の前に選んだ「やり方」を守りたいときには最良の選択肢となるでしょう。

例えば、もしあなたがブラウザで動くWebアプリケーションを作っているなら、Webフレームワークを選んでその上で開発するのは順当なやり方です。そのアプリケーションの「核」は、ブラウザとWebサーバー上のコードとのインタラクションだからです。フレームワークの製作者はこのようなアプリケーションを設計するために、一つの「信頼できる」方法を選択しており、多かれ少なかれ、柔軟な「アプリケーションのテンプレート」を「穴埋め」できるようにしています。このようなアーキテクチャのベストプラクティスに頼ることができれば、迅速に成果を出すための大きな資産となりえます。

しかしながら、もしアプリケーションのコアがブラウザとのインタラクションではなく、複雑であろうビジネスサービスに特化しており、単にREST/HTTPのインターフェースを経由して世界に繋げようとしているなら、Webフレームワークはあなたが必要としているものではないのかもしれません。このようなケースでは、アプリケーションのアーキテクチャはインターフェース層ではなく、コアにとって何が理にかなっているのかによって規定されるべきです。また、ビューテンプレート、アセット管理、JavaScriptとCSSの生成/操作/minify、ローカライズ支援、AJAXサポートなどの、ブラウザに特化したコンポーネントによって恩恵を受けることもないでしょう。

Akka HTTPは「非フレームワーク」として特別に設計されました。フレームワークが嫌いだからではなく、フレームワークが正しい選択肢ではないユースケースのために。Akka HTTPはHTTPに基づくインテグレーション層を構築し、脇役に徹するようなものとして作られています。したがって通常、あなたはAkka HTTP「の上に」アプリケーションを作るのではなく、何でも理にかなったものの上にアプリケーションを構築し、単にHTTPのインテグレーションが必要だからAkka HTTPを使うのです。

一方で、もしあなたがフレームワークの指針に沿ってアプリケーションを作るのがお好みであれば、Play FrameworkかLagomを使ってみてください。どちらもAkkaを内部的に使用しています。

## Akka HTTPを使用する

Akka HTTPは独自のリリースサイクルを有する、Akka自体とは独立したモジュールとして提供されています。Akka HTTPは Akka 2.5と、それ以降Akka HTTP 10.1.xのサポート期間中にリリースされた、バージョン2.xで @ref[互換性があります](compatibility-guidelines.md)。しかし、`akka-actor`や`akka-stream`に依存しては *いけません* 。ですから、ユーザーは実行するAkkaのバージョンを選択肢、選択したバージョンの`akka-stream`を手動で依存関係に追加する必要があります。

sbt
:   @@@vars
    ```
    "com.typesafe.akka" %% "akka-http"   % "$project.version$" $crossString$
    "com.typesafe.akka" %% "akka-stream" % "$akka.version$" // or whatever the latest version is
    ```
    @@@

Gradle
:   @@@vars
    ```
    compile group: 'com.typesafe.akka', name: 'akka-http_$scala.binary_version$',   version: '$project.version$'
    compile group: 'com.typesafe.akka', name: 'akka-stream_$scala.binary_version$', version: '$akka.version$'
    ```
    @@@

Maven
:   @@@vars
    ```
    <dependency>
      <groupId>com.typesafe.akka</groupId>
      <artifactId>akka-http_$scala.binary_version$</artifactId>
      <version>$project.version$</version>
    </dependency>
    <dependency>
      <groupId>com.typesafe.akka</groupId>
      <artifactId>akka-stream_$scala.binary_version$</artifactId>
      <version>$akka.version$</version> <!-- Or whatever the latest version is -->
    </dependency>
    ```
    @@@

Akka HTTPが`akka-http`と`akka-http-core`の、二つの主要モジュールであることに注意してください。`akka-http`は`akka-http-core`に依存しているので、後者を明示的に指定する必要はありません。低レベルのAPIを単独で使う場合は、Scalaのバージョンが最近リリースされた`2.11`か`2.12`であることを確認する必要があるかもしれません。

代替手段として、[Giter8](http://www.foundweekends.org/giter8/)テンプレートを使ってAkka HTTPが設定済みの新規sbtプロジェクトをブートストラップできます。

@@@ div { .group-scala }
```sh
sbt -Dsbt.version=0.13.15 new https://github.com/akka/akka-http-scala-seed.g8
```
@@@
@@@ div { .group-java }
```sh
sbt -Dsbt.version=0.13.15 new https://github.com/akka/akka-http-java-seed.g8
```
From there on the prepared project can be built using Gradle or Maven.
@@@

さらなる情報は @scala[[template
project](https://github.com/akka/akka-http-scala-seed.g8)]@java[[template
project](https://github.com/akka/akka-http-java-seed.g8)] にあります。バージョン0.13.13以上のsbtが必要なことにご注意ください。

## HTTP サーバー用のRouting DSL

高レベルのAkka HTTPルーティングAPIはHTTPの「ルート」と、それをどう取り扱うべきかを記述するためのDSLを提供します。
各ルートは１つ以上の、特定の１種のリクエストを扱うように絞り込む`Directive`で構成されます。

例えば、あるルートはリクエストの`path`をマッチングすることから始まります。pathが"/hello"の場合のみマッチし、次にHTTPの`get`リクエストだけを扱うように絞り込みます。マッチしたら文字列リテラルでリクエストを`完了`し、その文字列をレスポンスボディとするHTTP OKとして送り返されます。

リクエストとレスポンスのボディをネットワーク上でのフォーマットとアプリケーションで使われるオブジェクトとの相互変換は"magnet"パターンで暗黙的に導入されたマーシャラーの内部で実行され、ルート宣言とは分離されています。これはスコープ内で暗黙的なマーシャラーが利用可能な限り、どんな種類のオブジェクトでもリクエストを`完了`できるということです。

@@@ div { .group-scala }
デフォルトのマーシャラーはStringやByteStringといったシンプルな型に限って提供されていますが、例えばJSON型のマーシャラーなどをご自身で定義できます。追加モジュールで、spray-jsonライブラリを使ったJSONのシリアライズを提供しています。(詳細は @ref[JSON Support](common/json-support.md) をご覧ください。)
@@@
@@@ div { .group-java }
外部ライブラリのJacksonを使用することで、`akka-http`でのJSONサポートが可能です。
(詳細は @ref[JSON Support](common/json-support.md#json-jackson-support-java) をご覧ください。)
@@@

Route DSLを使って生成された @unidoc[Route] は、HTTPリクエストの提供を開始するためのポートにバインドされます。

Scala
:   @@snip [HttpServerExampleSpec.scala]($test$/scala/docs/http/scaladsl/HttpServerExampleSpec.scala) { #minimal-routing-example }

Java
:   @@snip [HttpServerMinimalExampleTest.java]($test$/java/docs/http/javadsl/HttpServerMinimalExampleTest.java) { #minimal-routing-example }

サーバーを起動したら、ブラウザで [http://localhost:8080/hello](http://localhost:8080/hello) にアクセスしてページを開いたり、`curl http://localhost:8080/hello`でターミナルから呼び出したりすることができます。

一般的なユースケースはリクエストに対し、JSONに変換するマーシャラーを持ったモデルオブジェクトを使って返信するケースです。このケースは２箇所の異なったルートで見ることができます。一つ目は非同期のデータベースにデータベースにクエリを投げ、@scala[`Future[Option[Item]]`]@java[`CompletionStage<Optional<Item>>`]型の結果をJSONレスポンスにマーシャルするルートです。二つ目はやってくるリクエストから`Order`をアンマーシャルし、それをデータベースに保存し、終わったらOKを返すルートです。

Scala
:   @@snip [SprayJsonExampleSpec.scala]($test$/scala/docs/http/scaladsl/SprayJsonExampleSpec.scala) { #second-spray-json-example }

Java
:   @@snip [JacksonExampleTest.java]($test$/java/docs/http/javadsl/JacksonExampleTest.java) { #second-jackson-example }

When you run this server, you can update the inventory via `curl -H "Content-Type: application/json" -X POST -d '{"name":"hhgtg","id":42}' http://localhost:8080/create-order` on your terminal - adding an item named `"hhgtg"` and having an `id=42`; and then view the inventory either in a browser, at a url like: [http://localhost:8080/item/42](http://localhost:8080/item/42) - or on the terminal,
via `curl http://localhost:8080/item/42`.

The logic for the marshalling and unmarshalling JSON in this example is provided by the @scala["spray-json"]@java["Jackson"] library
(details on how to use that here: @scala[@ref[JSON Support](common/json-support.md))]@java[@ref[JSON Support](common/json-support.md#json-jackson-support-java))].

One of the strengths of Akka HTTP is that streaming data is at its heart meaning that both request and response bodies
can be streamed through the server achieving constant memory usage even for very large requests or responses. Streaming
responses will be backpressured by the remote client so that the server will not push data faster than the client can
handle, streaming requests means that the server decides how fast the remote client can push the data of the request
body.

Example that streams random numbers as long as the client accepts them:

Scala
:   @@snip [HttpServerExampleSpec.scala]($test$/scala/docs/http/scaladsl/HttpServerExampleSpec.scala) { #stream-random-numbers }

Java
:   @@snip [HttpServerStreamRandomNumbersTest.java]($test$/java/docs/http/javadsl/HttpServerStreamRandomNumbersTest.java) { #stream-random-numbers }

Connecting to this service with a slow HTTP client would backpressure so that the next random number is produced on
demand with constant memory usage on the server. This can be seen using curl and limiting the rate
`curl --limit-rate 50b 127.0.0.1:8080/random`

Akka HTTP routes easily interacts with actors. In this example one route allows for placing bids in a fire-and-forget
style while the second route contains a request-response interaction with an actor. The resulting response is rendered
as json and returned when the response arrives from the actor.

Scala
:   @@snip [HttpServerExampleSpec.scala]($test$/scala/docs/http/scaladsl/HttpServerExampleSpec.scala) { #actor-interaction }

Java
:   @@snip [HttpServerActorInteractionExample.java]($test$/java/docs/http/javadsl/HttpServerActorInteractionExample.java) { #actor-interaction }

When you run this server, you can add an auction bid via `curl -X PUT http://localhost:8080/auction?bid=22&user=MartinO` on the terminal; and then you can view the auction status either in a browser, at the url [http://localhost:8080/auction](http://localhost:8080/auction), or, on the terminal, via `curl http://localhost:8080/auction`.

More details on how JSON marshalling and unmarshalling works can be found in the @ref[JSON Support section](common/json-support.md).

Read more about the details of the high level APIs in the section @ref[High-level Server-Side API](routing-dsl/index.md).

## Low-level HTTP server APIs

The low-level Akka HTTP server APIs allows for handling connections or individual requests by accepting
@unidoc[HttpRequest] s and answering them by producing @unidoc[HttpResponse] s. This is provided by the `akka-http-core` module.
APIs for handling such request-responses as function calls and as a @unidoc[Flow[HttpRequest, HttpResponse, \_]] are available.

Scala
:   @@snip [HttpServerExampleSpec.scala]($test$/scala/docs/http/scaladsl/HttpServerExampleSpec.scala) { #low-level-server-example }

Java
:   @@snip [HttpServerLowLevelExample.java]($test$/java/docs/http/javadsl/HttpServerLowLevelExample.java) { #low-level-server-example }

Read more details about the low level APIs in the section @ref[Core Server API](server-side/low-level-api.md).

## HTTP client API

The client APIs provide methods for calling a HTTP server using the same @unidoc[HttpRequest] and @unidoc[HttpResponse] abstractions
that Akka HTTP server uses but adds the concept of connection pools to allow multiple requests to the same server to be
handled more performantly by re-using TCP connections to the server.

Example simple request:

Scala
:   @@snip [HttpClientExampleSpec.scala]($test$/scala/docs/http/scaladsl/HttpClientExampleSpec.scala) { #single-request-example }

Java
:   @@snip [ClientSingleRequestExample.java]($test$/java/docs/http/javadsl/ClientSingleRequestExample.java) { #single-request-example }

Read more about the details of the client APIs in the section @ref[Consuming HTTP-based Services (Client-Side)](client-side/index.md).

## The modules that make up Akka HTTP

Akka HTTP is structured into several modules:

akka-http
: Higher-level functionality, like (un)marshalling, (de)compression as well as a powerful DSL
for defining HTTP-based APIs on the server-side, this is the recommended way to write HTTP servers
with Akka HTTP. Details can be found in the section @ref[High-level Server-Side API](routing-dsl/index.md)

akka-http-core
: A complete, mostly low-level, server- and client-side implementation of HTTP (incl. WebSockets)
Details can be found in sections @ref[Core Server API](server-side/low-level-api.md) and @ref[Consuming HTTP-based Services (Client-Side)](client-side/index.md)

akka-http-testkit
: A test harness and set of utilities for verifying server-side service implementations


@@@ div { .group-scala }
akka-http-spray-json
: Predefined glue-code for (de)serializing custom types from/to JSON with [spray-json](https://github.com/spray/spray-json)
Details can be found here: @ref[JSON Support](common/json-support.md)
@@@

@@@ div { .group-scala }
akka-http-xml
: Predefined glue-code for (de)serializing custom types from/to XML with [scala-xml](https://github.com/scala/scala-xml)
Details can be found here: @ref[XML Support](common/xml-support.md)
@@@
@@@ div { .group-java }
akka-http-jackson
: Predefined glue-code for (de)serializing custom types from/to JSON with [jackson](https://github.com/FasterXML/jackson)
@@@
