# 利用者が増えても自動でスケールするシステムの開発手法


AWSを十分に活用すれば、自動でAWSの以下の恩恵に預かることができます。
- アクセス数が多くても自動でスケールアウトしてすることができる。
- アクセス数が少ない場合、自動でスケールインするようにできる。利用がない時は課金額をゼロに近くできる。
- アップデートが自動で安全に適用される。
- セキュリティが高いシステムが低コストで構築できる。
- ユーザーにストレスの無い、快適に動作するシステムが低コストで開発できる。
- システムダウンの少ない可用性の高いシステムを低コストで運用できる。

そのためには原則、以下の通りにします。

- EC2でWEBアプリケーションを稼働させるべきではない。WEBアプリケーションはLambdaで稼働させる。
- DBはDynamoDBで稼働させる。
- ファイルはS3に保存する。
- フロントエンドはNext.jsで開発しCloudFrontとS3で公開する。
- 認証にはCognitoを使用する。
- データの送受信はAppSyncで構築する。

## アーキテクチャの例

アーキテクチャの例は以下の通りです。

```uml
@startuml

	!define AWSPuml https://raw.githubusercontent.com/awslabs/aws-icons-for-plantuml/v15.0/dist
	
	!include AWSPuml/AWSCommon.puml
	!include AWSPuml/AWSExperimental.puml
	!include AWSPuml/Groups/all.puml
	!include AWSPuml/ApplicationIntegration/AppSync.puml
	!include AWSPuml/Compute/LambdaLambdaFunction.puml
	!include AWSPuml/Database/DynamoDB.puml
	!include AWSPuml/General/Documents.puml
	!include AWSPuml/General/Multimedia.puml
	!include AWSPuml/General/Tapestorage.puml
	!include AWSPuml/General/User.puml
	!include AWSPuml/General/AWSManagementConsole.puml
	!include AWSPuml/MediaServices/ElementalMediaConvert.puml
	!include AWSPuml/MachineLearning/Transcribe.puml
	!include AWSPuml/NetworkingContentDelivery/CloudFront.puml
	!include AWSPuml/SecurityIdentityCompliance/Cognito.puml
	!include AWSPuml/Storage/SimpleStorageService.puml
	!includeurl AWSPuml/NetworkingContentDelivery/CloudFront.puml
	
	' define custom group for Amazon S3 bucket
	AWSGroupColoring(S3BucketGroup, #FFFFFF, AWS_COLOR_GREEN, plain)
	!define S3BucketGroup(g_alias, g_label="Amazon S3 bucket") AWSGroupEntity(g_alias, g_label, AWS_COLOR_GREEN, SimpleStorageService, S3BucketGroup)
	
	' Groups are rectangles with a custom style using stereotype - need to hide
	hide stereotype
	skinparam linetype ortho
	skinparam rectangle {
		BackgroundColor AWS_BG_COLOR
		BorderColor transparent
	}
	
	rectangle "$UserIMG()\nユーザー" as user
	
	AWSCloudGroup(cloud){
		RegionGroup(region) {
	
			rectangle "$LambdaLambdaFunctionIMG()\nLambda\n処理" as lambda
				user -> lambda: <$Callout_4>\lPost
	
			rectangle "$CognitoIMG()\nCognito\n認証管理" as Cognito
				lambda <-> Cognito: ユーザー認証管理
				user <--> Cognito: <$Callout_3>\l認証
	
			S3BucketGroup(s3) {
				rectangle "$AWSManagementConsoleIMG()\nWEBサイト" as websites3
			}
		}
	
		rectangle "$CloudFrontIMG()\nCroudFront\nCDN" as cloudfront
			cloudfront <-> websites3: <$Callout_2>\l静的ファイル
			user --> cloudfront: <$Callout_1>\lWEBサイト\n訪問
	
		rectangle "$AppSyncIMG()\nAppSync" as AppSync
			AppSync <-> Cognito: ユーザー認証管理
	
		rectangle "$DynamoDBIMG()\nDynamoDB" as DynamoDB
			AppSync <--> DynamoDB: <$Callout_7>\lデータ受け渡し
			user --> AppSync: <$Callout_5>\lデータ要求
	}

@enduml
```

