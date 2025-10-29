# EC2 Deployment Setup Guide

This document explains how to set up GitHub Actions to automatically deploy your static website to your EC2 instance with nginx.

## Prerequisites

1. **EC2 Instance**: You mentioned you already have nginx installed ✅
2. **GitHub Repository**: Your code should be pushed to a GitHub repository
3. **SSH Access**: You need SSH access to your EC2 instance

## Required GitHub Secrets

You need to add the following secrets to your GitHub repository:

### 1. Navigate to Repository Settings

- Go to your GitHub repository
- Click on "Settings" tab
- Click on "Secrets and variables" → "Actions"

### 2. Add These Secrets

Click "New repository secret" and add each of the following:

#### `EC2_SSH_KEY`

- **Name**: `EC2_SSH_KEY`
- **Value**: Your private SSH key content (the entire content of your `.pem` or private key file)
- **Example**:

```
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEA...
[Your private key content]
...
-----END RSA PRIVATE KEY-----
```

#### `EC2_HOST`

- **Name**: `EC2_HOST`
- **Value**: Your EC2 instance public IP address or domain name
- **Example**: `54.123.456.789` or `your-domain.com`

#### `EC2_USER`

- **Name**: `EC2_USER`
- **Value**: The SSH username for your EC2 instance
- **Common values**:
  - `ubuntu` (for Ubuntu instances)
  - `ec2-user` (for Amazon Linux instances)
  - `admin` (for Debian instances)

## EC2 Instance Setup

Make sure your EC2 instance is properly configured:

### 1. Nginx Configuration

Since you mentioned nginx is already installed, ensure it's configured to serve from `/var/www/html`:

```bash
# Check nginx status
sudo systemctl status nginx

# Ensure nginx is enabled to start on boot
sudo systemctl enable nginx

# Check nginx configuration
sudo nginx -t
```

### 2. Directory Permissions

```bash
# Ensure the web directory exists with proper permissions
sudo mkdir -p /var/www/html
sudo chown -R www-data:www-data /var/www/html
sudo chmod -R 755 /var/www/html
```

### 3. Security Group Settings

Ensure your EC2 security group allows:

- **SSH (Port 22)**: For deployment access
- **HTTP (Port 80)**: For web traffic
- **HTTPS (Port 443)**: If you plan to use SSL

### 4. SSH Key Setup

Make sure:

- Your SSH key pair is correctly configured
- The private key is added to GitHub secrets
- The public key is in `~/.ssh/authorized_keys` on your EC2 instance

## Workflow Explanation

The GitHub Actions workflow (`.github/workflows/deploy-to-ec2.yml`) does the following:

1. **Triggers**: Runs on pushes to the `main` branch
2. **Checkout**: Downloads your repository code
3. **SSH Setup**: Configures SSH connection to your EC2 instance
4. **Deploy**:
   - Creates a temporary directory on EC2
   - Copies your files (`index.html`, `styles.css`, `README.md`) to EC2
   - Moves files to nginx web directory (`/var/www/html`)
   - Sets proper permissions
   - Reloads nginx
5. **Verify**: Checks if the website is accessible

## Testing the Deployment

1. **Push to main branch**: The workflow will trigger automatically
2. **Monitor the workflow**: Go to "Actions" tab in your GitHub repository
3. **Check your website**: Visit `http://YOUR_EC2_IP` to see your deployed site

## Troubleshooting

### Common Issues:

1. **Permission Denied (SSH)**

   - Check if the SSH key is correct
   - Ensure the EC2_USER is correct for your instance type

2. **Connection Refused**

   - Verify security group allows SSH (port 22)
   - Check if EC2_HOST IP/domain is correct

3. **Nginx Issues**

   - SSH into your instance and run: `sudo nginx -t`
   - Check nginx logs: `sudo tail -f /var/log/nginx/error.log`

4. **File Permissions**
   - Ensure www-data has ownership of `/var/www/html`
   - Check file permissions are 755 for directories and 644 for files

### Manual Testing:

You can test the deployment process manually:

```bash
# SSH into your instance
ssh -i your-key.pem ec2-user@your-ec2-ip

# Check nginx status
sudo systemctl status nginx

# Check if files are in the right place
ls -la /var/www/html/

# Test nginx configuration
sudo nginx -t
```

## Security Notes

- Never commit your private SSH key to the repository
- Use GitHub secrets for all sensitive information
- Consider using IAM roles and AWS Systems Manager for more secure deployments
- Regularly rotate your SSH keys

## Next Steps

After successful deployment, you might want to:

1. Set up a custom domain name
2. Configure SSL/HTTPS with Let's Encrypt
3. Set up monitoring and logging
4. Implement blue-green deployments for zero downtime

---

**Ready to deploy?**

1. Add the required secrets to your GitHub repository
2. Push your code to the main branch
3. Watch the magic happen in the Actions tab! 🚀
