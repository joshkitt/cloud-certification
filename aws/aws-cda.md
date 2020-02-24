Amazon Web Services - Certified Developer Associate

### Elastic Beanstalk
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


