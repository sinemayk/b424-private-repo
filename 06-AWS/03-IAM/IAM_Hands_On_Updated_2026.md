# Hands-on IAM-01 — Identity and Access Management (Updated October 2026)

**Course:** Techpro Education — AWS & DevOps  
**Region for EC2 lab:** US East (N. Virginia), `us-east-1`  

> **Security baseline:** Use AWS IAM Identity Center for human access when it is available, MFA for interactive sign-in, temporary credentials for CLI/applications, and least-privilege permissions. The IAM-user steps are retained to explain IAM fundamentals in a controlled teaching account—not as the recommended production identity pattern. **Never share the root password, MFA code, access key ID/secret key, or downloaded credentials on a class screen.**

## Learning outcomes

After completing this lab, students can:

1. Explain the root user, IAM users, IAM Identity Center, groups, policies, and roles.
2. Distinguish authentication from authorization.
3. Describe console credentials, long-term access keys, and temporary role credentials.
4. Create an IAM user and group for a restricted classroom example.
5. Create and test a resource-scoped S3 policy.
6. Explain an IAM role's trust policy versus its permissions policy.
7. Attach an EC2 role and validate that the AWS CLI uses temporary credentials.
8. Troubleshoot sign-in and permission errors without exposing credentials.
9. Remove all temporary lab resources and check potential charges.

## Prerequisites and safe preparation

- An **instructor-approved AWS account** with permission to perform the activities. Avoid the organization's production account.
- **MFA configured**, especially on the root account. Root MFA is required for root console sign-in.
- An existing administrative **role / federated sign-in** for routine work (prefer IAM Identity Center). If the account has not been configured, the account owner must complete initial security setup privately.
- For Part 2, permission to manage IAM users, groups, and policies; IAM is **global** (not tied to an EC2 Region).
- For Part 4, permission to create an EC2 role, instance profile, EC2 instance and a private S3 test bucket/object.
- EC2 may incur charges for instance usage, EBS, public IPv4 and other resources. Students should validate current prices and any account credits before starting.
- Record the names of **only resources created for this lab** so they can be cleaned up afterward.

---

## Part 1 — Root account and access-management overview

### Step 1 — Understand the AWS sign-in identities

Explain to students:

| Identity / access type | What it is | Recommended use |
|---|---|---|
| AWS account root user | Account-owner sign-in with powerful privileges | Only tasks requiring root credentials; MFA required |
| IAM Identity Center user | Workforce identity accessing roles/permission sets | Preferred routine human access |
| IAM user | Long-term identity in one AWS account | A limited number of special cases and this classroom demonstration |
| IAM role | Assumable AWS identity with trust and permission policies | Applications, EC2, delegated access and short-lived sessions |
| IAM user group | Collection of IAM users inheriting identity-policy permissions | Group users by shared permissions; **groups cannot sign in or assume roles** |

**Do not routinely sign in as root.** In particular, do not create root access keys.

### Step 2 — Check IAM dashboard and account alias

1. Sign in as the authorized lab administrator (prefer a role-based sign-in).
2. Navigate to **IAM > Dashboard** and identify the account ID and IAM sign-in URL.
3. If the account's alias is not already set and the instructor approves, create an **AWS account alias**.
4. Explain that an alias simplifies the sign-in URL but **is not a secret** and does not replace MFA.

**Validation:** Can students tell the difference between an AWS account alias and an IAM username?

### Optional account-owner demonstration — Billing access

Some IAM lab versions instruct students to sign in as root to enable billing for IAM identities. This is **not a required IAM-user creation step**.

Only the account owner/instructor, if needed and permitted:

1. Sign in as **root** with MFA for this specific account-setting task.
2. Open **Billing and Cost Management > Account**.
3. Under **IAM user and role access to Billing information**, choose **Edit > Activate IAM access > Update**.
4. Sign out of root immediately.
5. Grant a separate IAM identity or role the **specific billing permissions** required. Merely activating IAM access **does not grant billing permissions**.

