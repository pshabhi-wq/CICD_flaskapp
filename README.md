# Student Registration System

A **Flask** web application to manage student records backed by **MongoDB**, deployed automatically via a **GitHub Actions** CI/CD pipeline to **AWS EC2** using **Docker** and **Amazon ECR**.

---

## Features

* List all registered students on the home page
* Add new student entries
* Update existing student information
* Delete student entries safely with confirmation
* Health check endpoint (\`/health\`) verifying application and database status

---

## Tech Stack

* **Backend:** Python, Flask
* **Database:** MongoDB (MongoDB Atlas)
* **Testing:** Pytest, Mongomock
* **Containerization:** Docker, Amazon ECR
* **CI/CD & Cloud:** GitHub Actions, AWS EC2 
* **Notifications:** SMTP / Gmail Action

---

---
## Folder Structure
```bash
CICD_flaskapp/
├── .github/
│   └── workflows/
│       └── ci-cd.yml                          # GitHub Actions CI/CD pipeline workflow
├── Screenshots_terminaloutput/                 # Pipeline, AWS, and local execution evidence
│   ├── ECR_images.JPG                         # AWS ECR repository showing pushed Docker images
│   ├── Flaskapp_onec2.JPG                     # Running Flask app container / EC2 deployment
│   ├── Pipeline_success_all_stages.JPG        # Successful GitHub Actions workflow execution
│   ├── RunTestfailure_emailnotification.JPG   # Email notification received on test stage failure
│   ├── Testfailurepipeline.JPG                # GitHub Actions pipeline failure log during test run
│   ├── VSCODE_Terminal_output.txt             # Local terminal execution output from VS Code
│   ├── ec2instance.JPG                        # AWS EC2 instance console overview
│   ├── enabling docker on ec2.JPG             # Command execution enabling Docker service on EC2
│   ├── installing docker on ec2.JPG           # Command execution installing Docker on EC2
│   ├── mongodbcluster.JPG                     # MongoDB Atlas database cluster configuration
│   ├── pipeline_success_email.JPG             # Email notification received on successful deployment
│   ├── powershell_termianl_output.txt         # PowerShell terminal output logs
│   └── reposecrets.JPG                        # Configured GitHub Repository Secrets
├── templates/
│   ├── base.html                              # Base HTML template with Bootstrap 5
│   ├── index.html                             # Home page listing all students
│   ├── add_student.html                       # Add student form page
│   └── update_student.html                    # Update student details form page
├── app.py                                     # Flask application routes and logic
├── test_app.py                                # Pytest unit and integration tests
├── requirements.txt                           # Python project dependencies
├── Dockerfile                                 # Containerization instructions for Flask app
├── .dockerignore                              # Files excluded from Docker image build
├── .env.example                               # Template for environment variables
└── README.md                                  # Project documentation
```
---

## Setup Instructions for flaskapp

### 1. Clone the repository

```bash
git clone <your-repo-url>
cd <repo-folder>
```

### 2. Create and activate a virtual environment

```bash
python -m venv venv
# Activate venv
# Windows:
venv\Scripts\activate
# Linux / Mac:
source venv/bin/activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

**`requirements.txt` example:**

```
Flask
Flask-PyMongo
python-dotenv
bson
```

### 4. Configure environment variables

Create a `.env` file in the project root:

```
MONGO_URI=<your-mongodb-connection-string>
SECRET_KEY=<your-secret-key>
```

### 5. Run the application

```bash
python app.py
```

Open your browser at: [http://localhost:8000](http://localhost:8000)

---

---

## Notes

* Make sure MongoDB is running and accessible via the URI in `.env`
* Delete action includes a confirmation page to prevent accidental deletion
* Uses `ObjectId` from `bson` to work with MongoDB document IDs
* If you use MongoDB Atlas on macOS, install dependencies again (`pip install -r requirements.txt`). This project now uses `certifi` CA bundle explicitly to avoid common TLS certificate verification failures with `pymongo`.

---

## Setup instructions for CICD Pipeline

## Prerequisites

### 1. AWS Resources
* **AWS ECR Repository:** Private ECR repository named \`flaskapp\`.
* **AWS EC2 Instance:** AMI Server instance with inbound port \`5000\` (Flask app) and port \`22\` (SSH) open in Security Groups.
* **EC2 Dependencies:** Docker and AWS CLI installed on the EC2 host, with the \`ec2-user\` user added to the \`docker\` group (\`sudo usermod -aG docker ec2-user\`).

### 2. IAM Permissions
* An IAM User with policies granting access to ECR (\`AmazonEC2ContainerRegistryPowerUser\`) to allow pushing and pulling images.

---

## Pipeline Secrets Configuration

Navigate to **GitHub Repository > Settings > Secrets and variables > Actions** and configure the following secrets:

| Secret Name | Description | Example Value |
| :--- | :--- | :--- |
| \`MONGO_URI\` | Connection string for MongoDB Atlas | \`mongodb+srv://user:pass@cluster.mongodb.net/flask_db\` |
| \`AWS_ACCESS_KEY_ID\` | AWS IAM Access Key ID | \`AKIA...\` |
| \`AWS_SECRET_ACCESS_KEY\` | AWS IAM Secret Access Key | \`wJalrXUtnFEMI...\` |
| \`AWS_SESSION_TOKEN\` | *(Optional)* Temporary token for AWS Sandbox | \`IQoJb3JpZ2luX2Vj...\` |
| \`AWS_REGION\` | AWS Region code | \`us-east-1\` |
| \`ECR_REPOSITORY\` | Amazon ECR Repository Name | \`flaskapp\` |
| \`EC2_HOST\` | Public IP or DNS of the EC2 Instance | \`54.210.xx.xx\` |
| \`EC2_USERNAME\` | SSH username for the EC2 Instance | \`ec2-user\` |
| \`EC2_SSH_KEY\` | Private SSH key (\`.pem\`) contents | \`-----BEGIN OPENSSH PRIVATE KEY-----...\` |
| \`MAIL_USERNAME\` | Sender Gmail address | \`your_email@gmail.com\` |
| \`MAIL_PASSWORD\` | Gmail App Password (16-char without spaces) | \`abcdefghijklmnop\` |
| \`NOTIFICATION_EMAIL\` | Recipient email for alerts | \`recipient@gmail.com\` |

---

## Deployment Strategy: Connection via SSH

This pipeline uses **SSH (\`appleboy/ssh-action\`)** to connect to the AWS EC2 instance. 

* **Why SSH was chosen:** It provides a lightweight, direct, and cloud-agnostic deployment mechanism that requires no additional AWS agent setup (like AWS SSM Agent) on the host instance. It executes script commands directly over an encrypted remote shell key pair.

**Deployment Steps Executed via SSH:**
1. Authenticates Docker to Amazon ECR.
2. Stops and removes any existing \`flask-app\` container.
3. Pulls the newly built image tagged with the commit SHA (\`\${{ github.sha }}\`).
4. Runs the new container passing the \`MONGO_URI\` environment variable.
5. Performs a health check via \`curl http://localhost:5000/health\` to confirm app stability.

---

## Manual Deployment Guide (Pipeline Fallback)

If GitHub Actions is unavailable, reproduce the deployment manually from your local terminal:


# 1. Log in to Amazon ECR
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com

# 2. Build the Docker Image locally
docker build -t flaskapp:latest .

# 3. Tag and Push to ECR
docker tag flaskapp:latest <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/flaskapp:latest
docker push <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/flaskapp:latest

# 4. SSH into EC2 Instance
ssh -i /path/to/key.pem ec2-user@<EC2_PUBLIC_IP>

# 5. Execute Deployment Commands on EC2
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com
docker stop flask-app || true
docker rm flask-app || true
docker pull <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/flaskapp:latest
docker run -d --name flask-app --restart unless-stopped -e MONGO_URI="your_mongo_uri" -p 5000:5000 <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/flaskapp:latest

# 6. Verify Health
curl http://localhost:5000/health



---
## Verification & Screenshots Evidence

The `Screenshots_terminaloutput/` folder contains complete evidence documenting the project implementation:

* **Pipeline Execution & Email Alerts**
  * `Pipeline_success_all_stages.JPG` – Confirms end-to-end completion of tests, image build, and deployment.
  * `Testfailurepipeline.JPG` – Documents pipeline failure handling during test execution.
  * `pipeline_success_email.JPG` – Verifies success notification email sent via SMTP.
  * `RunTestfailure_emailnotification.JPG` – Verifies failure alert notification email sent via SMTP.

* **AWS & Infrastructure Documentation**
  * `reposecrets.JPG` – Shows configured GitHub Repository Secrets.
  * `ec2instance.JPG` – Confirms active EC2 instance configuration.
  * `installing docker on ec2.JPG` & `enabling docker on ec2.JPG` – Documents Docker setup on the EC2 host.
  * `ECR_images.JPG` – Confirms pushed container images in Amazon ECR.
  * `Flaskapp_onec2.JPG` – Confirms the live application running inside Docker on EC2.
  * `mongodbcluster.JPG` – Documents active MongoDB Atlas cluster configuration.

* **Terminal Output Logs**
  * `VSCODE_Terminal_output.txt` – Local development logs from VS Code.
  * `powershell_termianl_output.txt` – Terminal output logs from local PowerShell runs.



## License

MIT License

---



