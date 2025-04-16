# Setting Up a Remote Linux Server on AWS with SSH Access

Setting up a remote Linux server on Amazon Web Services (AWS), configuring secure SSH access with two key pairs, and optionally installing fail2ban for enhanced security. By the end, you'll have a running EC2 instance accessible via SSH and protected against brute-force attacks.

## 1. Set Up a Remote Linux Server on AWS

### 1.1 Create an AWS Account

If you don’t already have an AWS account, [register here](https://aws.amazon.com/). New users can use the AWS Free Tier for eligible services.

### 1.2 Launch an EC2 Instance

1. **Log in to the AWS Management Console** and go to the EC2 Dashboard.
2. Click **Launch Instances**.
3. **Select an Amazon Machine Image (AMI)**: Choose a Linux distribution like Ubuntu Server or Amazon Linux.
4. **Choose an Instance Type**: Pick `t2.micro` for free-tier eligibility.
5. **Configure Network Settings**:
   - Ensure the security group allows inbound SSH (port 22) from your IP or `0.0.0.0/0` (not recommended for production).
6. **Launch the Instance**:
   - Create an initial SSH key pair when prompted (e.g., `initial-key`).
   - Download the private key file (`.pem`) and store it securely for the first connection.

## 2. Create Two New SSH Key Pairs

### 2.1 Generate Keys Locally

On your local machine, create two RSA key pairs:

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/key1
ssh-keygen -t rsa -b 4096 -f ~/.ssh/key2
```

- Replace `key1` and `key2` with your preferred names.
- Optionally set a passphrase for added security.

This generates private keys (`key1`, `key2`) and public keys (`key1.pub`, `key2.pub`) in `~/.ssh/`.

### 2.2 Add Public Keys to the Server

1. **Connect to the server** using the initial key pair:
   ```bash
   ssh -i <path-to-initial-private-key> ubuntu@<server-ip>
   ```
   - Replace `<path-to-initial-private-key>` with the path to your `.pem` file.
   - Replace `<server-ip>` with your instance’s public IP address.
2. **Append the new public keys** to `~/.ssh/authorized_keys`:
   ```bash
   cat ~/.ssh/key1.pub >> ~/.ssh/authorized_keys
   cat ~/.ssh/key2.pub >> ~/.ssh/authorized_keys
   ```
   - If on Windows, manually copy the public key contents if `cat` isn’t available.

## 3. Test SSH Access with Both Keys

Verify connectivity with each new key:

1. **Connect with the first key**:
   ```bash
   ssh -i ~/.ssh/key1 ubuntu@<server-ip>
   ```
2. **Connect with the second key**:
   ```bash
   ssh -i ~/.ssh/key2 ubuntu@<server-ip>
   ```

If both succeed, your keys are properly set up.

## 4. Configure `~/.ssh/config` for Easier Access

Simplify SSH connections with aliases:

1. **Edit your local SSH config file**:
   ```bash
   nano ~/.ssh/config
   ```
2. **Add entries for both keys**:
   ```plaintext
   Host alias1
       HostName <server-ip>
       User ubuntu
       IdentityFile ~/.ssh/key1

   Host alias2
       HostName <server-ip>
       User ubuntu
       IdentityFile ~/.ssh/key2
   ```
   - Replace `<server-ip>` with your instance’s public IP.
3. **Test the aliases**:
   ```bash
   ssh alias1
   ssh alias2
   ```

Now you can connect using these aliases without specifying keys or IPs manually.

## 5. Stretch Goal: Install and Configure fail2ban

fail2ban monitors logs and bans IPs after repeated failed login attempts, bolstering security.

### 5.1 Install fail2ban

On your server:

```bash
sudo apt update
sudo apt install fail2ban
```

### 5.2 Configure fail2ban

1. **Copy the default configuration**:
   ```bash
   sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
   ```
2. **Edit `jail.local`** to enable the SSH jail:
   ```bash
   sudo nano /etc/fail2ban/jail.local
   ```
   - Locate the `[sshd]` section and update it:
     ```plaintext
     [sshd]
     enabled = true
     port = ssh
     maxretry = 5
     bantime = 600
     ```
   - `maxretry`: Failed attempts before banning.
   - `bantime`: Ban duration in seconds (600s = 10 minutes).
3. **Restart fail2ban**:
   ```bash
   sudo systemctl restart fail2ban
   ```

### 5.3 Check Status

Confirm fail2ban is running and protecting SSH:

```bash
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

## 6. Troubleshooting

- **Permission Denied (publickey)**:
  - Check key permissions: `chmod 600 ~/.ssh/key1`
  - Ensure public keys are in `~/.ssh/authorized_keys` on the server.
- **fail2ban Not Starting**:
  - Check status: `sudo systemctl status fail2ban`
  - View logs: `sudo journalctl -u fail2ban`
- **Connection Timeout**:
  - Verify the server’s public IP and security group SSH rule.
  - Ensure the instance is running in the AWS Console.
