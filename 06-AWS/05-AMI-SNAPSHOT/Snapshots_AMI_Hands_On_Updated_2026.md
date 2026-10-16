# Hands-on EC2: Amazon EBS Snapshots, AMIs, and Data Lifecycle Manager

**Techpro Education — AWS & DevOps | Reviewed: October 2026**  
**Environment:** Amazon Linux 2023, Amazon EC2, Amazon EBS, Amazon Data Lifecycle Manager (DLM)  
**Teaching method:** Console-first, followed by validation and troubleshooting.

## Purpose and learning outcomes

Students will learn to:

1. Deploy an NGINX web server on Amazon Linux 2023 using EC2 user data.
2. Identify an EC2 instance's root EBS volume and create an EBS snapshot.
3. Distinguish a **single-volume snapshot**, a **multi-volume snapshot set**, and an **AMI**.
4. Register an AMI from a root-volume snapshot and create a second AMI directly from a running EC2 instance.
5. Launch another EC2 instance from a custom AMI and confirm the web page persists.
6. Understand safe **private** AMI sharing (not public AMI publication).
7. Select a WordPress AMI from the current AWS Marketplace catalog (optional).
8. Configure a tag-based DLM snapshot policy and remove unused resources afterwards.

> **Important safety update:** Do not run `chmod -R 777`, allow SSH from the entire internet, publish classroom AMIs publicly, or disable the account-level AMI block-public-access setting. EC2, EBS, snapshot, public IPv4, and Marketplace charges may apply. Check pricing and your account's Free Tier eligibility before the lesson.

## Prerequisites

- AWS training account with permission to manage EC2 instances, security groups, EBS volumes and snapshots, AMIs, and DLM policies/related service roles.
- Region **US East (N. Virginia), `us-east-1`** for all steps, or replace the Region consistently.
- Default VPC and an appropriate public subnet with an internet gateway route (if the account has no default VPC, use an instructor-prepared VPC).
- An SSH key pair, if SSH is being demonstrated; restrict TCP 22 inbound to your personal public IPv4 address **`x.x.x.x/32`**. Alternatively, use EC2 Instance Connect or SSM Session Manager where supported and configured.
- A budget alert configured ahead of the class; avoid launching optional Marketplace products for every student.
- Use the same VPC and AWS Region when launching copies from an AMI; verify that security groups are correctly selected for **each** new instance.

### Suggested lab security group

| Type | Protocol | Port | Source |  |
|---|---|---:|---|---|
| SSH | TCP | 22 | **Your public IP `/32`** |  |
| HTTP | TCP | 80 | `0.0.0.0/0` |  |

Name: `techpro-snapshots-web-sg`. **No need to open HTTPS (443)** unless you explicitly configure TLS. Do not add a broad inbound `All traffic` rule.

---

## Part 1 — Launch NGINX and create your first EBS snapshot

### 1. Launch the source EC2 instance

Open **EC2 → Instances → Launch instances**:

| Setting | Value |
|---|---|
| Name | `SampleInstance` |
| AMI | **Amazon Linux 2023** (official Amazon-owned) |
| Architecture | **x86_64** for this example |
| Instance type | `t3.micro` (if available and affordable) |
| Key pair | Your lab key pair, only if using SSH |
| VPC | Default or instructor-prepared VPC |
| Subnet | Public subnet |
| Auto-assign public IPv4 | Enable for browser demonstration |
| Security group | `techpro-snapshots-web-sg` |
| Tag | `Backup=techpro-snapshot-lab` (important for Part 6) |
| User data | Script below |

Paste this EC2 **Advanced details → User data** script:

```bash
#!/bin/bash

dnf upgrade -y
dnf install -y nginx

cat > /usr/share/nginx/html/index.html <<'EOF'
<!doctype html>
<html lang="en">
<head><meta charset="utf-8"><title>Techpro Snapshot Lab</title></head>
<body>
  <h1>Techpro Education: NGINX Snapshot Lab</h1>
  <p>This web server was configured with Amazon EC2 user data.</p>
</body>
</html>
EOF

chmod 755 /usr/share/nginx/html
chmod 644 /usr/share/nginx/html/index.html
systemctl enable --now nginx
```

**Why this version?** The core lab does not depend on a third-party repository or wildcard permissions. If you specifically want the original Techpro `designer` website, see **Optional: deploy the designer site** below.

### 2. Validate the running NGINX server

Wait for the instance status checks. Browser: `http://<EC2-PUBLIC-IP>` (the source EC2 instance must have a public address).

If connected by SSH (Linux/macOS terminal):

```bash
chmod 400 your-key.pem
ssh -i your-key.pem ec2-user@<EC2-PUBLIC-IP>
```

Commands *inside the EC2 instance*:

