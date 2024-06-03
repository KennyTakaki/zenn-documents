---
title: "CloudFrontとCognitoを連携させる際に困ったこと(Lambda@Edgeのパッケージサイズ制限)"
emoji: "🍣"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [AWS,Serverless,Cloudfront,Cognito]
published: true
---
# 概要
いま調査中のソリューションの中でCloudfrontとCognitoの連携がなされていて、その部分が少しうまく動いていないようなので挙動をしっかり理解したかったので一度実装してみることにした。これまで経験したことがあるのはAmplifyを用いたユーザ認証の実装で、ツールに依存していた。この部分の知識を深堀することになる。  

基本的には[こちらの記事](https://dev.classmethod.jp/articles/cloudfront-s3-cognito-authentication/)をベースに実施すれば問題なかった。独自ドメインに関しては特に設定せずにすすめた。  

ただ、手順の中で一点だけはまってしまったので対応を記事に残しておく。

# 何がおこったか
Cloudfront に登録する Lambda@Edge のパッケージのサイズ上限に引っかかりました。公式が提供している以下のパッケージを利用したところ、Lambdaに登録するアセットが1MBを超えてしまいました。（参照記事の中ではどのように解決しているのだろうか…）
https://github.com/awslabs/cognito-at-edge

おおむね以下のようにコマンドを実施し、index.jsにCognitoのユーザプールの情報を記述して簡単なハンドラを作成します。

``` command
npm init
npm install cognito-at-edge
touch index.js
```

```index.js
const { Authenticator } = require('cognito-at-edge');

const authenticator = new Authenticator({
  // Replace these parameter values with those of your own environment
  region: 'ap-northeast-1', // user pool region
  userPoolId: 'ap-northeast-1_hogefuga', // user pool ID
  userPoolAppId: 'hogefugapokopoko', // user pool app client ID
  userPoolDomain: 'xxxyyyzzz.auth.ap-northeast-1.amazoncognito.com', // user pool domain
});

exports.handler = async (request) => authenticator.handle(request);
```

ここまでで以下のような構成になっているはずです。
```
ls
index.js  node_modules  package-lock.json  package.json
```

これらをLambdaにアップロードするパッケージにすると作成したpkgが1MBを超えてしまいLambda@Edgeとして展開できませんでした。
```
zip -r pkg.zip .

 ls -lh pkg.zip 
-rw-r--r--. 1 ec2-user ec2-user 1.5M Jun  3 13:35 pkg.zip
```
(Lambdaのcreateコマンド割愛)
![alt text](/images/articles/cloudfront-cognito/1mb.png)

# 試してみた対応策
## 対策１
パッケージ部分を Lambda Layers に登録して本体から参照させようと考えました。ところが、Lambda @ Edgeは Lambda Layersをサポートしていないようで失敗に終わりました。制約は事前に確認してから行動しないとだめですね。
![alt text](/images/articles/cloudfront-cognito/LambdaLayers.png)

## 対策2
インストールしたパッケージの中から実行時に不要と思われるコードを削除してみました。
パッケージ内に JavaScript は開発時にビルドしてミニファイされたコードと元のコードの間を取り持つ*.map というファイルが存在していました。
これらは（おそらく）デバッグ時に実行中のスクリプトから元のソースを参照するために必要なはずです。

```
find -name "*.map" | xargs -I{} ls -lh {}
-rw-r--r--. 1 ec2-user ec2-user 198K Jun  3 13:30 ./node_modules/axios/dist/browser/axios.cjs.map
-rw-r--r--. 1 ec2-user ec2-user 263K Jun  3 13:30 ./node_modules/axios/dist/node/axios.cjs.map
-rw-r--r--. 1 ec2-user ec2-user 199K Jun  3 13:30 ./node_modules/axios/dist/esm/axios.js.map
-rw-r--r--. 1 ec2-user ec2-user 155K Jun  3 13:30 ./node_modules/axios/dist/esm/axios.min.js.map
-rw-r--r--. 1 ec2-user ec2-user 228K Jun  3 13:30 ./node_modules/axios/dist/axios.js.map
-rw-r--r--. 1 ec2-user ec2-user 163K Jun  3 13:30 ./node_modules/axios/dist/axios.min.js.map
-rw-r--r--. 1 ec2-user ec2-user 6.3K Jun  3 13:30 ./node_modules/abort-controller/dist/abort-controller.js.map
-rw-r--r--. 1 ec2-user ec2-user 6.2K Jun  3 13:30 ./node_modules/abort-controller/dist/abort-controller.mjs.map
-rw-r--r--. 1 ec2-user ec2-user 37K Jun  3 13:30 ./node_modules/abort-controller/dist/abort-controller.umd.js.map
-rw-r--r--. 1 ec2-user ec2-user 37K Jun  3 13:30 ./node_modules/event-target-shim/dist/event-target-shim.js.map
-rw-r--r--. 1 ec2-user ec2-user 37K Jun  3 13:30 ./node_modules/event-target-shim/dist/event-target-shim.mjs.map
-rw-r--r--. 1 ec2-user ec2-user 37K Jun  3 13:30 ./node_modules/event-target-shim/dist/event-target-shim.umd.js.map
```

これらだけでは削減サイズが足りなかったのでテスト用のコードが格納されたディレクトリを削除しました。対象は以下です。
```
find -type d -name "test" 
./node_modules/pino/test
./node_modules/fast-redact/test
./node_modules/on-exit-leak-free/test
./node_modules/pino-abstract-transport/test
./node_modules/pino-std-serializers/test
./node_modules/process-warning/test
./node_modules/quick-format-unescaped/test
./node_modules/sonic-boom/test
./node_modules/thread-stream/test
```

これによって、Lambdaに登録するパッケージを1MB以下にすることができました。

```
ls -lh pkg.zip 
-rw-r--r--. 1 ec2-user ec2-user 972K Jun  3 13:48 pkg.zip
```

# 結果
対策２で作成したパッケージを Lambda@Edge に登録することで正常にCloudFrontを通したCognitoの認証を実装することができました。
![alt text](/images/articles/cloudfront-cognito/cognito.png)

# 今後
JavaScriptの扱いになれておらず、大きなパッケージサイズをどのようにハンドリングしてよいかわからなかったのが反省点です。
今回はわざわざ手動でテストコードを削除したり、開発時に必要と思われるコードを削除して対応しましたが、かなり邪道な気がしています。
もう少しよい方法がないか調べてみようと思います。