# flask-app-demo

Here's a simple step-by-step guide for a DevOps project using AWS EC2, Jenkins, Docker, and Git. This project involves setting up a CI/CD pipeline to deploy a web application automatically whenever changes are pushed to a Git repository.

### **Overview:**
- **Tools Used:** AWS EC2, Jenkins, Git, Docker.
- **Objective:** Deploy a simple web application (e.g., a Node.js app) using a Jenkins pipeline on an AWS EC2 instance with Docker.

### **Pre-requisites:**
- AWS account (with access to launch EC2 instances).
- Basic knowledge of Git, Jenkins, and Docker.
- An SSH key pair for accessing the EC2 instance.

---

### **Step 1: Launch an AWS EC2 Instance**
1. **Log in to AWS Console** and navigate to EC2.
2. **Launch an EC2 instance** with the following:
   - **AMI:** Choose Ubuntu 22.04 LTS.
   - **Instance Type:** t2.micro (free-tier eligible).
   - **Key Pair:** Create or use an existing key pair for SSH access.
   - **Security Group:** Allow SSH (port 22), HTTP (port 80), and Jenkins (port 8080).

3. Once the instance is running, **connect to the EC2 instance** using SSH:
   ```bash
   ssh -i your-key-pair.pem ubuntu@your-ec2-public-ip
   ```

---

### **Step 2: Install Jenkins on EC2**
1. **Update packages and install Java:**
   ```bash
   sudo apt update
   sudo apt install openjdk-11-jdk -y
   ```

2. **Add Jenkins repository and install Jenkins:**
   ```bash
   curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee \
     /usr/share/keyrings/jenkins-keyring.asc > /dev/null
   echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
     https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
     /etc/apt/sources.list.d/jenkins.list > /dev/null
   sudo apt update
   sudo apt install jenkins -y
   ```

3. **Start Jenkins and enable it to run at startup:**
   ```bash
   sudo systemctl start jenkins
   sudo systemctl enable jenkins
   ```

4. **Access Jenkins through the browser** using `http://your-ec2-public-ip:8080`.

5. **Unlock Jenkins** with the initial admin password:
   ```bash
   sudo cat /var/lib/jenkins/secrets/initialAdminPassword
   ```

---

### **Step 3: Install Docker on EC2**
1. **Install Docker:**
   ```bash
   sudo apt install docker.io -y
   sudo systemctl start docker
   sudo systemctl enable docker
   ```

2. **Add Jenkins user to the Docker group** to allow Jenkins to execute Docker commands:
   ```bash
   sudo usermod -aG docker jenkins
   ```

3. **Restart Jenkins** to apply the changes:
   ```bash
   sudo systemctl restart jenkins
   ```

---

### **Step 4: Set Up a Sample Node.js Application**
1. **Create a new GitHub repository** for your Node.js application.
2. **Clone the repository** into your local machine and create a simple Node.js app:
   ```bash
   git clone https://github.com/your-username/your-repository.git
   cd your-repository
   ```

3. **Add a simple `app.js` file:**
   ```javascript
   const express = require('express');
   const app = express();
   const PORT = process.env.PORT || 3000;

   app.get('/', (req, res) => {
     res.send('Hello, World!');
   });

   app.listen(PORT, () => {
     console.log(`Server is running on port ${PORT}`);
   });
   ```

4. **Create a `Dockerfile` for your Node.js app:**
   ```dockerfile
   FROM node:14
   WORKDIR /app
   COPY package*.json ./
   RUN npm install
   COPY . .
   EXPOSE 3000
   CMD ["node", "app.js"]
   ```

5. **Create a `Jenkinsfile` for CI/CD pipeline:**
   ```groovy
   pipeline {
       agent any
       stages {
           stage('Clone Repository') {
               steps {
                   git 'https://github.com/your-username/your-repository.git'
               }
           }
           stage('Build Docker Image') {
               steps {
                   script {
                       docker.build('nodejs-app')
                   }
               }
           }
           stage('Run Docker Container') {
               steps {
                   script {
                       docker.withRun('nodejs-app') {
                           sh 'echo "App is running"'
                       }
                   }
               }
           }
       }
   }
   ```

6. **Push these files** to your GitHub repository:
   ```bash
   git add .
   git commit -m "Initial commit"
   git push origin main
   ```

---

### **Step 5: Configure Jenkins Pipeline**
1. **Go to Jenkins Dashboard** > **New Item** > **Pipeline**.
2. **Name your pipeline** (e.g., `Nodejs-App-Pipeline`) and click **OK**.
3. **In the pipeline configuration**, choose:
   - **Pipeline script from SCM**.
   - **SCM:** Git.
   - **Repository URL:** Add your GitHub repository link.
   - **Branch to build:** `*/main`.

4. Click **Save** and **Build Now** to start the pipeline. This will:
   - Clone the repository.
   - Build the Docker image.
   - Run the Docker container with the Node.js application.

---

### **Step 6: Access the Deployed Application**
1. After the Jenkins pipeline successfully builds and runs the container, you can access your Node.js app.
2. **To access the app**, you may need to expose the Docker container’s port (3000) to the EC2 instance’s security group.

---

### **Source Code Summary**
Here are the key files for this project:
1. **app.js** (Node.js app)
2. **Dockerfile** (For creating Docker images)
3. **Jenkinsfile** (Pipeline configuration)

You can find the full source code and instructions [here](https://github.com/your-username/your-repository). (Make sure to replace this with your actual repository URL.)

This setup will give you hands-on experience with AWS EC2, Docker, and Jenkins for deploying a simple application through a CI/CD pipeline. Let me know if you have any questions or need further customization!
