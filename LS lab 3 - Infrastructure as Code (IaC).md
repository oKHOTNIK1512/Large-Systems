---
tags: Large Systems
---
:::success
# LS lab 3 - Infrastructure as Code (IaC)
Name: Ivan Okhotnikov
:::

## Task 1 - IaC Theory
:::warning
Briefly answer for the following questions what is and for what:
- **git repository:**
`/.git`
`/.github`
`.gitignore`
`.gitmodules`

- **ansible directory:**
`ansible.cfg`
`inventory folder`
`roles folder`
`tasks`
`defaults`
`files`
`handlers`
`templates`
`playbooks folder`

- **terraform folder:**
`main.tf`
`variables.tf`
`outputs.tf`
:::

## Implementation:
:::info
> - **git repository:**

`/.git` - the directory that contains all the information about the repository. Initialized using the `git init` command
`/.github` - a directory that stores additional data for the Github version control system (usually configurations for pipelines are stored there)
`.gitignore` - a file that lists files and directories whose changes do not need to be tracked
`.gitmodules` - This is a configuration file that stores the correspondence between the URL of the project and the local one under the directory to which you downloaded it

> - **ansible directory:**

`ansible.cfg` - ansible configuration file
`inventory folder` - directory for storing managed nodes
`roles folder` - it is necessary to simplify the reuse of yaml files, usually consists of several folders
`tasks` - yaml files with instructions for execution
`defaults` - default configuration file
`files` - all additional files that are needed to perform tasks
`handlers` - tasks that are performed when a notification is received
`templates` - contains templates that are used to create configuration files
`playbooks folder` - a folder that contains instructions for multiple execution on nodes (in yaml format)

> - **terraform folder:**

`main.tf` - the main terraform file. It describes all the modules and actions that need to be performed
`variables.tf` - the file in which variables are initialized for use in the main file
`outputs.tf` - file containing initialization of output variables

:::


