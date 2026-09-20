# Static Website Hosting on AWS (S3 + CloudFront + CI/CD)

A production-style static website deployment for **anthonymuokem.com**, using Amazon S3 for storage, CloudFront for global content delivery over HTTPS, ACM for SSL, Hostinger for DNS and a GitHub Actions CI/CD pipeline for automated deployments.

## Architecture

```
GitHub Repo (push to main)
     │
     ▼
GitHub Actions CI/CD Pipeline
  - Syncs files to S3
  - Creates CloudFront invalidation
     │
     ▼
S3 Bucket: www.anthonymuokem.com
  - index.html / style.css / pool.jpg
  - Public access blocked
     │
     ▼
CloudFront Distribution (MyStaticWebsite)
  - Origin: S3 bucket
  - Alternate domain name (CNAME): www.anthonymuokem.com
  - SSL: ACM certificate (us-east-1)
  - Default root object: index.html
     │
     ▼
Hostinger DNS
  - DNS records pointed to CloudFront distribution
     │
     ▼
User Browser → https://www.anthonymuokem.com
```

## Components

### 1. Storage (Amazon S3)
- **Bucket name:** `www.anthonymuokem.com` — matches the domain name for clean CloudFront/DNS integration
- All public access blocked (S3 is not exposed directly; only CloudFront can serve content)
- Static website hosting enabled on the bucket
- Hosts `index.html`, `style.css`, and image assets (e.g. `pool.jpg`)

### 2. Content Delivery & SSL (CloudFront + ACM)
- **CloudFront distribution:** `MyStaticWebsite`
  - Origin set to the S3 bucket
  - Alternate domain name (CNAME): `www.anthonymuokem.com`
  - Default root object: `index.html`
- **ACM (AWS Certificate Manager):**
  - SSL/TLS certificate requested in **us-east-1** (required region for CloudFront certificates)
  - Certificate validated and propagated then attached to the CloudFront distribution
  - Enables HTTPS access on the custom domain

### 3. DNS (Hostinger)
- Domain `anthonymuokem.com` purchased and managed on Hostinger
- DNS records pointed from Hostinger to the CloudFront distribution domain name
- Once propagated, `www.anthonymuokem.com` resolves directly to the CloudFront distribution

### 4. CI/CD Pipeline (GitHub Actions)
- **IAM User:** `GitHubPerm`
  - Custom policy `Github-S3-Cloudfront-Policy` granting:
    - S3 permissions (upload/sync objects to the bucket)
    - CloudFront permission to create invalidations
  - Access key generated and tagged `GitHub-S3-CloudF-AccessKey`
- **GitHub repository:**
  - Stores the website source (`index.html`, `style.css`, images)
  - Repository secrets configured with: AWS Access Key ID, AWS Secret Access Key, S3 bucket name, CloudFront distribution name and AWS region (Europe)
  - Cloned locally for development via the command line
- **GitHub Actions workflow:**
  - Triggers on push to the repo
  - Deploys updated files to the S3 bucket
  - Creates a CloudFront invalidation so changes go live immediately (bypassing the CDN cache)

## How It Works

1. Website files are edited locally and pushed to the GitHub repository.
2. GitHub Actions automatically syncs the changed files to the S3 bucket.
3. The workflow triggers a CloudFront invalidation to clear cached content.
4. CloudFront serves the updated site over HTTPS at `www.anthonymuokem.com`, using the ACM certificate for SSL and Hostinger DNS to resolve the domain.
5. Verified end-to-end by editing `index.html`, pushing the change, and confirming the update appeared on the live site after the pipeline ran.

## Setup Summary

1. I purchased a domain on Hostinger (`anthonymuokem.com`).
2. Then i created an S3 bucket named to match the domain (`www.anthonymuokem.com`), block public access and enable static website hosting.
3. I uploaded website files (`index.html`, `style.css`, images) to the bucket.
4. I requested an SSL certificate in ACM (**us-east-1**) for the domain and validate it.
5. I created a CloudFront distribution (`MyStaticWebsite`) with the S3 bucket as origin, the domain as an alternate domain name, the ACM certificate attached and `index.html` as the default root object.
6. I added DNS records on Hostinger pointing to the CloudFront distribution domain name.
7. I confirmed the site loads at `https://www.anthonymuokem.com`.
8. I created an IAM user (`GitHubPerm`) with a scoped policy (`Github-S3-Cloudfront-Policy`) for S3 upload and CloudFront invalidation and generated an access key.
9. I created a GitHub repository added AWS credentials and config as repository secrets and then cloned it locally.
10. I set up a GitHub Actions workflow to sync files to S3 and invalidate the CloudFront cache on every push.
11. Then tested the pipeline by editing a file, pushing to GitHub and confirming the live site updates.

## Tech Stack

- **Hosting/Storage:** Amazon S3
- **CDN/SSL:** Amazon CloudFront, AWS Certificate Manager (ACM)
- **DNS:** Hostinger
- **CI/CD:** GitHub, GitHub Actions
- **IAM:** Scoped IAM user and policy for automated deployments (least-privilege access)

## Author

Built by Anthony Muokem as a hands-on AWS static hosting and CI/CD project.
