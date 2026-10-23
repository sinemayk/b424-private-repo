# Techpro Education — AWS S3-02 Hands-On (Updated October 2026)

## S3 Lifecycle Management and Cross-Region Replication

**Audience:** Beginner AWS/DevOps students · **Regions:** Source `us-east-1` (N. Virginia); destination `us-east-2` (Ohio) · **Purpose:** Create lifecycle rules, explain timing and small-object limitations, configure private cross-Region replication, and test whole-bucket, prefix and tag filters.

**Classroom assets:** `S3_Lab_Assets.zip` → `Object/` contains `index.html`, `eagle.jpg`, `inside-folder.txt`, `outside_folder.txt`, `tag_file.txt`, and `tag_file 2.txt`. (The latter has a space in its filename.) The source materials use slightly different names in different steps; use the actual files from the archive.

**Important defaults:** Keep **Block all public access ON**, **ACLs disabled**, and default **SSE-S3 encryption** for all buckets in this lab. Replication does **not** require public S3 website hosting or disabling public access. CloudFront and website endpoints are covered in S3-01. Region-to-Region transfer, storage in two Regions and requests can incur charges.

## Learning outcomes

Students will create a lifecycle rule filtered to selected objects, explain the 128 KB transition constraint and minimum retention periods, set up S3 CRR with an IAM role, test all-object/prefix/tag replication, inspect object replication status, and delete resources.

## Part 1 — S3 Lifecycle Management

### Step 1: Create a private bucket

1. Open the Amazon S3 Console and create `techpro-lifecycle-<unique-suffix>` in `us-east-1` (general purpose bucket).
2. Leave **Object Ownership: Bucket owner enforced**, **Block Public Access ON**, default SSE-S3 and **versioning off** for this first exercise.
3. Create an object prefix named `archive/` using **Create folder**. Upload `Object/eagle.jpg` (about 95 KB) if desired as a *small-file counterexample*.
4. For a more useful demonstration, create a local 256 KiB test file and upload it to key `archive/large-test.bin`. On a Unix-like workstation:

   ```bash
   dd if=/dev/urandom of=large-test.bin bs=1024 count=256
   ```

   On Windows, create or reuse a file over 128 KB. The object must be large enough for the default lifecycle transition rules, but it will **not transition immediately** in this class.

### Step 2: Create a lifecycle rule

1. Open your bucket → **Management → Lifecycle rules → Create lifecycle rule**.
2. Rule name: `techpro-archive-retention`.
3. Apply rule to **a subset of objects** with prefix `archive/`. Do not apply a deletion rule to the whole bucket.
4. Under lifecycle actions, enable **Transition current versions**:
   - S3 Standard → **S3 Standard-IA** after **30 days**;
   - (optional follow-up) → **S3 Glacier Flexible Retrieval** after **90 days**.
5. Optionally enable **Expire current versions** after **365 days** in this disposable training bucket. Explain that expiration can permanently remove data in unversioned buckets.
6. Review cost and minimum duration warnings, then save the lifecycle rule.
7. Check **Management → Lifecycle rules** to verify the rule is enabled and properly scoped.

**Why you do not see movement now:** Object age is evaluated over days; new 2024+ lifecycle configurations do not transition objects **smaller than 128 KB by default**. Many files in `Object/` are intentionally very small. Also, S3 Standard-IA has a 30-day minimum storage duration and minimum billable size. For versioned buckets, current-version expiration normally introduces a delete marker; noncurrent version expiration is a separate action.

### Step 3: Validate with AWS CLI (optional)

```bash
aws s3api get-bucket-lifecycle-configuration \
  --bucket techpro-lifecycle-REPLACE-ME \
  --region us-east-1
```

**Done condition:** The rule is enabled and scoped to `archive/`, and the student explains why there is no same-day storage-class change.

## Part 2 — Create private source and destination buckets for CRR

1. Create `techpro-crr-source-<unique-suffix>` in `us-east-1`.
2. Create `techpro-crr-dest-<unique-suffix>` in `us-east-2`.
3. **Enable bucket Versioning on both buckets**, which is mandatory for replication.
4. Use Bucket owner enforced, Block Public Access ON, and **SSE-S3** in both buckets. For simplicity, do not use SSE-KMS (it requires extra key permissions).
5. Upload `Object/index.html` or `Object/eagle.jpg` to the source **before** configuring replication. Label this the **pre-existing object**; it will help demonstrate why live replication does not automatically copy existing keys.

**Important:** Do *not* create public bucket policies or enable static website hosting on either bucket. These are private replication test buckets.

## Part 3 — Create a replication IAM role

**Recommended beginner approach:** Have the S3 Console **create a new IAM role** as part of the replication-rule wizard. Review the resulting role rather than hardcoding a role with `s3:Get*` across many resources.

An example S3 role trust policy is:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": { "Service": "s3.amazonaws.com" },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

