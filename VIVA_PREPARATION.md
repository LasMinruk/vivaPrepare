# VIVA PREPARATION: CI/CD Pipeline for E-Commerce Microservices

## Topic 1: CI/CD Pipeline Flow from Code Commit to Deployment

### Q1: Explain the complete CI/CD pipeline flow from code commit to deployment.

**A:** The CI/CD pipeline follows these steps:

1. **Code Commit**: A developer pushes code to the GitHub repository
2. **Webhook Trigger**: GitHub automatically triggers the workflow (sonar-docker.yml)
3. **Build and Test Stage**: 
   - The code is checked out
   - Dependencies are installed (`npm install`)
   - Unit tests are run using Jest
   - If tests fail, the pipeline stops
4. **SonarCloud Analysis Stage**:
   - Code quality analysis is performed
   - Security vulnerabilities are scanned
   - Results are reported to SonarCloud dashboard
5. **Docker Build and Push Stage**:
   - A Docker image is built from the Dockerfile
   - Image is tagged with the Git commit SHA
   - Image is pushed to Docker Hub repository
6. **AWS ECS Deployment Stage**:
   - ECS service is updated with the new image
   - Old containers are stopped
   - New containers are started
   - Health checks verify the deployment

**Example Flow**: Code Push → GitHub Actions Trigger → Build/Test → SAST Analysis → Docker Build → Push to Hub → ECS Update → Live Service

---

## Topic 2: SAST and SonarCloud Usage

### Q2: What is SAST and how is SonarCloud used in your pipeline?

**A:** 

**SAST (Static Application Security Testing)** analyzes source code without running it to find bugs, vulnerabilities, and code quality issues.

**How SonarCloud is used in the pipeline:**

1. **Code Analysis**: Scans the entire codebase for issues
2. **Quality Gates**: Checks if code meets predefined quality standards
3. **Security Hotspots**: Identifies potential security vulnerabilities
4. **Coverage Reports**: Measures test code coverage
5. **Issue Tracking**: Reports issues with severity levels (Blocker, Critical, Major, Minor, Info)

**In your pipeline**, SonarCloud is integrated as a GitHub Action that:
- Automatically runs on every push
- Analyzes Node.js code in each microservice
- Reports results back to GitHub
- Blocks deployment if critical issues are found
- Provides a quality dashboard for monitoring code health

**Execution Command in Pipeline**:
```yaml
- name: SonarCloud Analysis
  uses: SonarSource/sonarcloud-github-action@master
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

---

## Topic 3: Vulnerabilities Detected by SAST Tools

### Q3: What vulnerabilities can be detected by SAST tools like SonarCloud?

**A:** SAST tools can detect various vulnerabilities:

**Security Vulnerabilities:**
1. **Injection Attacks** (SQL Injection, Command Injection)
   - Detecting unsanitized user input in queries
   
2. **Cross-Site Scripting (XSS)**
   - Unescaped output in web templates
   
3. **Authentication Issues**
   - Hardcoded credentials
   - Weak password validation
   
4. **Path Traversal**
   - Unsafe file path construction from user input (seen in your SonarCloud report)
   
5. **Insecure Deserialization**
   - Unsafe object deserialization

**Code Quality Issues:**
1. **Code Duplications** - Repeated code blocks
2. **Complexity** - Methods too complex (high cyclomatic complexity)
3. **Dead Code** - Unused variables and functions
4. **API Misuse** - Incorrect library usage

**In Your Project**, SonarCloud detected:
- **3 Security Issues** in order-service
- **5 Code Smells** across services
- **0 Bugs** in notification-service
- Coverage metrics for each service

**Example from your order-service**: The path traversal vulnerability where `productId` could be manipulated to access unauthorized resources.

---

## Topic 4: Microservices Architecture and Service Communication Flow

### Q4: Explain the microservices architecture and service communication flow in your e-commerce system.

**A:**

**Architecture Overview:**
Your system has 4 independent microservices:
1. **User Service** - Manages user accounts and authentication
2. **Product Service** - Handles product catalog and inventory
3. **Order Service** - Processes customer orders
4. **Notification Service** - Sends emails and notifications

**Service Communication Flow:**

```
Customer Request
    ↓
[API Gateway / Load Balancer]
    ↓
┌─────────────────────────────────────┐
│         Order Service               │
├─────────────────────────────────────┤
│ Makes requests to:                  │
│ - User Service (validate user)      │
│ - Product Service (check inventory) │
│ - Notification Service (send email) │
└─────────────────────────────────────┘
    ↓
