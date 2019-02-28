# Serverless Round <small>Build Phase</small>

Since you are championing the security tasks for your team, you pick up the two tasks for the WildRydes application.  Please read through and complete the following tasks.  Good Luck!

## Task 1 <small>Reduce the attack surface of the origin</small>

Ensure the application serves content out through the CloudFront Distribution and that your end users can **only** access the application through CloudFront URLs and not Amazon S3 URLs. As part of this configuration your end users should **not** be able to affect the availability or integrity of the application. 

!!! info "Key Security Benefits"

    * Obfuscates the S3 origin
    * Forces traffic over HTTPS with custom certificates
    * Adds DDoS protection to your application and enables the future use of <a href="https://aws.amazon.com/waf/" target="_blank">AWS WAF</a> and <a href="https://aws.amazon.com/shield/" target="_blank">AWS Shield</a>

### View the Existing Policy

First, view the existing S3 bucket policy to see what permissions the previous engineers created.

1. Go to the <a href="https://s3.console.aws.amazon.com/s3/home" target="_blank">Amazon S3</a> console
2. Click on the **identity-wksp-serverless-<*ACCOUNT#*\>-us-east-1-wildrydes** bucket.
3. Click on the **Permissions** tab and then click on **Bucket Policy**.

!!! question "What's wrong with this policy? What does `"Principal": "*"` mean? "
    
    Both `"Principal": "*"` and `"Principal":{"AWS":"*"}` grant permission to everyone (also referred to as anonymous access).  Use caution when granting anonymous access to your S3 bucket. When you grant anonymous access, anyone in the world can access your bucket. We highly recommend that you never grant any kind of anonymous write access to your S3 bucket.

### Modify Principal

Since the current bucket allows for anonymous access, you need to change this to only allow access from the CloudFront Distribution.

1. Go to the <a href="https://console.aws.amazon.com/cloudfront/" target="_blank">Amazon CloudFront</a> console.  You should see a Web Distribution for the WildRydes web application.
2. Click on **Origin Access Identities** in the left navigation. You should see an identity named **Unicorn OAI**. 

    !!! info "CloudFront Origin Access Identity"
        An Origin Access Identity (OAI) is a special CloudFront identity that you can associate with a Distribution in order restrict access using AWS IAM.  You can also find the OAI by viewing your Web Distribution

3. Copy down the ID.
4. Go back to the <a href="https://s3.console.aws.amazon.com/s3/home" target="_blank">Amazon S3</a> console and open up the bucket policy.
5. Replace the **principal** with the following and click **save**:

``` json
"Principal": {
	"AWS": "arn:aws:iam::cloudfront:user/CloudFront Origin Access Identity <OAI ID>"
},
```

!!! info

    You could also use the canonical user id as the principal: `"CanonicalUser": "<OAI S3CanonicalUserId>"`

### Modify Actions

Now that the principal is restricted to the identity associated with the CloudFront distribution you can take a closer look at the permissions.

1. Go back to the <a href="https://s3.console.aws.amazon.com/s3/home" target="_blank">Amazon S3</a> console and open up the bucket policy.

    !!! question "Does CloudFront really need access to Delete Objects?"
	
2. The distribution is acting as a CDN for the static site so it only needs read access to the S3 bucket.  Change the actions to ensure an end user can not affect the integrity of the site.

### Test the new bucket policy 

Now that the bucket policy has been updated, go validate that you can not access the website using an S3 URL.

1. Open the <a href="https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks?filter=active" target="_blank">Amazon CloudFormation</a> console (us-east-1)
2. Click on the **Identity-RR-Wksp-Serverless-Round** stack.
3. Click on **Outputs** and click on **WebsiteS3URL**.

!!! question "Are you still able to access the site using the S3 URL?"

### Solve the Mystery

So you've modified the bucket policy to restrict access to read only actions from the CloudFront Distribution but for some reason you are still able to access the site using S3 URLs.  Do some investigation into why this is the case and put in the additional control necessary to restrict the traffic.

!!! tip
    What other access controls exist within S3? Look into the following resources:

    * <a href="https://docs.aws.amazon.com/AmazonS3/latest/dev/access-control-block-public-access.html" target="_blank">S3 Block Public Access</a> (easiest)
    * <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_notprincipal.html" target="_blank">AWS IAM Policy Elements: NotPrincipal</a> (hardest)

    Be sure to clear your cache when testing!


## Task 2 <small>Set up application user management</small>

Set up user management for the application using Cognito User pools.  To reduce the operational overhead of creating and maintaining forms and custom logic for authentication, the decision has been made to use the Cognito hosted-UI to integrate the application with the User Pool.

As part of the user experience users should be able to sign themselves up, they should have to validate their email address, and be required to create a password that meets the *password complexity requirements for applications* set in your security standards.

!!! info "Password Complexity Requirements for Applications"
    * Minimum length of 10 characters
    * Must include symbols
    * Must include numbers
    * Must include uppercase characters
    * Must include lowercase characters

!!! info "Application Integration Requirements"
    * Implicit grant OAuth flow
    * Scopes: email openid
    * Upon successful authentication the user should be redirected to ride.html

### Configure User Pool

1. Go to the <a href="https://console.aws.amazon.com/cognito/home?region=us-east-1" target="_blank">Amazon Cognito</a> console (us-east-1)
2. Click on **Manage User Pools** and then click on the **WildRydes** pool.
3. Click on **Policies** in the left navigation and modify the password policy, enable users to sign themselves up, and save your changes.
4. Click on **MFA and Verifications** in the left navigation, enable email verification, and save your changes.

### Configure App Integration

1. Click on **App Client Settings** in the left navigation and enter the following and click **Save changes**:
	* Enabled Identity Providers: **Cognito User Pool**
	* Callback URL: **<WebsiteCloudFrontURL\>/ride.html**
	* Sign Out URL: **<WebsiteCloudFrontURL\>/index.html**
	* Allowed OAuth Flows: **Implicit Grant**
	* Allowed OAuth Scope: **email** **openid**
	
2. Click on **Domain Name** in the left navigation, enter a unique domain name, and save your changes.

### Construct the Hosted-UI URL

Now that your User Pool and App Integration have been configured you can construct the URL to allow users to sign-in via the Cognito Hosted Wed UI (built-in webpages for signing up and signing in your users).

```
<your_domain>/login?response_type=<code or token>&client_id=<your_app_client_id>&redirect_uri=<your_callback_url>

```

!!! tip
    Replace the values in <> (including the carrots) to the correct values.  All can be found in your Cognito configuration.  The response type is based on the OAuth flow.

    <a href="https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools-configuring-app-integration.html" target="_blank">Reference Documentation</a>


1.  Go to the [S3 console](https://s3.console.aws.amazon.com/s3/home) and click on the bucket named:
    
     **identity-wksp-serverless-<ACCOUNT#\>-us-east-1-wildrydes**.

2. Open **index.html** and add the hosted UI URL to the **Giddy Up** button.
3. Upload **index.html** back to the S3 bucket.

***

After you have completed these tasks you can move on to the Verify Phase.

!!! warning
    If you are doing this as part of an AWS sponsored event STOP here and wait for further instructions on the hand off to the next team.
