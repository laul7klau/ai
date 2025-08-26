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
- AWS bedrock infra files to deploy bedrock vector db, knowledge base, agent AI, API gateway.
  - Deploy files 1vectordb to 4apigw sequentially.
- AWS EC2 file to deploy VPC, EC2 BIG-IP and so on.
  - Deploy 5bigip. 
