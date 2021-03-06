# Using Elastic Beanstalk with Other AWS Services<a name="AWSHowTo"></a>

The topics in this chapter discusses the integration of Elastic Beanstalk with resources from other AWS services that are not managed by Elastic Beanstalk as part of your environment\.


+ [Architectural Overview](#AWSHowTo.architecture)
+ [Using Elastic Beanstalk with Amazon CloudFront](AWSHowTo.cloudfront.md)
+ [Logging Elastic Beanstalk API Calls with AWS CloudTrail](AWSHowTo.cloudtrail.md)
+ [Using Elastic Beanstalk with Amazon CloudWatch](AWSHowTo.cloudwatch.md)
+ [Using Elastic Beanstalk with Amazon CloudWatch Logs](AWSHowTo.cloudwatchlogs.md)
+ [Using Elastic Beanstalk with DynamoDB](AWSHowTo.dynamoDB.md)
+ [Using Elastic Beanstalk with Amazon ElastiCache](AWSHowTo.ElastiCache.md)
+ [Using Elastic Beanstalk with Amazon Elastic File System](services-efs.md)
+ [Using Elastic Beanstalk with AWS Identity and Access Management](AWSHowTo.iam.md)
+ [Using Elastic Beanstalk with Amazon Relational Database Service](AWSHowTo.RDS.md)
+ [Using Elastic Beanstalk with Amazon Simple Storage Service](AWSHowTo.S3.md)
+ [Using Elastic Beanstalk with Amazon Virtual Private Cloud](vpc.md)

## Architectural Overview<a name="AWSHowTo.architecture"></a>

The following diagram illustrates an example architecture of Elastic Beanstalk across multiple Availability Zones working with other AWS products such as Amazon CloudFront, Amazon Simple Storage Service \(Amazon S3\), and Amazon Relational Database Service \(Amazon RDS\)\.

![\[Elastic Beanstalk Architecture Diagram\]](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/images/aeb-architecture_crossaws2.png)

To plan for fault\-tolerance, it is advisable to have N\+1 Amazon EC2 instances and spread your instances across multiple Availability Zones\. In the unlikely case that one Availability Zone goes down, you will still have your other Amazon EC2 instances running in another Availability Zone\. You can adjust Auto Scaling to allow for a minimum number of instances as well as multiple Availability Zones\. For instructions on how to do this, see [Your AWS Elastic Beanstalk Environment's Auto Scaling Group](using-features.managing.as.md)\. For more information about building fault\-tolerant applications, go to [ Building Fault\-Tolerant Applications on AWS](http://media.amazonwebservices.com/AWS_Building_Fault_Tolerant_Applications.pdf)\. 

The following sections discuss in more detail integration with Amazon CloudFront, Amazon CloudWatch, Amazon DynamoDB Amazon ElastiCache, Amazon RDS, Amazon Route 53, Amazon Simple Storage Service, Amazon VPC , and IAM\.