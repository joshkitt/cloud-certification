

# Amazon Web Services - Certified Developer Associate

### Elastic Beanstalk

- Maximum of 1000 application versions
  - Use lifecycle policy to remove old versions
  - Can keep source bundle in S3 and not delete it with lifecycle setting

Uses CloudFormation - creates a stack for each environment

Environments
- Types
  - Web server environment
  - Worker environment - long-running tasks
- Can be named whatever you want

Can provision RDS with Elastic Beanstalk
- Better to de-couple RDS form EB so it doesn't get deleted with EB stack
- Can migrate from EB:
  - Take RDS snapshot
  - Enable deletion protection
  - Create new environment
  - Deploy blue/green and change to new environment
  - Terminate old environment
  - Delete stack
  
Deployment Modes
- All at Once
  - __Causes downtime__
  - Fastest
  - No additional cost
- Rolling - update a few instances at a time
  - Updates in batches called buckets
  - Can take a long time depending on bucket size
  - No additional cost
- Rolling with Additional Batches
  - Maintains capacity
  - Adds additional instances running the new version in batch / buckets
  - Temporary over-provision
  - __Additional cost__
  - Good for prod
- Immutable
  - Zero downtime
  - High cost
  - New code deployed to new, temporary ASG
  - New ASG is merged into existing ASG
  - Old instances are removed from existing ASG
  - Longest deployment
  - Quick rollback
  - Good for prod

Blue / Green Deployment
- Zero downtime
- Create new environment ("green")
- Test / validate new environment
- Direct a percentage of traffic to the new environment - Route 53 weighted policy
- Swap URLs when done
- Shut down old environment ("blue")

Elastic Beanstalk Extensions
- .ebextensions directory in zip file deployable
- .config files are used to customize Elastic Beanstalk (ex. logging.config)

CLI - "eb deploy", "eb terminate", etc.

Deployment is done via zip file
- Package dependencies with the source code to improve deployment performance / speed

cron.yml - configure scheduled tasks for a Worker Tier

HTTPS:
- Load certificate onto Load Balancer
- Configure security group rules
- Use ALB rule to redirect to HTTPS

## CICD
Continuous Integration, Continuous Delivery (deployment)

Steps:
- Code
- Build
- Test
- Deploy
- Provision

### AWS CodeCommit
- Version control
- Private Git repositories
- No size limits
- Fully managed
- Highly available
- Collaborate with team
- Code is in a central repository
- Security
  - SSH keys
  - HTTPS
  - Multi factor authentication (MFA)
  - AuthZ - IAM policies
  - Encryption
    - Repositories are automatically encrypted using KMS
    - HTTPS or SSH only
- Notifications
  - AWS SNS
  - AWS Lambda
  - AWS CloudWatch Event Rules -> publishes to SNS Topic
    - Pull request events
    - Commit events
    
### AWS CodeBuild
- Fully managed build service
- No servers to manage or provision
- No build queue
- Uses Docker to build (Java, Python, etc.)
  - Can use your own base Docker images
- Security
  - Integrates with KMS to encrypt artifacts
  - Integrates with IAM for build permissions
- __buildspec.yml__ file at root of code
  - Define environment variables
  - SSM Parameter Store
  - Phases
    - Install
    - Pre build
    - Build
    - Post build
  - Artifacts: artifacts to store in S3 (encrypted)
  - Cache: files and dependencies to cache
- Logs to S3 or CloudWatch
- CloudWatch alarms for notifications
- Can cache build in S3
- Artifacts are stored in S3

### AWS CodePipeline

__Stages can have multiple Action Groups__

