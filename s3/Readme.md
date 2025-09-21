# Amazon S3 — Step-by-Step Hands-On Tutorial 📦🌐

## 🎯 Goal
Create an S3 bucket, upload files, experiment with public vs private access, enable static website hosting, and apply common production best practices (CORS, lifecycle, versioning, encryption).

⏱ Estimated time: 20–45 minutes  
✅ Prerequisites: AWS account, Console access, (optional) AWS CLI configured (`aws configure`) and basic CLI familiarity.

---

## 1 — Create the bucket (Console + CLI)

**Console**
1. Open S3 in the AWS Console → *Create bucket*  
2. Bucket name: `my-unique-bucket-name-12345`  
3. Region: `us-west-2` (or your preferred)  
4. Leave **Block public access** ON for now  
5. Click **Create bucket**  

**CLI**
```bash
aws s3api create-bucket --bucket my-unique-bucket-name-12345 --region us-west-2 --create-bucket-configuration LocationConstraint=us-west-2
```

---

## 2 — Upload files (Console + CLI)

**Console**: Bucket → *Upload* → drag & drop `index.html`, `error.html`, images → *Upload*  

**CLI**
```bash
aws s3 cp index.html s3://my-unique-bucket-name-12345/index.html
aws s3 sync ./site s3://my-unique-bucket-name-12345/
```

---

## 3 — Public vs Private: Bucket Policy

Default = Block Public Access. To allow public, disable it and add a bucket policy.

**Example policy:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-unique-bucket-name-12345/*"
    }
  ]
}
```

---

## 4 — Enable Static Website Hosting

**Console**: Bucket → *Properties → Static website hosting* → Enable → Index = `index.html`, Error = `error.html`  

**CLI**
```bash
aws s3 website s3://my-unique-bucket-name-12345 --index-document index.html --error-document error.html
```

---

## 5 — Serve via CloudFront (Recommended)

1. Create CloudFront distribution → Origin = S3 bucket  
2. Restrict bucket access to CloudFront  
3. Add ACM certificate for HTTPS  
4. Point domain (Route 53) to CloudFront  

✅ CloudFront adds HTTPS, caching, security.

---

## 6 — CORS Configuration

**Example**
```xml
<CORSConfiguration>
  <CORSRule>
    <AllowedOrigin>https://example.com</AllowedOrigin>
    <AllowedMethod>GET</AllowedMethod>
    <AllowedMethod>HEAD</AllowedMethod>
    <AllowedHeader>*</AllowedHeader>
  </CORSRule>
</CORSConfiguration>
```

---

## 7 — Versioning & Lifecycle

**Enable versioning**
```bash
aws s3api put-bucket-versioning --bucket my-unique-bucket-name-12345 --versioning-configuration Status=Enabled
```

**Lifecycle rule example**
- Transition to STANDARD_IA after 30 days  
- Expire after 365 days  

---

## 8 — Server-side Encryption (SSE)

Enable default encryption with AES256:
```bash
aws s3api put-bucket-encryption --bucket my-unique-bucket-name-12345 --server-side-encryption-configuration '{"Rules":[{"ApplyServerSideEncryptionByDefault":{"SSEAlgorithm":"AES256"}}]}'
```

---

## 9 — Logging & Monitoring

```bash
aws s3api put-bucket-logging --bucket my-unique-bucket-name-12345 --bucket-logging-status '{"LoggingEnabled":{"TargetBucket":"my-log-bucket","TargetPrefix":"logs/"}}'
```

Enable CloudTrail for API logging.

---

## 10 — Test & Validate

```bash
curl -I http://my-unique-bucket-name-12345.s3-website-us-west-2.amazonaws.com
```

---

## 11 — Troubleshooting

- **403 AccessDenied** → Check Block Public Access & bucket policy  
- **404 Not Found** → Ensure `index.html` exists  
- **CORS errors** → Fix `AllowedOrigin` in CORS  
- **HTTPS required** → Use CloudFront + ACM  

---

## 12 — Cleanup

```bash
aws s3 rm s3://my-unique-bucket-name-12345 --recursive
aws s3api delete-bucket --bucket my-unique-bucket-name-12345 --region us-west-2
```


## ✅ Best Practices
- Use **CloudFront + OAC** instead of public buckets  
- Apply **least privilege IAM policies**  
- Enable **encryption, versioning, lifecycle**  
- Keep **logs & CloudTrail** for audits  
- Use **signed URLs** for temporary access  