## なぜEC2でWEBアプリケーションを稼働させるべきではないか

以下の理由から、EC2でWEBアプリケーションを稼働させるべきではありません。
- EC2の本番環境にどんどん手が加えられ、いわゆる「秘伝のタレ」化してしまい、トラブルが発生した時に本番環境を新たに再構築できない事態が発生しうる
- EC2のSLAは99.5%と非常に低く、よく落ちる
- 新規開発システムをEC2でWEBアプリケーションを稼働させるメリットが無い。既存システムを利用するためであっても、EC2を利用するのではなくさくらインターネットの「さくらのVPS」やNTTの「WebARENA Indigo」などのサービスを利用したほうが運用コストが低くシステムの停止も少なくサービス品質も高い。
- EC2は自動ではスケールアップもスケールアウトもされず、急に大量のアクセスが来た場合に対応できない。また、スケールダウンもスケールインも自動ではされないため、たとえ1件もアクセスが来なくても同じ額が課金されてしまう。
- スケールアップもスケールアウトもしないシステムの場合、アクセス数の予測が難しい場合や日時によってアクセス数に増減がある場合などはアクセス数が最も多い瞬間に合わせてインスタンスの性能を決定しなければならないため、無駄が多い。

極力WEBアプリケーションはLambdaで稼働させるべきです。  
EC2と比較した、LambdaでWEBアプリケーションを稼働させるメリットは以下の通りです。
- EC2ではアクセスが無い時でも課金されるが、Lambdaであれば課金はアクセスに応じてのミリ秒単位であるため、運用コストが抑えられる(極端な話、アクセスが0件であれば課金されない)
- EC2ではセキュリティアップデートを自己責任でシステムを停止させて実施しなければならないが、Lambdaであればセキュリティアップデートが自動で安全に実施されシステム停止も発生しない（※Lambdaでもセキュリティアップデートを手動でする設定もある）
- EC2ではアクセスに応じてスケールアップ・スケールアウトさせるのが難しいが、Lambdaでは自動でスケールするため、急な大量のアクセスにも柔軟に対応できる
- EC2はSLAが99.5%(最大月間3.6時間も、サービスが落ちていることを許容する計算になる)しかないが、LambdaはSLAが99.95%(最大でも月間21.6分しかサービスが落ちていることを許容しない計算になる)であるため品質が高い

# AWSの各サービスの使い方

## AWS Lambda

AWSの最も先進的かつ便利なサービスのうちの1つです。

以下のメリットがあります。
- 課金がミリ秒単位であるため、運用コストが抑えられる(極端な話、アクセスが0件であれば課金されない)
- SLAが99.95%と、非常に高い稼働率が期待できる。
- 様々なプログラミング言語に対応しており、処理の目的に応じてプログラミング言語を使い分けることができる。
- コールドスタートが非常に高速で、EC2では対応できないような場面にも対処できる。
  - Javaの場合普通に使用するとコールドスタートは遅いが、その場合でもGraalVMを利用すればコールドスタートも高速になる。
- 自動でスケールアウトするため、

## Amazon Cognito

ユーザーの認証を管理するサービスです。
現代のWEBサービスに必要な
- ユーザー登録
- メール・電話番号認証
- クッキー管理
- SNS認証
などの機能を一通り備えています。

一般的なWEBサービスで必要とされる機能をほぼ網羅しており、ほとんどのユースケースで必要十分な機能を備えています。

## Amazon S3

ファイルを保存するストレージです。
非常に様々な用途が想定されていて、たとえば以下のようなこともできます。
- インターネットへ公開するファイルを保存する。
- ユーザーから保存を受け付けるがユーザーへの公開はしない、書き込み専用読み込み不可ストレージ。
- AppSyncやLambdaから、ファイル名を主キーとしたデータソースとして利用する

