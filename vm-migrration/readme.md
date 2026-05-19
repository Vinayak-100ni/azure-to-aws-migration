# Azure to AWS Environment Migration Plan

---

# 1. IAM Dashboard

## Step 1.1: Open AWS Dashboard

1. Login to the AWS Console.
2. In the AWS search box, search for:

```text
AWS Identity and Access Management
```

3. Open the IAM Dashboard.

---

## Step 1.2: Open IAM Users

1. In the left sidebar, click:

```text
Users
```

2. Click the **Create user** button visible at the top corner.

---

## Step 1.3: Create IAM User

1. In the **User name** field, enter a username for the migration user.
2. Click the **Next** button.

---

## Step 1.4: Attach Required Policies

In the permissions section:

1. Select:

```text
Attach policies directly
```

2. Search and attach the following policies:

```text
AWSApplicationMigrationFullAccess
AWSApplicationMigrationAgentInstallationPolicy
AWSApplicationMigrationConversionServerPolicy
```

3. Select all required policies.
4. Click the **Next** button.

---

## Step 1.5: Create User

1. Review the configuration.
2. Click the **Create user** button.

The IAM user will now be created with permissions required for Azure to AWS migration using:

- AWS Application Migration Service

---

## Step 1.6: Add Custom IAM Policy for PassRole Permission

AWS MGN requires additional `iam:PassRole` permission to allow the migration service to pass the conversion server role during migration operations.

### Create Custom Inline Policy

1. Open the created IAM user.
2. Go to the **Permissions** tab.
3. Click:

```text
Add permissions
Create inline policy
```

