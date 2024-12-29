
# How to create an S3 Bucket Policy

## SOP

1. Learn the basics of the [AWS CloudFormation service](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html){target="_blank"} and how to write CloudFormation templates. Our review and deployment process requires that the S3 bucket policy be formatted in a CloudFormation template.
2. Learn the basics of writing AWS S3 bucket policies.

!!! note

    Unfortunately, we restrict the creation of bucket policies in every situation, meaning, you will not be able to perform rapid testing or experiment with writing bucket policies to learn how they work. In order to test your bucket policy, you will have to embed it in a CloudFormation template, and submit it for review by following the rest of the steps in this SOP. I recommend studying the [bucket policy examples CHS gives here](https://code.chs.usgs.gov/usgs-chs/chs-library/templates/cloudformation-templates/s3){target="_blank"} so you aren't completely winging it.

3. Once you have a CloudFormation template containing an S3 bucket policy approximating what you believe will fulfill your use case, you must deploy it using a [GitLab Custom Infrastructure project](https://code.chs.usgs.gov/CHS-Custom-Infrastructure){target="_blank"}. If you don't see your AWS account listed in that Group, you might need to [request that a Custom Infrastructure project be created for your AWS account](https://taskmgr.chs.usgs.gov/plugins/servlet/desk/portal/8/create/457){target="_blank"}.
4. Once you have found the Custom Infrastructure project associate with your AWS account or you have had one created, you can follow the ReadMe in the Custom Infrastructure project, which will give you details for how to use it to deploy your CloudFormation template.

!!! note

    The CHS Support team will typically review your bucket policy to make sure it is secure, but won't always check the syntax. If you aren't confidant that the CloudFormation template was written correctly, let CHS Support know so they can review the template for syntax flaws as well as policy security.

## Why you can't simply create an S3 bucket policy like normal

Bucket Policies are a restricted resource in CHS-provisioned AWS accounts. This is because bucket policies are considered a possible ingress/egress point between an AWS Account and the public internet, and it is the responsibility of CHS to ensure that this does not result in an insecure environment.

The scope of who has permission to access a given S3 bucket is defined in the bucket policy that is associated with it, therefore it is possible to grant anyone with an internet connection access to an S3 bucket by attaching a public bucket policy to it. This feature can be useful for sharing public data, but poses a severe security risk if anyone is careless about how they configure their bucket policies and how they store private data in S3. For this reason, CHS has made the creation of S3 bucket policies impossible without first submitting the bucket policy to CHS for review.
