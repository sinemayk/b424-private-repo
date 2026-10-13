# Hands-on EC2-01: Install Apache Web Server on Amazon Linux 2023

**AWS & DevOps | Techpro Education**  
**Updated:** October 2026  
**Goal:** Launch an EC2 instance, connect securely, serve a custom webpage, and automate Apache setup with EC2 user data.

> **Cost and security reminder:** AWS charges depend on your account, Region, instance type, EBS storage and public IPv4 usage. Do not assume this lab is free. Create a lab-specific security group, restrict SSH to **My IP**, and terminate lab resources after class.

## Outline

- **Part 1:** Get to know the Apache HTTP Server
- **Part 2:** Launch an Amazon Linux 2023 EC2 instance and connect with SSH
- **Part 3:** Install Apache and display a customized **Hello World** webpage
- **Part 4:** Automate Apache installation using an EC2 **user-data** script
- **Part 5:** Validate, troubleshoot, and clean up

## Prerequisites

- An AWS account with permission to launch and terminate EC2 instances, create key pairs, and configure security groups.
- Access to the **EC2 Console** in **US East (N. Virginia)**, Region ID `us-east-1`.
- An SSH client (Terminal, PowerShell/OpenSSH, or another trusted SSH client).
- Your network's public IPv4 address, used to restrict SSH access.
- Awareness that EC2, EBS volumes, and public IPv4 addresses may incur charges.

---

## Part 1 — What is Apache?

**Apache HTTP Server**, often called **Apache** or `httpd`, is open-source web server software. It receives HTTP requests and returns webpages and other content to the browser.

**Simple example:** A user enters `http://PUBLIC_IP` in a browser. The browser sends a request over **TCP port 80**. The Apache service on the EC2 instance responds with the page stored at `/var/www/html/index.html`.

**Key terms for students**

| Term | Meaning |
|---|---|
| EC2 instance | A virtual server running in AWS |
| AMI | Amazon Machine Image, used to create the instance |
| Instance type | Determines its compute and memory characteristics |
| Security group | Stateful firewall rules for the instance |
| SSH (TCP 22) | Secure remote terminal access |
| HTTP (TCP 80) | Standard unencrypted web traffic; acceptable for this temporary demo |
| `httpd` | The Apache web server package and service name |
| EC2 user data | Startup instructions that can automate initial configuration |

---

## Part 2 — Launch an Amazon Linux 2023 instance and connect

### Step 1. Choose the instance configuration

In **AWS Console → EC2 → Instances → Launch instances**, configure:

| Setting | Lab choice | Why |
|---|---|---|
| Name | `techpro-apache-lab` | Easy to identify for cleanup |
| AMI | **Amazon Linux 2023 AMI** (x86_64) | Current Amazon Linux distribution |
| Instance type | `t3.micro` | Small instance appropriate for a short demonstration; confirm price |
| Key pair | Create or select an SSH key pair | Required for SSH access in this exercise |
| Region | **US East (N. Virginia)** (`us-east-1`) | Consistent class environment |
| VPC | Default VPC **if available** | Simpler beginner setup |
| Subnet | A public subnet with a route through an Internet Gateway | Required for direct internet access |
| Auto-assign public IPv4 | **Enable**, if not enabled through subnet settings | Allows browser and SSH access from the internet |
| Security group | **Create a new security group** | Lab traffic rules should be explicit |
| Storage | Default AMI EBS root volume, if suitable | Stores the operating system and website files |

**Inbound security-group rules**

| Rule | Protocol / port | Source |
|---|---|---|
| SSH | TCP 22 | **My IP** (`YOUR_PUBLIC_IP/32`) |
| HTTP | TCP 80 | `0.0.0.0/0` **only for this public demonstration** |

Do **not** set SSH to `0.0.0.0/0` (Anywhere IPv4). For production web services, use HTTPS and tighter access controls appropriate for your architecture.

**Instructor note:** A default VPC is not guaranteed to exist in every account. If it has been removed, use an approved public subnet or prepare the VPC in advance. If SSH is not required, EC2 Instance Connect or Systems Manager may be alternatives, depending on setup.

### Step 2. Launch and verify

Choose **Launch instance**. Wait until the instance is in the **Running** state and its status checks pass. On its **Details** tab, copy the **Public IPv4 address** or **Public IPv4 DNS** value.

