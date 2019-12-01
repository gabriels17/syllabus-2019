# Deployment

## Part 1 - Installing Terraform

> What is Terraform?
>
> Terraform is an open-source infrastructure as code software tool created by HashiCorp. It enables users to define and provision a datacenter infrastructure using a high-level configuration language known as Hashicorp Configuration Language, or optionally JSON.

Follow the instructions at [Install Terraform](https://www.terraform.io/intro/getting-started/install.html).\
You are done when you can run:
```bash
terraform --version
```

## Part 2 - Creating AWS Profile

Setup your AWS Client by logging in to AWS Educate and clicking the `Account Details` and
following the instructions.

<span style="color:red">**IMPORTANT**</span> For some reason an AWS educate session is only 3 hours, so you might need to do this part again if it expires before you finish day 4. None of your automated scripts should take this limitation into account, they can assume that the credentials are valid.

## Part 3 - Creating a Terraform File

Create a file called `infrastructure.tf`:

```hcl-terraform
provider "aws" {
  shared_credentials_file = "~/.aws/credentials"
  region                  = "us-east-1"
}

resource "aws_instance" "game_server" {
  ami           = "ami-0ac019f4fcb7cb7e6"
  instance_type = "t2.micro"
  tags = {
    Name = "GameServer"
  }
}
```

This file describes the infrastructure you want Terraform to create, in this case we are
telling it that we want to use AWS as our provider (one of many cloud providers supported
by Terraform) and where it can find the credentials run commands on AWS.\
Then we tell it to create an `aws_instance` named `game`, the instance is created using
the provided ami (Amazon Machine Image) in this case a machine running Ubuntu 18.04 LTS
of type micro (free-tier machine).

Now run:

```bash
terraform init
```

This will initialize terraform by creating a .terraform directory, since it's computer
generated you should add it to your .gitignore file.

Before you run Terraform to deploy your instance, run the plan command to see the execution
plan for your file.

```bash
terraform plan
```

It should tell you that Terraform will perform the following actions `+ aws_instance.game`,
there is a + on the left side because the instance will be created.

Then deploy your instance using Terraform by:

```bash
terraform apply
```

Terraform will generate a *.tfstate file, you should also add it to .gitignore since it can
contain secrets that do not belong in a public repository.\
The recommended way to keep track of the tfstate file is to store the state remotely that
won't be necessary for this course.\
Keep in mind that this file is used by Terraform to keep track of the resources it is
managing for you so if you accidentally delete it, it will lose track of your existing
resources. If that happens go to the AWS management console and delete the instances
manually then recreate them using Terraform.

Open the AWS Console in your browser and navigate to your EC2 instances, you should now
see your newly created instance there.

## Part 4 - Accessing the Instance

Before we change the Terraform file to include network configurations, go to your AWS
Console and create a new Key-Pair file named GameKeyPair, download it and store it at
`~/.aws/GameKeyPair.pem`. We will configure the instance to be accessible using this
file.

Now update your `infrastructure.tf` file:

```hcl-terraform
provider "aws" {
  shared_credentials_file = "~/.aws/credentials"
  region                  = "us-east-1"
}

resource "aws_security_group" "game_security_group" {
  name = "GameSecurityGroup"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 3000
    to_port     = 3000
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "game_server" {
  ami                    = "ami-0ac019f4fcb7cb7e6"
  instance_type          = "t2.micro"
  key_name               = "GameKeyPair"
  vpc_security_group_ids = [aws_security_group.game_security_group.id]
  tags = {
    Name = "GameServer"
  }

  # This is used to run commands on the instance we just created.
  # Terraform does this by SSHing into the instance and then executing the commands.
  # Since it can take time for the SSH agent on machine to start up we let Terraform
  # handle the retry logic, it will try to connect to the agent until it is available
  # that way we know the instance is available through SSH after Terraform finishes.
  provisioner "remote-exec" {
    inline = [
      "# Managed to SSH into instance",
    ]

    connection {
      host        = coalesce(self.public_ip, self.private_ip)
      type        = "ssh"
      user        = "ubuntu"
      private_key = file("~/.aws/GameKeyPair.pem")
    }
  }
}

output "public_ip" {
  value = aws_instance.game_server.public_ip
}
```

Now run `terraform apply` again and it will destroy our old instance and create a new
one using the above settings.

To get the IP of your new instance you can use:
```bash
terraform output public_ip
```

## Part 5 - Install Dependencies

We've just managed to create a fresh Ubuntu instance, but we still have not managed to
run our API on it. To do that we will create a script that installs all the dependencies
needed to run our app. Since we are using Docker we only need `docker` to run our images
and `docker-compose` to connect them.

`scripts/initialize_game_api_instance.sh`:
```bash
#!/bin/bash

echo 'This script installs everything needed to run our API on the instance'
echo 'and then starts the API.'

echo 'Installing Docker'
sudo apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) \
    stable"
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io
sudo usermod -aG docker ubuntu

echo 'Install Docker Compose'
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# TODO exit 1 if there is no docker-compose.yml file present.

echo 'Starting the API'
sudo docker-compose up -d
```

and `infrastructure.tf`:
```hcl-terraform
# TODO Comment 2-3 sentences.
provider "aws" {
  shared_credentials_file = "~/.aws/credentials"
  region                  = "us-east-1"
}

# TODO Comment 2-3 sentences.
resource "aws_security_group" "game_security_group" {
  name = "GameSecurityGroup"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 3000
    to_port     = 3000
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# TODO Comment 2-3 sentences.
resource "aws_instance" "game_server" {
  ami                    = "ami-0ac019f4fcb7cb7e6"
  instance_type          = "t2.micro"
  key_name               = "GameKeyPair"
  vpc_security_group_ids = [aws_security_group.game_security_group.id]
  tags = {
    Name = "GameServer"
  }

  # TODO Comment 1-2 sentences.
  provisioner "file" {
    source      = "scripts/initialize_game_api_instance.sh"
    destination = "/home/ubuntu/initialize_game_api_instance.sh"

    connection {
      host        = coalesce(self.public_ip, self.private_ip)
      type        = "ssh"
      user        = "ubuntu"
      private_key = file("~/.aws/GameKeyPair.pem")
    }
  }

  # TODO Comment 1-2 sentences.
  provisioner "file" {
    source      = "docker-compose.yml"
    destination = "/home/ubuntu/docker-compose.yml"

    connection {
      host        = coalesce(self.public_ip, self.private_ip)
      type        = "ssh"
      user        = "ubuntu"
      private_key = file("~/.aws/GameKeyPair.pem")
    }
  }

  # This is used to run commands on the instance we just created.
  # Terraform does this by SSHing into the instance and then executing the commands.
  # Since it can take time for the SSH agent on machine to start up we let Terraform
  # handle the retry logic, it will try to connect to the agent until it is available
  # that way we know the instance is available through SSH after Terraform finishes.
  # TODO Comment 1-2 sentences.
  provisioner "remote-exec" {
    inline = [
      "chmod +x /home/ubuntu/initialize_game_api_instance.sh",
    ]

    connection {
      host        = coalesce(self.public_ip, self.private_ip)
      type        = "ssh"
      user        = "ubuntu"
      private_key = file("~/.aws/GameKeyPair.pem")
    }
  }
}

# TODO Comment 1-2 sentences.
output "public_ip" {
  value = aws_instance.game_server.public_ip
}
```
(Added files to copy and changes the remote-exec command)

Apply these changes to Terraform, you may have to use `terraform destroy`, because Terraform
does not take adding a file as reason enough to recreate an instance.

Now we have a fresh instance running and we have copied the `instance_game_api_instance.sh`
initialization script and the `docker-compose.yml` file over to the instance's home directory.

Now you can SSH into your machine using:
```bash
ssh -i "~/.aws/GameKeyPair.pem" ubuntu@$(terraform output public_ip)
```

Now you could run the initialization script while you are logged into your instance, but we want
to do this automatically. You can use:
```bash
ssh -o StrictHostKeyChecking=no -i "~/.aws/GameKeyPair.pem" ubuntu@$(terraform output public_ip) "./initialize_game_api_instance.sh"
```

To SSH into a machine and run a command on it (in this case `./initialize_game_api_instance.sh`).

Verify that the API is running:
```bash
$ curl $(terraform output public_ip):3000/status
The API is running!
```

Now create a script `scripts/deploy.sh`:
1. It should destroy any running Terraform managed instances.
2. Create a new instance using Terraform.
3. Run the initialization script on the new instance.
4. It should run without the user being prompted for approval.

### How do I know I'm done?

Start with no running instances on AWS.\
Run:
```bash
./scripts/deploy.sh
```
It should start a new EC2 instance using Terraform.\
Run:
```bash
curl $(terraform output public_ip):3000/status
```
You should see output:
```text
The API is running!
```

## Part 6 - Comment the Terraform File

Comment the Terraform file in the indicated places.

## Handin

You should store all the source files in your repository:

```bash
├── item_repository
│   ├── app.js
│   ├── database.js
│   ├── Dockerfile
│   └── package.json
├── assignments
│   ├── day01
│   │   └── answers.md
│   └── day02
│       └── answers.md
├── scripts
│   ├── deploy.sh
│   ├── initialize_game_api_instance.sh
│   └── setup_local_dev_environment.sh
├── docker-compose.yml
└── infrastructure.tf
```

They must be placed at these location to get full marks.
