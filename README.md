# 本ドキュメント「よりよいシステム開発の手引き」について

本ドキュメント「よりよいシステム開発の手引き」は、よりよいシステム開発のための手引きです。  
以下について4つの章に分けて説明しています。

- AIによるプログラム生成により開発されたプログラムの品質を担保する手法
- 生産性が高く修正の追跡が可能な仕様書の管理手法
- 保守性が高くセキュリティが高いシステム開発の手法
- 利用者が増えても自動でスケールするシステムの開発、および運用手法

本ドキュメントはMarkdownで書かれGitHubで公開されており、そしてまさに今も追加され続けており誰でも(あなたも！)加筆修正に参加することができます。  
また、誤字脱字を発見したり内容に誤りがあれば、いつでもGitHub Issuesに投稿してください。

- GitHub https://github.com/nishidemasami/new-architecture-document-for-nexus
- GitHub Issues https://github.com/nishidemasami/new-architecture-document-for-nexus/issues

Markdownで書かれたドキュメントは、誰もが読みやすいようにHTMLとPDFで公開しています。  
HTML版はHonkitで生成されGitHub Pagesで公開されており、GitHub Actionsによって常に最新情報に更新され続けています。(MermaidやPlantUMLはHTML版だと少し表示が怪しいですが……。)  
PDF版はCalibreで生成されGitHub Releasesで管理されています。これもHTML版と同様にGitHub Actionsによって常に最新情報に更新され続けています。  

- HTML版 https://nishidemasami.github.io/new-architecture-document-for-nexus/  
- PDF版 https://github.com/nishidemasami/new-architecture-document-for-nexus/releases/download/pdf_relrase/new-architecture-document-for-nexus.pdf  

## おことわり

- このドキュメントでは®マーク、™マークなどの登録表示に関するマークは省略しております。この省略はいかなる意味においても登録表示を軽視する意図を持ちません。
- 記載の会社名、団体名、機関名、個人名、組織名、製品名、およびアイコンには、会社/団体の商標または登録商標などを一部含みます。
- このドキュメントで紹介するサービスはすべて国際的な情報セキュリティ規格である「ISO/IEC 27001」を取得していることを確認していますが、実際にサービスを使用する際にはその案件で求められているセキュリティ要件を満たすかどうかを各自別途確認してください。

## 各章の説明

* [本ドキュメント「よりよいシステム開発の手引き」について](README.md)

本章です。

* [AIによるプログラム生成（またはフリーランス・オフショア・未熟なプログラマーなど）により開発されたプログラムの品質を担保する手法](markdown/how_to_develop.md)

この章では、プログラムの品質担保の手法について説明します。

* [生産性が高く修正の追跡が可能な仕様書管理手法]()

この章では、要件定義書や仕様書などのドキュメント管理の手法について説明します。

* [保守性が高くセキュリティが高いシステム開発の手法]()

この章では、保守性やセキュリティをシステム開発の段階からコントロールするための手法について説明します。

* [利用者が増えても自動でスケールするシステムの開発手法]()

この章では、スケーラビリティをシステム開発の段階からコントロールするための手法について説明します。

* [付録：Mermaid記法 サンプル集](markdown/appendix/mermaid_samples.md)

* [付録：PlantUML記法 サンプル集](markdown/appendix/plantuml_samples.md)

* [付録：Markdown記法 サンプル集](markdown/appendix/markdown_samples.md)
