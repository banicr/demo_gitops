# Production Hardening Updates

Production-grade Kubernetes resources and security configurations.

## Changes

### 1. ✅ ServiceAccount Enabled
**File:** `helm/demo-flask-app/values.yaml`

```yaml
serviceAccount:
  create: true
  name: "demo-flask-app"
```

**Impact:** Follows least-privilege principle, no longer uses cluster default service account.

---

### 2. ✅ NetworkPolicy Added
**File:** `helm/demo-flask-app/templates/networkpolicy.yaml` (NEW)

**Features:**
- Restricts ingress to same namespace + optional ingress controller
- Allows DNS egress (UDP port 53)
- Configurable external egress

**Configuration:**
```yaml
networkPolicy:
  enabled: true
  ingress:
    enabled: false
  egress:
    allowExternal: true
```

---

### 3. ✅ PodDisruptionBudget Added
**File:** `helm/demo-flask-app/templates/poddisruptionbudget.yaml` (NEW)

**Features:**
- Only created when `replicaCount > 1`
- Ensures `minAvailable: 1` during disruptions
- Protects against node drains and cluster updates

---

### 4. ✅ Separate Liveness and Readiness Probes
**File:** `helm/demo-flask-app/values.yaml`

```yaml
livenessProbe:
  httpGet:
    path: /healthz/live    # Lightweight process check

readinessProbe:
  httpGet:
    path: /healthz/ready   # Full dependency check
```

**Behavior:**
- Liveness failure → Pod restarted
- Readiness failure → Pod removed from service (not killed)

---

### 5. ✅ Graceful Shutdown Configuration
**File:** `helm/demo-flask-app/templates/deployment.yaml`

```yaml
spec:
  template:
    spec:
      terminationGracePeriodSeconds: 45
      containers:
        - lifecycle:
            preStop:
              exec:
                command: ["/bin/sh", "-c", "sleep 10"]
```

**Shutdown sequence:**
1. SIGTERM sent
2. Pod removed from service
3. preStop hook (10s)
4. Gunicorn graceful shutdown (30s)
5. SIGKILL after 45s if needed

---

### 6. ✅ Increased Resource Limits
**File:** `helm/demo-flask-app/values.yaml`

```yaml
resources:
  limits:
    cpu: 1000m
    memory: 512Mi
  requests:
    cpu: 200m
    memory: 256Mi
```

---

## Files

**New:**
- `helm/demo-flask-app/templates/networkpolicy.yaml`
- `helm/demo-flask-app/templates/poddisruptionbudget.yaml`

**Modified:**
- `helm/demo-flask-app/values.yaml`
- `helm/demo-flask-app/templates/deployment.yaml`

---

## Deployment

```bash
helm lint helm/demo-flask-app/
helm template demo-flask-app helm/demo-flask-app/ --validate

# Deploy
git add .
git commit -m "feat: add production-grade Kubernetes resources"
git push origin main
```

ArgoCD will sync automatically within 3 minutes.
