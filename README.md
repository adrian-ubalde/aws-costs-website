# AWS Costs Website

This repository contains the front-end code of the [AWS Costs website](https://awscosts.thesmartsuperlist.com/), a website that shows its users the current month-to-date balance of their AWS bill. (Note: For the remainder of this document, I will refer to the static front-end as "the website".)

The website is currently a work in progress. As such, it currently has a single, temporary homepage.

## Why build this website?

Often as AWS customers, we forget to consider how much money we are being charged. As a result, I have seen teams that have had to spend massive amounts of time and effort auditing and reducing their AWS usage retrospectively. If there were a quicker and easier way to check our bill than navigating through the AWS Console, I think we'd pay more attention to the financial costs of the decisions we make as we build our applications.

## How is the website built?

### AWS Amplify Console and AWS CloudFront

Used for its:
1. Static website hosting features.
2. Git-based continuous deployment features.

### AWS CloudFormation

The website is provisioned in Amplify Console using CloudFormation.

### AWS Route53

A Route53 hosted-zone is used for domain routing while the website's domain is registered with Amazon Registrar as of writing this document. 

### GitHub Actions

When a change is made to the CloudFormation template that provisions the website, it is automatically processed and submitted to the CloudFormation service via a GitHub Actions workflow.

## How are website content and assets deployed?

When changes are pushed to this repository, the continuous deployment mechanism offered by AWS Amplify Console is automatically triggered which builds, tests, and deploys those changes to the live website.

This feature has saved me a significant amount of time from having to write my own deployment pipeline from scratch.

## What other alternatives did I consider when building this website?

### AWS S3 and CloudFront via CloudFormation

This website hosting approach is simple and easy to understand but does not address a few key issues:
* The use of a CloudFront distribution requires an SSL certificate. A certificate managed by the CloudFormation `AWS::CertificateManager::Certificate` resource does not get validated automatically.
* This approach doesn't address the deployment of the website's files into S3. 
* This approach doesn't address cache invalidation in CloudFront.

### AWS Cloud Development Kit (CDK)

To host the website, I wanted to use an AWS CloudFront CDN distribution to enable HTTPS on the website and provide low-latency to the end-user. However, the use of a distribution requires an SSL certificate. To create an SSL certificate, I considered using the CDK for its [`DnsValidatedCertificate` construct](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-certificatemanager.DnsValidatedCertificate.html) which programatically validates the certificate. However, one downside of the construct is that it is still an experimental API at the time of writing this document.

To deploy static files, I considered using the CDKs [S3-deployment construct](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-s3-deployment-readme.html). It works as advertised and it's a simple solution that involves the use of a single Lambda function. However, it is still an experimental API at the time of writing this document.

### AWS Amplify CLI

I chose the Amplify Console over the Amplify CLI because I think a Continuous Deployment pipeline written declaratively is easier to understand than an imperative approach.

### AWS Amplify Console

Ultimately, I decided to use the Amplify Console because it:
* Offers end-user low-latency via AWS's CloudFront CDN.
* Enables HTTPS on your website. 
* Supports automated custom domain creation via AWS Route53. 

#### Cons of using Amplify Console

One downside of choosing the Amplify Console over the combination of AWS S3 plus AWS CloudFront is that I have reduced visibility into the CloudFront distribution behind the an Amplify Console hosted website. Similarly, I have reduced visibility into the S3 buckets behind an Amplify Console hosted website. So in choosing to use Amplify Console for website hosting, I'm sacrificing the flexibility of being able to customise the website's CloudFront distribution settings and S3 settings in order to reap the benefits that Amplify Console offers.

Also worth keeping in mind when using the Amplify Console is that AWS charges for:
- Each continuous build minute.
- Traffic-served.
- Storage/GB-stored. It is not clear how space can be cleaned up.
