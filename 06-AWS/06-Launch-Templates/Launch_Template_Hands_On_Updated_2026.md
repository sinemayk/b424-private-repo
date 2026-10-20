# Hands-On EC2 — Launch Templates and Versioning (2026)

**Platform:** Amazon EC2 · Amazon Linux 2023 · NGINX  
**Lab goal:** Create one Launch Template with three versions, launch a fresh EC2 instance from each version, compare the results, and clean up.  
**Region example:** US East (N. Virginia), `us-east-1`. You may use another supported Region.  
**Important:** A Launch Template describes how to launch instances. It **does not update existing running instances** when a new version is created.

## Learning outcomes

After the lab, students should be able to:

1. Explain Launch Templates, template versions, and the distinction between **Default** and **Latest**.
2. Create a reusable EC2 Launch Template using Amazon Linux 2023.
3. Create versions 1, 2, and 3 without overwriting the older versions.
4. Use EC2 **user data** to install NGINX and deploy a static website.
5. Verify the result and troubleshoot cloud-init, SSH, and HTTP connectivity.
6. Remove lab resources to prevent unnecessary charges.

## Prerequisites and cost/security reminders

- AWS account with permission to create EC2 instances, security groups, launch templates, and key pairs; avoid using the root user for daily work.
- An SSH key pair in the selected AWS Region. Keep the private key confidential.
- A default VPC with a public subnet and Internet Gateway, or another configured VPC/public subnet.
- A student machine with a working SSH client. On Linux/macOS, restrict the `.pem` file with `chmod 400 <key>.pem`. On Windows, use OpenSSH/PowerShell and secure key-file permissions as appropriate.
- `t3.micro` is a **teaching example**, not a promise of Free Tier eligibility. EC2 compute, EBS storage, and public IPv4 addresses may incur charges. Review pricing and optionally create an AWS Budget before starting.
- The security group must be created in the **same VPC** where the lab instances will launch.

## Lab overview

| Template version | User data | Expected result |
|---|---|---|
| **V1 — Base** | None | Amazon Linux 2023 EC2; SSH works; no NGINX page |
| **V2 — NGINX** | Install/start NGINX | Default NGINX web page responds on TCP 80 |
| **V3 — Designer** | Install NGINX/Git, deploy Techpro designer site | Custom website appears on TCP 80 |

> We will launch **separate instances** from each version. Changing a template version does **not** rerun user data on instances created earlier.

---

## Part 1 — Create a dedicated Security Group

1. Open **EC2 Console → Network & Security → Security Groups → Create security group**.
2. Enter name `Launch_Temp_Sec_group`, description `Security group for EC2 Launch Template lab`.
3. Select the VPC that contains your lab subnet.
4. Configure **inbound** rules:

   | Type | Protocol/port | Source | Why |
   |---|---|---|---|
   | SSH | TCP 22 | **My IP** (your public IP `/32`) | Instructor/student administration |
   | HTTP | TCP 80 | `0.0.0.0/0` | Make the public demo web page reachable |

5. Keep suitable outbound connectivity so the instance can reach package repositories and GitHub (the default outbound rule normally permits this). Create the group.
6. **Never open SSH to `0.0.0.0/0` for this lab.** If your public IP changes, update the SSH rule.

**Instructor prompt:** Ask students why a publicly accessible website needs inbound TCP 80, while SSH should be limited to trusted IPs.

## Part 2 — Launch Template V1 (baseline)

### Step 1 — Create Launch Template

1. Navigate to **EC2 → Instances → Launch Templates → Create launch template**.
2. Under **Launch template name**, enter `MyLaunchTemplate`.
3. Under **Template version description**, enter `V1 - Base AL2023`.
4. Under **Auto Scaling guidance**, leave the default for this *manual* launch lesson. You can enable guidance when building an Auto Scaling group in a later course.
5. Under **Application and OS Images (AMI)**, choose **Amazon Linux 2023** from Quick Start, using the **x86_64** architecture compatible with `t3.micro`.
   - **Do not paste the old hardcoded AMI ID** `ami-0df8c184d5f6ae949`. AMI IDs are Region-specific and can become outdated.
   - Advanced/CLI users can look up the Amazon Linux 2023 AMI via Systems Manager public parameter `/aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64`.
6. **Instance type:** choose `t3.micro` (or a cost-appropriate compatible alternative).
7. **Key pair:** choose your existing key pair. The console displays the **key pair name**, not the private `.pem` file contents.
8. **Network settings:** select the previously created `Launch_Temp_Sec_group`.
   - For this lab, leave the **subnet** unspecified inside the template, then choose the subnet **when launching**. This also makes the template more reusable in a later Auto Scaling lesson.
   - Avoid conflicting settings under both **Network interfaces** and basic **Security groups**.
