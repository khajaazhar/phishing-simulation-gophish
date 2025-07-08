# Phishing-Simulation-GoPhish
This project demonstrates a complete setup and deployment of a phishing simulation campaign using [GoPhish](https://getgophish.com/), an open-source phishing framework. The campaign was hosted on an AWS EC2 instance, using Mailgun for SMTP email delivery, and included custom hardening techniques to evade detection by security tools.

---

## Tools & Technologies Used

- **GoPhish** (Phishing framework)
- **AWS EC2** (Ubuntu Server)
- **GoLang** (To build GoPhish)
- **Mailgun** (SMTP Email Service)
- **Certbot** (TLS Certificate for HTTPS)
- **DNS Configuration** (TXT, MX, CNAME for Mailgun)
- **Custom Domain** (Registered & linked to EC2)
- **Temporary Mail Tools** (for testing)

---

## Setup Process

###  EC2 Setup & GoPhish Installation

1. **Launch EC2 Instance** with Ubuntu.
2. Connect via **SSH** (Ensure `.pem` file permissions are secure).
3. Update system:
   
   sudo apt update && sudo apt upgrade -y
Install GoLang:

sudo apt install golang-go -y

Clone & build GoPhish:

git clone https://github.com/gophish/gophish.git
cd gophish
go build

📬 Mailgun SMTP Setup
Register a domain and configure DNS (TXT, MX, CNAME) records.

Create SMTP credentials and obtain server info.

Update config.json in GoPhish with Mailgun SMTP and set "0.0.0.0" for accessibility.

🔐 TLS Certificate
Install Certbot and generate a certificate:

sudo apt install certbot
sudo certbot certonly --manual --preferred-challenges dns -d <yourdomain>
Add certificate paths in config.json and update port to 443 with TLS enabled.

🔄 Automation via Systemd
Create a systemd service for GoPhish:

sudo nano /etc/systemd/system/gophish.service
Add the following:

[Unit]
Description=GoPhish Service
[Service]
Type=simple
WorkingDirectory=/home/ubuntu/gophish/
ExecStart=/home/ubuntu/gophish/gophish
[Install]
WantedBy=multi-user.target
Enable & start service:

sudo systemctl enable gophish.service
sudo systemctl start gophish.service

Phishing Campaign Steps
Access GoPhish dashboard at https://<public-ip>:3333

Create:

Email Templates (with embedded phishing links)

Landing Pages (with credential capture + redirect)

User Groups (targets with emails)

Sending Profiles (SMTP settings)

Launch campaign and monitor dashboard for:

Email sent

Link clicked

Data submitted

🛡️ Hardening Techniques
To reduce detection by mail filters and security tools:

Replaced GoPhish headers:


sed -i.bak 's/X-Gophish-Contact/X-Contact/g' . -type f
sed -i.bak 's/X-Gophish-Signature/X-Signature/g' . -type f
Renamed identifiers:

const RecipientParameter = "id" // instead of "rid"
Updated server name from "gophish" to "IGNORE" in config files.

Recompiled and restarted the service.
