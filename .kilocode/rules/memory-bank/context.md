# Context

## Current State
Java Spring Boot scaffolder template deploys to AWS ECS Fargate via ECR. Shared AWS infrastructure deployed via CloudFormation (stack `backstage-shared-infra` in us-east-2). GitHub Actions plugin wired into CI/CD tab. Code review completed and fixes applied. CloudFormation stack needs update to add ecsTaskExecutionRole (was missing, causing deploy failures).

## What Was Done
1. Created `examples/spring-boot-template/template.yaml` - Scaffolder template definition with wizard form (name, description, owner, javaPackage, springBootVersion, repoUrl)
2. Created template content skeleton under `examples/spring-boot-template/content/`:
   - `catalog-info.yaml` - Backstage catalog descriptor with `github.com/project-slug` annotation
   - `build.gradle` - Spring Boot + Gradle build with Java 21 toolchain
   - `settings.gradle` - Gradle settings
   - `gradle/wrapper/gradle-wrapper.properties` - Gradle 8.13 wrapper config
   - `Dockerfile` - Multi-stage build (Gradle build -> Eclipse Temurin 21 JRE)
   - `.gitignore` - Java/Gradle gitignore
   - `README.md` - Project documentation (updated with AWS deployment info)
   - `.github/workflows/ci-cd.yml` - GitHub Actions pipeline (build, test, deploy to AWS ECR + ECS Fargate)
   - `aws/task-definition.json` - ECS Fargate task definition template
   - `src/main/java/${{values.javaPackageDir}}/Application.java` - Spring Boot main class
   - `src/main/resources/application.yml` - Spring config with actuator
   - `src/test/java/${{values.javaPackageDir}}/ApplicationTests.java` - Context load test
3. Added Spring Boot template location to `app-config.yaml` and `app-config.production.yaml` catalog locations
4. Installed `@backstage-community/plugin-github-actions` (v0.21.1) in frontend package
5. Updated `EntityPage.tsx` - Replaced CI/CD placeholder with GitHub Actions plugin
6. Created `aws/shared-infrastructure.yml` - CloudFormation template for shared infra (VPC, subnets, IGW, ECS cluster, security group, IAM role, CloudWatch log group)

## AWS Deployment Architecture
- Shared infra: CloudFormation stack `backstage-shared-infra` in us-east-2
  - VPC (10.0.0.0/16) with 2 public subnets
  - ECS cluster `backstage-apps`
  - Security group allowing TCP 8080 inbound
  - IAM `ecsTaskExecutionRole` with ECR pull + CloudWatch logs permissions
  - CloudWatch log group `/ecs/backstage-apps`
- Per-service: CI/CD creates ECR repo, registers task definition, creates/updates ECS service
- No ALB -- Fargate tasks get public IPs directly (demo tradeoff: IP changes on redeploy)
- Required GitHub secrets: AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY

## What Does Not Exist Yet
- gradlew / gradlew.bat wrapper scripts (not needed for CI since setup-gradle uses gradle-version: '8.13')
- Kubernetes cluster configuration (separate from ECS deployment)
- Proper RBAC (still using allow-all policy)
- Production catalog sources beyond examples
- End-to-end testing of the full scaffolder flow

## Infrastructure Status
- AWS shared infrastructure: CloudFormation stack `backstage-shared-infra` deployed in us-east-2
- IAM user created with ECR + ECS permissions
- GitHub org secrets configured: AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY (visibility: Public repositories only)
- GitHub fine-grained PAT (ca-backstage) created with admin, code, workflows permissions on all org repos
- ECR repos created automatically by CI/CD (comptest6, comptest7 exist)
- ECS cluster `backstage-apps` exists (created by CloudFormation stack)

## Next Steps
1. Update CloudFormation stack to add ecsTaskExecutionRole: `aws cloudformation update-stack --stack-name backstage-shared-infra --template-body file://aws/shared-infrastructure.yml --capabilities CAPABILITY_NAMED_IAM --region us-east-2`
2. Wait for stack update: `aws cloudformation wait stack-update-complete --stack-name backstage-shared-infra --region us-east-2`
3. Re-run comptest9 CI/CD (or scaffold a new repo) to verify deploy succeeds end-to-end
4. Verify: task def registered -> service created -> task running -> public IP accessible
5. Check if CI/CD IAM user needs iam:PassRole permission (may surface as next error)

## Deploy Command
```
aws cloudformation create-stack \
  --stack-name backstage-shared-infra \
  --template-body file://aws/shared-infrastructure.yml \
  --capabilities CAPABILITY_NAMED_IAM \
  --region us-east-2
```
Wait for completion:
```
aws cloudformation wait stack-create-complete --stack-name backstage-shared-infra --region us-east-2
```
Verify outputs:
```
aws cloudformation describe-stacks --stack-name backstage-shared-infra --region us-east-2 --query "Stacks[0].Outputs"
```

## Recent Changes
- Fixed comptest9 deploy failure: ecsTaskExecutionRole did not exist. Added IAM role to CloudFormation template (aws/shared-infrastructure.yml) with ecs-tasks.amazonaws.com trust policy and AmazonECSTaskExecutionRolePolicy managed policy. Stack update required.
- Code review: fixed ECS health check (curl -> wget for Alpine compatibility in task-definition.json)
- Code review: eliminated double task definition registration (replaced amazon-ecs-deploy-task-definition action with CLI update-service using the already-registered ARN)
- Code review: disabled plain JAR in build.gradle so Dockerfile COPY glob only picks up the fat JAR
- Code review: notification in template.yaml now sends to actual owner instead of hardcoded guest
- Code review: removed redundant HealthController.java (Actuator /actuator/health already exists)
- Code review: extracted GitHub OAuth clientId/clientSecret to env vars in app-config.yaml
- Code review: disabled guest auth in app-config.production.yaml
- Fixed comptest7 deploy failure: "TaskDefinition not found" when creating ECS service
- Fixed: Added `repoVisibility: public` to template.yaml publish:github step
- Created `aws/shared-infrastructure.yml` CloudFormation template for shared AWS infrastructure
- Replaced CI/CD workflow: GHCR push -> ECR push + ECS Fargate deploy