9. **Storage:** keep the selected Amazon Linux 2023 root EBS volume settings, or select **gp3** if you deliberately configure the root volume. Do not assume the old `8 GiB gp2` screenshot matches today's AMI defaults.
10. **Resource tags:** add instance tag `Name = Webserver-V1` if the console option is available under launch template resource tags. These are **EC2 instance tags**, not template tags.
11. **Advanced details:** leave **User data** empty. When configurable, prefer **IMDSv2 required** (HTTP tokens required).
12. Select **Create launch template** and confirm **version 1** appears.

### Step 2 — Launch an EC2 instance from V1

1. In **Launch Templates**, select `MyLaunchTemplate` → **Actions → Launch instance from template**.
2. Explicitly choose template **version 1**.
3. Select a **public subnet**. Ensure a **public IPv4 address** is assigned if you plan to connect from the Internet; in some networking setups this needs an explicit setting.
4. Number of instances: **1**. Review and choose **Launch instance**.
5. Wait for both EC2 status checks to pass. Connect by SSH:

   ```bash
   ssh -i "<your-key>.pem" ec2-user@<EC2-PUBLIC-IP>
   ```

   On Linux/macOS, first run `chmod 400 "<your-key>.pem"` locally.

6. Confirm the OS:

   ```bash
   cat /etc/os-release
   ```

7. Try `curl -I http://localhost`. There is **no NGINX installation in V1**, so the HTTP connection normally fails. That is the expected result for the baseline instance.

**Checkpoint:** Template V1 exists and a new Amazon Linux 2023 instance is running.

---

## Part 3 — Launch Template V2 (NGINX via user data)

1. Go to **Launch Templates** → select `MyLaunchTemplate` → **Actions → Modify template (Create new version)**.
2. Choose **version 1** as the **source version**, and enter description `V2 - NGINX`.
3. Change the EC2 instance `Name` tag to `Webserver-V2`.
4. Open **Advanced details → User data**, and paste this shell script **as plain text in the console**:

   ```bash
   #!/bin/bash
   set -e

   # Amazon Linux 2023: install the web server
   dnf install -y nginx

   # Start now and at future boots
   systemctl enable --now nginx
   ```

5. Select **Create launch template** (this creates **version 2**; it does not overwrite V1).
6. Select **Actions → Launch instance from template**, and **explicitly choose version 2**.
7. Choose public subnet/public IPv4 and launch **one new instance**.
8. Wait for EC2 readiness and user-data processing. Open:

   `http://<V2-PUBLIC-IP>`

   The default NGINX landing page should appear. If not, SSH into the **V2 instance** and run:

   ```bash
   sudo systemctl status nginx
   curl -I http://localhost
   sudo tail -n 80 /var/log/cloud-init-output.log
   ```

**Instructor explanation:** EC2 user data shell scripts usually run **once during the initial launch boot**, as `root`. `systemctl enable --now nginx` both starts the service immediately and enables it on later boots.

**Checkpoint:** Version 2 launches a **new** instance and serves the default NGINX page; the V1 instance has not changed.

---

## Part 4 — Launch Template V3 (Techpro designer website)

1. In **Launch Templates**, select `MyLaunchTemplate` → **Actions → Modify template (Create new version)**.
2. Choose **version 2** as the source version; description `V3 - Designer Website`.
3. Change the instance tag to `Name = Webserver-V3`.
4. Replace the V2 user data with the following V3 script:

```bash
#!/bin/bash
dnf update -y
dnf install nginx -y
yum install git -y
systemctl start nginx
cd /usr/share/nginx/html
git clone https://github.com/techproedu/designer.git
chmod -R 777 /usr/share/nginx/html
rm index.html
cp -R ./designer/. .
systemctl restart nginx
systemctl enable nginx
```

5. Create **version 3**.
6. **Launch instance from template → version 3 → choose public subnet/public IPv4 → launch one instance**.
7. Open `http://<V3-PUBLIC-IP>` in the browser. You should see the **Techpro designer** website rather than the default NGINX welcome page.
8. If the page is missing, SSH into V3 and check:

   ```bash
   sudo systemctl status nginx
   ls -la /usr/share/nginx/html
   curl -I http://localhost
   sudo tail -n 100 /var/log/cloud-init-output.log
   ```

