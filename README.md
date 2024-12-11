AWS DEVELOPER ASSOCIATE (DVA-C02) EXAM NOTES - David Galera, December 2024
- [Cloudfront](#cloudfront)
- [CDK](#cdk)
- [ECS](#ecs)
- [SQS](#sqs)
- [Parameter store](#parameter-store)
- [Kinesis](#kinesis)
- [Codebuild](#codebuild)
- [CodeDeploy](#codedeploy)
- [Application Load Balancer (ALB)](#application-load-balancer-alb)
- [Auto Scaling](#auto-scaling)
- [Beanstalk](#beanstalk)
- [Cloudwatch logs](#cloudwatch-logs)
- [Cloudwatch metrics](#cloudwatch-metrics)
- [SQS](#sqs-1)
- [Lambda](#lambda)
- [Step functions](#step-functions)
- [API Gateway](#api-gateway)
- [RDS](#rds)
- [EC2](#ec2)


## Cloudfront
Client - Cloudfront (HTTPS), Cloudfront - Backend/origin (HTTPS)
Origin protocol policy: HTTPS Only or Match Viewer (same proto as the viewer)

**Signed URL** is generated by our application with SDK, give access to 1 file. Give access to a **path**, no matter the origin (work for more than s3 origins). They are key-pairs, only root account can manage them. Can **filter by IP, path, date, expiration**. Can leverage caching features. 2 types of signers used to **sign URLS and cookies**:
1. **trusted key group** (recommended), can rotate keys. Generate public/key pair, private key is used by services to generate the URL and public key is used by Cloudfront to verify the URL, you can associate a high number of public keys with your CloudFront distribution. The public key can be regenerated from the private key. You create a **key group** and add up to 5 public keys, this group is what cloudfront references to allow EC2 (for example) to create signed URLs.
2. **Cloudfront key pair** (deprecated, must use root account), only can be created by the **root account**. Up to two active CloudFront key pairs per AWS account


**Signed Cookies** give access to multiple files, static URL.

Cost for **data out** varies per edge location. 3 price classes:
- Price Class all -> all regions, expensive.
- Price Class 200 -> almost all regions, exclude the most expensive ones.
- Price Class 100 -> Only the least expensive regions.

**Multiple origins**

Route based on content-type, path pattern and more e.g. /api/* route to an ALB, /* route to s3. **Origins can serve static and dynamic content**. Cloudfront can improve performance for both cacheable content and dynamic content while Global Accelerator is for non-HTTP.

**Origin groups**
- Do failover, increase HA. 1 primary origin and 1 secondary origin
- Failover EC2s, s3 buckets etc

**Field level encryption**

Encrypt at the edge location, uses asymmetric encryption, specify up to 10 fields in a POST request and the **public key** to encrypt them (at the edge). Fields will be decrypted in the backend e.g. (EC2) using a **private key**

**Real time logs**

All cloudfront real-time requests can be send to Kinesis Data Streams. You can use it in a % of requests or for specific fields or cache behaviours (paths)

## CDK
Workflow:
1. Create the app from a template provided by AWS CDK. `cdk init`
2. Add code to the app to create resources within stacks - Add custom code as is needed for your application.
3. Build the app (optional) (compile) it.
4. Synthesize one or more stacks in the app to create an AWS CloudFormation template. This step catches logical errors in defining resources.
5. Deploy one or more stacks to your AWS account - It is optional (though good practice) to synthesize before deploying `cdk deploy`

## ECS
ECS container agent (EC2) configuration at **/etc/ecs/ecs.config**. Here you specify the `ECS_CLUSTER='your_cluster_name' ` that the instance containers will belong to.

**Tasks**: The smallest deployable unit in ECS. The container images to use.
The CPU and memory allocation for each container.
Networking and IAM roles.
Environment variables and volumes.

**Services**: A service is a long-running ECS component that manages tasks and ensures that the desired number of task instances are running and available. Can integrate with load balancers to distribute traffic across tasks. Allow you to specify scaling policies to adjust the number of tasks based on metrics (e.g., CPU usage).

ALB uses dynamic port mapping (EC2 launch type only) -> Allow multiple tasks from the same service to run on the same EC2 instance, each task has assigned (dynamically) a unique HOST PORT (EC2 port). ALB routes to the HOST PORT

If you terminate a container instance in the **RUNNING** state, that container instance is automatically removed, or deregistered, from the cluster

If you terminate a container instance while it is in the **STOPPED** state, that container instance isn't automatically removed from the cluster. You will **need to deregister your container instance** in the STOPPED state by using the Amazon ECS console or AWS Command Line Interface.

## SQS
Amazon SQS can scale transparently to handle the load without any provisioning instructions from you.

## Parameter store
SecureString are parameters that have a **plaintext parameter name** and an **encrypted parameter value**. Parameter Store uses **AWS KMS** to encrypt and decrypt the parameter values of SecureString parameters, 1 API call per decryption.

## Kinesis
The Amazon Kinesis Client Library (KCL) delivers all records for a given partition key to the same record processor

## Codebuild
`CODEBUILD_KMS_KEY_ID` The identifier of the AWS KMS key that CodeBuild is using to encrypt the build output artifact (for example, `arn:aws:kms:region-ID:account-ID:key/key-ID` or alias/key-alias).

## CodeDeploy
- In Place Deployment: Only for EC2/On-premises. The application on each instance in the deployment group is stopped, the latest application revision is installed, and the new version of the application is started and validated. You can use a load balancer so that each instance is deregistered during its deployment and then restored to service after the deployment is complete.
- Blue/green Deployment
  - Lambda: All deploys are Blue/Green
  - ECS: traffic is shifted to a replacement task set in the same service. Traffic shifting can be **linear or canary**
  - EC2/On-premises: Instances are provisioned for the **replacement** environment, latest revision is installed in them, optional testing, new instances are registered in the ALB target group. Instances in the original environment are deregistered and can be terminated or kept running. 



## Application Load Balancer (ALB)
Sticky sessions (AWSALB cookie) are a mechanism to route requests to the same target in a target group. To use sticky sessions, the clients must support **cookies**.

A Classic Load Balancer with HTTP or HTTPS listeners might route more traffic to higher-capacity instance types.

After you disable an Availability Zone, the targets in that Availability Zone remain registered with the load balancer. However, even though they remain registered, the load balancer does not route traffic to them

## Auto Scaling
Target Tracking Scaling policy metrics:
- ALBRequestCountPerTarget
- ASGAverageCPUUtilization
- ASGAverageNetworkOut - Average number of bytes sent out on all network interfaces by the Auto Scaling group

## Beanstalk
You can deploy any version of your application to any environment. Environments can be long-running or temporary. When you terminate an environment, you can save its configuration to recreate it later.

**Failed deployment**: Elastic Beanstalk will replace the failed instances with instances running the application version from the most recent successful deployment

**Immutable deployments** perform an immutable update to launch a full set of new instances running the new version of the application in a separate Auto Scaling group, alongside the instances running the old version. Immutable deployments can prevent issues caused by partially completed rolling deployments

**Traffic-splitting deployments** let you perform canary testing as part of your application deployment. In a traffic-splitting deployment, Elastic Beanstalk launches a full set of new instances just like during an immutable deployment. It then forwards a specified percentage of incoming client traffic to the new application version for a specified evaluation period.

**Rolling deployments**, Elastic Beanstalk splits the environment's Amazon EC2 instances into batches and deploys the new version of the application to one batch at a time.

## Cloudwatch logs
You can export from multiple **log groups** or multiple **time ranges** to **s3 bucket**

## Cloudwatch metrics
You can configure custom metrics:
- Standard resolution (default), with data having a one-minute granularity
- High resolution, with data at a granularity of one second
  
Monitoring:
- Basic monitoring
- Detailed monitoring: provides more frequent metrics, published at one-minute intervals, instead of the five-minute intervals used in Amazon EC2 basic monitoring. Detailed monitoring is offered by only some services.

## SQS
Visibility timeout 0-12 hours, default to 30 seconds.

1 byte < Message size < 256 KB

**DeleteQueue** -> When you delete a queue, the deletion process takes up to 60 seconds. Meanwhile, messages sent to the queue can succed and be consumed.

## Lambda
Alias can only point to function version, not another alias.

VPC: When you connect a function to a VPC, Lambda creates an **elastic network interface** for each combination of the security group and subnet in your function's VPC configuration.

## Step functions
A Task state (`"Type": "Task"`) represents a single unit of work performed by a state machine.
`Resource` field is a required parameter for `Task` state.

`"Type": "Wait"` delays the state machine from continuing for a specified time

`"Type": "Fail"` stops the execution of the state machine and marks it as a failure unless it is caught by a Catch block

## API Gateway
**Promote a stage**: The promotion can be done by redeploying the API to the prod stage OR updating a stage variable value from the stage name of test to that of prod.

## RDS
IAM authorization: Available for **MySQL, PostGres and MariaDB**

You can enable **storage autoscaling** and set the max storage limit, triggers:
- Free available space is less than 10 percent of the allocated storage.
- The low-storage condition lasts at least five minutes.
- At least six hours have passed since the last storage modification.

## EC2
Change `DeleteOnTermination` default=True for the root volume, and default=False for the rest of the volumes:
- When the instance is running can only be changed via CLI
- From the console can only be set when you launch a new instance 
