---
layout: post
title: "Jekyll &amp; S3"
category: posts
description: "A tutorial on getting Jekyll set up with AWS S3 for highly available and cheap blogging"
modified: 2014-05-07 06:15:53 +0000
comments: true
tags: [jekyll, aws]
image:
  feature: 2014-05-07/jekyll_AWS.jpg
  credit: 
  creditlink: 
share: 
---
<section id="table-of-contents" class="toc">
  <header>
    <h3>Contents</h3>
  </header>
<div id="drawer" markdown="1">
*  Auto generated table of contents
{:toc}
</div>
</section><!-- /#table-of-contents -->

I've been using wordpress on again and off again for many years now, but I was ready to try something new. One of the main drivers for trying something new was [Amazon S3](http://aws.amazon.com/s3/). For those of you not familiar with S3, it is

* **Secure**
S3 is an object storage, and you store static content in it. In other words, SQL injections etc which something like Wordpress brings along just isn't possible.
* **Reliable**
99.99999999999% durability and 99.99% availability. No single point of failure. Yes please!
* **Scalable**
Can scale in terms of storage and request rate.
* **Inexpensive**
You only pay for what you use, and as of writing the storage fee in Tokyo is now about 3.3Yen/GB. That is *cheap*.

So as long as my content is static, S3 would be a godsend, as it would fix all the aspects I don't like about LAMP type setups like Wordpress.

![Jekyll Logo]({{ site.url }}/images/2014-05-07/jekyll-logo.png)
{: .image-pull-right; width: 100px;}

Well, this is where [Jekyll](https://github.com/jekyll/jekyll) comes into play.

>Jekyll is a simple, blog-aware, static site generator perfect for personal, project, or organization sites. Think of it like a file-based CMS, without all the complexity. Jekyll takes your content, renders Markdown and Liquid templates, and spits out a complete, static website ready to be served by Apache, Nginx or another web server. Jekyll is the engine behind GitHub Pages, which you can use to host sites right from your GitHub repositories.

When it comes to marrying S3 and Jekyll, it seems I'm not the first person to have this brilliant idea. Even Werner Vogels, the CTO of Amazon Web Services, runs his blog, [All things distributed](http://www.allthingsdistributed.com/2011/08/Jekyll-amazon-s3.html) this way. Neat.

## Setup

Ok, so let's get to the meat and potatoes of this blog post, which is how to get this up and running.

### Prerequisites

Some things you will need before we get started.

* An [AWS](http://aws.amazon.com) account.
If you don't already have an account, you can sign up for free, and get a [1 year free usage tier](http://aws.amazon.com/free/). As of writing, 5GB of S3 space, and 15GB of outbound traffic is included for free, which should mean that you can host a smaller blog for free using this guide.
* A machine with Ruby installed.
If you are running OS X you will already have this installed. Personally I just spun up an Ubuntu EC2 instance on AWS, but that is outside the scope of this blog post.
* *Optionally* have a domain name, such as `tokyotexture.com`, ready.

### Configuring Jekyll locally
First of all, we will need to install Jekyll. I will use a theme called [minimal mistakes](http://mmistakes.github.io/minimal-mistakes/), which is what this particular blog happens to run, as a starting line. 

#### Installing Jekyll
1. Install [bundler](http://bundler.io) by running `gem install bundler`.
2. Get minimal mistakes latest source, `wget https://github.com/mmistakes/minimal-mistakes/archive/master.zip`. For those of you familiar with github, feel free to clone instead.
3. Unzip and change directory to the unzipped minimal mistakes theme folder.
4. Install jekyll by running `bundle install` from the theme folder. Bundler will look at the `Gemfile` in that folder and figure out what gems are needed and install accordingly.

Ok, so you made it this far, which means you are done installing Jekyll. But first let's check that things are working as intended.

#### Basic Jekyll configuration and testing
1. From within the theme folder, run `sudo bundle exec jekyll serve`. Running this command will user bundler to execute `jekyll` with the `serve` parameter. This will create a very basic web server.
2. Now verify that it is serving up something at `http://localhost:4000`. Hopefully it is serving up something at this point.

Now you can start configuring `_config.yml`, looking at the files in the `_posts/` folder to figure out the syntax etc.

### Deployment

Alright, so now you have something very basic up and running locally on your own machine, but I promised very cheap and extremely reliable hosting; right? That's right.

#### Creating an S3 bucket

1. Login to the [AWS Management console](https://console.aws.amazon.com) and select S3
2. Create a new bucket
3. The bucket name should be the same as your domain name, in my case `www.tokyotexture.com`
4. Pick the region of your liking. The natural choice would be a region where you have most of your readers.
![Create a new S3 bucket]({{ site.url }}/images/2014-05-07/createbucket.png)
5. After you click create, enable static website hosting.
![Enable S3 website hosting]({{ site.url }}/images/2014-05-07/websitehosting.png)

#### Setting up Route53 DNS entries

Assuming you have have a domain name you want to point to the newly created S3 bucket, you can easily set that up using Route53. This part assumes you have already set up your hosted zone with Route53, and if you haven't, [read this Route53 documentation](http://docs.aws.amazon.com/Route53/latest/DeveloperGuide/CreatingNewDNS.html).

Create a new `A record` using Route53's `alias records` to point to the specific S3 bucket.

![Route53 alias record]({{ site.url }}/images/2014-05-07/aliasrecord.png)

#### Installing s3_website and deploying

It seems I am not the first person to realize that Jekyll + S3 = ♡, and there is already a tool called [s3_website](https://github.com/laurilehmijoki/s3_website) that will do pretty much everything to push your data to S3 on a regular basis.

* To install you simply run `gem install s3_website`
* Go to your website directory
* Run `s3_website cfg create`, which will create a default configuration file called `s3_website.yml`.
* Set the AWS credentials, which we created in the S3 step, into to `s3_website.yml`.
* Run `s3_website cfg apply` which will apply the necessary settings to your S3 bucket.
* Run `sudo bundle exec jekyll build` to create the static version of your jekyll site.
* Finally run `s3_website push` to push the above build into S3.

For more details see the [s3_website site](https://github.com/laurilehmijoki/s3_website).

