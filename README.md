# Containerizing a Python Flask App With Docker and Deploy on AWS Using Amazon ECS, API Gateway and Elastic load balancer (ELB)

## Project Overview

This project focuses on containerizing a Python Flask Application for Sports API management using Docker and deploying it on AWS with a robust, scalable architecture. The deployment leverages Amazon ECS (Fargate) for container orchestration, an Elastic Load Balancer (ELB) for distributing traffic across multiple containers, and API Gateway for exposing the application’s REST endpoints securely. This setup ensures high availability, scalability, and efficient handling of incoming requests, making the application production-ready.

## Architectural Diagram
<img width="1094" alt="Screenshot 2025-01-27 at 4 03 07 AM" src="https://github.com/user-attachments/assets/7b688e7b-1564-4a9b-920f-a779f961ef29" />

## Features
- Containerized Python Flask application and its dependencies into a lightweight, portable Docker container.
- Deployment on AWS ECS (Fargate).
- Ensures even traffic distribution across multiple container instances using Elastic Load Balancer (ELB).
- Exposes RESTful API endpoints of the Flask application securely.
- Amazon ECR for Docker Image Management.
- API management and routing using API Gateway
- Exposes Application's REST API endpoints securely.

## PreRequisites
- SerpAPI: Signup for Free account and subscribe for API key.
- AWS Free account with appropriate roles for user: (i.e AmazonEC2ContainerRegistryFullAccess).
- AWS CLI Installed and Configured to interact with AWS account.
- SerAPI Library Installed.
- Docker Desktop Installed: To Build Docker file and Push Docker Images.

## Technological Stacks
- AWS Cloud platform
- AWS Core services: Amazon ECS (Fargate), Elastic Load Balancer (ELB), Amazon ECR, API Gateway
- Containrization Service: Docker
- Programming Language: Python 3.x
- IAM Security (Principle of Least Priviledge for AWS Core services)

## Set Up Instructions

1. Clone the Repository

       git clone https://github.com/olanuges90/containerized-flask.git
       cd containeirzed-flask/
   
2. Create ECR Repository

   In the CLI, run the following commands:

       aws configure 
       aws ecr create-repository --repository-name sports-api --region us-east-1

3. Authenticate Docker with ECR. Build, Tag and Push Image.

   In the CLI, run the following command:
   Replace <AWS_ACCOUNT> with your AWS ACCOUNT ID
   
       #Authenticate Docker with ECR
       aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com

       #Build and tag the Docker image
       docker build --platform linux/amd64 -t sports-api .
       docker tag sports-api:latest <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/sports-api:sports-api-latest

       #Push the Docker image
       docker push <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/sports-api:sports-api-latest
   
4. Set Up Amazon ECS (Fargate)
   - Create ECS Cluster:
     - In your AWS Management console, search for ECS, click  Clusters and create Cluster.
     - Provide a Cluster name, select AWS Fargate (serverless) for Infrastructure and create Cluster.
   - Define a Task Definition:
     - Navigate to Task Definitions, click Create New Task Definition.
     - Specify a unique task definition family name.
     - Select AWS Fargate as Launch type.
     - Specify a container name and Image URI (<AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/sports-api:sports-api-latest).
     - Type 8080 as Container port, select TCP as Protocol and HTTP as App Protocol.
     - Click Add Environment variables. Add Key as "SPORTS_API_KEY", Value as <YOUR_SERAPI_KEY>.
     - Then click Create.
       
5. Create an ECS service
     - Create Service:
       - Navigate to Clusters, click on Clusters and select the created Cluster.
       - Select Service and click Create, select the Task Definition Family created.
       - Assign a Service name.
       - Select "Replica" as Service type.
       - Choose "2" as Desired Task (We need 2 containers to handle traffic spike).
     - Network Configuration:
       - Create a New Security Group.
       - Select "All TCP" as Type and Source as "Anywhere".
    - Configure Load Balancing:
       - Load Balancing Type: Select Application Load Balancer.
       - Assign a name for Load balancer.
       - Health Check group: Type "/sports".
       - Then click Create.
         
  6. Verify Application Load Balancer
      - In the EC2 Dashboard, Navigate to Application Load Balance and select.
      - Verify the state of the Load Balancer as "Provisioning". scroll down to service and wait till deployment and Task is completed (2/2).
      - Click the ALB created, Navigate to the DNS Name.
      - Copy and Paste the DNS name in your browser to confirm the API is accessible and click Enter.
 
  7. Set Up API Gateway
    - Create API:
     - In the AWS management console, search for API Gateway and click.
     - Click Create API.
     - Navigate to REST API and click Build.
     - Assign a Name to your API and click Create API.
- Create a Resource:
     - Click Create Resource and Define Resource Path (/sports).
     - Click Create Resource.
- Create a Method:
     - Click Create Method.
     - Method Type: Select GET.
     - Integration Type: Select HTTP.
     - Toggle on "HTTP Proxy integration".
     - HTTP Method: Select GET.
     - Endpoint URL: Copy your ALB DNS Name and Paste. Type your Endpoint "/sports" behind the ALB URL.
     - Then Click Create Method.

  8. API Deployment
  - Click Deploy API.
  - Stage: Select New Stage.
  - Stage Name: Type Dev.
  - Then Deploy.
 
  9. Test the Deployment
  - Access the application via the API Gateway endpoint or ALB public URL.
    - Copy Invoke URL and Paste in your Browser.
    - Add your endpoint "/sports" behind the URL and click Enter.

           https://<api-gateway-id>.execute-api.us-east-1.amazonaws.com/prod/sports

  ## Future Enhancements
  - Built a frontend responsive design.
  - Add HTTPS support to the ELB using an SSL certificate from AWS Certificate Manager (ACM)
  - Configure CloudWatch for logging and monitoring.

         
 

         