## Amazon CloudFront

全世界に分散して配置されているCDNです。
世界中のどこからアクセスしても速く、S3に用意されたファイルをWEBで配信するのに便利です。

***
ここから書きかけです
***

## Amazon DynamoDB

## Amazon Aurora Serverless v2

DynamoDBが使えない領域での第二の選択肢です。
最小構成で、一切利用が無い場合でも北米リージョンで$43.2、東京リージョンで$72のコストがかかる。(当然ながら、DynamoDBでは一切利用が無い場合は保存データ料金を除いて無料です。)

## Amazon RDS

## AWS AppSync

## Amazon SQS

## Amazon SES

## Amazon EC2

EC2は非常に高価ですが、「スポットインスタンス」というインスタンスがあり、これがどういうものかというと「空いている時間に利用できる、料金が非常に安いインスタンス」です。
夜間バッチなどのバッチ処理などに利用すると非常に安価にデータ処理がおこなえます。

## Amazon ECS・Amazon EKS

Dockerコンテナを立ち上げるためのサービスがAmazon ECSやAmazon EKSです。Kubernetesによる運用が必要な場合はEKSを使用しますが、多くの場合はECSで十分です。

KubernetesをAWS上で運用できるサービスです。EKSは料金面ではGoogle CloudのGKE(Google Kubernetes Engine)よりも割高で、Kubernetesが必要な場合にはGKEの利用をおすすめします。

起動に時間がかかる（数分）ので、オンライン処理には通常は利用できません。バッチ処理などで利用します。

Amazon ECSは以下などを使用してコンテナを立ち上げます。
- AWS Fargate
- AWS EC2
- Amazon ECS on AWS Outposts
などを使用します。

### AWS Fargate

EC2よりは管理する項目が少なく、Lambdaよりは自由度が高い、EC2とLambdaの中間のような存在がFargateです。新規開発ならばLambdaのメリットが圧倒的に強いので見劣りしがちですが、高負荷処理や時間のかかる処理、同時実行数が多い処理などのLambdaでは対応できないようなユースケースではFargateが必要になります。

### AWS EC2

スポットインスタンスを使用するならば安価に処理することができます。

### Amazon ECS on AWS Outposts

オンプレで処理する方法です。これは自社データセンターを持っているような会社が使うものなのでここでは説明を割愛します。


# AWSを使用したWEBアプリケーションの開発方法

どうやってAWSを使用したWEBアプリケーションの開発をおこなってゆけばよいかを説明します。

とは言っても、当然ながらAWSでどうやってWEBアプリケーションを開発してゆけばよいのかは公式ドキュメントによくまとまっているため、ここでは引用に留めます。


## 使用するアーキテクチャ

### サーバーレスアーキテクチャ

## 使用する手法

### CI/CD

## 使用する言語

### プログラミング言語

実装に使用するプログラミング言語は、主にTypeScript、Go、およびRustを使用します。
これらの選定理由は以下の通りです。

- 静的型付けにより、プログラムの正しさの検証が容易になる。
- メジャーな言語であるため困ったときにネット上にも書籍上にも資料がたくさん見つけられる。

#### TypeScript

#### Go

#### Rust

### IDL(インタフェース記述言語)

インタフェースの記述に使用するIDLは、主にProtocol Buffers、GraphQLを使用します。

#### Protocol Buffers

#### GraphQL



## 使用する技術

### Knative

### Docker

AWS Fargate

## Serverless Framework

## React.js・Next.js

# SLA

## 各サービスのSLA一覧

| サービス名 | SLA |
| -- | -- |
| Amazon EC2 | 99.5% |
| Amazon Elastic Kubernetes Service (Amazon EKS) | 99.95% |
| Amazon S3 | 99.9% |
| Amazon CloudFront | 99.9% |
| Amazon Cognito | 99.9% |
| AWS Lambda | 99.95% |
| AWS AppSync | 99.95% |
| Amazon DynamoDB | 99.999% ※1リージョンの場合は99.99% |
| Amazon Route 53 | 100% |