**Validation:** Verify that the intended non-root principal sees only the billing pages it is authorized to see. Do not assign `AdministratorAccess` merely to demonstrate billing.

### Optional cost-monitoring exercise — AWS Budgets

Open **Billing and Cost Management > Budgets > Create budget**, and choose a suitable template such as a **Monthly cost budget**. Add the instructor's notification email and a small appropriate lab threshold. A **Zero spend budget** tracks selected Free Tier overages and is not a comprehensive zero-dollar spending guarantee.

> **Important:** Budgets send notifications; they do **not** automatically stop EC2 instances or enforce an account-wide hard spending limit.

---

## Part 2 — IAM users, groups and scoped policies

### Step 1 — Create a classroom IAM user

IAM Console > **Users > Create user**:

1. Enter `lab-reader-01` (or a similar unique name).
2. **Do not enable console access** if the task requires programmatic IAM concepts only. For a console sign-in demonstration, optionally enable console access, set a secure one-time password and configure MFA before use.
3. Do **not** attach `AdministratorAccess` for daily use. Do not create access keys as a default step.
4. Create the user and verify it appears in the Users list.

**Validation:** The user has no permissions unless a policy, group, or other relevant resource-based grant provides them.

### Step 2 — Prepare a private S3 bucket for the demonstration

If a designated lab bucket is not already provided:

1. As an authorized lab administrator, open **S3 > Create bucket**.
2. Choose a globally unique name, such as `techpro-iam-lab-<unique-suffix>`; record its exact name.
3. Leave **Block all public access** enabled and use the default modern Object Ownership settings.
4. Upload a harmless small file such as `demo.txt`, containing `Hello from Techpro IAM lab`.

Do **not** make the bucket or object public. Deleting the test bucket later is optional **only if it was created exclusively for this lab**.

### Step 3 — Create a least-privilege S3 read policy

