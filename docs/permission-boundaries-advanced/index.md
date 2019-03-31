# Permissions boundary workshop -<small> Advanced edition </small>
# <small> Overview </small>

Your company has deployed a production three tier web application on AWS. Various teams work on different aspects of the architecture including the web admins who need to be able to create IAM roles for Lambda functions. In order to let them quickly iterate it was decided that permissions boundaries would be used to delegate permissions to the web admins so they can create the roles needed without being able to escalate their permissions or impact the resources of other teams. 

**AWS Service/Feature Coverage**: 

* AWS IAM [users & roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id.html)
* AWS IAM [permission boundaries](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html) 
* AWS IAM [identifiers or resource restrictions](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_identifiers.html)
* AWS [Lambda](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)
 
## Agenda

The round consists of a [**BUILD**](./build.md) phase followed by a [**VERIFY**](./verify.md) phase. 

### Phases 
**BUILD** (60 min): First each team will act as the admins of the account and carry out the activities involved in the **BUILD** phase where they will set up access for the web admins. 

Then each team will hand credentials for the webadmins IAM role in their account to another team to carry out the **VERIFY** phase. 

**VERIFY** (30 min): In the **Verify** phase each team will then act as the web admins validate that the requirements were set up correctly in the **BUILD** phase.

!!! info "Team or Individual Exercise"
	This workshop can be done as a team exercise or individually. If done as part of an AWS sponsored event then you will most likely be split into teams of around 2-3 people (you are of course free to work on your own.) If you are working in a team then try to divide up the tasks. 

### Presentation

If you are doing this workshop as part of an AWS event then there will usually be a 30 minute presentation before the hands-on exercise. Here is the [presentation deck](./presentation.pdf).

## Requirements

??? info "Click here for the account architecture"

	Account architecture: ![architecture](./images/architecture.png)

The goal of this workshop is to set up the web admins so they can create an IAM role and attach it to a Lambda function which will then read an S3 bucket. The webadmin need the necessary permissions to do this without also being able to escalate their permissions or impact the resources of other teams in the same AWS account. The web admins should only have access to the following resources:

1. IAM policies and roles created by the web admins 
2. S3 bucket: The roles the web admins create should only be allowed to list the log files in the webadmins folder of the bucket that starts with `"shared-logging-"` and ends with `"-data"`

#### [Click here to go to the BUILD phase](./build.md)