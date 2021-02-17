# Ultimate-AWS-Cheat-Sheet (Set-up Guide)

## Objective

Short Summary: Host an infrequently used WordPress Website completely on AWS as inexpensively as possible.

Longer version:

The Ultimate AWS Cheat Sheet is a wordpress Website hosted on Amazon Elastic Compute Cloud. The objective of this project is to host the website completely using AWS services for as little as possible. The project should adhere to the core principles of the AWS Well-Architected Principles but should prioritize cost-optomization. We will architect several different solutions and then choose the most appropriate path depending on what the customer values.
  
This Readme will be seperated into several different sections
  

1. Objective
2. Planning phase
3. Lightsail vs EC2 AMI/RDS
4. Analysis
5. Route 53
6. Further Cost Optomizations
7. Conclusion
8. Future Goals

  
  
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

1. Vertical Scaling

This can be achieved by creating an EBS Snapshot and creating a new more powerful instance using the EBS Snapshot. This is the preferred method if the website gains consistent traction as a more sustained load would be better handled by more powerful hardware instead of multiple instaces and can avoid the cost of a load balancer

2. Horizontal Scaling

This can be achieved via a load balancer. This is the preferred method if the website spikes in popularity infrequently (ie. one or two popular articles). This is more costlier and complicated as now a load balancer is introduced as well as creating a new ENI, CloudFormation Template and keeping track of extra EC2 Instance costs. However this does provide greater Availability.


