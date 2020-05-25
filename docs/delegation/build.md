# アクセス委任 ラウンド (構築フェーズ)

| 重要な注意事項 |
|---|
| 以下の手順に進む前に、必ず **[シナリオセクション](./index.md)** をお読みください。 |
|---|
| AWS コンソールには、AWS 機能に関するニュースや情報が表示される場合があります。その場合はウィンドウを自由に閉じて、画面を広げてください |
| チームでアカウントを共有している場合、一部のサービスは一度に1人しか使用できないため、各自でコンソールを使用する必要があります。 |


## AWS へのサインイン

このワークショップの実施方法に応じて、以下のドロップダウンのいずれかを展開して AWS にサインインします。  

??? info  "AWS 主催の Event Engine を利用したイベント " 

    <p style="font-size:20px;">
      **Step 1** : Event Engineの一時的な資格情報を取得する
    </p>
    
    1. <a href="https://dashboard.eventengine.run" target="_blank">Event Engine dashboard</a> ダッシューボードにアクセスします。
    2. **チーム ハッシュコード** を入力します。
    3. **AWS Console** をクリックします。
    4. 資格情報セクションの下のエクスポートコマンドをコピーします（後の手順で必要になります）。
    5. Event Engine ウィンドウで **コンソールを開く** をクリックします。 

??? info "Click AWS 主催のイベント"

    1. ウェブブラウザの別のタブで、指示された URL に移動し、ログインします。 
    
    2. ログイン後、AWS Account (AWS アカウント) ボックスをクリックし、その下に表示されている Account ID  (画像の赤いボックス) をクリックします。マネジメントコンソールの下にリンクが表示されるので、これをクリックして、AWS コンソールに移動します。  
    
    ![login-page](./images/login.png)

??? info "個人の AWS アカウントを使用している場合"

    ウェブブラウザの別のタブで、https://aws.amazon.com/console にアクセスし、アカウントにログインします。

## Macie を有効にする

1. このラボでは Macie を利用するので、まず Amazon Macie が有効になっているか確認します。メインコンソールから **Macie** を選択します。

2. **米国東部（バージニア北部）**リージョンを選択します。

3. Macie コンソールで、**Getting Started** ボタンが表示された場合は、Amazon Macie が無効になっています。その場合は、**Getting Started** をクリックし、リージョンにバージニア北部が選択されていることを確認して、**Macie を有効化** をクリックします。

## GuardDuty を有効にする

1. このラボでは GuardDuty も利用するので、GuardDuty を有効にします。

2. メインコンソールから **GuardDuty** を選択します。
3. **今すぐ始める** ボタンが表示されたら、クリックします。  

4. リージョンがバージニア北部に設定されていない場合は、リージョンを **バージニア北部** を選択します。

5. **GuardDutyの有効化** をクリックします。

## 環境の構築