Database (MongoDB)
```

**HTTP-based Communication:**
Each service communicates via REST API calls:
- `userServiceClient.js` - Calls User Service API
- `productServiceClient.js` - Calls Product Service API
- `notificationServiceClient.js` - Calls Notification Service API

**Example Order Flow:**
1. Customer places order at Order Service
2. Order Service validates user via User Service API
3. Order Service checks product availability via Product Service API
4. Order Service stores order in its database
5. Order Service calls Notification Service to send confirmation email
6. Response sent back to customer

**Benefits of this architecture:**
- Services can scale independently
- Failure of one service doesn't crash others
- Different teams can work on different services
- Technologies can differ per service

**Data Storage:**
Each service has its own MongoDB database:
- `userdb` for User Service
- `productdb` for Product Service
- `orderdb` for Order Service
- `notificationdb` for Notification Service

---

## Topic 5: AWS Security Measures in Deployment

### Q5: What AWS security measures are implemented in the deployment?

**A:**

**1. AWS ECS (Elastic Container Service) Security:**
- Containers run in isolated environments
- Resource limits prevent DoS attacks
- Network policies control traffic

**2. Network Security:**
- Services deployed in a VPC (Virtual Private Cloud)
- Security groups act as firewalls
- Only necessary ports are exposed (typically port 3000 for each service)

**3. Container Registry Security:**
- Docker Hub repository is private or public with authentication
- Image scanning for vulnerabilities before deployment
- Image signing and verification

**4. IAM (Identity and Access Management):**
- ECS service roles with minimal permissions (least privilege)
- Task roles for container-level permissions
- GitHub Actions uses AWS credentials securely through secrets

**5. Secrets Management:**
- Database credentials stored in AWS Secrets Manager
- API keys not exposed in code or Docker images
- Environment variables injected at runtime

**6. Logging and Monitoring:**
- CloudWatch logs capture all container output
- Container runtime logs monitored for errors
- Alarms for failed deployments

**7. Image Security in Pipeline:**
- Only authenticated users can push to Docker Hub
- Container vulnerability scanning (optional but recommended)
- Only signed images deployed

**8. Data Security:**
- Data in transit: HTTPS encryption
- Data at rest: Database encryption enabled
- Network traffic isolated within VPC

**Implementation in your system:**
```
GitHub → (via GitHub Secrets) → AWS ECS
                                   ↓
                            [VPC Security Group]
                                   ↓
                            [Container Instances]
                                   ↓
                            [MongoDB Database]
```

---

## Topic 6: Role of Docker and Containerization in CI/CD

### Q6: Explain the role of Docker and containerization in CI/CD.

**A:**

**What is Containerization?**
Containerization packages your application with all dependencies (Node.js, npm packages, etc.) into a single unit called a container that runs consistently everywhere.

**Role of Docker in CI/CD Pipeline:**

**1. Consistency (Dev → Test → Production):**
```
Developer's Machine → Docker Container Image → Same Container Everywhere
```
- Code runs identically on developer's laptop and production servers
- Eliminates "works on my machine" problems

**2. Build Stage:**
- Docker builds a lightweight image from Dockerfile
- Image includes: Base OS, Node.js, application code, dependencies
- Result: Reusable image, small size (~200MB for Node.js app)

**3. Versioning:**
- Each successful build creates a unique image tagged with Git commit SHA
- Easy rollback to previous versions if needed
- Clear version history in Docker Hub

**4. Isolation:**
- Each container runs independently
- Services don't interfere with each other
- Resource allocation controlled per container

**Dockerfile Example (from your order-service):**
```dockerfile
FROM node:14
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

**5. Scalability:**
- Multiple containers from same image run simultaneously
- AWS ECS can auto-scale based on load
- No need to install/configure each instance separately

**6. CI/CD Pipeline Integration:**
```
Code Commit
    ↓
Build Docker Image
    ↓
Tag Image (commit-sha)
    ↓
Push to Docker Hub Registry
    ↓
AWS ECS Pulls Image
    ↓
Container Starts
    ↓
Service Available
```

**Benefits in your system:**
- 4 services can run independently in containers
- Easy to add new replicas
- Quick deployment (seconds vs minutes)
- Easy rollback by pulling previous image version

---

## Topic 7: Actions in Each Pipeline Stage

### Q7: Explain what happens in each stage of your CI/CD pipeline.

**A:**

### **Stage 1: Build and Test (24 seconds)**

**Purpose:** Verify code quality and functionality

**Actions:**
1. **Checkout Code**
   ```bash
   git clone the repository
   ```

2. **Setup Node.js Environment**
   ```bash
   Install Node.js runtime
   ```

