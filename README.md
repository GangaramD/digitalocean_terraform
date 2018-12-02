# Digitalocean_terraform


# Setps to install Terraform:
1. sudo apt-get -y install / sudo yum install -y zip unzip (if these are not installed)
2. wget https://releases.hashicorp.com/terraform/0.9.8/terraform_0.9.8_linux_amd64.zip
3. unzip terraform_0.9.8_linux_amd64.zip
4. sudo mv terraform /usr/local/bin/
5. Confirm terraform binary is accessible: terraform --version


# Step to create Droplet and install Apache 

# 1. Generate a api token from the digitalocean UI and export it to the terminal,

 export DO_PAT=YOUR_PERSONAL_ACCESS_TOKEN

# 2. Get the MD5 SHA of you rsa_pub key:

	To get the SHa values: ssh-keygen -E md5 -lf ~/.ssh/id_rsa.pub | awk '{print $2}'

   MD5:69:f6:aa:f6:a2:94:58:f2:41:b0:fb:69:09:e6:87:dd

2.2 Add you rsa_pub key to your Digital ocean 


3. Above are the variables which we will be using in our droplett creation

4. Initailase the project directory.

	$terraform init

5. Now test the project sing the terraform plan, it downloads the respective plugin.

  $terraform plan \
    -var "do_token=${DO_PAT}" \
    -var "pub_key=$HOME/.ssh/id_rsa.pub" \
    -var "pvt_key=$HOME/.ssh/id_rsa" \
    -var "ssh_fingerprint=your SHA value" 



6. Now create the provider.tf file which has all the variables that we will be using in the project.
   $ vi mytest.tf

     variable "do_token" {}
     variable "pub_key" {}
     variable "pvt_key" {}
     variable "ssh_fingerprint" {}

     provider "digitalocean" {
       token = "${var.do_token}"
     }


     Save and Exit

7. Now create the mytest.tf file which contains the resourse of 	creating the droplets.
	
	$vi mytest.tf

	resource "digitalocean_droplet" "web" {
	    image = "ubuntu-16-04-x64"
	    name = "mytest"
	    region = "blr1"
	    size = "s-1vcpu-1gb"
	    private_networking = true
	    ssh_keys = [
	      "${var.ssh_fingerprint}"
	    ]

	connection {
	      user = "root"
	      type = "ssh"
	      private_key = "${file(var.pvt_key)}"
	      timeout = "2m"
	  }

	provisioner "remote-exec" {
	    inline = [
	      "export PATH=$PATH:/usr/bin",
	      # install tomcat
	       "sudo apt-cache search apache",
	       "sudo apt-get install apache2"
	       
	       # install nginx
	       #"sudo apt-get update",
	       #"sudo apt-get -y remove nginx",
	    ]
	  }
	}

	Save and Exit.

8. Now run the terraform plan and then apply cmd.

	$terraform plan \
    -var "do_token=${DO_PAT}" \
    -var "pub_key=$HOME/.ssh/id_rsa.pub" \
    -var "pvt_key=$HOME/.ssh/id_rsa" \
    -var "ssh_fingerprint=your SHA value"

    (It diplays the output stating "mytest" droplet will be created)


    $terraform apply \
    -var "do_token=${DO_PAT}" \
    -var "pub_key=$HOME/.ssh/id_rsa.pub" \
    -var "pvt_key=$HOME/.ssh/id_rsa" \
    -var "ssh_fingerprint=your SHA value" 

    (It launches a droplet and install Apache at port 80)
