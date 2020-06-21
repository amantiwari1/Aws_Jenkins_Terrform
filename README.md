# aws_terraform

## 	Starting The Website in aws and terraform 

![Create repository](images/ShooterScreenshot-0-15-06-20.png)

### First create  terraform file
```
touch web.tf 
```
### Edit the terraform file for aws, null and tls (SSH RSA Gerenate) 
```
provider "aws" {
  profile =  "aman1"
  region  = "ap-south-1"
}

resource "null_resource" "null_remote" {
	
}

resource "tls_private_key" "webserver_private_key" {
 algorithm = "RSA"
 rsa_bits = 4096
}
```


### download package aws, null and tls using file 

```
terraform init
```

### Create Key pair
```

resource "local_file" "private_key" {
 content = tls_private_key.webserver_private_key.private_key_pem
 filename = "webserver_key.pem"
 file_permission = 0400

}


resource "aws_key_pair" "webserver_key" {
 key_name = "webserver"
 public_key = tls_private_key.webserver_private_key.public_key_openssh

}
```

### Create Security Groups (Firewall) including http and SSH
```

resource "aws_security_group" "allow_http_ssh" {

  name        = "allow_http" 
  description = "Allow http inbound traffic"
  vpc_id      = "vpc-075e88e4d7296ca92"

ingress {
    description = "http"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  } 


  ingress {
    description = "ssh"
    from_port   = 22
    to_port     = 22
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
```

### Now,
### Create EC2 ( Web Server ) with key pairs and security groups and install wget, Jenkins  
```
resource "aws_instance" "web" {
  ami           = "ami-0447a12f28fddb066"
  instance_type = "t2.micro"
  key_name = aws_key_pair.webserver_key.key_name
  security_groups = [aws_security_group.allow_http_ssh.name]



provisioner "remote-exec" {
    inline = [
      "sudo yum update -y",
      "sudo yum install java-1.8.0-openjdk -y",
      "sudo yum install wget git -y",
      "sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo",
      "sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key",
      "sudo yum install jenkins -y",
      "sudo service jenkins start",
      "sudo chkconfig jenkins on",
    ]
      


  connection {
  type     = "ssh"
  user     = "ec2-user"
  private_key = tls_private_key.webserver_private_key.private_key_pem
  host     = aws_instance.web.public_ip
}
  }

  tags = {
    Name = "Web"
  }

 }

```

### Open Jenkins Url 
```
resource "null_resource" "nulllocal1"  {

	provisioner "local-exec" {
	    command = "explorer http://${aws_instance.web.public_ip}:8080/"
	    interpreter = ["PowerShell", "-Command"]
  	}

  	provisioner "remote-exec" {
    inline = [
      "sleep 20",
      "sudo cat /var/lib/jenkins/secrets/initialAdminPassword",
    ]
      


  connection {
  type     = "ssh"
  user     = "ec2-user"
  private_key = tls_private_key.webserver_private_key.private_key_pem
  host     = aws_instance.web.public_ip
}
  }
}
```



### All Output 
```
output "myos_ip" {
  value = aws_instance.web.public_ip
}

```

### After Save the file web.tf

### Open CMD

```
cd I:\aman\terra
```
### then Run it
```
terraform apply
```
