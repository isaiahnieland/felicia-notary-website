# Deploy to notarybyfelicia.com (S3 + CloudFront + Route 53)

Your domain **notarybyfelicia.com** is on Route 53. Follow these steps once to wire the site to S3, put CloudFront in front (for HTTPS), and point the domain at CloudFront. After that, every push to `main` will deploy via GitHub Actions.

---

## One-time AWS setup

Do this in **us-east-1** (N. Virginia). CloudFront and ACM certificates for CloudFront must be in us-east-1.

### 1. S3 bucket

1. **S3** → **Create bucket**
2. **Bucket name:** e.g. `notarybyfelicia.com` (or keep `notary-website-prod` if you already use it)
3. **Region:** `us-east-1`
4. **Block Public Access:** Leave **all four** checked for now (CloudFront will use OAC to read from S3; no public bucket needed)
5. Create bucket

No need to enable “Static website hosting” when using CloudFront with OAC.

---

### 2. CloudFront distribution

1. **CloudFront** → **Create distribution**
2. **Origin domain:** Choose your S3 bucket (e.g. `notarybyfelicia.com.s3.us-east-1.amazonaws.com`) — use the **S3 bucket endpoint**, not the “website” endpoint
3. **Origin access:** **Origin access control (OAC)** → **Create control setting** (defaults are fine) → Create
4. **Bucket policy:** When CloudFront shows “Copy policy” / “Copy the policy below…”, open your S3 bucket → **Permissions** → **Bucket policy** → paste that policy and save (so CloudFront can read from the bucket)
5. **Default root object:** `index.html`
6. **Alternate domain names (CNAMEs):** `notarybyfelicia.com` (and `www.notarybyfelicia.com` if you want www)
7. **Custom SSL certificate:** **Request certificate** (opens ACM in a new tab — see step 3) — or choose an existing one after you create it
8. **Viewer protocol policy:** **Redirect HTTP to HTTPS**
9. Create distribution. Note the **Distribution domain name** (e.g. `d1234abcd.cloudfront.net`) and **Distribution ID** (e.g. `E1234ABCD5678`).

---

### 3. ACM certificate (for HTTPS)

1. **ACM** (Certificate Manager) → **Request certificate**
2. **Fully qualified domain names:**  
   - `notarybyfelicia.com`  
   - `www.notarybyfelicia.com` (optional)
3. **Validation:** **DNS validation**
4. **Key algorithm:** RSA
5. Request. Then for each domain, ACM will show a **CNAME name** and **CNAME value**.
6. In **Route 53** → **Hosted zones** → **notarybyfelicia.com** → **Create record** for each:
   - **Record name:** paste the CNAME name (often `_abc123.notarybyfelicia.com`)
   - **Record type:** CNAME
   - **Value:** paste the CNAME value
   - Create. Repeat for the second domain if you added www.
7. Wait until the certificate status is **Issued** (usually 5–30 minutes).
8. Back in **CloudFront** → your distribution → **Edit** → **Custom SSL certificate** → select this certificate → Save.

---

### 4. Route 53 — point domain to CloudFront

In **Route 53** → **Hosted zones** → **notarybyfelicia.com**:

1. **Create record**
   - **Record name:** leave empty (apex `notarybyfelicia.com`)
   - **Record type:** A
   - **Alias:** On
   - **Route traffic to:** **Alias to CloudFront distribution** → choose your distribution
   - Create
2. Create another record for **AAAA** (same alias target) so IPv6 works
3. If you use **www** and added it to CloudFront/ACM:
   - **Record name:** `www`
   - **Record type:** A (and AAAA) → Alias to same CloudFront distribution

---

### 5. GitHub Actions (deploy on push)

1. Repo → **Settings** → **Secrets and variables** → **Actions**
2. **Secrets:** `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` (from your IAM user that can write to S3 and invalidate CloudFront)
3. **Variables:**
   - `S3_BUCKET` = your bucket name (e.g. `notarybyfelicia.com` or `notary-website-prod`)
   - `CLOUDFRONT_DISTRIBUTION_ID` = your CloudFront distribution ID (e.g. `E1234ABCD5678`)
   - `AWS_REGION` = `us-east-1` (optional; workflow defaults to it)

Your existing workflow (`.github/workflows/deploy.yml`) will:

- Sync the repo to S3 on every push to `main`
- Invalidate CloudFront so changes show immediately

---

## Deploy right now (one-time or manual)

If you want to upload the site **immediately** without waiting for a push:

```bash
cd /Users/isaiahnieland/felicia-notary-website

# Replace BUCKET_NAME with your bucket (e.g. notarybyfelicia.com or notary-website-prod)
aws s3 sync . s3://BUCKET_NAME \
  --delete \
  --exclude '.git/*' \
  --exclude '.github/*' \
  --exclude '.gitignore' \
  --exclude '*.md' \
  --exclude 'README*' \
  --region us-east-1
```

Then invalidate CloudFront (replace `DISTRIBUTION_ID`):

```bash
aws cloudfront create-invalidation --distribution-id DISTRIBUTION_ID --paths "/*" --region us-east-1
```

You need AWS CLI configured (`aws configure`) with credentials that can write to the bucket and invalidate the distribution.

---

## Checklist

- [ ] S3 bucket created (public access blocked; bucket policy allows CloudFront OAC)
- [ ] CloudFront distribution created (OAC, default root `index.html`, HTTPS redirect)
- [ ] ACM certificate requested and validated via DNS in Route 53
- [ ] CloudFront updated to use the ACM certificate
- [ ] Route 53 A and AAAA records for notarybyfelicia.com (and www) pointing to CloudFront
- [ ] GitHub repo variables: `S3_BUCKET`, `CLOUDFRONT_DISTRIBUTION_ID`; secrets: AWS keys
- [ ] First deploy: run `aws s3 sync` (and optional invalidation) or push to `main`

After that, **notarybyfelicia.com** will serve the site over HTTPS, and every push to `main` will deploy automatically.
