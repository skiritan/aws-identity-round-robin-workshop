# パーミッションバウンダリー ラウンド 

あなたの会社は、AWS で３層の Web アプリケーションを本番稼働しています。多数のチームがこのアプリケーションの色々な部分で作業していますが、必ずしもチーム間のコミュニケーションがうまく取れているわけではありません。つい最近、ウェブフロントエンドを担当していたチームが Lambda 関数を誤って設定してしまい、アプリケーションチームのリソースに影響を与えてしまいました。あなたは副社長の依頼を受けて、Web 管理者が必要なロールを作成できるように、パーミッションバウンダリーを使用してアクセス権限を委譲します。またその一方で、権限を不正に昇格させたり、他のチームのリソースに影響を与えたりすることないがよう、安全で継続的に利用できる環境を構築していきます。

**AWS のサービス/機能の範囲**: 

* AWS IAM [のパーミッションバウンダリー](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html) 
* AWS IAM [ID またはリソース制限](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_identifiers.html)
* AWS IAM [ユーザーとロール](https://docs.aws.amazon.com/IAM/latest/UserGuide/id.html)
* AWS [Lambda](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)

## アジェンダ

このラウンドは <a href="./build/" target="_blank">**構築フェーズ**</a> と <a href="./verify/" target="_blank">**検証フェーズ**</a> に分かれています。 

**構築** (60 分): まず、アカウントの管理者として Web 管理者のアクセス権限を設定します。その後、この IAM ユーザーの認証情報を別のチームに渡し、そのチームが**検証**フェーズを行います。

**検証** (30 分): 各チームは、Web 管理チームのメンバーになったつもりで**構築フェーズ**で設定が正しく行われたかの検証と、Web 管理者が許可されていないアクションが実行できないかの調査を行います。

!!! info "チーム演習または個人演習"
	このワークショップは、チームで行うことも、個人で行うこともできます。AWS イベントの場合は、2～3人程度のチームに分かれて行うこともあります。この場合は作業を分担して行ってください。 

このワークショップにおけるパーミッションバウンダリーの３つの要素を以下に示します。あなたがこのセクションの**構築フェーズ**を実行するとき、あなたはアカウントの管理者として行動します。次のセクションの**検証フェーズ**では、あなたは委任された管理者 (Web 管理者)として行動します。委任された管理者は、アクセス権の境界が設定されているため、**「制限されている」**と見なすことができる範囲でロールを作成できます。 


![mechanism](./images/permission-boundaries.png)

<!--### Point system
There is a point system for both the **BUILD** and **VERIFY**  activities. Each team also starts out with a number of points they can exchange for hints for various sections. 

Points earned during **VERIFY** Phase:

* 5 points for each requirement fulfilled by the team in the **BUILD** phase 
* 15 points for every major exploit found (components of an individual exploit cannot be combined, e.g. a public bucket that allows Read, Write and List is one exploit. 
-->

## 要件

??? info "アカウントの構成についてはここをクリックしてください"

	Account architecture: ![architecture](./images/architecture.png)

このワークショップのゴールは、Web管理者を設定し、この Web 管理者が S3 バケットを読み込むための Lambda 関数用の IAM ロールを作成できるようにすることです。Web 管理者は、権限を不正に昇格させたり、同じ AWS アカウントの他のチームのリソースに影響を与えたりすることないよう、適切な制限を与える必要があります。Web管理者がアクセスできるのは以下のリソースのみです。

1. Web 管理者が作成した IAM ポリシーとロール 
2. Web 管理者が作成した Lambda 関数
3. S3 バケット : Web 管理者は、``web-admins-``で始まり、``-data``で終わるバケットのみアクセスできます。

## プレゼンテーション

このワークショップを AWS イベントとして行っている場合、通常、演習の前に 30 分間のプレゼンテーションが行われます。プレゼンテーション資料は、 [こちらのリンク](./presentation_JP.pdf)から参照可能です。



Next (次へ) をクリックし、**構築フェーズ**に進みます。
