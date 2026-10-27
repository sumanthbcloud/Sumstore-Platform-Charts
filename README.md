# Sumstore-Platform-Charts

Enterprise reusable Helm chart platform for SUM Store microservices.

## 1. Purpose of the Chart

The `microservice` chart is the centralized, standardized Kubernetes deployment blueprint used across all SUM Store backend services and frontend SPAs (`cart`, `catalogue`, `dispatch`, `payment`, `ratings`, `shipping`, `user`, `web`). It provides a single, audited operational standard for:
- Deployments, replica management, and container specifications
- ConfigMap and Secret injection
- Kubernetes Services and port mapping
- Liveness, readiness, and startup health probes
- Resource requests and limits
- Non-root ServiceAccount governance

## 2. Ownership Boundary

- **Platform Team (Owner):** Maintains chart templates, `values.schema.json` validation, CI pipelines, OCI ECR releases, and semantic versioning in `Sumstore-Platform-Charts`.
- **Application Teams (Consumers):** Supply service-specific runtime parameters via GitOps values files (e.g. `image.repository`, `image.tag`, resource sizing, environment variables). Consumers do not edit chart templates directly.

## 3. Consumer Model

Workloads consume this chart as a versioned OCI dependency:
```yaml
dependencies:
  - name: microservice
    version: 1.0.0
    repository: "oci://961544628309.dkr.ecr.us-east-1.amazonaws.com/sumstore/charts"
    alias: cart
```

## 4. Values Contract

| Key | Type | Default | Description |
|---|---|---|---|
| `replicaCount` | integer | `1` | Number of pod replicas |
| `image.repository` | string | `""` | Container image repository URI |
| `image.tag` | string | `""` | Container image tag (immutable commit SHA) |
| `image.pullPolicy` | string | `IfNotPresent` | Image pull policy |
| `containerPort.enabled` | boolean | `true` | Whether container exposes an HTTP port |
| `containerPort.port` | integer | `8080` | Container listening port |
| `service.enabled` | boolean | `true` | Whether to create a ClusterIP Service |
| `service.type` | string | `ClusterIP` | Service type |
| `service.port` | integer | `8080` | Service port |
| `service.targetPort` | integer/string | `8080` | Service target port |
| `config` | object | `{}` | Key-value pairs mounted as environment variables via ConfigMap |
| `existingSecret` | string | `""` | Name of existing Secret to inject via `envFrom` |
| `serviceAccount.create` | boolean | `false` | Whether to create a ServiceAccount |
| `serviceAccount.name` | string | `""` | Custom ServiceAccount name |
| `probes.startup.enabled` | boolean | `false` | Enable/configure startup probe |
| `probes.readiness.enabled`| boolean | `false` | Enable/configure readiness probe |
| `probes.liveness.enabled` | boolean | `false` | Enable/configure liveness probe |
| `resources` | object | `{}` | Container CPU/memory requests and limits |

## 5. Semantic Versioning (SemVer) Policy

Chart versioning strictly follows Semantic Versioning (`MAJOR.MINOR.PATCH`):
- **PATCH (`x.y.Z`):** Backward-compatible template fixes, formatting adjustments, or documentation updates. Existing values contracts remain completely valid.
- **MINOR (`x.Y.0`):** Backward-compatible capability additions (e.g. optional annotations, new optional configuration blocks). Existing consumer values files require no modifications.
- **MAJOR (`X.0.0`):** Breaking changes to chart values contracts, template removals, or renamed schema fields requiring migration planning.

> **Note on `appVersion`:** `appVersion` in `Chart.yaml` reflects the platform chart bundle version baseline. It is **NOT** used as the application container image tag. Application tags come dynamically from Git commit SHAs in GitOps value files.

## 6. OCI / ECR Publication & Immutable Release Policy

- **Registry URI:** `oci://961544628309.dkr.ecr.us-east-1.amazonaws.com/sumstore/charts`
- **Artifact:** `microservice:<SEMVER>`
- **Tag Immutability:** ECR repository `sumstore/charts/microservice` enforces `IMMUTABLE` tag retention and server-side `AES256` encryption.
- **Release Guard:** Workflows verify whether a semantic version has already been pushed to ECR. Overwriting an existing chart version tag is strictly disallowed.

## 7. How GitOps Consumes a Pinned Version

In the umbrella chart (`Sumstore-GitOps/helm/sumstore/Chart.yaml`), update dependencies to pin the released OCI version:
```yaml
dependencies:
  - name: microservice
    version: 1.0.0
    repository: "oci://961544628309.dkr.ecr.us-east-1.amazonaws.com/sumstore/charts"
    alias: cart
```
Run `helm dependency update` to lock the OCI artifact hash in `Chart.lock`.