1. **新しいブラウザタブ** で次の **Deploy to AWS** リンクを開き、us-east-1 リージョンにロギング環境をデプロイします  :  [![Deploy IamEssInitAccount.yaml in us-east-1](./images/deploy-to-aws.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=esslab&templateURL=https://s3.amazonaws.com/sa-security-specialist-workshops-us-east-1/identity-workshop/essround/EssInitAccount.yaml)

3. スタックの作成 ページが表示されます。**次へ** をクリックします。

4. スタックの詳細を指定 ページにはスタックの名前(**esslab**)が自動入力されています。**次へ** をクリックします。

5. スタックオプション ページで、**次へ** をクリックします。

6. レビュー ページで、**IAMリソース作成についての承認** チェックボックスをオンにし、**スタックの作成** をクリックします  
CloudFormation がリソース作成を開始します。**この作成には約 5 分かかります**。ブラウザウィンドウを更新して進捗状況を確認し、**esslab** スタックのステータス値に **CREATE_COMPLETE** が表示されるまで待機します。

6. **esslab** の CloudFormation スタックの **出力** を確認してください。以下の図のようになります。

    ![Account output](./images/IamEssOutputStack.png)

    *LoggingBucketName* は、AWS CloudTrail がログを配信するバケット名です。後ほど参照できるように、この値をテキストエディタ等にコピーしておきます。

    *SecAdministratorRoleURL* は、後ほどのラボで一時的にセキュリティ管理者ロールに「切り替える」 (セキュリティ管理者のアクセス権限を取得する) ために使用する URL です。このロールには AWS CloudTrail、Amazon GuardDuty、Amazon Inspector、Amazon Macie に対する完全な管理権限があります。

    *SecOperatorRoleURL* は、後ほどのラボで一時的にセキュリティオペレーターロールに「切り替える」ために使用する URL です。このロールのポリシーを後ほど変更して、AWS CloudTrail、Amazon GuardDuty、Amazon Inspector、Amazon Macie の「読み取り専用」の権限のみを持つようにします。

7. CloudTrail のログ保存の設定を見てみましょう。CloudTrail の情報を S3 バケットに収集する上で最も重要なことは、S3 バケットに適切なアクセス権が設定されていることです。

    S3 コンソールに移動し、手順 6 でメモした *LoggingBucketName* のバケットをクリックした後、**アクセス権限** をクリックし、**バケットポリシー** をクリックします。このポリシーにより、CloudTrail サービスで *LoggingBucket* の ACL を読み取ることができ、さらには AWS アカウント ID を含むプレフィックスを持つログを作成することもできるようになっています。このポリシーの詳細については この[リンク](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/create-s3-bucket-policy-for-cloudtrail.html)を参照してください。

8. **概要** タブをクリックし、**AWSLogs** フォルダをクリックします。そうすると、以下に示すようにアカウントの AWS アカウント ID が表示されます。

    ![LoggingBucket](./images/IamEssBucket.png)

    これは、アカウントの AWS CloudTrail ログが S3 バケットに到達し始めたことを示しています。ログが表示されるまでに最大5分かかる場合があります。さらに年、月、日ごとのディレクトリに移動してログを確認できます。ログは「.gz」（gzip）形式で保存されます。ファイル解凍ツールがあり、これらのログファイルの中身を確認したい場合は、お使いのワークステーションにダウンロードしてアーカイブを展開してみてください。繰り返しますが、ログが表示されるまでに数分かかる場合があります。

## ロールについて

ロールとは、複数のポリシーおよび信頼関係を含む、セキュリティプリンシパル、アクターです。ユーザー、アプリケーション、AWS のサービスは、ロールに関連付けられたアクセス権を代わりに使用するため、そのロールを*引き受ける*ことができます。*ポリシー*は、ロールが実行できる AWS アクションを定義します。*信頼関係*によって、ロールを引き受けることができるユーザーが定義されます。

先ほど作成した CloudFormation スタックでは 2 つのロールが作成されました。1 つは、CloudTrail、GuardDuty、Inspector、Macie へのフルアクセス権を持つセキュリティ管理者ロールです。もう 1 つは、それらのサービスへの読み取り専用のアクセス権を持つセキュリティオペレーターロールです (このロールは、後ほど変更を加えて完成します)。これからセキュリティ管理者ロールに切り替えますが、その前に、ロールに関連付けらた権限を確認し、どのように動作するかを確認していきます。

1. IAM コンソールに移動し、左側にある **ロール** を選択し、文字列 *SecAdministrator* を検索します。ブラウザの検索機能することも、検索ボックスに「*SecAdministrator*」と入力して検索することもできます。検索されたロールをクリックします。
    ポリシー定義は以下の図のようになります  

    ![SecAdministratorRolePolicy](./images/IamEssSecAdminPolicy.png)

    すべてのポリシーを表示するために、**さらに表示** をクリックします。このロールには 6つのマネージドポリシーが含まれており、そのうちの 5 つは AWS が GuardDuty、Inspector、CloudTrail、IAM、SNS 用に提供しているものです。このラボでは SNS と IAM を直接操作することはありませんが、コンソールの操作性向上のためにそれらのポリシーも含まれています。

    Amazon Macie のための 6 番目のマネージドポリシーは、カスタムマネージドポリシーの作成方法について説明するためのものです。各マネージドポリシーをクリックして、それぞれの権限を確認してください。Amazon EFS (Elastic File System) など、このポリシーでは許可されてないサービスもあることにも注意してください。この許可されていないアクセス権については、ワークショップの後半で確認します。  

2. ロールの概要ページで、**信頼関係** タブをクリックします。ページには信頼できるエンティティ (ロールを引き受けることができるエンティティ) のセクションがあり、以下のような12 桁の AWS アカウント ID が表示さています。これは、このアカウントのプリンシパルであればこのロールを引き受けることができることを意味しています。

    ![SecAdministratorTrustRel](./images/IamEssAdminTrustRel.png)

3. 信頼関係の設定の中身は JSON ポリシードキュメントです。**信頼関係の編集** をクリックすると、以下のようなポリシーが表示されます。

    ![SecAdministratorTrustRel](./images/IamEssAdminTrustPol.png)

    プリンシパルエントリには、「：root」で終わる値が表示されます。これは、アカウント自体を参照する特別なプリンシパル識別子です。このポリシーは、アカウントのすべてのプリンシパル（ユーザー、ロールなど）が sts：AssumeRole アクションを使用してロールのIDを取得することを許可することを示しています。プリンシパル識別子の詳細については、[こちら](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_identifiers.html)をご覧ください。

    STS とは *Security Token Service* の略です。アクセスキーとシークレットアクセスキーを含むロールのセキュリティポリシーに関連付けられた API が呼び出されると、サービスに関連付けられた AssumeRole API によって、一時的な認証情報が作成されます。アプリケーションはこのセキュリティトークンを使用して、ロールに定義されたポリシーを使用して他の AWS API を呼び出します。

## セキュリティ管理者ロールへの切り替え

ここでは、AWS コンソールでロールを切り替える方法について説明します。ロールを切り替えると、AWS コンソールの権限を別のロールに含まれる権限に一時的に置き換えることができます。このラボでは、自分の AWS アカウントのロールに切り替えます。他の AWS アカウントのロールに切り替えて、自分のアカウント以外のリソースにアクセスすることもできます。これは*クロスアカウントアクセス*と呼ばれます。

ロールを切り替えるには2つの方法があります。コンソールから自分のアカウント名をドロップダウンし[スイッチロール]をクリックする方法。または、切り替えを行うURLを使用する方法です。今回は両方の方法を使用してみます。  

1. **CloudFormation コンソール**に移動し、構築した *esslab* という名前の CloudFormation スタックの **出力** タブを表示します。

2. *SecAdministratorRoleURL* の隣にある URL をクリックします。（この URL を表示するには下にスクロールする必要がある場合があります） 新しいブラウザタブウィンドウが開き、以下のような画面が表示されます。

    ![SecAdministratorRole](./images/IamEssSwitchSecAdminRole.png)

    ボックスには、アカウント ID (AWS アカウントの ID)、CloudFormation によって作成されたロール名、および表示名が含まれます。切り替えるロールの表示色も選択することもできます。

    **ロールの切り替え** をクリックします。

    下記のように、コンソールウィンドウの上部に *SecAdministrator* という名前の新しいロールラベルが表示されます。これは、 SecAdministrator ロールの権限が有効な権限として「一時的に」置き換えられたことを意味しています

    ![SecAdministratorLabel](./images/IamEssSwitchSecAdminRolePost.png)

    では、セキュリティ管理者としてどのような権限があるか、各サービスに順番にアクセスして確認していきます。

3.  **Amazon EFS コンソール**に移動します。コンソールにアクセスできないことを示すメッセージが表示されます。これは、このロールのポリシーが Amazon EFS へのアクセスを許可していないためです。

    ![EFSConsole](./images/IamEssEFSConsole.png)

4. **Amazon Inspector コンソール**に移動します。Inspectorの管理権限があることを確認するために、既存のテンプレートのクローンを作成してみます。**評価テンプレート** をクリックし、*LampInspectorAssessmentTemplate* で始まるテンプレート名を選択して **クローン** をクリックします。フォームのセクションが表示されるので、下にスクロールダウンし[ **作成** ]ボタンをクリックします。画面を更新すると、*LampInspectorAssessment* で始まる2つのテンプレートが表示され、[名前] を広げると、新しくクローンが作成されていることがわかります。評価テンプレートを正常に複製できたことで、Inspector の管理アクセス権限を持っていることが確認できました。

5. **GuardDuty コンソール** に移動し、**設定** メニュー項目を選択します。**結果のエクスポートオプション** の **更新された結果の頻度** という名前のフィールドまでスクロールし、値を[**1時間ごとに CWE と S3 を更新する** ]に変更し、**保存** をクリックします。ウィンドウの上部に、設定が保存されたことを示すメッセージが表示されます。これは、GuardDutyに完全にアクセスできることを示しています。

6. (2020年5月25日現在：Macieの機能拡張により正しくアクセスできない場合があります。その場合は次へ進んでください)
    **Macie コンソール**に移動し、**US East (N. Virginia)** リージョンを選択して、**設定** をクリックし、**Content Type**アイコンをクリックします。そうすると、ファイルタイプのリストが表示されます。*application/cap* などのファイルタイプの編集ボタン(鉛筆マーク) をクリックして、*Enabled* フラグの値を ***No-disabled***  に変更して**Save** します。これは同様に、Macie への管理アクセス権があることを示しています。

7. **CloudTrail コンソール**に移動します。

8. **証跡情報** をクリックし、名前が *esslab* で始まる証跡をクリックします。

9. 画面右上の **ログ記録** スイッチを OFF (オフ) にすると、確認メッセージが表示されるので **次へ** をクリックします。さらに、 **ログ記録** を ON (オン) に戻します。これにより、CloudTrail への管理アクセス権があることが確認できます。

10. Inspector、Macie、GuardDuty、および CloudTrail への管理アクセス権があることを確認が終了しました。以前のロールに戻るために、SecAdministrator ラベルをクリックし、メニューの右下にある **…..****に戻る** を選択します。（コンソールにはロールの履歴が保持されるため、後でロールを簡単に切り替えることができます）

    ![SwitchBacktoAdmin](./images/IamSwitchFromSecAdmin.png)

    通常のロールに戻ると、SecAdministrator のロールラベルは表示されなくなります。

## セキュリティオペレーターロールのアクセス権の調整

セキュリティ管理者ロールに切り替える方法を確認したので、次はセキュリティオペレーターロールのアクセス権限を変更して、Macie、GuardDuty、Inspector、および CloudTrail へのアクセス権を読み取り専用にしていきます。

1. IAM コンソールで、**ロール** を選択し、*SecOperator* を検索します。表示されたロールをクリックします。ロールには、以下ようなアクセス権が与えられています。

    ![SecOperPolicy](./images/IamEssSecOperPolicy.png)

2. 繰り返しになりますが、5 つのマネージドポリシー (Inspector、GuardDuty、CloudTrail、IAM、SNS 用の 5つの AWS マネージドポリシーと、Macie 用のカスタムマネージドポリシー) があることを確認してください。Inspector、GuardDuty、CloudTrail のマネージドポリシーは、サービスへのフルアクセス権を提供していますが、Macie ポリシー (名前に *SecOperatorMaciePolicy* が含まれるもの) は、その名前にもかかわらず、Macie へのフルアクセス権を提供しています。

3.  赤い矢印で示す X 印をクリックして、AmazonCloudTrailFullAccess、AmazonGuardDutyFullAccess、および AmazonInspectorFullAccess ポリシーを削除します。Inspector、CloudTrail、GuardDuty への読み取り専用のアクセスポリシーを追加します。また、SecOperatorMacie ポリシーを変更して、Macie への読み取り専用のアクセス権を提供するようにします。

    手順は記載していません。ヒントが必要な場合は、以下のドロップダウンを開いてください。

    <details>
<summary><strong>ここをクリックしてヒントを表示</strong></summary><p>
    <br/>
    
    情報のリンクを記載します。新しいブラウザタブで以下のリンクを開いて確認してください。


    [Amazon CloudTrail のアクセス権限の管理](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/control-user-permissions-for-cloudtrail.html)
    
    [Amazon GuardDuty のアクセス権限の管理](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_managing_access.html)
    
    [Amazon Inspector のアクセス権限の管理](https://docs.aws.amazon.com/IAM/latest/UserGuide/list_amazoninspector.html)
    
    [Controlling access to Amazon Macie](https://docs.aws.amazon.com/macie/latest/userguide/macie-access-control.html)
    
    </details>

4. CloudFormation コンソールに移動し、以前構築した *esslab* という名前の CloudFormation スタックの出力タブを表示します。

5. SecOperatorRoleURL の隣にある URL をクリックします。新しいブラウザタブウィンドウが開き、以下のように表示されます。

    ![SecOperatorRole](./images/IamEssSwitchSecOperRole.png)

    ボックスには、アカウント ID (AWS アカウントの ID)、CloudFormation によって作成されたロール名、および表示名が含まれます。切り替えるロールの表示色も選択することもできます。

    **ロールの切り替え** をクリックします。

     下記のように、コンソールウィンドウの上部に *SecOperator* という名前の新しいロールラベルが表示されます。これは、 *SecOperator* ロールの権限が有効な権限として「一時的に」置き換えられたことを意味しています
    
    ![SecOperatorLabel](./images/IamEssSwitchSecOperRolePost.png)

    では同様に、セキュリティオペレーターとしてどのような権限があるか、各サービスに順番にアクセスして確認していきます。

6. **Amazon Inspector コンソール**に移動します。**評価テンプレート** をクリックし、*LampInspectorAssessmentTemplate* で始まるテンプレートの左にあるボックスを**両方**選択して **削除** をクリックします。削除の確認に **はい** をクリックします。数秒後に、inspector：DeleteAssessmentTemplateアクションを呼び出す権限がないことを示すエラーメッセージが表示されます。


7. **GuardDuty コンソール**に移動し、**設定** をクリックし、**更新された結果の頻度** フィールドを別の値に変更し、**保存** をクリックします。UpdateDetector アクションを実行する権限がないことを示すエラーメッセージが表示されます (スクロールして表示する必要がある場合があります) 。これは、GuardDuty への読み取り専用のアクセス権しかないためです。
8. (2020年5月25日現在：Macieの機能拡張により正しくアクセスできない場合があります。その場合は次へ進んでください) **Macie コンソール**に移動し、US East (N. Virginia) リージョンを選択して、**設定** をクリックし、**Content Type**アイコンをクリックします。そうすると、ファイルタイプのリストが表示されます。*application/cap* などのファイルタイプを選択し、選択したファイルタイプを編集して *Enabled*  フラグの値を変更し、**Save** をクリックします。Macie への読み取り専用のアクセス権しかないため、エラーメッセージが表示されます。
9. **CloudTrail コンソール**に移動します。
10. **証跡情報** をクリックし、名前が *esslab* で始まる証跡をクリックします。
11. 画面右上の **ログ記録** スイッチを OFF (オフ) に切り替え、**次へ** をクリックします。CloudTrail への読み取り専用のアクセス権しかないため、エラーメッセージが表示されます。
12. ここで、デフォルトのロールに戻ります。そうすると、SecOperator ロールラベルはコンソールに表示されなくなります。
12. ワークショップをチームの一員を実施している場合、時間があれば、他のチームにアカウント認証情報を渡して、次の[検証フェーズ](./verify.md) で他のチームの設定を確認してみてください。それ以外の場合は、[検証フェーズ](./verify.md) の**クリーンアップセクション**まで進んでください。

