# Permission boundaries workshop <small>- Advanced edition</small>
# <small>VERIFY phase</small>

You are now in the **VERIFY** phase. It is time to put on the hat of the webadmins and test out their permissions. Permissions boundaries are about delegation so handing this access off to another team will simulate that experience.

You should have received from another team the following information:

* IAM role ARN:	arn:aws:iam::`ACCOUNT_ID`:role/**webadmin**
* Resource restriction for both the roles and policies: /webadmins/`resource restriction`
* permissions boundary name: **webadminspermissionsboundary**
* Permission policy name: **webadminspermissionpolicy**

You will be creating up an IAM policy, an IAM role and a Lambda function. The Lambda function should be able to list files in a particular S3 bucket. The webadmins should not be able to impact any resources in the account that they do not own.


??? info "Application architecture"
	
	![image1](./images/architecture.png)

## Task 1 <small>Create a policy</small>
	
#### This policy will be attached to the role you create in Task 2 which will then be passed to a Lambda function you create in Task 3

### Walk Through

* Use the following JSON to create a file named verifypolicy (replace the Account ID): 
	* `{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Action":["logs:CreateLogGroup","logs:CreateLogStream","logs:PutLogEvents","s3:*"],"Resource":"*"}]}`
* Create the policy
	* `aws iam create-policy --policy-name NAME_OF_POLICY --path /NAME_OF_PATH/ --policy-document file://verifypolicy`

## Task 2 <small>Create a role</small>

#### This role will be passed to the Lambda function you create in the following task

### Walk Through

* Use the following JSON to create a file named trustpolicy (replace the Account ID): 
	* `{ "Version": "2012-10-17", "Statement": { "Effect": "Allow", "Principal": { "Service": "lambda.amazonaws.com"}, "Action": "sts:AssumeRole" } }`
* Create the role:
	* `aws iam create-role --role-name NAME_OF_ROLE --path NAME_OF_PATH --assume-role-policy-document trustpolicy --permissions-boundary arn:aws:iam::Account_ID:policy/webadminspermissionsboundary  file://verifyassumerole`
* Attach the policy you created in Task 1 to the role:
	* `aws iam attach-role-policy --policy-arn arn:aws:iam::Account_ID:policy/NAME_OF_PATH/NAME_OF_POLICY --role-name NAME_OF_ROLE`
		
## Task 3 <small>Create and test a Lambda function</small>

#### Finally you will create a **Node.js 8.10** Lambda function using the sample code below and pass the IAM role you just created

### Walk Through

* Create a file `index.js` using the code sample below and then zip the file (replace `"WEB_ADMIN_BUCKET_NAME"` with the name of bucket that begins with `"web-admins-"` and ends in `"-data"`. In order to get the bucket name, just run `aws s3 ls` using the webadmins role.)

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
* Create a Lambda function
	* `aws lambda create-function --function-name NAME_OF_FUNCTION --runtime nodejs8.10 --role arn:aws:iam::Account_ID:role/NAME_OF_PATH/NAME_OF_ROLE --handler index.handler --region us-east-2 --zip-file fileb://NAME_OF_ZIP_FILE`
* Invoke the Lambda function and make sure it is generating logs in CloudWatch logs and that it is able to list the objects in the bucket.
	* `aws lambda invoke --function-name NAME_OF_FUNCTION --region us-east-2 --invocation-type RequestResponse outputfile.txt`
* Examine the output file. It should show a number of log files in the S3 bucket that the Lambda function read. 
* Check for the logs
	* ?????????????