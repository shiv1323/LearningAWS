# ☁️ Cloudflare, ACM, CloudFront & GitHub Actions – Deployment Guide

---

## 🔐 Cloudflare: DNS & Security

- 🌐 **Domain**: `yourdomain.com`
- 📍 **Nameservers**: Updated to Cloudflare’s
- 🔒 **SSL/TLS Mode**: Full (strict)
- 🛡️ **Security Level**: Medium / High
- 🚀 **Performance Settings**:
  - Auto Minify (HTML, CSS, JS): ✅
  - Brotli Compression: ✅
  - Rocket Loader: ⚠️ Optional

**DNS Settings**
```
A     @        192.0.2.1
CNAME www      yourdomain.com
CNAME cdn      your-cf-id.cloudfront.net
```

---

## 📄 ACM: AWS Certificate Manager

- 🔐 **Type**: Public Certificate
- 🌍 **Domains**: `yourdomain.com`, `www.yourdomain.com`
- ✅ **Validation**: DNS via Route53 or Cloudflare (manual)
- 🔄 **Renewal**: Automatic
- 📌 **Attached To**: CloudFront Distribution

**Steps**:
1. Go to ACM in AWS Console
2. Request Public Certificate
3. Add DNS Records in Cloudflare manually for validation

---

## 🌐 CloudFront: CDN for Fast Content Delivery

- 🚚 **Origin**: S3 Static Bucket / EC2 / API Gateway
- 🔒 **SSL**: Custom SSL from ACM attached
- 📦 **Cache Policy**: 
  - TTL: 86400 (1 day)
  - Invalidate after deployments
- 📛 **CNAMEs**: `cdn.yourdomain.com`, `www.yourdomain.com`

**Invalidation Example (After Deploy)**:
```bash
aws cloudfront create-invalidation --distribution-id YOUR_ID --paths "/*"
```

---

## 🤖 GitHub Actions: CI/CD Pipeline

**Common Use**: Build → Deploy to S3/EC2 → Invalidate CloudFront

**Example Workflow: `.github/workflows/deploy.yml`**
```yaml
name: Deploy to S3 + Invalidate CloudFront

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      - name: Install Node
        uses: actions/setup-node@v3
        with:
          node-version: '18'

      - run: npm install
      - run: npm run build

      - name: Deploy to S3
        uses: jakejarvis/s3-sync-action@master
        with:
          args: --acl public-read --delete
        env:
          AWS_S3_BUCKET: ${{ secrets.AWS_S3_BUCKET }}
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          SOURCE_DIR: "build"

      - name: Invalidate CloudFront
        run: |
          aws cloudfront create-invalidation --distribution-id ${{ secrets.CF_DIST_ID }} --paths "/*"
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

---

## ✅ Checklist Summary

- [x] Cloudflare DNS set up and proxied
- [x] SSL Certificate issued via ACM
- [x] CloudFront attached to ACM and S3
- [x] GitHub Actions setup for CI/CD
- [x] Cache invalidation on deploy