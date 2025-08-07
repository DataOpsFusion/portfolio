# Deployment Automation

Master the art of automated deployments that are safe, fast, and reliable. Learn deployment strategies that minimize risk and maximize confidence.

## Deployment Fundamentals

### What Makes a Good Deployment?
- **Zero downtime**: Users never notice the deployment
- **Fast rollback**: Quick recovery if something goes wrong
- **Automated**: No manual steps that can fail
- **Observable**: Know what's happening during deployment
- **Repeatable**: Same process works across environments

### Common Deployment Challenges
- **Database migrations**: Schema changes with data
- **Configuration drift**: Environments get out of sync
- **Service dependencies**: Coordinating multiple services
- **Traffic management**: Routing users safely
- **State management**: Handling stateful applications

## Deployment Strategies

### 1. Recreate Deployment
**How it works**: Stop old version, deploy new version
**Pros**: Simple, clean slate
**Cons**: Downtime during deployment

```yaml
# Kubernetes recreate deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  strategy:
    type: Recreate
  replicas: 3
  template:
    spec:
      containers:
      - name: app
        image: myapp:v2.0
```

**Use when**: Development environments, maintenance windows acceptable

### 2. Rolling Update
**How it works**: Gradually replace instances one by one
**Pros**: No downtime, automatic rollback
**Cons**: Mixed versions during deployment

```yaml
# Rolling update configuration
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  replicas: 5
```

**Use when**: Stateless applications, backward compatibility

### 3. Blue-Green Deployment
**How it works**: Deploy to parallel environment, switch traffic
**Pros**: Instant rollback, test production load
**Cons**: Double resources, database complexity

```bash
# Blue-green with load balancer
# Deploy to green environment
kubectl apply -f green-deployment.yaml

# Test green environment
curl -H "Host: myapp.com" http://green-lb-ip/health

# Switch traffic to green
kubectl patch service myapp -p '{"spec":{"selector":{"version":"green"}}}'

# Monitor and rollback if needed
kubectl patch service myapp -p '{"spec":{"selector":{"version":"blue"}}}'
```

**Use when**: Critical applications, complex rollback scenarios

### 4. Canary Deployment
**How it works**: Route small percentage of traffic to new version
**Pros**: Risk reduction, real user feedback
**Cons**: Complex traffic management, monitoring needed

```yaml
# Istio canary deployment
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: myapp
spec:
  http:
  - match:
    - headers:
        canary:
          exact: "true"
    route:
    - destination:
        host: myapp
        subset: v2
  - route:
    - destination:
        host: myapp
        subset: v1
      weight: 95
    - destination:
        host: myapp
        subset: v2
      weight: 5  # 5% of traffic to new version
```

**Use when**: High-risk changes, user-facing features

### 5. A/B Testing Deployment
**How it works**: Route traffic based on user attributes
**Pros**: Feature validation, user segmentation
**Cons**: Complex routing logic, data analysis needed

```javascript
// Feature flag based routing
function getAppVersion(user) {
  if (user.betaUser || user.id % 100 < 10) {
    return 'v2';  // 10% of users get new version
  }
  return 'v1';
}
```

**Use when**: Feature experiments, gradual rollouts

## Database Deployment Strategies

### Migration Patterns

#### 1. Backward Compatible Migrations
Safe migrations that work with both versions:

```sql
-- Phase 1: Add new column (nullable)
ALTER TABLE users ADD COLUMN new_email VARCHAR(255);

-- Phase 2: Populate new column
UPDATE users SET new_email = email WHERE new_email IS NULL;

-- Phase 3: Make column non-nullable
ALTER TABLE users ALTER COLUMN new_email SET NOT NULL;

-- Phase 4: Drop old column (after deployment)
ALTER TABLE users DROP COLUMN email;
```

#### 2. Expand-Contract Pattern
```sql
-- Expand: Add new structure
ALTER TABLE users ADD COLUMN user_preferences JSONB;

-- Deploy application that writes to both old and new
-- (Application handles dual writes)

-- Contract: Remove old structure
ALTER TABLE users DROP COLUMN preference_json;
```

### Zero-Downtime Database Changes
```yaml
# Database migration job
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration
spec:
  template:
    spec:
      containers:
      - name: migrate
        image: migrate/migrate
        args: [
          "-path", "/migrations",
          "-database", "postgres://...",
          "up"
        ]
      restartPolicy: OnFailure
```

## Infrastructure as Code Deployments

### Terraform Deployments
```hcl
# Terraform with workspaces
resource "aws_instance" "web" {
  count         = terraform.workspace == "prod" ? 3 : 1
  ami           = var.ami_id
  instance_type = terraform.workspace == "prod" ? "t3.medium" : "t3.micro"
  
  tags = {
    Name        = "web-${terraform.workspace}-${count.index}"
    Environment = terraform.workspace
  }
}
```

