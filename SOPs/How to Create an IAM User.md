
# How to create an IAM User

## Use Cases

IAM Users are pre-approved for the following use cases:

- S3 Access
- Athena/Tableau access
- Pangeo central S3 bucket access
- Globus S3 Access
- Cloud Backup Software

!!! note

    The reason these use cases are approved is because they require the ability to remain authenticated to an AWS account in order to function correctly. IAM Roles cannot be used in these situations, because Roles require PIV authentication and only allow temporary access before another session must be started, requiring another PIV login.

## SOP

1. Learn the basics of the [AWS CloudFormation service](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html){target="_blank"} and how to write CloudFormation templates. Our review and deployment process requires that the IAM User be formatted in a CloudFormation template.
2. Learn the basics of writing AWS IAM Users and IAM Policies. For examples, view the [CHS IAM CloudFormation template library](https://code.chs.usgs.gov/usgs-chs/chs-library/templates/cloudformation-templates/iam){target="_blank"}.
3. When writing the IAM User, you must ensure that the csr-IAM-User-Permissions-Boundary IAM policy is attached to the new User as a Permissions Boundary. All examples provided by CHS will include this by default. CHS might also require that the IAM User be limited in other ways. For example, you might be instructed to limit the permissions to a set of IP address to ensure that if someone outside of USGS found the User Access Key, they still wouldn't be able to perform any actions as the IAM User.

!!! note

    The IAM User Permissions boundary is very restrictive. It should be permissive enough to cover all pre-approved use cases, but is restrictive to the point that the IAM User will not be useful for use cases that fall outside of the pre-approved set.

4. Once you have a CloudFormation template containing your new IAM User, you must deploy it using a [GitLab Custom Infrastructure project](https://code.chs.usgs.gov/CHS-Custom-Infrastructure){target="_blank"}. If you don't see your AWS account listed in that Group, you might need to [request that a Custom Infrastructure project be created for your AWS account](https://taskmgr.chs.usgs.gov/plugins/servlet/desk/portal/8/create/457){target="_blank"}.
5. Once you have found the Custom Infrastructure project associate with your AWS account or you have had one created, you can follow the ReadMe in the Custom Infrastructure project, which will give you details for how to use it to deploy your CloudFormation template.

!!! note

    The CHS Support team will typically review your IAM User to make sure it is secure, but won't always check the syntax. If you aren't confidant that the CloudFormation template was written correctly, let CHS Support know so they can review the template for syntax flaws as well as security.

## Why you can't simply create an IAM User like normal

There are restrictions on the creation of IAM Users because of their nature as the tools that grant access and permissions to an AWS Account. It is important for CHS to ensure to the best of our ability that no bad actors have access to CHS-provisioned AWS Accounts. IAM Users are more restricted than IAM Roles, because they have no secondary authentication method; if you have an Access Key (username+password) for an IAM User you are assumed to be a trusted entity. In other words, anyone, anywhere in the world may log in as an IAM User if they manage to find its access keys-- They don't even need to know which account they're logging into.

Contrast this with the ADFS IAM Roles, where you first have to prove to AWS that you are trusted by and authenticated through USGS Active Directory (meaning you have signed into GFE with your PIV card AND your PIN), and only then will AWS generate a *temporary* Access Key for you to log in with.

!!! note

    This is why you can automated stuff like S3 backups with IAM Users, but not with IAM Roles.
