# Day 3 - Provision environment in AWS.

## Objectives

- To be able to run an application on an AWS EC2 instance.

## Step 1 - Setting up a student account
Sign up for a student account on [AWS Educate](https://aws.amazon.com/education/awseducate/)
Step 1: Select Join AWS Educate  
Step 2: Select Student ![Step 2](Step2.png)
Step 3. Fill out the following form ![Step 3](Step3.png)
Step 4. Read the Terms and Conditions ![Step 4](Step4.png)
Step 5. Confirm your email address ![Step 5](Step5.png)  

... Wait for approval (can take 1-2 days)

Step 6. You should get an email with the subject "AWS Educate Application Approved", setup a password for your aws educate account

Step 7. Login to aws educate

Step 8. Go to AWS Account

Step 9. Select "I would like to use an AWS Educate Starter Account" then pres "Get Started" ![Step 9](Step9.png)  

Step 10. Press "Create Starter Account" ![Step 10](Step10.png)  

... Wait while the account is being provisioned

Step 11. Go to your aws educate starter account ![Step 11](Step11.png)  

Step 12. Read the Terms and Conditions

Step 13. You should see ![Step 13](Step13.png)  

## Step 2 - Getting started with Amazon Web Services

> What is AWS?   
> Amazon Web Services (AWS) is the world’s most comprehensive and broadly adopted cloud platform, offering over 165 fully featured services from data centers globally. Millions of customers —including the fastest-growing startups, largest enterprises, and leading government agencies—trust AWS to power their infrastructure, become more agile, and lower costs. From [amazon.com](https://aws.amazon.com/what-is-aws/)

### Create a Key Pair

> What is a Key Pair?   
> Amazon EC2 uses public–key cryptography to encrypt and decrypt login information. Public–key cryptography uses a public key to encrypt a piece of data, and then the recipient uses the private key to decrypt the data. The public and private keys are known as a key pair. Public-key cryptography enables you to securely access your instances using a private key instead of a password.   
> When you launch an instance, you specify the key pair. You can specify an existing key pair or a new key pair that you create at launch. At boot time, the public key content is placed on the instance in an entry within ~/.ssh/authorized_keys. To log in to your instance, you must specify the private key when you connect to the instance. From [https://docs.aws.amazon.com](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)

Set up your AWS account.
From the Vocareum page click on the AWS console.

Now inside the Console go to:\
`Services > EC2 > Network & Security > Key Pairs`\
Create a new Key Pair named `MyKeyPair`, we will use this key later to access our instance.\
So make sure to download it and store it at a safe location (not inside a public repository).\
If you lose it you can not access instances protected with it, but you can create a new key
and a new instance.

### Launching an Instance

> Amazon Elastic Compute Cloud (EC2) is the Amazon Web Service you use to create and run virtual machines in the cloud. AWS calls these virtual machines 'instances'.

Now go to:\
`Services > EC2 > Instances > Instances`\
And launch an instance (use defaults if not specified):\
Select AMI: `Ubuntu Server 18.04 LTS (HVM), SSD Volume Type`\
Instance Type: `t2.micro`\
Name Security Group: `MySecurityGroup`\
Launch with Key Pair: `MyKeyPair`

You should now see a new instance in the console, wait until it's state turns
green.

Right click on your instance and click connect, follow the instructions to connect to your instance.

### Starting a Server
Login to the EC2 instance.
Create a index.html
```
echo "Hello, World" > index.html
```
Run the busybox webserver
```
busybox httpd -f -p 8080
```

Now there is a server running inside your instance listening on port `8080`.\
Check if the server is running on the instance:
```bash
$ curl localhost:8080
Hello, World
```

Now check if the server is accessible through the internet:
```bash
$ curl http://<EC2_INSTANCE_PUBLIC_IP>:8080
# Command will timeout
```

The reason you can not connect to your instance is that port `8080` is not open, so the
request is not reaching the server. By default only port `22` is open since you need that
to manage your instance via SSH.

> What are AWS security groups?   
> AWS security groups (SGs) are associated with EC2 instances and provide security at the protocol and port access level. Each security group — working much the same way as a firewall — contains a set of rules that filter traffic coming into and out of an EC2 instance. Unlike network access control lists (NACLs), there are no “Deny” rules. If there is no rule that explicitly permits a particular data packet, it will be dropped. From [CloudAcademy](https://cloudacademy.com/blog/aws-security-groups-instance-level-security/)

Now to open a port, go to:\
`Services > EC2 > Network & Security > Security Groups`\
Edit the security group for your instance, you need to add an inbound TCP rule to
allow traffic through port `8080` from anywhere.

Check if the server is running (this should output "Hello, World")
```
$ curl http://<EC2_INSTANCE_PUBLIC_IP>:8080
Hello, World
```

## Step 4 - Terminate instance

Terminate the instance you just created.

## Step 5 - Install AWS CLI version 1

[Install AWS Cli version 1](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv1.html), once you are
done verify using:
```bash
aws --version
```

Also remember to add `aws` to the setup script you created in day 1.

### How do I know I'm done?

- [ ] I've created an account on AWS
- [ ] I have one AWS instance running
- [ ] I have a simple webserver running
- [ ] I can visit my AWS site using its public DNS, example
      `http://ec2-SOME-NUMBERS.us-west-2.compute.amazonaws.com/` and see my
      application running
- [ ] I have terminated the instance.
- [ ] I have AWS cli installed.