```bash
sudo systemctl status nginx 
curl -I http://localhost
sudo cloud-init status --wait
```

Expected: NGINX is **active (running)**, `curl -I` returns HTTP `200 OK`, and cloud-init ends successfully. The exact result depends on completion of the package installation and network availability.

**Optional: deploy the original Techpro `designer` website**:

```bash
sudo dnf install -y git
git clone --depth 1 https://github.com/techproedu/designer.git /tmp/techpro-designer
ls /tmp/techpro-designer
sudo cp -a /tmp/techpro-designer/. /usr/share/nginx/html/
sudo find /usr/share/nginx/html -type d -exec chmod 755 {} +
sudo find /usr/share/nginx/html -type f -exec chmod 644 {} +
sudo systemctl reload nginx
```

If the repository structure changes or no index page exists at its root, inspect and adapt the copy step. Do **not** set permissions to `777`.

### 3. Create a multi-volume EBS snapshot set for the instance

1. In **EC2 → Snapshots → Create snapshot**, set **Resource type = Instance**.
2. Select `SampleInstance` by instance ID.
3. Include the root volume (and any additional EBS volumes, if present).
4. Set a description and a tag such as `Name=SampleInstance-snapshot-set`.
5. Choose **Create snapshot**. Wait until all created snapshots show **Completed**.

**Instructor explanation:** Instance resource type creates **a snapshot for each included EBS volume**, not a single file containing a complete EC2 instance or memory state. In a single-root-volume demo you will see one snapshot. Multiple EBS volumes produce multiple snapshots.

### 4. Identify the ROOT volume snapshot

1. Open `SampleInstance` → **Storage** and copy the root **Volume ID** and root device name.
2. Open **EC2 → Snapshots → Owned by me** and identify the snapshot corresponding to that root Volume ID.
3. Write down the Snapshot ID (`snap-...`) and confirm **Completed**.

> If you intend to build an AMI from a snapshot, you need the **snapshot of the root EBS volume**, not an arbitrary data volume. AWS generally recommends **Actions → Image and templates → Create image** for full instance-based AMI creation, especially when the instance has additional volumes.

---

## Part 2 — Create an AMI directly from the EC2 instance

This is the recommended and simplest path for a classroom example.

1. Open **EC2 → Instances**, select `SampleInstance`.
2. Choose **Actions → Image and templates → Create image**.
3. Name: `techproAMI-Instance`.
4. Description: `NGINX web server copied from SampleInstance`.
5. Keep the **Reboot instance** option selected for a more consistent point-in-time filesystem state, unless the instructor has a reason to choose otherwise.
6. Review the proposed block-device mappings and tags. Choose **Create image**.
7. Open **EC2 → AMIs → Owned by me** and wait until the new AMI is **Available**.
8. Open **EC2 → Snapshots** and find the EBS snapshot(s) associated with this AMI.

**Validate the result:**

1. Select `techproAMI-Instance` → **Launch instance from AMI**.
2. Name: `RestoredFromInstanceAMI`.
3. Select a compatible instance type and the lab security group (again, do not open SSH to everyone).
4. Launch; wait for instance status checks. Check `http://<NEW-PUBLIC-IP>`.
5. Expected: the same Techpro web page appears **even without re-entering the original user-data script**, because the page and enabled NGINX service were present in the image's root filesystem.

**Key takeaway:** An AMI is reusable server launch material; it is not a live synchronization service. Changes made on a running source server after the AMI was created do not automatically appear in instances later launched from that AMI.

---

## Part 3 — Create a volume from the snapshot and an AMI from the root snapshot

### Option A: Restore data as a new EBS volume (recommended snapshot recovery demonstration)

1. In **EC2 → Snapshots**, select the completed root-volume snapshot.
2. Choose **Actions → Create volume from snapshot** (or create a new volume and select this snapshot).
3. Choose an Availability Zone compatible with an instance to which you may attach it; example: the source instance's AZ.
4. Use a reasonable `gp3` configuration. Do not enable optional paid fast restore or provisioned initialization settings for this basic lab.
5. Name the restored volume `RestoredVolume` and choose **Create volume**.
6. Wait for **Available**. Inspect the new Volume ID (`vol-...`), size, encryption, and AZ.
7. Optional instructor extension: attach the restored volume as a **secondary** disk to a compatible EC2 instance in the **same AZ** and inspect the existing filesystem. **Do not run `mkfs` on this restored disk**; formatting it would erase the copied data. Inspect the filesystem first (`lsblk -f` and `blkid`), then use safe read-only mounting if you know the correct device/partition.

**Important:** EBS volumes are bound to one Availability Zone. Snapshots are Region-level resources that can be copied across Regions.

