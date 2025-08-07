# Container Orchestration

Master Docker and Kubernetes to deploy applications that scale, heal themselves, and run anywhere.

## Why Containers and Orchestration?

### The Container Revolution
Containers solve the "it works on my machine" problem:
- **Consistency**: Same environment everywhere
- **Portability**: Run anywhere containers run
- **Efficiency**: Better resource utilization than VMs
- **Speed**: Fast startup and deployment

### Why Orchestration?
Managing containers manually doesn't scale:
- **Service discovery**: How do containers find each other?
- **Load balancing**: Distribute traffic across instances
- **Health checks**: Restart failed containers
- **Rolling deployments**: Update without downtime
- **Scaling**: Add/remove instances based on demand

## Docker Fundamentals

### Container Basics
```dockerfile
# Dockerfile example
FROM python:3.9-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .
EXPOSE 8000

CMD ["python", "app.py"]
```

### Multi-Stage Builds
```dockerfile
# Build stage
FROM node:16 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Docker Compose for Local Development
```yaml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/myapp
    depends_on:
      - db
  
  db:
    image: postgres:13
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

## Kubernetes Architecture

### Core Concepts

#### Pods
The smallest deployable unit:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: app
    image: my-app:v1.0
    ports:
    - containerPort: 8080
```

#### Deployments
Manage replica sets and rolling updates:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: app
        image: my-app:v1.0
        ports:
        - containerPort: 8080
```

#### Services
Expose applications to network traffic:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  ports:
  - port: 80
    targetPort: 8080
  type: LoadBalancer
```

### Advanced Kubernetes Concepts

#### ConfigMaps and Secrets
```yaml
# ConfigMap for configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_host: "db.example.com"
  debug_mode: "false"

---
# Secret for sensitive data
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
data:
  database_password: cGFzc3dvcmQ=  # base64 encoded
```

#### Persistent Volumes
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

#### Ingress Controllers
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
  - hosts:
    - myapp.example.com
    secretName: myapp-tls
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-app-service
            port:
              number: 80
```

## Helm: Kubernetes Package Manager

### Chart Structure
```
my-app-chart/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
└── charts/
```

### Templating with Helm
```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.app.name }}
spec:
  replicas: {{ .Values.app.replicas }}
  selector:
    matchLabels:
      app: {{ .Values.app.name }}
  template:
    spec:
      containers:
      - name: {{ .Values.app.name }}
        image: {{ .Values.app.image }}:{{ .Values.app.tag }}
```

## Service Mesh with Istio

### Traffic Management
- Intelligent routing
- Load balancing
- Circuit breakers
- Retries and timeouts

### Security
- mTLS between services
- Authorization policies
- Certificate management

### Observability
- Distributed tracing
- Metrics collection
- Service graphs

## Production Best Practices

### Resource Management
```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "250m"
  limits:
    memory: "512Mi"
    cpu: "500m"
```

### Health Checks
```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

### Security Contexts
```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  readOnlyRootFilesystem: true
  allowPrivilegeEscalation: false
```

## Hands-On Projects

### Project 1: Microservices on Kubernetes
Deploy a complete microservices application:
- Multiple services with different languages
- Service-to-service communication
- Shared databases and caches
- API gateway with authentication

### Project 2: GitOps with ArgoCD
Implement GitOps deployment:
- Git-based configuration management
- Automated deployments
- Environment promotion
- Rollback capabilities

### Project 3: Multi-Cluster Setup
Deploy across multiple Kubernetes clusters:
- Cluster federation
- Cross-cluster service discovery
- Multi-region deployments
- Disaster recovery

---

*Container orchestration is the foundation of modern application deployment. Master these concepts and you'll be ready for any scale.*
