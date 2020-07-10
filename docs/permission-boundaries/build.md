# パーミッションバウンダリー ラウンド <small>         構築フェーズ  </small>

Web 管理者に権限を委任する一連のタスクを以下に示します。これらのタスクでは、ポリシーを作成とテストを行います。チームでこのラウンドを行う場合は、タスクを実行する人とテストを行う人に担当をを分けることもできます。 

## 設定手順

以下のドロップダウンのいずれかを展開し、指示に従ってください。 (このワークショップが **AWS** イベントか**個人**かによって選択してください)

??? info "AWS 主催のイベント"


	**コンソールログイン:** このワークショップに AWS の公式イベントで参加している場合、あなたのチームはアカウントの URL とログイン認証情報を所持している必要があります。AWS SSO を使ってアカウントにログインできるので、この URL を参照しログインします。 
	
	ログイン後、**AWS Account** ボックスをクリックし、その下に表示されている Account ID (アカウント ID) (画像の赤いボックス) をクリックします。**マネジメントコンソール** のリンクが表示されるので、これをクリックしてコンソールに移動します。 
	
	![login-page](./images/login.png)
	
	リージョンが Ohio (us-east-2) に設定されていることを確認します。
	
	**CloudFormation:** 以下の CloudFormation スタックを起動して環境を設定します。
	
	リージョン| デプロイ
	------|-----
	US East 2 (Ohio) | [![Deploy permissions boundary round in us-east-2](./images/deploy-to-aws.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-2#/stacks/new?stackName=Perm-Bound&templateURL=https://identity-round-robin.awssecworkshops.com/permission-boundaries/identity-workshop-web-admins.yaml)
	
	1. 上の　**Deploy to AWS** ボタンをクリックします。これにより、テンプレートを実行するコンソールに自動的に移動します。 
	2. **テンプレートの指定** セクションで **次へ** をクリックします。
	3. **スタックの詳細を指定** セクションで **次へ** をクリックします (スタック名は既に入力されています。入力されていない場合は任意の名前を入力します)。
	4. **スタックオプションの設定** セクションで **次へ** をクリックします。
	5. 最後に、テンプレートによって **IAM ロールが作成されることを承認** し、**スタックの作成** をクリックします。
	
	これによって CloudFormation コンソールに戻ります。ページを更新すると、スタックの作成が開始されることが確認できます。次に進む前に、スタックが **CREATE_COMPLETE** であることを確認してください。


??? info "個人の AWS アカウントを使用している場合"



	通常どおりアカウントにログインします。
	
	**CloudFormation:** 以下の CloudFormation スタックを起動して環境を設定します。
	
	リージョン| デプロイ
	------|-----
	US East 2 (Ohio) | [![Deploy permissions boundary round in us-east-2](./images/deploy-to-aws.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-2#/stacks/new?stackName=Perm-Bound&templateURL=https://identity-round-robin.awssecworkshops.com/permission-boundaries/identity-workshop-web-admins.yaml)
	
	1. 上の　**Deploy to AWS** ボタンをクリックします。これにより、テンプレートを実行するコンソールに自動的に移動します。 
	2. **テンプレートの指定** セクションで **次へ** をクリックします。
	3. **スタックの詳細を指定** セクションで **次へ** をクリックします (スタック名は既に入力されています。入力されていない場合は任意の名前を入力します)。
	4. **スタックオプションの設定** セクションで **次へ** をクリックします。
	5. 最後に、テンプレートによって **IAM ロールが作成されることを承認** し、**スタックの作成** をクリックします。
	
	これによって CloudFormation コンソールに戻ります。ページを更新すると、スタックの作成が開始されることが確認できます。次に進む前に、スタックが **CREATE_COMPLETE** であることを確認してください。

## タスク１ <small>マネージドポリシー、IAM ロール、および Lambda 機能を作成する権限を持つ IAM ユーザー、IAM ポリシーを作成する</small>

IAM ポリシーを構築して、Web 管理者がマネージドポリシー、IAM ロール、および Lambda 機能を作成できるようにします。また、Web 管理者自身が作成したポリシー、ロール、Lambda 機能も編集できるようにします。 

!!! Attention "注意"
    各タスクで提供されているヒントを確認する際には、アカウント ID の追加、リソース制限を正しく使用してください。また必要に応じて、指定されたリージョンを変更してください。この項目のいずれかが間違っていると、ポリシーに問題が発生する可能性があります

### ウォークスルー

* [IAM コンソール](https://console.aws.amazon.com/iam/home) に移動します。  
* IAM コンソールの最初の画面 (ダッシュボード画面) には、**IAM ユーザのサインインリンク**が表示されます。このリンクをコピーしてください。この URL にはアカウント ID が含まれており、このアカウントを他のチームに渡して検証フェーズを行う際に必要になります。  
![image1](./images/iam-dashboard.png)
* 左側のメニューで **ユーザー** をクリックし、**ユーザを追加**をクリックします。**`webadmin`** という名前の新しい IAM ユーザーを作成します。**AWS** **マネジメントコンソールへのアクセス** のチェックボックスをオンにしてから、パスワードを自動生成するか、カスタムパスワードを設定します。**パスワードのリセットが必要** のチェックボックスを**オフ**にします。次のアクセス許可の設定で、AWS マネージドポリシー **IAMReadOnlyAccess** と **AWSLambdaReadOnlyAccess** をユーザーに付与します。  
![image1](./images/create-iam-user.png)
* 次に、左側のメニューで **ポリシー** をクリックします。以下のヒントに基づいて、新しい IAM ポリシーを作成します。作成した **`webadmin`** ユーザーにこのポリシーも付与します。  

!!! hint "ヒント"
	[IAM ID](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_identifiers.html): IAM ポリシーでは、名前またはパス指定のリソース制限を使用します。以下の項目"Resource"にある「**????**」は、リソース制限として有効なものに置き換える必要があります。既存のリソース (ロール、Lambda 関数) を調べて、Web 管理者が所有する既存のリソースへのアクセス権がポリシーによって付与されることを確認します。「**????**」を置き換えることがこのラウンドの鍵となります。 

``` json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "CreateCustomerManagedPolicies",
            "Effect": "Allow",
            "Action": [
                "iam:CreatePolicy",
                "iam:DeletePolicy",
                "iam:CreatePolicyVersion",
                "iam:DeletePolicyVersion",
                "iam:SetDefaultPolicyVersion"
            ],
            "Resource": "arn:aws:iam::ACCOUNT_ID:policy/????"
        },
        {
            "Sid": "CreateRoles",
            "Effect": "Allow",
            "Action": [
                "iam:CreateRole",
                "iam:UpdateRole",
                "iam:DeleteRole",
                "iam:AttachRolePolicy",
                "iam:DetachRolePolicy"
            ],
            "Resource": [
                "arn:aws:iam::ACCOUNT_ID:role/????"
            ]
        },
        {
            "Sid": "LambdaFullAccesswithResourceRestrictions",
            "Effect": "Allow",
            "Action": "lambda:*",
            "Resource": "arn:aws:lambda:us-east-2:ACCOUNT_ID:function:????"
        },
        {
            "Sid": "PassRoletoLambda",
            "Effect": "Allow",
            "Action": "iam:PassRole",
            "Resource": "arn:aws:iam::ACCOUNT_ID:role/????",
            "Condition": {
                "StringLikeIfExists": {
                    "iam:PassedToService": "lambda.amazonaws.com"
                }
            }
        },
        {
            "Sid": "AdditionalPermissionsforLambda",
            "Effect": "Allow",
            "Action": ["kms:ListAliases", "logs:Describe*", "logs:ListTagsLogGroup", "logs:FilterLogEvents", "logs:GetLogEvents"],
            "Resource": "*"
        }
    ]
}
```

As you complete the following tests, keep in mind the resource restriction you set up in the policy above (**????**). Use the **IAM users sign-in link** you gathered earlier to login: 
以下の動作確認をする際には、上記のポリシーで設定したリソース制限に注意してください(**?????部分**)。         ログインするには、前にコピーしておいた **IAM** **ユーザーのサインインリンク**を使用します。  

* 別のブラウザを使用して　**webadmin** IAM ユーザーでログインし、ユーザーがポリシーを作成できることを確認します。 （この時点ではポリシーの内容は重要ではありません）
* リソース制限に従ってロールを作成できることを確認します。このロールは信頼できるエンティティとして Lambda を使用する必要があります (このロールを使用して次のタスクをテストします)。  
* 最後に、先ほど作成したロールを付与した Lambda 関数を作成できることを確認します。

!!! question "質問"

	* ここでリソース制限を使用するのはなぜですか？
	* リソース制限を行うには、命名とパス指定の 2 つの方法があります。AWS コンソールと CLI の両方を使用してポリシーを作成できるオプションはどちらですか？  

## タスク２ <small>パーミッションバウンダリーを作成する</small>

Web 管理者は、IAM ポリシー、IAM ロール、および Lambda 関数を作成できますが、作成できるロールのアクセス権限は制限する必要があります。もし制限しない場合、Web 管理者が完全な管理者権限を持つポリシーを新しく作成してロールに付与し、そのロールを Lambda 関数に渡して広範なアクセス権を付与することができてしまいます （故意か過失かにかかわらず）。ロールの有効なアクセス権に対して制限を設定するには、パーミッションバウンダリーを使用します。パーミッションバウンダリーを使って、Web 管理者が作成するロールに対して、以下の [有効なアクセス権](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html) のみ許可できるよう制限する必要があります。

>	i. ロググループの作成 (ただし、他のロググループを上書きすることはできません)

>	ii. ログストリームの作成とログの書き込み  

>	iii. 名前が `"web-admins-"` で始まり `"-data"`で終わる S3 バケットのオブジェクトのリスト取得

### ウォークスルー  : 

* Web 管理者向けのパーミッションバウンダリーとして使用する、新しい IAM ポリシーを作成します。  

!!! hint "ヒント"
	[IAM ID](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_identifiers.html): 以下の項目"Resource"にある「**????**」は、リソース制限として有効なものに置き換える必要があります。

``` json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "CreateLogGroup",
            "Effect": "Allow",
            "Action": "logs:CreateLogGroup",
            "Resource": "arn:aws:logs:us-east-2:ACCOUNT_ID:*"
        },
        {
            "Sid": "CreateLogStreamandEvents",
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "arn:aws:logs:us-east-2:ACCOUNT_ID:log-group:/aws/lambda/????:*"
        },
        {
            "Sid": "AllowedS3GetObject",
            "Effect": "Allow",
            "Action": [
                "s3:List*"
            ],
            "Resource": "arn:aws:s3:::web-admins-ACCOUNT_ID-*"
        }
    ]
}
```

ポリシーの名前は **`webadminpermissionboundary`** とします。

!!! question "質問"

	* このパーミッションバウンダリーを適用する対象は何ですか？  
	* パーミッションバウンダリーと標準の IAM ポリシーの違いは何でしょうか？
	* 現時点で、このパーミッションバウンダリーはどのようにしてテストできますか？  

## タスク３ <small>Web 管理者向けのアクセス権のポリシーを更新して、パーミッションバウンダリーを組み込む</small>

### ウォークスルー

先ほど作成したパーミッションバウンダリーを参照するポリシーを作成します。新しいポリシーの作成には、以下の例をそのまま使用することをお勧めします。

以下のポリシーには、これまで扱わなかった２つのセクションが追加されていることに注意してください (最後の 2 つのセクション)。このセクションは、アクセス権のポリシーやパーミッションバウンダリー自体の変更、削除を拒否することを目的としています。

!!! hint "ヒント"
	[パーミッションバウンダリー](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html): 以下の項目"Resource"にある「**????**」は、リソース制限として有効なものに置き換える必要があります。

``` json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "CreateCustomerManagedPolicies",
            "Effect": "Allow",
            "Action": [
                "iam:CreatePolicy",
                "iam:DeletePolicy",
                "iam:CreatePolicyVersion",
                "iam:DeletePolicyVersion",
                "iam:SetDefaultPolicyVersion"
            ],
            "Resource": "arn:aws:iam::ACCOUNT_ID:policy/????"
        },
        {
        	  "Sid": "RoleandPolicyActionswithnoPermissionBoundarySupport",
            "Effect": "Allow",
            "Action": [
            		"iam:UpdateRole",
                	"iam:DeleteRole"
            ],
            "Resource": [
                "arn:aws:iam::ACCOUNT_ID:role/????"
            ]
        },
        {
            "Sid": "CreateRoles",
            "Effect": "Allow",
            "Action": [
                "iam:CreateRole",
                "iam:AttachRolePolicy",
                "iam:DetachRolePolicy"
            ],
            "Resource": [
                "arn:aws:iam::ACCOUNT_ID:role/????"
            ],
            "Condition": {"StringEquals": 
                {"iam:PermissionsBoundary": "arn:aws:iam::ACCOUNT_ID:policy/webadminpermissionboundary"}
            }
        },
        {
            "Sid": "LambdaFullAccesswithResourceRestrictions",
            "Effect": "Allow",
            "Action": "lambda:*",
            "Resource": "arn:aws:lambda:us-east-2:ACCOUNT_ID:function:????"
        },
        {
            "Sid": "PassRoletoLambda",
            "Effect": "Allow",
            "Action": "iam:PassRole",
            "Resource": "arn:aws:iam::ACCOUNT_ID:role/????",
            "Condition": {
                "StringLikeIfExists": {
                    "iam:PassedToService": "lambda.amazonaws.com"
                }
            }
        },
        {
            "Sid": "AdditionalPermissionsforLambda",
            "Effect": "Allow",
            "Action": 	["kms:ListAliases", "logs:Describe*", "logs:ListTagsLogGroup", "logs:FilterLogEvents", "logs:GetLogEvents"],
            "Resource": "*"
        },
        {
            "Sid": "DenyPermissionBoundaryandPolicyDeleteModify",
            "Effect": "Deny",
            "Action": [
                "iam:CreatePolicyVersion",
                "iam:DeletePolicy",
                "iam:DeletePolicyVersion",
                "iam:SetDefaultPolicyVersion"
            ],
            "Resource": [
                "arn:aws:iam::ACCOUNT_ID:policy/webadminpermissionboundary",
                "arn:aws:iam::ACCOUNT_ID:policy/webadminpermissionpolicy"
            ]
        },
        {
            "Sid": "DenyRolePermissionBoundaryDelete",
            "Effect": "Deny",
            "Action": "iam:DeleteRolePermissionsBoundary",
            "Resource": "*"
        }
    ]
}
```

* 新しいポリシーに  **`webadminpermissionpolicy`** という名前を付け、webadmin ユーザーに関連付けます。以前に追加したテスト用のポリシーは削除します 。
* 作業が完了すると、**webadmin** ユーザーには、webadminpermissionpolicy、IAMReadOnlyAccess、および AWSLambdaReadOnlyAccess の 3 つのポリシーのみが適用されている状態になります。
* **webadmin** としてコンソールにログインしたブラウザから再度、ユーザーがポリシーの作成できることを確認し、ロールの作成 (ロールにアクセス権のポリシーとパーミッションバウンダリーの両方を割り当てる)、およびそのロールを渡す Lambda 関数を作成できることを確認します。リソース制限に注意してください。

!!! question "質問"

	* DeletePolicy アクションの拒否を追加したのはなぜですか？
	* このパーミッションバウンダリー設定を削除する機能を拒否しなかった場合どうなりますか？


## タスク４ <small>     **検証**フェーズに必要な情報を収集する</small>

### ウォークスルー

Web 管理者用の IAM ユーザーが設定できたので、この情報を **検証タスク** を実施する別のチームに渡します。設定に関する情報をまとめて、その情報を次のチームに渡してください。

他のチームに渡す必要のある情報は以下のとおりです。 推奨される名前で設定した場合、下記のフォームがそのまま利用できます。また、あなたのチームも他のチームからフォームを入手し、あなたも**検証フェーズ**に取り組むことができるようにしてください。

* IAM ユーザのサインインリンク:	**https://Account_ID.signin.aws.amazon.com/console**
* IAM ユーザ名: **webadmin**
* IAM ユーザ パスワード:	
* リソース制限 ID :	
* パーミッションバウンダリー名: **webadminpermissionboundary**
* パーミッションバウンダリーのポリシー名: **webadminpermissionpolicy**

!!! hint "ヒント" 
	情報を提供してくれたチームに、あなたのチームの情報を配布しないようすると、チーム数が奇数の場合でも、配布先がないチームを出さないようにすることができます。

以上のタスクを完了したら、[**検証フェーズ**](./verify.md)に進んでください。

