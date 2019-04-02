# Permission boundaries workshop <small>- Advanced edition</small>
# <small>VERIFY phase</small>

You are now in the **VERIFY** phase. It is time to put on the hat of the webadmins and test out their permissions. Permissions boundaries are about delegation so verifying (acting as the delegated admins) the work of another team that did the build phase (acting as the admins) will simulate that experience.

If doing this as part of an AWS event you should have received from another team the following information:

* Webadmins role ARN:	arn:aws:iam::`YOUR_ACCOUNT_ID`:role/**webadmins**
* Resource restriction for both the roles and policies: /webadmins/`Resource restriction you used`
* Permissions boundary name: **webadminspermissionsboundary**
* Permission policy name: **webadminspermissionpolicy**

As the webadmins you will be creating an IAM policy, an IAM role and a Lambda function. The Lambda function should be able to list files in a particular S3 bucket. The webadmins should not be able to escalate their permissions or impact the IAM policies and roles created by other teams.

??? info "Application architecture"
	
	![image1](./images/architecture.png)

## Task 1 <small>Create a policy</small>
	
#### Your goal is to create a Lambda function. You will need to create an IAM role, attach two policies (a trust policy and permission policy) to it and then pass the role to the Lambda function. First you create the permission policy which just needs to allow log file creation and s3:ListBucket. You are in a hurry though and give the role s3:*. The policy you create here will later be attached to the role you create in Task 2 which will then be passed to a Lambda function you will create in Task 3

### Walk Through

* Use the following JSON to create a file named verifypolicydoc (replace the Account ID): 
	* `{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Action":["logs:CreateLogGroup","logs:CreateLogStream","logs:PutLogEvents","s3:*"],"Resource":"*"}]}`
* Create the policy (**there is a key parameter missing from the command below**)
	* `aws iam create-policy --policy-name NAME_OF_POLICY --policy-document file://verifypolicy`
<!-- `aws iam create-policy --policy-name NAME_OF_POLICY --path /NAME_OF_PATH/ --policy-document file://verifypolicy` -->

## Task 2 <small>Create a role</small>

#### The role you create here will be passed to the Lambda function you create in task 3

### Walk Through

* Use the following JSON to create a file named verifytrustpolicy (replace the Account ID): 
	* `{ "Version": "2012-10-17", "Statement": { "Effect": "Allow", "Principal": { "Service": "lambda.amazonaws.com"}, "Action": "sts:AssumeRole" } }`
* Create the role (**there is a key parameter missing from the command below**)
	* `aws iam create-role --role-name NAME_OF_ROLE --path /NAME_OF_PATH/ --assume-role-policy-document file://verifytrustpolicy`
<!-- `aws iam create-role --role-name NAME_OF_ROLE --path /NAME_OF_PATH/ --assume-role-policy-document file://verifytrustpolicy --permissions-boundary arn:aws:iam::Account_ID:policy/webadminspermissionsboundary` -->
* Attach the policy you created in Task 1 to the role:
	* `aws iam attach-role-policy --policy-arn arn:aws:iam::Account_ID:policy/NAME_OF_PATH/NAME_OF_POLICY --role-name NAME_OF_ROLE`
		
## Task 3 <small>Create and test a Lambda function</small>

#### Finally you will create a **Node.js 8.10** Lambda function using the sample code below and pass the IAM role you just created

### Walk Through

* Create a file `index.js` using the code sample below and then zip the file. Replace `"SHARED_LOGGING_BUCKET_NAME"` with the name of bucket that begins with `"shared-logging-"` and ends in `"-data"`. In order to get the bucket name, just run `aws s3 ls` using the webadmins role.)

``` node
const AWS = require('aws-sdk');
const s3 = new AWS.S3();

exports.handler = async (event) => {
  console.log('Loading function');
  const allKeys = [];
  await getKeys({ Bucket: 'SHARED_LOGGING_BUCKET_NAME' , Prefix: 'webadmins'}, allKeys);
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
* Create a Lambda function
	* `aws lambda create-function --function-name NAME_OF_FUNCTION --runtime nodejs8.10 --role arn:aws:iam::Account_ID:role/NAME_OF_PATH/NAME_OF_ROLE --handler index.handler --region us-east-2 --zip-file fileb://NAME_OF_ZIP_FILE`
* Invoke the Lambda function and make sure it is generating logs in CloudWatch logs and that it is able to list the objects in the bucket.
	* `aws lambda invoke --function-name NAME_OF_FUNCTION --region us-east-2 --invocation-type RequestResponse outputfile.txt`
* Examine the output file. It should show a number of log files in the S3 bucket that the Lambda function read. 

If you see files marked that **webadmins/you-should-SEE-this-file--webadmins...** then you have successfully verified that the webadmins can perform their role. Congratulations!

Extra Credit: What else could you do here to verify that the webadmins are not able to escalate their permissions or impact other policies and roles in the account?
