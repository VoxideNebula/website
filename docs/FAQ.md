# FAQ

## Can I let external collaborators use my AWS Account?

The short answer is no. To access a CHS-provisioned AWS account, a person needs DOI internal network access via a PIV card and GFE computer. If you have the ability to give them these things then you will be able to give an external collaborator access.

However, you may be aware of IAM User credentials. IAM Users are an AWS feature that allow someone to authenticate to an AWS account without verifying who that person is. It is assumed that if someone holds the password for an IAM User (otherwise known as the Access Key), then that person is permitted to authenticate via the account, no questions asked. This might sound like the perfect avenue for granting external collaborators access to your account, however, because of the inherent insecurity of this model, we don't permit the use of IAM Users, except in rare circumstances which do *not* include external collaboration.

## Can I create an IAM User?

Yes, under rare circumstances. There are restrictions on the creation of IAM Resources because of their nature as the tools that grant access and permissions to an AWS Account. It is important for CHS to ensure to the best of our ability that no bad actors have access to CHS-provisioned AWS Accounts. IAM Users are more restricted than IAM Roles, because they have no secondary authentication method; if you have an Access Key for an IAM User you are assumed to be a trusted entity. Contrast this with the ADFS IAM Roles, where you first have to prove to AWS that you are trusted by and authenticated through USGS Active Directory (meaning you have signed in with your PIV card AND your PIN),and only then will AWS generate a *temporary* Access Key for you to log in with.

!!! note

    This is why you can automate stuff like S3 backups with IAM Users, but not with IAM Roles.

## Can I create an IAM Role?

Yes, but with some restrictions. You must attach the csr-developer-permissions-boundary IAM policy as a Permissions Boundary during Role creation, otherwise you will receive a permissions error. You may create IAM Roles in the AWS Console in Development accounts, and through CloudFormation in Production accounts.

## How do I transition to IAM Identity Center?

Check out [this Blog post](./blog/posts/2024/Identity%20Center%20HowTo.md) that I wrote on the topic.
