MEAN Stack Application Deployment Summary
The following document outlines the complete deployment process of a full-stack MEAN (MongoDB, Express, Angular, Node.js) application using Docker, Jenkins, and AWS EC2. This includes the challenges faced, solutions implemented, and the final architecture used to ensure smooth CI/CD and reliable production deployment.
________________________________________
Tools Used
Docker & Docker Compose
Jenkins (Master–Agent Architecture)
AWS EC2
Instance Type: t2.xlarge (4 vCPU & 16 GB RAM)
________________________________________
Project Overview
The main objective was to deploy a MEAN stack application using containerized services and automate deployments using Jenkins CI/CD pipelines.
Docker Hub was used to store images, while an Nginx reverse proxy ensured clean routing without exposing backend ports. AWS EC2 instances provided the infrastructure to host both Jenkins and the application.
________________________________________
Challenges & Resolutions
1. Angular Frontend Errors
During the initial setup, executing the Angular build from the /frontend directory caused failures due to:
Angular CLI version mismatch
Incorrect or missing configurations
Fix:
The correct Angular CLI version was installed, configuration files were updated, and the frontend application was rebuilt successfully.
________________________________________
2. Backend Not Accepting Tutorial Additions
While the UI loaded successfully, adding new tutorials failed. This issue was traced to incorrect CORS handling and server routing logic.
Fix:
CORS was properly enabled after initializing the Express app:
const express = require("express");
const cors = require("cors");

const app = express();
app.use(cors());
After making changes in both frontend and backend code, the tutorial creation feature started functioning correctly.
________________________________________
3. Backend Container Crashing (MongoDB Not Found)
The backend container repeatedly exited because it was unable to connect to MongoDB.
Fix:
The MongoDB connection URL was updated inside the config:
module.exports = {
  url: "mongodb://mongodb:27017/dd_db"
};
Once the correct hostname was used, communication between the backend and MongoDB stabilized.
________________________________________
4. Dockerfile & Multi-Stage Build
To optimize performance and reduce image size:
Stage 1 builds the Angular application using Node 25 (Alpine)
Stage 2 uses Nginx to serve the compiled Angular files
The backend also uses a Node 25 Alpine-based image for lightweight deployment.
________________________________________
5. Jenkins Integration & Instance Performance Issues
Running Jenkins on the same EC2 instance as the application caused disk and performance problems.
Solution:
Two separate EC2 instances were created:
Jenkins Master
Jenkins Agent (Application Server)
SSH communication was configured between master and agent using:
ssh-keygen -t rsa
Adding the public key to authorized_keys
Adding the private key to Jenkins credentials
Additional permissions were granted:
sudo usermod -aG docker jenkins
sudo usermod -aG docker ubuntu
newgrp docker
sudo systemctl restart jenkins
sudo systemctl restart docker
With this setup, Jenkins could build and push Docker images from the agent node.
________________________________________
6. Webhooks for CI/CD
A GitHub webhook was configured to automatically trigger the Jenkins pipeline.
Whenever code is pushed to the repository:
The pipeline runs
A Docker image is built
The updated image is pushed to Docker Hub
The application is redeployed with the latest changes
This enabled a fully automated CI/CD process.
________________________________________
7. Nginx Reverse Proxy Setup
Initially, the application had to be accessed using ports such as IP:8081, which was not ideal for production.
Fix:
An Nginx reverse proxy was configured to:
Route frontend and backend traffic cleanly
Prevent direct exposure of application ports
Provide a single public endpoint for users
An Elastic IP was also assigned to ensure the application remains reachable even after EC2 restarts.

