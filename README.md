AWS DEVELOPER ASSOCIATE (DVA-C02) EXAM NOTES - David Galera, December 2024
- [CLI](#cli)
- [EBS](#ebs)
- [S3](#s3)
- [IAM](#iam)
- [Amazon Inspector](#amazon-inspector)
- [AWS Trusted Advisor](#aws-trusted-advisor)
- [Cloudfront](#cloudfront)
- [CDK](#cdk)
- [SAM](#sam)
- [ECS](#ecs)
- [SNS](#sns)
- [SQS](#sqs)
- [KMS](#kms)
- [Parameter store](#parameter-store)
- [Secrets Manager](#secrets-manager)
- [Kinesis data streams](#kinesis-data-streams)
- [CodeCommit](#codecommit)
- [CodeBuild](#codebuild)
- [CodeDeploy](#codedeploy)
- [CodePipeline](#codepipeline)
- [Application Load Balancer (ALB)](#application-load-balancer-alb)
- [Auto Scaling](#auto-scaling)
- [Beanstalk](#beanstalk)
- [Cloudwatch Logs](#cloudwatch-logs)
- [Cloudwatch metrics](#cloudwatch-metrics)
- [CloudWatch Alarms](#cloudwatch-alarms)
- [Lambda](#lambda)
- [Step functions](#step-functions)
- [API Gateway](#api-gateway)
- [Cognito](#cognito)
- [DynamoDB](#dynamodb)
- [ElastiCache](#elasticache)
- [RDS](#rds)
- [EC2](#ec2)
- [Cloudformation](#cloudformation)
- [SAM](#sam-1)
- [X-Ray](#x-ray)
- [CloudTrail](#cloudtrail)
- [Macie](#macie)
- [ECR](#ecr)
- [Athena](#athena)
- [AppConfig](#appconfig)
- [Amplify](#amplify)
- [AppSync](#appsync)

## CLI
`put-metric-data` command publishes metric data points to Amazon CloudWatch

`put-metric-alarm` creates or updates an alarm and associates it with the specified metric, metric math expression, or anomaly detection model.

`--dry-run`: checks whether you have the required permissions for the action, without actually making the request, and provides an error response. If you have the **required permissions**, the error response is `DryRunOperation`, otherwise, it is `UnauthorizedOperation`

`--page-size` - You can use the --page-size option to specify that the AWS CLI requests a smaller number of items from each call to the AWS service. The CLI still retrieves the full list but performs a larger number of service API calls in the background and retrieves a smaller number of items with each call.

Make few API calls -> `--max-items` and `--starting-token`

Add a role to EC2 instance:
- `aws iam create-instance-profile --instance-profile-name EXAMPLEPROFILENAME`
- `aws iam add-role-to-instance-profile --instance-profile-name EXAMPLEPROFILENAME --role-name EXAMPLEROLENAME`
- `aws ec2 associate-iam-instance-profile --iam-instance-profile Name=EXAMPLEPROFILENAME --instance-id i-012345678910abcde`

## EBS
- gp2: minimum of 100 IOPS (at 33.33 GiB and below) and a maximum of 16,000 IOPS (at 5,334 GiB and above)

- io1 4-16 GiB, 100-64k(Nitro) IOPS, ratio IOPS/GB <= 50:1

To encrypt an unencrypted EBS, you **must create a copy**.

You can configure your AWS account to enforce the encryption of the new EBS volumes and snapshot copies that you create. **Encryption by default is a Region-specific setting**. If you enable it for a Region, you cannot disable it for individual volumes or snapshots in that Region.
## S3
**SSE-KMS does NOT encrypt object metadata**.

If two writes are made to a single **non-versioned object** at the same time, it is possible that only a single **event notification** will be sent. If you want to **ensure** that an event notification is sent for every successful write, you should **enable versioning on your bucket**. With versioning, every successful write will create a new version of your object and will also send event notification.

S3 also provides **strong consistency for list operations**, so after a write, you can immediately perform a listing of the objects in a bucket with any changes reflected.

S3 delivers strong read-after-write consistency, helps when you need to immediately read an object after a write.

If you delete a bucket and immediately list all buckets, **the deleted bucket might still appear in the list** - Bucket configurations have an **eventual consistency** model.

**Replication**
- Replicated objects RETAIN metadata
- SRR and CRR -> S3 bucket level, a shared prefix level, or an object level using S3 object tags
- Lifecycle actions are not replicated

**ACLs**, customers can grant specific permissions (i.e. READ, WRITE, FULL_CONTROL) to specific users for an individual bucket or object.

**Query String Authentication** also referred as **Pre-signed URL**

**S3 Object Ownership** has two settings: 1. Object writer – The uploading account will own the object. 2. Bucket owner preferred – The bucket owner will own the object if the object is uploaded with the bucket-owner-full-control canned ACL. Without this setting and canned ACL, the object is uploaded and remains owned by the uploading account.

**Inventory** You can use it to audit and report on the **replication and encryption** status of your objects for business, compliance, and regulatory needs

**S3 Analytics**: Analyze **storage access patterns** to help you decide when to transition the right data to the right storage class

## IAM
**Access Analyzer**: You can set the scope for the analyzer to an **organization or an AWS account**. Identify unintended access to your resources and data, which is a security risk. Helps inspecting unused access to guide you toward least privilege.
When the **unused access analyzer is enabled**, IAM Access Analyzer continuously analyzes your accounts to identify unused access and creates a centralized dashboard with findings. The findings highlight unused roles, unused access keys for IAM users, and unused passwords for IAM users. 

**Access Analyzer for S3** helps review all buckets that have bucket access control lists (ACLs), bucket policies, or access point policies that grant public or shared access. Access Analyzer for S3 alerts you to buckets that are configured to allow access to anyone on the internet or other AWS accounts, including AWS accounts outside of your organization.

## Amazon Inspector
Amazon Inspector is an automated security assessment service that helps improve the **security and compliance** of applications deployed on AWS. Amazon Inspector automatically assesses applications for exposure, vulnerabilities, and deviations from best practices.

## AWS Trusted Advisor
AWS Trusted Advisor is an online tool that provides you real-time guidance to help you provision your resources following AWS best practices on **cost optimization**, security, fault tolerance, service limits, and performance improvement.

## Cloudfront
Client - Cloudfront (HTTPS), Cloudfront - Backend/origin (HTTPS)
Origin protocol policy: **HTTPS Only** or **Match Viewer** (same proto as the viewer)

You cannot directly integrate Cognito User Pools with CloudFront distribution as you have to create a separate Lambda@Edge function to accomplish the authentication via Cognito User Pools.

**Cloudfront functions**: When a viewer request to CloudFront results in a cache miss (the requested object is not cached at the edge location), CloudFront sends a request to the origin to retrieve the object. This is called an origin request. The origin request always includes the following information from the viewer request:
- The URL path (the path only, without URL query strings or the domain name)
- The request body (if there is one)
- The HTTP headers that CloudFront automatically includes in every origin request, including Host, User-Agent, and X-Amz-Cf-Id **CloudFront-Viewer-Country** header.

![cloudfront-func](cloudfront-func.jpg)

**Lambda@Edge** functions can only be created in the **us-east-1**

![lambda-edge](lambda-edge.jpg)

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
- CloudFront routes all incoming requests to the primary origin, even when a previous request failed over to the secondary origin
- CloudFront fails over to the secondary origin only when the HTTP method of the viewer request is GET, HEAD or OPTIONS. CloudFront **does not failover** when the viewer sends a different HTTP method (for example POST, PUT, and so on).

**Cache Behaviour**
- Order matters, upper cache-behaviour listings are evaluated first.
- You must create at least as many cache behaviors (including the default cache behavior) as you have origins if you want CloudFront to serve objects from all of the origins. Each cache behavior specifies the one origin from which you want CloudFront to get objects. If you have two origins and only the default cache behavior, the default cache behavior will cause CloudFront to get objects from one of the origins, but the other origin is never used.
- The path pattern for the default cache behavior is * and cannot be changed. If the request for an object does not match the path pattern for any cache behaviors, CloudFront applies the behavior in the default cache behavior.

**Field level encryption**: Encrypt at the edge location, uses asymmetric encryption, specify up to 10 fields in a POST request and the **public key** to encrypt them (at the edge). Fields will be decrypted in the backend e.g. (EC2) using a **private key**

**Real time logs**: All cloudfront real-time requests can be send to **Kinesis Data Streams**. You can use it in a % of requests or for specific fields or cache behaviours (paths)

To enable SSL between the origin and the distribution the Developer can configure the Origin Protocol Policy. Depending on the domain name used (CloudFront default or custom), the steps are different. To enable SSL between the end-user and CloudFront the Viewer Protocol Policy should be configured.

If you configure CloudFront to require HTTPS both to communicate with viewers and to communicate with your origin, here's what happens when CloudFront receives a request for an object:

1. A viewer submits an HTTPS request to CloudFront. There's some SSL/TLS negotiation here between the viewer and CloudFront. In the end, the viewer submits the request in an encrypted format.

2. If the object is in the CloudFront edge cache, CloudFront encrypts the response and returns it to the viewer, and the viewer decrypts it.

3. If the object is not in the CloudFront cache, CloudFront performs SSL/TLS negotiation with your origin and, when the negotiation is complete, forwards the request to your origin in an encrypted format.

4. Your origin decrypts the request, encrypts the requested object, and returns the object to CloudFront.

5. CloudFront decrypts the response, re-encrypts it, and forwards the object to the viewer. CloudFront also saves the object in the edge cache so that the object is available the next time it's requested.

6. The viewer decrypts the response.
## CDK
Workflow:
1. Create the app from a template provided by AWS CDK. `cdk init`
2. Add code to the app to create resources within stacks - Add custom code as is needed for your application.
3. Build the app (optional) (compile) it.
4. Synthesize one or more stacks in the app to create an AWS CloudFormation template. This step catches logical errors in defining resources.
5. Deploy one or more stacks to your AWS account - It is optional (though good practice) to synthesize before deploying `cdk deploy`

Synthesis is the process of converting AWS CDK stacks to AWS CloudFormation templates and assets. Changes should only be during the deployment phase and after AWS CloudFormation template has been created. This will allow it to roll back the change if there are any problems.

The CDK Toolkit already provides the ability to **convert CDK stacks into CloudFormation templates**. There is no need to install CloudFormation.

## SAM
`Transform` section and `Resources` sections are required; the `Globals` section is optional.

`aws cloudformation package` or `sam package` commands to prepare the local artifacts (local paths) that your AWS CloudFormation template references. The command uploads local artifacts to an S3 bucket, returns a copy of your template, replacing references to local artifacts with the S3 location where the command uploaded the artifacts e.g. `aws cloudformation package --template-file /path_to_template/template.json --s3-bucket bucket-name --output-template-file packaged-template.json`

Once that is complete the template can be deployed using the “aws cloudformation deploy” or “sam deploy” commands

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

`aws ecs create-service --service-name ecs-simple-service --task-definition ecs-demo --desired-count 10`
To create a new service you would use this command which creates a service in your default region called ecs-simple-service. The service uses the ecs-demo task definition and it maintains 10 instantiations of that task.

Central logs in Fargate: Using the `awslogs log driver` you can configure the containers in your tasks to send log information to CloudWatch Logs. If you're using the Fargate launch type for your tasks, you need to add the required logConfiguration parameters to your task definition to turn on the awslogs log driver.

If you're using the **EC2 launch type (and not Fargate)** for your tasks and want to turn on the awslogs log driver, your Amazon ECS container instances require at least **version 1.9.0 of the container agent**.

Task placement strategies can be specified when either running a task or creating a new service:
- **binpack** - Place tasks based on the least available amount of CPU or memory. This minimizes the number of instances in use.
- **random** - Place tasks randomly.
- **spread** - Place tasks evenly based on the specified value. Accepted values are instanceId (or host, which has the same effect) (instanceId -> distributes tasks evenly across the instances), or any platform or custom attribute that is applied to a container instance, such as `attribute:ecs.availability-zone`. Service tasks are spread based on the tasks from that service. Standalone tasks are spread based on the tasks from the same task group.

A **task placement constraint** is a rule that is considered during task placement. Task placement constraints can be specified when either running a task or creating a new service:
- `distinctInstance`: Place each task on a different container instance. This task placement constraint can be specified when either running a task or creating a new service.
- `memberOf`: Place tasks on container instances that satisfy an expression with Cluster Query Language

In Amazon ECS, create a Docker image that runs the **X-Ray daemon**, upload it to a Docker image repository, and then deploy it to your Amazon ECS cluster. You can use port mappings and network mode settings in your task definition file to allow your application to communicate with the daemon container.

![task-roles](task-roles.webp)

## SNS
By default, a topic **subscriber** receives every message that's published to the topic. To receive only a subset of the messages, a subscriber must assign a **filter policy** to the topic subscription. Amazon SNS supports policies that act on the message attributes or the message body.

## SQS
**MessageDeduplicationId**: The message deduplication ID is the token used for the deduplication of sent messages. If a message with a particular message deduplication ID is sent successfully, any messages sent with the same message deduplication ID are accepted successfully but aren't delivered during the 5-minute deduplication interval.

**FIFO queues not allowed** for s3 event notification destination.

Visibility timeout 0-12 hours, default to 30 seconds.

1 byte < Message size < 256 KB

120,000 inflight messages (20k FIFO) (received from a queue by a consumer, but not yet deleted from the queue)

**DeleteQueue** -> When you delete a queue, the deletion process takes up to 60 seconds. Meanwhile, messages sent to the queue can succed and be consumed.

Amazon SQS can scale transparently to handle the load without any provisioning instructions from you.

Max message size -> **256 KB**. To manage large messages, you can use Amazon S3 and the Amazon **SQS Extended Client Library for Java**. This is especially useful for storing and consuming messages up to **2 GB**.

Delay queues time range - 0 sec, 15 min

The **default message retention period** is 4 days (60s - 14 days). `SetQueueAttributes` action.

## KMS
Support sending data up to **4 KB** to be encrypted directly.

To encrypt an EBS volume attached to an EC2 you need 2 KMS APIs: `GenerateDataKeyWithoutPlaintext` and `Decrypt`

**Envelope encryption** reduces the network load since only the request and delivery of the much smaller **data key** go over the network. The data key is used locally in your application or encrypting AWS service, avoiding the need to send the entire block of data to AWS KMS and suffer network latency.

![kms](kms.jpg)

`GenerateDataKey` API returns a plaintext version of the key and a copy of the key encrypted under a KMS key. The application can use the plaintext key to encrypt data, and then discard it from memory as soon as possible to reduce potential exposure. Data key caching and increase service quota to avoid ThrottlingErrors

`Encrypt` API encrypts data under a specified KMS key.

*Data keys* are encryption keys that you can use to encrypt data, including large amounts of data and other data encryption keys. **KMS does not store, manage, or track your data keys**, or perform cryptographic operations with data keys. You must use and manage data keys outside of AWS KMS – this is potentially less secure as you need to manage the security of these keys.

To **create a data key**, call the `GenerateDataKey` operation. AWS KMS generates the data key. Then it encrypts a copy of the data key under a symmetric encryption KMS key that you specify. The operation returns a plaintext copy of the data key and the copy of the data key encrypted under the KMS key. The following image shows this operation.

![datakey](datakey.webp)

After using the **plaintext data key to encrypt data**, remove it from memory as soon as possible. You can safely store the encrypted data key with the encrypted data, so it is available to decrypt the data.

![plaintext data key to encrypt data](plaintextkey.webp)

`GenerateDataKeyWithoutPlainText` returns only an **encrypted data key**

## Parameter store
`SecureString` are parameters that have a **plaintext parameter name** and an **encrypted parameter value**. Parameter Store uses **AWS KMS** to encrypt and decrypt the parameter values of `SecureString` parameters, 1 API call per decryption.

You **cannot use a resource-based policy with a parameter** in the Parameter Store.

Parameter Store supports parameter policies that are available for parameters that use the advanced parameters tier. Parameter policies help you manage a growing set of parameters by allowing you to assign specific criteria to a parameter such as an **expiration date or TTL**. Parameter policies are especially helpful in forcing you to update or delete passwords and configuration data stored in Parameter Store, a capability of AWS Systems Manager. So this option is incorrect.

**Does not support automatic key rotation.**
## Secrets Manager
You can attach **resource-based** policies to a **secret**, to allow Principals e.g. IAM **Roles**, **Users**, other AWS **accounts**... to access them.

Secrets Manager natively supports rotating credentials for databases hosted on Amazon **RDS** and Amazon **DocumentDB** and clusters hosted on Amazon **Redshift**.
## Kinesis data streams

Limits can be exceeded by either data throughput (1MB/s per shard) or the number of PUT records (1000 puts/s). While the capacity limits are exceeded, the **put data call** will be rejected with a `ProvisionedThroughputExceeded` exception (data stream’s input data rate exceeded).

The `Amazon Kinesis Client Library (KCL)` delivers all records for a given partition key to the same record processor

Each `PutRecords` request can support up to 500 records. Each record in the request can be as large as 1 MiB, up to a limit of 5 MiB for the entire request, including partition keys. Each shard can support writes up to 1,000 records per second, up to a maximum data write of 1 MiB per second.

**Kinesis Agent** is a stand-alone Java software application that offers an easy way to collect and send data to Kinesis Data Streams. The agent continuously monitors a set of files and sends new data to your stream. The agent handles file rotation, checkpointing, and retry upon failures. It delivers all of your data in a reliable, timely, and simple manner. It also emits Amazon CloudWatch metrics to help you better monitor and troubleshoot the streaming process.

You can install the agent on Linux-based server environments such as web servers, log servers, and database servers. After installing the agent, configure it by specifying the files to monitor and the stream for the data. After the agent is configured, it durably collects data from the files and reliably sends it to the stream.

The agent can also pre-process the records parsed from monitored files before sending them to your stream. You can enable this feature by adding the dataProcessingOptions configuration setting to your file flow. One or more processing options can be added and they will be performed in the specified order.

## CodeCommit
Data in AWS CodeCommit repositories is encrypted in transit and at rest.

Credential types:
- SSH Keys: a locally generated public-private key pair that you can associate with your IAM user to communicate with CodeCommit repositories over SSH.
- Git credentials: An IAM -generated user name and password pair you can use to communicate with CodeCommit repositories over HTTPS.
- AWS Access keys: you can use with the credential helper included with the AWS CLI to communicate with CodeCommit repositories over HTTPS.

Migrate GitHub repos to CodeCommit -> Git credentials generated from IAM

## CodeBuild
`buildspec.yml` in the root directory.

`post_build` phase is an optional sequence. It represents the commands, if any, that CodeBuild runs after the build. For example, you might use Maven to package the build artifacts into a JAR or WAR file, or you might push a Docker image into Amazon ECR. Then you might send a build notification through Amazon SNS.

`install phase` commands are run during installation.

![postbuild](post-build.webp)

`CODEBUILD_KMS_KEY_ID` The identifier of the AWS KMS key that CodeBuild is using to encrypt the build output artifact (for example, `arn:aws:kms:region-ID:account-ID:key/key-ID` or alias/key-alias).

A typical application build process includes phases like preparing the environment, updating the configuration, downloading dependencies, running unit tests, and finally, packaging the built artifact. CodeBuild can upload artifacts to s3, attach an IAM role with s3 permissions.

Dependent files do not change frequently between builds, you can noticeably **reduce your build time by caching dependencies**.

![cache](codebuild-cache.jpg)

Can push metrics to Cloudwatch, scope: Project level or AWS account level. These metrics include the number of total builds, failed builds, successful builds, and the duration of builds

CodeBuild scales automatically to meet peak build requests.

**Debug Codebuild**: Run AWS CodeBuild **locally** using CodeBuild Agent. With the Local Build support for AWS CodeBuild, you just specify the location of your source code, choose your build settings, and CodeBuild runs build scripts for compiling, testing, and packaging your code.
- Test the integrity and contents of a buildspec file locally.
- Test and build an application locally before committing
- Identify and fix errors quickly from your local development environment.

It is **not possible to SSH into the CodeBuild Docker container**, that's why you should test and fix errors locally.

## CodeDeploy
CodeDeploy can deploy software packages using an archive that has been uploaded to an Amazon **S3 bucket**. The archive file will typically be a **.zip file** containing the code and files required to deploy the software package.

`appspec.yml` for specifying **deployment hooks**. The content in the 'hooks' section of the AppSpec file varies, depending on the compute platform for your deployment. An EC2/On-Premises deployment hook is executed once per deployment to an instance. You can specify one or more scripts to run in a hook. Some hooks:
- **ValidateService** is the last deployment lifecycle event. It is used to verify the deployment was completed successfully.
- **AfterInstall** - You can use this deployment lifecycle event for tasks such as configuring your application or changing file permissions.
- **ApplicationStart** - You typically use this deployment lifecycle event to restart services that were stopped during **ApplicationStop**
- **AllowTraffic** - During this deployment lifecycle event, internet traffic is allowed to access instances after a deployment. This event is reserved for the AWS CodeDeploy agent and cannot be used to run scripts

EC2 deployment example hooks: **BeforeInstall > AfterInstall > ApplicationStart > ValidateService**
Lambda valid hooks: **BeforeAllowTraffic > AfterAllowTraffic**
ECS valid hooks: **BeforeInstall > AfterInstall > AfterAllowTestTraffic > BeforeAllowTraffic**. `BeforeAllowTraffic` lifecycle event occurs before the updated task set is moved to the target group that is receiving live traffic. `AfterAllowTraffic` lifecycle event occurs after the updated task set is moved to the target group that is receiving live traffic.

![hooks](hooks.jpg)

![hooks-v2](hooksv2.jpg)

The **AppSpec file** is used to:
- Map the source files in your application revision to their destinations on the instance.
- Specify custom permissions for deployed files.
- Specify scripts to be run on each instance at various stages of the deployment process.
- MINIMUM properties required in `resources` for Lambda deployments: `name, alias, currentversion, and targetversion`

During deployment, the **CodeDeploy agent** looks up the name of the current event in the hooks section of the **AppSpec file**. If the event is not found, the CodeDeploy agent moves on to the next step. If the event is found, the CodeDeploy agent retrieves the list of scripts to execute. The scripts are run sequentially, in the order in which they appear in the file. The status of each script is logged in the CodeDeploy agent log file on the instance.

**Only 2 deployment options**:

**In Place Deployment**: Only for EC2/On-premises. The application on each instance in the deployment group is stopped, the latest application revision is installed, and the new version of the application is started and validated. You can use a load balancer so that each instance is deregistered during its deployment and then restored to service after the deployment is complete.
  
**Blue/green Deployment**:
- Lambda: All deploys are Blue/Green. Traffic is shifted from one version of a Lambda function to a new version of the same Lambda function. `CodeDeployDefault.LambdaLinear10PercentEvery1Minute`, `CodeDeployDefault.LambdaCanary10Percent5Minutes`.
- ECS: traffic is shifted from the task set with the original version of an application to a replacement task set in the same service. Traffic shifting can be **linear or canary**: `CodeDeployDefault.ECSCanary10Percent15Minutes`, `CodeDeployDefault.ECSLinear10PercentEvery10Minutes`
- EC2: **New** instances are provisioned for the **replacement** environment, latest revision is installed in them, optional testing, new instances are registered in the ALB target group causing traffic to be **re-route** from one set of instances in the original environment to the replacement set of instances. Instances in the original environment are deregistered and can be terminated or kept running. `HalfAtAtime` is only available for EC2/on-premises.
- On-premises: B/G deploys do not work.

![codedeploy](codedeploy.jpg)

**CodeDeploy agent** is a software package that, when installed and configured on an instance, makes it possible for that instance to be used in CodeDeploy deployments. The CodeDeploy agent archives revisions and log files on instances. The CodeDeploy agent cleans up these artifacts to conserve disk space. You can use the `:max_revisions:` option in the agent configuration file to specify the number of application revisions to the archive by entering any positive integer. CodeDeploy also archives the log files for those revisions. All others are deleted, except for the log file of the last successful deployment

CodeDeploy **rolls back deployments** by redeploying a previously deployed revision of an application as a new deployment. These rolled-back deployments are technically new deployments, with new deployment IDs, rather than restored versions of a previous deployment.

**CodeDeploy Deployment Groups**: You can specify one or more deployment groups for a CodeDeploy application. The deployment group contains settings and configurations used during the deployment. Most deployment group settings depend on the compute platform used by your application. Some settings, such as rollbacks, triggers, and alarms can be configured for deployment groups for any compute platform.

In an EC2/On-Premises deployment, a deployment group is a set of individual instances targeted for deployment. A deployment group contains individually tagged instances, Amazon EC2 instances in Amazon EC2 Auto Scaling groups, or both.

## CodePipeline
You can add an **approval action** to a stage in a CodePipeline pipeline at the point where you want the pipeline to stop so someone can manually approve or reject the action. **Approval actions can't be added to Source stages**. Source stages can contain only source actions.

- CodePipeline can be configured as an event source for **EventBridge**

- CodePipeline **cannot** be configured as a **trigger for Lambda**.
  
## Application Load Balancer (ALB)
After you create a target group, you cannot change its target type. The following are the possible target types:
- `Instance` - The targets are specified by instance ID
- `IP` - The targets are IP addresses. You can specify IP addresses from **specific CIDR blocks only**. **You can't specify publicly routable IP addresses.**
- `Lambda` - The target is a Lambda function

Sticky sessions (AWSALB cookie) are a mechanism to route requests to the same target in a target group. To use sticky sessions, the clients must support **cookies**.

A Classic Load Balancer with HTTP or HTTPS listeners might route more traffic to higher-capacity instance types.

After you disable an Availability Zone, the targets in that Availability Zone remain registered with the load balancer. However, even though they remain registered, the load balancer **does not route** traffic to them

**Access logs**: provides access logs that capture detailed information about requests sent to your load balancer and stores them in **S3**, no additional charge. Each log contains information such as the time the request was received, the client's IP address, latencies, request paths, and server responses.

**Request tracing**: track HTTP requests. The load balancer adds a header with a trace identifier to each request it receives. Request tracing will not help you to analyze latency specific data.

![alb](alb.jpg)

**statusCodes**:
- HTTP 403 - HTTP 403 is 'Forbidden' error. You configured an AWS WAF web access control list (web ACL) to monitor requests to your Application Load Balancer and it blocked a request.
- HTTP 500 - HTTP 500 indicates 'Internal server' error. There are several reasons for their error: A client submitted a request without an HTTP protocol, and the load balancer was unable to generate a redirect URL, there was an error executing the web ACL rules.
- HTTP 503 - HTTP 503 indicates 'Service unavailable' error. This error in ALB is an indicator of the target groups for the load balancer having no registered targets.
- HTTP 504 - HTTP 504 is 'Gateway timeout' error. Several reasons for this error, to quote a few: The load balancer failed to establish a connection to the target before the connection timeout expired, The load balancer established a connection to the target but the target did not respond before the idle timeout period elapsed.

## Auto Scaling
Target Tracking Scaling policy metrics:
- ALBRequestCountPerTarget
- ASGAverageCPUUtilization
- ASGAverageNetworkOut - Average number of bytes sent out on all network interfaces by the Auto Scaling group

## Beanstalk
Fully managed, capacity provisioning, load-balanced and autoscaling app: **Beanstalk + RDS**. AWS Elastic Beanstalk will handle all capacity provisioning, load balancing, and auto-scaling for the web front-end and Amazon RDS provides push-button scaling for the backend.

Source bundle uploaded from the console:
- Single Zip, Must not exceed 512 MB
- Not include a parent folder or top-level directory (subdirectories are fine)
- `cron.yaml` file is required for **worker environment**

Migrate a beanstalk environment from account A to account B: You must use saved configurations to migrate an Elastic Beanstalk environment between AWS accounts. You can save your environment's configuration as an object in Amazon Simple Storage Service (Amazon S3) that can be applied to other environments during environment creation, or applied to a running environment. Saved configurations are YAML formatted templates that define an environment's platform version, tier, configuration option settings, and tags. Create a saved configuration in Team A's account and download it to your local machine. Make the account-specific parameter changes and upload to the S3 bucket in Team B's account. From Elastic Beanstalk console, create an application from 'Saved Configurations'

You can deploy any version of your application to any environment. Environments can be long-running or temporary. When you terminate an environment, **you can save its configuration** to recreate it later.

`.ebextensions/<mysettings>.config` : You can add AWS Elastic Beanstalk configuration files (`.ebextensions`) to your web application's source code to configure your environment and customize the AWS resources that it contains. This **does not help with managing versions**. `.ebextensions` must be placed at the **root** of the source code, `option_settings` section of a configuration file defines values for configuration options (Elastic Beanstalk environment, the AWS resources in it, and the software that runs your application).

Any resources created as part of your `.ebextensions` is part of your Elastic Beanstalk template and **will get deleted if the environment is terminated**

Configure HTTPS for an ALB via `.ebextensions` -> To update your AWS Elastic Beanstalk environment to use HTTPS, you need to configure an HTTPS listener for the load balancer in your environment. Two types of load balancers support an HTTPS listener: Classic Load Balancer and Application Load Balancer. ALB to backend use HTTP.

**lifecycle policy** for versions:

![lifecycle](lifecycle.jpg)

**Failed deployment**: Elastic Beanstalk will replace the failed instances with instances running the application version from the most recent successful deployment

**All at once**: **Quickest deployment method. Short loss of service**. Deploys the new application version to each instance. Then, the web proxy or application server might need to restart. As a result, your application might be unavailable to users (or have low availability) for a short time.

**Immutable deployments**: Longest deployment. Perform an immutable update to launch a full set of **new instances** running the new version of the application in a **new Auto Scaling group**, alongside the instances running the old version. Immutable deployments can prevent issues caused by partially completed rolling deployments. **Quick and safe rollback** in case the deployment fails. With this method, Elastic Beanstalk performs an immutable update to deploy your application. In an immutable update, a second ASG is launched in **your environment** and the new version serves traffic alongside the old version until the new instances pass health checks.

**Traffic-splitting deployments** let you perform canary testing as part of your application deployment. In a traffic-splitting deployment, Elastic Beanstalk launches a full set of new instances just like during an immutable deployment. It then forwards a specified percentage of incoming client traffic to the new application version for a specified evaluation period.

**Rolling deployments**, Elastic Beanstalk splits the environment's Amazon EC2 instances into batches and deploys the new version of the application to one batch at a time, **instances are the same**.

**Rolling with additional batch**: **Slower than rolling**. Launches an extra batch of instances, then performs a rolling deployment (**updates the existing instances**). Launching the extra batch **takes time**, and ensures that the same **bandwidth is retained throughout the deployment**. This policy also **avoids any reduced availability**, although at a cost of an even **longer deployment** time compared to the Rolling method. Finally, this option is suitable if you must maintain the same bandwidth throughout the deployment

**Blue/Green**: Create a new “stage” environment and deploy updates there. DNS changes

![deployments.jpg](deployments.jpg)

![beanstalk.jpg](beanstalk.jpg)

![worker.jpg](worker.jpg)

## Cloudwatch Logs
The **CloudWatch agent** enables you to do the following:
- Collect system-level metrics from on-premises servers. These can include servers in a hybrid environment as well as servers not managed by AWS.
- Collect logs from Amazon EC2 instances and on-premises servers, running either Linux or Windows Server.
- To enable the CloudWatch agent to send data from an on-premises server, you must specify the access key and secret key of the IAM user that you created earlier.

You can export from multiple **log groups** or multiple **time ranges** to **s3 bucket**

Log group data is **always encrypted** in CloudWatch Logs. You can optionally use AWS AWS Key Management Service for this encryption. If you do, the encryption is done using an AWS KMS (AWS KMS) customer master key (CMK). Encryption using AWS KMS is enabled at the log group level, by associating a CMK with a log group, either when you create the log group or after it exists.

![log](log.jpg)

**Metric filters** define the terms and patterns to look for in log data as it is sent to **CloudWatch Logs**. CloudWatch Logs uses these metric filters to turn log data into numerical CloudWatch metrics that you can graph or **set an alarm on**. Filters do not retroactively filter data. Filters only publish the metric data points for **events that happen after the filter was created**. Filtered results return the first 50 lines, which will not be displayed if the timestamp on the filtered results is earlier than the metric creation time.

You can create **cross-account cross-Region dashboards**, which summarize your CloudWatch data from multiple AWS accounts and multiple Regions into one dashboard. Dashboards can be created from the console or programmatically.

## Cloudwatch metrics
You can configure custom metrics:
- Standard resolution (default), with data having a one-minute granularity
- High resolution, with data at a granularity of one second
  
Monitoring:
- Basic monitoring
- Detailed monitoring: provides more frequent metrics (but will not collect custom data), published at **one-minute intervals**, instead of the **five-minute intervals** used in Amazon EC2 basic monitoring. Detailed monitoring is offered by only some services.
- High resolution `PutMetric` API call with `StorageResolution` param. `GetMetricStatistics` specify 1, 5, 10, 30 or any multiple of 60 for **high-resolution**. Multiple of 60 for **standard-resolution**

Metric data is kept for 15 months, enabling you to view both up-to-the-minute data and historical data.
Data points with a period of less than 60 seconds are available for 3 hours. These data points are high-resolution custom metrics. Data points with a period of 60 seconds (1 minute) are available for 15 days Data points with a period of 300 seconds (5 minute) are available for 63 days Data points with a period of 3600 seconds (1 hour) are available for 455 days (15 months)

Display metrics from several apps in a single dashboard: A `namespace` is a container for CloudWatch metrics. Metrics in different namespaces are isolated from each other, so that metrics from different applications are not mistakenly aggregated into the same statistics.

## CloudWatch Alarms
If you set an alarm on a **high-resolution metric**, you can specify a **high-resolution alarm** with a period of **10 seconds or 30 seconds**, or you can set a regular alarm with a period of any multiple of 60 seconds

A metric alarm has the following possible states:
- OK – The metric or expression is within the defined threshold.
- ALARM – The metric or expression is outside of the defined threshold.
- INSUFFICIENT_DATA – The alarm has just started, the metric is not available, or not enough data is available for the metric to determine the alarm state.

An alarm watches a single metric over a specified time, and performs one or more specified actions, based on the value of the metric relative to a threshold over time. The action is a notification sent to an Amazon SNS topic or an Auto Scaling policy. You can also add alarms to dashboards.

## Lambda
Event source mapping need lambda role permissions to invoke the source services:
- SQS
- DynamoDB
- Kinesis

The **default account limit of 1,000 concurrent executions** will mean you can only assign up to 900 executions to the function (100 must be left unreserved)

`context.awsRequestId` -> fetch the unique identifier associated with each invocation.

`/tmp` is 512MB of temporary space

ENV variables can have a maximum size of **4 KB**

`Alias` can only point to function version, not another alias.

VPC: When you connect a function to a VPC, Lambda creates an **elastic network interface** for each combination of the security group and subnet in your function's VPC configuration.

Lambda does not support functions that use **multi-architecture container images**.

Image max size: 10 GB

To increase **provisioned concurrency** automatically as needed, use the Application Auto Scaling API to register a target and create a scaling policy. By allocating provisioned concurrency before an increase in invocations, you can ensure that all requests are served by **initialized instances** with very low latency. Provisioned Concurrency **cannot be used with the $LATEST version**. This feature can only be used with **published versions and aliases** of a function.

Lambda always encrypts environment variables at rest.

**Key Configuration**: On a per-function basis, you can configure Lambda to use an encryption key that you create and manage in AWS Key Management Service. These are referred to as customer managed customer master keys (CMKs) or customer managed keys. If you don't configure a customer managed key, Lambda uses an AWS managed CMK named aws/lambda, which Lambda creates in your account

**Encryption helpers**: The Lambda console lets you encrypt environment variable values client side, before sending them to Lambda. This enhances security further by preventing secrets from being displayed unencrypted in the Lambda console, or in function configuration that's returned by the Lambda API. The console also provides sample code that you can adapt to decrypt the values in your function handler.

![lambda-encryption](lambda-encryption.webp)
## Step functions
A Task state (`"Type": "Task"`) represents a single unit of work performed by a state machine.
`Resource` field is a required parameter for `Task` state.

`"Type": "Wait"` delays the state machine from continuing for a specified time

`"Type": "Fail"` stops the execution of the state machine and marks it as a failure unless it is caught by a Catch block

**Express Workflows** support event rates of more than 100,000 per second.

**Standard Workflows** are more suitable for long-running, durable, and auditable workflows where repeating workflow steps is expensive, support human approval steps.

Express Workflows have a maximum duration of five minutes and Standard workflows have a maximum duration of one year.

In the Amazon States Language, these fields filter and control the flow of JSON from state to state:
- InputPath: You can use InputPath to select **a portion of the state input**.
- OutputPath: OutputPath enables you to select a portion of the state output to pass to the next state. This enables you to filter out unwanted information and pass only the portion of JSON that you care about.
- ResultPath: Use ResultPath to **combine a task result with task input**, or to select one of these. The path you provide to ResultPath controls what information passes to the output. Use ResultPath in a Catch to include the error with the original input, instead of replacing it. 
- Parameters
- ResultSelector



## API Gateway
A **stage** is a named reference to a deployment, which is a snapshot of the API. You use a stage to manage and optimize a particular deployment. For example, you can configure stage settings to enable caching, customize request throttling, configure logging, define stage variables, or attach a canary release for testing.

**Promote a stage**: The promotion can be done by redeploying the API to the prod stage OR updating a stage variable value from the stage name of test to that of prod.

**Lambda authorizer**: Lambda authorizer uses bearer token authentication strategies, such as OAuth or SAML. You must first create the AWS Lambda function that implements the logic to authorize and, if necessary, to authenticate the caller.

![lambda-auth](lambdas.jpg)

The default TTL value for API caching is 300 seconds. The maximum TTL value is 3600 seconds. TTL=0 means caching is disabled.

API Gateway calls **AWS Security Token Service** (STS) to assume the IAM role, so make sure that AWS STS is enabled for the Region.

Access logging -> Only $context variables are supported (not $input, and so on).

Your account is charged for accessing method-level CloudWatch metrics, but not the API-level or stage-level metrics.

**Latency** metric measures the time between when API Gateway receives a request from a client and when it returns a response to the client. The latency includes the **integration latency** and other API Gateway overhead.

**API Keys and Usage Plans**

You can use API keys together with usage plans or Lambda authorizers to control access to your APIs. API Gateway can generate API keys on your behalf, or you can import them from a CSV file. You can generate an API key in API Gateway, or import it into API Gateway from an external source.

To associate the newly created key with a usage plan the CreatUsagePlanKey API can be called. This creates a usage plan key for adding an existing API key to a usage plan.

![usage-plans](usage-plans.jpeg)

There are two types of API logging in CloudWatch: **execution logging** and **access logging**.

API Gateway **doesn't support** direct installation of **third-party SSL/TLS certificates**

In Lambda **proxy integration**, when a client submits an API request, API Gateway passes to the integrated Lambda function the raw request as-is, except that the order of the request parameters is not preserved. This request data includes the request headers, query string parameters, URL path variables, payload, and API configuration data.
## Cognito
With **adaptive authentication**, you can configure your user pool to block suspicious sign-ins or add second factor authentication in response to an increased risk level.

For each sign-in attempt, Amazon Cognito generates a risk score for how likely the sign-in request is to be from a compromised source. This risk score is based on many factors, including whether it detects a new device, user location, or IP address.

You can add a **custom logo** or customize the **CSS** for the Cognito hosted web UI.

SAML-based identity federation is used with directory sources such as Microsoft Active Directory, not Facebook or Google.

With **developer authenticated identities**, you can register and authenticate users via your own existing authentication process, while still using Amazon Cognito to synchronize user data and access AWS resources.

Using **developer authenticated identities** involves interaction between the end user device, your backend for authentication, and Amazon Cognito.

Amazon Cognito Identity Pools can **support unauthenticated identities** by providing a unique identifier and AWS credentials for users who do not authenticate with an identity provider
## DynamoDB
TTL deletes are available at no additional cost.

DynamoDB streams -> item level log for up to 24 hours.

By default, the `Scan` operation processes data sequentially. Amazon DynamoDB returns data to the application in 1 MB increments, and an application performs additional Scan operations to retrieve the next 1 MB of data.
- Use **parallel scans** for faster scans. For table size > 20 GB. 
- Reduce `page-size` -> uses fewer read operations and creates a "pause" between each request.
- The `Limit` parameter can be used to reduce the `page-size`. The Scan operation provides a Limit parameter that you can use to set the page size for your request. Each Query or Scan request that has a smaller page size uses fewer read operations and creates a **"pause"** between each request.

2 backup methods: on-demand and PITR, they copy the table to s3 but you do NOT have access to the s3 bucket.

DynamoDB uses **eventually consistent reads** by default. Read operations (such as GetItem, Query, and Scan) provide a `ConsistentRead` parameter to read the most recent value.

- `UpdateItem` - Edits an existing item's attributes, or adds a new item to the table if it does not already exist. You can put, delete, or add attribute values.
- `GetItem` - Returns a set of attributes for the item with the given primary key.
- `PutItem` - Creates a new item, or replaces an old item with a new item. If an item that has the same primary key as the new item already exists in the specified table, the new item completely replaces the existing item.
- `DescribeTable` - Return information about the table such as the current status of the table, when it was created, the primary key schema, and any indexes on the table.

**Transactions**: 
- `TransactWriteItems`: idempotent, groups up to 25 write actions (distinct items) in a single all-or-nothing operation. The aggregate size of the items in the transaction cannot exceed **4 MB**. Consume 2 WCU per request of 1 KB.
- `TransactGetItems`

With a `BatchWriteItem` operation, it is possible that **only some of the actions** in the batch succeed while the others do not

**adaptive capacity**:
![adaptive capacity](adaptive.jpg)

![gsi](gsi.jpg)

If your application doesn't require strongly consistent reads, consider using **eventually consistent reads**. Eventually consistent reads are cheaper and are **less likely to experience high latency**.
**DynamoDB Global Tables**: you can specify the AWS Regions where you want the table to be available. This can significantly reduce latency for your users. So, reducing the distance between the client and the DynamoDB endpoint is an important performance fix to be considered.

Amazon DynamoDB **Encryption Client**. This client-side encryption library enables you to protect your table data before submitting it to DynamoDB. With *server-side encryption*, your data is encrypted in transit over an HTTPS connection, decrypted at the DynamoDB endpoint, and then re-encrypted before being stored in DynamoDB. *Client-side encryption* provides end-to-end protection for your data from its source to storage in DynamoDB.

The `BatchGetItem` operation in DynamoDB permits the retrieval of multiple items from **one or more tables** in a single operation, thereby minimizing network traffic and improving overall application performance.
## ElastiCache
All the nodes in a Redis cluster must reside in the same region

While using Redis with cluster mode enabled, there are some limitations:

1. You cannot manually promote any of the replica nodes to primary.
2. Multi-AZ is required.
3. You can only change the structure of a cluster, the node type, and the number of nodes by restoring from a backup.

When you add a read replica to a cluster, all of the data from the primary is copied to the new node. From that point on, whenever data is written to the primary, the changes are **asynchronously propagated** to all the read replicas, for both the **Redis** offerings (cluster mode enabled or cluster mode disabled)

Redis-compatible in-memory data structure service that can be used as a data store or cache. In addition to strings, **Redis** supports lists, sets, sorted sets, hashes, bit arrays, and hyperlog logs. Applications can use these more advanced data structures to support a variety of use cases. For example, you can use Redis Sorted Sets to easily implement a game leaderboard that keeps a list of players sorted by their rank.

**Cluster Mode** also allows for more flexibility when designing new workloads with unknown storage requirements or heavy write activity. In a read-heavy workload, one can scale a single shard by adding read replicas, up to five, but a write-heavy workload can benefit from additional write endpoints when cluster mode is enabled

![lazy](lazy.jpg)

![write](write.jpg)

![redis](redis.webp)

## RDS
IAM authorization: Available for **MySQL, PostGres and MariaDB**

You can enable **storage autoscaling** and set the max storage limit, triggers:
- Free available space is less than 10 percent of the allocated storage.
- The low-storage condition lasts at least five minutes.
- At least six hours have passed since the last storage modification.

Automated backups (0-35 days retention) are Region bound while manual snapshots and Read Replicas are supported across multiple Regions.

Aurora MySQL DB -> `max_connections` up to 16000

Amazon RDS can be used with *managed rotation*. When you rotate a secret, you update the credentials in both the secret and the database or service. Some services offer managed rotation, where the service configures and manages rotation for you.
## EC2
Query the metadata at http://169.254.169.254/latest/meta-data. local IP address

Query the user data at http://169.254.169.254/latest/user-data - This address retrieves the user data that you specified when launching your instance.

Change `DeleteOnTermination` default=True for the root volume, and default=False for the rest of the volumes:
- When the instance is running **can only be changed via CLI**
- From the console can only be set when you launch a new instance 

A **Zonal Reserved Instance** provides a capacity reservation in the specified Availability Zone. Capacity Reservations enable you to reserve capacity for your Amazon EC2 instances in a specific Availability Zone for any duration

**Regional Reserved Instance** does not provide capacity reservation.

**Detailed Monitoring** you define the frequency at which the metric data has to be sent to CloudWatch, from 5 minutes to 1-minute frequency window.

List of **ephemeral port ranges**:
- Many Linux kernels (including the Amazon Linux kernel) use ports 32768-61000.
- Requests originating from Elastic Load Balancing use ports 1024-65535.
- Windows operating systems through Windows Server 2003 use ports 1025-5000.
- Windows Server 2008 and later versions use ports 49152-65535.
- A NAT gateway uses ports 1024-65535.
- AWS Lambda functions use ports 1024-65535.

Default credential provider chain SDK for Java:
1. Env vars
2. Java system properties
3. ~/.aws/credentials file
4. Amazon ECS container credentials: AWS_CONTAINER_CREDENTIALS_RELATIVE_URI var
5. Instance profile credentials
## Cloudformation
`Fn::FindInMap` returns the value corresponding to keys in a two-level map. Map of all the possible values for the base AMI: `!FindInMap [ MapName, TopLevelKey, SecondLevelKey ]`

`Conditions` can only be associated with `Resources` or `Output` so that Cloudformation only creates the resource or output if the condition is True.

**Parameters** Valid types:
```String – A literal string
Number – An integer or float
List<Number> – An array of integers or floats
CommaDelimitedList – An array of literal strings that are separated by commas
AWS::EC2::KeyPair::KeyName – An Amazon EC2 key pair name
AWS::EC2::SecurityGroup::Id – A security group ID
AWS::EC2::Subnet::Id – A subnet ID
AWS::EC2::VPC::Id – A VPC ID
List<AWS::EC2::VPC::Id> – An array of VPC IDs
List<AWS::EC2::SecurityGroup::Id> – An array of security group IDs
List<AWS::EC2::Subnet::Id> – An array of subnet IDs
```

`AllowedValues` -> CommaDelimitedList

To share information between stacks, export a stack's output values. Other stacks that are in the **same** AWS account and region can import the exported values.

To export a stack's output value, use the Export field in the Output section of the stack's template. To import those values, use the **Fn::ImportValue** function in the template for the other stacks.

`cloudformation package` command packages the local artifacts (local paths) that your AWS CloudFormation template references. The command will upload local artifacts, such as your source code for your AWS Lambda function.

`cloudformation deploy` command deploys the specified AWS CloudFormation template by creating and then executing a changeset

You can upload all the code as a **zip to S3** and refer the object in `AWS::Lambda::Function` block. Or you can **write code directly in the template**, this is possible for simple functions using **Node.js or Python** which allow you to declare the code inline in the CloudFormation template.

Solve `DELETE_FAILED` state of a stack: To delete the stack you must choose to delete the stack in the console and then select to retain the resource(s) that failed to delete. This can also be achieved from the AWS CLI: `aws cloudformation delete-stack --stack-name my-stack --retain-resources <resource>`
## SAM

SAM supports the following resource types:
- AWS::Serverless::Api
- AWS::Serverless::Application
- AWS::Serverless::Function
- AWS::Serverless::HttpApi
- AWS::Serverless::LayerVersion
- AWS::Serverless::SimpleTable
- AWS::Serverless::StateMachine

![SAM](sam.jpg)

Test a Lambda locally with AWS SAM CLI -> `sam local invoke <function>`

## X-Ray
To ensure efficient tracing and provide a representative sample of the requests that your application serves, the X-Ray SDK applies a sampling algorithm to determine which requests get traced. By default, the **X-Ray SDK records the first request each second, and five percent of any additional requests**. X-Ray sampling is enabled directly from the AWS console, hence your application code does not need to change.

You can use X-Ray to collect data across AWS Accounts. The **X-Ray agent can assume a role to publish data into an account different from the one in which it is running**. This enables you to publish data from various components of your application into a central account.

You can use X-Ray to track requests from applications or services spread across **multiple Regions**.

**Index** your XRay traces to search and filter: **Annotations** are simple key-value pairs with string, number, or Boolean values that are indexed for use with filter expressions. Use annotations to record data that you want to use to group traces in the console, or when calling the GetTraceSummaries API.

X-Ray indexes up to 50 annotations per trace.

**Metadata** are key-value pairs with values of **any type**, including objects and lists, but that is not indexed. Use metadata to record data you want to store in the trace but don't need to use for searching traces.

A **Segment** provides the resource's name, details about the request, and details about the work done.
**Segment document** conveys information about a segment to X-Ray. A segment document can be up to **64 kB**. Send **segment documents** directly to X-Ray by using the `PutTraceSegments` API. Search for segments with `GetTraceSummaries`.

In ECS deploy X-Ray daemon agent as a **sidecar container** and provide the correct **IAM task role** to the X-Ray container. `AWS_XRAY_DAEMON_ADDRESS` By default, the SDK uses 127.0.0.1:2000 for both trace data (UDP) and sampling (TCP).

![xray](xray.jpg)

## CloudTrail
S3 bucket owner also needs to be object owner to get the **object access logs**.

If the bucket owner is also the object owner, the bucket owner gets the object access logs. Otherwise, the bucket owner must get permissions, through the object ACL, for the same object API to get the same object-access API logs.

## Macie

![macie](macie.jpg)

## ECR
`$(aws ecr get-login --no-include-email)` The get-login command retrieves a token that is valid for a specified registry for 12 hours, and then it prints a docker login command with that authorization token.

`docker pull 1234567890.dkr.ecr.eu-west-1.amazonaws.com/demo:latest`

## Athena
- Athena is compatible with Hive-style partition formats.
- Athena can pull data directly from the S3 source.
- There is a cost associated with partitioning data. A higher number of partitions can also increase the overhead from retrieving and processing the partition metadata. Multiple smaller files can counter the benefit of using partitioning. If your data is heavily skewed to one partition value, and most queries use that value, then the overhead may wipe out the initial benefit

Handle Athena timeouts for **Hive** -> MSK REPAIR TABLE command to update metadata in the catalog

## AppConfig
AWS AppConfig will automatically encrypt data at rest using AWS owned keys and AWS Key Management Service (KMS). **This layer cannot be disabled or altered by the customer**. The customer can add a second layer of encryption protection that they can control and manage using customer managed keys.

## Amplify
AWS Amplify supports connecting branches from the production code environment that will clone a repository so that features can be developed, tested and rolled back, if necessary, without impacting the latest version of the published application.

## AppSync
AWS AppSync’s API Cache settings provides three options: None, Full request caching, and Per-resolver caching. The cache settings can be configured for full request caching to cache all requests and responses.