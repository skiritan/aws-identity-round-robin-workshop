# アクセス委任 ラウンド (検証フェーズ)

| 重要な注意事項 |
|---|
| 以下の手順に進む前に、必ず**[シナリオセクション](./index.md)**と**[構築フェーズ](./build.md)** をお読みください。 |

## 検証の課題: 環境が正しく構築されたかのテスト

前の構築フェーズでは、*自分が*構築した環境をテストしました。このセクションの目標は、別のチームにより構築された環境のセキュリティを評価することです。

| 重要な注意事項 |
|---|
| 以下の確認には、**他のチームの資格情報（ログイン情報）**を使用してください。 |

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

??? info "AWS 主催のイベント"

    1. ウェブブラウザの別のタブで、指示された URL に移動し、ログインします。 
    
    2. ログイン後、AWS Account (AWS アカウント) ボックスをクリックし、その下に表示されている Account ID  (画像の赤いボックス) をクリックします。マネジメントコンソールの下にリンクが表示されるので、これをクリックして、AWS コンソールに移動します。  
    
    ![login-page](./images/login.png)

??? info "個人の AWS アカウントを使用している場合"

    ウェブブラウザの別のタブで、https://aws.amazon.com/console にアクセスし、アカウントにログインします。

## セキュリティオペレーターロールの検証

1. CloudFormation コンソールに移動し、*esslab* という名前の CloudFormation スタックの出力タブを表示します。

2. SecOperatorRoleURL の隣にある URL をクリックします。新しいブラウザタブウィンドウが開き、以下の図が表示されます。

    ![SecOperatorRole](./images/IamEssSwitchSecOperRole.png)

    ボックスには、アカウント ID (AWS アカウントの ID)、CloudFormation によって作成されたロール名、および表示名が含まれます。切り替えるロールの表示色も選択することもできます。

    **ロールの切り替え** をクリックします。

    下記のように、コンソールウィンドウの上部に *SecAdministrator* という名前の新しいロールラベルが表示されます。これは、 SecAdministrator ロールの権限が有効な権限として「一時的に」置き換えられたことを意味しています

    ![SecOperatorLabel](./images/IamEssSwitchSecOperRolePost.png)

    では、セキュリティオペレーターとしてどのような権限があるか、各サービスに順番にアクセスして確認していきます。


3. **Amazon Inspector コンソール**に移動します。**評価テンプレート** をクリックし、*LampInspectorAssessmentTemplate* で始まるテンプレートの左にあるボックスを**両方**選択して **削除** をクリックします。削除の確認に **はい** をクリックします。数秒後に、inspector：DeleteAssessmentTemplateアクションを呼び出す権限がないことを示すエラーメッセージが表示されます。 これは、Inspector への読み取り専用のアクセス権しかないためです
4. **GuardDuty コンソール**に移動し、**設定** をクリックし、**更新された結果の頻度** フィールドを別の値に変更し、**保存** をクリックします。UpdateDetector アクションを実行する権限がないことを示すエラーメッセージが表示されます (スクロールして表示する必要がある場合があります) 。これは、GuardDuty への読み取り専用のアクセス権しかないためです。
5.  (2020年5月25日現在：Macieの機能拡張により正しくアクセスできない場合があります。その場合は次へ進んでください) **Macie コンソール**に移動し、US East (N. Virginia) リージョンを選択して、**設定** をクリックし、**Content Type**アイコンをクリックします。そうすると、ファイルタイプのリストが表示されます。*application/cap* などのファイルタイプを選択し、選択したファイルタイプを編集して *Enabled*  フラグの値を変更し、**Save** をクリックします。Macie への読み取り専用のアクセス権しかないため、エラーメッセージが表示されます。

6. **CloudTrail コンソール**に移動します。
7. **証跡情報** をクリックし、名前が *esslab* で始まる証跡をクリックします。
8. 画面右上の **ログ記録** スイッチを OFF (オフ) に切り替え、**次へ** をクリックします。CloudTrail への読み取り専用のアクセス権しかないため、エラーメッセージが表示されます。

## 調査結果の共有

この環境を構築したチームと調査結果を共有します。相違点があればそのことについても話し合ってください。

## クリーンアップ

アカウントへの請求を防ぐために (他のアイデンティティラウンドを行っている場合は特に)、ここで作成したインフラストラクチャをクリーンアップすることをお勧めします。以下のセクションのいずれかの指示に従ってください。

??? info  "AWS 主催の Event Engine を利用したイベント " 

    次の手順に従って、コンポーネントを削除します。
    
    1. [Amazon Macie の無効化](https://docs.aws.amazon.com/macie/latest/userguide/macie-disable.html).  Macie を無効にするには、SecAdministrator ロールに戻るか、コンソールに再度サインインする必要があるかもしれません。

??? info "AWS 主催のイベント"

    クリーンアップは必要ありません。AWS が実施します。

??? info "個人の AWS アカウントを使用している場合"

    次の手順に従って、コンポーネントを削除します。
    
    1. [SecOperator ロールの削除](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_manage_delete.html).
    
    2. [CloudFormation スタック(esslab)の削除](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-delete-stack.html).  スタックが削除されるまで待機します。
    
    3. [Amazon Macie の無効化](https://docs.aws.amazon.com/ja_jp/macie/latest/user/macie-suspend-disable.html).
    
    4. [Amazon GuardDuty の無効化](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_suspend-disable.html).
    
    5. [logging バケットの削除](https://docs.aws.amazon.com/AmazonS3/latest/user-guide/delete-bucket.html).
    
    6. [Amazon Inspector テンプレートの削除](https://docs.aws.amazon.com/inspector/latest/userguide/inspector_assessments.html#delete_assessment_via_console).


おめでとうございます！  これでアクセス委任ラウンドが無事完了しました。
