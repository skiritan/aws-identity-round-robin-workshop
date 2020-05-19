# パーミッションバウンダリー ラウンド <small>検証フェーズ</small>

ここからは**検証**フェーズです。Web 管理者の立場でアクセスのテストを行います。 

他のチームから以下の情報を入手しているはずです :

* IAM ユーザのサインインリンク:	**https://Account_ID.signin.aws.amazon.com/console**
* IAM ユーザ名: **webadmin**
* IAM ユーザ パスワード:	
* リソース制限 ID :	
* パーミッションバウンダリー名: **webadminpermissionboundary**
* パーミッションバウンダリーのポリシー名: **webadminpermissionpolicy**

Web管理者がアクセスできるのは以下のリソースのみです :


1. Web 管理者が作成した IAM ポリシーとロール 
2. Web 管理者が作成した Lambda 関数

Web 管理者が、自分が所有していないリソース (IAM ユーザー、ロール、S3 バケット、Lambda 関数など) に影響を与えることがあってはいけません。  

IAM ポリシー、IAM ロール、および Lambda 関数を設定していきます。Lambda 関数は、S3 バケット内のファイルのリストを取得できるようにします。その結果を確認することで、Web 管理者が実行できる範囲がわかります。 

アプリケーションのアーキテクチャ: ![image1](./images/architecture.png)

## タスク１ <small>         カスタマー管理 IAM ポリシーを作成する  </small>

* まず、カスタマー管理 IAM ポリシーを作成します。これは、Lambda 関数に付与するロールのアクセス権限です。この関数は S3 とともに動作しますが、ここではアクセス権の境界がどのように動作するかを確認するため、以下の基本的なポリシー（ Lambda ロギングアクセス権と S3 フルアクセス権）を付与します。  
* リソース制限が設定されており、ポリシーに特定の名前を使用する必要があることに注意してください  

``` json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents",
                "s3:*"
            ],
            "Resource": "*"
        }
    ]
}
```

## タスク２ <small>IAM ロールを作成する</small>

* 次に、IAM ロールを作成します。このロールのサービスとして Lambda を選択します。作成したポリシーとパーミッションバウンダリー(標準名では **webadminpermissionboundary** ) を付与します。
* リソース制限が設定されており、ロールに特定の名前を使用する必要があることに注意してください  
	
## タスク３ <small>Lambda 関数を作成する</small>

* 最後に、以下のコードを使用して **Node.js 8.10** の Lambda 関数を作成し、作成した IAM ロールをその関数に追加します。`"WEB_ADMIN_BUCKET_NAME"` を、`"web-admins-"` から始まり`"-data"`で終わるバケット名に置き換える必要があります。

``` node
const AWS = require('aws-sdk');
const s3 = new AWS.S3();

exports.handler = async (event) => {
  console.log('Loading function');
  const allKeys = [];
  await getKeys({ Bucket: 'WEB_ADMIN_BUCKET_NAME' }, allKeys);
  return allKeys;
};

async function getKeys(params, keys){
  const response = await s3.listObjectsV2(params).promise();
  response.Contents.forEach(obj => keys.push(obj.Key));

  if (response.IsTruncated) {
    const newParams = Object.assign({}, params);
    newParams.ContinuationToken = response.NextContinuationToken;
    await getKeys(newParams, keys); 
  }
}
```

* Lambda 関数をテストし、CloudWatch logsにログが生成されていること、およびバケット内のオブジェクトの一覧が取得できることを確認します。（テストするには、テストイベントを作成する必要があります。テストのパラメータは重要ではありません）

## タスク４ <small>他にできることがあるかどうかを調査する</small>

最後のタスクは、アカウント内で他に何ができるかを調査することです、Web 管理者が所有していないリソースに影響を与える操作ができるか確認してください。



#

おめでとうございます！  これでパーミッションバウンダリー ラウンド が無事完了しました。