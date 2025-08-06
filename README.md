🚀 Wanderlust - DevSecOps Automated CI/CD Pipeline

## 1️⃣ Prerequisites

Before starting, ensure:

* Ubuntu server or Jenkins node is ready.
* You have **sudo privileges**.
* Jenkins is installed and running on port `8080`.
* SonarQube server is running and accessible.
* GitHub repository: [`wanderlust_devops`](https://github.com/VaibhaviSugandhi1733/wanderlust_devops.git).

---

## 2️⃣ Install Required Tools

### 2.1 System Update

```bash
sudo apt update
```

### 2.2 Install Java (Required for Jenkins and Sonar Scanner)

```bash
sudo apt install fontconfig openjdk-17-jre -y
```

### 2.3 Install Jenkins

```bash
wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins-io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list
sudo apt-get update
sudo apt-get install jenkins -y
sudo systemctl enable jenkins
sudo systemctl start jenkins
```

### 2.4 Install Docker

```bash
sudo apt install docker.io -y
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER
sudo chown $USER /var/run/docker.sock
```

(Reboot if needed for permissions to apply.)

### 2.5 Install Docker Compose

```bash
sudo apt install docker-compose -y
```

### 2.6 Install Trivy

```bash
sudo apt install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo tee /usr/share/keyrings/trivy-keyring.asc
echo "deb [signed-by=/usr/share/keyrings/trivy-keyring.asc] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/trivy.list
sudo apt update
sudo apt install trivy -y
```

### 2.7 Install OWASP Dependency-Check

```bash
wget https://github.com/jeremylong/DependencyCheck/releases/download/v8.4.2/dependency-check-8.4.2-release.zip
unzip dependency-check-8.4.2-release.zip -d /opt/
sudo ln -s /opt/dependency-check/bin/dependency-check.sh /usr/local/bin/dependency-check.sh
```

---

## 3️⃣ Jenkins Configuration

1. **Access Jenkins**:
   Open browser → `http://<server-ip>:8080`
2. **Unlock Jenkins**:

   ```bash
   sudo cat /var/lib/jenkins/secrets/initialAdminPassword
   ```
3. **Install Plugins**:

   * Git
   * SonarQube Scanner
   * OWASP Dependency-Check
   * Pipeline
   * Docker Pipeline
4. **Configure Global Tools**:

   * Add SonarQube server under **Manage Jenkins → Configure System**
   * Add Docker credentials if pushing images to DockerHub
5. **Create Credentials**:

   * SonarQube token (as `sonar-token-id`)
   * GitHub credentials if private repo

---

## 4️⃣ Clone the Repository

On Jenkins server:

```bash
cd /var/lib/jenkins/workspace
git clone https://github.com/VaibhaviSugandhi1733/wanderlust_devops.git
```

---

## 5️⃣ Prepare Jenkins Pipeline

1. Open Jenkins → **New Item → Pipeline**
2. Add repo URL (if using Pipeline from SCM)
3. Point to `Jenkinsfile` in the repository
4. Save

---

## 6️⃣ End-to-End Pipeline Execution

### Stage 1: **Checkout Code**

* Jenkins pulls the `wanderlust_devops` repo from GitHub.

### Stage 2: **Install Tools (First Run Only)**

* Java, Docker, Docker Compose, Trivy, OWASP Dependency-Check installed automatically if not already present.

### Stage 3: **SonarQube Code Analysis**

* Jenkins triggers Sonar Scanner to analyze source code.
* Sonar Quality Gates enforce minimum quality standards.

### Stage 4: **Dependency Check**

* OWASP Dependency-Check scans for vulnerable libraries in `package.json` or other files.

### Stage 5: **Build Docker Image**

* Jenkins builds a Docker image for the backend service.
* Trivy scans the image for vulnerabilities.

### Stage 6: **Run Automated Tests**

* Unit and integration tests are executed.
* Jenkins archives test results.

### Stage 7: **Deploy via Docker Compose**

* Jenkins runs `docker-compose down` (if running) and `docker-compose up -d --build` to deploy the latest app.

### Stage 8: **Post-Build Actions**

* Jenkins generates reports:

  * SonarQube report
  * OWASP Dependency-Check report
  * Trivy vulnerability scan report

---

## 7️⃣ Verify Deployment

1. Check running containers:

   ```bash
   docker ps
   ```
2. Access the application in browser:

   ```
   http://<server-ip>:<mapped-port>
   ```

---

## 8️⃣ Security & Quality Reports

* **SonarQube**: View in SonarQube dashboard
* **Dependency-Check**: `dependency-check-report.html` in Jenkins artifacts
* **Trivy**: Vulnerability scan logs in Jenkins console

---

## 9️⃣ Optional (Push to DockerHub)

If you want to push the Docker image:

```bash
docker login
docker tag wanderlust:latest <dockerhub-username>/wanderlust:latest
docker push <dockerhub-username>/wanderlust:latest
```

---

