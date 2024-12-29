# Chapter 1 - In the beginning there was IAM

Freshly created AWS accounts are comparable to email accounts. When you create a new email account, you may log in with the email address and password. After logging in, you may perform any action on your emails; you may freely write emails, delete emails, block senders, forward emails, reply to emails, etc. You are the full owner of the email account, and there is a one-to-one relationship between you and your account. If anyone else has your email address and password, they may also log into your email account and do all of the things that you can do.

AWS accounts function similarly. A new AWS account has one access point: an email and password login. When you log in with that email and password, you are logging in as "Root". Anyone logged in as Root has full access over the account, just like an email account. They may freely create resources, destroy resources, and modify resources with no restrictions.

This is where the similarities between email and AWS end, however. While there's no world in which you would want to share access to a single email account with multiple people, you will almost always need to share access to a single AWS account with multiple people. So how is this handled? Do we share the Root credentials with all of the people on our team? We certainly *could* do this, but what if Trent just needs access to billing information, and Jennifer only needs to upload footage of river water levels to AWS simple storage service (S3)? Would it be responsible to also give them full permissions to create, delete, or update all other resources? Finally, what if one of the team members doesn't practice good security and accidentally leaks the Root credentials? As you can see, this isn't an ideal situation. Where then, does the solution lie?

This is where AWS Identity and Access Management (IAM) comes in. IAM is a service *within* an AWS account that provides tools for managing access *to* the AWS account. Using IAM, the Root user can create unique "accounts" for every member of the team, so that each team member has a different username and password to log into the AWS account. Let's instead refer to the individual "accounts" as "identities". So for each individual AWS Account there can be many distinct identities that may access it. The most basic type of identity is an IAM User. Each IAM User can be provisioned with its own unique username+password credentials. Additionally, we can give each User their own permissions, so Trent's User can be given access only to the billing details in the account, and Jennifer's User can be given access only to S3. If someone has the username+password combination for an IAM User, they are identified as the person who is allowed to access the User; in other words, if someone uses Jennifer's IAM User credentials to log into the AWS Account, they are assumed to be Jennifer and are allowed to log into the account.

Many problems arise under the IAM User model. Do we assign permissions for each new User? Who decides what permissions a new User gets? What if a bad actor gets ahold of IAM User credentials? As you can see, it is difficult to scale when permissions are managed on a per User basis, and security concerns around password hygiene remain. A second type of identity called IAM Roles were designed to address these concerns. IAM Roles are similar to Users in that developers can use them to gain access to an AWS Account. Unlike IAM Users, however, IAM Roles offload the work of identifying who may access them. Where IAM Users are content to identify and trust someone simply by the username and password that person submits, IAM Roles only trust people that meet a specific condition, called a trust relationship, which can be customized for each Role. The best example of this are Roles with a trust relationship to an Active Directory/Entra group. These kinds of Roles will allow people to login only if they can prove that they belong to the trusted AD/Entra group, which means that the person *first* has to be placed into the group, then they have to acquire a DOI GFE laptop, then they have to insert their PIV card, and then log into the laptop with their PIN. Only when all those conditions are met will the person (or trusted entity) be allowed to login to the account as the IAM Role. Furthermore, the session will expire after a certain amount of time.

Here is an outline of the differences between IAM Roles and IAM Users:

|IAM Users | IAM Roles |
| --- | --- |
|Trusts whoever has the correct username+password|Trusts whoever meets a specific condition (defined upon Role creation)|
|Credentials don't expire|Session credentials expire (time before expiration customizable up to 12 hours)|
|Should be limited to one person|Can be accessible by multiple people in a group, provided that group has a trust relationship to the Role|

In USGS, we use the IAM Role feature of AWS to provision access to AWS Accounts. This allows us to define a couple of general use IAM Roles with varying levels of permissions, which can each be used by many people to access a single AWS Account. The specific IAM Roles CHS stages in each AWS Account are developers-console-access, developers-iac-restricted, and readonly. Here's a basic outline of what each role allows when a developer assumes them:

|developers-console-access|developers-iac-restricted|readonly|
|---|---|---|
|Developers may perform the full set of actions CHS allows in the AWS Console GUI|Developers may perform a subset of the actions CHS allows in the AWS Console GUI, but may perform the full set of actions through CloudFormation|Developers may view all information available in the AWS Console GUI, but may not create, update, or delete resources|
|Available in dev accounts|Available in dev and prod accounts|Available in all accounts|

The responsibility of delegating access to an AWS Account is offloaded to the group that uses the AWS account. The Account's COUA can choose who has access to each Role by placing developers in their corresponding AD/Entra groups, which are unique to each AWS Account and were created by the COUA when we migrated to IAM Identity Center.

!!! note "Advanced: Permissions Sets vs IAM Roles"

    While I am using the term IAM Role, technically the resources defined in the table above are actually Identity Center Permissions Sets. In each AWS Account, an IAM Role is created for each permissions set defined for your account. So you are still logging into your AWS account with the IAM Role that mirrors the Permissions Set, but CHS only has to manage three Permissions Sets across all of the accounts in our Organization (Plus all of the custom permissions sets that customers have requested)

When a developer is given access to one or more AWS Accounts, they can log into the account via this portal:

[AWS Access Portal :octicons-package-dependents-16:](https://d-926773f86b.awsapps.com/start/#/?tab=accounts){.md-button target="_blank"}

## Summary

In CHS, developers are given access to an AWS Account by getting a COUA to place them in the AD/Entra groups associated with their accounts IAM Roles. There is a many to one relationship with the IAM Roles, meaning just a single IAM Role is sufficient to provide many developers access and there is no limit to how many people can simultaneously access the Account through that IAM Role. Permissions for what a developer is allowed to do in an AWS Account are defined on the Role they are accessing through. CHS has staged about three different Roles into each AWS Account, each with varying permissions levels. COUAs decide who gets access to each Role by linking specific AD/Entra groups to them and then only placing the developer into the groups that are linked to the Roles that he/she needs access to.

!!! tip

    If the few Roles that CHS stages in your AWS Account do not meet your needs. CHS has the ability to create custom Roles. You will need to submit a support ticket to the CHS Help Center detailing the permissions you would like on the Role, though the permissions can't exceed the [CHS Permissions Boundary](https://code.chs.usgs.gov/usgs-chs/CHS-IaC/baseline/StackSets/Custom-Account/All/-/blob/master/cloudformation/csr-iam-managedpolicies.yaml#L530){target="_blank"}, amd what you would like the Role to be called (should be as short as possible because there is a character limit).
