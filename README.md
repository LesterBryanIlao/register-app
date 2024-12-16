DevOps-Project-with-Jenkins-Maven-SonaQube-Docker-and-EKS

Steps:
1. Setup Jenkins-Master
    a. Launch Jenkins-Master in EC2
        > Ubuntu
        > t2.micro
        > atleast 15GiB storage
    b. Connect via SSH or EC2 Instance Connect (to update hostname)
        1. sudo apt update
        2. sudo apt upgrade
        3. sudo nano /etc/hostname (update hostname to jenkins-master)
        4. cat /etc/hostname (check if hostname is updated)
        5. sudo init 6 (reboot)
    c. Install Java
        6. sudo apt install openjdk-17-jre
        7. java --version (check if java is installed)
    d. Install Jenkins
        8. Update security group to allow port 8080 inbound
        9. Get copy weekly release from: https://www.jenkins.io/doc/book/installing/linux/
            sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
                https://pkg.jenkins.io/debian/jenkins.io-2023.key
            echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
                https://pkg.jenkins.io/debian binary/ | sudo tee \
                /etc/apt/sources.list.d/jenkins.list > /dev/null
            sudo apt-get update
            sudo apt-get install jenkins
        10. jenkins --version (check if jenkins is installed)
        11. sudo systemctl enable jenkins (start jenkins service on boot)
        12. sudo systemctl start jenkins (start jenkins service)
        13. systemctl status jenkins (check if jenkins service is running)
    e. Allow SSH
        1. sudo nano /etc/ssh/sshd_config (update sshd_config to allow port 22)
        2. Uncomment PubkeyAuthentication yes
        3. Uncomment AuthorizedKeysFile .ssh/authorized_keys .ssh/authorized_keys2

2. Setup Jenkins-Agent
    a. Do steps a-c from "Setup Jenkins-Master"
    d. Install Docker
        1. sudo apt-get install docker.io
        2. docker --version (check if docker is installed)
        3. sudo usermod -aG docker $USER (add ubuntu to docker group)
        4. groups username (check if docker group is added)
        5. sudo init 6 (reboot)
3. Allow SSH
    a. sudo nano /etc/ssh/sshd_config (update sshd_config to allow port 22)
    b. Uncomment PubkeyAuthentication yes
    c. Uncomment AuthorizedKeysFile .ssh/authorized_keys .ssh/authorized_keys2
    d. sudo systemctl reload ssh (reload ssh service)
    e. Do steps a-c to Jenkins-Master and Jenkins-Agent EC2 instance
    f. Go to Jenkins-Master EC2 instance 
    g. ssh-keygen (generate ssh key) [Note: Just press enter for all questions]
    h. cd .ssh (move to ssh directory)
    i. copy the contents of id_rsa.pub
    j. Go to Jenkins-Agent EC2 instance
    k. cd .ssh (move to ssh directory)
    l. sudo nano authorized_keys (add the contents of id_rsa.pub)
    m. cat authorized_keys (check if authorized_keys is updated)

4. Open Jenkins-Master 
    a. <public-ip>:8080 (open in browser)
    b. sudo cat /var/lib/jenkins/secrets/initialAdminPassword (get password)
    c. login using the password
    d. Install Suggested Plugins
    e. Fill in the details
        1. username (e.g. clouduser)
        2. password (e.g. clouduser)
        3. full name
        4. password 
    

5. Setup Node
    a. Manage Jenkins -> Nodes -> Built-In Node -> Configure -> Number of executors -> 0 -> Save
    b. Nodes -> New Node -> put "Jenkins-Agent" -> Tick "Permanent Agent" -> Create
    c. Add Details:
        1. Number of executors: 2
        2. Remote root directory: /home/ubuntu
        3. Labels: "Jenkins-Agent"
        4. Usage: "Use this node as much as possible"
        5. Launch method: "Launch agents via SSH"
        6. Host: <private-ip of Jenkins-Agent>
        7. Credentials:
            > Add -> Jenkins
            > Domain: Global credentials
            > Kind: SSH Username with private key
            > ID: jenkins-agent
            > Description: Jenkins-Agent
            > Username: ubuntu
            > Private key: Enter directly
                > Go to Jenkins-Master
                > cd .ssh
                > cat id_ed25519
                > Copy the contents
            > Save
            > Select the newly created Jenkins-Agent
        8. Host Key Verification Strategy: "Non verifying Verification Strategy"
        9. Save
    d. Test
        1. Dashboard -> Create job -> put "Test" -> click Pipeline -> Click OK
        2. Pipeline Script -> Hello World -> Apply -> Save -> Build Now
