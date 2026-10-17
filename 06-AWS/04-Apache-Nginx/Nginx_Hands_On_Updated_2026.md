# Hands-on NGINX-01 — Deploy a Website on Amazon Linux 2023 EC2

**Course:** AWS & DevOps — Techpro Education  
**Reviewed:** 13 October 2026  
**Level:** Beginner  
**Goal:** Launch an EC2 instance, install NGINX, publish the Techpro `designer` website, test it, and automate the same setup with EC2 user data.

## Learning outcomes

By the end of this lab, students can:

1. Launch Amazon Linux 2023 EC2 with appropriate network permissions.
2. Connect securely using SSH.
3. Install and control NGINX with `dnf` and `systemctl`.
4. Deploy a website without unsafe `chmod 777` permissions.
5. Verify the website locally and from their browser.
6. Automate first-boot configuration using EC2 user data and troubleshoot `cloud-init`.
7. Clean up lab resources to reduce AWS costs.

## Part 1 — Create a dedicated Security Group

In the AWS Console select **EC2 → Security Groups → Create security group**.

| Setting | Value | Purpose |
|---|---|---|
| Name | `nginx-lab-sg` | Easy to identify and remove after lab |
| VPC | Default VPC, if present | For this beginner lab |
| Inbound SSH | TCP 22, **My IP** (`your-public-ip/32`) | Allows administration from your network |
| Inbound HTTP | TCP 80, `0.0.0.0/0` | Public demo site over HTTP |
| Outbound | Default allow outbound for the lab | Needed for package installation and GitHub |

**Safety:** Do **not** configure SSH `0.0.0.0/0` as the standard classroom rule. If your public IP changes, update the SSH rule. HTTP/80 is only for this public demo; production websites typically use HTTPS/443.

## Part 2 — Launch and connect to an EC2 instance

In **EC2 → Instances → Launch instances**, select:

| Parameter | Recommended lab setting |
|---|---|
| Name | `nginx-web-lab` |
| AMI | **Amazon Linux 2023**, x86_64 |
| Instance type | `t3.micro` (verify price and Free Tier eligibility for your account) |
| Region | US East (N. Virginia), `us-east-1`, or your instructor's region |
| VPC | Default VPC, if available |
| Subnet | Public subnet with a route to an Internet Gateway |
| Auto-assign public IPv4 | Enabled, for direct browser/SSH access |
| Security group | `nginx-lab-sg` |
| Key pair | Create/select a key pair; keep the `.pem` private |

**Linux / macOS / Git Bash (key permissions as supported by your environment):**

```bash
chmod 400 ~/Downloads/techpro-key.pem
ssh -i ~/Downloads/techpro-key.pem ec2-user@<EC2-PUBLIC-IP>
```

For Windows PowerShell, use the PEM file path such as `C:\\Users\\YourName\\Downloads\\techpro-key.pem`; configure file permissions appropriately if the SSH client rejects the key.

Validate your instance:

```bash
cat /etc/os-release
whoami
hostname
```

Expected username: `ec2-user`. The distribution identifies itself as Amazon Linux 2023.

## Part 3 — Install NGINX and publish the designer website

### Step 1: Update and install packages

```bash
sudo dnf upgrade -y
sudo dnf install -y nginx git rsync
```

`dnf` is Amazon Linux 2023's package manager. `git` downloads the class website, and `rsync` copies the site without copying the repository's `.git` folder.

### Step 2: Start NGINX and enable it on boot

```bash
sudo systemctl enable --now nginx
sudo systemctl status nginx
sudo nginx -t
curl -I http://localhost
```

`enable --now` enables automatic startup and starts the service immediately. `nginx -t` checks syntax and configuration. `curl -I` requests HTTP response headers; a successful lab typically shows `HTTP/1.1 200 OK`.

Open `http://<EC2-PUBLIC-IP>` in your browser. Confirm the default NGINX page is visible before deploying the designer site.

### Step 3: Clone Techpro's sample website

```bash
git clone https://github.com/techproedu/designer.git /tmp/techpro-designer
ls -la /tmp/techpro-designer
```

The repository currently contains an `index.html` and static web assets. It is an **external dependency**; clone requires internet connectivity. Review any external code before using it in a production system.

### Step 4: Copy the static files to NGINX's document root

```bash
sudo cp -R /tmp/techpro-designer/. /usr/share/nginx/html/
sudo chown -R root:root /usr/share/nginx/html
sudo find /usr/share/nginx/html -type d -exec chmod 755 {} +
sudo find /usr/share/nginx/html -type f -exec chmod 644 {} +
```

