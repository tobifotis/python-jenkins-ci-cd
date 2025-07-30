# Jenkins BlueOcean CI/CD Setup with Docker

This README provides a comprehensive guide for setting up Jenkins with the BlueOcean UI using Docker, configuring Docker-in-Docker agents, and running your first CI/CD pipeline from a GitHub repository.

---

## Prerequisites

- Docker installed on your machine
- GitHub repository with a `Jenkinsfile`
- Basic knowledge of shell commands

---

# 1. Installation

## Step 1: Build the Jenkins BlueOcean Docker Image
You can either build your own Jenkins BlueOcean image or pull an existing one.

### Option A: Build Custom Jenkins BlueOcean Image
```bash
docker build -t myjenkins-blueocean:latest .
```

### Option B: Pull Prebuilt Jenkins BlueOcean Image
```bash
docker pull devopsjourney1/jenkins-blueocean:2.332.3-1
docker tag devopsjourney1/jenkins-blueocean:2.332.3-1 myjenkins-blueocean:2.332.3-1
```

## Step 2: Create the Docker Network
```bash
docker network create jenkins
```

## Step 3: Run the Jenkins Container
### For Windows:
```powershell
docker run --name myjenkins-blueocean --restart=on-failure --detach `
  --network jenkins --env DOCKER_HOST=tcp://docker:2376 `
  --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 `
  --volume jenkins-data:/var/jenkins_home `
  --volume jenkins-docker-certs:/certs/client:ro `
  --publish 8080:8080 --publish 50000:50000 myjenkins-blueocean:latest
```

### For MacOS / Linux:
```bash
docker run --name myjenkins-blueocean --restart=on-failure --detach \
  --network jenkins --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
  --publish 8080:8080 --publish 50000:50000 \
  --volume jenkins-data:/var/jenkins_home \
  --volume jenkins-docker-certs:/certs/client:ro \
  myjenkins-blueocean:latest
```

## Step 4: Access Jenkins
Open your browser and go to:
```
http://localhost:8080
```

## Step 5: Retrieve Admin Password
```bash
docker exec myjenkins-blueocean cat /var/jenkins_home/secrets/initialAdminPassword
```

---

# 2. Setup Cloud Docker Agent

## Step 1: Launch Alpine/Socat Helper Container
```bash
docker run -d --restart=always -p 127.0.0.1:2376:2375 --network jenkins \
  -v /var/run/docker.sock:/var/run/docker.sock alpine/socat \
  tcp-listen:2375,fork,reuseaddr unix-connect:/var/run/docker.sock
```

## Step 2: Get Container IP Address
```bash
docker ps
# Note the container ID

docker inspect <container_id>
# Look for the IPAddress field
```

## Step 3: Add Docker Cloud in Jenkins
- Go to: `Manage Jenkins -> Docker -> Add New Cloud`
- Cloud name: `docker`
- Type: `Docker`
- Docker Host URI: `tcp://<IPAddress>:2375`
- Click 'Enabled' and then 'Test Connection'
- Click 'Save'

## Step 4: Add Docker Agent Templates
- Navigate to: `Manage Jenkins -> Docker -> Clouds -> docker -> Add Docker Agent Template`
- **Labels**: `docker-agent-alpine`
- **Enabled**: Checked
- **Name**: `docker-agent-alpine`
- **Docker Image**: `jenkins/agent-alpine-jdk17`
- **Instance Capacity**: 2
- **Remote File System Root**: `/home/jenkins`
- Save your configuration

## Step 5: Configure Job to Use Cloud Agent
- Open any job (e.g., `my_first_job`)
- Go to `Configure`
- Check `Restrict where this project can be run`
- Enter label: `docker-agent-alpine`
- Save

---

# 3. Setup CI/CD Pipeline from GitHub Repository

## Step 1: Create New Pipeline Job
- Jenkins → `New Item`
- Name: `my_first_build_pipeline`
- Type: `Pipeline`
- OK

## Step 2: Link to GitHub Repo
- In the configuration:
  - `Pipeline -> Definition`: Pipeline script from SCM
  - `SCM`: Git
  - `Repository URL`: `https://github.com/tobifotis/python-jenkins-ci-cd`
  - `Script Path`: `Jenkinsfile`
  - Save

## Step 3: Build
- Click `Build Now`

---

# 4. Run Shell Commands in Jenkins Pipeline

In a pipeline or freestyle job, you can use shell commands:
```bash
echo "Hello World"
echo "The build ID of this job is ${BUILD_ID}"
echo "The build URL of this job is ${BUILD_URL}"

ls -ltr
echo "1234" > test.txt
ls -ltr
```

---

# 5. Useful Jenkins Docker Commands

## SSH into Jenkins Container
```bash
docker exec -it myjenkins-blueocean bash
```

## Explore Jenkins Workspace
```bash
cd /var/jenkins_home/workspace
ls -ltr
cd my_first_job
ls -ltr
cat test.txt
```

---

# 6. Bonus: Run Python Script via Freestyle Job

## Create Job: `my_python_job`
- Type: Freestyle Project
- Source Code Management → Git
  - Repository URL: *<your repository>*
- Build Steps → Execute Shell:
```bash
python3 helloworld.py
```

---

# References
- Jenkins Docker Installation: https://www.jenkins.io/doc/book/installing/docker/
