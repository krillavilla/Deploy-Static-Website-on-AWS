# Deploy Static Website on AWS
### Project Overview

The cloud is perfect for hosting static websites that only include HTML, CSS, and JavaScript files that require no server-side processing. The whole project has two major intentions to implement:

* Hosting a static website on S3 and
* Accessing the cached website pages using CloudFront content delivery network (CDN) service. Recall that CloudFront offers low latency and high transfer speeds during website rendering.

_Note that Static website hosting essentially requires a public bucket, whereas the CloudFront can work with public and private buckets._

In this project, you will deploy a static website to AWS by performing the following steps:

1. You will create a public S3 bucket and upload the website files to your bucket.
2. You will configure the bucket for website hosting and secure it using IAM policies.
3. You will speed up content delivery using AWS’s content distribution network service, CloudFront.
4. You will access your website in a browser using the unique CloudFront endpoint.

### Prerequisites:
* AWS Account
* [Student-ready starter code](https://drive.google.com/open?id=15vQ7-utH7wBJzdAX3eDmO9ls35J5_sEQ) - Download and unzip this file.

### Topics Covered:
* S3 bucket creation
* S3 bucket configuration
* Website distribution via CloudFront
* Access website via web browser

### Dependencies
______________________________________________________________________________________

#### Cloud Services

* Amazon Web Services S3 - Resource hosting and deployments
* AWS CloudFront - CDN for SPA
* AWS EKS - Backend API
* AWS DynamoDB - Persistent Datastore
* AWS Cognito - User Authentication

#### Performance Tracking and DevOps Tools:

* AWS CloudWatch - Performance and Health checks
* Sentry - Bug Reporting:

  * Alternates:
  * TBD
  

* Google Analytics - Usage Reporting:
  * Alternates:
  * Mixpanel
  

* Travis (CI/CD)


#### Frameworks:
* Ionic (Javascript) (Frontend)
* Node.js (Javascript) (Backend)



### Section 1: Create S3 Bucket

1. Navigate to the “AWS Management Console” page, type “S3” in the “Find Services” box and then select “S3”.

![img.png](img.png)


2. The Amazon S3 dashboard displays. Click “Create bucket”.

![Create a bucket](img_1.png)

3. In the **General configuration**, enter a “Bucket name” and a region of your choice. Note: Bucket names must be globally unique.

![img_2.png](img_2.png)
_**One of the convenient naming conventions is my-123456789-bucket, where you can replace 123456789 with your 12 digit AWS account ID._**

4. In the **Bucket settings for Block Public Access section**, uncheck the “Block all public access”. It will enable the public access to the bucket objects via the S3 object URL.

_**Note** - We are allowing the public access to the bucket contents because we are going to host a static website in this bucket. **Hosting requires the content should be publicly readable**._

![img_3.png](img_3.png)

5. Click “Next” and click “Create bucket”.


6. Once the bucket is created, click on the name of the bucket to open the bucket to the contents.

![img_4.png](img_4.png)
_Bucket my-**014421265158**-bucket configuration and content_


### Section 2: Upload files to S3 Bucket

1. Once the bucket is open to its contents, click the “Upload” button.

![img_5.png](img_5.png)
Click on the **Upload** button

2. Click the "Add files" and “Add folder” button, and upload the [Student-ready starter code](https://drive.google.com/open?id=15vQ7-utH7wBJzdAX3eDmO9ls35J5_sEQ) folder content from your local computer to the S3 bucket.

![img_6.png](img_6.png)
Click "Add files" to upload the `index.html` file, and click "Add folder" to upload the `css`, `img`, and `vendor` folders.

![img_7.png](img_7.png)
Do not select the `udacity-starter-website` folder. Instead, upload its content one-by-one.

![img_8.png](img_8.png)
Successfully uploaded starter code in the bucket

### Need help with uploading the files to S3?

Sometimes the local machine's network setting or firewall might block, or the browser's Adblocker may prohibit the file upload, such as buysellads.svg file. In such a case, here are the workarounds:

* **Workaround 1**

Try using Chrome browser, and turn off the Adblocker, if not already. Here are the steps to turn off the Adblocker in Chrome:

1. At the top right, click More (three dots) >> Settings.
2. Click Security and Privacy >> Site Settings.
3. Click Additional content settings >> Ads.
4. Turn off Block ads on sites that show intrusive or misleading ads.

* **Workaround 2**

Use CLI commands to upload the files and folders:

1. Verify the AWS CLI configuration. If not configured already, use:
```
aws configure list
aws configure 
aws configure set aws_session_token "<TOKEN>" --profile default
```

2. Upload files

```
# Create a PUBLIC bucket in the S3, and verify locally as 
aws s3api list-buckets 
# Download and unzip the udacity-starter-website.zip 
cd udacity-starter-website 
# Assuming the bucket name is my-bucket-202203081 and your PWD is the "udacity-starter-website" folder 
# Put a single file. 
aws s3api put-object --bucket my-bucket-202203081 --key index.html --body index.html 
# Copy over folders from local to S3 
aws s3 cp vendor/ s3://my-bucket-202203081/vendor/ --recursive 
aws s3 cp css/ s3://my-bucket-202203081/css/ --recursive 
aws s3 cp img/ s3://my-bucket-202203081/img/ --recursive
```

### Section 4: Secure Bucket via IAM

1. Click on the “Permissions” tab.

![img_9.png](img_9.png)
Go to the **Permissions** tab. See that the bucket allows public access for hosting.

2. The “Bucket Policy” section shows it is empty. Click on the Edit button.

![img_10.png](img_10.png)
Empty bucket policy. Check this policy again after setting up the CloudFront distribution.

3. Enter the following bucket policy replacing `your-website` with the name of your bucket and click “Save”.

```
{
"Version":"2012-10-17",
"Statement":[
 {
   "Sid":"AddPerm",
   "Effect":"Allow",
   "Principal": "*",
   "Action":["s3:GetObject"],
   "Resource":["arn:aws:s3:::your-website/*"]
 }
]
}
```
You will see warnings about making your bucket public, but this step is required for static website hosting.

_**Note** - If we were not learning about static website hosting, we could have made the bucket private and wouldn't have to specify any bucket access policy explicitly. In such a case, once we set up the **CloudFront distribution**, it will automatically update the current bucket access policy to access the bucket content. The CloudFront service will make this happen by using the **Origin Access Identity user**._

### Section 5: Configure S3 Bucket

1. Go to the **Properties** tab and then scroll down to edit the **Static website hosting** section.
![img_11.png](img_11.png)
Go to the **Properties** tab

![img_12.png](img_12.png)
Edit the **Static website hosting** section

2. Click on the “Edit” button to see the **Edit static website hosting** screen. Now, enable the **Static website hosting**, and provide the default home page and error page for your website.

![img_13.png](img_13.png)
Enable the static website hosting, and provide the home page and error page.


_Did you notice that enabling the static website hosting requires you to make your bucket public?
In the snapshot above, it says "For your customers to access the content at the website endpoint, you must make all your content **publicly readable**._"

3. For both “Index document” and “Error document”, enter “index.html” and click “Save”. After successfully saving the settings, check the **Static website hosting** section again under the **Properties** tab. You must now be able to view the [website endpoint](https://docs.aws.amazon.com/AmazonS3/latest/dev/WebsiteEndpoints.html) as shown below:

![img_14.png](img_14.png)
Copy the website endpoint for future use.

### Section 6: Distribute Website via CloudFront

1. Select “Services” from the top left corner and enter “cloud front” in the “Find a service by name or feature” text box and select “CloudFront”.
![img_15.png](img_15.png)

2. From the CloudFront dashboard, click “Create Distribution”.
![img_16.png](img_16.png)

3. For “Select a delivery method for your content”, click “Get Started”.
![img_17.png](img_17.png)


4. Use the following details to create a distribution:

|     |     |
| --- | --- |
| **Field** | **Value** |
| Origin > Domain Name | Don't select the bucket from the dropdown list. Paste the Static website hosting endpoint of the form `<bucket-name>.s3-website-region.amazonaws.com` |
| Origin > Enable Origin Shield | No  |
| Default cache behavior | Use default settings |
| Cache key and origin requests | Use default settings |

![img_18.png](img_18.png)
Configurations - Origin details


![img_19.png](img_19.png)
Configurations - Cache behavior, key and origin requests

5.Leave the defaults for the rest of the options, and click “Create Distribution”. It may take up to 10 minutes for the CloudFront Distribution to get created.

**Note**: It may take up to **_10 minutes_** for the CloudFront Distribution to be created.

6. Once the status of your distribution changes from “In Progress” to “Deployed”, copy the endpoint URL for your CloudFront distribution found in the “Domain Name” column.

_**Note** - Remember, as soon as your CloudFront distribution is **Deployed**, it attaches to S3 and starts caching the S3 pages. CloudFront may take 10-30 minutes (or more) to cache the S3 page. Once the caching is complete, the CloudFront domain name URL will stop redirecting to the S3 object URL._

![img_20.png](img_20.png)


In this example, the Domain Name value is `dgf7z6g067r6d.cloudfront.net`, but **_yours will be different_**.

### Section 7: Access Website in Web Browser

**Note** - _In the steps below, the exact domain name and the S3 URLs will be different in your case._

1. Open a web browser like Google Chrome, and paste the copied CloudFront domain name (such as, `dgf7z6g067r6d.cloudfront.net`) **without appending** `/index.html` at the end. The CloudFront domain name should show you the content of the default home-page, as shown below:

![img_21.png](img_21.png)
The figure above shows the page displayed at `https://dgf7z6g067r6d.cloudfront.net`

2. Access the website via website-endpoint, such as `http://<bucket-name>.s3-website.us-east-2.amazonaws.com/`.


3. Access the bucket object via its S3 object URL, such as, `https://<bucket-name>.s3.amazonaws.com/index.html`.

All three links: CloudFront domain name, S3 object URL, and website-endpoint will show you the same `index.html` content.

**_If we were not "hosting" the website on S3, we could have made the bucket private and host the content only through the CloudFront domain name. In such a case, we cannot access the private content using S3 object URL and website-endpoint._**

#### Troubleshooting Tip

1. After completing the project instructions, if you are unable to view the website contents, refer to the following guide: [I’m using an S3 website endpoint as the origin of my CloudFront distribution. Why am I getting 403 Access Denied errors?](https://aws.amazon.com/premiumsupport/knowledge-center/s3-website-cloudfront-error-403/)


2. Refer to this official tutorial [Using a website endpoint as the origin, with anonymous (public) access allowed](https://aws.amazon.com/premiumsupport/knowledge-center/cloudfront-serve-static-website/), and verify if you have used the correct domain for your distribution. It should essentially be the **Static website hosting** endpoint of the form `<bucket-name>.s3-website-region.amazonaws.com`

![img_22.png](img_22.png)
Use the Static website hosting endpoint to create the distribution






