Open **IAM > Policies > Create policy > JSON**. Replace `YOUR-LAB-BUCKET` with the exact bucket name:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ListOneBucket",
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::YOUR-LAB-BUCKET"
    },
    {
      "Sid": "ReadObjectsInOneBucket",
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::YOUR-LAB-BUCKET/*"
    }
  ]
}
```

Review the policy using the IAM editor and applicable IAM Access Analyzer checks. Name it `TechproLabS3ReadOnly`.

**Explain the JSON:**

- `Version` is the IAM policy-language version (normally `2012-10-17`), **not today's date**.
- `Statement` is an array of permission rules.
- `Effect` can be `Allow` or `Deny`.
- `Action` identifies the AWS operations; `s3:ListBucket` and `s3:GetObject` have different resource requirements.
- `Resource` restricts permissions to a bucket ARN or an object ARN, respectively.
- A `Condition` could impose additional restrictions, but is not necessary here.

**Validation:** Check that both statements name the specific private bucket, not wildcard resources, and do not use `s3:*`.

### Step 4 — Create IAM user groups

In **IAM > User groups**, create the following classroom groups:

| IAM group | Classroom scenario | Permissions |
|---|---|---|
| `Database` | Database team | Discuss a task-scoped policy; no full administrator policy |
| `Developer` | Developer reading a lab S3 bucket | Attach `TechproLabS3ReadOnly` |
| `AWS_DEVOPS` | Operations team | Discuss role assumption; do not default to broad `PowerUserAccess` |

Add `lab-reader-01` to `Developer`. Check the user's **Permissions** tab for group-derived policies.

**Explain:** An IAM group is not an authenticated principal and cannot assume an IAM role. Users can inherit a policy from the group and can have other policies attached. `PowerUserAccess` is broad and not a least-privilege classroom policy.

**Challenge:** If a user is removed from `Developer`, the group-derived S3 policy no longer applies. Other applicable policies may still grant access.

### Step 5 — Check the restricted user

If console access was deliberately enabled, use the IAM account-specific sign-in URL, the training user's password, and its MFA method. Test whether the user can see the permitted S3 resource. Because the S3 Console may need additional list actions to display all buckets, not seeing a bucket in the general list **does not prove** `s3:GetObject` is denied.

Do not create permanent IAM-user CLI keys just to test this task. The repeatable CLI workflow uses the instructor-provided Identity Center role or the EC2 instance role in the next sections.

---

## Part 3 — Credentials, CLI and troubleshooting

### Step 1 — Preferred CLI authentication: IAM Identity Center

**Prerequisite:** Identity Center is already configured, and the instructor has assigned a role/permission set to the student's identity. Otherwise, demonstrate this step centrally or skip it.

Install or verify AWS CLI version 2:

```bash
aws --version
aws configure sso
```

Complete the interactive wizard with the organization's SSO start URL/issuer URL, Identity Center Region, account, role, and profile name (example: `lab`). Then:

```bash
aws sso login --profile lab
aws sts get-caller-identity --profile lab
```

**Expected:** AWS account ID and an ARN of the authenticated session. Replace `lab` with the profile name you chose.

**Alternative:** If Identity Center is unavailable, students may use AWS CloudShell when signed into an already-authorized non-root Console principal and run `aws sts get-caller-identity`. Do not create a new long-term access key for the sake of the demonstration.

### Step 2 — Forgot console password

An authorized IAM administrator can open **IAM > Users > select user > Security credentials > Console sign-in** and reset that user's console password if it has one. Configure or require MFA as appropriate. Do not show new passwords to the class. Identity Center credentials must be recovered through the configured identity provider instead.

Check the account alias, username, password, MFA, permission boundaries, and sign-in account.

### Step 3 — Lost a secret access key

A secret access key cannot be retrieved after initial creation. **No IAM-user long-term keys are required in this lab.** For exceptional approved legacy workflows, create a new key securely, update the application, verify it works, deactivate the old key, and delete it. Never place keys in slides, emails, repositories, or EC2 user data.

Click User>>>>> Select user---->>Security Credential--->> Go Access keys ---> Deactivate---->> Delete --->>>Create new 

### Step 4 — Investigate `AccessDenied`

Common causes: wrong AWS identity/profile; missing `Allow`; an applicable explicit `Deny`; Organizations controls, session policies or permissions boundaries; wrong S3 ARN; missing role trust or `sts:AssumeRole` permission.

```bash
aws sts get-caller-identity --profile lab
aws s3 ls s3://YOUR-LAB-BUCKET --profile lab
```

The S3 command only succeeds if **this session's** effective permissions include `s3:ListBucket` for the named bucket. Attaching a policy to a different IAM user does not automatically grant the CLI session those permissions.

---

## Part 4 — Create IAM role, attach to EC2 and access S3

### Step 1 — Create a least-privilege EC2 role

In **IAM > Roles > Create role**, choose:

1. **Trusted entity:** AWS service.
2. **Use case:** EC2.
3. **Permissions:** Attach `TechproLabS3ReadOnly` (the custom policy from Part 2).
4. **Role name:** `TechproEC2S3ReadRole`.

A typical simplified trust policy is:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {"Service": "ec2.amazonaws.com"},
      "Action": "sts:AssumeRole"
    }
  ]
}
```

**Trust policy = who may assume the role. Permissions policy = what this role is allowed to do.**

### Step 2 — Launch an EC2 instance

| EC2 setting | Recommended lab choice |
|---|---|
| Region | N. Virginia, `us-east-1` |
| AMI | **Amazon Linux 2023**, not “Amazon Linux 3” |
| Instance type | `t3.micro` if eligible, or another suitable low-cost option |
| Network | Default VPC, if available, with a usable subnet |
| IAM instance profile | `TechproEC2S3ReadRole` |
| Instance metadata | Prefer IMDSv2 required |
| Security group | SSH TCP/22 from **My IP /32**, not `0.0.0.0/0` |
| HTTP | Not needed in this IAM exercise |
| Storage/public IPv4 | Use only what is needed; charges may apply |

If SSH access is required, use the **EC2 SSH key pair** (which is not an IAM access key):

```bash
chmod 400 /path/to/YOUR-KEY.pem
ssh -i /path/to/YOUR-KEY.pem ec2-user@YOUR-EC2-PUBLIC-IP
```

On Windows, OpenSSH, Git Bash, and WSL may use different file permission conventions. Session Manager may be used instead when the lab has the necessary SSM role permissions and connectivity.

### Step 3 — Verify EC2's identity

Inside the instance:

```bash
aws --version
aws sts get-caller-identity
```

Expected assumed-role ARN pattern (values vary):

```text
arn:aws:sts::123456789012:assumed-role/TechproEC2S3ReadRole/i-xxxxxxxxxxxxxxxxx
```

**Never put permanent IAM-user keys in an EC2 instance** using `aws configure` for this lab. AWS SDK and CLI can get the instance role's short-lived credentials automatically.

### Step 4 — Verify read-only S3 access

```bash
aws s3 ls s3://YOUR-LAB-BUCKET
aws s3 cp s3://YOUR-LAB-BUCKET/demo.txt /tmp/iam-demo.txt
cat /tmp/iam-demo.txt
```

**Expected:** List and download succeed if bucket policy and other controls permit them. This works through the role's `s3:ListBucket` and `s3:GetObject` permissions on the private lab bucket.

For a safe expected-denial example (read-only API call):

```bash
aws iam list-users
```

**Expected:** AccessDenied, unless another permission unexpectedly grants IAM list access. If it succeeds, inspect effective policies.

### Step 5 — Troubleshooting

- No `aws` command? Verify/install AWS CLI v2 according to official AWS documentation.
- Wrong role? Check **EC2 > instance > Security > IAM role** and `aws sts get-caller-identity`.
- `AccessDenied`? Verify both role policies, actual bucket name, object path, and other AWS policy controls.
- Login attempts use unexpected credentials? Check AWS environment variables and configured profiles, which may override the instance-role provider.
- Recent IAM edits? Allow a short propagation delay, then retry.
- SSH fails? Check the security group's **My IP /32** rule, subnet route, public IP, key-pair file, and username `ec2-user`.

---

## Cleanup — Always complete

**Remove only lab-owned resources, not shared/production items.**

1. Terminate the lab EC2 instance; inspect remaining EBS and related resources.
2. Empty/delete the S3 test bucket **only if it was created for this lab**.
3. Remove `lab-reader-01` from groups; delete the lab user and groups.
4. Detach and delete the demo role and custom S3 policy after they are no longer in use.
5. Remove any special-case access keys that were actually created.
6. Check Billing, Cost Explorer and AWS Budgets. **Budgets are alerts, not automatic spending blocks.**

### Final instructor questions

1. Can an IAM group assume a role? Why not?
2. What is the difference between the root user and an IAM Identity Center user?
3. How do trust policies differ from permission policies?
4. Why are S3 bucket and object ARNs distinct in the policy?
5. Why does an EC2 instance use temporary role credentials instead of permanent access keys?

## Official AWS references (reviewed October 2026)

- IAM security best practices: https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html
- Workforce identity planning: https://docs.aws.amazon.com/IAM/latest/UserGuide/gs-identities.html
- Root security and required MFA: https://docs.aws.amazon.com/IAM/latest/UserGuide/root-user-best-practices.html
- IAM groups: https://docs.aws.amazon.com/IAM/latest/UserGuide/id_groups.html
- Policy evaluation: https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html
- IAM roles: https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html
- Temporary EC2 role credentials: https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_use-resources.html
- Long-term access keys: https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html
- AWS CLI and IAM Identity Center: https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sso.html
- Billing access: https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/control-access-billing.html
- AWS Budgets: https://docs.aws.amazon.com/cost-management/latest/userguide/budget-templates.html
- Amazon Linux 2023: https://docs.aws.amazon.com/linux/
