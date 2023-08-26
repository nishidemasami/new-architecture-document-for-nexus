# PlantUMLテスト


```uml
@startuml
'Copyright 2019 Amazon.com, Inc. or its affiliates. All Rights Reserved.
'SPDX-License-Identifier: MIT (For details, see https://github.com/awslabs/aws-icons-for-plantuml/blob/master/LICENSE)

!define AWSPuml https://raw.githubusercontent.com/awslabs/aws-icons-for-plantuml/v15.0/dist

!include AWSPuml/AWSCommon.puml
!include AWSPuml/AWSExperimental.puml
!include AWSPuml/Groups/all.puml
!include AWSPuml/Compute/LambdaLambdaFunction.puml
!include AWSPuml/General/Documents.puml
!include AWSPuml/General/Multimedia.puml
!include AWSPuml/General/Tapestorage.puml
!include AWSPuml/General/User.puml
!include AWSPuml/MediaServices/ElementalMediaConvert.puml
!include AWSPuml/MachineLearning/Transcribe.puml
!include AWSPuml/Storage/SimpleStorageService.puml

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
sprite Callout_1 <svg width="18" height="18"><circle cx="9" cy="9" r="9" fill="black" /><text x="5" y="13" fill="#FFFFFF" font-size="12">1</text></svg>

rectangle "$UserIMG()\nユーザー" as user
AWSCloudGroup(cloud){
	RegionGroup(region) {
		S3BucketGroup(s3) {
			rectangle "$MultimediaIMG()\n\t動画\t" as video
			rectangle "$TapestorageIMG()\n\t音声\t" as audio
			rectangle "$DocumentsIMG()\n\tテキスト\t" as transcript

			user -r-> video: <$Callout_1>\lアップロード
			video -r-> audio
			audio -r-> transcript
		}

		rectangle "$LambdaLambdaFunctionIMG()\nObjectCreated\nevent handler" as e1
		rectangle "$ElementalMediaConvertIMG()\nAWS Elemental\nMediaConvert" as mediaconvert
		rectangle "$TranscribeIMG()\nAmazon Transcribe\n" as transcribe

		video -d-> e1: <$Callout_2>
		e1 -[hidden]r-> mediaconvert
		mediaconvert -[hidden]r-> transcribe
		mediaconvert -u-> audio: <$Callout_3>
		transcribe -u-> transcript: <$Callout_4>

		StepFunctionsWorkflowGroup(sfw) {
			rectangle "$LambdaLambdaFunctionIMG()\nextract audio" as sfw1
			rectangle "$LambdaLambdaFunctionIMG()\ntranscribe audio" as sfw2

			e1 -r-> sfw1: 起動
			sfw1 -r-> sfw2
			sfw1 -u-> mediaconvert
			sfw2 -u-> transcribe
		}
	}
}
@enduml
```
