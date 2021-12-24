# ECS personal project - DRAFT

This is my personal project to learn how to use AWS ECS and Honeycomb. I wanted to see how I could 
upload and use a Docker image I created to [ECS](https://aws.amazon.com/ecs/). Then, I wanted to 
see how I could export my CloudWatch logs to [Honeycomb](https://www.honeycomb.io/). There I could 
create reports and graphs of the logs.

Please note, if you want to see the logs in Honeycomb. You will have to use an *nginx* or similar image.

## Pre-requisites
Have a Honeycomb account. If you do not have one, you can sign up for a free one on their site [here](https://ui.honeycomb.io/signup).

## Steps to using a Docker image in ECS
1. Login to the [AWS management console](https://aws.amazon.com/console/) using an account that has access to ECS
2. Navigate to Amazon ECS
3. We are going to use the ECS wizard, so click on the **Get Started** button
4. For *Container Image* you can either use *nginx* image or you can use one of your own. I used one of 
my own and to do this go to step 5. Other wise go to step 10
5. To use your own image, click on the **Edit** button
6. For **Container name**, I used *my-web-app*
7. For **Image**, this is where you can pull an image from Docker Hub. And to do this the format is similar to the 
`Docker pull` command: `<account>/<repo name>:<image name>`
8. Leave all other settings as is
9. Click on the **Update** button to finish up
10. Click on the **Next** button
11. On the **Define your service** page, we will need a load balancer. So, for *Load balancer type* select 
*Application Load Balancer*
12. Click on the **Next** button
13. \(Optional\) On the **Configure your cluster** page, update the *Cluster name* to *my-web-app-cluster*
14. Click on the **Next** button
15. Finally, on the **Review** page, to create the Cluster click on the **Create** button. Now, you will have 
to wait a few minutes for the Cluster and its resources to be created.
16. Once, the ECS service has been created click on the **View Service** button

## To view you ECS service, web site
To see the web site, we need the DNS entry for the load balancer used by our ECS service

1. On the **Service** page, click on the entry for the *Target Group Name*
2. On the **Target groups** page, click on the name of the target group. It should be of the format: `EC<random characters>`
3. On the specific Target group page, click on the *Load Balancer*
4. On the **Load Balancers** page, ensure the load balancer is selected
5. Under the details for the load balancer, scroll down. Under the *Description* tab, copy the *DNS name*. This will be the URL to your site
6. Paste the *DNS name* (URL) in a browser to your site

## Exporting data from CloudWatch Logs to Honeycomb
I wanted to obtain the web events (as IP addresses and their actions) that "hit" my website. So, to do this 
I needed to obtain the relevant logs from AWS [CloudWatch](https://aws.amazon.com/cloudwatch/) from my ECS cluster. 
Thankfully, ECS creates the relevant log groups and logs in CloudWatch that I can export to Honeycomb for observability
purposes.


1. First login to your Honeycomb account, you can use the link [here](https://ui.honeycomb.io/login)
2. To setup getting logs from CloudWatch to Honeycomb, we will be using be using their [Generic JSON integration for Cloudwatch](https://github.com/honeycombio/agentless-integrations-for-aws#generic-json-integration-for-cloudwatch) as a reference point
3. Following the [Generic JSON integration for Cloudwatch](https://github.com/honeycombio/agentless-integrations-for-aws#generic-json-integration-for-cloudwatch), we will use a AWS Cloudformation integration stack for our logs. Now, following the directions click on [this link](https://console.aws.amazon.com/cloudformation/home#/stacks/new?stackName=honeycomb-cloudwatch-integration&templateURL=https://s3.amazonaws.com/honeycomb-builds/honeycombio/integrations-for-aws/LATEST/templates/cloudwatch-logs-json.yml) to launch the AWS Cloudformation Console to create the integration stack
4. In the **Create Stack** page that appears, keep all the filled-in values and options
5. Click the **Next** button
6. From the *Generic JSON integration for Cloudwatch* directions, you will need to supply the following parameters:
   - Stack Name - For this you can use the default value
   - Cloudwatch Log Group Name
   - Your honeycomb write key (optionally encrypted)
   - Target honeycomb dataset
7. To get the *Cloudwatch Log Group Name* do the following:
   1. In the AWS management console, go to *CloudWatch*
   2. In the left-hand menu, expand the *Logs* menu item
   3. Click on the *Log groups* menu item
   4. In the **Log groups** window, we are looking for the log group nane with the format:
      `/ecs/<name of the task definition`
      And if you used the default value the log group nane you are looking for is:
      `/ecs/first-run-task-definition`
8. To create a Honeycomb dataset do the following:
   