## 月間ダウンタイム早見表(1ヵ月を2592000秒として計算)

|SLA|ダウンタイム(秒)|ダウンタイム(分)|
| -- | -- | -- |
|99.5%|12960|216|
|99.9%|2592|43.2|
|99.95%|1296|21.6|
|99.99%|259.2|4.32|

# 用語集

## リージョン

AWSのデータセンター単位。
世界中にある。日本にも東京と大阪の2つある。

各種料金が北部バージニアリージョンは一番安い。
リージョン間をまたぐ通信にはまた別に料金が発生する。

## VPC

Virtual Private Cloudの略。ローカルネットワークみたいなもの。
特にRDSは通常のケースではインターネットに公開しないのでVPC内で完結する設計にする。
※要注意ポイント：VPCはAZをまたぐことができる。VPC内の通信だからと油断して料金が発生する通信が大量に発生するような設計をしてしまわないように注意。
なお、VPCをまたぐ通信はVGWを作成しないといけないが、今回はVPC間通信は使わないためVGWの説明は省略する。

## サブネット

VPCの中にある小さなネットワークのこと。
サブネットはAZをまたげない。

## サブネットグループ

サブネットのグループのこと。
サブネットグループは複数のAZにまたがっていることが必須となっている。

## セキュリティグループ

ファイアウォールルールのセット。
ちなみにKUSANAGIのデフォルトのセキュリティグループはポート22・80・443を通す設定になっている。

## AMI

アマゾンマシンイメージの略。
EC2のOSのテンプレート。全自動でEC2のインスタンスが利用可能な状態までできあがるので便利。
その用途に応じたセキュリティグループも用意してくれる。

## EC2

仮想マシンみたいなもの。
基本的にAMIから作ります。
もしAMIから作らない場合、いわゆる「秘伝のタレ」化してしまい二度と同じものを作ることができなくなり、障害が起きた時にサービスを継続できない状況にすらなりうるので、必ずAMIから作り、サーバー構築も自動化かつコード化(Infrastructure as Code)します。

## Infrastructure as Code

サーバー構築をコード化することです。
以下のメリットがあります。
- 開発環境が開発者間で差異がなくなり、「この人の開発環境でだけ動かない」みたいなことがなくなる
- 開発環境とテスト環境、また、テスト環境と本番環境の差異が文字ベースでわかるようになり、障害対応がスムーズになる


## Route53

AWSのDNSサーバ。ドメイン取得もRoute53でできる。
普通に使うとただのDNSサーバだが、AWSのいくつかのサービス(CloudFront、Elastic
Beanstalk（いくつか制約があるが）、ELB、S3)に対してAレコードをCNAMEレコードみたいな便利な使い方をすることができます。また、Route53はDNSフェイルオーバーによりリージョン障害だけでなくAWS全体の障害にも対応できるが、長くなるので説明は省略します。
今回はRoute53でAレコードをCloudFrontに紐付ける。これを使いたいためにRoute53以外の選択肢が無くなっている。

## ELB

EC2のロードバランサー。Elastic Load Balancingの略。
ALBとNLBとCLBがある。
昔は一つしかなかったが、あとからALBとNLBができたので従来のELBがCLB(クラシックロードバランサー)となった。
ネット上にはCLBをELBと言ってる記事があるので注意が必要。
ALBがL7ロードバランサー、NLBがL4ロードバランサー、CLBがL4/L7ロードバランサー。
負荷分散のため、高可用性のためにELBで分散する時は複数AZに分散することになる。複数リージョンに分散するのはELBではできない。複数リージョン分散にはRoute53を使うが、設計が非常に煩雑。

## ALB
ELBの中で一番設定が柔軟にできる。
ちなみに、必ず2つ以上のサブネットを選択する必要がある。（そのサブネットたちに処理を分散するため。）

