# aws-lambda-manager

## Install lambda globally
```js
npm install -g https://github.com/asknivi/aws-lambda-manager.git
```

## Overview

This function creates a new lambda function in a specific region. 
Execution To use this run the following:
```js
lambda create <deployment spec>
```
 This function takes the following command line switches to control the deployment further:
	-n = skips upload of the zip file in the package spec to s3. This can be used in the event that there are multiple lambda functions to deploy from the zip package and the latest version of the zip file is already uploaded to s3. The simple use case for using this flag is when you are deploying multiple lambda functions from the same package one after another.
	
-p <profile> = The AWS cli profile to use when uploading code. This flag is necessary when you have multiple cli profiles setup.
-r <region> = The region in which to create the lambda function.
-b <bucket> = Bucket storage for lambda fucntion.
-s <stage> = If you would like you can also assign a lambda alias to your function that allows you to have different environments that control the behavior of the lambda function. Typically we have staging and production environments for all of our lambda functions and use a combination of redis and JSON object files to obtain environment variables that control the behavior of the lambda function.
##### Results
When the lambda function is created two things happen:
A FunctionArn key is created in the lambdaconfig object in the deployment spec. This ARN is used by the update and set-stage commands to update the proper lambda function. The updated deployment spec should be checked into source control.
NOTE: as a consequence of this we can only practically deploy a lambda function to a single region. If there is a need to deploy a lambda function to multiple regions the manner in which this command and others needs to change.
A history object is created and is saved to the same directory as the deployment spec. This history object should be checked into source control as it gives us a full traceable history of lambda deployments. See below for details on the history object.

#### lambda update 

This function updates an existing lambda function using the FunctionArn value that is stored in the lambdaconfig object. 
Execution
To use this run the following:
```js
lambda update <deployment spec>
e.g lambda update deploy/app-chatbot.json -p nivi -r ap-south-1 -b nivi-lambdatemp-ap-south-1
```
This function takes the following command line switches to control the deployment further:
	-n = skips upload of the zip file in the package spec to s3. This can be used in the event that there are multiple lambda functions to deploy from the zip package and the latest version of the zip file is already uploaded to s3. The simple use case for using this flag is when you are deploying multiple lambda functions from the same package one after another.
-p <profile> = The AWS cli profile to use when uploading code. This flag is necessary when you have multiple cli profiles setup.
-r <region> = The region to use as the context for the AWS command line tools.
Results
Upon successful completion of the deployment, the history object is updated. This history object should be checked into source control.

### lambda set-stage
#### Overview
This function creates or updates a lambda alias. It is possible to create an infinite number of aliases, but generally we will want to create at least two:
staging - This represents the stage before production deployment and is used to configure the lambda environment for testing. 
production - This represents the production deployment of a lambda function.  
In both cases, the environment configuration for a lambda function is stored in both redis and also in JSON objects stored with the code. 
Execution
To use this run the following:
```js
lambda set-stage <deployment spec> <stage name> ‘<version number | “$LATEST”>
```	
<stage name> - The stage name to assign to a specific version of the lambda function.
<version number | “$LATEST”> - The version number of the lambda function to point the alias at. This is a version number contained in the history object that is stored with the deployment spec. We can also use a special version number called $LATEST, which represents the latest deployed version of lambda code and is always present. Typically we always point the staging alias to the $LATEST version of the lambda function deployed so that whenever a new one is deployed staging always points to that. If there is a need to support more unstable code then this might change. 
This function takes the following command line switches to control the deployment further:
	-p <profile> = The AWS cli profile to use when uploading code. This flag is necessary when you have multiple cli profiles setup.
-r <region> = The region to use as the context for the AWS command line tools.
