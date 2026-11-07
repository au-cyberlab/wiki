---
weight: 1
---
The email system is implemented by [Mailcow][1] — an open-source, Docker-based
mail server suite. It bundles Postfix for sending emails, Dovecot for receiving
emails, and SOGo for webmail.

## 1. Setup

To deploy the email system, add `email` to a lab configuration that is being deployed or your `deployment_config.yml` file.

## 2. Usage

### 2.1 Webmail access

On your development machine, you can forward the Mailcow web interface
to your machine. (Make sure to replace `MAILCOW_IP` with the IP address of the
Mailcow server, and `USER@HOST` with your local jumpbox details.)

```
ssh -i cyberlab-jumpbox-ssh-key.pem -L 8080:MAILCOW_IP:443 -Nf USER@HOST
```

You should now be able to access the Mailcow webmail at
`https://localhost:8080`!

Alternatively, you can VPN into the CyberLab and access the webmail at `https://<MAILCOW_IP>`

Your browser may complain about TLS certificates (they are self-signed). Ignore
this warning and click the button to proceed. (On Firefox this is
"Advanced..." > "Accept the Risk and Continue")

### 2.2 Default accounts

A number of email accounts are created by default. The credentials for these
should be available in `~/emails.txt` on the local jumpbox.

There is one account unlisted in this file, whose credentials are available in
`~/mailcow_installation.txt` (on the email server). This file also contains an
API key for programmatic interaction with Mailcow.

The default admin account has the following credentials:

Username: `admin`

Password: `moohoo`

## 3. Email clients

### 3.1 aerc

> [!NOTE]
> We have support for automated aerc (TUI email client) deployments.
> Documentation will be added in the future.
> It is not currently recommended to use this email client as an issue with certificates has not been resolved yet.

## 4. Contributors

Kent Rattley ([@kent-200][2]) and Minh Nguyen ([@minhkhanhnguyen-uoa][3]) added
the email system to the CyberLab.


[1]: https://github.com/mailcow/mailcow-dockerized
[2]: https://github.com/kent-200
[3]: https://github.com/minhkhanhnguyen-uoa
