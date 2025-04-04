# Deploying a Containerized Flask Application on AWS with Docker, ECS, API Gateway, and ELB, Automated using GitHub Actions for CI/CD and Monitored Using CloudWatch Metrics.


## An Enhancement to https://github.com/olanuges90/containerized-flask

## Project Overview

This project focuses on containerizing a Python Flask Application for Sports API management using Docker and deploying it on AWS with a robust, scalable architecture. The deployment leverages Amazon ECS (Fargate) for container orchestration, an Elastic Load Balancer (ELB) for distributing traffic across multiple containers, and API Gateway for exposing the application’s REST endpoints securely. This setup ensures high availability, scalability, and efficient handling of incoming requests, making the application production-ready.

## Architectural Diagram
![cicd drawio](https://github.com/user-attachments/assets/70c6bdfc-03ee-4d11-9234-03a41b289d29)


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

  ## PROJECT ENHANCEMENT
1. IAM Permissions: Before I began the enhancement, I updated the IAM permissions.
  -   EC2ContainerRegistryFullAccess: Required in CI/CD pipelines where applications are built, pushed, and deployed using Docker images stored in ECR.
  -   GitHub-Actions-ECS-Deployment-Role: Required for CI/CD pipeline to interact with ECS, ECR, and other AWS services.
    
2.  Configure CloudWatch for monitoring
   - In the AWS Management Console, Search for Cloudwatch and Click.
   - Navigate to Alarms in the left panel and Select.
   - Click Create Alarm and Select Metric.
   - In the AWS namespaces, Select ECS and Click "ClusterName, ServiceName".
   - Highlight the Cluster name and Select the Metric.
   - Make the threshold type "Static" and Define the alarm condition as "Greater than Threshold (>)".
   - Define the threshold value.
   - Define the alarm state that will trigger this action.
   - Select Create a New Topic.
   - Provide a Unique Topic Name.
   - Provide Email endpoints that will receive the notification and Create Topic.
   - Enter an Alarm Name and Create Alarm.
   - Go to the Email provided as the endpoint for notification and Subscribe to the Notification.

     <img width="764" alt="Screenshot 2025-03-29 at 2 57 00 AM" src="https://github.com/user-attachments/assets/1e28fd1d-20d9-440f-a8c1-09ea097df0e5" />
     <img width="1147" alt="Screenshot 2025-03-27 at 9 57 19 AM" src="https://github.com/user-attachments/assets/1f462f96-bdc2-46e3-92de-4322cc4226ac" />
       
 3. GitHub Action for Continous Integration and Continous Delivery.
    - Create the Workflow directory in the Repo.

             .github/workflows/
    - Inside the directory, create the workflow file to define the workflow.

              deploy.yml
    - Add Github Secrets:

       - In the Github Repository. Nagivate to Settings.
       - Click on Actions, Secrets and Variables. Click the New Rpository secret button.
       - Enter a Value and Pass the secrets: AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY AWS_REGION, ECR_REPOSITORY, ECR_CLUSTER and ECR_SERVICE.
   - Commit and Push.

        <img width="1439" alt="Screenshot 2025-03-29 at 5 16 24 AM" src="https://github.com/user-attachments/assets/65a75416-5a4f-470f-9134-dc1860748a8a" />


 
        <img width="983" alt="Screenshot 2025-03-27 at 10 01 00 AM" src="https://github.com/user-attachments/assets/a048df34-3010-4d0f-a0f1-dfb819901684" />


        <img width="982" alt="Screenshot 2025-03-27 at 10 05 06 AM" src="https://github.com/user-attachments/assets/73dd9b8a-3959-4089-9269-66a47fd76400" />

   4. Destroy Worflow.
      - Navigate the the workflow file in your repo and look for the steps which defines all the actions in the worflow.
      - Remove or comment out.
      - Commit Changes and Push.
  ## NOTE: I also used Terraform  Deploy the Entire Application and to Keep it simple, I have added the Terraform Configuration in another Repository. 






         
 

         