## Task 2 - Choose your application/microservices
:::warning
Base level (it means that this task will be evaluated very meticulously): find (much better to write) a simple application with REST interface. For example, it could be a web server that returns "Hello to SNE family! Time is: < current time in Innopolis >". Use whatever programming language that you want (python, golang, C#, java...).
Semi-Bonus: prepare microservices instead of standalone application, e.g. full stack web application with web server, database...
:::

## Implementation:
:::info
> Base level (it means that this task will be evaluated very meticulously): find (much better to write) a simple application with REST interface. For example, it could be a web server that returns "Hello to SNE family! Time is: < current time in Innopolis >". Use whatever programming language that you want (python, golang, C#, java...).

> Semi-Bonus: prepare microservices instead of standalone application, e.g. full stack web application with web server, database...


I wrote a primitive application that allows you to create/delete and display a list of users
Figure 1-3 shows the process of working with the rest interface for working with my application
All users are stored in a mysql database, the application is written in the php programming language, and nginx is responsible for the web server

<center>

![](https://i.imgur.com/VvWYtJl.png)
Figure 1: Creating a user
</center>

<center>

![](https://i.imgur.com/LPYAXt7.png)
Figure 2: Display a list of users
</center>

<center>

![](https://i.imgur.com/RtwlMqR.png)
Figure 3: Deleting a user by his ID
</center>

[My github repository is here](https://github.com/IvanHunters/large-systems-application)
:::


## Task 3 - Dockerize your application
:::warning
1. Build Docker image for your application (make Dockerfile).
Look for the best Docker practices, try to follow them and put into report.
Bonus: use docker-compose in the case of microservices.
:::

## Implementation:
:::info
> 1. Build Docker image for your application (make Dockerfile).
Look for the best Docker practices, try to follow them and put into report.

I created a custom php 7.4-fpm image that will copy the project from the project directory, install the necessary programs (such as composer, php7.4-mysql, etc.)

Also in the image, I prepare the application for launch and clean up unnecessary apt dependencies

<center>

![](https://i.imgur.com/BZMgUI4.png)
Figure 4: Dockerfile content
</center>

> Bonus: use docker-compose in the case of microservices.
For this task, I wrote a docker-compose file in which I used variables from the .env file

Variables describe which ports to open for nginx and phpmyadmin, as well as the user name,
root password and base user, database to create in mysql


<center>

![](https://i.imgur.com/ZKYTEGU.png)
Figure 5: Part of the docker-compose file, which can be found and [viewed here](https://github.com/IvanHunters/large-systems-application/blob/main/docker-compose.yml)
</center>


:::


## Task 4 - Deliver your app using Software Configuration Management
:::warning
1. Get your personal cloud account. To avoid some payment troubles, take a look on free tier that e.g. AWS or GCP offers for you (this account type should be fit for a lab).

2. Use Terraform to deploy your required cloud instance.
Look for the best Terraform practices, try to follow them and put into report.
Bonus: use Packer to deploy your cloud image/docker container.

3. Choose SCM tool. Suggestions:
Ansible (default choice)
SaltStack
Puppet
Chef
...


4. Using SCM tool, write a playbook tasks to deliver your application to cloud instance. Try to separate your configuration files into inventory/roles/playbooks files. In real practice, we almost newer use poor playbooks where everything all in one.
Also try to use the best practices for you SCM tool and put them into report.
Use Molecula and Ansible Lint to test your application before to deliver to cloud (or their equivalents).
:::

## Implementation:
:::info
> 1. Get your personal cloud account. To avoid some payment troubles, take a look on free tier that e.g. AWS or GCP offers for you (this account type should be fit for a lab).

I have created an account in the Google Cloud service
To start working with it via IaC, I will need to create access
To do this, I logged into the Google Cloud Admin console

<center>

![](https://i.imgur.com/S7TpmR4.png)
Figure 6: Google Cloud Admin Console
</center>

It remains to create access

Then I went to the "IAM & Admin -> Service Accounts" tab

<center>

![](https://i.imgur.com/OwV1tRg.png)
Figure 7: Service Accounts section window
</center>

To create a service account, I click on the create account button

<center>

![](https://i.imgur.com/1JpScYV.png)
Figure 8: Window for creating a new account
</center>

I specify the account name

I click `CREATE AND CONTINUE`

I specify the `OWNER` role so that the account has all the necessary access
<center>

![](https://i.imgur.com/aH3vJaO.png)
Figure 8: Window for creating a new account
</center>


Now I need to go to the settings of the created account and create keys to work from under this account

<center>

![](https://i.imgur.com/wFwdQib.png)
Figure 9: Window for editing service account settings
</center>

And create a key in JSON format (after which it will be saved on my computer)

This completes the preparatory work with Google Cloud

> 2. Use Terraform to deploy your required cloud instance.
Look for the best Terraform practices, try to follow them and put into report.

I decided to organize work with terraforms through several files:
- The main file (`main.tf `), which specifies the json file that I downloaded earlier, the file with the connection of variables (`variables.tf `), which is necessary because variables cannot be used just like that from the file (`gcp.tfvars`)

This approach will help to add security, because in gcp.tfvars I will store my ssh key, which I will export to `google cloud` and other variables (which will allow in the future to simply add the `gcp.tfvars` file to `.gitignore` and use the phablon file)

File `main.tf`

```bash=1
provider "google" {
  credentials = file(var.credential_file)
  project = var.project
  region = var.region
  zone = var.zone
}

resource "google_compute_instance" "ubuntu" {
  name = var.instance_name
    network_interface {
      network = "default"
      # pin public_ip
      access_config {
        network_tier = "PREMIUM"
      }
   }
   # set image name and size of disk
   boot_disk {
      initialize_params {
         image = var.image
         size = var.disk_size
      }
   }
   # machine type
   machine_type = var.machine_type
}

# works with firewall
resource "google_compute_firewall" "rules" {
  network     = "default"
  name = var.firewall_rules_name

  allow {
    protocol  = "tcp"
    # open ip addresses for my services
    ports     = var.firewall_opened_tcp_ports
  }
}
# set ssh-key for external connections to my instance
resource "google_compute_project_metadata" "default" {
  metadata = {
    ssh-keys = "${var.ssh_username}:${file(var.ssh_filename)}"
  }

}
```

File `variables.tf`

```bash=1
variable "credential_file" {
  type = string
}
variable "project" {
  type = string
}
variable "instance_name" {
  type = string
}
variable "region" {
  type = string
}
variable "zone" {
  type = string
}
variable "image" {
  type = string
}
variable "machine_type" {
  type = string
}
variable "disk_size" {
  type = string
}
variable "firewall_rules_name" {
  type = string
}
variable "firewall_opened_tcp_ports" {
  type = list(string)
}
variable "ssh_username" {
  type = string
}
variable "ssh_filename" {
  type = string
}
```

File `output.tf`
Because I want to see in the output of the command the `ip address` to connect to my virtual machine, so as not to enter the web interface
```bash=1
output "ip" {
  value = google_compute_instance.ubuntu.network_interface.0.access_config.0.nat_ip
}
```


File `gcp.tfvars`

```bash=1
credential_file = "eastern-bridge-340402-4d03190ce472.json" #my credential file
project = "eastern-bridge-340402" # locate in the main page of the google cloud console
instance_name = "my-application"
region = "us-central1"
zone = "us-central1-a"
image = "packer-1644153323" #for bonus part
machine_type = "e2-small"
disk_size = "30"
firewall_rules_name = "open-ports-for-application"
firewall_opened_tcp_ports = ["7070", "8085"]
ssh_username = "ohotbjgftru" #my username for google cloud
ssh_filename = "/home/st11/.ssh/id_rsa.pub"
```

I can start the deployment with the command:
```bash=1
terraform apply -var-file="gcp.tfvars"
```
Moreover, the flag is required when using external variables, which will allow you to have several such configurations in the future, which is very convenient

<center>

![](https://i.imgur.com/FRnvLzo.png)
Figure 10: Part of the output of the terraform program (since I asked to output the ip address for an external machine to me, it is now in the output, which is convenient)
</center>

I can also work with the output data from terraform, this is done through the command. This can be used to automatically create a host in `/etc/ansible/hosts`

```bash=1
terraform output <output variable>
```

<!-- 
```bash=1
ssh-keygen -f "/home/st11/.ssh/known_hosts" -R "$(terraform output ip_ubuntu | sed 's/"//g')"
ssh-keygen -f "/home/st11/.ssh/known_hosts" -R "$(terraform output ip_fedora | sed 's/"//g')"
sudo sh  -c 'cat /etc/ansible/hosts.bak | sed "s/\*\*SERVER_HOST\*\*/$(terraform output ip)/g" > /etc/ansible/hosts'
```
 -->

> Bonus: use Packer to deploy your cloud image/docker container.

`Packer` is a convenient tool for creating your own images based on existing ones with the addition of your initial settings, installations, etc.

Below is a file with the settings I need
This:
- Installing the latest version of docker + docker compose

File `packer.json`

```json=1
{
     "builders": [{
         "type": "googlecompute",
         "account_file": "../terraform/eastern-bridge-340402-4d03190ce472.json",
         "project_id": "eastern-bridge-340402",
         "source_image": "ubuntu-1604-xenial-v20180405",
         "ssh_username": "packer",
         "zone": "europe-central2-a"

     }],
     "provisioners": [{
         "type": "shell",
         "inline": [
             "sleep 30",
             "sudo apt-get update",
             "sudo apt install -y  apt-transport-https ca-certificates curl software-properties-common",
             "wget https://download.docker.com/linux/ubuntu/gpg -O gpg",
             "sudo apt-key add gpg",
             "sudo add-apt-repository \"deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable\"",
             "rm gpg",
             "sudo apt update",
             "sudo apt install -y docker-ce",
             "sudo curl -L \"https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)\" -o /usr/local/bin/docker-compose",
             "sudo chmod +x /usr/local/bin/docker-compose"
         ]
     }]
}
```

I can create an image with the command `packer build packer.json`

During execution, `packer` will create an instance inside google cloud, execute commands inside it, and then create an image based on the virtual machine, followed by deleting the created virtual machine

The packer output will contain the installation process and the final image_id, based on which I can create my virtual machine (as I used earlier in terraform)

<center>

![](https://i.imgur.com/n7AVm8C.png)
Figure 11: Packer output
</center>

>3. Choose SCM tool. Suggestions:
>>Ansible (default choice)
SaltStack
Puppet
Chef


I will use Ansible

> 4. Using SCM tool, write a playbook tasks to deliver your application to cloud instance. Try to separate your configuration files into inventory/roles/playbooks files. In real practice, we almost newer use poor playbooks where everything all in one.
>>Also try to use the best practices for you SCM tool and put them into report.
Use Molecula and Ansible Lint to test your application before to deliver to cloud (or their equivalents).

...


Ansible is a tool for automating the configuration of virtual machines based on ssh connection
You just need to specify their public ip addresses in the file `/etc/ansible/hosts` (first you need to have the public ssh key already on the server inside the file `/home/user/.ssh/authorized_keys`)

I broke the playbooks into several parts:
1 - initializing the application (git clone, docker volume create, copy .end.example to .env)
2 - launching the application (docker compose build/up)
3 - launching migrations for the database from inside my php container
4 - shutting down the application, cleaning docker and deleting the project folder

File `init_application.yaml`

```yaml=1
---
- name: Initialization before launching the application
  hosts: all
  user: ohotbjgftru
  tasks:          
      - name: "Clonning new git repository.."
        command: "git clone https://github.com/IvanHunters/large-systems-application.git"
        
      - name: "Creating docker volume..."
        command: "docker volume create --name=database"
        become: yes
        become_method: sudo
        args:
          warn: false
        
      - name: "Copy environments file from template"
        command: "cp .env.example .env"
        args:
          chdir: "large-systems-application"
```

File `run_docker_application.yaml`

```yaml=1
---
- name: Running application
  hosts: all
  user: ohotbjgftru
  tasks:
      - name: "Building application..."
        command: "docker-compose build"
        args:
          chdir: "large-systems-application"
        become: yes
        become_method: sudo

      - name: "Running application..."
        command: "docker-compose up -d"
        args:
          chdir: "large-systems-application"
        become: yes
        become_method: sudo        
```

File `run_database_migration.yaml`

```yaml=1
---
- name: Runing db migrations
  hosts: all
  user: ohotbjgftru
  tasks:
      - name: "Waiting 1 minutes before will starting all services..."
        command: "sleep 1m"
        
      - name: "Running db migrations..."
        command: "docker exec -it app_php php artisan migrate"
        become: yes
        become_method: sudo
```

File `remove_application.yaml`

```yaml=1
---
- name: Initialization before launching the application
  hosts: all
  user: ohotbjgftru
  tasks:
      - name: "Stopping services if exists"
        command: "docker-compose stop"
        args:
          chdir: "large-systems-application"
        become: yes
        become_user: root
        
      - name: "Removing containers of services if exists"
        command: bash -c "yes | docker-compose rm -a"
        args:
          chdir: "large-systems-application"
        become: yes
        become_user: root
        
      - name: "Removing old docker volumes if exists..."
        command: "docker volume rm database"
        become: yes
        become_user: root
        args:
          warn: false
        
      - name: "Removing old project dir if exists..."
        command: "rm -rf large-systems-application"
        args:
          warn: false
```



<center>

![](https://i.imgur.com/JHy0BUq.png)
Figure 12: Starting the initialization of the project and its launch
</center>

<center>

![](https://i.imgur.com/ufhHkhA.png)
Figure 13: Starting database migration (so that the project can work)
</center>



After completing all the necessary steps, I can make sure that the application works as needed (by sending REST requests) and opening phpMyAdmin

<center>

![](https://i.imgur.com/Ymyyn3M.png)
Figure 14: Request to create a user
</center>

<center>

![](https://i.imgur.com/IyOuuJ9.png)
Figure 15: Request to withdraw all users
</center>

<center>

![](https://i.imgur.com/Wp5vWkr.png)
Figure 16: Request to delete a user
</center>

<center>

![](https://i.imgur.com/ydJs9eC.png)
Figure 17: phpMyAdmin Page
</center>
:::



## Task 5 - Teamwork with the version control system (optional)
:::warning
You can do this task if you have a different projects with your cooperation team and you want to be
more familiar with git.
In production, before deploying the version of the service/application, we receive a review from
colleagues.
1. Create and log in to your personal version control system account on the git engine:
github (default choice)
gitlab
bitbucket
...
2. Create a repository with your application/microservices and all required scripts/configs.
3. Synchronize your local and remote repository.
4. Create a separate branch for development, as well as protect your master branch in the repository from direct commits to it.
5. Create a Pull Request from your developer branch to the master branch.
6. Your colleague should get explanation about your work and conduct a review of your PR.
7. Receive an approvement, merge your PR and synchronize the local and remote repositories.
Bonus: implement steps 2 and 4 via Terraform.
:::

## Implementation:
:::info
>1. Create and log in to your personal version control system account on the git engine:
github (default choice)
gitlab
bitbucket

For this task I will use Github
First of all, I will need to create a token for my account with the rights as below (Figure 18)

<center>

![](https://i.imgur.com/5jhvfOS.png)
Figure 18: The form of creating a personal token
</center>

> 2. Create a repository with your application/microservices and all required scripts/configs.
> 3. Synchronize your local and remote repository.
> 4. Create a separate branch for development, as well as protect your master branch in the repository from direct commits to it.

To create a repository, branches, and main branch constraints, I wrote the following terraform configuration file:

```json=1
provider "github" {
	token = "my_token"
}

resource "github_repository" "default" {
	name        = "ls-project"
	description = "My awesome ls project"
	visibility  = "public"
	auto_init   = true
}


resource "github_branch_default" "default"{
	repository = github_repository.default.name
	branch     = "main"
}

resource "github_repository_file" "default" {
	repository          = github_repository.default.name
	branch              = "main"
	file                = ".gitignore"
	content             = "**/*.tfstate"
	commit_message      = "Create file using terraform"
	commit_author       = "IvanHunters"
	commit_email        = "ivdev@volsu.ru"
	overwrite_on_create = true
}


resource "github_branch" "development" {
	repository = github_repository.default.name
	branch     = "development"
}

resource "github_repository_collaborator" "Ilya" {
    repository = github_repository.default.name
    username   = "Tealdris"
    permission       = "admin"
}

resource "github_repository_collaborator" "Vladimir" {
    repository = github_repository.default.name
    username   = "mrRahmat"
    permission       = "admin"
}


resource "github_branch_protection" "default" {
  repository_id  = github_repository.default.name
  pattern          = "main"
  enforce_admins   = true
   required_pull_request_reviews {
    dismiss_stale_reviews  = true
    restrict_dismissals    = true
   }
 }
```

In it I indicated:
In the resource 'github_repository', the name, description of my repository, indicated that the repository would be open and asked to do the initial initialization
In the resource `github_branch_default` I indicated that the `main` branch will be the main one
In the resource `github_branch` I added a new branch - `development`
In the `github_repository_collaborator` resource I added Ilya and Vladimir to the list of maintainers of the project
And also in conclusion, I added the resource `github_branch_protection` to protect my main branch (so that it was impossible to push into it just like that and the merge was carried out after the pull request)
After terraforming I can make sure everything works

<center>

![](https://i.imgur.com/Vfombpo.png)
Figure 19: Rules for protecting the `main` branch
</center>
<center>

![](https://i.imgur.com/w9TTsQH.png)
Figure 20: List of added mainteiners
</center>


> 5. Create a Pull Request from your developer branch to the master branch.
> 6. Your colleague should get explanation about your work and conduct a review of your PR.
> 7. Receive an approvement, merge your PR and synchronize the local and remote repositories.

Then I asked Ilya to push to the `development` branch and make a request to merge with the main branch (figure 21)

<center>

![](https://i.imgur.com/JlwdjRF.png)
Figure 21: Pull request from Ilya
</center>

After the appruv of his request, the page will look like this:
<center>

![](https://i.imgur.com/xEvbUrQ.png)
Figure 22: Completed pull request
</center>

Then I asked Vladimir to make his edits to the `development` branch and send a pull request

<center>

![](https://i.imgur.com/lFLmkk6.png)
Figure 23: Pull-request from Vladimir
</center>

Then I added a comment (so that Vladimir would make more edits) (figure 24 - 25)
<center>

![](https://i.imgur.com/Fj1CwXP.png)
Figure 24: Pull-request from Vladimir
</center>
<center>

![](https://i.imgur.com/XhbqoTv.png)
Figure 25: Pull-request from Vladimir
</center>

And after successfully adding edits (new pushs are added to PR automatically) I have allowed PR

<center>

![](https://i.imgur.com/fEPMaV7.png)
Figure 26: PR Resolution
</center>


:::

## Task 6 - Play with SCM cases
:::warning
The goal is to easily, reliably and quickly maintain different kinds of systems at once. Prepare two different guest systems e.g. an Ubuntu Server and Fedora. Then play with some Software
Configuration Management (SCM) to automate system preparations on those. You might create and provide your own ideas or follow to some suggestions:
NTP
default shell
Apache Web Server
Docker
Nginx & PHP-FPM & MariaDB & WordPress
Ansible Vault (definitely good choice)
IPtables (close all machine ports exclude ssh, http, https)
development environments organization (ssh keys management...)
...
Never forget to try to use the best practices. Complete as many tasks as possible (at least two cases for a team of two people, one case for an individual work).
:::

## Implementation:
:::info

For my task: installing docker, it would be a good tone to create an image using packer and execute the commands below, but in the SCM task, there will be SCM

File `prepare_centos.yaml`
In it, I will forcibly specify the name of the host that I specified for 'centos`

```yaml=1
---
- name: Initialization before launching the application
  hosts: centos
  user: ohotbjgftru
  tasks:
    - name: "Install yum-utils and git"
      command: "yum install -y yum-utils git wget"
      become: true
      become_method: sudo

    - name: "Add repository for latest docker"
      command: " yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo"
      become: true
      become_method: sudo

    - name: "Install latest docker"
      command: " yum install -y docker-ce docker-ce-cli containerd.io"
      become: true
      become_method: sudo

    - name: "Run docker service"
      command: " service docker start"
      become: true
      become_method: sudo

    - name: "Download latest docker-compose"
      command: bash -c "wget  \"https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)\" -O /usr/local/bin/docker-compose"
      become: true
      become_method: sudo
        
    - name: "Install latest docker-compose"
      command: " chmod +x /usr/local/bin/docker-compose"
      become: true
      become_method: sudo

    - name: "Copy in bin dir"
      command: " ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose"
      become: true
      become_method: sudo
```

It is also important to note that the previously existing connection is not suitable, so I specified a separate `python_interpretator` for it

```bash=1
[servers]
centos ansible_host="34.121.100.31"
ubuntu ansible_host="34.121.86.88"

[all:vars]
ubuntu ansible_python_interpreter=/bin/python3
centos ansible_python_interpreter=/usr/libexec/platform-python
```

Now, to deploy my application to several environments, it's enough just to run ansible-playbook in such a sequence:

```bash=1
ansible-playbook prepare_centos.yaml init_application.yaml run_docker_application.yaml run_database_migration.yaml
```

:::


## References:
:::warning
1. [Git documentation](https://git-scm.com/doc)
2. [Ansible documentation](https://docs.ansible.com)
3. [Terraform Documentation](https://www.terraform.io/docs)
4. [Deploying Microservices with Docker Compose](https://medium.com/devplanet/deploying-microservices-with-docker-compose-1aff3d8af9a5)
5. [My app](https://github.com/MrRahmat/react-nodejs-rest-docker)
6. [Best practices for writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
7. [Terraform. Get Started](https://learn.hashicorp.com/collections/terraform)
10. [Install Packer](https://learn.hashicorp.com/tutorials/packer/get-started-install-cli)
11. [Ansible playbook example](https://github.com/zalari/ansible-deploy-docker-compose/blob/master/tasks/main.yml)
13. [Introducing review requests](https://github.blog/2016-12-07-introducing-review-requests/)
14. [Managing GitHub with Terraform](https://www.hashicorp.com/blog/managing-github-with-terraform)
15. [Terraform & GitHub](https://terraform-eap.website.yandexcloud.net/docs/providers/github/index.html)
::