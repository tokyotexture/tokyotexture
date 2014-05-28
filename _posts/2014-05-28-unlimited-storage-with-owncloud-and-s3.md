---
layout: post
title: "Unlimited storage with ownCloud and S3"
modified: 2014-05-28 04:17:00 +0000
category: posts
tags: [aws, storage, owncloud, s3]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
---

Ok, so you want to be in control of your own storage, but you also want all the benefits of unlimited, cheap and extremely redundant storage that Amazon S3 provides. Well, you are in luck. [ownCloud](http://owncloud.org/) is an easy (and open source) way of easily interfacing with Amazon S3.

>ownCloud provides universal access to your files via the web, your computer or your mobile devices — wherever you are.
>It also provides a platform to easily view & sync your contacts, calendars and bookmarks across all your devices and enables basic editing right on the web.

This all sounds great, but how to get started? Fortunately enough, the nice guys over at [Begood Technologies](http://owncloud.jp/) have already pre-packaged everything in a simple 1-click installer on the AWS Marketplace.

In this blog post I will go through on how to set up ownCloud in your own AWS account, and then connect it to unlimited and cheap storage in Amazon S3.

----

## Prerequisites

* Have an [AWS account](http://aws.amazon.com) already set up
* Have an EC2 [Key Pair](http://docs.aws.amazon.com/gettingstarted/latest/wah/getting-started-create-key-pair.html) ready.

That's it, now let's get started.

## ownCloud setup

Head over to the ["ownCloud powered by owncloud.jp" marketplace listing](https://aws.amazon.com/marketplace/pp/B00F8I7IXG/ref=srh_res_product_title?ie=UTF8&sr=0-3&qid=1380794974138). The screen might differ slightly depending on how new your account it, but you will want to

1. Pick the region where you want to launch (the one closest to you)
2. Choose the instance type you want. The bigger the instance type, the faster it will be, but it will also cost more. ownCloud is free, but EC2 will be charged per hour. See the right side table for pricing. If you are just testing it out, t1.micro will be part of the free tier. You can always change this later.
3. Choose the premade key pair. You will need this if you want to SSH in to the box (which you will most likely not need).
4. Just click launch

![marketplace]({{ site.url }}/images/2014-05-28/owncloud-marketplace-2.png)

After a couple of minutes, you will have an instance up and running with ownCloud preinstalled. You can find the adress to the instance either by

* Checking the [software subscribtions page](https://aws.amazon.com/marketplace/library/ref=mrc_prm_manage_subscriptions) and clicking "access software"
* or, by going to the instance screen in the [EC2 dashboard](https://console.aws.amazon.com/ec2/) (make sure you are looking at the same region as where you created the instance).

If you type the address to the server, you should get an account setup screen for ownCloud as follows:

![owncloud setup]({{ site.url }}/images/2014-05-28/owncloud-account-setup.png)

* Choose administrator user name and password
* Set data folder to /home/data
* Choose MySQL as your database
 * database username: admin
 * database password: owncloud5
 * database name: owncloud
 * database host: localhost

Finally click "Finish setup", and you should get the ownCloud dashboard.

![owncloud dashboard]({{ site.url }}/images/2014-05-28/owncloud-dashboard.png)

## Amazon S3 and ownCloud integration

Now we have a working ownCloud server, but we want to integrate it with Amazon S3 for unlimited, cheap and reliable storage.

### Create an S3 bucket

Go to the [S3 console](https://console.aws.amazon.com/s3/) and click "Create bucket".

![create S3 bucket]({{ site.url }}/images/2014-05-28/create-s3-bucket.png)

### Setup IAM permissions

Now we want to create a IAM user which can only access that bucket.

Go to the [IAM dashboard](https://console.aws.amazon.com/iam/) and click Users, then "Create New Users"

![IAM-user-creation]({{ site.url }}/images/2014-05-28/IAM-user-creation.png)

Note the access key and secret key, alternatively download the credentials as a .csv for safe keeping. You will need these later on!

![IAM credentials]({{ site.url }}/images/2014-05-28/IAM-user-credentials.png)

Now we need to define the permissions for this user by attaching a user policy. Choose the "S3 full access" as a baseline.

![IAM-user-attach-policy]({{ site.url }}/images/2014-05-28/IAM-user-attach-policy.png)

![IAM 2]({{ site.url }}/images/2014-05-28/IAM-user-attach-policy-2.png)

![IAM 3]({{ site.url }}/images/2014-05-28/IAM-user-attach-policy-2.png)

However, as you can see, this would grant permissions for everything S3 related. If you want to be a bit more specific, and only grant permissions to the bucket we just created, change the policy to something like this, where "demo-owncloud" is your bucket name.

{% raw %}
	{
	 "Version":"2012-10-17",
	 "Statement": [
	   {
	     "Sid": "AllowUserToSeeBucketListInTheConsole",
	     "Action": ["s3:ListAllMyBuckets", "s3:GetBucketLocation"],
	     "Effect": "Allow",
	     "Resource": ["arn:aws:s3:::*"]
	   },
	   {
	     "Sid": "AllowAllS3ActionsInUserFolder",
	     "Effect": "Allow",
	     "Action": ["s3:*"],
	     "Resource": ["arn:aws:s3:::demo-owncloud/*"]
	   }
	 ]
	}
{% endraw %}


### Setup Amazon S3 in ownCloud

First of all click the "+ apps" button in the bottom right in ownCloud and enable "External storage support".

![owncloud apps]({{ site.url }}/images/2014-05-28/owncloud-enable-external-storage.png)

Then head over to the administration page and find "External Storage". Once there choose "Amazon S3" from the "Add storage" pulldown menu.

![owncloud admin]({{ site.url }}/images/2014-05-28/owncloud-admin.png)

* Folder name: Set to whatever you want.
* Access Key: Enter the access key we created for the IAM user
* Secret Key: Enter the secret key we created for the IAM user
* Bucket: Enter bucket name (example: demo-owncloud)
* Region: Enter the region where you created the bucket. You will need the programmatic name. For example, Tokyo is "ap-northeast-1". You can find all [region mappings here](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html).
* Applicable: Set the relevant users you want to expose this function to.

I found ownCloud to be somewhat finicky regarding naming/regions of the buckets. Possibly some validation logic being too strict (?). Once you get it working you will see a green box to the left of the row. If it didn't work, you will get a red box.

Alright, let's see if we can upload a document.

![owncloud upload]({{ site.url }}/images/2014-05-28/owncloud-upload.png)

Now let's see if it is really in S3.

![S3 upload check]({{ site.url }}/images/2014-05-28/S3-upload-check.png)

Awesome! My upload to ownCloud, which I manage by myself, was securely and reliably stored on S3. As of writing the GB cost in the Tokyo region was about 3.4Yen for triple redundancy. That's cheap.

## Where to go from here?

Now you can start to play around with ownCloud itself, and setup permissions etc just the way you want it. For more information check out the [ownCloud homepage](http://owncloud.org)
