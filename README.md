# AWS CDK Immersion Day Workshop

## Overview

We will follow [AWS CDK Immersion Day Workshop](https://catalog.us-east-1.prod.workshops.aws/workshops/10141411-0192-4021-afa8-2436f3c66bd8/\
en-US), and create two stacks.

1. SQS and SNS.
2. Lambda.
3. API Gateway and Lambda.

-------------------------------------------------
## Stack 1

See the previous repos for 
* (starting mock cdk environment)[https://github.com/megnergit/AWS_CDK_localstack_C1]
* (editing and running resources)[https://github.com/megnergit/AWS_CDK_Lambda_C2]

```./lib.aws-stack.ts
# cat ./lib/aws-stack.ts
// import * as cdk from 'aws-cdk-lib';
import { Duration, Stack, StackProps } from 'aws-cdk-lib';
import * as sns from 'aws-cdk-lib/aws-sns';
import * as subs from 'aws-cdk-lib/aws-sns-subscriptions';
import * as sqs from 'aws-cdk-lib/aws-sqs';
import { Construct } from 'constructs';

export class AwsStack extends Stack {
  constructor(scope: Construct, id: string, props?: StackProps) {
    super(scope, id, props);

    const queue = new sqs.Queue(this, 'AwsQueue', {
      visibilityTimeout: Duration.seconds(300)
    });

    const topic = new sns.Topic(this, 'AwsTopic');

    topic.addSubscription(new subs.SqsSubscription(queue));

    // The code that defines your stack goes here

    // example resource
    // const queue = new sqs.Queue(this, 'AwsQueue', {
    //   visibilityTimeout: cdk.Duration.seconds(300)
    // });
  }
}
```
### Synth 
Check what resources are to be created.

```
c synth 
```

This output yaml document. 

```
# c synth | grep Type
    Type: AWS::SQS::Queue
    Type: AWS::SQS::QueuePolicy
    Type: AWS::SNS::Subscription
    Type: AWS::SNS::Topic
    Type: AWS::CDK::Metadata
    Type: AWS::SSM::Parameter::Value<String>
```
All right. 

### Deploy

```
c deploy
```

Check resources created.  
```
AWS_DEFAULT_OUTPUT=json a cloudformation \
list-stack-resources \
--stack-name AwsStack | grep Type

            "ResourceType": "AWS::SQS::Queue",
            "ResourceType": "AWS::SNS::Topic",
            "ResourceType": "AWS::CDK::Metadata",
            "ResourceType": "AWS::SQS::QueuePolicy",
            "ResourceType": "AWS::SNS::Subscription",

```
All right. 


### Destroy

Remove all resources from ```./lib/aws-stack.ts```. 

```
// import * as cdk from 'aws-cdk-lib';
import { Duration, Stack, StackProps } from 'aws-cdk-lib';
import * as sns from 'aws-cdk-lib/aws-sns';
import * as subs from 'aws-cdk-lib/aws-sns-subscriptions';
import * as sqs from 'aws-cdk-lib/aws-sqs';
import { Construct } from 'constructs';

export class AwsStack extends Stack {
  constructor(scope: Construct, id: string, props?: StackProps) {
    super(scope, id, props);

    /*

    const queue = new sqs.Queue(this, 'AwsQueue', {
      visibilityTimeout: Duration.seconds(300)
    });

    const topic = new sns.Topic(this, 'AwsTopic');

    topic.addSubscription(new subs.SqsSubscription(queue));
    */

    // The code that defines your stack goes here

    // example resource
    // const queue = new sqs.Queue(this, 'AwsQueue', {
    //   visibilityTimeout: cdk.Duration.seconds(300)
    // });
  }
}
```

Check what will happen when we execute the code.

```
# c diff
Stack AwsStack
Hold on while we create a read-only change set to get a diff with accurate replacement information (use --no-change-set to use a less accurate but faster template-only diff)
Could not create a change set, will base the diff on template differences (run again with -v to see the reason)
IAM Statement Changes
┌───┬────────────────────┬────────┬────────────────────┬────────────────────┬────────────────────┐
│   │ Resource           │ Effect │ Action             │ Principal          │ Condition          │
├───┼────────────────────┼────────┼────────────────────┼────────────────────┼────────────────────┤
│ - │ ${AwsQueue.Arn}    │ Allow  │ sqs:SendMessage    │ Service:sns.amazon │ "ArnEquals": {     │
│   │                    │        │                    │ aws.com            │   "aws:SourceArn": │
│   │                    │        │                    │                    │  "${AwsTopic}"     │
│   │                    │        │                    │                    │ }                  │
└───┴────────────────────┴────────┴────────────────────┴────────────────────┴────────────────────┘
(NOTE: There may be security-related changes not in this list. See https://github.com/aws/aws-cdk/issues/1299)

Resources
[-] AWS::SQS::Queue AwsQueue AwsQueue1C7A82F4 destroy
[-] AWS::SQS::QueuePolicy AwsQueue/Policy AwsQueuePolicy72651C10 destroy
[-] AWS::SNS::Subscription AwsQueue/AwsStackAwsTopicE081838E AwsQueueAwsStackAwsTopicE081838E02F43A52 destroy
[-] AWS::SNS::Topic AwsTopic AwsTopic8402D4FD destroy


✨  Number of stacks with differences: 1
```
All right.

Then deploy the code. This will remove the resources. 
```
c deploy
```

You will see the resource status ```DELETE_COMPLETE```.


```
AWS_DEFAULT_OUTPUT=json a cloudformation \
list-stack-resources \
--stack-name AwsStack | more

{
    "StackResourceSummaries": [
        {
            "LogicalResourceId": "AwsQueue1C7A82F4",
            "PhysicalResourceId": "http://sqs.eu-central-1.localhost.localstack.cloud:4566/00000000
0000/AwsStack-AwsQueue1C7A82F4-8c43a2c8",
            "ResourceType": "AWS::SQS::Queue",
            "LastUpdatedTimestamp": "2024-08-07T12:58:57.285000Z",
            "ResourceStatus": "DELETE_COMPLETE",  <======== HERE
            "DriftInformation": {
                "StackResourceDriftStatus": "NOT_CHECKED"
            }
        },
....
```

-------------------------------------------------
## Stack 2

Now we will start [API Gateway and Lambda function part](
https://catalog.us-east-1.prod.workshops.aws/workshops/10141411-0192-4021-afa8-2436f3c66bd8/en-US/2000-typescript-workshop/300-hello-cdk/320-lambda).

### Create lambda function

Create a directory.
```
$ mkdir lambda
$ tree -L 1
.
├── README.md
├── bin
├── cdk.json
├── cdk.out
├── jest.config.js
├── lambda
├── lib
├── node_modules
├── package-lock.json
├── package.json
├── test
└── tsconfig.json
```

Create a javascript file.
```
$ cd lambda
$ touch hello.js
```

The content is like this.

```
$ cat hello.js

exports.handler = async function(event) {
    console.log('request:', JSON.stringify(event, undefined, 2))
    return {
        statusCode: 200,
        headers: { 'Content-Type': 'text/plain' },
        body: 'Hello, CDK! You have hit ${event.path}\n',

    }
}
```

This function do two things.
* print request content to console.
* return URL path when called

### Rewrite ./lib/aws-stack.ts

Rewrite the code, but add 'output' so that we can test the Lambda function with curl. 

```./lib/aws-stack.ts
// import * as cdk from 'aws-cdk-lib';
import {Stack, StackProps, CfnOutput} from 'aws-cdk-lib';
import {Code, Function, Runtime, FunctionUrlAuthType} from 'aws-cdk-lib/aws-lambda';
import { Construct } from 'constructs';

export class AwsStack extends Stack {
  constructor(scope: Construct, id: string, props?: StackProps) {
    super(scope, id, props);

// defines an AWS Lambda resource
  const myFunction = new Function(this, 'HelloHandler', {
    runtime: Runtime.NODEJS_18_X,   // execution environment
    code: Code.fromAsset('lambda'), // code loaded from lambda directory
    handler: 'hello.handler',       // file is "hello", function is "handler"
  });

  const myFunctionUrl = myFunction.addFunctionUrl({
    authType: FunctionUrlAuthType.NONE,
  });

  new CfnOutput(this, "myFunctionUrlOutput", {
    value: myFunctionUrl.url,
  });

  }
}
```

### Check and deploy

```
# c diff
```

then, 

```
# c deploy
....
Outputs:
AwsStack.myFunctionUrlOutput = http://xxxxxxxxxx.lambda-url.xxxxxxxxxx.localhost.localstack.cloud:4566/
Stack ARN: arn:aws:cloudformation:xxxxxxxx

```

Test with curl.

```
# curl http://hs4qbsexgvxpe3phsttz1zgopwik3uew.lambda-url.eu-central-1.localhost.localstack.cloud:4566/
Hello, CDK! You have hit ${event.path}
```

All right.
-------------------------------------------------
## Stack 3

Edit the code
```
// import * as cdk from 'aws-cdk-lib';
import {Stack, StackProps, CfnOutput} from 'aws-cdk-lib';
import {Code, Function, Runtime, FunctionUrlAuthType} from 'aws-cdk-lib/aws-lambda';
import { Construct } from 'constructs';
import {LambdaRestApi} from 'aws-cdk-lib/aws-apigateway';

export class AwsStack extends Stack {
  constructor(scope: Construct, id: string, props?: StackProps) {
    super(scope, id, props);

// defines an AWS Lambda resource
  const hello = new Function(this, 'HelloHandler', {
    runtime: Runtime.NODEJS_18_X,   // execution environment
    code: Code.fromAsset('lambda'), // code loaded from lambda directory
    handler: 'hello.handler',       // file is "hello", function is "handler"
  });

  const helloUrl = hello.addFunctionUrl({
    authType: FunctionUrlAuthType.NONE,
  });

  new CfnOutput(this, "helloUrlOutput", {
    value: helloUrl.url,
  });

  // define API Gateway REST API resource
  const myGateway = new LambdaRestApi(this, "Endpoint", {
    handler: hello,
  });
  }
}
```
Deploy it.
```
c diff
```
and 
```
c deploy
....

Outputs:
AwsStack.Endpoint8024A810 = https://xxxxxxxxx.execute-api.localhost.localstack.cloud:4566/prod/

```

Test is
```
# curl https://80z87lqi5m.execute-api.localhost.localstack.cloud:4566/prod/
Hello, CDK! You have hit /
```
All right. 


-------------------------------------------------
## Trouble shooting

* Changeset not found
```
c deploy --method=direct
c bootstrap -f
```

If nothing worked, start from 
```
c init -l typescript
```
or 
```
docker run 
.... 
```
To delete Changeset
```
aws cloudformation delete-change-set \
    --change-set-name \
      arn:aws:cloudformation:us-east-1:123456789012:changeSet/SampleChangeSet/1a2345b6-0000-00a0-a123-00abc0abc000
```
See []'Updating stacks using change sets'](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks-changesets.html).

-------------------------------------------------

# END
-------------------------------------------------