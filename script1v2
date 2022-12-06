terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
    }

  }

  required_version = ">= 1.0"
}

provider "aws" {
  profile = "second"
  region  = "eu-central-1"
}


resource "aws_vpc" "aws_vps_1" {
  tags = {
    Name = "AWS-VPS-1"
  }

  tags_all = {
    Name = "AWS-VPS-1"
  }

  cidr_block         = "10.0.0.0/16"
  enable_dns_support = true
  instance_tenancy   = "default"
}


resource "aws_internet_gateway" "aws_internet_gateway" {
  tags = {
    Name = "AWS-Internet-Gateway"
  }

  tags_all = {
    Name = "AWS-Internet-Gateway"
  }

  vpc_id = aws_vpc.aws_vps_1.id
}


resource "aws_subnet" "aws_bastion_subnet" {
  tags = {
    Name = "AWS-Bastion-Subnet"
  }

  tags_all = {
    Name = "AWS-Bastion-Subnet"
  }

  availability_zone                   = "eu-central-1a"
  cidr_block                          = "10.0.100.0/24"
  map_public_ip_on_launch             = true
  private_dns_hostname_type_on_launch = "ip-name"
  vpc_id                              = aws_vpc.aws_vps_1.id
}


resource "aws_subnet" "aws_backend_subnet" {
  tags = {
    Name = "AWS-Backend-Subnet"
  }

  tags_all = {
    Name = "AWS-Backend-Subnet"
  }

  availability_zone_id                = "euc1-az2"
  cidr_block                          = "10.0.200.0/24"
  private_dns_hostname_type_on_launch = "ip-name"
  vpc_id                              = aws_vpc.aws_vps_1.id
}


resource "aws_route_table" "aws_public_route_table" {
  tags = {
    Name = "AWS-Public-Route-Table"
  }

  tags_all = {
    Name = "AWS-Public-Route-Table"
  }

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.aws_internet_gateway.id
  }

  vpc_id = aws_vpc.aws_vps_1.id
}


resource "aws_route_table" "aws_private_route_table" {
  tags = {
    Name = "AWS-Private-Route-Table"
  }

  tags_all = {
    Name = "AWS-Private-Route-Table"
  }

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.aws_nat_gateway.id
  }

  vpc_id = aws_vpc.aws_vps_1.id
}


resource "aws_route_table_association" "bastion" {
  subnet_id      = aws_subnet.aws_bastion_subnet.id
  route_table_id = aws_route_table.aws_public_route_table.id
}


resource "aws_route_table_association" "backend" {
  subnet_id      = aws_subnet.aws_backend_subnet.id
  route_table_id = aws_route_table.aws_private_route_table.id
}


resource "aws_eip" "aws_elastic_ip" {
  tags = {
    Name = "AWS-Elastic-IP"
  }

  tags_all = {
    Name = "AWS-Elastic-IP"
  }

  network_border_group = "eu-central-1"
  public_ipv4_pool     = "amazon"
  vpc                  = true
}


resource "aws_nat_gateway" "aws_nat_gateway" {
  tags = {
    Name = "AWS-NAT-Gateway"
  }

  tags_all = {
    Name = "AWS-NAT-Gateway"
  }

  allocation_id = aws_eip.aws_elastic_ip.id
  connectivity_type = "public"
  subnet_id         = aws_subnet.aws_bastion_subnet.id
}


resource "aws_security_group" "aws_bastion_sg" {
  tags = {
    Name = "AWS-Bastion-SG"
  }

  tags_all = {
    Name = "AWS-Bastion-SG"
  }

  description = "test"
  egress {
    cidr_blocks = ["0.0.0.0/0"]
    from_port   = 0
    protocol    = "-1"
    to_port     = 0
  }

  ingress {
    cidr_blocks = ["0.0.0.0/0"]
    from_port   = -1
    protocol    = "icmp"
    to_port     = -1
  }

  ingress {
    cidr_blocks = ["0.0.0.0/0"]
    from_port   = 22
    protocol    = "tcp"
    to_port     = 22
  }

  name   = "AWS-Security_group1"
  vpc_id = aws_vpc.aws_vps_1.id
}