For the custom-policy instructor extension, ensure permissions are scoped to the **exact source and destination ARNs** and include read access to source replication configuration/object versions and write/replication operations on destination versions. A suitable baseline for this same-account, SSE-S3 case is:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadSourceConfig",
      "Effect": "Allow",
      "Action": ["s3:GetReplicationConfiguration", "s3:ListBucket"],
      "Resource": "arn:aws:s3:::REPLACE-SOURCE-BUCKET"
    },
    {
      "Sid": "ReadSourceVersions",
      "Effect": "Allow",
      "Action": ["s3:GetObjectVersionForReplication", "s3:GetObjectVersionAcl", "s3:GetObjectVersionTagging"],
      "Resource": "arn:aws:s3:::REPLACE-SOURCE-BUCKET/*"
    },
    {
      "Sid": "ReplicateToDestination",
      "Effect": "Allow",
      "Action": ["s3:ReplicateObject", "s3:ReplicateDelete", "s3:ReplicateTags"],
      "Resource": "arn:aws:s3:::REPLACE-DEST-BUCKET/*"
    }
  ]
}
```

The console-generated role may include other actions based on settings, and advanced cases (cross-account, KMS-encrypted objects, Object Lock and ownership changes) need additional configuration. Do not use this baseline unchanged in those scenarios.

## Part 4 — Replicate all new objects

1. Open the **source** bucket → **Management → Replication rules → Create replication rule**.
2. Name: `TechproCRR-All`; status: **Enabled**.
3. Scope: **all objects** in source bucket.
4. Destination: choose the private destination bucket in `us-east-2`.
5. IAM role: choose **Create new role**, or select the scoped role from Part 3.
6. Leave optional **S3 Replication Time Control**, **replication metrics**, and advanced encryption features disabled unless you specifically want to teach their charges and prerequisites.
7. When asked about **existing objects**, choose **not to create a Batch Replication job** for this basic exercise. Explain that the old object will stay only in source unless you deliberately use **S3 Batch Replication**.
8. Save the rule. Upload a **new** file as `new-after-rule.txt` to source.
9. Check the source object's **Replication status** (`PENDING`, then `COMPLETED`, or `FAILED`). Check the destination bucket for `new-after-rule.txt`.

**Instructor note:** Replication is asynchronous; it is **not guaranteed instantaneous**. S3 Replication Time Control (RTC) is an optional feature with a 15-minute service-level commitment for qualifying replication; check the current AWS SLA for its precise percentage. Do not claim normal replication always completes in 15 minutes.

## Part 5 — Replication by prefix

1. Disable the broad `TechproCRR-All` rule to isolate the next demonstration.
2. Add **new** rule `TechproCRR-Prefix`, filter **prefix `wild/`** (include the trailing slash), target the same private destination, use the same approved IAM role.
3. Upload `outside_folder.txt` to the **root** of source bucket; confirm it does not replicate under the filtered rule.
4. Upload the supplied `inside-folder.txt` as key `wild/inside-folder.txt`.
5. Wait for status and check that only the matching `wild/inside-folder.txt` appears in destination.

**Teaching note:** A prefix filter matches object-key text. `wild/` is a key prefix, not a physical directory.

## Part 6 — Replication by object tag

1. Disable `TechproCRR-Prefix` to avoid overlapping matches in this demonstration.
2. Create rule `TechproCRR-Tag` with a **tag filter**: `Name = eagle`.
3. Upload an untagged test file `Object/tag_file.txt` and verify it does **not** match.
4. Upload the supplied `Object/tag_file 2.txt` (rename locally to `tag_file_2.txt` if preferred), and specify its **object tag during the upload**: `Name=eagle`.
5. Confirm this new tagged object replicates. Do not assume tagging an already-uploaded object later will always cause live replication without a new eligible object version.

**Optional:** Demonstrate S3 Batch Replication for the pre-existing object from Part 2. This requires a Batch Operations job, a separate appropriate IAM role, and additional charges; it is not part of the core beginner lab.

## Validation and troubleshooting

| Symptom | Explanation | Check |
|---|---|---|
| No lifecycle transition after 5 minutes | Transitions are age-based and asynchronous | Rule days, scope, file size threshold |
| Small `eagle.jpg` does not transition | Under default 128 KB threshold | Use `large-test.bin` >128 KB |
| Destination empty | Could be before-rule object, missing Versioning, broken IAM, unmatched filter | Upload a fresh matching object; inspect replication status |
| `FAILED` replication status | Permission/encryption/configuration problem | Review IAM role, versioning, Region, tags |
| Untagged object replicated unexpectedly | Broad rule still active | Disable earlier replication rule; check rule priority and scope |
| Direct bucket URL denies access | Expected private bucket behavior | Inspect through authorized console/CLI |

## Cleanup (mandatory)

1. Disable/delete source bucket replication rules and lifecycle rules.
2. Empty **source and destination** buckets, including every object **version and delete marker**. Merely removing the latest object view will not fully empty a versioned bucket.
3. Delete all three buckets (`lifecycle`, `CRR source`, `CRR destination`) after verifying they are empty.
4. Delete any IAM policy/role made exclusively for this lab after confirming it is not used elsewhere.
5. Check billing for storage, replication requests and cross-Region data transfer.

## Official AWS references

- [S3 Lifecycle](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lifecycle-mgmt.html)
- [Lifecycle transitions and 128 KB rules](https://docs.aws.amazon.com/AmazonS3/latest/userguide/lifecycle-transition-general-considerations.html)
- [Replication requirements](https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication-requirements.html)
- [Replication overview and Batch Replication](https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication.html)
- [S3 Replication Time Control](https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication-time-control.html)
- [S3 Versioning](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Versioning.html)

**End-of-lab condition:** Students can explain and configure a safe lifecycle policy, prove that **new matching objects** replicate across Regions, distinguish whole-bucket / prefix / tag scopes, and clean up their AWS resources.
