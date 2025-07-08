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

### Step 1: Launch and Connect to EC2

- Create an Ubuntu EC2 instance on AWS.
- Ensure `.pem` file has correct permissions.
- Connect via SSH:
  ```bash
  ssh -i gophish.pem ubuntu@<your-ec2-public-ip>
  ```

### Step 2: System Preparation

- Update the system:
  ```bash
  sudo apt update && sudo apt upgrade -y
  ```

- Install GoLang:
  ```bash
  sudo apt install golang-go -y
  ```

### Step 3: Install GoPhish

- Clone the GoPhish repository and build:
  ```bash
  git clone https://github.com/gophish/gophish.git
  cd gophish
  go build
  ```

### Step 4: Configure Mailgun

- Register a domain.
- Add DNS records (TXT, MX, CNAME) provided by Mailgun.
- Create an SMTP user and retrieve the SMTP host, username, and password.
- Update `config.json`:
  - Change `127.0.0.1` to `0.0.0.0` to allow external access.
  - Add SMTP credentials in the sending profile via the GoPhish dashboard.

### Step 5: Generate TLS Certificate

- Install Certbot:
  ```bash
  sudo apt install certbot
  ```

- Generate certificate:
  ```bash
  sudo certbot certonly --manual --preferred-challenges dns -d <yourdomain>
  ```

- Add the generated certificate paths to `config.json` under the `phish_server` section.
- Update the port to `443` and set `"use_tls": true`.

### Step 6: Create a Systemd Service

- Create and edit the service file:
  ```bash
  sudo nano /etc/systemd/system/gophish.service
  ```

  Paste the following:
  ```ini
  [Unit]
  Description=GoPhish Service

  [Service]
  Type=simple
  WorkingDirectory=/home/ubuntu/gophish/
  ExecStart=/home/ubuntu/gophish/gophish

  [Install]
  WantedBy=multi-user.target
  ```

- Enable and start the service:
  ```bash
  sudo systemctl enable gophish.service
  sudo systemctl start gophish.service
  ```

---

## Launching the Phishing Campaign

### Step 7: Access the GoPhish Admin Panel

- Visit:
  ```
  https://<your-ec2-public-ip>:3333
  ```

### Step 8: Create Required Components

- Email Templates with `{{.URL}}` for phishing links.
- Landing Pages with credential capture + redirect (e.g., to google.com).
- User Groups containing target email addresses.
- Sending Profiles using Mailgun SMTP credentials.

### Step 9: Launch the Campaign

- Select the template, landing page, and group.
- Monitor the dashboard for:
  - Email deliveries
  - Link clicks
  - Credential submissions

---

## Hardening the Phishing Server

### Step 10: Header Obfuscation

- Replace GoPhish-specific email headers:
  ```bash
  sudo find . -type f -exec sed -i.bak 's/X-Gophish-Contact/X-Contact/g' {} +
  sudo find . -type f -exec sed -i.bak 's/X-Gophish-Signature/X-Signature/g' {} +
  ```

### Step 11: Identifier Renaming

- In `models/campaign.go`, change:
  ```go
  const RecipientParameter = "id"
  ```

- In `config/config.go`, change all `"gophish"` references to `"IGNORE"`.

### Step 12: Rebuild and Restart

- Recompile the GoPhish binary:
  ```bash
  go build
  ```

- Restart the service:
  ```bash
  sudo systemctl restart gophish.service
  ```

### Step 13: Add Custom Headers (Optional)

- Add realistic headers in the GoPhish sending profile to blend in with real emails.

---

## Testing and Results

- Used [10minutemail.com](https://10minutemail.com) for testing.
- Verified email delivery through Mailgun.
- Phishing link led to landing page with credential capture.
- Data captured in dashboard and user redirected to Google.

---

## Disclaimer

This project was built for educational and authorized testing purposes only. Do not use phishing tools without explicit legal permission.

---

## Author

**Khaja Azhar Uddin**  
**Date:** July 2025
