# The most cost-effective method of hosting a WordPress Site completely on AWS

## Objective

Link to the Target Website: 	http://ultimateawscheatsheet.com/

Short Summary: There are multiple ways to host a wordpress website on AWS with different levels of cost-effectiveness. Skip to the [conclusion](#Conclusion) for the results.

Longer version:

The Ultimate AWS Cheat Sheet is a dynamic wordpress Website hosted on Amazon Elastic Compute Cloud. The objective of this project is to host the website completely using AWS services for as little as possible. The project should adhere to the core principles of the AWS Well-Architected Principles but should prioritize cost-optomization. We will architect several different solutions and then choose the most appropriate path depending on what the customer values.

Keep in mind we will be adhering to the [5 Pillars of the AWS Well Architected Framework](https://aws.amazon.com/blogs/apn/the-5-pillars-of-the-aws-well-architected-framework/)
  
This ReadMe will be seperated into several different sections
  

1. Objective
2. Planning phase
3. Lightsail vs Wordpress EC2 AMI
4. S3 Integration
5. Route 53
6. Further Cost Optomizations
7. Additional Add-Ons
8. Conclusion

This project will cover multiple AWS services and recommends a general understanding of the following services: (Lightsail, EFS, RDS, EC2, Lambda, S3. Cloudfront, Application Load Balancer, Auto Scaling, Route 53, AWS Shield, IAM)

  
## Planning Phase

Due to time constraints, creating a custom website utilizing HTML, CSS & Javascript was not a possibility, so we began to look for alternatives. **Wordpress was ultimately chosen as an alternative as it was inexpensive, flexible with AWS and easy to use.** Amazon has already posted a whitepaper regarding how to install Wordpress using Amazon Lightsail however we would also like to make this available on all Availability Zones. Currently the Wordpress Lightsail Instance is unavailable in the US West region (US WEST-1 N. California) and while the latency difference wouldn't particularly be noticable. 

We will try multiple methods and calculate the **cost, performance, scalability and availability**. 

For Test Instances we will utilize a dummy site with dummy data using the ["Demo Data Creator Plugin"](https://wordpress.org/plugins/demo-data-creator/) 

### T2 VS T3a / Micro or Nano

Part of architecting any solution on AWS should start with estimating Hardware Requirements as the first step as this dictates the majority of the pricing for this project. For running a Wordpress instance (with most likely less than 500 visitors per month) a single nano instance should theoretically be more than enough performance to handle this workload. Depending on the type of the nano instance this method could prove to be unoptimal. **Pricing calculations will all be done based on the 1-year reserved instance pricing and is the cost per hour.** 

T2 cores are based off of Intel Architecture (Intel Xeon) while T3a cores are based off of (AMD EPYC 7000) architecture.

Lets break down the possible configurations:

[T2 Instances:](https://i.imgur.com/n5UMxPy.png)

T2 Nano ($0.0030)

T2 Micro ($0.0070)

[T3 Instances:](https://i.imgur.com/3FtouFx.png)

T3a Nano ($0.0028)

T3a Micro ($0.0056)

In the most probable scenario (less than 500 visitors per month) a T3a Nano Instance would be the ideal configuration for several key reasons.

1. This is the least expensive pricing option
2. Offers greater performance due to the additional VCPU core compared to T2 as well as better cpu architecture
3. Has much greater Burst so the website will be able to handle a relatively large amount of stress for prolonged periods
4. Generates more credits per hour and credits are stored in a weekly amount instead of a daily limit

Choosing this method does come with one big caveat however:

Similar to the T2Unlimited instances, the T3 instances allow for much greater burst and no performance throttling which can ***drastically*** increase the price of the instance if the hardware is being heavily utilized above the baseline. Without adding the additional protection of AWS Shield Advanced, more complicated DDOS attacks or SQL injections (due to the mySQL database used by this WordPress Instance) could cause the system to run at maximum resources.

Ultimatly this decision depends on the customer's values and expectations. Considering that I (a college student) am the customer in this case, I want to prioritize lower costs and sacrifice on the performance side of things. This means disabling the Unlimited Burst option *(Turned on by default for T3 instances)*

At the moment the customer has stated that they expect <500 users, however a key pillar of AWS is ***Scalability***. There is two ways to scale in this case.

### Scaling

1. Vertical Scaling

This can be achieved by creating an EBS Snapshot and creating a new more powerful instance using the EBS Snapshot. This is the preferred method if the website gains consistent traction as a more sustained load would be better handled by more powerful hardware instead of multiple instaces and can avoid the cost of a load balancer. In lieu of a load balancer, we can use Cloudwatch to monitor cpu utilization, from there use SNS (to inform the user), SQS to trigger a SQS function that triggers a [Lambda function that takes an EBS Snapshot](https://aws.amazon.com/blogs/compute/automating-amazon-ebs-snapshot-management-with-aws-step-functions-and-amazon-cloudwatch-events/) and creates a more powerful EC2 Instance consisting of a T3 Micro (Limited Burst). If the wordpress is reaching storage capacity, it may be worth switching to RDS to manage the SQL database, however this can be costly. **EDIT(2021/02/17) The Wordpress site will need to be manually migrated as the wordpress migration cannot be activated through automation yet (WIP)**

2. Horizontal Scaling

This can be achieved via a load balancer. This is the preferred method if the website spikes in popularity infrequently (ie. one or two popular articles). This is more costlier and complicated as now a load balancer is introduced as well as creating a new ENI, CloudFormation Template and keeping track of extra EC2 Instance costs. However this does provide greater Availability.

Now that the EC2 Instance type has been selected (T3a Nano Limited Burst and possibility to move to T3a micro if there is a sustained load) we can move onto a comparison with Lightsail

## Lightsail vs EC2/RDS

Lightsail is one of the most popular services available on AWS due to it's simplicity. In under 2 minutes, a user can create and host a wordpress instance. While this accessibility is certainly nice and the costs are very competitive with on-demand instances, it is less cost-efficent compared to reserved instances. 

It is assumed that Lightsail uses T2 Nano (limited) instances for its least expensive plan ($3.5 USD). This can be inferred as it lists having only [1 VCPU](https://aws.amazon.com/lightsail/pricing/) similar to the T2 options on the EC2 Menu. 

Lightsail does come with some benefits for the pricing, mainly the 1TB of Transfer included in the plan. It's also incredibly easy to setup Automatic EBS Snapshots which if we 

All in all if the customer would like a very simple to setup and maintain solution without any unexpected costs, Lightsail is certainly the way to go. In the case of AWS Ultimate Cheat Sheet, we are aiming to maximize our performance per dollar while maintaining scalability so the use of scalable T3a instances are the better solution. Lightsail instances are also only available in certain availabilty zones, so if latency is a concern this should be avoided unless the user also wants to setup a CDN (Content Delivery Network)

## S3 Integration

Part of AWS Cheat Sheet is the integration of hands on video tutorials which means sizable amounts of data. By hosting these in S3 buckets, we can save on the costs of running a higher storage instance while ensuring that all of our content is safely hosted in the cloud. S3 videos can be embeded into the wordpress site using this [plugin](https://github.com/anthony-mills/s3-video) as long as the proper permissions are allowed. That being said in the case of AWS Ultimate Cheat Sheet since the data does not need to be secure, it would actually better for the SEO if it was Youtube links embedded in the site. Considering we don't know the expected traffic on the site, it would make the most sense to go with [S3 Intelligent Tiering](https://aws.amazon.com/s3/storage-classes/) for maximum cost savings.

## Route 53

Route 53 is a useful service for registering a domain name and DNS for the site. It offers different pricing according to the suffix used for the site. In this case we opted for the most-common .com suffix which cost about $12 USD. The process is very similar on both lightsail and on a standard EC2 instance. We registered the name UltimateAWSCheatSheet.com

## Further Cost Optomization

In order to further cost optomize we should look at the different parts of the arcitechure and see where we can cut back. Using a T3a Nano (limited burst) on a reserved instance would be the most-cost effective hardware option which would mean avoiding Lightsail. 

Losing the ability to scale by avoiding both CloudWatch monitoring and a Load Balancer would also save costs

Finally registering a domain with the suffix .link ($5 USD) would be the most-cost effective URL

## Additional Add-Ons

#### Cloudfront

Amazon Cloudfront allows the user to asccess static and dynamic content with lower latency by caching the most accessed material at end-points closer to the end-user. This is especially useful if you are using Lightsail Wordpress instance in an area with no closeby availability zones.

#### EFS (Elastic File System)

Amazon EFS is a scalable network file system that allows the user to access unstructered wordpress data much faster such as plugins, themes, add-ons, or PHP files

# Conclusion

Let's recap all of the options:

Most Cost Effective -> T3a Nano (Limited Burst) -> 2.026 USD per Month

This option is not scalable at all and will throttle under load once all bursts are used. One year term

Cost Effective and Vertically Scalable -> T3a Nano + CloudWatch + Step-Function + Lambda + EBS Snapshot + (Potential T3a Micro) -> $3-4

This method is good for a long term approach as the website grows and can scale accordingly

Most User-Friendly Method -> LightSail -> 3.5 USD per month + 0.08 USD/gb of EBS Snapshots

The plan can be changed month to month to meet demand, so it is pretty easily scalable.

*Please note all of these costs do not include the added S3 storage as that is variable and the domain name depends on the suffix (or top level domain)

### After weighing out all of the options, and considering everything from a customer perspective we've decided to go with a Lightsail instance for four main reasons:

1. Accessibility: If the admin ever needs to be changed, granting permissions to a single Lightsail instance via IAM is simpler than granting access to multiple different services (altough a group and policy could be made to mitigate this)
2. This follows the tenant of not being 
3. Scaling the instances both vertically and horizontally can be done very easily via Load balancers built into Lightsail and changing the Instance type by launching a new Lightsail instance
4. Most importantly, month to month flexibility. The client is unsure how much content they will be putting into this wordpress website and may need to hibernate it for periods of time

## The key take away! (IE REFLECTION)

* It's important to always well architect solutions with a long-term plan from the start. This can help mitigate problems down the road with scalability/reliability * 

* Turn off Unlimited Burst on T3 instances to save money and prevent an unexpected bill *

* T3 instances are more cost-effective and better performing compared to their T2 counter-parts *

* Migrating Wordpress Instances cannot be automated (just of yet... keep an eye for future projects) *


## Question for Readers (please comment below)

Why does Lightsail use T2 instances by default for wordpress? (noted by only 1 CPU Core)


Thank you for reviewing my project! :)


Resources Used:

https://lightsail.aws.amazon.com/ls/docs/en_us/articles/amazon-lightsail-tutorial-launching-and-configuring-wordpress

https://aws.amazon.com/blogs/compute/deploying-a-highly-available-wordpress-site-on-amazon-lightsail-part-1-implementing-a-highly-available-lightsail-database-with-wordpress/

https://aws.amazon.com/getting-started/hands-on/build-wordpress-website/

https://aws.amazon.com/blogs/architecture/wordpress-best-practices-on-aws/

http://d0.awsstatic.com/whitepapers/deploying-wordpress-with-aws-elastic-beanstalk.pdf

https://aws.amazon.com/blogs/aws/new-t3-instances-burstable-cost-effective-performance/
