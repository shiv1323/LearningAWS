# 📘 AWS Infrastructure Documentation

---

## 🔹 S3: Static File Storage

- ✅ **Bucket Name**: `your-bucket-name`
- 📦 **Used For**: Hosting frontend / static assets / backups
- 🔒 **Public Access Blocked**: ✅ Recommended
- 🗃️ **Versioning**: On / Off
- 🔐 **Encryption**: SSE-S3 / SSE-KMS
- ♻️ **Lifecycle Rules**: Auto-archive to Glacier after 30 days

---

## 🔹 EC2: Application Server

**Instance Info**  
- 💻 Type: `t3.micro` / `t2.small`  
- 🧩 AMI: Amazon Linux / Ubuntu  
- 🔐 Key Pair: `your-key.pem`  
- 🌐 Elastic IP: Yes / No  

**Security Groups**
```
Inbound:
 - SSH (22)
 - HTTP (80)
 - HTTPS (443)
Outbound:
 - All Traffic (Default)
```

**User Data Script**
```bash
#!/bin/bash
sudo yum update -y
cd /var/www/html
git clone https://github.com/yourrepo/project.git
```

---

## 🔹 RDS: SQL Database

- 🛢️ **Engine**: MySQL / PostgreSQL  
- 📦 **Storage**: 20 GB GP2  
- ☁️ **Multi-AZ**: No / Yes  
- 🌐 **Public Access**: ❌ Recommended  
- 🔁 **Backup Retention**: 7 days  
- 🔗 **Connection**:
  ```bash
  mysql -h your-db.rds.amazonaws.com -u admin -p
  ```

---

## 🔹 Database Migration (RDS → RDS)

**Method Chosen:**
- ✅ `mysqldump` + import via CLI
- ⛔ Snapshot cannot import to an existing DB
- 🛠️ AWS DMS (optional for zero downtime)

**Steps (Manual Way):**
1. SSH into EC2 or local machine
2. Export old DB:
   ```bash
   mysqldump -h old-db.rds.amazonaws.com -u admin -p old_db > backup.sql
   ```
3. Import into new DB:
   ```bash
   mysql -h new-db.rds.amazonaws.com -u admin -p new_db < backup.sql
   ```

---

## 🔹 ACM: SSL Certificate

- 📄 **Type**: Public Certificate  
- 🌍 **Domains**: `yourdomain.com`, `www.yourdomain.com`  
- ✅ **Validation**: DNS-based  
- 🔄 **Auto-Renewal**: Enabled  
- 🔒 **Attached To**: CloudFront / Load Balancer

---

## 🔹 CloudFront: CDN

- 🌐 **Origin**: S3 / EC2  
- 🆔 **Distribution ID**: `E2XXXXXXX`  
- 🔐 **SSL**: ACM Certificate attached  
- ⚡ **Caching Policy**: TTL = 1 day  
- 🌍 **Custom Domain**: `cdn.yourdomain.com`  
- 🌎 **Geo Restriction**: Off

---

## 📈 Architecture Overview

```
[S3] → [CloudFront] → [User]
[EC2] → [App Backend] → [RDS]
                ↓
         [Certificate: ACM]
```

---

## ✅ Notes & Checklists

- [x] Static site in S3
- [x] EC2 instance running
- [x] RDS migrated
- [x] SSL added via ACM
- [x] CloudFront setup
- [ ] Route53 DNS (Optional)