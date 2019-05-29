# Serverless Identity Round <small>Task 1</small>

Since you are championing the security tasks for your team, you pick up the two tasks for the WildRydes application.  Please read through and complete the following tasks.  Good Luck!

## Build Phase <small>Reduce the attack surface of the S3 origin</small>

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
2. Click on the **Identity-RR-Wksp-Serverless-Round** stack or the **module-a7932bd25ca64049a57fd5bb055782db** stack (this is the stack name when created using Event Engine).
3. Click on **Outputs** and click on **WebsiteS3URL**.

!!! question "Are you still able to access the site using the S3 URL?"

### Solve the Mystery

So you've modified the bucket policy to restrict access to read only actions from the CloudFront Distribution but for some reason you are still able to access the site using S3 URLs.  Do some investigation into why this is the case and put in the additional control necessary to restrict the traffic.

!!! tip
    What other access controls exist within S3? Look into the following resources:

    * <a href="https://docs.aws.amazon.com/AmazonS3/latest/dev/access-control-block-public-access.html" target="_blank">S3 Block Public Access</a> (easiest)
    * <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_notprincipal.html" target="_blank">AWS IAM Policy Elements: NotPrincipal</a> (hardest)

    Be sure to clear your cache when testing!


## Verify Phase <small>Reduce the attack surface of the S3 origin</small>

Now that the additional identity control has been added to the application, you have been tasked with acting as an end user and manually testing to verify that the control has been put in place correctly and that the requirements have been met.

*Ensure the application serves content out through the CloudFront Distribution and that your end users can only access the application through CloudFront URLs and not Amazon S3 URLs. As part of this configuration your end users should not be able to affect the availability or integrity of the application.*

!!! tip "Verification Checklist"

    * You can access the site through the CloudFront Distribution URL (<WebsiteCloudFrontURL\>).

    * You are restricted from accessing any of the application resources through S3 URLs (<WebsiteS3URL\>).

	    > Try some deep links (e.g. <WebsiteS3URL\>/js/vendor/unicorn-icon)

    * You can not delete or modify any of the application resources through the CloudFront Distribution.

	    > Try using something like curl or Postman to make requests with different HTTP verbs (e.g. Delete).  Below is an example using curl:  

        ```
        curl -i -X DELETE <WebsiteCloudFrontURL>/index.html
        ```
***

After you have completed task you can move on to task 2.