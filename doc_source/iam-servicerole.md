# Managing Elastic Beanstalk Service Roles<a name="iam-servicerole"></a>

When you launch an environment in the AWS Elastic Beanstalk environment management console, the console creates a default service role, called `aws-elasticbeanstalk-service-role`, and attaches managed policies with default permissions to it\. 

Elastic Beanstalk provides a managed policy for enhanced health monitoring, and one with additional permissions required for managed platform updates\. The console assigns both of these policies to the default service role\. The managed service role policies follow\.

**Managed Service Role Policies**

+ **AWSElasticBeanstalkEnhancedHealth** – Grants permissions for Elastic Beanstalk to monitor instance and environment health\.

  ```
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Action": [
          "elasticloadbalancing:DescribeInstanceHealth",
          "elasticloadbalancing:DescribeLoadBalancers",
          "elasticloadbalancing:DescribeTargetHealth",
          "ec2:DescribeInstances",
          "ec2:DescribeInstanceStatus",
          "ec2:GetConsoleOutput",
          "ec2:AssociateAddress",
          "ec2:DescribeAddresses",
          "ec2:DescribeSecurityGroups",
          "sqs:GetQueueAttributes",
          "sqs:GetQueueUrl",
          "autoscaling:DescribeAutoScalingGroups",
          "autoscaling:DescribeAutoScalingInstances",
          "autoscaling:DescribeScalingActivities",
          "autoscaling:DescribeNotificationConfigurations"
        ],
        "Resource": [
          "*"
        ]
      }
    ]
  }
  ```

+ **AWSElasticBeanstalkService** – Grants permissions for Elastic Beanstalk to update environments on your behalf to perform managed updates\.

  ```
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Sid": "AllowCloudformationOperationsOnElasticBeanstalkStacks",
              "Effect": "Allow",
              "Action": [
                  "cloudformation:*"
              ],
              "Resource": [
                  "arn:aws:cloudformation:*:*:stack/awseb-*",
                  "arn:aws:cloudformation:*:*:stack/eb-*"
              ]
          },
          {
              "Sid": "AllowS3OperationsOnElasticBeanstalkBuckets",
              "Effect": "Allow",
              "Action": [
                  "s3:*"
              ],
              "Resource": [
                  "arn:aws:s3:::elasticbeanstalk-*",
                  "arn:aws:s3:::elasticbeanstalk-*/*"
              ]
          },
          {
              "Sid": "AllowOperations",
              "Effect": "Allow",
              "Action": [
                  "autoscaling:AttachInstances",
                  "autoscaling:CreateAutoScalingGroup",
                  "autoscaling:CreateLaunchConfiguration",
                  "autoscaling:DeleteLaunchConfiguration",
                  "autoscaling:DeleteAutoScalingGroup",
                  "autoscaling:DeleteScheduledAction",
                  "autoscaling:DescribeAccountLimits",
                  "autoscaling:DescribeAutoScalingGroups",
                  "autoscaling:DescribeAutoScalingInstances",
                  "autoscaling:DescribeLaunchConfigurations",
                  "autoscaling:DescribeLoadBalancers",
                  "autoscaling:DescribeNotificationConfigurations",
                  "autoscaling:DescribeScalingActivities",
                  "autoscaling:DescribeScheduledActions",
                  "autoscaling:DetachInstances",
                  "autoscaling:PutScheduledUpdateGroupAction",
                  "autoscaling:ResumeProcesses",
                  "autoscaling:SetDesiredCapacity",
                  "autoscaling:SuspendProcesses",
                  "autoscaling:TerminateInstanceInAutoScalingGroup",
                  "autoscaling:UpdateAutoScalingGroup",
                  "cloudwatch:PutMetricAlarm",
                  "ec2:AuthorizeSecurityGroupEgress",
                  "ec2:AuthorizeSecurityGroupIngress",
                  "ec2:CreateSecurityGroup",
                  "ec2:DeleteSecurityGroup",
                  "ec2:DescribeAccountAttributes",
                  "ec2:DescribeImages",
                  "ec2:DescribeInstances",
                  "ec2:DescribeKeyPairs",
                  "ec2:DescribeSecurityGroups",
                  "ec2:DescribeSubnets",
                  "ec2:DescribeVpcs",
                  "ec2:RevokeSecurityGroupEgress",
                  "ec2:RevokeSecurityGroupIngress",
                  "ec2:TerminateInstances",
                  "ecs:CreateCluster",
                  "ecs:DeleteCluster",
                  "ecs:DescribeClusters",
                  "ecs:RegisterTaskDefinition",
                  "elasticbeanstalk:*",
                  "elasticloadbalancing:ApplySecurityGroupsToLoadBalancer",
                  "elasticloadbalancing:ConfigureHealthCheck",
                  "elasticloadbalancing:CreateLoadBalancer",
                  "elasticloadbalancing:DeleteLoadBalancer",
                  "elasticloadbalancing:DeregisterInstancesFromLoadBalancer",
                  "elasticloadbalancing:DescribeInstanceHealth",
                  "elasticloadbalancing:DescribeLoadBalancers",
                  "elasticloadbalancing:DescribeTargetHealth",
                  "elasticloadbalancing:RegisterInstancesWithLoadBalancer",
                  "iam:ListRoles",
                  "iam:PassRole",
                  "logs:CreateLogGroup",
                  "logs:PutRetentionPolicy",
                  "rds:DescribeDBInstances",
                  "rds:DescribeOrderableDBInstanceOptions",
                  "s3:CopyObject",
                  "s3:GetObject",
                  "s3:GetObjectAcl",
                  "s3:GetObjectMetadata",
                  "s3:ListBucket",
                  "s3:listBuckets",
                  "s3:ListObjects",
                  "sns:CreateTopic",
                  "sns:GetTopicAttributes",
                  "sns:ListSubscriptionsByTopic",
                  "sns:Subscribe",
                  "sqs:GetQueueAttributes",
                  "sqs:GetQueueUrl"
              ],
              "Resource": [
                  "*"
              ]
          }
      ]
  }
  ```

