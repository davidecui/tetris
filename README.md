# Tetris Web

A classic Tetris game playable in the browser, served by a Python FastAPI backend. Production-ready with Docker, Kubernetes manifests, and a Helm chart designed as a reusable Krateo blueprint.

## Quick Start — Local Development

```bash
# Install dependencies
pip install -r requirements.txt

# Run the server
uvicorn app.main:app --host 0.0.0.0 --port 8080

# Open in browser
# http://localhost:8080
```

Health check:

```bash
curl http://localhost:8080/health
# {"status":"ok"}
```

## Controls

| Key | Action |
|-----|--------|
| ← → | Move piece |
| ↓ | Soft drop |
| ↑ / Space | Rotate |
| C | Hold piece |
| P | Pause / Resume |
| R | Restart |

## Docker

```bash
# Build
docker build -t tetris-web:latest .

# Run
docker run -p 8080:8080 tetris-web:latest

# Open http://localhost:8080
```

The container runs as a non-root user and exposes port **8080**.

## Kubernetes (raw manifests)

The `skeleton/k8s/` directory provides ready-to-use manifests:

```bash
# Apply (assumes an image is available to the cluster)
kubectl apply -f skeleton/k8s/

# Port-forward to test locally
kubectl port-forward svc/tetris-web 8080:80
```

The raw manifests use a **ClusterIP** Service. To expose externally, change the type to `NodePort` or `LoadBalancer`, or add an Ingress resource.

### What's included

- **Deployment** — 1 replica, liveness/readiness probes on `/health`, resource requests/limits.
- **Service** — ClusterIP, port 80 → targetPort 8080.

## Helm Chart

A self-contained Helm chart lives under `skeleton/chart/`.

```bash
# Lint
helm lint ./skeleton/chart

# Template (dry-run)
helm template tetris-web ./skeleton/chart

# Install
helm install tetris-web ./skeleton/chart

# Install with overrides
helm install tetris-web ./skeleton/chart \
  --set replicaCount=2 \
  --set service.type=NodePort \
  --set image.repository=ghcr.io/myorg/tetris-web \
  --set image.tag=v1.0.0

# Upgrade
helm upgrade tetris-web ./skeleton/chart

# Uninstall
helm uninstall tetris-web
```

### Configurable Values

See [`skeleton/chart/values.yaml`](skeleton/chart/values.yaml) for all defaults. Key parameters:

| Parameter | Default | Description |
|-----------|---------|-------------|
| `replicaCount` | `1` | Number of replicas |
| `image.repository` | `tetris-web` | Container image repo |
| `image.tag` | `latest` | Container image tag |
| `image.pullPolicy` | `IfNotPresent` | Pull policy |
| `service.type` | `ClusterIP` | Service type |
| `service.port` | `80` | Service port |
| `service.targetPort` | `8080` | Container port |
| `ingress.enabled` | `false` | Enable Ingress |
| `app.healthPath` | `/health` | Probe endpoint |
| `resources.requests.cpu` | `50m` | CPU request |
| `resources.requests.memory` | `64Mi` | Memory request |
| `resources.limits.cpu` | `200m` | CPU limit |
| `resources.limits.memory` | `128Mi` | Memory limit |

## Krateo Blueprint Compatibility

The chart includes a [`skeleton/chart/values.schema.json`](skeleton/chart/values.schema.json) file (JSON Schema draft-07) that:

- Validates all input values.
- Provides `title` and `description` fields suitable for rendering in a self-service platform UI.
- Uses `enum` constraints for `image.pullPolicy`, `service.type`, and `ingress.pathType`.
- Defines sensible defaults so the chart works out of the box.
- Is structured as a **blueprint contract** — platform teams can consume this chart with minimal customization.

## Project Structure

```
├── app/
│   ├── main.py            # FastAPI backend
│   └── static/
│       ├── index.html      # Game page
│       ├── style.css       # Styling
│       └── tetris.js       # Game engine
├── skeleton/
│   ├── k8s/
│   │   ├── deployment.yaml # Raw K8s Deployment
│   │   └── service.yaml    # Raw K8s Service
│   └── chart/
│       ├── Chart.yaml
│       ├── values.yaml
│       ├── values.schema.json # Krateo-ready schema
│       └── templates/
│           ├── _helpers.tpl
│           ├── deployment.yaml
│           ├── service.yaml
│           └── ingress.yaml
├── Dockerfile
├── .dockerignore
├── requirements.txt
└── README.md
```

## License

MIT
