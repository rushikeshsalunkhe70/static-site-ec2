# Static Website Deployment on AWS EC2 Using GitHub Actions

A beginner-friendly DevOps project that hosts a static website on an AWS
EC2 Ubuntu server and automatically deploys website updates through
GitHub Actions.

## Project Overview

Website files are stored in GitHub. When changes are pushed to the
`main` branch, GitHub Actions runs a deployment workflow using a
self-hosted runner on a Windows laptop. The workflow connects to EC2
over SSH, transfers website files using SCP, validates the Nginx
configuration, and reloads Nginx.

## Architecture

``` text
Developer edits HTML / CSS / JavaScript
                  |
                  v
             Git commit
                  |
                  v
             GitHub push
                  |
                  v
          GitHub Actions workflow
                  |
                  v
      Self-hosted runner (Windows)
                  |
             SSH + SCP
                  |
                  v
       AWS EC2 (Ubuntu Server)
                  |
                  v
                Nginx
                  |
                  v
             Live website
```

## Technologies Used

-   HTML, CSS, JavaScript -- static website frontend
-   Git -- source control and version history
-   GitHub -- remote source-code repository
-   GitHub Actions -- CI/CD workflow automation
-   Self-hosted runner -- executes the workflow on a Windows machine
-   AWS EC2 -- cloud virtual machine hosting the website
-   Ubuntu -- EC2 operating system
-   Nginx -- web server
-   SSH -- secure remote connection
-   SCP -- secure file transfer

## Repository

https://github.com/rushikeshsalunkhe70/static-site-ec2

## Project Structure

``` text
static-site-ec2/
├── .github/
│   └── workflows/
│       └── deploy.yml
├── site/
│   ├── index.html
│   ├── styles.css
│   └── script.js
├── .gitignore
└── README.md
```

## Deployment Workflow

The workflow is configured in `.github/workflows/deploy.yml`.

1.  A developer updates website files locally.
2.  Git stages and commits the changes.
3.  `git push origin main` sends the commit to GitHub.
4.  A push to `main` triggers the GitHub Actions workflow.
5.  The workflow checks out the repository and validates required
    website files.
6.  The self-hosted runner prepares SSH access using repository secrets.
7.  The workflow connects to EC2 and transfers the contents of `site/`
    to a temporary directory using SCP.
8.  Files are copied into Nginx's web root, `/var/www/html`.
9.  The workflow runs `sudo nginx -t` to validate the Nginx
    configuration.
10. Nginx is reloaded to serve the updated website.

## Prerequisites

-   AWS account and Ubuntu EC2 instance
-   Nginx installed and configured on EC2
-   GitHub repository containing this project
-   Git installed on the Windows development machine
-   OpenSSH client (`ssh` and `scp`) available in PowerShell
-   Registered and configured GitHub self-hosted runner
-   SSH access to EC2 using a private key

## AWS EC2 Configuration

The website is served from:

``` text
/var/www/html
```

Useful commands on EC2:

``` bash
sudo systemctl status nginx
sudo systemctl start nginx
sudo systemctl enable nginx
sudo nginx -t
sudo systemctl reload nginx
ls -l /var/www/html
curl localhost
```

## GitHub Actions Secrets

In the GitHub repository, open **Settings → Secrets and variables →
Actions** and configure:

  Secret          Purpose
  --------------- ------------------------------------------------
  `EC2_HOST`      Current public IPv4 address or DNS name of EC2
  `EC2_USER`      SSH username; for this Ubuntu server, `ubuntu`
  `EC2_SSH_KEY`   Private SSH key used by the workflow

Never put private keys or secret values in the workflow file or README.
Do not share the private key.

## Starting the Project

### 1. Start EC2

1.  Open AWS Console → EC2 → Instances.
2.  Select the project instance.
3.  If stopped, choose **Instance state → Start instance**.
4.  Wait until the instance is running and status checks pass.

### 2. Check the public IP

Find the current **Public IPv4 address** in the EC2 instance details.

If it changed, update the `EC2_HOST` GitHub repository secret. Also
verify that the security group allows SSH from your trusted current
public IP.

### 3. Connect to EC2 from Windows PowerShell

Use your own PEM file path and the current EC2 public IP:

``` powershell
ssh -i "C:\path\to\your-key.pem" ubuntu@YOUR_EC2_PUBLIC_IP
```

Keep the private key secure. Never upload it to GitHub.

### 4. Start the self-hosted runner

Open a separate PowerShell window on the Windows machine where the
runner is installed:

``` powershell
cd C:\actions-runner
.\run.cmd
```

Wait for:

``` text
Listening for Jobs
```

Keep this window open while deployments are needed.

### 5. Open the website

Visit:

``` text
http://YOUR_EC2_PUBLIC_IP
```

## Deploying Website Changes

Open PowerShell in your local project directory:

``` powershell
cd "C:\path\to\static-site-ec2"

git status
git add .
git commit -m "Update website"
git push origin main
```

After pushing:

1.  Open the repository's **Actions** tab.
2.  Open the latest workflow run.
3.  Confirm that it completes successfully.
4.  Refresh the live website. Use `Ctrl + Shift + R` if the browser
    shows cached content.

### CSS cache-busting

If the updated CSS file is deployed but the browser continues showing
old styles, change the stylesheet reference in `site/index.html`:

``` html
<link rel="stylesheet" href="styles.css?v=2">
```

Increment the version query (for example, `v=3`) when needed. Commit and
push the HTML change so the browser requests the new stylesheet URL.

## Security Notes

-   Restrict inbound SSH (port 22) to a trusted administrator IP using a
    `/32` source rule where practical.
-   Allow HTTP (port 80) publicly only if the website is intended to be
    public.
-   Store SSH private keys in GitHub Actions secrets, not in the
    repository.
-   Do not share or commit `.pem` files.
-   Keep GitHub repository secrets private.
-   Verify a new EC2 host key when the public IP changes.
-   Consider HTTPS and a domain name for a production deployment.

## Troubleshooting

  -----------------------------------------------------------------------
  Problem                             Checks
  ----------------------------------- -----------------------------------
  Website does not open               Confirm EC2 is running, Nginx is
                                      active, and inbound HTTP port 80 is
                                      allowed

  SSH connection fails                Check current EC2 IP, key-file
                                      path, username, and inbound SSH
                                      rule

  GitHub Actions job stays queued     Confirm the runner is running and
                                      displays `Listening for Jobs`

  Workflow fails                      Open the failed workflow step and
                                      inspect its logs

  HTML changes appear but CSS does    Check the deployed CSS file,
  not                                 stylesheet link, and browser cache

  Website shows old content           Hard-refresh with
                                      `Ctrl + Shift + R`; try an
                                      Incognito window

  EC2 public IP changed               Update `EC2_HOST` and verify the
                                      new SSH host key
  -----------------------------------------------------------------------

## Stopping the Project

1.  In the self-hosted runner PowerShell window, press `Ctrl + C`.
2.  If connected to EC2 over SSH, run `exit`.
3.  In AWS Console, select the EC2 instance and choose **Instance state
    → Stop instance**.

Stopping EC2 stops its compute usage, but EBS storage and other
associated resources may continue to incur charges.

## Outcome

This project demonstrates an automated static website deployment
pipeline using GitHub Actions, a Windows self-hosted runner, SSH/SCP,
AWS EC2, Ubuntu, and Nginx. Pushing changes to the `main` branch
triggers deployment, reducing the need to copy website files manually.

## Author

**Rushikesh Salunkhe**

GitHub: https://github.com/rushikeshsalunkhe70
