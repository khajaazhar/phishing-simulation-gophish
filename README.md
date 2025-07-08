# Phishing-Simulation-GoPhish

This project demonstrates a complete setup and deployment of a phishing simulation campaign using [GoPhish](https://getgophish.com/), an open-source phishing framework. The campaign was hosted on an AWS EC2 instance, with Mailgun configured for SMTP email delivery, and included multiple hardening techniques to evade detection by modern email security solutions.

---

## Tools and Technologies Used

- GoPhish (Phishing framework)
- AWS EC2 (Ubuntu Server)
- GoLang (For building GoPhish from source)
- Mailgun (SMTP email service)
- Certbot (TLS certificate generation)
- DNS configuration (TXT, MX, CNAME records)
- Custom domain (linked to EC2 instance)
- Temporary email tools (for safe testing)

---

## Setup Process

### 1. Launch and Connect to EC2

- Create an Ubuntu EC2 instance on AWS.
- Ensure `.pem` file has correct permissions.
- Connect via SSH:
  
  ssh -i gophish.pem ubuntu@<your-ec2-public-ip>
2. System Preparation
Update the system:


sudo apt update && sudo apt upgrade -y
Install GoLang:


sudo apt install golang-go -y
3. Install GoPhish
Clone the GoPhish repository and build:


git clone https://github.com/gophish/gophish.git
cd gophish
go build

4. Configure Mailgun
Register a domain.

Add DNS records (TXT, MX, CNAME) provided by Mailgun.

Create an SMTP user and retrieve the SMTP host, username, and password.

Update config.json:

Change 127.0.0.1 to 0.0.0.0 to allow external access.

Add SMTP credentials in the sending profile via the GoPhish dashboard.

5. Generate TLS Certificate
Install Certbot:


sudo apt install certbot
Generate certificate:


sudo certbot certonly --manual --preferred-challenges dns -d <yourdomain>
Add the generated certificate paths to config.json under the phish_server section.

Update the port to 443 and set "use_tls": true.

6. Create a Systemd Service
Create and edit the service file:


sudo nano /etc/systemd/system/gophish.service
Paste the following:


[Unit]
Description=GoPhish Service
[Service]
Type=simple
WorkingDirectory=/home/ubuntu/gophish/
ExecStart=/home/ubuntu/gophish/gophish
[Install]
WantedBy=multi-user.target

Enable and start the service:


sudo systemctl enable gophish.service
sudo systemctl start gophish.service
Launching the Phishing Campaign
Access the GoPhish admin panel at:

https://<your-ec2-public-ip>:3333
Create the following components:

Email Templates with {{.URL}} for phishing links.

Landing Pages with credential capture + redirect (e.g., to google.com).

User Groups containing target email addresses.

Sending Profiles using Mailgun SMTP credentials.

Launch the campaign and monitor:

Email deliveries

Link clicks

Credential submissions

Hardening the Phishing Server
To reduce the likelihood of detection by security tools, the following steps were taken:

Header Obfuscation
Replace GoPhish-specific email headers:


sudo find . -type f -exec sed -i.bak 's/X-Gophish-Contact/X-Contact/g' {} +
sudo find . -type f -exec sed -i.bak 's/X-Gophish-Signature/X-Signature/g' {} +
Identifier Renaming
In models/campaign.go, change:


const RecipientParameter = "id"
(originally "rid")

In config/config.go, change:

Server name "gophish" → "IGNORE"

Rebuild and Restart
Recompile the GoPhish binary:


go build
Restart the service:


sudo systemctl restart gophish.service
Add Custom Headers (Optional)
Add realistic headers in the GoPhish sending profile to blend in with real emails.