To allow Elastic Beanstalk to assume the `aws-elasticbeanstalk-service-role` role, the service role specifies Elastic Beanstalk as a trusted entity in the trust relationship policy:

```
{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Sid": "",
        "Effect": "Allow",
        "Principal": {
          "Service": "elasticbeanstalk.amazonaws.com"
        },
        "Action": "sts:AssumeRole",
        "Condition": {
          "StringEquals": {
            "sts:ExternalId": "elasticbeanstalk"
          }
        }
      }
    ]
}
```

When you launch an envitonment using the [[ERROR] BAD/MISSING LINK TEXT](eb3-create.md) command of the Elastic Beanstalk Command Line Interface \(EB CLI\) and don't specify a service role through the `--service-role` option, Elastic Beanstalk creates the default service role `aws-elasticbeanstalk-service-role`\. If the default service role already exists, Elastic Beanstalk uses it for the new environment\.

If you use the `CreateEnvironment` action of the Elastic Beanstalk API to create an environment, specify a service role with the `ServiceRole` configuration option in the `aws:elasticbeanstalk:environment` namespace\. See  for details on using enhanced health monitoring with the Elastic Beanstalk API\. 

When you create an environment by using the Elastic Beanstalk API, and don't specify a service role, Elastic Beanstalk creates a service\-linked role\. This is a unique type of service role that is predefined by Elastic Beanstalk to include all the permissions that the service requires to call other AWS services on your behalf\. The service\-linked role is associated with your account\. Elastic Beanstalk creates it once, then reuses it when creating additional environments\. You can also use IAM to create your account's service\-linked role in advance\. When your account has a service\-linked role, you can use it to create an environment by using the Elastic Beanstalk API, the Elastic Beanstalk console, or the EB CLI\. For details about using service\-linked roles with Elastic Beanstalk environments, see [Using Service\-Linked Roles for Elastic Beanstalk](using-service-linked-roles.md)\.


