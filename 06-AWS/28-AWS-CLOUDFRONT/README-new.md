# Hands-on CF-01 : Configuring Cloudfront with Route53, ACM and S3 as Static Website

## Part 1 - Creating a Certificate

- Go to Certificate Manager service and hit "Request a certificate"

- Select "Request a public certificate" and hit "Next"

```text

      -  Domain names               : "[your donamin name].net" (naked domain name)

      -  Select validation method   : "DNS validation"

      -  Key algorithm              : RSA 2048
  
      -  Add tags                   : Skip this part
  
      -  Review and click "Request"
```

- On Certificates page Click on your newly created certificate.

- Under Domains pane click "Create record in Route 53" button.

- Click "Create records" on the pop-up menu.

- It takes a while to be ready (Issued).  

## Part 2 - Creating a Static WebSite Hosting

1. Go to S3 service and create a bucket with domain name: "[your-donamin-name].net"

- Public Access Enabled
- Upload Files named "index.html" and "ryu.jpg" in "static-web-v1" folder
- Permissions >>> Bucket Policy >>> Edit >>> Paste bucket Policy - Don't forget to change bucket name in arn.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::BUCKET_NAME_HERE/*"
        }
    ]
}
```

- Properties >>> Static Website Hosting >>> Edit >>> Enable >>> Index document : index.html

## Part 3 - Create CloudFront Distribution

* First, we need to copy the endpoint of the S3 static website bucket.

* Go to **S3**.

* Click on your S3 static website bucket.

* Go to:

```text
Properties → Static website hosting
```

* Copy the **Bucket website endpoint**.

Example:

```text
your-domain-name.net.s3-website-us-east-1.amazonaws.com
```

Important: Copy the endpoint **without** `http://`.

---

### Create CloudFront Distribution

* Go to **CloudFront** service.

* Click **Create a CloudFront distribution**.

### Origin Settings

Configure the origin as follows:

```text
Origin domain:
    Paste the S3 static website endpoint manually

Origin type:
    Custom origin
```

Important:

Do **not** select the S3 bucket from the dropdown list.

Because we are using **S3 Static Website Hosting**, we must use the **website endpoint** manually as a custom origin.

Example:

```text
your-domain-name.net.s3-website-us-east-1.amazonaws.com
```

### Protocol Settings

For the origin protocol, use:

```text
Origin protocol policy:
    HTTP only
```

Reason:

S3 static website endpoints support HTTP, not HTTPS.

CloudFront will provide HTTPS for users, but CloudFront connects to the S3 website endpoint over HTTP.

---

### Default Cache Behavior

Configure:

```text
Viewer protocol policy:
    Redirect HTTP to HTTPS

Allowed HTTP methods:
    GET, HEAD

Cache policy:
    CachingOptimized
```

---

### Web Application Firewall, WAF

For this basic hands-on:

```text
Do not enable security protections
```

For production environments, AWS WAF is recommended.

---

### Settings

Configure:

```text
Alternate domain name, CNAME:
    [your-domain-name].net
    www.[your-domain-name].net

Custom SSL certificate:
    Select your newly created ACM certificate

Default root object:
    Leave empty
```

Important:

Because we are using the **S3 static website endpoint**, S3 already handles the index document.

So in this specific hands-on, you can leave **Default root object** empty.

If you add `index.html`, it usually still works, but with S3 website hosting, the website endpoint already knows the index document from the S3 Static Website Hosting settings.

---

* Leave the other settings as default.

* Click **Create distribution**.

* It may take some time for the distribution to be deployed.

* Check that the distribution status becomes:

```text
Enabled
```

or:

```text
Deployed
```

* When it is deployed, copy the **Distribution domain name**.

Example:

```text
d123456abcdef8.cloudfront.net
```

---

### Test CloudFront Domain

Open the CloudFront distribution domain in the browser:

```text
https://d123456abcdef8.cloudfront.net
```

You should see your static website content.

### Key Takeaway

When S3 Static Website Hosting is used, CloudFront origin must be the **S3 website endpoint**, not the private S3 REST endpoint.

---

## Part 4 - Creating Route 53 Record Sets, Alias

* Go to:

```text
Route 53 → Hosted zones
```

* Click on your public domain name.

* Click **Create record**.

Create the root domain record:

```text
Record name:
    Leave empty

Record type:
    A

Alias:
    Turn on

Route traffic to:
    Alias to CloudFront distribution

Region:
    US East (N. Virginia) [us-east-1]

Distribution:
    Choose your CloudFront distribution endpoint

Routing policy:
    Simple routing
```

* Click **Create records** at the bottom.

---

### Optional: Create www Record

Create another record for `www`:

```text
Record name:
    www

Record type:
    A

Alias:
    Turn on

Route traffic to:
    Alias to CloudFront distribution

Region:
    US East (N. Virginia) [us-east-1]

Distribution:
    Choose your CloudFront distribution endpoint

Routing policy:
    Simple routing
```

* Click **Create records**.

---

### Test the Domain

Go to the target domain name in the browser:

```text
https://[your-domain-name].net
```

Also test:

```text
https://www.[your-domain-name].net
```

Check if it is working with the **https** protocol.

Show the content of the web page.

### Key Takeaway

Route 53 sends users to CloudFront, and CloudFront serves the website from the S3 static website endpoint.

---

## Part 5 - Configuring CloudFront Distribution

## Step-1 - Invalidation

CloudFront caches website files. If we update a file in S3, CloudFront may still show the old cached version.

* Go to your S3 bucket hosting the website.

* Replace the old `ryu.jpg` file with a new `ryu.jpg` file.

Important:

The file name in your HTML is:

```text
ryu.jpg
```

So if your `index.html` uses:

```html
<img src="ryu.jpg">
```

then the new image should also be uploaded as:

```text
ryu.jpg
```

If the image is inside a folder, for example:

```text
image-v2/ryu.jpg
```

then your HTML must also point to that path:

```html
<img src="image-v2/ryu.jpg">
```

---

### Test Before Invalidation

* Go to the target domain name in the browser:

```text
https://[your-domain-name].net
```

* Notice that the image may not change immediately.

Reason:

```text
CloudFront is still serving the cached old image.
```

---

### Create Invalidation

* Go to **CloudFront**.

* Select the newly created distribution.

* Select the **Invalidations** tab.

* Click **Create invalidation**.

* Enter the path:

```text
/ryu.jpg
```

If the image is inside a folder, use the full path:

```text
/image-v2/ryu.jpg
```

* Click **Create invalidation**.

* Wait until the invalidation status becomes:

```text
Completed
```

* Check the website again.

* If the browser still shows the old image, use hard refresh:

```text
Windows/Linux:
    Ctrl + F5

Mac:
    Cmd + Shift + R
```

---

### Optional CLI Command

```bash
aws cloudfront create-invalidation \
  --distribution-id DISTRIBUTION_ID \
  --paths "/ryu.jpg"
```

To invalidate everything:

```bash
aws cloudfront create-invalidation \
  --distribution-id DISTRIBUTION_ID \
  --paths "/*"
```

### Key Takeaway

CloudFront improves performance by caching content, but after updating files in S3, we may need invalidation to remove old cached files.

---

## Step-2 - Geographic Restriction

CloudFront can allow or block users based on country.

* Select your CloudFront distribution.

* Go to:

```text
Security → Geographic restrictions
```

or in some console versions:

```text
Distribution settings → Restrictions → Geographic restrictions
```

* Click **Edit**.

Configure:

```text
Restriction type:
    Block list

Countries:
    United States
```

* Click **Save changes**.

* Wait until the distribution finishes deploying.

* Go to your website and test it.

Expected result:

```text
403 Forbidden
```

if your request is detected as coming from a blocked country.

Important:

If you are not in the blocked country, the website may not be blocked for you. For classroom demonstration, you can explain that CloudFront checks the viewer location before serving the content.

### Key Takeaway

Geographic restriction is applied at CloudFront level before the request reaches the S3 website endpoint.

---

## Part 6 - Clean Up

To avoid unnecessary AWS charges, clean up resources after the hands-on.

### 1. Disable and Delete CloudFront Distribution

* Go to **CloudFront**.

* Select your distribution.

* Click **Disable**.

* Wait until the distribution is fully disabled and deployed.

* Then click **Delete**.

Important:

You cannot delete an active CloudFront distribution. You must disable it first.

---

### 2. Delete Route 53 Records

Go to:

```text
Route 53 → Hosted zones → your domain
```

Delete the records you created:

```text
A record for root domain
A record for www
AAAA records if you created them
```

Do not delete:

```text
NS
SOA
```

---

### 3. Empty and Delete S3 Bucket

* Go to **S3**.

* Open your bucket.

* Empty the bucket.

* Delete the bucket.

---

### 4. Delete ACM Certificate

* Go to **ACM**.

* Make sure you are in:

```text
us-east-1
```

* Delete the certificate if it is no longer used.

---

## Final Summary

In this hands-on, we configured a static website using:

```text
S3 Static Website Hosting
CloudFront
ACM
Route 53
```

The request flow is:

```text
User → Route 53 → CloudFront → S3 Static Website Endpoint
```

CloudFront provides:

```text
HTTPS
Caching
Custom domain support
Geographic restrictions
Invalidation
```

S3 provides:

```text
Static website hosting
index.html
website endpoint
```

The most important point is:

```text
When using S3 Static Website Hosting, use the S3 website endpoint as a custom origin in CloudFront.
```
