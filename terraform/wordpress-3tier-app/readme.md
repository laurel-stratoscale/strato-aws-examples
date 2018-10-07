# Wordpress Example

This example shows you how to use Terraform to create a 3-tier application environment for Wordpress within a Symphony region.

## What this example deploys

**Networks**

1 VPC

1 Database subnet (AWS has a requirement that the db_subnet_group has two subnets in two availability zones)

1 Web subnet

1 Public subnet

**Other resources**

1 Bastion jump server

1 load balancer

2 or more web servers (based on Ubuntu Xenial)

1 RDS instance (MySQL 5.7)

2 elastic IPs from the Symphony edge network

## Before you begin

Before you can use this Terraform example, you need to:

* First, do some setup tasks within Symphony.

* Then, edit the sample `terraform.tfvars` file to specify your environment-specific values for various variables.

Each task is described below.


### Before you begin: Symphony setup tasks

Before you can use this Terraform example, you need to do the following tasks within the Symphony GUI:

1. Make sure you have used Symphony to:

    * Create a **VPC-enabled project**

    * Obtain **access and secret keys** for that project

   For information on how to do these tasks, click [here](../README.md). 

2. **Upload the image** you will use for your web server(s) into Symphony:

    Use the Ubuntu Xenial cloud image at this URL:
    
    https://cloud-images.ubuntu.com/xenial/current/xenial-server-cloudimg-amd64-disk1.img
    
    To upload the image into Symphony:
    
    **Menu** > **Applications** > **Images** > **Create** > specify URL
    
    After you upload the image, set its Scope to Public:
    
    **Menu** > **Applications** > **Images** > Select the image > **Modify** > set the **Scope** field to Public
    
    
3. Get the **AMI ID** for the image you just uploaded into Symphony:

    **Menu** > **Applications** > **Images**
    
    Click the name of the image you just uploaded and copy the AWS ID value -- it has a format like this:
    
    ami-1b8ecb82893a4d1f9d500ce33d90496c
    
4. Enable load balancing:

    **Menu** > **Region Management** > **Settings** > **Services & Support** > click Load Balancing toggle to ON.
    
5. Enable and initialize RDS using a MySQL 5.7 engine:

    **Menu** > **Region Management** > **Settings** > **Services & Support** > click Database toggle to ON.
    
    **Menu** > **Database** > **Engines** > MySQL 5.7.00 > toggle **Enabled** switch to ON.
    
    
### Before you begin: edit `terraform.tfvars`

Use the included `terraform.tfvars` file as a template. For each variable, fill in your environment-specific value, as described below:

_BASIC VARIABLES_

The following variables: **symphony_ip**, **access_key**, **secret_key** are described [here](../ec2-instance/README.md).

_WEB SERVER VARIABLES_

**web_number**

Number of web servers (load balancer will automatically manage target groups).

**web_ami**

The AMI ID for the Xenial image you jsut uploaded into Symphony. It has a format like this:
    
ami-1b8ecb82893a4d1f9d500ce33d90496c

**web_instance_type**

The instance type you want to use for your web server(s). Example: t2.medium

**public_keypair_path**

The path to the public key file that Terraform will pass to Symphony for authentication.

Background: You can generate a keypair using a tool such as `ssh-keygen`. Place the public and private keys on the machine on which you are running Terraform, for example:

Assume you put the public key file here: `/path/to/myKey.pub`

And you put the private key file here: `/path/to/myPrivateKey`

In this example, you would set the `public_keypair_path` variable to `/path/to/myKey.pub`.

_DATABASE VARIABLES_

_Note that the Wordpress container uses the Wordpress database by default._

**db_password**

Database password.

**db_user**

Database user.

## How to use

1. Install the most recent version of Terraform.

2. Run `terraform init`.

3. Run `terraform apply`. Take a look at the `terraform apply`output and note that it includes the elastic IP of the load balancer (`lb_eip`). Make a note of this load balancer EIP, because you will need it to access Wordpress.

4. Point your browser to Wordpress at this location:

    `http://<load_balancer_eip>:80`

    Initially, you will see the Wordpress installation and configuration pages.

    **How to get the <load_balancer_eip>**:

    * This load balancer EIP was part of the output from `terraform apply`. You can always redisplay that output by running `terraform refresh`.

    * You can also get the load balancer EIP from the Symphony GUI:

        **Menu** > **Load Balancing** > **Load Balancers** > select the load balancer that Terraform just created.

        The field marked **Floating IP** contains the load balancer EIP that you need to access Wordpress.
        
### SSH access to web servers

There may be times when you want to SSH into the web servers that are part of your Wordpress deployment. This can be for troubleshooting or other purposes:

**To SSH into a web server**

1. Run `terraform refresh` to get the following IPs:

    * Bastion Elastic IP
    
    * Private IPs of the web servers
    
  Sample output from `terraform refresh`:
    
  ```
  Bastion Elastic IP = 10.45.58.106
  lb_eip = 10.45.58.109
  web-server private ips = {
   i-5f7345e70753440ab40ae144c688aa11 = 192.168.20.11
   i-603e5000d9774651aec56d67759a2679 = 192.168.20.14
   i-94776cf02c234a2ab894d9f0ce1ac4a5 = 192.168.20.6
  ```
  
2. Use `ssh-add -K` to store your private key in a keychain, for example:
   
   `ssh-add -K ~/path/to/MyPrivateKey`
   
3. You can now SSH into a web server through the Bastion instance, using the following `ssh -J` syntax:

   `ssh -J username@<ip_of_bastion_server> username@<private_ip_of_web_server>`
   
   Example. This example assumes that you used the recommended Xenial image, which has a username of `ubuntu`. It uses values from the sample `terraform refresh` output above:
    
   `ssh -J ubuntu@10.45.58.106 ubuntu@192.168.20.11`
   
