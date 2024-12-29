# Chapter 2 - What do we do with this power?

Access to an AWS Account is a powerful and scary thing. AWS is a very advanced and comprehensive suite of development tools. Because of the flexibility of AWS, it is possible to configure resources in a way that compromises the security of USGS, or accrue a massive unexpected bill. You may have heard stories of startup companies accidentally writing an infinite loop into their AWS projects, accruing a bill of over $20,000 USD in a single evening. You may have also heard stories of data leaks involving the AWS storage service (S3), where the personal data of employees or customers was leaked because of poor S3 bucket configurations. Unfortunately, these are very real risks when using the cloud. If we don't understand exactly what we're doing in an AWS account we are at risk of becoming yet another cautionary tale.

To avoid security issues, CHS heavily utilizes the tools in IAM to restrict which actions developers are allowed to perform in AWS. CHS takes on a lot of the heavy lifting to secure the environment and keep it federally compliant. However, This has an unfortunate side-effect: The learning curve for developing in AWS is already steep, and with these restrictions the curve gets steeper. A misconception about CHS is that its purpose is to make cloud adoption easier. Unfortunately, the primary purpose of CHS is actually to make cloud adoption *compliant*, which often makes cloud adoption *more* complicated, not less.

## What are the Restrictions in CHS AWS Accounts?

### FedRAMP

The main restriction for using cloud software in a federal context is FedRAMP. In order to use any single piece of cloud software in the government, it must first be tested and found compliant in accordance to the FedRAMP standards. Each individual service under the AWS umbrella must be found compliant by FedRAMP standards.

[:material-check-all: FedRAMP Services in Scope](https://aws.amazon.com/compliance/services-in-scope/FedRAMP/){.md-button target="_blank"}

It is CHS' responsibility to ensure services in AWS that have not been approved by FedRAMP are not used, therefore, CHS uses IAM to block you from using them.

[Here is a full list of AWS Services that are permitted for use in the CHS permissions boundary](https://code.chs.usgs.gov/usgs-chs/CHS-IaC/baseline/StackSets/Custom-Account/All/-/blob/master/cloudformation/csr-iam-managedpolicies.yaml#L542){target="_blank"}.

!!! note

    Sometimes AWS services are approved as compliant with the FedRAMP standard, but have yet to be added to the CHS permissions boundary. In that case, you may [request that the service be reviewed and added to the permissions boundary](https://taskmgr.chs.usgs.gov/plugins/servlet/desk/portal/10/create/447){target="_blank"}.

### Security

Some actions associated with the services that *are* FedRAMP approved present a security risk to USGS if they are unmitigated. Currently there are three examples of these kinds of actions:

1. Sharing resources to other AWS accounts. When you have the ability to share resources from one AWS account to another, you are not allowed to share them with AWS accounts outside of the CHS organization.
2. The creation of high-risk resources with conditional allowances. Currently, there are three examples of resources that pose security risks and must be [reviewed by the CHS support team before deployment](https://code.chs.usgs.gov/CHS-Custom-Infrastructure/CHS/CHS/-/blob/development/README.md?ref_type=heads){target="_blank"}.
    1. Bucket policies in the S3 service.
    2. Users in the IAM service.
    3. Resource Shares in the RAM service.
3. Finally there are some actions that are denied outright with no conditional allowances.

## Best Practices

The last type of restriction in CHS-provisioned AWS Accounts is the most controversial. CHS has chosen to enforce some best practices. There are currently two examples of this.

### CloudFormation in Prod accounts

In production AWS accounts, resources may only be deployed through the CloudFormation service. Certain actions are allowed in the console, such as starting and stopping EC2 instances.

### S3 Buckets and SSL

S3 buckets must accept SSL encrypted traffic only as mandated by DOI. All new AWS Accounts have an automation in place to enforce this.