### Option B: Create an AMI from the root-volume snapshot (advanced demonstration)

1. In **EC2 → Snapshots**, select the **root-volume** snapshot, not an arbitrary data snapshot.
2. Choose **Actions → Create image from snapshot**.
3. Image name: `techproAMI-RootSnapshot`.
4. Match the **architecture (`x86_64`)**, **root device name**, **virtualization type (HVM)**, and applicable launch settings to the source Linux image. Check **EC2 instance → Storage** and the source AMI properties; use the AWS documentation if any setting is unclear.
5. Register the image and wait for **EC2 → AMIs → Owned by me → Available**.
6. Launch a compatible EC2 instance and test the NGINX page.

**Caution:** This path reconstructs an AMI from a root disk snapshot. It is not always a complete substitute for an AMI created from an instance (for example, a multi-volume server may require additional block-device mappings). With commercial or licensed images, use the correct image billing and licensing information. Prefer Part 2 for normal operations.

### Optional: cross-Region backup exercise

1. Choose a snapshot you own → **Actions → Copy snapshot**.
2. Choose the destination Region, e.g. `us-west-2`; review encryption/KMS and data-transfer/storage charges.
3. In the destination Region, locate the copied snapshot; create an EBS volume **in an AZ of that destination Region** if desired.
4. Delete the extra copy/volume during cleanup.

---

## Part 4 — AMI sharing: private  or Public

### Private Sharing
1. Open **EC2 → AMIs → Owned by me**, choose `techproAMI-Instance`.
2. Show **Actions → Edit AMI permissions** and explain the *Private* versus *Public* options, without changing to Public.
3. Explain that, if the instructor has a controlled **second AWS account**, a privately shared AMI can grant launch permission to that account ID.
4. If the AMI's EBS snapshots are encrypted, cross-account launch/sharing may additionally require correct customer-managed KMS key and snapshot permissions. Some encrypted/default-key arrangements cannot be shared as-is.
5. Keep **Block public access for AMIs enabled** and leave the AMI private.

### Public Sharing
Select the  `TechproAMI_3`

- Click `Edit AMI permissions` from the `Actions` menu.

- Select `AMI availability` -----> `Public` and click `Save Changes`.

- It will be public after a while. Share the AMI ID.

- Tell the student to go to Instance-----> Launch Instance------> Choose AMI------> Community AMI and paste the "AMI ID" in the search bar.

- Delete all AMIs and Snapshots. (We first derigester AMIs and then delete snapshots)

**Key takeaway:** Public AMI publishing is a production governance decision, not a beginner lab prerequisite.

---

## Part 5 — WordPress from AWS Marketplace (optional instructor demonstration)

1. Open **EC2 → Launch instances** and browse the current **AWS Marketplace AMIs**.
2. Search `WordPress`, identify a reputable verified publisher, and check the product's latest documentation. Ex:"WordPress Certified by Bitnami and Automattic" (Free tier)(ami-047738b584476b2fe)

3. Review **software pricing, license/usage terms, instance requirements, storage, operating system, and Region support** before accepting terms.
4. Select a suitable instance type and storage configuration; an optional WordPress lab can cost money even when the EC2 type otherwise qualifies for introductory offers.
5. Security group: HTTP 80 from public for initial demo, SSH 22 restricted to your IP; enable HTTPS 443 only with an actual HTTPS configuration.
6. Launch and follow the **chosen publisher's current documentation** for initial username, password and application administration URL (often `/wp-admin`). **Do not assume credentials always appear in EC2 system logs.**
7. Test the WordPress page and promptly terminate the instance afterwards. Check EBS volumes and any allocated public IP addresses.

Instructor note: This part is optional and should use a **single demonstration instance** rather than one Marketplace instance per student.

---

## Part 6 — Automate EBS snapshots with Amazon Data Lifecycle Manager (DLM)

### 1. Confirm the target tag

Select **EC2 → Instances → SampleInstance → Tags** and confirm:

```text
Key:   Backup
Value: techpro-snapshot-lab
```

This tag is used to select your **instance** for a tag-based custom DLM EBS snapshot policy. Do not confuse **policy tags** with **target resource tags**.

### 2. Create the lifecycle policy

Open **EC2 → Elastic Block Store → Lifecycle Manager → Create lifecycle policy**. AWS Console labels may differ slightly as the interface evolves.

| Setting | Lab value |
|---|---|
| Policy type | **EBS snapshot policy** (custom/tag-based) |
| Target resource type | **Instance** |
| Target resource tag | `Backup=techpro-snapshot-lab` |
| Description | `Techpro lab daily snapshot policy` |
| IAM role | Default DLM service role if available |
| Schedule | `Daily`, every 24 hours |
| Starting time | A valid time window shown by the console, e.g. around `03:00 UTC` |
| Retention type | Count |
| Retention | 5 snapshots / snapshot sets, as the console describes |
| Copy tags from source | Optional: enabled |
| Cross-Region copy | Disabled for this basic lab |
| Cross-account sharing | **Disabled** for this basic lab |
| Policy status | Enabled for demonstration; disable/remove after class |