**Why not `chmod -R 777`?** It grants every user write access to the website files. The server needs to **read** static files, not permission for everyone to change them. These commands make directories readable/traversable and files readable, while leaving ownership with root.

### Step 5: Validate the published website

```bash
ls -l /usr/share/nginx/html/index.html
sudo nginx -t
curl -I http://localhost
curl http://localhost | head -n 12
```

Then refresh `http://<EC2-PUBLIC-IP>` in your browser. Static file changes usually do **not** require an NGINX restart. If you edit a configuration file later, run `sudo nginx -t` before `sudo systemctl reload nginx`.


## Part 4 — Automate with EC2 user data

Launch **a fresh Amazon Linux 2023** instance using the same dedicated security group. In **Advanced details → User data**, paste this complete script:

```bash
#!/bin/bash
dnf update -y
dnf install nginx -y
dnf install git -y
systemctl start nginx
cd /usr/share/nginx/html
git clone https://github.com/techproedu/designer.git
chmod -R 777 /usr/share/nginx/html
rm index.html
cp -R ./designer/. .
systemctl restart nginx
systemctl enable nginx
```

It can take a few minutes for package installation and site deployment to complete. When the instance is ready, check:

```bash
cloud-init status --long
sudo tail -n 80 /var/log/cloud-init-output.log
sudo systemctl is-active nginx
curl -I http://localhost
```

Browse to `http://<NEW-EC2-PUBLIC-IP>`.

## Troubleshooting

| Symptom | Possible cause | Check |
|---|---|---|
| SSH connection timeout | SSH rule does not include your current public IP; wrong route/IP | Security group inbound TCP 22 from My IP, public IPv4, subnet route |
| SSH `Permission denied (publickey)` | Wrong username, key pair, or PEM permissions | Use `ec2-user`, correct `.pem`, appropriate permissions |
| Site opens locally but not in browser | HTTP/80 blocked, wrong public IP, wrong route | `curl -I http://localhost`, SG HTTP rule, public IPv4 |
| NGINX service inactive | Installation or config error | `sudo journalctl -u nginx -n 50 --no-pager` |
| NGINX reports a configuration error | Invalid directive or syntax | `sudo nginx -t` |
| Default page remains | Files copied to wrong path, cache, or deployment failed | `ls /usr/share/nginx/html`, `curl http://localhost` |
| GitHub clone fails | Missing outbound internet, DNS or GitHub access | `git --version`, DNS, NAT/Internet Gateway, cloud-init log |
| User-data did not complete | Package or Git clone error | `/var/log/cloud-init-output.log` and `cloud-init status --long` |

Additional checks:

```bash
sudo ss -ltnp | grep ':80'
sudo tail -n 30 /var/log/nginx/error.log
sudo tail -n 30 /var/log/nginx/access.log
```

## Important classroom note

NGINX and Apache both normally listen on TCP port 80. If you install them on the **same** instance and address, they can conflict. Use separate EC2 instances for parallel practice, or stop one service before starting the other.

## Cleanup and done condition

**Done condition:** The designer site responds locally with `curl`, loads in a browser via the EC2 public IP, and the fresh user-data instance can reproduce the result without manual configuration.

At the end, terminate both lab instances; review and remove unused volumes, Elastic IPs, and custom security groups that are no longer needed. Verify billing/resource cleanup in the AWS Console. Instance usage, storage, data transfer, and public IPv4 can incur charges.

## Official references

- AWS — [EC2 user data](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html)
- AWS — [EC2 launch/network security settings](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-launch-parameters.html)
- NGINX — [Beginner's Guide](https://nginx.org/en/docs/beginners_guide.html)
- Techpro class site — [designer repository](https://github.com/techproedu/designer)

###  Reset cloud-init
```bash
sudo cloud-init clean --logs --reboot
```
This clears cloud-init's cached state and reboots the instance so initialization runs again.

server {
    listen 80 default_server;
    listen [::]:80 default_server;

    server_name _;

    root /usr/share/nginx/html;
    index index.html;

    # Nginx website
    location / {
        try_files $uri $uri/ =404;
    }

    # Redirect /apache to /apache/
    location = /apache {
        return 301 /apache/;
    }

    # Apache website (reverse proxy)
    location /apache/ {
        proxy_pass http://127.0.0.1:8080/;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Prefix /apache;
    }
}