resource "aws_security_group" "aws_backend_sg" {
  tags = {
    Name = "AWS-Backend-SG"
  }

  tags_all = {
    Name = "AWS-Backend-SG"
  }

  description = "test2"
  egress {
    cidr_blocks = ["0.0.0.0/0"]
    from_port   = 0
    protocol    = "-1"
    to_port     = 0
  }

  ingress {
    cidr_blocks = ["10.0.0.0/16"]
    description = "ssh"
    from_port   = 22
    protocol    = "tcp"
    to_port     = 22
  }

  ingress {
    cidr_blocks = ["10.0.0.0/16"]
    description = "ping"
    from_port   = -1
    protocol    = "icmp"
    to_port     = -1
  }

  ingress {
    cidr_blocks = ["10.0.0.0/16"]
    description = "mikatux/ftps-server"
    from_port   = 3000
    protocol    = "tcp"
    to_port     = 3010
  }

  ingress {
    cidr_blocks = ["10.0.0.0/16"]
    description = "ftp"
    from_port   = 21
    protocol    = "tcp"
    to_port     = 21
  }

  ingress {
    cidr_blocks = ["10.0.0.0/16"]
    description = "chonjay21/ftps"
    from_port   = 60000
    protocol    = "tcp"
    to_port     = 60010
  }

  name   = "AWS-Security-group2"
  vpc_id = aws_vpc.aws_vps_1.id
}



resource "aws_instance" "aws_bastion_vm" {
  tags = {
    Name = "AWS-Bastion-VM"
  }

  tags_all = {
    Name = "AWS-Bastion-VM"
  }

  ami                         = "ami-0caef02b518350c8b"
  associate_public_ip_address = true
  availability_zone           = "eu-central-1a"
  capacity_reservation_specification {
    capacity_reservation_preference = "open"
  }

  instance_initiated_shutdown_behavior = "stop"
  instance_type                        = "t2.micro"
  key_name                             = "aws"
  metadata_options {
    http_endpoint               = "enabled"
    http_put_response_hop_limit = 1
    http_tokens                 = "optional"
    instance_metadata_tags      = "disabled"
  }

  private_ip = "10.0.100.62"
  root_block_device {
    delete_on_termination = true
    volume_size           = 8
    volume_type           = "gp2"
  }

  source_dest_check      = true
  subnet_id              = aws_subnet.aws_bastion_subnet.id
  tenancy                = aws_vpc.aws_vps_1.instance_tenancy
  vpc_security_group_ids = [aws_security_group.aws_bastion_sg.id]
}


resource "aws_instance" "aws_backend_vm" {
  tags = {
    Name = "AWS_Backend-VM"
  }

  tags_all = {
    Name = "AWS_Backend-VM"
  }

  ami               = "ami-0caef02b518350c8b"
  availability_zone = "eu-central-1a"
  capacity_reservation_specification {
    capacity_reservation_preference = "open"
  }

  instance_initiated_shutdown_behavior = "stop"
  instance_type                        = "t2.micro"
  key_name                             = "aws"
  metadata_options {
    http_endpoint               = "enabled"
    http_put_response_hop_limit = 1
    http_tokens                 = "optional"
    instance_metadata_tags      = "disabled"
  }

  private_ip = "10.0.200.254"
  root_block_device {
    delete_on_termination = true
    volume_size           = 8
    volume_type           = "gp2"
  }

  source_dest_check      = true
  subnet_id              = aws_subnet.aws_backend_subnet.id
  tenancy                = aws_vpc.aws_vps_1.instance_tenancy
  vpc_security_group_ids = [aws_security_group.aws_backend_sg.id]
}





