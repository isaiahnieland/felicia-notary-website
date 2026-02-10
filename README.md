# Felicia Nieland — Mobile Notary (website)

Static website for Felicia’s mobile notary business. **Clay County, Missouri.** Deploys to **AWS S3** via GitHub Actions. Hosting is S3-only for now; CloudFront can be added later for a custom domain and HTTPS.

## Run locally

Open the folder in a browser or use a simple server:

```bash
# From the repo root (Python 3)
python3 -m http.server 8000
# Then open http://localhost:8000
```

Or open `index.html` directly in a browser (links will work for same-directory files).

## Set up the GitHub repo (first time)

Run these in your terminal from the **website folder** (e.g. `felicia-notary-website`).

**1. Initialize git and make the first commit**

```bash
cd /Users/isaiahnieland/felicia-notary-website   # or your path to this folder
git init
git add .
git commit -m "Initial commit: static notary site for Clay County MO"
```

**2. Create the repo on GitHub**

- Go to [github.com/new](https://github.com/new).
- **Repository name:** `felicia-notary-website` (or `notarybyfelicia`).
- **Public.** Do **not** add a README, .gitignore, or license (you already have them).
- Click **Create repository**.

**3. Add the remote and push**

GitHub will show you commands; use these (replace `YOUR_USERNAME` with your GitHub username):

```bash
git remote add origin https://github.com/isaiahnieland/felicia-notary-website.git
git branch -M main
git push -u origin main
```

**Option: GitHub CLI** — If you use `gh` and are logged in:

```bash
gh repo create felicia-notary-website --public --source=. --remote=origin --push
```

**4. Link GitHub Actions to AWS** — See [Link GitHub Actions to AWS](#link-github-actions-to-aws) below.

After each push to `main`, the workflow will sync the site to S3 (and optionally invalidate CloudFront if you add it later).

## Link GitHub Actions to AWS

Do this after you have an S3 bucket (and optionally a CloudFront distribution). You’ll create an IAM user for GitHub, then add its keys and your bucket/distribution IDs to the repo.

### 1. Create an IAM user for GitHub Actions

1. In the AWS Console go to **IAM → Users → Create user**.
2. **User name:** e.g. `github-actions-felicia-notary`. No console login.
3. **Attach policies:** Create an inline policy (or a custom managed policy) with this JSON (replace `YOUR-CLOUDFRONT-DIST-ID` and `YOUR-ACCOUNT-ID` when you add CloudFront; if you don’t have CloudFront yet, remove the `cloudfront` block):

//I choose not to do the inline access. I gave the user Amazons3fullAccess and CloudFrontFullAccess for now

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:PutObject", "s3:DeleteObject", "s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::notary-website-prod",
        "arn:aws:s3:::notary-website-prod/*"
      ]
    }
  ]
}
```

4. Create the user.

### 2. Create access keys for the IAM user

1. Open the user → **Security credentials** tab.
2. **Access keys** → **Create access key**.
3. Use **Application running outside AWS** (or “Command Line Interface”).
4. Copy the **Access key ID** and **Secret access key** (you won’t see the secret again).

### 3. Add secrets and variables in GitHub

1. Repo → **Settings → Secrets and variables → Actions**.
2. **Secrets** tab → **New repository secret**:
   - Name: `AWS_ACCESS_KEY_ID`, Value: the access key ID.
   - Name: `AWS_SECRET_ACCESS_KEY`, Value: the secret access key.
3. **Variables** tab → **New repository variable**:
   - `S3_BUCKET` = `notary-website-prod` (your bucket name).
   - `CLOUDFRONT_DISTRIBUTION_ID` = your CloudFront distribution ID (optional until you have CloudFront).
   - `AWS_REGION` = e.g. `us-east-1` (optional; workflow defaults to `us-east-1`).

After this, each push to `main` will run the **Deploy to AWS** workflow and sync the site to S3 (and invalidate CloudFront if `CLOUDFRONT_DISTRIBUTION_ID` is set).

## AWS setup (one-time)

You need: **S3 bucket** and **IAM user** for GitHub Actions. CloudFront and a custom domain are optional and can be added later.

### 1. S3 bucket (hosting for now)

- Create a bucket (e.g. `notary-website-prod`).
- **To view the site via S3 only:** In the bucket go to **Properties → Static website hosting → Edit**. Enable it, set **Index document** to `index.html`, **Error document** to `index.html` (or leave blank). Save. Use the **Bucket website endpoint** URL (e.g. `http://notary-website-prod.s3-website-us-east-1.amazonaws.com`) to open the site. The bucket must allow public read for this to work: in **Permissions** add a bucket policy that allows `s3:GetObject` for the bucket (AWS has a template for “Public read” under Static website hosting).
- **If you add CloudFront later:** You can turn off public access and serve only via CloudFront with OAC; until then, static website hosting + public read is the simplest way to have a working URL.

### 2. CloudFront (optional — add later)

When you want a custom domain and HTTPS: create a CloudFront distribution with your S3 bucket as origin, use **Origin Access Control (OAC)**, set default root object to `index.html`, and add your domain + ACM certificate. Then set the repo variable `CLOUDFRONT_DISTRIBUTION_ID` so deploys invalidate the cache.

### 3. IAM user for GitHub Actions

Create an IAM user (no console login). Attach a policy that allows:

- `s3:PutObject`, `s3:DeleteObject`, `s3:ListBucket` on the website bucket (and prefix if you use one).
- `cloudfront:CreateInvalidation` on the CloudFront distribution.

Use this user’s access key and secret in GitHub repo secrets as above.

### 4. Domain (optional — TBD)

- **Domain name:** Placeholder until you decide (see `felicia-notary-business/MASTER-PLAN.md` for options).
- When decided: register the domain (Route 53, Cloudflare, or Namecheap). Felicia pays ~$12–15/year.
- Point the domain to the CloudFront distribution (A/ALIAS in Route 53, or CNAME at another registrar).
- Request an ACM certificate in **us-east-1** for the domain; validate via DNS (CNAME). Add the domain and cert to the CloudFront distribution and turn on HTTPS redirect.

## Calendly (free)

- **Booking link:** Edit `js/config.js` and set `CALENDLY_BOOKING_URL` to Felicia’s Calendly booking URL (e.g. `https://calendly.com/username/notary`).
- Felicia sets up a **free** Calendly account, one event type (“Notary appointment”), and availability: **weekends all day, weekdays after 5:00 PM**. She connects her Google Calendar so her full-time job blocks don’t show as available.
- The “Book now” button on the site uses this URL.

## Contact page

- Update `contact.html` with Felicia’s real phone number and email (replace the placeholder `tel:` and `mailto:` links).

## Handoff for Felicia

- **She does not need to edit the website.** You (or she) only updates: Calendly URL in `js/config.js`, and contact details in `contact.html`, then push to GitHub; the site deploys automatically.
- **To change availability:** She updates her Google Calendar and/or her Calendly availability. No code changes.
- **Pricing:** When Missouri rates are finalized, update the table in `pricing.html`.
