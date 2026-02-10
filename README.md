# Felicia Nieland — Mobile Notary (website)

Static website for Felicia’s mobile notary business. **Clay County, Missouri.** Deploys to AWS (S3 + CloudFront) via GitHub Actions.

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
git remote add origin https://github.com/YOUR_USERNAME/felicia-notary-website.git
git branch -M main
git push -u origin main
```

**Option: GitHub CLI** — If you use `gh` and are logged in:

```bash
gh repo create felicia-notary-website --public --source=. --remote=origin --push
```

**4. Configure Actions (after first push)**

In the repo: **Settings → Secrets and variables → Actions.**

- **Secrets:** `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` (from your IAM user).
- **Variables:** `S3_BUCKET` (e.g. `notary-website-prod`), `CLOUDFRONT_DISTRIBUTION_ID` (optional until CloudFront exists), `AWS_REGION` (optional, default `us-east-1`).

After each push to `main`, the workflow will sync the site to S3 and invalidate CloudFront.

## AWS setup (one-time)

You need: **S3 bucket**, **CloudFront distribution**, **ACM certificate** (when you have a domain), and **IAM user** for GitHub Actions.

### 1. S3 bucket

- Create a bucket (e.g. `notary-website-prod`).
- Turn on **Block public access** (all four options). The site will be served only via CloudFront.
- Do **not** enable “Static website hosting” if you use CloudFront with OAC (recommended).

### 2. CloudFront distribution

- Origin: your S3 bucket.
- Use **Origin Access Control (OAC)** so CloudFront can read from the bucket (no public bucket policy).
- Update the bucket policy to allow the CloudFront service principal (AWS will show the policy when you create OAC).
- Default root object: `index.html`.
- Error pages: 404 → `/index.html` (or `/404.html` if you add one) so client-side-style routes work if you add them later.
- After the domain is chosen: add **Alternate domain names (CNAMEs)** and attach an **ACM certificate** (request the cert in **us-east-1**). Redirect HTTP → HTTPS.

### 3. IAM user for GitHub Actions

Create an IAM user (no console login). Attach a policy that allows:

- `s3:PutObject`, `s3:DeleteObject`, `s3:ListBucket` on the website bucket (and prefix if you use one).
- `cloudfront:CreateInvalidation` on the CloudFront distribution.

Use this user’s access key and secret in GitHub repo secrets as above.

### 4. Domain (TBD)

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
