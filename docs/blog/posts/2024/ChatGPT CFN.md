---
date:
  created: 2024-12-29
authors:
  - ben
categories:
  - AI
  - Tips
---

# Using DOI ChatGPT to write CloudFormation templates

One of the more useful things you can leverage ChatGPT for is CloudFormation templates. It can be used to help you synthesize templates from descriptions of the resources that you require. It can check a template that you've already written for inconsistencies or bugs. Or it can just be used as a jumping off point to push past blank page syndrome. Regardless of your experience level with CloudFormation, you will be able to find some use for it.

In this post, I give details on how I use DOI ChatGPT to lighten my workload when writing CloudFormation templates.

<!-- more -->

## Setup

!!! warning

    I'll start this post with a disclaimer. Be careful deploying templates that haven't been reviewed by an experienced cloud engineer. It can be a costly mistake if you deploy a CloudFormation template that you don't fully understand. You have been warned!

[:material-chat: DOI ChatGPT](https://doichatgpt.doi.net/){.md-button target="_blank"}

Hit the button above to go to the DOI ChatGPT webpage. If you've never used DOI ChatGPT, the layout shouldn't be unfamiliar. The application is formatted like Slack or Teams, with different conversations saved on the left navigation window.

Select New chat and you will have the opportunity to add a system prompt as well as adjust the temperature of the conversation.

### System Prompt

I like to adjust the system prompt to include any contextual information that ChatGPT might need, so when I submit later prompts, I can bypass any explanations about what I want. For example, I can set up a specific chat to be used for checking CloudFormation templates with this System Prompt:

> You are an AWS expert who is highly technically proficient in all AWS services and how they relate to CloudFormation. You should treat each incoming prompt as a CloudFormation template, and your job is to provide a summary of what the CloudFormation template is supposed to deploy as well as point out any inconsistencies or errors in the CloudFormation template

With the above system prompt, I don't have to explain what I want from ChatGPT every time I submit a template, I merely need to paste the template in the prompt and it will perform the checks I requested. Please take my prompt and modify it to your needs.

Here is another system prompt you could use in a separate chat for creating starting templates from a natural language prompt:

> You are an AWS engineer whose job is to assist customers with AWS CloudFormation. Treat all incoming prompts as schematics for a CloudFormation template that you must generate. Here is a list of rules you must keep in mind:
>
>CloudFormation templates you generate must be as secure as possible.
>
>When asked to generate a cloudformation template, you must attach a permissions boundary to each IAM Role in the template. You do not need to create the permissions boundary policy because it already exists, and it is named csr-developer-permissions-boundary
>
>You should use CloudFormation best practices, such as avoiding hardcoded values and utilizing parameters when appropriate, using Pseudo parameters when possible, and providing helpful outputs when applicable.

!!! tip

    If you have an ongoing chat, but you would like to modify the system prompt for the chat, you will have to clear the chat history and start over by clicking the button that looks like three horizontal, slightly offset parallel lines near the cog symbol at the top of the chat.

### Temperature

I haven't experimented with this very much, but I would imagine that you would want to keep your answers on the precise side for technical questions. That's what I do, and it seems to work out well.

!!! note

    Be sure to name your chat by clicking the pencil icon in the navigation bar on the left. I have a "Check CFN" chat, and a "Create CFN" chat using the two prompts I shared in the last section. You can get even more organized by creating an "AWS" folder by clicking the "new folder" button near the "New chat" button, and then dragging your CFN chats into it.

## Example

Let's go through an extremely basic example of CloudFormation template generation using DOI ChatGPT and see what we get. I will be using my chat with the second example system prompt I gave above, and a temperature setting of 0.1. This is the conversation prompt I will use:

> I need an S3 bucket with the bucket name parameterized and its default value set to "bens-cool-website-bucket". I would also like a bucket policy attached to the bucket that enforces SSL traffic, and allows public read as this will be for a bucket website.

Here's the template it generated for me:

``` yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: CloudFormation template to create an S3 bucket for a static website with enforced SSL traffic and public read access.

Parameters:
  BucketName:
    Type: String
    Default: bens-cool-website-bucket
    Description: The name of the S3 bucket for the static website.

Resources:
  WebsiteS3Bucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Ref BucketName
      WebsiteConfiguration:
        IndexDocument: index.html
        ErrorDocument: error.html

  BucketPolicy:
    Type: AWS::S3::BucketPolicy
    Properties:
      Bucket: !Ref WebsiteS3Bucket
      PolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal: "*"
            Action: s3:GetObject
            Resource: !Sub "${WebsiteS3Bucket.Arn}/*"
            Condition:
              Bool:
                aws:SecureTransport: "true"

  S3BucketRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: S3BucketAccessRole
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: "ec2.amazonaws.com"
            Action: "sts:AssumeRole"
      Policies:
        - PolicyName: S3BucketAccessPolicy
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Effect: Allow
                Action:
                  - s3:PutObject
                  - s3:GetObject
                Resource: !Sub "${WebsiteS3Bucket.Arn}/*"
      PermissionsBoundary: arn:aws:iam::aws:policy/csr-developer-permissions-boundary

Outputs:
  BucketNameOutput:
    Description: The name of the created S3 bucket.
    Value: !Ref WebsiteS3Bucket
  BucketWebsiteURL:
    Description: The URL of the S3 bucket website.
    Value: !GetAtt WebsiteS3Bucket.WebsiteURL
```

If you know CloudFormation, you can see that it did a pretty good job. However, there's something strange that it added to the template, and I suspect it was because of the system prompt I defined. It created an IAM Role called S3BucketRole, when I never asked for one. I suspect this is because of the second rule I gave in my prompt:

> When asked to generate a cloudformation template, you must attach a permissions boundary to each IAM Role in the template. You do not need to create the permissions boundary policy because it already exists, and it is named csr-developer-permissions-boundary

Though it did a great job of attaching the correct permissions boundary to the Role, I didn't need an IAM Role to begin with. Additionally, this template would've failed to deploy because no account id is defined in the permissions boundary arn. I will try to fix the issue of superfluous IAM Role generation by adjusting the system prompt rule to:

>If the CloudFormation template contains an IAM Role, you must attach a permissions boundary to each IAM Role in the template. You do not need to create the permissions boundary policy because it already exists, and it is named csr-developer-permissions-boundary. If the CloudFormation template doesn't contain an IAM Role, ignore this rule.

And here are the results if I try again:

``` yaml

AWSTemplateFormatVersion: '2010-09-09'
Description: CloudFormation template to create an S3 bucket for a static website with enforced SSL traffic and public read access.

Parameters:
  BucketName:
    Type: String
    Default: bens-cool-website-bucket
    Description: The name of the S3 bucket for the website.

Resources:
  WebsiteS3Bucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Ref BucketName
      WebsiteConfiguration:
        IndexDocument: index.html
        ErrorDocument: error.html

  BucketPolicy:
    Type: AWS::S3::BucketPolicy
    Properties:
      Bucket: !Ref WebsiteS3Bucket
      PolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal: "*"
            Action: s3:GetObject
            Resource: !Sub "${WebsiteS3Bucket.Arn}/*"
            Condition:
              Bool:
                aws:SecureTransport: "true"

Outputs:
  BucketNameOutput:
    Description: The name of the S3 bucket created for the website.
    Value: !Ref WebsiteS3Bucket

```

Much better! To my eyes, this template is exactly what I was hoping for with the prompt I gave. It even took the context I gave for this being an S3 bucket website, and wrote an accurate description of the template. I could now copy this template and submit it for review in my account's Custom Infrastructure project, but since this is a demo for DOI ChatGPT, I won't go any further.

## Conclusion

Thank you for reading my post about how to use DOI ChatGPT for CloudFormation. I hope that you find it a useful tool as you gain skills with AWS, and that you enjoyed reading this post.

...
