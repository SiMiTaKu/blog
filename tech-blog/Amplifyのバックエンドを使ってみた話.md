# Amplifyのバックエンドを使ってみた話


# MEMO
## cdk bootstrap（CDKToolkit）ってなんだ？
まず[公式ドキュメント](https://docs.aws.amazon.com/ja_jp/cdk/v2/guide/bootstrapping.html)を読んでみる。

### ブートスとラッピング
ブートストラップはAWS Cloud Development Kit（AWS CDK）で使用するAWS環境を準備するプロセスです。 
CDKスタックをAWS環境に、デプロイする前にまず環境をブートストラップする必要があります。


### ブートストラップとは？

CDKが使用するAWS環境内の特定のAWSリソースをプロビジョニングすることで、環境を準備します。
準備されるリソースは以下のとおり

| リソース名     | 説明 |
| -------------- | ---- |
| Amazon Simple Storage Service (Amazon S3) バケット | AWS Lambda 関数コードやアセットなどの CDK プロジェクトファイルを保存するために使用されます。 |
| Amazon Elastic Container Registry (Amazon ECR) リポジトリ | Docker イメージを保存するために使用されます。 |
| AWS Identity and Access Management (IAM) ロール | デプロイを実行するために が必要とするアクセス許可を付与 AWS CDK するように設定されています。 |

### ブートストラップの仕組み
CDKで使用されるリソースとその設定は、AWS CloudFormationテンプレートで定義されます。
このテンプレートはCDKチームによって作成および管理されます。
このテンプレートの最新バージョンについては、aws-cdkのGitHubリポジトリーの「[
  bootstrap-template.yaml
](
  https://github.com/aws/aws-cdk/blob/main/packages/aws-cdk/lib/api/bootstrap/bootstrap-template.yaml
)」を参照してください。

環境をブートストラップするには、AWS CDKコマンドラインインターフェイス（AWS CDK CLI）cdk bootstrapコマンドを使用します。CDKはテンプレートCLIを取得し、ブートストラップスタックと呼ばれるスタックAWS CloudFormationとしてにデプロイします。デフォルトでは、スタック名はですCDKToolkit。このテンプレートをデプロイすることで、は環境にリソースをCloudFormationプロビジョニングします。デプロイ後、ブートストラップスタックが環境のAWS CloudFormationコンソールに表示されます。

テンプレートを変更するか、cdk bootstrapコマンドでCDK CLIオプションを使用して、ブートストラップをカスタマイズできます。

AWS CDKで使用する各環境は、AWS環境は独立しています。まずブートストラップする必要があります。

