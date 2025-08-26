# ai
This project builds AWS resources required to demo LLM RAG on AWS Bedrock

## Prerequisites:
- Install aws cli on your client notebook.
- If your organization implements individual SSO accounts:
  - Login to your profile on the CLI:
    aws sso login --profile <PROFILE_NAME>
    aws sso login --profile Users-358712379163

  - To verify you're logged in, run:
    aws sts get-caller-identity --profile Users-358712379163

## Steps:
There are 5 Cloudformation yaml files in this repository to deploy:
- [Deploy AWS bedrock](### Deploy AWS bedrock)
- [Deploy BIG-IP](### Deploy BIG-IP)
- [Deploy stack](### Destroy stack)

### Deploy AWS bedrock
This section deploys 4 files to create the following AWS bedrock resources: 
1. Run this to create 3 IAM Roles, AttachedPolicyLambdab OpenSearchServerless collection, DataAccesPolicy, Encryption policy, Create/Delete S3bucket, Lambda function:
```
aws cloudformation create-stack --region us-west-2 --stack-name dbStack --profile Users-358712379163 --template-body file://./1vectordb.yaml --capabilities CAPABILITY_AUTO_EXPAND CAPABILITY_NAMED_IAM CAPABILITY_NAMED_IAM --parameters ParameterKey=IAMUserArn,ParameterValue=arn:aws:iam::358712379163:user/User07
```

2. Run this to create the knowledge base, AI agent stack:
```
aws cloudformation create-stack --region us-west-2 --stack-name kbagentStack --profile Users-358712379163 --template-body file://./2kbagent.yaml --capabilities CAPABILITY_AUTO_EXPAND CAPABILITY_NAMED_IAM CAPABILITY_NAMED_IAM --parameters ParameterKey=AmazonBedrockExecutionRoleForKnowledgeBasearn,ParameterValue=arn:aws:iam::358712379163:role/AmazonBedrockExecutionRoleForKnowledgeBase-inagent-kb ParameterKey=CollectionArn,ParameterValue=arn:aws:aoss:us-west-2:358712379163:collection/lpecu56aqhhl8iw0i2m6 ParameterKey=S3bucketarn,ParameterValue=arn:aws:s3:::inagent-kb-358712379163 ParameterKey=DataSource,ParameterValue=inagent-kb-358712379163
```

3. Run this to the client stack:
```
aws cloudformation create-stack --region us-west-2 --stack-name clientstack --profile Users-358712379163 --template-body file://./3client.yaml --capabilities CAPABILITY_AUTO_EXPAND CAPABILITY_NAMED_IAM CAPABILITY_NAMED_IAM
```

4. Run this to create
```
aws cloudformation create-stack --region us-west-2 --stack-name apigwStack --profile Users-358712379163 --template-body file://./4apigw.yaml --capabilities CAPABILITY_AUTO_EXPAND CAPABILITY_NAMED_IAM CAPABILITY_NAMED_IAM --parameters ParameterKey=LayerBucketName,ParameterValue=clientstack-bedrock-layer-bucket ParameterKey=BedrockAgentId,ParameterValue=VWJUQ5M47T ParameterKey=BedrockAgentAlias,ParameterValue=J6PQDVVG0B
```

### Deploy BIG-IP
This section creates a VPC, BIG-IP EC2 instanace and other required resources.
1. Run this to deploy the entire VPC stack
```
aws cloudformation create-stack --region us-west-2 --stack-name bigipStack --profile Users-358712379163 --template-body file://./quickstart.yaml --parameters ParameterKey=restrictedSrcAddressMgmt,ParameterValue=0.0.0.0/0 ParameterKey=restrictedSrcAddressApp,ParameterValue=0.0.0.0/0
```

2. Run this to retrieve the private key of the BIG-IP and save it to your local drive
```
aws ssm get-parameter --name "/ec2/keypair/key-0af28da890e61a1a5" --with-decryption --query Parameter.Value --output text --region us-west-2 --profile Users-358712379163 > mykey.pem
```

### Destroy stack
Before running the following commands to destroy the stacks, on the AWS console do the following:
- Empty the clientstack-bedrock-layer-bucket and KB S3 buckets
- Delete the AI agent alias

```
aws cloudformation delete-stack --stack-name apigwStack --profile Users-358712379163
```
```
aws cloudformation delete-stack --stack-name clientstack --profile Users-358712379163
```
```
aws cloudformation delete-stack --stack-name kbagentStack --profile Users-358712379163
```
```
aws cloudformation delete-stack --stack-name dbStack --profile Users-358712379163
```

   