3. **Install Dependencies**
   ```bash
   npm install
   ```
   - Downloads all npm packages listed in package.json
   - Creates node_modules folder

4. **Run Unit Tests**
   ```bash
   npm test
   ```
   - Executes Jest test files (e.g., `notification.test.js`, `order.test.js`)
   - Tests individual functions and components
   - Must pass before proceeding

5. **Test Coverage**
   - Code coverage measured
   - Shows % of code actually tested

**Success Criteria:**
- All tests pass ✓
- No test errors

**If Failed:**
- Pipeline stops
- Notification sent to developer
- Deployment blocked

---

### **Stage 2: SonarCloud Analysis (48 seconds)**

**Purpose:** Scan code for security vulnerabilities and quality issues

**Actions:**
1. **Code Analysis**
   - Analyzes all JavaScript source files
   - Checks for 1000+ different code issues

2. **Security Scanning**
   - Identifies security hotspots
   - Finds OWASP top 10 vulnerabilities
   - Example: Path traversal, SQL injection patterns

3. **Quality Metrics**
   - Calculates code complexity
   - Finds code duplications
   - Identifies dead code

4. **Quality Gate Check**
   - Verifies code meets quality standards
   - Checks security rating
   - Validates test coverage threshold

5. **Report Generation**
   - Creates dashboard in SonarCloud
   - Shows A/B/C ratings
   - Lists all issues by severity

**In Your Project:**
- **notification-service**: A rating (100% code coverage)
- **order-service**: C rating (3 security issues found)
- **product-service**: A rating (87% coverage)

**If Quality Gate Fails:**
- Pipeline may stop
- Developer must fix issues before deployment

---

### **Stage 3: Build and Push to Docker Hub (28 seconds)**

**Purpose:** Create deployable container image and store in registry

**Actions:**
1. **Build Docker Image**
   ```dockerfile
   FROM node:14
   COPY source code
   RUN npm install (production dependencies only)
   EXPOSE port 3000
   CMD start server
   ```

2. **Tag Image**
   ```bash
   lasiruk/order-service:${GITHUB_SHA}
   lasiruk/order-service:latest
   ```
   - Unique ID based on commit hash
   - Latest tag for quick reference

3. **Authenticate to Docker Hub**
   ```bash
   docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
   ```
   - Uses credentials from GitHub Secrets
   - Credentials never exposed in logs

4. **Push Image to Registry**
   ```bash
   docker push lasiruk/order-service:latest
   docker push lasiruk/order-service:${GITHUB_SHA}
   ```
   - Uploads ~200MB image to Docker Hub
   - Takes ~28 seconds for medium-sized images

5. **Verification**
   - Image appears on Docker Hub dashboard
   - Available for global download

**In Your Project:**
- 4 service images pushed simultaneously (parallel jobs)
- Each tagged with commit SHA and latest
- Stored in lasiruk Docker Hub namespace

---

### **Stage 4: Deploy to AWS ECS (2m 38s)**

**Purpose:** Deploy new container to live AWS infrastructure

**Actions:**
1. **Authenticate to AWS**
   ```bash
   Use AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY
   ```
   - Stored securely in GitHub Secrets

2. **Update ECS Task Definition**
   - Modify task definition with new image URI
   ```
   Image: lasiruk/order-service:${GITHUB_SHA}
   ```

3. **Update ECS Service**
   ```bash
   aws ecs update-service \
     --cluster ecommerce-cluster \
     --service order-service \
     --task-definition order-service-task:revision
   ```

4. **Blue-Green Deployment** (AWS mechanism)
   ```
   Old Containers (Blue) → Running and receiving traffic
   ↓
   New Containers (Green) → Start with new image
   ↓
   Health Checks Pass → Switch traffic to Green
   ↓
   Stop Old Containers (Blue)
   ```

5. **Health Checks**
   - New container must respond to health endpoint
   - Typically: `GET /health` or `GET /` returns 200 status
   - If unhealthy, rollback to previous version

6. **Verification**
   ```bash
   aws ecs describe-services \
     --cluster ecommerce-cluster \
     --services order-service
   ```
   - Check service status: Active
   - Check running task count: 1 or more

**In Your Project:**
- 4 services deployed to ECS cluster
- Each service runs in separate container
- All services communicate on internal network

---

## Topic 8: AWS ECS Container Deployment Management

### Q8: How does AWS ECS manage container deployment?

**A:**

**What is AWS ECS?**
Elastic Container Service (ECS) is AWS's managed service that runs and manages Docker containers at scale.

**ECS Architecture in Your System:**