> A DLM snapshot policy can use Instance targets to create multi-volume snapshot sets, and a separate **EBS-backed AMI policy** can automate AMI creation. DLM does not automatically manage unrelated snapshots created manually. A DLM schedule is not guaranteed to produce a snapshot immediately after clicking Save.

### 3. Validate

1. Check the policy appears in **Lifecycle Manager** with the intended tag, policy state, and schedule.
2. Wait for the configured schedule if you specifically need an automatic snapshot to appear; use manual snapshots in Parts 1–3 for immediate classroom validation.
3. After a scheduled run, locate generated snapshots; compare descriptions and tags to the DLM policy.
4. Explain retention: older **DLM-managed** snapshots are cleaned up according to policy rules, not every snapshot in the account.

### 4. Cleanup

After the demonstration, **disable and delete the training policy** and inspect any already-created snapshots. Removing a DLM policy does not guarantee that all older snapshots are deleted.

---

## Troubleshooting and checks

| Problem | Checks |
|---|---|
| Web page times out | EC2 is running; public IPv4 assigned; public subnet route; inbound TCP 80 allowed |
| User data seems ignored | User data normally runs on first boot; `sudo cloud-init status --wait` and inspect logs |
| Snapshot stays pending | Allow time for EBS snapshot completion; refresh EC2 Snapshots and check Region |
| Create-image option unavailable | Confirm root EBS snapshot versus other resources; use instance → Create image for normal AMI workflow |
| Launched AMI has no website | Check security group 80, matching AMI/Region/architecture, NGINX enablement and application files |
| DLM creates no snapshots | Check policy enabled, schedule time, target tags on correct instance, permissions, and DLM service role |
| Cannot share an encrypted AMI | Review snapshot encryption, customer-managed KMS key and cross-account permissions; do not disable public sharing safeguards |

### Optional CLI verification (run from a configured AWS CLI environment)

```bash
aws sts get-caller-identity
aws ec2 describe-images --owners self --region us-east-1 \
  --query 'Images[*].[ImageId,Name,State]' --output table
aws ec2 describe-snapshots --owner-ids self --region us-east-1 \
  --query 'Snapshots[*].[SnapshotId,VolumeId,State]' --output table
aws dlm get-lifecycle-policies --region us-east-1
```

These commands require appropriate IAM permissions and a configured AWS CLI profile, AWS IAM Identity Center session, or temporary role credentials. Do not create permanent access keys just for this lab.

---

## Mandatory end-of-class cleanup checklist

- [ ] Disable/delete the lab DLM policy; inspect snapshots it already created.
- [ ] Terminate **SampleInstance**, **RestoredFromInstanceAMI**, and any other lab/WordPress EC2 instances.
- [ ] Deregister custom AMIs and choose to delete associated snapshots if available; inspect any snapshots still present and delete ones not needed.
- [ ] Delete the manually created snapshot or snapshot set after confirming no remaining AMI/restore dependency.
- [ ] Delete any additional **available** EBS volumes, cross-Region copies, or orphan resources created for the exercise.
- [ ] Release any unused Elastic IP address and delete the lab security group if no resources still reference it.
- [ ] Review **Billing and Cost Management** for EBS snapshot storage, EBS volumes, EC2 compute, IPv4 charges and Marketplace software.

**Done condition:** At least one verified custom AMI was used to launch a working NGINX page, students can explain why snapshot and AMI serve different purposes, the DLM policy was configured, and all lab resources have been reviewed for cleanup.

---

## Official AWS documentation (verified October 2026)

- [Create an EBS-backed AMI (from instance or root snapshot)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/creating-an-ami-ebs.html)
- [Create a snapshot of an EBS volume](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-create-snapshot.html)
- [Create multi-volume EBS snapshots](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-create-snapshots.html)
- [Overview of EBS snapshots](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-snapshots.html)
- [Create an EBS volume from a snapshot](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-creating-volume.html)
- [Copy EBS snapshots](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-copy-snapshot.html)
- [Make AMIs public: sharing and security considerations](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/sharingamis-intro.html)
- [Amazon Data Lifecycle Manager](https://docs.aws.amazon.com/ebs/latest/userguide/snapshot-lifecycle.html)
- [Create custom DLM policies](https://docs.aws.amazon.com/ebs/latest/userguide/snapshot-ami-policy.html)
- [Deregister AMI and avoid continuing costs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/deregister-ami.html)
