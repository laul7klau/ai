# ai
This project builds AWS resources required to demo LLM RAG on AWS Bedrock

## Prerequisites:
- Install aws cli on your client notebook.
- If your organization implements individual SSO accounts:
  - Login to your profile on the CLI:  
    USERNUM=??  
    USERPROFILE=Users-$USERNUM  
    aws sso login --profile <PROFILE_NAME>  
    aws sso login --profile $USERPROFILE  
  
  - To verify you're logged in, run:  
    aws sts get-caller-identity --profile $USERPROFILE
    
  
## Steps:
There are 5 Cloudformation yaml files in this repository to deploy:
- [Deploy AWS bedrock](#Deploy-AWS-bedrock)
- [Deploy BIG-IP](#Deploy-BIGIP)
- [Destroy stack](#Destroy-stack)

### Deploy AWS bedrock
This section deploys 4 files to create the following AWS bedrock resources: 
1. Set up environment variables:  
USERNUM=??  
IAMUSERARN=arn:aws:iam::$USERNUM:user/??  
  
   USERPROFILE=Users-$USERNUM  
   DATASOURCE=inagent-kb-$USERNUM  
   S3BUCKETARN=arn:aws:s3:::inagent-kb-$USERNUM  
  
2. Run this to dbstack: 3 IAM Roles, AttachedPolicyLambdab OpenSearchServerless collection, DataAccesPolicy, Encryption policy, Create/Delete S3bucket, Lambda function:
```
aws cloudformation create-stack --region us-west-2 --stack-name dbStack --profile $USERPROFILE --template-body file://./1vectordb.yaml --capabilities CAPABILITY_AUTO_EXPAND CAPABILITY_NAMED_IAM CAPABILITY_NAMED_IAM --parameters ParameterKey=IAMUserArn,ParameterValue=$IAMUSERARN
```

2. Run this to create kbstack: the knowledge base, AI agent stack:
```
COLLECTIONARN=??  
AMAZONBEDROCKEXCEUTIONROLEFORKNOWLEDGEBASE=arn:aws:iam::$USERNUM\:role/AmazonBedrockExecutionRoleForKnowledgeBase-inagent-kb
  
aws cloudformation create-stack --region us-west-2 --stack-name kbagentStack --profile $USERPROFILE --template-body file://./2kbagent.yaml --capabilities CAPABILITY_AUTO_EXPAND CAPABILITY_NAMED_IAM --parameters ParameterKey=AmazonBedrockExecutionRoleForKnowledgeBasearn,ParameterValue=$AMAZONBEDROCKEXCEUTIONROLEFORKNOWLEDGEBASE ParameterKey=CollectionArn,ParameterValue=$COLLECTIONARN ParameterKey=S3bucketarn,ParameterValue=$S3BUCKETARN ParameterKey=DataSource,ParameterValue=$DATASOURCE
```

3. Run this to the apigw stack:
```
BEDROCKAGENTID=??  
BEDROCKAGENTALIAS=??  
  
aws cloudformation create-stack --region us-west-2 --stack-name apigwStack --profile $USERPROFILE --template-body file://./3apigw.yaml --capabilities CAPABILITY_AUTO_EXPAND CAPABILITY_NAMED_IAM CAPABILITY_NAMED_IAM --parameters ParameterKey=BedrockAgentId,ParameterValue=$BEDROCKAGENTID ParameterKey=BedrockAgentAlias,ParameterValue=$BEDROCKAGENTALIAS  
```

### Deploy BIGIP
This section creates a VPC, BIG-IP EC2 instanace and other required resources.
Set USERPROFILE if you haven't:  
USERNUM=??  
USERPROFILE=Users-$USERNUM  
  
1. Run this to deploy the entire VPC stack
```
aws cloudformation create-stack --region us-west-2 --stack-name bigipStack --profile $USERPROFILE --template-body file://./4bigip.yaml --parameters ParameterKey=restrictedSrcAddressMgmt,ParameterValue=0.0.0.0/0 ParameterKey=restrictedSrcAddressApp,ParameterValue=0.0.0.0/0
```

2. Run this to retrieve the private key of the BIG-IP and save it to your local drive
```
aws ssm get-parameter --name "/ec2/keypair/key-???" --with-decryption --query Parameter.Value --output text --region us-west-2 --profile $USERPROFILE > mykey.pem
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

   
