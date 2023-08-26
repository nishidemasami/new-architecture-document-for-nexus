# PlantUMLテスト


```uml
@startuml

!define AWS_BG_COLOR #FFFFFF

!include ../plantuml/AWS_IMAGES.puml

' Groups are rectangles with a custom style using stereotype - need to hide
hide stereotype
skinparam linetype ortho
skinparam rectangle {
	BackgroundColor AWS_BG_COLOR
	BorderColor transparent
}
skinparam Arrow {
	Color #666666
	FontColor #666666
	FontSize 12
}
skinparam defaultTextAlignment center
skinparam PackageTitleAlignment Left

rectangle "UserIMG\nユーザー" as user

rectangle "AWSCloudIMG AWS Cloud" <<CloudGroup>>{
	rectangle "RegionIMG Region" <<RegionGroup>>{

		rectangle "LambdaLambdaFunctionIMG\nLambda\n処理" as lambda
			user <--> lambda: ④\lなんらかの処理をPost

		rectangle "CognitoIMG\nCognito\n認証管理" as Cognito
			lambda <-> Cognito: ユーザー認証管理
			user <--> Cognito: ③\l認証

		rectangle "SimpleStorageServiceIMG\nWEBサイト" as websites3

		rectangle "AppSyncIMG\nAppSync" as AppSync
			AppSync <-> Cognito: ユーザー認証管理

		rectangle "DynamoDBIMG\nDynamoDB" as DynamoDB
			AppSync <--> DynamoDB: ⑥\lデータ受け渡し
			user <--> AppSync: ⑤\lなんらかのデータ要求
	}

	rectangle "CloudFrontIMG\nCroudFront\nCDN" as cloudfront
		cloudfront <-> websites3: ②\l静的ファイル配信
		user --> cloudfront: ①\lWEBサイト\n訪問
}

@enduml
```
