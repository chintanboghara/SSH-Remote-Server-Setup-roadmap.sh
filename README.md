# Setting up a remote Linux server on AWS and configuring SSH access:

### **1. Set Up a Remote Linux Server on AWS**
1. **Create an AWS Account**: Register at [AWS](https://aws.amazon.com/).
2. **Launch an EC2 Instance**:
   - Go to the EC2 Dashboard and click "Launch Instances."
   - Select an Amazon Machine Image (AMI), such as Ubuntu Server or Amazon Linux.
   - Choose an instance type (e.g., `t2.micro` for free-tier usage).
   - Configure network settings, ensuring port 22 (SSH) is open in the security group.
   - Launch the instance, creating an initial SSH key pair to download (for the first connection).

### **2. Create Two New SSH Key Pairs**
1. **Generate Keys Locally**:
   Run the following command twice to create two separate key pairs:
   ```bash
   ssh-keygen -t rsa -b 4096 -f ~/.ssh/key1
   ssh-keygen -t rsa -b 4096 -f ~/.ssh/key2
   ```
   Replace `key1` and `key2` with your desired filenames.
2. **Add Public Keys to the Server**:
   - Connect to the server using the initial key:
     ```bash
     ssh -i <path-to-initial-private-key> ubuntu@<server-ip>
     ```
   - Add the new public keys to `~/.ssh/authorized_keys` on the server:
     ```bash
     cat ~/.ssh/key1.pub >> ~/.ssh/authorized_keys
     cat ~/.ssh/key2.pub >> ~/.ssh/authorized_keys
     ```

### **3. Test SSH Access with Both Keys**
1. Connect using the first key:
   ```bash
   ssh -i ~/.ssh/key1 ubuntu@<server-ip>
   ```
2. Connect using the second key:
   ```bash
   ssh -i ~/.ssh/key2 ubuntu@<server-ip>
   ```

### **4. Configure `~/.ssh/config` for Easier Access**
1. Edit the SSH config file:
   ```bash
   nano ~/.ssh/config
   ```
2. Add entries for both keys:
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
3. Save and test:
   ```bash
   ssh alias1
   ssh alias2
   ```
