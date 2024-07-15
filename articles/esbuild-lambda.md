---
title: "esbuild でバンドルしてLambda@Edgeにデプロイしてみる"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [AWS,Serverless,Lambda,JavaScript]
published: true
---

# 概要
Node.jsをランタイムとしたLambdaの開発を実施したことがなく、JavaScriptの周辺ツールの扱いに慣れていない。前回はベタで記述したエントリポイントとパッケージのディレクトリをそのままLambdaFunctionに登録したが、今回はesbuildでバンドリングして実行してみた。結果敵に動作したのでこれでよいと思う。

# 詳細
[前回](https://zenn.dev/frommiddle1/articles/cloudfront-cognito)の記事でNode.jsのLambda@Edgeの関数コードのサイズ制限に引っかかったと記載したのだが、普通にバンドルすればよいだけだった。

適当に漁ったページにもTypeScriptで記述すればesbuildのバンドルが利用可能だと記述してある。（SAMのドキュメントが参照もとだけど、たぶん行けるだろう）
> To build and package Node.js AWS Lambda functions, you can use the AWS SAM CLI with the esbuild JavaScript bundler. The esbuild bundler supports Lambda functions that you write in TypeScript.

## 手順
まずはパッケージに対してesbuildを開発用の依存関係としてインストールする。
```
npm install --save-exact --save-dev esbuild
```

エントリポイントのコードの拡張しを.tsに変更する。
この時点でパッケージにするディレクトリは以下のような感じになっている。
```
ls
index.ts  node_modules  package-lock.json  package.json
```

これをesbuildでバンドルする。yarnやpnpmは使ってないのでnpxでローカルインストールしたesbuildを実行している。
```
 npx esbuild --bundle index.ts --minify --outfile=index.js --platform=node
```
出力されたLambdaをzip圧縮する。サイズも十分に小さい。
```
zip upload.zip index.js 

ls -lh upload.zip 
-rw-r--r--. 1 ec2-user ec2-user 80K Jun  8 03:18 upload.zip
```
このupload.zipを改めてLambda関数に登録してCloudFrontのビューワーリクエストに対するLambda@Edgeを差し替えた。
ちなみにコードは人間が読めないようなものになっている。エントリポイントとなるハンドラがよくわからず、これでLambdaが動作するのかが自身がなかったが、どうやらそこは問題ないらしい。
  
更新後も無事Cognitoのログインページが表示されたので成功だ。
![alt text](/images/articles/esbuild-lambda/cognito.png)

# 感想
JavaScriptというかNode.js回りは周辺ツールが多くて煩雑だバンドラーも昔はwebpackでバンドルして、BabelでJS構文のバージョン変換を行っていた印象だったが、いまはeslintが登場している。言語そのもののよりツール類に対す理解が追い付いていないのが課題だ。