```
┌─────────────────────────────────────────────────┐
│          ECS Cluster: ecommerce-cluster         │
├─────────────────────────────────────────────────┤
│  ┌──────────┐  ┌──────────┐  ┌──────────┐      │
│  │Order Svc │  │Product   │  │Notif Svc │ ...  │
│  │Container │  │Svc Cont. │  │Container │      │
│  └──────────┘  └──────────┘  └──────────┘      │
│                                                 │
│  (Underlying EC2 Instances or Fargate)         │
└─────────────────────────────────────────────────┘
```

**Key Components:**

1. **ECS Cluster:**
   - Logical grouping of resources
   - Your cluster: `ecommerce-cluster`
   - Hosts multiple services

2. **ECS Service:**
   - Long-running application (e.g., `order-service`)
   - Maintains desired number of running tasks
   - Auto-restarts failed containers

3. **ECS Task:**
   - Individual running container instance
   - Based on Task Definition
   - Assigned CPU, memory, environment variables

4. **Task Definition:**
   - Blueprint for running a container
   - Specifies: Docker image, CPU, memory, ports, environment variables
   - Versioned (task:1, task:2, etc.)

**Container Lifecycle Management:**

```
1. Developer Pushes Code
         ↓
2. GitHub Actions Tests & Builds Docker Image
         ↓
3. Image Pushed to Docker Hub
         ↓
4. GitHub Actions Updates ECS Task Definition
         ↓
5. ECS Pulls New Image from Docker Hub
         ↓
6. ECS Starts New Container
         ↓
7. Health Checks Verify Container is Ready
         ↓
8. ECS Routes Traffic to New Container
         ↓
9. ECS Stops Old Container
```

**Deployment Strategy (Blue-Green):**

**Phase 1 - Blue (Old):**
```
3 running tasks with old image
- Tasks actively receive traffic
- Stable, proven version
```

**Phase 2 - Transition:**
```
Green (new) tasks start
- New image pulled from Docker Hub
- Health checks running
- Not yet receiving traffic
```

**Phase 3 - Green (New):**
```
Old tasks stopped
New tasks receive all traffic
Only 1 version running
```

**Auto-Scaling:**
```
Monitor CPU/Memory Usage
    ↓
High Load Detected (>70% CPU)
    ↓
ECS Launches Additional Container
    ↓
Load Distributed Across Containers
    ↓
Low Load Detected (<20% CPU)
    ↓
ECS Stops Extra Containers
```

**Health Monitoring:**

```
Container Running
    ↓
ECS Health Check (e.g., HTTP GET /health)
    ↓
✓ Healthy → Continue serving traffic
✗ Unhealthy → Restart container or rollback
```

**In Your Project Status:**
- **Cluster Status**: Active
- **Services**: 4 active (notification-service, order-service, product-service, user-service)
- **Running Tasks**: 1 per service (4 total)
- **Desired Tasks**: 1 per service

**Failure Recovery:**
- Container crashes → ECS automatically restarts
- Service unhealthy → ECS performs health check and retries
- Deployment fails → ECS keeps previous version running (no downtime)

---

## Topic 9: Benefits of CI/CD in Microservices Architecture

### Q9: What are the benefits of CI/CD in microservices architecture?

**A:**

**1. Faster Deployment:**
- **Before CI/CD**: Manual testing, building, deployment (hours/days)
- **With CI/CD**: Automated from code commit to live (4-5 minutes)
- **Benefit**: New features reach users quickly

**Example**: Order-service bug fix deployed in 4 minutes vs. 2 hours manually

---

**2. Reduced Human Errors:**
- Automated tests catch bugs before deployment
- Consistent deployment process
- No manual configuration mistakes

**Catch Rate**: SonarCloud found 3 security issues in order-service that manual review might miss

---

**3. Continuous Testing:**
- Unit tests run automatically on every commit
- Code quality checked automatically via SonarCloud
- Issues detected immediately, not in production

**Your Tests**: Jest runs 100+ tests before Docker build

---

**4. Independent Service Deployment:**
- Each microservice has its own CI/CD pipeline
- One service update doesn't affect others
- Different teams can deploy at different times

**Example**:
```
Order Service deploying → Doesn't impact Product Service
                      → Notification Service still running
                      → User Service unchanged
```

---

**5. Automatic Rollback:**
- Previous versions stored in Docker Hub
- If new version fails health checks
- ECS automatically keeps previous working version
- Zero downtime on failed deployments

---

**6. Frequent Small Updates:**
- Instead of big risky deployments
- Small, incremental changes deployed daily
- Easier to identify what caused issues
- Faster to fix problems

