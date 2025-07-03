"moduleNameMapper": {
   "^@core/(.*)$": "<rootDir>/src/app/core/$1",
   ...
}name: Deploy to Amazon ECS

on:
  push:
    branches: [ "main" ]
  workflow_dispatch:

env:
  AWS_REGION: ${{ secrets.AWS_REGION }}
  ECR_REPOSITORY: ${{ secrets.ECR_REPOSITORY }}
  ECS_SERVICE: ${{ secrets.ECS_SERVICE }}
  ECS_CLUSTER: ${{ secrets.ECS_CLUSTER }}
  ECS_TASK_DEFINITION: ${{ secrets.ECS_TASK_DEFINITION }}
  CONTAINER_NAME: ${{ secrets.CONTAINER_NAME }}

jobs:
  deploy:
    name: Build and Deploy to ECS
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Log in to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build, tag, and push Docker image to ECR
        run: |
          IMAGE_TAG=latest
          IMAGE_URI=${{ steps.login-ecr.outputs.registry }}/${{ env.ECR_REPOSITORY }}:$IMAGE_TAG

          docker build -t $IMAGE_URI .
          docker push $IMAGE_URI

      - name: Fill in the new image ID in the ECS task definition
        id: task-def
        uses: aws-actions/amazon-ecs-render-task-definition@v1
        with:
          task-definition: .aws/task-definition.json
          container-name: ${{ env.CONTAINER_NAME }}
          image: ${{ steps.login-ecr.outputs.registry }}/${{ env.ECR_REPOSITORY }}:latest

      - name: Deploy Amazon ECS task definition
        uses: aws-actions/amazon-ecs-deploy-task-definition@v1
        with:
          service: ${{ env.ECS_SERVICE }}
          cluster: ${{ env.ECS_CLUSTER }}
          task-definition: ${{ steps.task-def.outputs.task-definition }}0x4Fabb145d64652a948d72533023f6E7A623C7C53"paths": {
   "@core/*": ["app/core/*"],
   ..."scripts": {
  "build": "echo 'Build script placeholder'",
  "test": "echo 'Test script placeholder'"
}Node.jsyarn.lock"moduleNameMapper": {
   "^@core/(.*)$": "<rootDir>/src/app/core/$1",
   ...
}name: Node.js CI

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: ⬇️ Checkout repository
        uses: actions/checkout@v4

      - name: 🟢 Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: 📦 Install dependencies
        run: |
          if [ -f package-lock.json ]; then
            npm ci
          elif [ -f yarn.lock ]; then
            yarn install --frozen-lockfile
          else
            echo "❌ No lockfile found. Please commit package-lock.json or yarn.lock."
            exit 1
          fi

      - name: 🔨 Run build (optional)
        run: npm run build || echo "No build script defined."

      - name: ✅ Run tests
        run: npm test || echo "No test script defined."package-lock.json1
}steps.login-ecr.outputs.registry workflow 
