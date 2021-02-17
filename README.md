# Hosting an Inexpensive Worpress Site (An Architect Approach)

## Objective

Short Summary: Host an infrequently used WordPress Website completely on AWS as inexpensively as possible.

Longer version:

The Ultimate AWS Cheat Sheet is a dynamic wordpress Website hosted on Amazon Elastic Compute Cloud. The objective of this project is to host the website completely using AWS services for as little as possible. The project should adhere to the core principles of the AWS Well-Architected Principles but should prioritize cost-optomization. We will architect several different solutions and then choose the most appropriate path depending on what the customer values.
  
This ReadMe will be seperated into several different sections
  

1. Objective
2. Planning phase
3. Lightsail vs EC2 AMI/RDS
4. S3 Integration
5. Route 53
6. Further Cost Optomizations
7. Conclusion

  
## Planning Phase

Due to time constraints, creating a custom website utilizing HTML, CSS & Javascript was not a possibility, so we began to look for alternatives. **Wordpress was ultimately chosen as an alternative as it was inexpensive, flexible with AWS and easy to use.** Amazon has already posted a whitepaper regarding how to install Wordpress using Amazon Lightsail however we would also like to make this available on all Availability Zones. Currently the Wordpress Lightsail Instance is unavailable in the US West region (US WEST-1 N. California) and while the latency difference wouldn't particularly be noticable. 

We will try multiple methods and calculate the **cost, performance, scalability and availability**. 

