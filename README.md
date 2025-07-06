## 📦 Project: **Chef-Provisioned EC2 Apache Server**

---

## 📑 Components:

* **AWS EC2 instance (Amazon Linux 2023)**
* **Chef Workstation (your local system)**
* **Chef Infra Client (on EC2)**
* **Chef Cookbook: apache-install**
* **Knife EC2 plugin (optional — for provisioning EC2 via Chef itself)**

---

## 📁 Project Structure:

```
chef-ec2-project/
├── cookbooks/
│   └── apache-install/
│       ├── recipes/
│       │   └── default.rb
│       └── metadata.rb
├── kitchen.yml
├── knife.rb
└── README.md
```

---

## ✅ Step 1️⃣: Install Chef Workstation (on your local)

```bash
curl -L https://omnitruck.chef.io/install.sh | sudo bash -s -- -P chef-workstation
chef --version
```

---

## ✅ Step 2️⃣: Launch EC2 Instance (manual or Terraform)

Example EC2 User Data (for Amazon Linux 2023 — install Chef client):

```bash
#!/bin/bash
curl -L https://omnitruck.chef.io/install.sh | sudo bash -s -- -P chef
chef-client --version
```

Or spin up using AWS CLI:

```bash
aws ec2 run-instances --image-id ami-0abcdef1234567890 --count 1 --instance-type t2.micro --key-name my-key --security-group-ids sg-12345 --subnet-id subnet-12345 --user-data file://userdata.sh
```

---

## ✅ Step 3️⃣: Generate Chef Repo

```bash
chef generate repo chef-ec2-project
cd chef-ec2-project
```

---

## ✅ Step 4️⃣: Create Apache Cookbook

```bash
chef generate cookbook cookbooks/apache-install
```

---

## ✅ Step 5️⃣: Write Cookbook Recipe

**cookbooks/apache-install/recipes/default.rb**

```ruby
package 'httpd' do
  action :install
end

service 'httpd' do
  action [:enable, :start]
end

file '/var/www/html/index.html' do
  content '<h1>Hello from Chef-provisioned EC2 instance!</h1>'
  mode '0755'
  owner 'apache'
  group 'apache'
end
```

---

## ✅ Step 6️⃣: Update Metadata

**cookbooks/apache-install/metadata.rb**

```ruby
name 'apache-install'
maintainer 'Atul'
version '0.1.0'
depends ''
```

---

## ✅ Step 7️⃣: Upload Cookbook to Node (manual SSH)

SSH into EC2:

```bash
ssh -i my-key.pem ec2-user@<ec2-public-ip>
```

Copy cookbook:

```bash
scp -i my-key.pem -r cookbooks/apache-install ec2-user@<ec2-public-ip>:/home/ec2-user/
```

Run Chef Client in local mode:

```bash
sudo chef-client --local-mode --runlist 'recipe[apache-install]'
```

---

## ✅ Step 8️⃣: Validate

Visit `http://<ec2-public-ip>` → You should see:

```
Hello from Chef-provisioned EC2 instance!
```

---

## 📘 Optional: Test with Test Kitchen (local or via SSH)

**kitchen.yml**

```yaml
driver:
  name: ssh

provisioner:
  name: chef_zero

platforms:
  - name: amazonlinux-2

suites:
  - name: default
    run_list:
      - recipe[apache-install]
    attributes: {}
```

Run:

```bash
kitchen converge
kitchen verify
```

---

## 📖 README.md (for the repo)

```markdown
# Chef EC2 Apache Project 🚀

## 📑 Objective:
Provision an EC2 instance and configure it using Chef Infra Client to install Apache web server with a custom web page.

## 📦 Components:
- AWS EC2 (Amazon Linux 2023)
- Chef Infra Client
- Chef Cookbook: apache-install

## 📜 Usage:
1. Install Chef Workstation
2. Launch EC2 instance
3. Copy cookbook to instance
4. Run Chef in local mode
5. Validate web page via browser

## 🔗 Author:
Atul (Cloud Solutions Architect)
```

---

## 📦 Repo Name Suggestion:

`chef-ec2-apache-setup`

---

## 📌 Bonus: Knife EC2 Plugin Automation (if desired)

You can optionally install the Knife EC2 plugin to provision instances directly from your Chef Workstation:

```bash
chef gem install knife-ec2
knife ec2 server create ...
```

(Requires AWS Access Key/Secret in knife.rb)

---

## ✅ Summary:

You now have a fully working **Chef-based EC2 configuration project** — clean, modular, and testable.

Awesome — let’s build you a clean **GitHub-ready Chef EC2 Apache Setup project** with Terraform infra, Chef cookbook, EC2 User Data, and docs. I’ll generate the full repo structure, code files, and docs right here.

---

## 📦 Project: `chef-ec2-apache-setup`

---

## 📁 Final Repo Structure:

```
chef-ec2-apache-setup/
├── terraform/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── userdata.sh
├── cookbooks/
│   └── apache-install/
│       ├── metadata.rb
│       └── recipes/
│           └── default.rb
├── README.md
```

---

## 📑 Full Code & Docs:

---

### 📄 terraform/main.tf

```hcl
provider "aws" {
  region = "ap-south-1"
}

resource "aws_key_pair" "ec2_key" {
  key_name   = "chef-key"
  public_key = file("~/.ssh/id_rsa.pub")
}

resource "aws_security_group" "web_sg" {
  name        = "chef-web-sg"
  description = "Allow HTTP and SSH"
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port   = 80
    to_port     = 80
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

resource "aws_instance" "chef_ec2" {
  ami                    = "ami-0f5ee92e2d63afc18" # Amazon Linux 2023 in ap-south-1
  instance_type          = "t2.micro"
  key_name               = aws_key_pair.ec2_key.key_name
  vpc_security_group_ids = [aws_security_group.web_sg.id]
  user_data              = file("userdata.sh")
  tags = {
    Name = "chef-ec2-server"
  }
}
```

---

### 📄 terraform/variables.tf

```hcl
variable "region" {
  default = "ap-south-1"
}
```

---

### 📄 terraform/outputs.tf

```hcl
output "instance_public_ip" {
  value = aws_instance.chef_ec2.public_ip
}
```

---

### 📄 terraform/userdata.sh

```bash
#!/bin/bash
curl -L https://omnitruck.chef.io/install.sh | sudo bash -s -- -P chef
chef-client --version
```

---

### 📄 cookbooks/apache-install/recipes/default.rb

```ruby
package 'httpd' do
  action :install
end

service 'httpd' do
  action [:enable, :start]
end

file '/var/www/html/index.html' do
  content '<h1>Deployed via Chef on EC2!</h1>'
  mode '0755'
  owner 'apache'
  group 'apache'
end
```

---

### 📄 cookbooks/apache-install/metadata.rb

```ruby
name 'apache-install'
maintainer 'Atul'
version '0.1.0'
```

---

### 📄 README.md

````markdown
# 🚀 Chef EC2 Apache Setup

## 📖 Overview
Provision an EC2 instance on AWS using Terraform, install Chef client via EC2 user data, and configure Apache using a Chef cookbook.

---

## 📦 Components
- **Terraform** for infra provisioning
- **Chef Infra Client** (installed on EC2)
- **Apache Web Server** via Chef cookbook

---

## 📜 Usage

### 1️⃣ Clone Repo
```bash
git clone https://github.com/yourname/chef-ec2-apache-setup.git
cd chef-ec2-apache-setup/terraform
````

### 2️⃣ Terraform Init & Apply

```bash
terraform init
terraform apply
```

Note down the public IP from outputs.

---

### 3️⃣ SSH Into EC2

```bash
ssh -i ~/.ssh/id_rsa ec2-user@<ec2-public-ip>
```

### 4️⃣ Copy Cookbook

```bash
scp -i ~/.ssh/id_rsa -r ../cookbooks/apache-install ec2-user@<ec2-public-ip>:/home/ec2-user/
```

---

### 5️⃣ Run Chef in Local Mode

```bash
sudo chef-client --local-mode --runlist 'recipe[apache-install]'
```

---

### 6️⃣ Validate

Open `http://<ec2-public-ip>` → Should show: **Deployed via Chef on EC2!**

---

## 🔗 Author

**Atul (Cloud Solutions Architect)**

---

## 📌 Notes

* Ensure your SSH key `id_rsa.pub` exists in `~/.ssh/`
* Uses **Amazon Linux 2023 AMI**

```
