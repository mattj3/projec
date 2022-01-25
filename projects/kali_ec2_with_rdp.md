# Creating a Kali Linux EC2 instance with Remote Desktop capabilities

1. Navigate to EC2 in the AWS console 
2. Click “Launch instances”, then type `Kali` in the Search bar (and hit enter)
3. Find the “Kali Linux” AMI by Kali Linux in the AWS Marketplace tab, then click “Select” (and “Continue” if prompted with an overview modal)
4. Select the Instance Type that is appropriate for you, the vendor recommended type will be preselected and at the time of writing this, Kali Linux recommended a t2.medium
5. Work your way through the following steps and make adjustments as you require 
    1. Configure Instance 
    2. Add Storage 
        - Note: 12 Gibibyte was pre-filled for me, but after tool install and Remote Desktop setup, I had less than 1 GiB remaining. 
    3. Add Tags 
    4. Configure Security Group 
        - Note: You will need port 3390 opened inbound for RDP to work (according to the future configuration provided by Kali Linux)
    5. Review and Launch 
        - Note: The creation of the instance can take longer when using AWS Marketplace AMIs compared to using AWS Quick Start AMIs
6. Once the instance is up and running, SSH in
    
    ```bash
    ssh -i ~/location/of/keypair.pem kali@ec2-x-xx-xx-xx.compute-1.amazonaws.com
    ```
    
    If you have issues with your .pem file, make sure to chmod 
    
    ```bash
    chmod 400 ~/location/of/keypair.pem
    ```
    
7. Next, head over to [https://www.kali.org/docs/general-use/xfce-with-rdp/](https://www.kali.org/docs/general-use/xfce-with-rdp/) and run through the setup of RDP with Xfce. As of January 2022 the below is the meat of the content. 
    
    ```bash
    # Here is the setup in a nutshell 
    wget https://gitlab.com/kalilinux/build-scripts/kali-wsl-chroot/-/raw/master/xfce4.sh
    chmod +x xfce4.sh
    sudo ./xfce4.sh
    sudo systemctl enable xrdp --now
    sudo /etc/init.d/xrdp start # not sure if needed, try without first
    
    ---
    # Only the above is required for setup
    # This is the script that wget, by chance the link fails in the future 
    #!/bin/sh
    echo "[+] Installing Xfce, this will take a while"
    apt-get update
    apt-get dist-upgrade -y
    apt-get install -y kali-desktop-xfce xrdp
    
    echo "[+] Configuring XRDP to listen to port 3390 (but not starting the service)..."
    sed -i 's/port=3389/port=3390/g' /etc/xrdp/xrdp.ini
    ```
    
8. Now, change the password for the kali user (this will be used to RDP in)
9. For the Remote Desktop app I chose to use Microsoft Remote Desktop. Download the application then move onto the next step. 
10. With the Microsoft Remote Desktop application open, we will click the “+” then “Add PC”
11. For “PC name” use either the Public IPv4 address (located in the AWS console → EC2 → click your instance → click “Details” tab) or the Public IPv4 DNS. 
    - Note: The script above configures port 3390. Be sure to add the port to the end of the IP Address/DNS. Ex. ec2-x-xx-xx-xx.compute-1.amazonaws.com:3390 
12. For the “User account” dropdown, click the “Add Use Account...” option
    1.  Username: `kali`
    2.  Password: `whateveryouchangedthepasswordto`
    3.  Click “Add”
13. Click “Add” once more at the bottom of the Add PC modal
14. Lastly, double click on the newly created tile and verify you are able to remote in to your newly created EC2 instance. 
    - Note: If you are using an instance smaller than the recommend, it may take a little while for things to load. Also, if you attempt to open the browser with a T2.micro, it will most likely brick the instance.