4. Select the **JSON** tab.
5. Add the following policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "iam:PassRole",
      "Resource": "arn:aws:iam::590184036962:role/service-role/AWSApplicationMigrationConversionServerRole"
    }
  ]
}
```

6. Click **Review policy**
7. Provide a policy name
8. Click **Create policy**

---

## Step 1.7: Generate Access Key and Secret Access Key

1. Open the created IAM user.
2. Go to:

```text
Security credentials
```

3. Scroll down to:

```text
Access keys
```

4. Click:

```text
Create access key
```

---

### Select Access Key Use Case

1. Select:

```text
Command Line Interface (CLI)
```

2. Check the confirmation checkbox.
3. Click **Next**.

---

### Add Description Tag

1. Optionally provide a description tag.
2. Click **Next**.

---

### Retrieve Access Keys

You will now see:

- Access Key ID
- Secret Access Key

> Important: Secret Access Key is shown only once. Store it securely.

---

# 2. AWS Application Migration Service

## Step 2.1: Open AWS Application Migration Service

1. Login to AWS Console.
2. Search for:

```text
AWS Application Migration Service
```

3. Open the dashboard.

---

## Step 2.2: Open Source Servers Section

1. In the left sidebar, click:

```text
Source servers
```

2. Click:

```text
Add server
```

---

## Step 2.3: Add Credentials in Server Replication Form

1. Paste:

- Access Key ID
- Secret Access Key

2. Copy the generated installation command or URL provided by AWS.

---

# 3. Connect to Azure Virtual Machine

1. Login to Azure Portal.
2. Open:

```text
Virtual Machines
```

3. Select the VM to migrate.
4. Copy the Public IP Address.

---

## SSH into Azure VM

```bash
ssh -i file.pem azureuser@YOUR_PUBLIC_IP
```

---

## Step 3.1: Disable Secure Boot

1. Open VM in Azure Portal.
2. Go to:

```text
Settings → Configuration
```

3. Uncheck:

```text
Enable Secure Boot
```

4. Save changes.
5. Restart VM if required.

---

# 4. Run Required Preparation Commands in Azure VM

## Step 4.1: Create New User

```bash
sudo adduser user_name
```

```bash
sudo usermod -aG sudo user_name
```

```bash
groups user_name
```

```bash
sudo whoami
```

```bash
su - aws_user
```

```bash
su aws_user
```

---

## Step 4.2: Install Required Linux Kernel Packages

```bash
sudo apt install linux-image-generic linux-headers-generic
```

Description:
Installs generic Linux kernel packages required for AWS compatibility.

---

## Step 4.3: Update Initramfs

```bash
sudo update-initramfs -u -k all
```

---

## Step 4.4: Update GRUB Configuration

```bash
sudo update-grub
```

---

## Step 4.5: Open GRUB Configuration File

```bash
sudo nano /etc/default/grub.d/40-force-partuuid.cfg
```

Comment out existing lines:

```bash
# GRUB_FORCE_PARTUUID="..."
```

Purpose:
Disables Azure-specific PARTUUID boot configuration.

---

## Step 4.6: Update GRUB Again

```bash
sudo update-grub
```

---

## Step 4.7: Reboot Server

```bash
sudo reboot
```

---

# 5. Switch Azure Kernel to Generic Linux Kernel (Ubuntu VM)

---

## Step 5.1: Open Azure Portal

1. Navigate to:

```text
Virtual Machines
```

2. Select target VM.

---

## Step 5.2: Open Serial Console

1. In left sidebar search:

```text
Serial Console
```

2. Open it.

---

## Step 5.3: Access GRUB Menu

1. During boot press:

```text
ESC
```

2. GRUB menu will open.

---

## Step 5.4: If ESC Timing Missed

1. Restart VM.
2. Open Serial Console again.
3. Press ESC immediately during boot.

---

## Step 5.5: Open Advanced Options

Select:

```text
Advanced options for Ubuntu
```

---

## Step 5.6: Select Generic Kernel

Select kernel similar to:

```text
Ubuntu, with Linux <version>-generic
```

---

## Step 5.7: Confirm Boot

System should boot using generic kernel successfully.

---

## Step 5.8: Completion

Close serial console after successful login.

---

# 6. Wait for Reboot

Wait approximately 2 minutes.

Reconnect using SSH:

```bash
ssh -i file.pem <sudo_user>@YOUR_PUBLIC_IP
```

---

## Step 6.1: Install AWS Replication Agent

### Download Installer

```bash
sudo wget -O ./aws-replication-installer-init https://aws-application-migration-service-ap-south-1.s3.ap-south-1.amazonaws.com/latest/linux/aws-replication-installer-init
```

---

### Make Installer Executable and Run

```bash
sudo chmod +x aws-replication-installer-init
```

```bash
sudo ./aws-replication-installer-init --region ap-south-1 --aws-access-key-id admin --aws-secret-access-key pass1234 --no-prompt
```

---

## Step 6.2: Wait for Replication Process

Replication time depends on:

- VM size
- Disk size
- Network speed
- Internet connectivity

AWS will continuously sync Azure VM data to AWS.

---

# 7. Verify Migrated Server in AWS

1. Open:

```text
AWS Application Migration Service
```

2. Go to:

```text
Source servers
```

3. Wait until server status becomes:

```text
Healthy
Ready for testing
```

---

## Step 7.1: Launch Test Instance

1. Select migrated server.
2. Click:

```text
Test and cutover
```

3. Select:

```text
Launch test instances
```

4. Click launch.

AWS will begin creating EC2 test instance.

---

## Step 7.2: Wait for AWS Configuration

AWS will:

- Configure networking
- Attach replicated disks
- Prepare EC2 instance
- Perform validations

This process may take time.

---

## Step 7.3: Access the Test EC2 Instance

1. Open:

```text
EC2 Console → Instances
```

2. Select migrated instance.
3. Click:

```text
Connect
```

---

### Option 1: EC2 Serial Console

Used for advanced troubleshooting.

---

### Option 2: SSH Connection

```bash
ssh -i your-key.pem ubuntu@PUBLIC_IP
```

---

## Step 7.4: Restore Application (PM2 Recovery Process)

### Remove Existing Logs

```bash
rm -rf /home/sdtech/.pm2/logs
```

---

### Recreate Logs Directory

```bash
mkdir -p /home/sdtech/.pm2/logs
```

---

### Restore PM2 Processes

```bash
pm2 resurrect
```

---

## Purpose of PM2 Restore

- Restores saved Node.js processes
- Recreates runtime state after migration
- Automatically starts applications

---

# Migration Complete

Your Azure VM has now been migrated successfully to AWS using AWS Application Migration Service (MGN).