## ACM
HTTPSの証明書を発行してくれる。
外部で証明書を発行しても一応設定はできるものの、これで発行しないとCloudFrontとELBの設定がちょっと面倒。
Route53で取得したドメインはACMで証明書発行が簡単にできる。

## CloudFront
キャッシュサーバみたいなもの。
CloudFrontの証明書は北部バージニアリージョンでないとダメらしいので今回は北部バージニアリージョンで作る。
WordPressなので/wp-admin/はキャッシュしないように設定する。
ちなみに、CloudFrontを通さないとAWSの通信量の請求が高くなる。



## RDS
EC2のデータベース特化版みたいなもの。
ちなみにRDSにはサーバレスというのがあるが全然サーバレスじゃない。

## エンドポイント
各リソースにアクセスするためのURL。
たとえば以下のようになる。
vpce-1234-abcdev-us-east-1.vpce-svc-123345.us-east-1.vpce.amazonaws.com
VPC エンドポイント ID、アベイラビリティーゾーン名、リージョン名などが含まれている。
Route53にはCloudFrontのエンドポイントを、CloudFrontにはELBのエンドポイントを、それぞれ入力する。
また、ブラウザからこのURLを入力すれば(直接見ることができる設定なら)直接見ることもできる。

## SLA
サービス品質保証(Service Level Agreement)の略。
EC2はマルチAZの場合はSLA99.99%を保証しているが、SLA99.99%というのは年間で52.56分停止する計算になる。
（ただし、RDSは99.95%、CloudFrontは99.9%、ELBはSLA99.99%がSLA。（すべてマルチAZでのSLA））
また、シングルAZの場合はSLA90%を保証しているが、これは年間で876時間停止する計算になる。
今回はテスト環境だったのでシングルAZで構築したが、本番環境はもうちょっとお金をかけてマルチAZで構築するようにする。(無料枠には当然収まらない。)
AWS障害のニュースを耳にする機会が増えているが、他人事だと考えず、自分の使っているEC2に障害が起こることを前提として要件定義する(お客さんと、AWS障害をどの程度許容するのか要相談)。

## AZ

アベイラビリティーゾーンの略。
物理的に別の場所に位置していて、1つが災害にあって障害などが発生しても、もう1つは死なないようにすることなどもできる。(マルチAZ構成のこと)
たとえば大阪リージョンの場合は大阪市西区や中央区などにばらけて存在している。
ただし、AZ間をまたぐ通信にはだいたい料金が発生するため注意。

## SLA



## 結果整合性


# 開発の勘どころ

## バッチ処理の実装

以下のような処理はバッチで実装することが多いと思います。
- 日時通知
- 案内メール送信


### AWS Lambda

### 開発言語

プログラミング言語は以下を使用します。

- TypeScript
- Go
- Rust
- Java(GraalVM)

### フレームワーク

プログラミング言語は以下を使用します。

- Serverless Framework

#### コールドスタートを避けようとしない
provisionedConcurrencyでは、指定した以上の接続が発生するとコールドスタートは避けられません。



# 非機能要件

## SLA
## セキュリティ
## アクセシビリティ対応
## 
## 
## 


# ISO 27001を取得しているサービス

実際のシステム開発では、AWSだけでなくLineやSlackなどの外部サービスを利用する必要があるケースもあります。

たとえば運用では、運用者（従業員だけでなく、外部委託なども想定）の管理や、システムからの運用者への通知を管理する必要があります。

運用で使用するためにLineやSlackを使用する場合、「ISO 27001を取得している」ことを以って提案すると通りやすいです。
以下はISO 27001を取得しているサービスの一覧です。

## AWS
## GCP
## Slack
## Line
## GitLab


# クラウドは実績があるのか？

クラウドを使用するために実績の例が必要であれば、以下が参考になるので確認してください。

## AWS
ソニー銀行


## GCP

みんなの銀行：日本初の「デジタルバンク」として Google Cloud に勘定系を構築。Cloud Spanner で銀行基幹システムで求められる可用性を実現
<https://cloud.google.com/blog/ja/topics/customers/minna-no-ginko-spanner>