**Validation:** The instance shows `Running`, it has the intended security group, and it has an internet-reachable public address.

### Step 3. Connect with SSH

From Linux or macOS Terminal, first protect your downloaded `.pem` file:

```bash
chmod 400 ~/Downloads/Techpro-Key.pem
ssh -i ~/Downloads/Techpro-Key.pem ec2-user@PUBLIC_IP
```

Replace `Techpro-Key.pem` and `PUBLIC_IP` with your actual values. The default username for Amazon Linux 2023 is **`ec2-user`**.

In Windows PowerShell with OpenSSH, the command is similar:

```powershell
ssh -i .\Techpro-Key.pem ec2-user@PUBLIC_IP
```

On Windows, the key may need restricted NTFS permissions if the SSH client rejects it. Use your organization's approved process to secure the key; do not expose the private key to other users.

**Validation:** You see a command prompt on the Amazon Linux EC2 instance. Run:

```bash
cat /etc/os-release
hostname
whoami
```

**Expected:** OS indicates **Amazon Linux 2023** and `whoami` prints `ec2-user`.

---

## Part 3 — Install and customize Apache

### Step 4. Update packages

```bash
sudo dnf upgrade -y
```

- `sudo`: run with administrator permissions.
- `dnf`: package manager used on Amazon Linux 2023.
- `upgrade`: install available package updates.
- `-y`: answer yes to installation prompts.

### Step 5. Install Apache

```bash
sudo dnf install -y httpd
```

`httpd` is the Apache package and service name on Amazon Linux 2023.

### Step 6. Start Apache and enable it at boot

```bash
sudo systemctl enable --now httpd
sudo systemctl status httpd
sudo systemctl is-enabled httpd
```

- `enable`: start Apache automatically after instance boot.
- `--now`: start Apache immediately as well.
- `status`: inspect whether the service is active.
- `is-enabled`: check startup behavior.

**Expected:** `active (running)` and `enabled`. Press `q` to quit the status display if necessary.

### Step 7. Test the default webpage

Inside the EC2 SSH terminal:

```bash
curl -I http://localhost
```

**Expected:** An HTTP response such as `HTTP/1.1 200 OK` (actual header wording may differ).

From your own computer, visit:

```text
http://PUBLIC_IP
```

**Expected:** Apache's default test page or a successful HTTP response.

If the website does not load, check **TCP 80** in the security group, public IP addressing, the subnet route and the Apache service. **Do not open all ports as a troubleshooting shortcut.**

### Step 8. Safely allow the lab user to edit the web directory

The original lesson used `chmod -R 777`. **Do not use that command**: it allows every local user to modify the website directory.

Instead, for this controlled classroom lab:

```bash
sudo chown ec2-user:apache /var/www/html
sudo chmod 755 /var/www/html
```

This lets `ec2-user` create and edit the webpage while Apache can read and serve it. It avoids making the directory world-writable. Larger deployments should use their own deployment account, ownership model, and hardened permissions.

### Step 9. Create the first page

```bash
cd /var/www/html
echo 'HELLO TECHPRO EDUCATION' > index.html
ls -l index.html
cat index.html
curl http://localhost
```

**Expected:** `HELLO TECHPRO EDUCATION` appears in both the terminal and browser after refreshing `http://PUBLIC_IP`.

No Apache restart is needed for a change to a static HTML file; Apache reads the file when serving it. You can still demonstrate service management with `sudo systemctl restart httpd` if you want to practice that command.

### Step 10. Customize the page using Vim

```bash
cd /var/www/html
vim index.html
```

Press **`i`** to enter Insert mode. Replace the page with:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>My First EC2 Web Server</title>
</head>
<body>
    <h1>Hello from Techpro Education!</h1>
    <p>This webpage is hosted on Apache running on Amazon EC2.</p>
</body>
</html>
```

Press **Esc**, type **`:wq`**, then press **Enter** to save and exit.

```bash
cat /var/www/html/index.html
curl http://localhost
```

Refresh `http://PUBLIC_IP` in your browser.

**Expected:** The browser displays **Hello from Techpro Education!**

---

## Part 4 — Automate the setup with EC2 user data

