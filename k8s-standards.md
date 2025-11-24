# Kubernetes Standards

## Manifest Structure

### Standard Project Layout

```
k8s-manifests/
├── README.md
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── configmap.yaml
│   └── kustomization.yaml
├── overlays/
│   ├── dev/
│   │   ├── kustomization.yaml
│   │   └── patches/
│   ├── staging/
│   │   └── kustomization.yaml
│   └── prod/
│       └── kustomization.yaml
└── helm/
    └── my-app/
        ├── Chart.yaml
        ├── values.yaml
        └── templates/
```

## Naming Conventions

### Resources

```yaml
# Format: <app-name>-<component>-<type>
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-api-deployment
  
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-api-service
```

### Labels
```yaml
metadata:
  labels:
    app.kubernetes.io/name: myapp
    app.kubernetes.io/component: api
    app.kubernetes.io/part-of: myapp-platform
    app.kubernetes.io/version: "1.2.3"
    app.kubernetes.io/managed-by: helm
    environment: production
```

### Annotations
```yaml
metadata:
  annotations:
    description: "Main API service"
    owner: "platform-team"
    deployment-date: "2024-01-15"
```

## Deployment Standards

### Always Include
1. Resource limits and requests
2. Health checks (liveness/readiness)
3. Labels and selectors
4. Image tags (never `:latest`)
5. Security context
6. Update strategy

### Good Deployment Example
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-api
  labels:
    app.kubernetes.io/name: myapp
    app.kubernetes.io/component: api
    app.kubernetes.io/version: "1.2.3"
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app.kubernetes.io/name: myapp
      app.kubernetes.io/component: api
  template:
    metadata:
      labels:
        app.kubernetes.io/name: myapp
        app.kubernetes.io/component: api
        app.kubernetes.io/version: "1.2.3"
    spec:
      serviceAccountName: myapp-api
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        fsGroup: 1000
      containers:
      - name: api
        image: myapp/api:1.2.3
        imagePullPolicy: IfNotPresent
        ports:
        - name: http
          containerPort: 8080
          protocol: TCP
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: myapp-secrets
              key: database-url
        - name: LOG_LEVEL
          valueFrom:
            configMapKeyRef:
              name: myapp-config
              key: log-level
        resources:
          requests:
            memory: "256Mi"
            cpu: "100m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /healthz
            port: http
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /ready
            port: http
          initialDelaySeconds: 10
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          capabilities:
            drop:
            - ALL
```

## Resource Management

### Always Set Requests and Limits
```yaml
resources:
  requests:
    memory: "256Mi"  # Guaranteed allocation
    cpu: "100m"      # 0.1 CPU cores
  limits:
    memory: "512Mi"  # Maximum allowed
    cpu: "500m"      # 0.5 CPU cores
```

### Resource Guidelines

#### Small Services
```yaml
resources:
  requests:
    memory: "128Mi"
    cpu: "50m"
  limits:
    memory: "256Mi"
    cpu: "200m"
```

#### Medium Services
```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "100m"
  limits:
    memory: "512Mi"
    cpu: "500m"
```

#### Large Services
```yaml
resources:
  requests:
    memory: "512Mi"
    cpu: "250m"
  limits:
    memory: "1Gi"
    cpu: "1000m"
```

### Quality of Service Classes

#### Guaranteed (Recommended for Production)
```yaml
# Requests = Limits
resources:
  requests:
    memory: "512Mi"
    cpu: "500m"
  limits:
    memory: "512Mi"
    cpu: "500m"
```

#### Burstable (Common)
```yaml
# Requests < Limits
resources:
  requests:
    memory: "256Mi"
    cpu: "100m"
  limits:
    memory: "512Mi"
    cpu: "500m"
```

## Health Checks

### Liveness Probe
Determines if container needs to be restarted
```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 30  # Wait for app to start
  periodSeconds: 10         # Check every 10s
  timeoutSeconds: 5         # Timeout after 5s
  failureThreshold: 3       # Restart after 3 failures
```

### Readiness Probe
Determines if container can receive traffic
```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5   # Check sooner than liveness
  periodSeconds: 5          # Check more frequently
  timeoutSeconds: 3
  failureThreshold: 3       # Remove from service after 3 failures
```

### Startup Probe
For slow-starting applications
```yaml
startupProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 0
  periodSeconds: 10
  failureThreshold: 30      # 5 minutes to start (30 * 10s)
```

### Probe Types

#### HTTP
```yaml
httpGet:
  path: /healthz
  port: 8080
  httpHeaders:
  - name: Custom-Header
    value: Awesome
```

#### TCP
```yaml
tcpSocket:
  port: 8080
```

#### Exec
```yaml
exec:
  command:
  - cat
  - /tmp/healthy
```

## Service Standards

### ClusterIP (Internal)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-api
spec:
  type: ClusterIP
  selector:
    app.kubernetes.io/name: myapp
    app.kubernetes.io/component: api
  ports:
  - name: http
    port: 80
    targetPort: 8080
    protocol: TCP
```

### LoadBalancer (External)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-public
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
spec:
  type: LoadBalancer
  selector:
    app.kubernetes.io/name: myapp
  ports:
  - name: https
    port: 443
    targetPort: 8080
```

### Headless Service (StatefulSet)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-database
spec:
  clusterIP: None  # Headless
  selector:
    app.kubernetes.io/name: myapp
    app.kubernetes.io/component: database
  ports:
  - port: 5432
```

## ConfigMaps and Secrets

### ConfigMap Example
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-config
data:
  # Simple values
  log_level: "info"
  api_timeout: "30"
  
  # Configuration file
  app.yaml: |
    server:
      port: 8080
      timeout: 30
    database:
      pool_size: 10
