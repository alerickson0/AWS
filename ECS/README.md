# ECS personal project

This is my personal project to learn how to use AWS ECS and Honeycomb. I wanted to see how I could 
upload and use a Docker image I created to be used in [ECS](https://aws.amazon.com/ecs/). Then, I 
wanted to see how I could export my CloudWatch logs from ECS to [Honeycomb](https://www.honeycomb.io/). 
There I could create reports and graphs of the logs.

Please note, if you want to see the logs in Honeycomb. You will have to use an *nginx* or similar image.

## Pre-requisites
- Have a Honeycomb account. If you do not have one, you can sign up for a free one on their site [here](https://ui.honeycomb.io/signup).
- Have an AWS account that has permissions to create: ECS resources, Cloudformation stacks, and IAM roles and resources

## Steps to using a Docker image in ECS
1. Login to the [AWS management console](https://aws.amazon.com/console/) using an account that has access to ECS
2. Navigate to Amazon ECS
3. We are going to use the ECS wizard, so click on the **Get Started** button
4. For *Container Image* you can either use the *nginx* image option or you can use one of your own. I used one of 
my own and to do this go to step 5. Other wise go to step 11
5. To use your own image, click on the **Configure** button for the *custom* container definition
6. For **Container name**, I used *my-web-app*
7. For **Image**, this is where you can pull an image from Docker Hub. And to do this the format is similar to the 
`Docker pull` command: `<account>/<repo name>:<image name>`
8. Leave all other settings as is
9. Set *Port mappings* to 80
10. Click on the **Update** button to finish up
11. Click on the **Next** button
12. On the **Define your service** page, we will need a load balancer. So, for *Load balancer type* select 
*Application Load Balancer*
13. Click on the **Next** button
14. \(Optional\) On the **Configure your cluster** page, update the *Cluster name* to *my-web-app-cluster*
15. Click on the **Next** button
16. Finally, on the **Review** page, to create the Cluster click on the **Create** button. Now, you will have 
to wait a few minutes for the Cluster and its resources to be created.
17. Once, the ECS service has been created click on the **View Service** button

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
3. Following the [Regex integration for Cloudwatch](https://docs.honeycomb.io/getting-data-in/integrations/aws/aws-cloudwatch-logs/#regex-integration), we will use a AWS Cloudformation integration stack for our logs. Now, following the directions click on [this link](https://us-west-2.console.aws.amazon.com/cloudformation/home?region=us-west-2#/stacks/create/template?stackName=honeycomb-cloudwatch-integration&templateURL=https://s3.amazonaws.com/honeycomb-builds/honeycombio/integrations-for-aws/LATEST/templates/cloudwatch-logs-regex.yml) to launch the AWS Cloudformation Console to create the integration stack. Note: This launches CloudFormation template in the us-west-2/Oregon region
4. In the **Create Stack** page that appears, keep all the filled-in values and options
5. Click the **Next** button
6. From the *Generic JSON integration for Cloudwatch* directions, you will need to supply the following parameters:
   - Stack Name - For this you can use the default value
   - Cloudwatch Log Group Name (you can supply up to 6 per installation)
   - Your honeycomb write key (optionally encrypted)
   - Target honeycomb dataset
   - re2 compatible regex pattern
7. To get the *Cloudwatch Log Group Name* do the following:
   1. In the AWS management console, go to *CloudWatch*
   2. In the left-hand menu, expand the *Logs* menu item
   3. Click on the *Log groups* menu item
   4. In the **Log groups** window, we are looking for the log group nane with the format:
      `/ecs/<name of the task definition`
      And if you used the default value the log group nane you are looking for is:
      `/ecs/first-run-task-definition`
8. (Optional steps) It's best to encrypt your honeycomb write key, to do this follow these steps as provided from this [article](https://github.com/honeycombio/agentless-integrations-for-aws#encrypting-your-write-key)
   1. In a command line/shell, you will need to create a KMS key using the AWS's KMS service using the following command:
      `aws kms create-key --description "used to encrypt secrets"`
   2. (Optional) You can create an alias for the key you created using the following command:
      `aws kms create-alias --alias-name alias/secrets_key --target-key-id=<key id>`
   3. Next, create a file containing your Honeycomb API key that will be passed into the encryption step. For example, 
   you can use the `echo` command
   4. Next, encrypt the API key using the following command:
      `aws kms encrypt --key-id=<KMS key ID> --plaintext fileb://path/to/file/my-key`
      Where *my-key* is the name of the file containing your API key
   5. Finally, record the `CiphertextBlob` and the last part of the KMS Key ID (example: `b46g70cc-19b5-486a-a163-a4502b7a52cc`).
   Use the `CiphertextBlob` for your *HoneycombWriteKey* in the Cloudformation stack
9. For the *HoneycombWriteKey*, enter either your Honeycomb API key or `CiphertextBlob` (if you encrypted your API key)
10. (Optional) For the *KMSKeyId*, enter the KMS Key ID you created for encryption
11. Now, to get the web events/traffic to Honeycomb, in the *RegexPattern* field, enter the regex pattern I created which is [here](cloudwatch_to_honeycomb_regex_pattern.txt)
12. Click the **Next** button
13. (Optional) Entter any relevant tag or tags in the *Tags* section
14. Going to use the rest of the defaults, so clikc the **Next** button
15. Click on the checkmark box acknowledging that AWS CloudFormation might create IAM resources in the **Capabilities** section
16. Click **Create Stack** to create the CloudFormation stack. Note: This will take quite a few minutes for the stack to be created
17. Now, when you go to your Honeycomb account, you should a *cloudwatch-logs* dataset

## For the future
In the future, I will detail more about:
- What data is being imported into Honeycomb
- What queries you can against the imported data in Honeycomb