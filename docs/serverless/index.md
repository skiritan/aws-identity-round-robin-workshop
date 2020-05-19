# サーバーレス アイデンティティ ラウンド

<!--
Welcome to the world of serverless!  Now you may be asking yourself, *What is serverless*? Well, it is an architecture paradigm that allows you to create your applications without provisioning or managing any servers.  Sounds great, right?  Organizations look at building serverless applications as a way of improving their scalability and reducing their operational overhead.  The responsibility of the underlying infrastructure is shifted off your plate so you can spend more time focusing on building your applications.

So with less infrastructure to manage you are no longer responsible for patching  your operating systems and the attack surface you need to worry about has been significantly reduced.  But with the use of serverless technologies comes *other* responsibility.  When you hear the word serverless you may think specifically of <a href="https://aws.amazon.com/lambda/" target="_blank">AWS Lambda</a> but it is important to remember that there are other services used within a serverless application and securing an application involves more than just securing your Lambda functions.  
-->

このラウンドでは、WildRydes というサーバーレスアプリケーション ( [aws-serverless-workshops](https://github.com/aws-samples/aws-serverless-workshops/tree/master/WebApplication) をこのラウンドの目的に合わせて変更したもの) のアイデンティティコントロールを改善することに重点を置いています。[AWS IAM](https://aws.amazon.com/iam/)、[Amazon S3](https://aws.amazon.com/s3/)、[Amazon CloudFront](https://aws.amazon.com/cloudfront/)、および [Amazon Cognito](https://aws.amazon.com/cognito/) などのさまざまなサービスを使用することで、異なるアイデンティティの概念に触れることができます。このラウンドを完了すると、標準の AWS アイデンティティコントロール機能を使用して、サーバーレスアプリケーションのセキュリティをよりコントロールする方法が理解できます。

**AWS のサービス/機能の範囲** : 

* S3 バケットポリシー
* S3 ACL
* CloudFront オリジンアクセスアイデンティティ
* Cognito ユーザープール
* Cognito Hosted UI

## **アジェンダ**

このラウンドは２つのタスクに分けられ、どちらも構築フェーズと検証フェーズがあります。構築フェーズでは、一連のビジネスレベルの機能要と非機能要件に基づいて、WildRydes アプリケーションのアイデンティティコントロールを評価、実装、および強化します。検証フェーズでは、エンドユーザーに代わって、要件が満たされていることを確認するために、設定内容をテストします。また、システム管理者が引き続きリソースを管理できることも確認します。

* **タスク１** (40分): S3オリジンの攻撃対象となりうる箇所を減らす
* **タスク２** (35分): アプリケーションのユーザ管理を設定する

!!! info "チーム演習または個人演習"
    このワークショップは、チームで行うことも、個人で行うこともできます。手順は、チームの一員として作業することを前提に記述されていますが、以下の手順を個人で実行することも可能です。AWS 主催のイベントとして行われる場合は、4～6 人程度のチームに分かれて実施します。各チームが構築フェーズを実施し、アカウントを別のチームに引き継ぎます。その後、各チームは検証フェーズを実行します。

## プレゼンテーション

<a href="./Identity-RR-Serverless-Round_JP.pdf" target="_blank">ワークショップのプレゼンテーション</a>

## 環境設定

環境を設定するには、次のドロップダウンセクションのいずれか (このワークショップの実施方法に応じて) を展開し、指示に従ってください。 

??? info "AWS 主催の Event Engine を利用したイベント" 

    <p style="font-size:20px;">
      **Step 1** : AWS コンソールへの接続
    </p>
    
    1. <a href="https://dashboard.eventengine.run" target="_blank">Event Engine ダッシューボード</a>にアクセスします。
    2. **チームハッシュコード** を入力します. 
    3. **AWS Console** をクリックします。（このラウンドのCloudFormationテンプレートは次善実行されています）

??? info "個人の AWSアカウントを利用" 

    リージョン| デプロイ
    ------|-----
    US East 1 (N. Virginia) | <a href="https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=Identity-RR-Wksp-Serverless-Round&templateURL=https://s3-us-west-2.amazonaws.com/sa-security-specialist-workshops-us-west-2/identity-workshop/serverless/environment.yml" target="_blank">![Deploy in us-east-1](./images/deploy-to-aws.png)</a>
    
    1. 上の **Deploy to AWS** ボタンをクリックします (右クリックして新しいタブで開きます)。これにより、テンプレートを実行するコンソールに自動的に移動します。 
    
    2. テンプレートの指定 セクションで **次へ** をクリックします。
    
    3. **スタックの詳細を指定** ステップで、**次へ** をクリックします。 
    
    4. **スタックオプションの設定** セクションで **次へ** をクリックします。
    5. 最後に、画面下部の **機能** の「テンプレートによって IAM ロールが作成されることを承認の **チェックボックスを有効** にし、**スタックの作成** をクリックします。
    
    上記を実行すると CloudFormation コンソールに戻ります。ページを更新すると、スタックの作成が開始されることが確認できます。次に進む前に、スタックが **CREATE_COMPLETE** になったことを確認してください。

## WildRydes のアイデンティティの改善

あなたは、動物のライドシェアアプリケーションを管理する新しい DevOps チームに参加しました。あなたはセキュリティに関する経験が買われて、セキュリティ関連のタスクを主導し、セキュリティのベストプラクティスを広め、セキュリティチームとの橋渡しを行う役割としてチームに加わっています。最近、チームは、WildRydes という新しいアプリケーションを引き継ぎました。

## アプリケーションの表示
1. <a href="https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks?filter=active" target="_blank">Amazon CloudFormation</a> コンソール (us-east-1) を開きます。
2. **Identity-RR-Wksp-Serverless-Round** スタック (Event Engine利用時は   **module-a7932bd25ca64049a57fd5bb055782db** スタック) をクリックします。
3. **出力** をクリックし、**WebsiteCloudFrontURL** をクリックします。

チームへの引き継ぎの一環として、製品チームはアプリケーションのビジョンを共有し、将来のバージョンにはより動的な機能が含まれることになるだろうと述べました。アーキテクチャを確認したところ、WildRydes アプリケーションは S3 バケットでホストされる静的なウェブサイトであることがわかりました。コンテンツ配信ネットワークとして使用される CloudFront ディストリビューションが設定され、ユーザー管理には Cognito ユーザープールが使用されています。

## 現在のアプリケーションアーキテクチャ

![Architecture](./images/architecture-start.png)

チームでアーキテクチャの評価や、脅威モデリングを行った結果、動作していない機能や設定ミスを多数発見しました。どうやら、セキュリティ制御は完全に実装できていなかったようです。このレビューの結果、複数のタスクがチームのバックログに追加され、優先度の高いタスクがいくつか作成されました。

***

Next (次へ) をクリックし、**タスク１**に進みます。