---
title: "CMS on AWS の調査"
emoji: "📝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [CMS,AWS]
published: false
---

# 目的
主に車両からデータを収集/利活用するためのソリューションがAWSから提案されている。
[Connected Mobility Solution on AWS](https://aws.amazon.com/jp/solutions/implementations/connected-mobility-solution-on-aws/)と命名されており、略称はCMS on AWSとなる。  
このCMS on AWSに興味がでたので、少し下調べをしようと思う。

## 必要となる技術スタック
CMS on AWS はソリューションが必要とする機能をモジュール化して提供しているが、すべてのモジュールの理解を網羅するには以下のページで参照されるAWSサービスを理解しておく方がよい。
https://docs.aws.amazon.com/solutions/latest/connected-mobility-solution-on-aws/aws-services-in-this-solution.html

おおよそはメジャーなサービス群で構成されているが[AWS Chalice](https://aws.github.io/chalice/)なんかは初耳だ。主にサーバレスでAPIを開発するためのFrameworkらしく、他のPythonフレームワークを触ったことがあるならなじみやすそうな記法でAPIが作成できるようである。



# 参照
#### GitHub リポジトリ
https://github.com/aws-solutions/connected-mobility-solution-on-aws

#### 実装リファレンス
https://docs.aws.amazon.com/solutions/latest/connected-mobility-solution-on-aws/solution-overview.html


## なんかエラーがでる。
(.venv) takaki@LAPTOP-8B3JRJ13:~/connected-mobility-solution-on-aws$ make upload
find: warning: you have specified the global option -maxdepth after the argument -type, but global options are not positional, i.e., -maxdepth affects tests specified before it as well as those specified after it.  Please specify global options before other arguments.
find: warning: you have specified the global option -mindepth after the argument -type, but global options are not positional, i.e., -mindepth affects tests specified before it as well as those specified after it.  Please specify global options before other arguments.
Creating required buckets...
Bucket (acdp-assets-056797692780-ap-northeast-1) ready.
Bucket (acdp-assets-056797692780-ap-northeast-1) ready.

Error parsing parameter '--body': Blob values must be a path to a file.
make: *** [Makefile:97: upload-backstage-assets-zip] Error 252