```

### Secret Example
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: myapp-secrets
type: Opaque
stringData:  # For plain text (will be base64 encoded)
  database-password: "MySecurePassword123"
  api-key: "abc123xyz"
```

### Using ConfigMap
```yaml
# Environment variable
env:
- name: LOG_LEVEL
  valueFrom:
    configMapKeyRef:
      name: myapp-config
      key: log_level

# Mount as volume
volumes:
- name: config
  configMap:
    name: myapp-config
volumeMounts:
- name: config
  mountPath: /etc/config
  readOnly: true
```

### Using Secrets
```yaml
# Environment variable
env:
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: myapp-secrets
      key: database-password

# Mount as volume
volumes:
- name: secrets
  secret:
    secretName: myapp-secrets
volumeMounts:
- name: secrets
  mountPath: /etc/secrets
  readOnly: true
```

## Security Best Practices

### Pod Security Context
```yaml
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 1000
    fsGroup: 1000
    seccompProfile:
      type: RuntimeDefault
```

### Container Security Context
```yaml
securityContext:
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
  capabilities:
    drop:
    - ALL
  runAsNonRoot: true
  runAsUser: 1000
```

### Network Policies
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-network-policy
spec:
  podSelector:
    matchLabels:
      app.kubernetes.io/name: myapp
      app.kubernetes.io/component: api
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app.kubernetes.io/name: myapp
          app.kubernetes.io/component: frontend
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - podSelector:
        matchLabels:
          app.kubernetes.io/name: myapp
          app.kubernetes.io/component: database
    ports:
    - protocol: TCP
      port: 5432
```

## Ingress Standards

### Basic Ingress
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  ingressClassName: nginx
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
            name: myapp-api
            port:
              number: 80
```

## Helm Chart Standards

### Chart.yaml
```yaml
apiVersion: v2
name: myapp
description: My application Helm chart
type: application
version: 1.2.3
appVersion: "1.2.3"
keywords:
  - myapp
  - api
maintainers:
  - name: Your Name
    email: you@example.com
```

### values.yaml Structure
```yaml
# Image configuration
image:
  repository: myapp/api
  tag: "1.2.3"
  pullPolicy: IfNotPresent

# Replica configuration
replicaCount: 3

# Service configuration
service:
  type: ClusterIP
  port: 80
  targetPort: 8080

# Ingress configuration
ingress:
  enabled: true
  className: nginx
  hosts:
    - host: myapp.example.com
      paths:
        - path: /
          pathType: Prefix

# Resources
resources:
  requests:
    memory: "256Mi"
    cpu: "100m"
  limits:
    memory: "512Mi"
    cpu: "500m"

# Autoscaling
autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80

# Environment-specific overrides
env:
  LOG_LEVEL: info
  API_TIMEOUT: "30"
```

## Kustomize Standards

### Base Kustomization
```yaml
# base/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- deployment.yaml
- service.yaml
- configmap.yaml

commonLabels:
  app.kubernetes.io/name: myapp
  app.kubernetes.io/part-of: myapp-platform
```

### Environment Overlay
```yaml
# overlays/prod/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: production

resources:
- ../../base

replicas:
- name: myapp-api
  count: 5

images:
- name: myapp/api
  newTag: 1.2.3

commonLabels:
  environment: production

patchesStrategicMerge:
- patches/resources.yaml
```

## StatefulSet Standards

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres
  replicas: 3
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:15
        ports:
        - containerPort: 5432
          name: postgres
        volumeMounts:
        - name: data
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi
```

## Common Mistakes to Avoid

### ❌ Don't
- Use `:latest` image tag
- Run containers as root
- Skip resource limits
- Forget health checks
- Hardcode values
- Use hostPort
- Store secrets in ConfigMaps
- Expose services unnecessarily

### ✅ Do
- Use specific image tags
- Run as non-root user
- Set resource requests/limits
- Implement all health checks
- Use ConfigMaps/Secrets for configuration
- Use proper service types
- Secure secrets properly
- Follow principle of least privilege

## YAML Best Practices

### Formatting
- 2 spaces for indentation
- Use `---` to separate resources
- Keep related resources in same file
- Use multi-line strings for readability

### Comments
```yaml
# This deployment runs our main API service
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-api
spec:
  # Scale to handle peak traffic
  replicas: 3
```

## Testing Manifests

### Validation
```bash
# Dry run
kubectl apply -f deployment.yaml --dry-run=client

# Server-side validation
kubectl apply -f deployment.yaml --dry-run=server

# With kubectl
kubectl apply --validate=true -f deployment.yaml
```

### Linting
```bash
# Using kubeval
kubeval deployment.yaml

# Using kube-score
kube-score score deployment.yaml
```

## Quick Reference

### Essential kubectl Commands
```bash
# Apply manifests
kubectl apply -f deployment.yaml

# Get resources
kubectl get pods
kubectl get deployments
kubectl get services

# Describe resources
kubectl describe pod myapp-api-abc123

# Logs
kubectl logs -f myapp-api-abc123

# Execute commands
kubectl exec -it myapp-api-abc123 -- /bin/sh

# Port forward
kubectl port-forward svc/myapp-api 8080:80

# Scale
kubectl scale deployment myapp-api --replicas=5
```

### Manifest Checklist
- [ ] Used specific image tags
- [ ] Set resource requests and limits
- [ ] Added liveness and readiness probes
- [ ] Applied proper labels and selectors
- [ ] Configured security contexts
- [ ] Used ConfigMaps for config
- [ ] Used Secrets for sensitive data
- [ ] Defined update strategy
- [ ] Added annotations for documentation
- [ ] Tested with dry-run

---