+ [Verifying the Default Service Role's Permissions](#iam-servicerole-verify)
+ [Updating an Out\-of\-Date Default Service Role](#iam-servicerole-update)
+ [Adding Permissions to the Default Service Role](#iam-servicerole-addperms)
+ [Creating a Service Role](#iam-servicerole-create)
+ [Using Service\-Linked Roles for Elastic Beanstalk](using-service-linked-roles.md)

## Verifying the Default Service Role's Permissions<a name="iam-servicerole-verify"></a>

The permissions granted by your default service role can vary depending on when it was created, the last time you launched an environment, and which client you used\. You can verify the permissions granted by the default service role in the IAM console\.

**To verify the default service role's permissions**

1. Open the [**Roles** page](https://console.aws.amazon.com/iam/home#roles) in the IAM console\.

1. Choose **aws\-elasticbeanstalk\-service\-role**\.

1. On the **Permissions** tab, in the **Managed Policies** and **Inline Policies** sections, review the list of policies attached to the role\.

1. To view the permissions that a policy grants, choose **Show Policy** next to the policy\.

## Updating an Out\-of\-Date Default Service Role<a name="iam-servicerole-update"></a>

If the default service role lacks the required permissions, you can update it by creating a new environment in the Elastic Beanstalk environment management console\.

Alternatively, you can add the managed policies to the default service role manually\.

**To add managed policies to the default service role**

1. Open the [**Roles** page](https://console.aws.amazon.com/iam/home#roles) in the IAM console\.

1. Choose **aws\-elasticbeanstalk\-service\-role**\.

1. On the **Permissions** tab, under **Managed Policies**, choose **Attach Policy**\.

1. Type **AWSElasticBeanstalk** to filter the policies\.

1. Select the following policies, and then choose **Attach Policies**:

   + `AWSElasticBeanstalkEnhancedHealth`

   + `AWSElasticBeanstalkService`

## Adding Permissions to the Default Service Role<a name="iam-servicerole-addperms"></a>

If your application includes configuration files that refer to AWS resources for which permissions aren't included in the default service role, Elastic Beanstalk might need additional permissions to resolve these references when it processes the configuration files during a managed update\. If permissions are missing, the update fails and Elastic Beanstalk returns a message indicating which permission it needs\. Add permissions for additional services to the default service role in the IAM console\.

**To add additional policies to the default service role**

1. Open the [**Roles** page](https://console.aws.amazon.com/iam/home#roles) in the IAM console\.

1. Choose **aws\-elasticbeanstalk\-service\-role**\.

1. On the **Permissions** tab, under **Managed Policies**, choose **Attach Policy**\.

1. Select the managed policy for the additional services that your application uses\. For example, `AmazonAPIGatewayAdministrator` or `AmazonElasticFileSystemFullAccess`\.

1. Choose **Attach Policies**\.

## Creating a Service Role<a name="iam-servicerole-create"></a>

If you can't use the default service role, create a service role\.

**To create a service role**

1. Open the [**Roles** page](https://console.aws.amazon.com/iam/home#roles) in the IAM console\.

1. Choose **Create New Role**\.

1. Type a name, and then choose **Next Step**\.

1. Under **AWS Service Roles**, choose **AWS Elastic Beanstalk**\.

1. Attach the `AWSElasticBeanstalkService` and `AWSElasticBeanstalkEnhancedHealth` managed policies and any additional policies that provide permissions that your application needs\.

1. Choose **Next Step**\.

1. Choose **Create Role**\.

You can apply your custom service role when you create an environment in the environment creation wizard or with the `--service-role` option on the `eb create` command\.