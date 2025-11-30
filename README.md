# EC2 configuration using Ansible

This ansible setup  configures a pre-provisioned EC2 instance.The playbook achieves the following objectives:

   - Copies a file (config.txt) to the /opt/ directory with specific permissions (0660).

   - Creates the required devops group and the non-root devops_user.

   - Installs the latest version of PostgreSQL.

   - Ensures the PostgreSQL service is enabled and running.

   - Installs and configures Apache web server (including enabling mod_rewrite and ensuring the service  is running on the default port 80).  

## Prerequisites
   - Ansible: Confirmed installation using ansible --version.
   - AWS Account: With permissions to launch EC2 instances and manage Security Groups.

## Steps 
1. Create an ec2 instance manually via console or use terraform:
   - Select an Ubuntu 22.04 LTS (or similar Debian-based) AMI, as the playbook uses the apt module.

   - Key Pair: Generate or assign an existing PEM key. You will need the path to this file.

   - Security Group: Create a new security group that allows:
        - SSH (Port 22): From your IP address only or everwhere for test.
        - HTTP (Port 80): From anywhere (0.0.0.0/0) to allow the public to access the Apache web server.
2. Setup passwordless aunthentication:
    Use the PEM key assigned to the instance to set up passwordless SSH login for the default user(usually ubuntu). This ensures Ansible can connect without prompts.
    ```bash
    # Set permissions on your key file first
    chmod 400 <PATH_TO_PEM_FILE>

    # Copy the SSH key to the remote authorized_keys file
    ssh-copy-id -f "-o IdentityFile=<PATH_TO_PEM_FILE>" ubuntu@<INSTANCE_PUBLIC_IP>
    ```
3. Configure the Ansible Inventory
    Open your inventory.ini file and replace the placeholders with the actual connection details for your EC2 instance. 

    Replace <INSTANCE_PUBLIC_IP> with your EC2 instance's public IP address.

4. Execute the Playbook
   Run the playbook from your main project folder, specifying your inventory file:
   ```bash
   ansible-playbook -i inventory.ini playbook.yaml
   ```
5. Verification
    After the playbook completes, verify the configuration:
     1. File Check: SSH into the EC2 instance as the designated non-root user.
        ```bash
        ssh ubuntu@<INSTANCE_PUBLIC_IP>
        ```
        - Then, check the file permissions:
        ```bash
            ls -l /opt/config.txt
            # Output should show: -rw-rw---- 1 devops_user devops ... /opt/config.txt
        ```

    2. Web Server Check: Open a browser and navigate to `http://<INSTANCE_PUBLIC_IP>` You should 
       see the Apache default welcome page.
    3. Database Check:
       ```bash
       sudo systemctl status postgresql
       # Should show: Active: active (running/exited)
       ```
       test the git diff
