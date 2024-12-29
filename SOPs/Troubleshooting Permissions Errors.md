# Troubleshooting Permissions Errors

If you've received a permissions error in a CHS AWS Account, Don't Panic! This guide will help you understand why you received the permissions error and will help you figure out what to do next.

## Is the AWS Service you are trying to use in the CHS Permissions Boundary

Not all AWS Services are available in CHS. This is due to the fact that not all services are FedRAMP approved.

- [List of FedRAMP approved services](https://aws.amazon.com/compliance/services-in-scope/FedRAMP/){target="_blank"}
- [List of services supported in CHS AWS Accounts](https://support.chs.usgs.gov/x/3oCTB){target="_blank"}

If the service is FedRAMP approved, but not yet supported in CHS AWS Accounts, you may [submit a request to add the service to the CHS boundary](https://taskmgr.chs.usgs.gov/plugins/servlet/desk/portal/9/create/466){target="_blank"}. Just remember to include details on the service you wish to use.

## Unable to deploy Resources in a Production Account

Make sure that you are deploying all resources in your Production account through the CloudFormation service, and you are passing the csr-Cloudformation IAM Role when creating the CloudFormation stack.

## Unable to deploy a Bucket Policy

Bucket Policies are a restricted resource. If you wish to create one you will have to follow a specific set of steps. More information can be found in the [How to Create an S3 Bucket Policy SOP](./SOPs/How%20to%20Create%20an%20S3%20Bucket%20Policy.md){target="_blank"}

## Unable to deploy IAM Resources

### Unable to create an IAM User

IAM Users are highly restricted resources. If you wish to create one you will have to follow a specific set of steps. More information can be found in the [How to Create an IAM User SOP](./SOPs/How%20to%20Create%20an%20IAM%20User.md){target="_blank"}

### Unable to create an IAM Role

#### Solution - Development Account

If you are operating in a Development account, make sure that you attach the csr-developer-permissions-boundary IAM Policy as a permissions boundary on the IAM Role you are attempting to create.

#### Solution - Production Account

If you are operating in a Production account, make sure that you are deploying the IAM Role through CloudFormation, and you have attached the csr-developer-permissions-boundary IAM Policy as a permissions boundary.

### Service Linked Role error

When trying to deploy resources in AWS, you may receive an error similar to the following:

!!! warning

    User: arn:aws::112233445566:assumed-role/adfs-developers/useremail@usgs.gov is not authorized to perform: iam:CreateRole on resource: arn:aws::629392778235:role/service-role/AWSBackupDefaultServiceRole because no identity-based policy allows the iam:CreateRole action.

This is because the service you are using is attempting to create a service-linked role. What does this mean? It means that whatever resource you are trying to create needs access to some other resource on AWS, therefore AWS attempted to automatically create an IAM Role for it in the background. However, AWS does not have the ability to attach a permissions boundary to service-linked roles, so it was not permitted to create the role.

#### Solution

If the service allows you to specify your own IAM Role, then you can create your own version of the service-linked role but with the addition of the csr-developers-permissions-boundary. You may have to search the web to find the exact configuration of the service-linked role. In the example above, I found the AWSBackupDefaultServiceRole configuration in the [AWS documentation](https://docs.aws.amazon.com/aws-backup/latest/devguide/iam-service-roles.html#creating-default-service-role-console){target="_blank"}, then I recreated the Role as the documentation described and attached csr-developer-permissions-boundary as a Permissions Boundary. Finally, I was permitted to use that custom role instead of the default IAM Role to create a Backup Plan in the AWS Backup service.

If the service does not allow you to specify your own IAM Role, you may have to request that CHS make an exception for this specific service-linked role. You may [submit a ticket to CHS](https://taskmgr.chs.usgs.gov/plugins/servlet/desk/portal/9/create/466){target="_blank"}, making sure to include the text for your specific error and a request that CHS make an exception for a service-linked role.
