- [Providers](#providers)
    - [Local Disk](#local-disk)
    - [Amazon S3](#amazon-s3)
    - [Rackspace CDN](#rackspace-cdn)
- [Troubleshooting](#troubleshooting)

<a name="providers"></a>
## Providers

Media Manager uses the Local Disk provider by default. You need to install [Drivers plugin](https://octobercms.com/plugin/october-drivers) before you can use Amazon S3 or Rackspace CDN features.

> **Note**: after you change Media Manager configuration, you should reset its cache. You can do that with pressing the **Refresh** button in the Media Manager toolbar.

<a name="local-disk"></a>
## Local Disk

By default Media Manager works with the storage/app/media subdirectory of the installation directory. In order to use Amazon S3 or Rackspace CDN, you should update the system configuration.

<a name="amazon-s3"></a>
## Configuring Amazon S3 access

To use Amazon S3 with October CMS, you should create S3 bucket, folder in the bucket and API user.

Sign up for Amazon AWS account or sign in with your existing account to AWS Console. Open S3 management panel. Create a new bucket and assign it any name (the name of the bucket will be a part of your public file URLs).

Create **media** folder in the bucket. The folder name doesn't matter. This folder will be a root of your Media Library.

By default files in S3 buckets cannot be accessed directly. To make the bucket public, return to the bucket list and click the bucket. Click **Properties** button in the right sidebar. Expand **Permissions** tab. Click **Edit bucket policy** link. Paste the following code to the policy popup window. Replace the bucket name with your actual bucket name:

    {
        "Version": "2008-10-17",
        "Id": "Policy1397632521960",
        "Statement": [
            {
                "Sid": "Stmt1397633323327",
                "Effect": "Allow",
                "Principal": {
                    "AWS": "*"
                },
                "Action": "s3:GetObject",
                "Resource": "arn:aws:s3:::BUCKETNAME/*"
            }
        ]
    }

Click **Save** button to apply the policy. The policy gives public read-only access to all folders and directories in the bucket. If you're going to use the bucket for other needs, it's possible to setup a public access to a specific folder in the bucket, just specify the directory name in the **Resource** value:

    "arn:aws:s3:::BUCKETNAME/media/*"

You should also create an API user that October CMS will use for managing the bucket files. In AWS console go to IAM section. Go to Users tab and create a new user. The user name doesn't matter. Make sure that "Generate an access key for each user" checkbox is checked when you create a new user. After AWS creates a user, it allows you to see the security credentials - the user **Access Key ID** and **Secret Access Key**. Copy the keys and put them into a temporary text file.

Return to the user list and click the user you just created. In the **Permissions** section click **Attach Policy** button. Select **AmazonS3FullAccess** policy in the list and click **Attach Policy** button.

Now you have all the information to update October CMS configuration. Open **config/filesystem.php** script and find the **disks** section. It already contains s3 configuration, you need to replace the API credentials and bucket information parameters:

Parameter | Value
------------- | -------------
**key** | the **Access Key ID** value of the user that you created before.
**secret** | the **Secret Access Key** value of the user that you created fore.
**bucket** | your bucket name.
**region** | the bucket region code, see below.

You can find the bucket region in S3 management console, in the bucket properties. The Properties tab displays the region name, for example Oregon. S3 driver configuration requires a bucket code. Use this table to find code for your bucket (you can also take a look at [AWS documentation](http://docs.aws.amazon.com/general/latest/gr/rande.html#s3_region)):

Region | Code
------------- | -------------
<span class="nowrap">**US East (Ohio)**</span> | us-east-2
<span class="nowrap">**US East (N. Virginia)**</span> | us-east-1
<span class="nowrap">**US West (N. California)**</span> | us-west-1
<span class="nowrap">**US West (Oregon)**</span> | us-west-2
<span class="nowrap">**Asia Pacific (Hong Kong)**</span> | ap-east-1
<span class="nowrap">**Asia Pacific (Mumbai)**</span> | ap-south-1
<span class="nowrap">**Asia Pacific (Osaka-Local)**</span> | ap-northeast-3
<span class="nowrap">**Asia Pacific (Seoul)**</span> | ap-northeast-2
<span class="nowrap">**Asia Pacific (Singapore)**</span> | ap-southeast-1
<span class="nowrap">**Asia Pacific (Sydney)**</span> | ap-southeast-2
<span class="nowrap">**Asia Pacific (Tokyo)**</span> | ap-northeast-1
<span class="nowrap">**Canada (Central)**</span> | ca-central-1
<span class="nowrap">**China (Beijing)**</span> | cn-north-1
<span class="nowrap">**China (Ningxia)**</span> | cn-northwest-1
<span class="nowrap">**EU (Frankfurt)**</span> | eu-central-1
<span class="nowrap">**EU (Ireland)**</span> | eu-west-1
<span class="nowrap">**EU (London)**</span> | eu-west-2
<span class="nowrap">**EU (Paris)**</span> | eu-west-3
<span class="nowrap">**EU (Stockholm)**</span> | eu-north-1
<span class="nowrap">**South America (São Paulo)**</span> | sa-east-1
<span class="nowrap">**Middle East (Bahrain)**</span> | me-south-1

Example configuration after update:

    'disks' => [
        ...
        's3' => [
            'driver' => 's3',
            'key'    => 'XXXXXXXXXXXXXXXXXXXX',
            'secret' => 'xxxXxXX+XxxxxXXxXxxxxxxXxxXXXXXXXxxxX9Xx',
            'region' => 'us-west-2',
            'bucket' => 'my-bucket'
        ],
        ...
    ]

Save **config/filesystem.php** script and open **config/cms.php** script. Find the section **storage**. In the **media** parameter update **disk**, **folder** and **path** parameters:

Parameter | Value
------------- | -------------
**disk** | use **s3** value.
**folder** | the name of the folder you created in S3 bucket.
**path** | the public path of the folder in the bucket, see below.

To obtain the path of the folder, open AWS console and go to S3 section. Navigate to the bucket and click the folder you created before. Upload any file to the folder and click the file. Click **Properties** button in the right sidebar. The file URL is in the **Link** parameter. Copy the URL and remove the file name and the trailing slash from it.

Example storage configuration:

    'storage' => [
        ...
        'media' => [
            'disk'   => 's3',
            'folder' => 'media',
            'path' => 'https://s3-us-west-2.amazonaws.com/your-bucket-name/media'
        ]
    ]

Congratulations! Now you're ready to use Amazon S3 with October CMS. Note that you can also configure Amazon CloudFront CDN  to work with your bucket. This topic is not covered in this document, please refer to [CloudFront documentation](http://aws.amazon.com/cloudfront/). After you configure CloudFront, you will need to update the **path** parameter in the storage configuration.

<a name="rackspace-cdn"></a>
## Configuring Rackspace CDN access

To use Rackspace CDN with October CMS, you should create Rackspace CDN container, folder in the container and API user.

Log into Rackspace management console and navigate to Storage / Files page. Create a new container. The container name doesn't matter, it will be a part of your public file URLs. Select **Public (Enabled CDN)** type for the new container.

Create **media** folder in the container. The folder name doesn't matter. This folder will be a root of your Media Library.

You should create an API user that October CMS will use for managing files in the CDN container. Open Account / User Management page in Rackspace console. Click **Create user** button. Fill in the user name (for example october.cdn.api), password, security question and answer. In the **Product Access** section select **Custom** and in the CDN row select **Admin**. Use **No Access** role in the **Account** section and use **Technical Contact** type in the **Contact Information** section. Save the user account. After saving the account you will see the Login Details section with the **API Key** row that contains a value you need to use in October CMS configuration files.

Now you have all the information to update October CMS configuration. Open **config/filesystem.php** script and find the **disks** section. It already contains Rackspace configuration, you need to replace the API credentials and container information parameters:

Parameter | Value
------------- | -------------
**username** | Rackspace user name (for example october.cdn.api).
**key** | the user's **API Key** that you can copy from Rackspace user profile page.
**container** | the container name.
**region** | the bucket region code, see below.
**endpoint** | leave the value as is.
**region** | you can find the region in the CDN container list in Rackspace control panel. The code is a 3-letter value, for example it's **ORD** for Chicago.

Example configuration after update:

    'disks' => [
        ...
        'rackspace' => [
            'driver'    => 'rackspace',
            'username'  => 'october.api.cdn',
            'key'       => 'xx00000000xxxxxx0x0x0x000xx0x0x0',
            'container' => 'my-bucket',
            'endpoint'  => 'https://identity.api.rackspacecloud.com/v2.0/',
            'region'    => 'ORD'
        ],
        ...
    ]

Save **config/filesystem.php** script and open **config/cms.php** script. Find the section **storage**. In the **media** parameter update **disk**, **folder** and **path** parameters:

Parameter | Value
------------- | -------------
**disk** | use **rackspace** value.
**folder** | the name of the folder you created in CDN container.
**path** | the public path of the folder in the container, see below.

To obtain the path of the folder, go to the CDN container list in Rackspace console. Click the container and open the media folder. Upload any file. After the file is uploaded, click it. The file will open in a new browser tab. Copy the file URL and remove the file name and trailing slash from it.

Example storage configuration:

    'storage' => [
        ...
        'media' => [
            'disk'   => 'rackspace',
            'folder' => 'media',
            'path' => 'https://xxxxxxxxx-xxxxxxxxx.r00.cf0.rackcdn.com/media'
        ]
    ]

Congratulations! Now you're ready to use Rackspace CDN with October CMS.

<a name="troubleshooting"></a>
## Troubleshooting

The most common issue with using remote services is the SSL connection problem. If you get SSL errors, make sure that your server has fresh SSL certificates of public Certificate Authorities (CA).