### Step 11. Launch a fresh EC2 instance

Create a **second temporary instance** with the same Amazon Linux 2023 AMI, suitable instance type, public subnet, and explicitly configured inbound HTTP port 80. Restrict SSH port 22 to My IP if SSH access is needed. Use a recognizable name, such as `techpro-apache-userdata`.

Under **Advanced details → User data**, paste this script **before** launching:

```bash
#!/bin/bash
set -e

# Update packages and install Apache
# EC2 user data runs as root by default, so sudo is unnecessary.
dnf upgrade -y
dnf install -y httpd

# Generate the website during first boot
cat > /var/www/html/index.html <<EOF
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Automated EC2 Web Server</title>
</head>
<body>
    <h1>Hello from Techpro Education!</h1>
    <p>This web server was created automatically with EC2 user data.</p>
    <p>Created at: $(date -u)</p>
</body>
</html>
EOF

# Start now and after future reboots
systemctl enable --now httpd
```

### Step 12. Verify the automated result

Launch the instance. **Allow several additional minutes** for startup tasks to finish. Visit:

```text
http://SECOND_INSTANCE_PUBLIC_IP
```

Or from your local terminal:

```bash
curl http://SECOND_INSTANCE_PUBLIC_IP
```

**Expected:** The custom automated webpage loads without you manually installing Apache on the second instance.

**Important behavior:** EC2 user-data scripts run with **root** privileges by default and usually execute only on the **first boot**. A reboot does not automatically re-run this script. The time stamp in the HTML is therefore the initial creation time, not the time of each webpage visit.

---

## Part 5 — Validation, troubleshooting and cleanup

### Quick troubleshooting checklist

| Symptom | Check | Useful command or action |
|---|---|---|
| SSH connection times out | My IP `/32`, correct public IP, SSH TCP 22 and route | Review instance **Security** and **Networking** |
| `Permission denied (publickey)` | Correct `.pem` file, key permissions and `ec2-user` username | `chmod 400 KEY.pem` on Linux/macOS |
| Browser times out | HTTP TCP 80, public IP and internet route | Confirm SG inbound rule and public subnet |
| HTTP refused or Apache not started | Apache service and port 80 | `sudo systemctl status httpd` |
| Site shows an unexpected page | Web root and `index.html` content | `ls -l /var/www/html; cat /var/www/html/index.html` |
| User-data webpage missing | First-boot script errors or script still running | `sudo tail -n 80 /var/log/cloud-init-output.log` |
| Apache configuration problem | Configuration syntax | `sudo httpd -t` |

**Local validation commands:**

```bash
sudo systemctl is-active httpd
sudo systemctl is-enabled httpd
curl -I http://localhost
sudo tail -n 80 /var/log/cloud-init-output.log
```

### Cleanup (required after class)

1. In **EC2 → Instances**, select both temporary instances and **Terminate** them once you no longer need them.
2. Check **EBS → Volumes** for any volumes not automatically deleted with the instances.
3. Check any Elastic IPs or other public IPv4 addresses associated with the lab and release resources you no longer need.
4. Remove the temporary lab-specific security group if it is no longer attached or needed.
5. Review the AWS Billing dashboard to make sure unintended resources are not left running.

> **Remember:** Stopping a Linux EC2 instance may stop its compute charges, but EBS volumes and certain other resources can still incur costs. Terminating an instance also requires checking associated resources and their deletion settings.

### Learning outcomes

By the end of the lab, students should be able to:

- Explain what EC2, Apache, SSH, security groups, and user data do.
- Launch a small Amazon Linux 2023 instance and connect securely.
- Install and manage Apache using `dnf` and `systemctl`.
- Serve an HTML page from `/var/www/html/index.html`.
- Automate setup with a first-boot user-data script.
- Troubleshoot access issues and clean up chargeable resources.

---

## Official AWS references (verified October 2026)

- [Tutorial: Install a LAMP server on Amazon Linux 2023](https://docs.aws.amazon.com/linux/al2023/ug/ec2-lamp-amazon-linux-2023.html)
- [Control traffic with security groups](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html)
- [Run commands when launching an instance using user data](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html)
- [On-Demand Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-on-demand-instances.html)
- [EC2 billing and purchasing options](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-purchasing-options.html)
