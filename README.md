CI/CD Pipeline for a Node.js Application
This project demonstrates a complete, automated CI/CD pipeline for a Node.js web application. The pipeline uses Jenkins to build, test, and deploy the application as a Docker container on AWS.

Public URL of Deployed Application: http://3.111.23.80:3000

Setup & Deployment Guide

The pipeline is defined in the Jenkinsfile and automates the following stages:

Build and Test: Installs dependencies (npm install) and runs automated tests (npm test). The pipeline stops if tests fail.

Build Docker Image: Uses the Dockerfile to create a versioned Docker image of the application, tagged with the Jenkins build number.

Push to AWS ECR: The image is pushed to a private and secure Amazon Elastic Container Registry (ECR) repository.

Deploy on EC2: The final stage stops the old container and runs a new one from the updated image pulled from ECR, making the changes live.
Tools & Services Used
Cloud Provider: Amazon Web Services (AWS) for all infrastructure.

Compute: AWS EC2 (t2.micro with Amazon Linux 2023) to host the Jenkins server and the final application container.

Container Registry: AWS ECR to securely store versioned Docker images.

Security: AWS IAM Roles for secure, keyless access from Jenkins to ECR, and Security Groups as a virtual firewall.

Monitoring: AWS CloudWatch for basic EC2 instance metrics.

CI/CD Server: Jenkins to orchestrate the entire automated workflow.

Containerization: Docker to package the application into a portable, reproducible image.

Source Control: GitHub to host the source code and trigger the pipeline via webhooks.
Challenges Faced & How You Solved Them
Challenge: Container Exited Immediately.

Problem: The deployed Docker container would start and then immediately stop. docker ps showed no running containers, but docker ps -a showed an "Exited" status.

Solution: We inspected the logs with docker logs <container-name> and found the error: Error: Cannot find module '/usr/src/app/server.js'. The root cause was a typo in the Dockerfile: the application's entrypoint was app.js, but the CMD instruction was calling server.js. Correcting this line to CMD [ "node", "app.js" ] solved the issue.

Challenge: Incomplete Source Repository.

Problem: Initially, the troubleshooting was difficult because the forked repository was missing the actual application code (app.js, package.json, etc.). This was the true origin of the "Cannot find module" error.

Solution: We identified that the application files were missing. We created the necessary files (app.js, package.json, and a test/test.js file) on the local machine and pushed them to the GitHub repository, which allowed the pipeline to finally build a complete and working image.

Challenge: Jenkins Node Was Offline.

Problem: The main Jenkins node was marked with a red cross and would not run any jobs.

Solution: This was diagnosed as a default Jenkins monitoring feature. On a small t2.micro instance with limited disk space, the "Free Disk Space" monitor automatically takes the node offline. The fix was to go to Manage Jenkins -> Nodes -> Built-in Node -> Configure and disable this monitor.

Possible Improvements If Given More Time
Infrastructure as Code (IaC): I would use Terraform to define all AWS resources (EC2, ECR, IAM, etc.) as code. This would make the entire environment version-controlled, automated, and easily reproducible.

Separate Build Environment: I would configure Jenkins to use a separate agent (worker) node for running builds. This improves security and performance by isolating the build environment from the Jenkins master.

Advanced Deployment Strategy: I would implement a Blue/Green deployment using an Application Load Balancer and an Auto Scaling Group. This would allow for zero-downtime releases by deploying the new version alongside the old one and only switching traffic after health checks pass.

Enhanced Security Scanning: I would add a security stage to the pipeline using a tool like Trivy to scan the Docker image for known vulnerabilities before pushing it to ECR.