**Important security changes compared with the original lab:**
- **Removed** `chmod -R 777 /usr/share/nginx/html`. World-writable document roots are unnecessary and insecure.
- **Removed** the risk of exposing a website's `.git` metadata. The deployment copies site files without copying `.git`.
- Uses `dnf` consistently with Amazon Linux 2023.
- The external GitHub repository is a *teaching dependency*: if it becomes unavailable, use a simple local `index.html` page instead.

**Checkpoint:** A new EC2 instance launched from V3 shows the designer website; V1 and V2 remain unchanged.

---

## Part 5 — Compare versions and configure the default

1. In **Launch Templates**, select `MyLaunchTemplate` and open the **Versions** tab.
2. Verify V1, V2, and V3 each appear as **numbered immutable versions**.
3. Locate **Latest version** and **Default version**. They can differ. A new version does **not automatically become the default**.
4. Optional: select **Actions → Set default version**, and set the default to **version 3**.
5. Launch a test instance **without explicitly selecting a version**, and explain that EC2 uses **Default**, not necessarily Latest.
6. If teaching Auto Scaling, explain that an Auto Scaling group can select **`$Default`**, **`$Latest`**, or an **explicit version number**. This lesson does **not** create an Auto Scaling group, because the core goal is Launch Template versioning.

### Optional AWS CLI verification

Run these only from a machine with the **AWS CLI installed and appropriate temporary credentials**:

```bash
aws ec2 describe-launch-templates \
  --launch-template-names MyLaunchTemplate \
  --query 'LaunchTemplates[*].[LaunchTemplateName,DefaultVersionNumber,LatestVersionNumber]' \
  --output table

aws ec2 describe-launch-template-versions \
  --launch-template-name MyLaunchTemplate \
  --query 'LaunchTemplateVersions[*].[VersionNumber,VersionDescription,DefaultVersion]' \
  --output table
```

Never place access keys or passwords inside a Launch Template or user-data script.

---

## Troubleshooting reference

| Symptom | What to check |
|---|---|
| Cannot SSH | Correct public IPv4, SSH from **My IP**, key pair, username `ec2-user`, public subnet, network ACL and route |
| Browser timeout | TCP 80 inbound, public IPv4/subnet, service started, user data completed |
| Browser shows old page | You launched an older **template version** or opened the public IP of an earlier instance |
| NGINX inactive | `sudo systemctl status nginx`; check installation logs |
| User-data script error | `sudo tail -n 100 /var/log/cloud-init-output.log` and `sudo cloud-init status --long` |
| Designer files missing | GitHub connectivity, DNS, repository availability, file contents under `/usr/share/nginx/html` |
| `curl -I localhost` works but public IP does not | Security group, network route, public IP, OS firewall if configured |

**Key takeaway:** A working local HTTP response proves the web service is active; it does *not* by itself prove external access is allowed.

## Final clean-up (required)

1. **EC2 → Instances:** Terminate the **V1**, **V2**, **V3** instances and any optional test instance. Ensure termination completes.
2. **EC2 → Launch Templates:** Delete `MyLaunchTemplate` when no longer required. Note: deleting a Launch Template does **not** terminate instances already created from it.
3. **EC2 → Volumes:** Verify no unattached, billable EBS volumes were left behind. Do not delete unrelated disks.
4. **EC2 → Security Groups:** Delete `Launch_Temp_Sec_group` once it is no longer attached to any resource.
5. **EC2 → Elastic IPs:** Release any Elastic IP addresses allocated solely for this lab (if used).
6. Review **Billing and Cost Management** and any AWS Budget you configured. Do not delete a student's shared resources.

## Instructor wrap-up questions

1. Is a Launch Template an EC2 instance? **No; it stores launch parameters.**
2. Can we edit V1 in place? **No; create a new template version.**
3. What is the difference between Default and Latest? **Default is used if no version is specified; Latest is the highest version number.**
4. Does changing V2 to V3 update existing instances? **No.**
5. Why shouldn't SSH allow `0.0.0.0/0`? **It exposes port 22 to the public internet.**
6. What does `systemctl enable --now nginx` mean? **Enable NGINX at boot and start it now.**

## Official references (verified October 2026)

- EC2 Launch Templates: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-launch-templates.html
- Create a Launch Template: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-launch-template.html
- Manage template versions and defaults: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/manage-launch-template-versions.html
- Launch templates for Auto Scaling: https://docs.aws.amazon.com/autoscaling/ec2/userguide/launch-templates.html
- Legacy launch configuration restrictions: https://docs.aws.amazon.com/autoscaling/ec2/userguide/launch-configurations.html
- EC2 user data: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html
- Find current AL2023 AMIs with SSM: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/finding-an-ami-parameter-store.html
- EC2 Security Groups: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/creating-security-group.html
