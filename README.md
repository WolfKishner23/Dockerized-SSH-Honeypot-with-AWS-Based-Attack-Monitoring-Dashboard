# 🔐 Cloud Honeypot Security Lab (AWS + Docker + Cowrie)

## 📌 Project Overview

This project demonstrates a cloud-based honeypot system deployed using Docker on AWS EC2. 

The system captures real-world SSH attack attempts using Cowrie honeypot, stores logs locally, uploads them to AWS S3, and visualizes attacker activity through a Flask-based dashboard.

This project simulates a basic SOC (Security Operations Center) monitoring pipeline.

---

## 🧱 Architecture

Attacker → Cowrie Honeypot (Docker) → Log Files → AWS S3 → Flask Dashboard

---

## 🚀 Technologies Used

```

- AWS EC2 (Ubuntu 22.04)
- AWS IAM (Role-based access)
- AWS S3 (Log storage)
- Docker & Docker Compose
- Cowrie SSH Honeypot
- Python (Flask)
- Linux (Ubuntu CLI)

```

---

## ⚙️ Setup Instructions

---

## 🖥️ 1. Launch EC2 Instance


```bash
- OS: Ubuntu 22.04
- Instance Type: t2.micro
- Storage: 10GB+

```
here i used 20GB 

---

# 🔐 Security Group

```bash
Port 22 → SSH (My IP)
Port 2222 → Honeypot SSH (0.0.0.0/0)
Port 5000 → Dashboard (0.0.0.0/0)
```


## 🔐 2. IAM Role Setup

```bash
- Create IAM Role: honeypot-role
- Attach Policy: AmazonS3FullAccess
- Attach role to EC2 instance
```

---

## 🐳 3. Install Docker



```bash
sudo apt update
sudo apt install docker.io -y
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker ubuntu

```

---

## 🐳 4. Deploy Cowrie Honeypot

```bash
mkdir honeypot-project
cd honeypot-project
```

Create docker-compose.yml:

```bash
version: '3'

services:
  cowrie:
    image: cowrie/cowrie:latest
    container_name: cowrie
    ports:
      - "2222:2222"
    volumes:
      - ./cowrie-data:/cowrie/cowrie-git/var
    restart: always
```
---


## 📁 5. Fix Log Directory Issue (IMPORTANT)

```bash
rm -rf cowrie-data
mkdir -p cowrie-data/log/cowrie
chmod -R 777 cowrie-data
```
👉 This fixes:

```bash
FileNotFoundError: var/log/cowrie/cowrie.json
```


---

## ▶️ 6. Run Honeypot

```bash
docker-compose up -d
```
---

## 🧪 7. Simulate Attack

```bash
ssh root@<EC2-IP> -p 2222
```

---

## 📁 8. Verify Logs

```bash
ls cowrie-data/log/cowrie/
```
---
## ☁️ 9. Create S3 Bucket

```bash
Bucket Name: honeypot-logs-yourname
```


---

## 📤 10. Upload Logs to S3

```bash
aws s3 sync ./cowrie-data/log/ s3://your-bucket-name/
```


---

## 🔁 11. Automate Upload (CRON)

```bash
crontab -e
```

Add:
```bash
*/5 * * * * aws s3 sync /home/ubuntu/honeypot-project/cowrie-data/log/ s3://your-bucket-name/
```

---

## 📊 12. Dashboard Setup

```bash
cd honeypot-project
mkdir dashboard
cd dashboard
pip3 install flask
```
Create app.py (Flask app)

👉 (paste code here)

```bash
from flask import Flask, render_template
import json

app = Flask(__name__)

LOG_FILE = "../cowrie-data/log/cowrie/cowrie.json"

def parse_logs():
    data = []

    try:
        with open(LOG_FILE) as f:
            for line in f:
                log = json.loads(line)
                
                if log.get("eventid") == "cowrie.login.failed":
                    data.append({
                        "ip": log.get("src_ip"),
                        "username": log.get("username"),
                        "password": log.get("password")
                    })
    except:
        pass

    return data

@app.route("/")
def index():
    logs = parse_logs()
    return render_template("index.html", logs=logs)

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)

```
---

Run: 

```bash
python3 app.py
```
Access:

```bash
http://<EC2-IP>:5000
```


---



## ⚠️ Challenges Faced & Solutions
---

❌ Issue 1: Logs not appearing

```bash
Error:
No such file or directory: cowrie-data/log/
```
✅ Solution:

```bash
mkdir -p cowrie-data/log/cowrie
chmod -R 777 cowrie-data
```
---

❌ Issue 2: Cowrie JSON logging failed

```bash
FileNotFoundError: var/log/cowrie/cowrie.json
```
✅ Solution:
```bash
Created required directory manually
```
---

❌ Issue 3: Cannot access container shell

```bash
bash/sh not found
```
✅ Solution:
```bash
Used docker logs instead for debugging
```
---

❌ Issue 4: Logs not syncing to S3

```bash
aws configure confusion
```

✅ Solution:
```bash
Used IAM role instead of access keys (best practice)
```
---

# 🔐 Security Concepts Applied

```bash
- Least Privilege (IAM role)
- Log Monitoring
- Attack Detection
- Secure Cloud Access
- Containerized Security Deployment
```
---
# 🧠 Key Learnings

```bash
- Docker container debugging
- Volume mapping issues
- Cloud security (IAM + S3)
- Log analysis
- Building monitoring dashboards
```
---

# 🚀 Future Improvements

```bash
- Geo-location tracking of attacker IPs
- Real-time alerts (Telegram/Email)
- ELK Stack integration
- Advanced visualization (charts)
- Threat intelligence integration
```
# 📌 Author
Ravindu Rangana
