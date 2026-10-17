# Hands-on Apache-01 — Deploy a Website on Amazon Linux 2023 EC2

**Course:** AWS & DevOps — Techpro Education  
**Reviewed:** 13 October 2026  
**Level:** Beginner  
**Goal:** Install Apache HTTP Server, publish the Techpro `designer` website, and automate the setup with EC2 user data.

## Learning outcomes

Students will learn to launch and connect to EC2, configure Security Groups, install Apache HTTP Server (`httpd`), publish static content safely, validate service health, use user data, and clean up their AWS resources.

## Part 1 — Introduction to Apache

Apache HTTP Server (called `httpd` on Amazon Linux) is a free, open-source, modular web server. It can serve static websites, handle requests for applications, and provide features such as authentication and reverse proxying.

**Key paths on Amazon Linux 2023:**

| Purpose | Path or command |
|---|---|
| Static website directory | `/var/www/html` |
| Main configuration | `/etc/httpd/conf/httpd.conf` |
| Extra configuration files | `/etc/httpd/conf.d/` |
| Service | `httpd` |
| Apache syntax check | `sudo httpd -t` |
| Logs | `/var/log/httpd/` |

These are standard Amazon Linux package locations, not universal paths for every operating system.

## Part 2 — Launch an Amazon Linux 2023 EC2 instance and connect

**EC2 settings**

| Parameter | Recommended lab value |
|---|---|
| Name | `apache-web-lab` |
| AMI | Amazon Linux 2023, x86_64 |
| Instance type | `t3.micro` (check account pricing and eligibility) |
| Region | N. Virginia (`us-east-1`) or instructor-selected Region |
| VPC | Default VPC if available |
| Subnet | Public subnet routed to an Internet Gateway |
| Auto-assign public IPv4 | Enabled |
| Key pair | Select/create a private PEM key |
| Security group | `apache-lab-sg` with rules below |

**Security Group inbound rules**

- SSH / TCP 22: **My IP** (`your-public-ip/32`), not everyone.
- HTTP / TCP 80: `0.0.0.0/0` for the temporary public website demonstration.
- Keep outbound access suitable for installing packages and downloading the sample repository.

`0.0.0.0/0` means all IPv4 addresses. It may be appropriate for the **public HTTP lab** but is not a good default for public SSH access. Production applications should use HTTPS/443 with a correctly configured certificate and limited administration access.

Connect using Linux, macOS or a terminal supporting OpenSSH:

```bash
chmod 400 ~/Downloads/techpro-key.pem
ssh -i ~/Downloads/techpro-key.pem ec2-user@<EC2-PUBLIC-IP>
```

On Windows PowerShell, supply the correct full `.pem` path and configure key-file permissions if your SSH client complains. Confirm your OS:

```bash
cat /etc/os-release
whoami
```

## Part 3 — Install Apache and deploy the Techpro designer site

### Step 1: Install packages

```bash
sudo dnf upgrade -y
sudo dnf install -y httpd git rsync
```

**Terminology:** `httpd` is the Apache package and service name used in Amazon Linux; `git` retrieves website files; `rsync` publishes them without unintentionally exposing Git metadata.

### Step 2: Check, start, and enable Apache

```bash
sudo systemctl status httpd
sudo systemctl enable --now httpd
sudo systemctl status httpd
sudo httpd -t
curl -I http://localhost
```

Before the service starts, `systemctl status httpd` may show inactive. After `enable --now`, expect **active (running)**. A successful `curl -I` generally returns `HTTP/1.1 200 OK`. `httpd -t` normally reports `Syntax OK`.

Open `http://<EC2-PUBLIC-IP>` in a browser. If it does not load, verify the Security Group allows HTTP TCP/80 and that your instance has a reachable public address.


### Step 3: Publish the website with safe permissions
Set permission of the files and folders under `/var/www/html/` folder to everyone.

```bash
sudo chmod -R 755 /var/www/html
```
OR
```bash
sudo find /var/www/html -type d -exec chmod 755 {} +
sudo find /var/www/html -type f -exec chmod 644 {} +
```
### Step 4: Clone the Techpro designer repository

```bash
git clone https://github.com/techproedu/designer.git
cp -R ./designer/. /var/www/html
cd /var/www/html
```

The class repo contains `index.html` and related static assets. Cloning requires outbound network access. Review third-party or remote content before publishing outside a lab.

### Step 5: Verify the website

```bash
ls -l /var/www/html/index.html
sudo httpd -t
curl http://<EC2-PUBLIC-IP>
```

Open `http://<EC2-PUBLIC-IP>` in your browser. For static content updates, an Apache restart is **not normally required**. If you update the Apache configuration, run `sudo httpd -t` first and then `sudo systemctl reload httpd` if appropriate.

## Part 4 — Automate Apache setup with EC2 user data

Launch a **new Amazon Linux 2023** EC2 instance. Choose a security group with SSH limited to My IP and HTTP/80 allowed for the demo. Add this script in **Advanced details → User data**:

```bash
#!/bin/bash
dnf update -y
dnf install httpd -y 
dnf install git -y
systemctl start httpd
systemctl enable httpd
cd /home/ec2-user
git clone https://github.com/techproedu/designer.git
sudo chmod -R 777 /var/www/html
cp -R ./designer/. /var/www/html
sudo systemctl restart httpd
```

AWS launches Linux user-data scripts **as root by default**. Do not add `sudo` to every line unnecessarily. User data normally runs on **first boot**, not every reboot. Allow a few minutes after instance launch for package installation and the GitHub clone.

Connect and validate:

```bash
cloud-init status --long
sudo systemctl is-active httpd
sudo httpd -t
curl http://localhost
```

Open `http://<NEW-EC2-PUBLIC-IP>` in the browser and confirm the designer site appears.

## Troubleshooting

| Problem | What to check |
|---|---|
| SSH timeout | Public IP, key pair, public subnet routing, TCP/22 rule limited to correct My IP |
| `Permission denied (publickey)` | User must be `ec2-user`; correct PEM file and permissions |
| Local `curl` succeeds, public browser fails | Security Group TCP/80, public IP assignment, VPC routes |
| Apache inactive | `sudo systemctl status httpd`, `sudo journalctl -u httpd -n 50 --no-pager` |
| Configuration error | `sudo httpd -t` and config files under `/etc/httpd/` |
| Missing designer page | `ls -la /var/www/html`, Git clone output, rsync command |
| User data incomplete | `cloud-init status --long` and `/var/log/cloud-init-output.log` |
| Website service port conflict | Make sure NGINX and Apache are not both trying to bind the same address:port 80 |

Additional diagnostics:

```bash
sudo ss -ltnp | grep ':80'
sudo tail -n 30 /var/log/httpd/error_log
sudo tail -n 30 /var/log/httpd/access_log
```

## Cleanup and completion

**Done condition:** The public Apache web page loads, `curl -I http://localhost` succeeds, `httpd` is active, and the fresh instance built with user data also publishes the page.

Terminate temporary EC2 instances after the class. Review unused EBS volumes, Elastic IPs, and custom security groups; remove unused resources and verify AWS Billing. EC2 compute, storage, data transfer, and public IPv4 may incur charges even in a teaching lab.

## Official references

- Apache — [About the Apache HTTP Server Project](https://httpd.apache.org/ABOUT_APACHE)
- Apache — [MPM documentation](https://httpd.apache.org/docs/2.4/mpm.html)
- AWS — [EC2 user data](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html)
- AWS — [EC2 launch/network settings](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-launch-parameters.html)
- Techpro class site — [designer repository](https://github.com/techproedu/designer)