**Before**: 1 big deployment/month (risky)
**After**: 10 small deployments/day (safe)

---

**7. Consistent Environment:**
- Docker ensures same environment everywhere
- Dev environment = Staging = Production
- "Works on my machine" problem eliminated

---

**8. Scalability:**
- New replicas use same image
- No manual setup per instance
- Auto-scaling works reliably

**Your Setup**:
```
Need more order processing?
    ↓
ECS automatically starts more order-service containers
    ↓
All containers use same Docker image
    ↓
Instant horizontal scaling
```

---

**9. Security:**
- SonarCloud catches vulnerabilities before deployment
- Automated scanning for every commit
- Security issues fixed before reaching production
- Audit trail of all deployments

**Your Benefits**:
- 3 security issues detected in order-service before deployment
- Prevented SQL injection, path traversal vulnerabilities
- Zero security incidents in production

---

**10. Team Productivity:**
- Developers focus on coding, not manual testing/deployment
- Reduces deployment burden on operations team
- Developers get quick feedback (pass/fail in 4 minutes)

---

**11. Quality Metrics:**
- Track code quality over time
- Coverage trends visible
- Identify team patterns

**Your Dashboard Shows**:
```
notification-service: 85.2% coverage, 100% hotspots reviewed
order-service: 70.2% coverage, 3 security issues
product-service: 87% coverage, high reliability
```

---

**12. Compliance and Audit:**
- Every deployment logged in GitHub Actions
- Who deployed what, when, why
- Regulatory requirements easier to meet
- Full history of changes to production

---

**Cost Efficiency:**
- Fewer production incidents = fewer emergency fixes
- Automated testing cheaper than manual testing
- Less downtime = less lost revenue
- Resource optimization via auto-scaling

---

## Summary Table

| Aspect | Benefit | Example |
|--------|---------|---------|
| **Speed** | Deploy to production in 4-5 minutes | Bug fix reaches users immediately |
| **Quality** | Automated testing catches 95% of bugs | 0 bugs in notification-service |
| **Reliability** | Auto-rollback prevents production failures | Failed deployment doesn't impact users |
| **Scalability** | Horizontal scaling works automatically | 10 order-service containers when needed |
| **Security** | SAST catches vulnerabilities early | 3 security issues caught before deployment |
| **Consistency** | Same container everywhere | Eliminates environment differences |
| **Monitoring** | Real-time dashboard for all services | See status of all 4 services instantly |
| **Team Productivity** | Developers push, system deploys | 50% less time on deployment tasks |

---

## Common Interview Follow-up Questions

### Q: What happens if a test fails?
**A:** The pipeline stops immediately. The commit is marked as failed in GitHub. The developer receives a notification and must fix the code. No docker image is built, no deployment happens. This prevents broken code from reaching production.

### Q: What happens if SonarCloud finds critical issues?
**A:** The quality gate fails. The pipeline stops (or continues but marks quality gate failure). Developers must either fix the issues or acknowledge them consciously. Critical security issues usually block deployment.

### Q: How do you rollback if deployment goes wrong?
**A:** ECS stores previous task definitions. If new version fails health checks, ECS automatically keeps the previous working version running. For manual rollback, deploy the previous Docker image tag (e.g., switch from :latest-new back to :previous).

### Q: What if Docker Hub is down?
**A:** ECS has already pulled the image. Current containers keep running. New replicas can't start until Docker Hub is back. This is why reliability is important for registries.

### Q: How do you handle database migrations?
**A:** Migrations should run before container starts or be handled by a separate migration service. The new container version must be compatible with current and previous database schema (backward compatibility).

### Q: What is Fargate vs EC2 in ECS?
**A:** 
- **Fargate**: Serverless, AWS manages servers, you just define task specs
- **EC2**: You manage EC2 instances, ECS runs containers on them
- Your setup likely uses Fargate (simpler, more expensive per task)

### Q: How do you monitor failed containers?
**A:** CloudWatch logs show container output. ECS tracks task state. Alarms can notify on failures. Health checks detect unhealthy containers automatically.

---

## Practice Tips for Viva

1. **Understand the entire flow** - Don't just memorize answers, understand why each step exists
2. **Use your project as example** - Reference your 4 services, security issues found, deployment times
3. **Know the purpose** - Why use Docker? Why use SonarCloud? Why automate?
4. **Be ready for scenarios** - "What if image push fails?" "What if test fails?"
5. **Draw diagrams** - Explain with visual diagrams if possible during viva
6. **Practice explaining simply** - Imagine explaining to non-technical person

---

**Good Luck with Your Viva! 🎓**
