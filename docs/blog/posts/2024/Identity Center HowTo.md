---
date:
  created: 2024-12-29
authors:
  - ben
categories:
  - IAM
  - Explained
---

# Identity Center explained

If you're not 100% familiar with the way that access is provisioned to CHS Accounts, it can be difficult to know what to ask when transitioning to Identity Center. In this post, I give a short explanation of the context surrounding that transition.

<!-- more -->

## Context

Identity Center exists to provide access to your AWS account. This access is represented by Permissions Sets, which are custom bundles of permissions defining the AWS actions that are permitted when accessing the AWS Account *through* the Permissions Set. If you navigate to the [USGS Identity Center access portal](https://d-926773f86b.awsapps.com/start/#/?tab=accounts), it will first check to see which Permissions Sets you are allowed to use, then you may select a Permissions Set to log into the AWS Account.

### Where do these Permissions Sets come from?

CHS has created three general use Permissions Sets, but you may request custom Permissions Sets if these do not fulfill your use cases (more information in the table below).

### How does Identity Center know which Permissions Sets you are allowed to use?

In USGS we use AD groups(1) ( <- See the annotation for more information). If you are the COUA in charge of your AWS Account, you will create AD groups, and then you will tell CHS which Permissions Set should be linked to each group. A developer can be given access to a Permissions Set by placing them in the associated AD group. Below is a table of the basic Permissions Sets that CHS deploys in each AWS Account
{ .annotate }

1. In reality, we use Entra, which is essentially just an alternative to Active Directory but with no support for nested groups. When you create an AD group with "AWS Identity Center" somewhere in the name, and set the "extensionAttribute2" to "O365", USGS Entra clones the group and tracks new user objects added to the group. It doesn't track any added SG groups because Entra doesn't support nested groups. Identity Center then uses the Entra group, and not the original AD group. The reason we use Entra instead of AD is to prepare for the full transition from AD to Entra in USGS, which will take place over the next few years.

|developers-console-access|developers-iac-restricted|readonly|
|---|---|---|
|Developers may perform the full set of actions CHS allows in the AWS Console GUI|Developers may perform a subset of the actions CHS allows in the AWS Console GUI, but may perform the full set of actions through CloudFormation|Developers may view all information available in the AWS Console GUI, but may not create, update, or delete resources|
|Available in dev accounts|Available in dev and prod accounts|Available in all accounts|

!!! note

    Even though there can be up to three available Permissions Sets per AWS account, it is best to think of them in two categories: ReadOnly access and Developer access.

    Therefore, you will often provide two AD groups. One for readonly access, and one for developers. Developers represents the maximum permissions allowed in an account.

    If you want something in between readonly and maximum permissions (developers), you can request a custom Permissions Set, but you will be required to tell CHS exactly what permissions you would like the Permissions Set to have. Then you can provide a new AD group to be linked to that Permissions Set.

## SOP

Click the button below to view the full SOP for transitioning to Identity Center in the CHS Help Center:

[:material-file-document: SOP](https://support.chs.usgs.gov/x/QwHVD){.md-button target="_blank"}

...
