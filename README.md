# ToDo-Test02

A TODO list web application with Vue.js frontend, .NET Web API backend, SQLite database, and containerization setup for deployment on AWS with CICD.

## Project Structure
- **frontend1/**: First instance of Vue.js frontend for TODO list functionality.
- **frontend2/**: Second instance of Vue.js frontend for load balancing.
- **backend/**: .NET Web API for handling TODO operations.
- **database/**: SQLite database configuration.
- **docker/**: Docker configuration files for containerization.

## Setup Instructions
1. **Clone the Repository**: Clone this repository to your local machine.
2. **Docker Setup**: Navigate to the `docker` directory and run `docker-compose up --build` to start all containers.
3. **Access the Application**: Open your browser and go to `http://localhost` to access the TODO list application through the Nginx load balancer.

## AWS CICD Deployment Steps
To set up Continuous Integration and Continuous Deployment (CICD) for this project on AWS, follow these detailed steps:

### Prerequisites
- An AWS account with appropriate permissions.
- Docker installed on your local machine for building and pushing images.
- AWS CLI installed and configured with your credentials.
- A GitHub repository for this project.

### Step 1: Set Up AWS ECR (Elastic Container Registry)
1. **Create Repositories**: In the AWS Management Console, navigate to ECR and create four repositories named `todo-frontend1`, `todo-frontend2`, `todo-backend`, and `todo-nginx`.
2. **Authenticate Docker**: Run `aws ecr get-login-password --region <your-region> | docker login --username AWS --password-stdin <your-account-id>.dkr.ecr.<your-region>.amazonaws.com` to authenticate Docker with ECR.
3. **Build and Push Images**:
   - Navigate to each Dockerfile directory (`docker/frontend1`, `docker/frontend2`, `docker/backend`, `docker/nginx`).
   - Build and tag images: `docker build -t <your-account-id>.dkr.ecr.<your-region>.amazonaws.com/todo-<component>:latest .`
   - Push images: `docker push <your-account-id>.dkr.ecr.<your-region>.amazonaws.com/todo-<component>:latest`.

### Step 2: Set Up AWS ECS (Elastic Container Service)
1. **Create a Cluster**: In the ECS console, create a new cluster named `todo-cluster` using the Fargate launch type.
2. **Define Task Definitions**:
   - Create task definitions for `frontend1`, `frontend2`, `backend`, and `nginx` using the images pushed to ECR.
   - Configure networking to use the same VPC and subnets for all tasks.
   - For `backend`, map port 5000; for `frontend1` and `frontend2`, map ports 8080 and 8081 respectively; for `nginx`, map port 80.
   - Add environment variables or volumes as needed (e.g., for SQLite database persistence).
3. **Create Services**:
   - Create services for each task definition in the `todo-cluster`.
   - For `frontend1` and `frontend2`, set desired count to 1 each for load balancing.
   - For `nginx`, configure it to route traffic to `frontend1` and `frontend2` on ports 8080 and 8081, and to `backend` on port 5000 for API calls.

### Step 3: Set Up Application Load Balancer (ALB)
1. **Create an ALB**: In the EC2 console, create an Application Load Balancer named `todo-alb`.
2. **Configure Target Groups**:
   - Create target groups for `frontend1` (port 8080), `frontend2` (port 8081), and `backend` (port 5000).
   - Register the corresponding ECS service instances to these target groups.
3. **Set Up Listener Rules**:
   - Add a listener on port 80 to forward traffic to the `nginx` service or directly to frontend target groups if not using a separate Nginx container.

### Step 4: Set Up CodePipeline for CICD
1. **Create a Pipeline**: In the AWS CodePipeline console, create a new pipeline named `todo-pipeline`.
2. **Source Stage**: Connect to your GitHub repository (`ToDo-Test01`) using GitHub OAuth or a personal access token. Set the repository and branch to monitor for changes.
3. **Build Stage**: Use AWS CodeBuild to build Docker images.
   - Create a build project named `todo-build` with a buildspec.yml file (you'll need to add this to your repository root).
   - The buildspec should build and push Docker images to ECR for each component.
4. **Deploy Stage**: Use AWS ECS to deploy updated images.
   - Configure the deploy action to update the ECS services with the new images from ECR.

### Step 5: Configure buildspec.yml
Add a `buildspec.yml` file to your repository root with instructions for CodeBuild to build and push Docker images. Example content:
```yaml
version: 0.2
phases:
  pre_build:
    commands:
      - echo Logging in to Amazon ECR...
      - aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com
  build:
    commands:
      - echo Build started on `date`
      - echo Building the Docker images...
      - cd docker/frontend1 && docker build -t $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/todo-frontend1:latest ../../frontend1
      - cd ../frontend2 && docker build -t $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/todo-frontend2:latest ../../frontend2
      - cd ../backend && docker build -t $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/todo-backend:latest ../../backend
      - cd ../nginx && docker build -t $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/todo-nginx:latest .
  post_build:
    commands:
      - echo Build completed on `date`
      - echo Pushing the Docker images...
      - docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/todo-frontend1:latest
      - docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/todo-frontend2:latest
      - docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/todo-backend:latest
      - docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/todo-nginx:latest
```

### Step 6: Test and Monitor
1. **Test the Deployment**: After setting up the pipeline, manually trigger a release to ensure everything deploys correctly.
2. **Monitor**: Use AWS CloudWatch to monitor logs and metrics for your ECS services and ALB to ensure the application is running smoothly.

By following these steps, you will have a fully automated CICD pipeline for your TODO list application on AWS, ensuring seamless updates and deployments.
