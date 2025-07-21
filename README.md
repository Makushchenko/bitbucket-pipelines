# bitbucket-pipelines

Bitbucket CI/CD projects for deploying Dockerized applications to AWS ECS (and DockerHub for Node.js).

## Prerequisites

- Bitbucket Pipelines enabled in your repository.  
- An AWS ECR registry to store built images.  
- Repository environment variables (in **Settings → Pipelines → Repository variables**):
  - `AWS_ACCESS_KEY_ID`
  - `AWS_SECRET_ACCESS_KEY`
  - `AWS_DEFAULT_REGION`
  - `AWS_REGISTRY_URL` (ECR login URI, e.g. `123456789012.dkr.ecr.eu-central-1.amazonaws.com/my-repo`)
- For DockerHub example:
  - `DOCKERHUB_USERNAME`
  - `DOCKERHUB_PASSWORD`
- `envsubst` (installed via `pip3 install envsubst` in pipelines when needed).

## Directory Structure

### `httpd-aws-ecs-deploy`

Contains a multi-branch pipeline for building and deploying an Apache HTTP Server container to AWS ECS:

- **`bitbucket-pipelines.yml`**  
  Defines pipelines for branches `feature/**`, `bugfix/dev/*`, `master`, `qa-master`, `dev-master`, and `sandbox-master`.  
  Each pipeline step:
  1. Installs AWS CLI and configures credentials.  
  2. Logs in to ECR, builds, tags (`${AWS_REGISTRY_URL}:<branch>-<BUILD_ID>`), and pushes the HTTPD image.  
  3. Uses the Atlassian `aws-ecs-deploy` pipe to update the ECS service with a rendered task definition.

- **`Dockerfile`**  
  Bases on `httpd:latest` and exposes port 80.

- **`task-definition-template.json`**  
  An AWS Fargate task definition template. Replace `${IMAGE_NAME}`, `${AWS_EXECUTION_TASK_ROLE}`, and `${AWS_TASK_ROLE}` at deploy time.

---

### `nginx-aws-ecs-deploy`

Provides a simple `main`-branch pipeline for an NGINX container:

- **`bitbucket-pipelines.yml`**  
  For `main` branch: installs AWS CLI, logs in to ECR, builds and pushes the `nginx:latest`-based image, then deploys via the `aws-ecs-deploy` pipe.

- **`Dockerfile`**  
  Builds from `nginx:latest` and exposes port 80.

- **`task-definition-template.json`**  
  Fargate task definition with placeholders for `${IMAGE_NAME}` and execution role ARN.

---

### `node-aws-ecs-deploy`

Demonstrates a Node.js application pipeline with DockerHub integration:

- **`bitbucket-pipelines.yml`**  
  Default pipeline:
  1. Builds the Node.js image (`${DOCKERHUB_USERNAME}/${BITBUCKET_REPO_SLUG}:${BUILD_NUMBER}`)  
  2. Logs in to DockerHub and pushes the image  
  3. Renders and deploys an ECS task via `aws-ecs-deploy` pipe.

- **`Dockerfile`**  
  - Uses `node:latest`
  - Installs dependencies from `package.json`
  - Copies `src/`
  - Exposes port 3000 and runs `npm start`.

- **`package.json`** & **`src/app.js`**  
  A simple Express.js “Hello World!” server listening on port 3000.

- **`task-definition-template.json`**  
  Fargate task definition for the Node.js app, with `${IMAGE_NAME}`

---

With this guide in place, anyone browsing the repo can quickly see what each pipeline does, which files live in each directory, and how to configure Bitbucket and AWS/DockerHub to get started.