### Pulumi Deployments
```python
import pulumi_aws as aws

# Environment-specific configuration
config = pulumi.Config()
instance_count = config.get_int("instance_count", 1)
instance_type = config.get("instance_type", "t3.micro")

# Create instances
for i in range(instance_count):
    aws.ec2.Instance(f"web-{i}",
        ami="ami-0c02fb55956c7d316",
        instance_type=instance_type,
        tags={"Name": f"web-{i}"}
    )
```

## GitOps Deployments

### ArgoCD Configuration
```yaml
# Application manifest
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/myapp-config
    targetRevision: HEAD
    path: k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```

### FluxCD with Helm
```yaml
# HelmRelease for Flux
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: myapp
spec:
  interval: 5m
  chart:
    spec:
      chart: myapp
      version: ">=1.0.0"
      sourceRef:
        kind: HelmRepository
        name: myapp-repo
  values:
    image:
      tag: "v1.2.3"
    replicas: 3
```

## Deployment Automation Tools

### GitHub Actions Deployment
```yaml
name: Deploy to Production
on:
  push:
    tags: ['v*']

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to Kubernetes
        uses: azure/k8s-deploy@v1
        with:
          manifests: |
            k8s/deployment.yaml
            k8s/service.yaml
          images: |
            myapp:${{ github.ref_name }}
```

### Jenkins Pipeline
```groovy
pipeline {
    agent any
    
    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['dev', 'staging', 'prod'],
            description: 'Target environment'
        )
        booleanParam(
            name: 'SKIP_TESTS',
            defaultValue: false,
            description: 'Skip test execution'
        )
    }
    
    stages {
        stage('Test') {
            when {
                not { params.SKIP_TESTS }
            }
            steps {
                sh 'npm test'
            }
        }
        
        stage('Build') {
            steps {
                sh 'docker build -t myapp:${BUILD_NUMBER} .'
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    if (params.ENVIRONMENT == 'prod') {
                        input message: 'Deploy to production?', ok: 'Deploy'
                    }
                }
                sh "kubectl set image deployment/myapp myapp=myapp:${BUILD_NUMBER}"
            }
        }
    }
    
    post {
        failure {
            slackSend(
                color: 'danger',
                message: "Deployment failed: ${env.JOB_NAME} - ${env.BUILD_NUMBER}"
            )
        }
    }
}
```

## Deployment Monitoring

### Health Checks
```yaml
# Kubernetes health checks
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 3

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
  timeoutSeconds: 3
  failureThreshold: 2
```

### Deployment Metrics
```python
# Custom deployment metrics
from prometheus_client import Counter, Histogram, Gauge

DEPLOYMENTS_TOTAL = Counter('deployments_total', 'Total deployments', ['status', 'environment'])
DEPLOYMENT_DURATION = Histogram('deployment_duration_seconds', 'Deployment duration')
ACTIVE_VERSION = Gauge('active_version', 'Current active version', ['service'])

# Track deployment
with DEPLOYMENT_DURATION.time():
    try:
        deploy_application()
        DEPLOYMENTS_TOTAL.labels(status='success', environment='prod').inc()
    except Exception as e:
        DEPLOYMENTS_TOTAL.labels(status='failure', environment='prod').inc()
        raise
```

### Automated Rollback
```yaml
# Automated rollback based on metrics
- name: Check deployment health
  run: |
    # Wait for deployment to stabilize
    sleep 60
    
    # Check error rate
    ERROR_RATE=$(curl -s "http://prometheus:9090/api/v1/query?query=rate(http_requests_total{status=~\"5..\"}[5m])" | jq '.data.result[0].value[1] | tonumber')
    
    if (( $(echo "$ERROR_RATE > 0.05" | bc -l) )); then
      echo "High error rate detected: $ERROR_RATE"
      kubectl rollout undo deployment/myapp
      exit 1
    fi
```

## Security in Deployments

### Secure Secret Management
```yaml
# External Secrets Operator
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: vault-backend
spec:
  provider:
    vault:
      server: "https://vault.example.com"
      path: "secret"
      auth:
        kubernetes:
          mountPath: "kubernetes"
          role: "myapp"

---
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: myapp-secrets
spec:
  secretStoreRef:
    name: vault-backend
    kind: SecretStore
  target:
    name: myapp-secrets
  data:
  - secretKey: database-password
    remoteRef:
      key: myapp
      property: db_password
```

### Container Scanning
```yaml
# Trivy container scanning
- name: Run Trivy vulnerability scanner
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: 'myapp:${{ github.sha }}'
    format: 'sarif'
    output: 'trivy-results.sarif'
    exit-code: '1'
    severity: 'CRITICAL,HIGH'
```

---

*Deployment automation is about building confidence through consistency. The goal is to make deployments so routine and reliable that they become invisible.*
