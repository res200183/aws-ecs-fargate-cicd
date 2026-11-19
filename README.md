AWS ECS Fargate CI/CD Pipeline

A self-directed AWS DevOps project demonstrating automated testing, Docker image builds, container deployment, monitoring, and scaling using AWS managed services.

Project Overview

This project implements an automated CI/CD pipeline that deploys a Python Flask application to Amazon ECS using AWS Fargate.

Every push to the GitHub "main" branch triggers AWS CodePipeline, which runs CodeBuild to test the application, build a Docker image, push it to Amazon ECR, and deploy the updated image to ECS Fargate.

The project was built as a hands-on learning exercise to demonstrate practical AWS Cloud and DevOps engineering skills.

Architecture

Developer
    |
    v
GitHub Repository (main)
    |
    v
AWS CodePipeline
    |
    +-- Source Stage
    |      |
    |      v
    |   GitHub Connection
    |
    +-- Build Stage
    |      |
    |      v
    |   AWS CodeBuild
    |      |
    |      +-- Install dependencies
    |      +-- Run pytest
    |      +-- Build Docker image
    |      +-- Generate unique image tag
    |      +-- Push image to Amazon ECR
    |      +-- Generate imagedefinitions.json
    |
    +-- Deploy Stage
           |
           v
       Amazon ECS
           |
           v
       AWS Fargate
           |
           v
   Application Load Balancer
           |
           v
       Flask Application

Additional services:

- Amazon CloudWatch Logs for container logging
- ECS Service Auto Scaling for CPU-based scaling
- CloudWatch Alarms for scaling decisions
- ECS Deployment Circuit Breaker for failed-deployment detection and rollback
- AWS IAM for service permissions

Technologies Used

Technology| Purpose
AWS CodePipeline| CI/CD orchestration
AWS CodeBuild| Automated testing and Docker image builds
Amazon ECR| Docker image registry
Amazon ECS| Container orchestration
AWS Fargate| Serverless container compute
Application Load Balancer| HTTP traffic routing and health checks
Amazon CloudWatch| Logging and monitoring
AWS IAM| Access control
Docker| Application containerization
Python and Flask| Application development
pytest| Automated application testing
Git and GitHub| Version control and CI/CD triggers
AWS CLI| Infrastructure configuration and management

CI/CD Workflow

1. A developer pushes a commit to the GitHub "main" branch.
2. AWS CodePipeline detects the change through the GitHub connection.
3. AWS CodeBuild installs the required Python dependencies.
4. Automated tests run using pytest.
5. CodeBuild builds the Docker image.
6. The image receives a unique tag based on build information.
7. CodeBuild authenticates to Amazon ECR and pushes the image.
8. CodeBuild generates "imagedefinitions.json" as a deployment artifact.
9. The CodePipeline Deploy stage updates the ECS service.
10. ECS launches tasks using the updated container image.
11. The Application Load Balancer performs application health checks.

The pipeline stops at the build stage if the build or automated tests fail.

Application Endpoints

GET /

Returns:

{
  "message": "AWS ECS CI/CD works!"
}

GET /health

Returns:

{
  "status": "healthy"
}

The "/health" endpoint is used by the Application Load Balancer to check application availability.

Both endpoints were successfully tested through the deployed ALB and returned HTTP 200 responses.

Docker Configuration

The application is packaged into a Docker image using a Python slim base image.

The Docker configuration includes:

- Python dependency installation
- A non-root container user
- Gunicorn as the application server
- Application port "8080"
- Docker layer caching through separate dependency installation

Automated Testing

The project includes pytest tests for the Flask application.

Tests verify that:

- The main endpoint responds successfully.
- The health endpoint responds successfully.
- The endpoints return the expected JSON responses.

Tests are integrated into the CodeBuild workflow.

ECS Deployment and Reliability

The application runs as an ECS service using AWS Fargate.

Deployment features include:

- ECS task definitions
- ECS service management
- Application Load Balancer integration
- HTTP health checks
- Deployment Circuit Breaker
- Automatic rollback configuration

The ECS Deployment Circuit Breaker is configured to detect failed deployments and enable rollback.

Auto Scaling

ECS Service Auto Scaling uses a target-tracking policy based on average CPU utilization.

Configuration:

- Minimum tasks: "1"
- Maximum tasks: "2"
- Target CPU utilization: "60%"
- Scale-out cooldown: "60 seconds"
- Scale-in cooldown: "120 seconds"

AWS automatically creates and manages the CloudWatch alarms associated with this scaling policy.

Logging and Monitoring

Container logs are sent to Amazon CloudWatch Logs.

Log group:

/ecs/flask-ecs-app

Log retention:

7 days

Monitoring includes ECS service status, task health, ALB health checks, and CPU-based scaling metrics.

Security Considerations

The project uses:

- IAM roles for CodePipeline, CodeBuild, and ECS
- IAM permissions for deployment operations
- Security groups for network access control
- A non-root Docker container user
- Amazon ECR for storing container images
- Unique Docker image tags instead of relying exclusively on "latest"

IAM permissions and network configuration should be reviewed and further restricted before adapting this project for production workloads.

Repository Structure

aws-ecs-fargate-cicd/
├── app/
│   ├── __init__.py
│   └── main.py
├── tests/
│   └── test_main.py
├── Dockerfile
├── buildspec.yml
├── requirements.txt
├── .dockerignore
├── .gitignore
└── README.md

Running the Application Locally

Install dependencies:

pip install -r requirements.txt

Run automated tests:

python3 -m pytest -v

Build the Docker image:

docker build -t flask-ecs-app .

Run the container:

docker run --rm -p 8080:8080 flask-ecs-app

Test the health endpoint:

curl http://localhost:8080/health

Deployment Verification

The following checks were completed successfully:

- GitHub changes triggered the CI/CD pipeline.
- The Source stage succeeded.
- The Build stage succeeded.
- The Deploy stage succeeded.
- ECS service status was "ACTIVE".
- Desired task count matched the running task count.
- No tasks remained pending.
- The ALB health endpoint returned HTTP 200.
- The main application endpoint returned HTTP 200.
- ECS Auto Scaling policy registration was verified.
- ECS Deployment Circuit Breaker and rollback were enabled.

Challenges and Troubleshooting

During implementation, several issues were identified and resolved.

CodePipeline IAM permissions

The initial deployment failed because the CodePipeline service role lacked the required ECS permissions. Additional IAM permissions were configured to allow ECS deployment operations.

Container name mismatch

The deployment failed because the container name generated in "imagedefinitions.json" did not match the ECS task definition.

The build configuration was corrected to use the exact container name:

flask-ecs-app

IAM policy formatting

An IAM policy passed JSON syntax validation but failed AWS policy parsing. The policy document was corrected before applying it successfully.

These troubleshooting steps provided practical experience diagnosing CI/CD and AWS deployment failures.

Cost Management

This project uses AWS resources that may incur charges, including:

- ECS Fargate tasks
- Application Load Balancer
- EC2 development instance
- CloudWatch Logs and other AWS resources

Resources should be stopped or deleted when not required.

Stopping the EC2 development instance does not automatically stop ECS Fargate tasks or remove the Application Load Balancer.

Future Improvements

Potential enhancements include:

- Infrastructure as Code using AWS CloudFormation or Terraform
- Deployment to private subnets with improved network isolation
- HTTPS using AWS Certificate Manager
- More restrictive IAM permissions
- Additional integration tests
- Automated security scanning
- Blue/green deployment strategies

These are future improvements and are not part of the current implementation.

Project Type

Personal, self-directed hands-on DevOps project.

Created to develop and demonstrate practical experience with AWS, Docker, CI/CD automation, container orchestration, monitoring, troubleshooting, and cloud infrastructure management.
