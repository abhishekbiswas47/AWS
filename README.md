## Creating/Launching Application using Terraform.
### Task Description:-
1. Create the key and security group which allow the port 80.
2. Launch EC2 instance.
3. In this Ec2 instance use the key and security group which we have created in step 1.
4. Launch one Volume (EBS) and mount that volume into /var/www/html
5. Developer have uploded the code into github repo also the repo has some images.
6. Copy the github repo code into /var/www/html
7. Create S3 bucket, and copy/deploy the images from github repo into the s3 bucket and change the permission to public readable.
8. Create a Cloudfront using s3 bucket(which contains images) and use the Cloudfront URL to  update in code in /var/www/html

### Following the given steps anyone can achieve the desired results:-
#### Step 1.
In the first step you have to declare your provider and its necessary login credentials, example:-
```
provider "aws" {
  region     = "ap-south-1"
  profile    = "abhi"
}
```
#### Step 2.
In the second step you have to create a security group which allows port number 80, which in turns provide the services for HTTP protocol and port number 22 which provides services for SSH protocol. Egress is not open for all IP's and all ports. Also, CIDR is configured for IPv4 not for IPv6. The following commands will perform the above query:-
```
resource "aws_security_group" "allow_http" {
  name        = "allow_http"
  description = "Allow HTTP SSH inbound traffic"
  vpc_id      = "vpc-d4ebf6bc"

  ingress {
    description = "HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "SSH"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = [ "0.0.0.0/0" ]
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }


  tags = {
    Name = "my_http"
  }
}
```
#### Step 3.
In the third and the most critical step as all the steps above revolves around this step. The instance which we are creating here is used to deploy webserver and nearly all other tasks are also done here. The isntance is launched using the keys and security groups created previously.
```
resource "aws_instance" "myweb" {
	ami		= "ami-005956c5f0f757d37"
	instance_type	="t2.micro"
	key_name          = "abhishek"
  	security_groups   = [ "allow_http" ]

	 connection {
    	type        = "ssh"
    	user        = "ec2-user"
    	private_key = file("C:/Users/Abhishek/Downloads/abhishek.pem")
    	host        = "${aws_instance.myweb.public_ip}"
  	}
  
 	 provisioner "remote-exec" {
    	inline = [
      	"sudo yum install httpd  -y",
      	"sudo service httpd start",
      	"sudo service httpd enable"
    	]
 	 }

	tags = {
		Name = "Abhishekos"
	}
}

output "o3" {
	value = aws_instance.myweb.public_ip
}

output "o4" {
	value = aws_instance.myweb.availability_zone
}
```
#### Step 4.
Next, i created an S3 bucket and also manipulated the terraform to save my bucket name in my local system inside my working repository. This is for the time when I destroy the infrastructure created by, it will ask for bucket name which I have made dynamic as we need a unique name for our bucket as S3 is a global service.
```
resource "aws_s3_bucket" "myuniquebucket1227" {
  bucket = "myuniquebucket1227" 
  acl    = "public-read"
  tags = {
    Name        = "uniquebucket1227" 
  }
  versioning {
	enabled =true
  }
}

resource "aws_s3_bucket_object" "s3object" {
  bucket = "${aws_s3_bucket.myuniquebucket1227.id}"
  key    = "1076883.jpg"
  source = "C:/Users/Abhishek/Downloads/1076883.jpg"
}
```
#### Step 5.