For Test Instances we will utilize a dummy site with dummy data using the ["Demo Data Creator Plugin"](https://wordpress.org/plugins/demo-data-creator/) 

### T2 VS T3a / Micro or Nano

Part of architecting any solution on AWS should start with estimating Hardware Requirements as the first step as this dictates the majority of the pricing for this project. For running a Wordpress instance (with most likely less than 500 visitors per month) a single nano instance should theoretically be more than enough performance to handle this workload. Depending on the type of the nano instance this method could prove to be unoptimal. **Pricing calculations will all be done based on the 1-year reserved instance pricing and is the cost per hour.** 

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

This can be achieved by creating an EBS Snapshot and creating a new more powerful instance using the EBS Snapshot. This is the preferred method if the website gains consistent traction as a more sustained load would be better handled by more powerful hardware instead of multiple instaces and can avoid the cost of a load balancer. In lieu of a load balancer, we can use Cloudwatch to monitor cpu utilization, from there use SNS (to inform the user), SQS to trigger a SQS function that triggers a [Lambda function that takes an EBS Snapshot](https://aws.amazon.com/blogs/compute/automating-amazon-ebs-snapshot-management-with-aws-step-functions-and-amazon-cloudwatch-events/) and creates a more powerful EC2 Instance consisting of a T3 Micro (Limited Burst).

2. Horizontal Scaling

This can be achieved via a load balancer. This is the preferred method if the website spikes in popularity infrequently (ie. one or two popular articles). This is more costlier and complicated as now a load balancer is introduced as well as creating a new ENI, CloudFormation Template and keeping track of extra EC2 Instance costs. However this does provide greater Availability.

Now that the EC2 Instance type has been selected (T3a Nano Limited Burst and possibility to move to T3a micro if there is a sustained load) we can move onto a comparison with Lightsail

## Lightsail vs EC2/RDS

Lightsail is one of the most popular services available on AWS due to it's simplicity. In under 2 minutes, a user can create and host a wordpress instance. While this accessibility is certainly nice and the costs are very competitive with on-demand instances, it is less cost-efficent compared to reserved instances. 

It is assumed that Lightsail uses T2 Nano (limited) instances for its least expensive plan ($3.5 USD). This can be inferred as it lists having only [1 VCPU](https://aws.amazon.com/lightsail/pricing/) similar to the T2 options on the EC2 Menu. 

Lightsail does come with some benefits for the pricing, mainly the 1TB of Transfer included in the plan. It's also incredibly easy to setup Automatic EBS Snapshots which if we 

All in all if the customer would like a very simple to setup and maintain solution without any unexpected costs, Lightsail is certainly the way to go. In the case of AWS Ultimate Cheat Sheet, we are aiming to maximize our performance per dollar while maintaining scalability so the use of scalable T3a instances are the better solution. Lightsail instances are also only available in certain availabilty zones, so if latency is a concern this should be avoided unless the user also wants to setup a CDN (Content Delivery Network)

## S3 Integration

Part of AWS Cheat Sheet is the integration of hands on video tutorials. By hosting these in S3 buckets, we can save on the costs while ensuring that all of our content is safely hosted in the cloud. S3 videos can be embeded into the wordpress site using this [plugin](https://github.com/anthony-mills/s3-video) as long as the proper permissions are allowed. That being said in the case of AWS Ultimate Cheat Sheet since the data does not need to be secure, it would actually better for the SEO if it was Youtube links embedded in the site. Considering we don't know the expected traffic on the site, it would make the most sense to go with [3 Intelligent Tiering](https://aws.amazon.com/s3/storage-classes/) for maximum cost savings.

## Route 53

Route 53 is a useful service for registering a domain name and DNS for the site. It offers different pricing according to the suffix used for the site. In this case we opted for the most-common .com suffix which cost about $12 USD. The process is very similar on both lightsail and on a standard EC2 instance. We registered the name UltimateAWSCheatSheet.com

## Further Cost Optomization

In order to further cost optomize we should look at the different parts of the arcitechure and see where we can cut back. Using a T3a Nano (limited burst) on a reserved instance would be the most-cost effective hardware option which would mean avoiding Lightsail. 

Losing the ability to scale by avoiding both CloudWatch monitoring and a Load Balancer would also save costs

Finally registering a domain with the suffix .link ($5 USD) would be the most-cost effective URL

## Conclusion

Let's recap all of the options:

Most Cost Effective -> T3a Nano (Limited Burst) + RDS -> 2.026 USD per Month

This option is not scalable at all and will throttle under load once all bursts are used. One year term

Cost Effective and Vertically Scalable -> T3a Nano + RDS + CloudWatch + Step-Function + Lambda + EBS Snapshot + (Potential T3a Micro) -> $3-4

This method is good for a long term approach as the website grows and can scale accordingly

Most User-Friendly Method -> LightSail -> 3.5 USD per month + 0.08 USD/gb of EBS Snapshots

The plan can be changed month to month to meet demand, so it is pretty easily scalable.

*Please note all of these costs do not include the added S3 storage as that is variable and the domain name depends on the suffix (or top level domain)

### After weighing out all of the options, and considering everything from a customer perspective we've decided to go with a Lightsail instance for three main reasons:

1. Accessibility: If the admin ever needs to be changed, granting permissions to a single Lightsail instance via IAM is simpler than granting access to multiple different services (altough a group and policy could be made to mitigate this)
2. Scaling the instances both vertically and horizontally can be done very easily via Load balancers built into Lightsail and changing the Instance type by launching a new Lightsail instance
3. Most importantly, month to month flexibility. The client is unsure how much content they will be putting into this wordpress website and may need to hibernate it for periods of time

## The key take away!

If anyone from Amazon ever reads this, I'd love to know why T3 instances aren't the default for Lightsail and is the Lightscale discount caused by mass economies of scale 

Thank you for reading and here is another link to the site: 

Resources Used:

https://lightsail.aws.amazon.com/ls/docs/en_us/articles/amazon-lightsail-tutorial-launching-and-configuring-wordpress
https://aws.amazon.com/blogs/compute/deploying-a-highly-available-wordpress-site-on-amazon-lightsail-part-1-implementing-a-highly-available-lightsail-database-with-wordpress/
https://aws.amazon.com/getting-started/hands-on/build-wordpress-website/
https://aws.amazon.com/blogs/architecture/wordpress-best-practices-on-aws/
http://d0.awsstatic.com/whitepapers/deploying-wordpress-with-aws-elastic-beanstalk.pdf

