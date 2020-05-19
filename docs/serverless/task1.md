# サーバーレス アイデンティティ ラウンド <small>タスク 1</small>

あなたは、チームのセキュリティタスクを担当者として、WildRydes アプリケーションの 2 つのタスクを実行していきます。以下の手順をすすめてタスクを完了させてください。

## 構築フェーズ  <small>S3オリジンの攻撃対象となりうる箇所を減らす</small>

アプリケーションは CloudFront ディストリビューション経由でコンテンツを配信しています。攻撃対象となりうる箇所を減らすため、エンドユーザーは CloudFront の URL 経由で**のみ**アクセス可能とし、Amazon S3 の URL には直接アクセスできないよう設定を変更してください。またこの設定変更によって、エンドユーザーがアプリケーションの可用性や整合性に影響を与えないようにしてください。  

!!! info "セキュリティ上の利点"

    * S3 オリジンを難読化する
    * カスタム証明書を使用し、強制的に HTTPS トラフィックを利用するようにする
    * アプリケーションに DDoS 保護を追加し、将来的に <a href="https://aws.amazon.com/waf/" target="_blank">AWS WAF</a> や <a href="https://aws.amazon.com/shield/" target="_blank">AWS Shield</a> を使用できるようにする

### 既存ポリシーの確認

まず、既存の S3 バケットポリシーを表示し、前任のエンジニアが作成したアクセス権を確認します。

1. <a href="https://s3.console.aws.amazon.com/s3/home" target="_blank">Amazon S3</a> コンソールに移動します。  
2. **identity-wksp-serverless-<*ACCOUNT#*\>-us-east-1-wildrydes** バケットをクリックします。  .
3. **アクセス権限** タブをクリックし、**バケットポリシー** をクリックします。

!!! question "このポリシーのどこが間違っているのでしょうか？ `"Principal": "*"` はどういう意味でしょうか? "
    

    `"Principal": "*"` および `"Principal":{"AWS":"*"}` の両方とも、全員にアクセス権を付与しています (匿名アクセスとも呼ばれます)。S3 バケットへの匿名アクセスを許可する場合は注意が必要です。匿名アクセスを許可すると、世界中の誰もがこのバケットにアクセス可能です。S3 バケットへの匿名書き込みアクセス権は許可しないことを強くお勧めします。

### プリンシパルの変更

現在の設定では匿名アクセスが許可されているため、これを変更して CloudFront ディストリビューションからのアクセスのみを許可する必要があります。

1. <a href="https://console.aws.amazon.com/cloudfront/" target="_blank">Amazon CloudFront</a> コンソールに移動します。WildRydes ウェブアプリケーションのウェブディストリビューションが表示されます。
2. 左のナビゲーションで **Origin Access Identities (オリジンアクセスアイデンティティ)** をクリックします。**Unicorn OAI** というアイデンティティが表示されます。 

    !!! info "CloudFront オリジンアクセスアイデンティティ"
        オリジンアクセスアイデンティティ (OAI) とは、AWS IAM を使用してアクセスを制限することができる、ディストリビューションに関連付けることができる特別な CloudFront アイデンティティです。ウェブディストリビューションを表示して OAI を見つけることもできます。

3. ID をコピーします。
4. <a href="https://s3.console.aws.amazon.com/s3/home" target="_blank">Amazon S3</a> に戻り、バケットポリシーを開きます。  
5. **principal** を以下のように置き換えて **保存** をクリックします。

``` json
"Principal": {
	"AWS": "arn:aws:iam::cloudfront:user/CloudFront Origin Access Identity <OAI ID>"
},
```

!!! info "補足"

    次のように、CanonicalUser(正規ユーザー) IDをプリンシパルとして使用することもできます。"CanonicalUser": "<OAI S3CanonicalUserId>": `"CanonicalUser": "<OAI S3CanonicalUserId>"`

### アクションの変更

これで、プリンシパルは CloudFront ディストリビューションに関連付けられたアイデンティティに制限されました。引き続きアクセス権を詳細に確認していきます。

1. <a href="https://s3.console.aws.amazon.com/s3/home" target="_blank">Amazon S3</a> に戻り、バケットポリシーを開いて確認します。  

    !!! question "CloudFront には、オブジェクトを削除するアクセス権が本当に必要でしょうか？"
	
2. ディストリビューションは静的なサイトの CDN として動作するため、必要なのは S3 バケットへの読み取りアクセスのみです。エンドユーザーがサイトの整合性に影響を与えないように、アクションを変更します。（手順は記載していません）

### 新しいバケットポリシーのテスト

バケットポリシーが更新されたので、S3 の URL を使用してウェブサイトに直接アクセスできないことを確認します。

1. <a href="https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks?filter=active" target="_blank">Amazon CloudFormation</a> コンソール (us-east-1) を開きます。  
2. **Identity-RR-Wksp-Serverless-Round** スタック （または **module-a7932bd25ca64049a57fd5bb055782db** スタック) をクリックします。  
3. **出力** をクリックし、**WebsiteS3URL** をクリックします。

!!! question "まだ S3 の URL を使用してサイトにアクセスできますか？"

### 残された問題への対応

バケットポリシーを変更して、CloudFront ディストリビューションからのみの読取りアクセス許可としましたが、何らかの理由で S3 の URL を使用したアクセスができています。引き続き調査を行い、その理由を解明して、トラフィックを制限するために設定を追加してください。

!!! ヒント
    S3 には他にどのようなアクセスコントロールがあるでしょうか?  次のリソースを確認してください。

    * <a href="https://docs.aws.amazon.com/AmazonS3/latest/dev/access-control-block-public-access.html" target="_blank">S3 ブロックパブリックアクセスの使用</a> (最も簡単)
    * <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_notprincipal.html" target="_blank">AWS IAM ポリシーの要素: NotPrincipal </a> (難しい)
    
    動作確認時には、必ずキャッシュをクリアしてください。


## 検証フェーズ  <small>S3オリジンの攻撃対象となりうる箇所を減らす</small>

 ここまでで、アプリケーションにアイデンティティコントロールが追加されたので、あなたはエンドユーザーとして手動テストを行い、コントロールが正しく導入され、要件が満たされていることを確認する必要があります。

アプリケーションが *CloudFront* ディストリビューション経由でコンテンツを提供すること、さらにはエンドユーザーが *CloudFront* の *URL* によってのみアプリケーションにアクセスでき、Amazon S3 の URL からはアクセスできないことを確認してください。この設定変更により、エンドユーザーがアプリケーションの可用性または整合性に影響を与えることがあってはいけません

!!! tip "確認チェックリスト"

    * CloudFront ディストリビューション URL (<WebsiteCloudFrontURL\>)からサイトにアクセスできます。
    
    * S3 URL (<WebsiteS3URL\>)経由でのアプリケーションリソースへのアクセスが制限されています。
    
        > フォルダ以下にあるリンクも試してください (例: <WebsiteS3URL\>/js/vendor/unicorn-icon)
    
    * いずれのアプリケーションも、CloudFront ディストリビューションを経由して削除または変更することはできません。
    
        > curl や Postman などを使用して、異なる HTTP メソッド (例: Delete) でリクエストを実行してみてください。以下に、curl を使用した例を示します。   
    
        ```
        curl -i -X DELETE <WebsiteCloudFrontURL>/index.html
        ```
***

以上のタスクを完了したら、タスク２に進んでください。