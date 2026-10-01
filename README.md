# Kuvik ADC Distribution

Public distribution repository for **Kuvik ADC** (Application Delivery Controller).

This repository holds **user-facing release artifacts** for the workload-cluster operator. Helm charts and container images are published to GitHub Container Registry; the chart, the image archive and their checksums are attached to GitHub Releases for air-gapped installs. Source code lives in private repositories.

## Latest release: v1.1.0

| Artifact | Reference |
|---|---|
| Container image | `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:1.1.0` (also `:latest`) — config digest `sha256:69cff258da3839d1fbacab2fe3fdf98af7250e1bc68fa24f0c08935b260fdfcd` |
| Helm chart (OCI) | `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:1.1.0` |
| Chart tarball | [kuvik-operator-1.1.0.tgz](https://github.com/Kuvik-io/kuvik-adc/releases/download/v1.1.0/kuvik-operator-1.1.0.tgz) |
| Image archive | [kuvik-operator-image-1.1.0.tar.gz](https://github.com/Kuvik-io/kuvik-adc/releases/download/v1.1.0/kuvik-operator-image-1.1.0.tar.gz) |
| Checksums | [SHA256SUMS](https://github.com/Kuvik-io/kuvik-adc/releases/download/v1.1.0/SHA256SUMS) |

Full release notes: [v1.1.0](https://github.com/Kuvik-io/kuvik-adc/releases/tag/v1.1.0). Earlier versions: [all releases](https://github.com/Kuvik-io/kuvik-adc/releases).

## What is this?

The **Kuvik Operator** runs on a workload Kubernetes cluster and registers `LoadBalancer`-type Services with a Kuvik LB control plane via gRPC. One operator per workload cluster. The Kuvik LB control plane allocates a VIP, provisions an LB Pod, and announces the route — without changing your application's Service manifest.

The operator is published only when it changes, so the latest release here is the one to install alongside any later Kuvik LB release.

## Install (online)

```bash
helm upgrade --install kuvik-operator \
  oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator --version 1.1.0 \
  --namespace kuvik-operator-system --create-namespace \
  --set controllerGRPCAddress=<LB-VIP>:19000 \
  --set clusterID=<your-cluster-id> \
  --set site=<site-label> \
  --set grpc.operatorRegistrationToken=<token-from-UI> \
  --set-string grpc.caCert="<base64-CA-from-UI>"
```

`controllerGRPCAddress`, `clusterID`, `site`, `grpc.operatorRegistrationToken` and `grpc.caCert` come from the Kuvik LB Management UI's **Add Workload Cluster** wizard.

## Install (air-gapped)

Download the three release assets on a connected machine and verify them before carrying them in:

```bash
curl -fLO https://github.com/Kuvik-io/kuvik-adc/releases/download/v1.1.0/kuvik-operator-1.1.0.tgz
curl -fLO https://github.com/Kuvik-io/kuvik-adc/releases/download/v1.1.0/kuvik-operator-image-1.1.0.tar.gz
curl -fLO https://github.com/Kuvik-io/kuvik-adc/releases/download/v1.1.0/SHA256SUMS
sha256sum -c SHA256SUMS        # both lines must end in OK
gunzip -k kuvik-operator-image-1.1.0.tar.gz                      # -> kuvik-operator-image-1.1.0.tar
```

The image's identity is its **config digest**, `sha256:69cff258da3839d1fbacab2fe3fdf98af7250e1bc68fa24f0c08935b260fdfcd`. It is the same in every registry the image is copied to; the manifest digest is not.

### Path 1 — your own registry (recommended)

```bash
REG=registry.example.internal/kuvik          # your registry and project

# Image: pick one
crane push kuvik-operator-image-1.1.0.tar ${REG}/kuvik-operator:1.1.0
#   or: skopeo copy docker-archive:kuvik-operator-image-1.1.0.tar docker://${REG}/kuvik-operator:1.1.0
#   or: docker load -i kuvik-operator-image-1.1.0.tar
#       docker tag ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:1.1.0 ${REG}/kuvik-operator:1.1.0
#       docker push ${REG}/kuvik-operator:1.1.0

# Verify: must print 69cff258da3839d1fbacab2fe3fdf98af7250e1bc68fa24f0c08935b260fdfcd
crane config ${REG}/kuvik-operator:1.1.0 | sha256sum
#   or: skopeo inspect --raw --config docker://${REG}/kuvik-operator:1.1.0 | sha256sum

# Chart: push it next to the image (or install straight from kuvik-operator-1.1.0.tgz below)
helm push kuvik-operator-1.1.0.tgz oci://${REG}/charts

# Registry credentials for the pods, if your registry needs them
kubectl create namespace kuvik-operator-system
kubectl -n kuvik-operator-system create secret docker-registry kuvik-regcred \
  --docker-server=<registry host> --docker-username=<user> --docker-password=<password>

helm upgrade --install kuvik-operator oci://${REG}/charts/kuvik-operator --version 1.1.0 \
  --namespace kuvik-operator-system --create-namespace \
  --set image.repository=${REG}/kuvik-operator \
  --set image.tag=1.1.0 \
  --set 'imagePullSecrets[0].name=kuvik-regcred' \
  --set controllerGRPCAddress=<LB-VIP>:19000 \
  --set clusterID=<your-cluster-id> \
  --set site=<site-label> \
  --set grpc.operatorRegistrationToken=<token-from-UI> \
  --set-string grpc.caCert="<base64-CA-from-UI>"
```

`imagePullSecrets` reaches both pods the chart runs — the operator and the pre-delete deregister Job. Leave it out if your registry allows anonymous pulls.

### Path 2 — import on every node (no registry)

The chart's default image is `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:1.1.0` with `imagePullPolicy: IfNotPresent`, and the archive carries exactly that name. Import it on **every node** the operator or its deregister Job can be scheduled on:

```bash
sudo ctr -n k8s.io images import kuvik-operator-image-1.1.0.tar      # k3s: sudo k3s ctr -n k8s.io images import …

helm upgrade --install kuvik-operator ./kuvik-operator-1.1.0.tgz \
  --namespace kuvik-operator-system --create-namespace \
  --set controllerGRPCAddress=<LB-VIP>:19000 \
  --set clusterID=<your-cluster-id> \
  --set site=<site-label> \
  --set grpc.operatorRegistrationToken=<token-from-UI> \
  --set-string grpc.caCert="<base64-CA-from-UI>"
```

A node without the image tries to pull from ghcr.io and stays in `ErrImagePull` — prefer Path 1.

## Optional: Gateway API support

The chart skips the `GatewayClass` when the Gateway API CRDs are absent, and the operator then runs in Service-only mode. To use Gateway API, install the upstream CRDs once and re-run the `helm upgrade` above:

```bash
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.2.0/standard-install.yaml
```

## Upgrading

Upgrade the **Kuvik ADC (the LB cluster) first, then the operator** on each workload cluster:

```bash
helm upgrade kuvik-operator oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator --version 1.1.0 \
  --namespace kuvik-operator-system --reset-then-reuse-values --wait
```

- Use `--reset-then-reuse-values` (Helm 3.14 or newer). `--reuse-values` also keeps the previous chart's default image tag, so the upgrade "succeeds" onto the old image.
- Either order keeps traffic flowing: a newer ADC handles an older operator's reports with the rules it used before, and an older ADC ignores the fields a newer operator adds. Features that need both sides — per-namespace TLS certificate references, the trust report the CA-rotation gate reads — take effect only once both are upgraded.
- Until a workload cluster's operator is upgraded, the ADC treats that cluster's CA trust as unverified and refuses a cross-signed CA rotation by default.

## Uninstall

```bash
helm uninstall kuvik-operator --namespace kuvik-operator-system
```

A `pre-delete` hook deregisters this cluster's services from the controller (`FullSync(empty)`); if the controller is unreachable it gives up after 60 seconds and the uninstall proceeds. `--no-hooks` skips it.

## License & trademarks

Operator binary is proprietary (Kuvik © 2026, all rights reserved). It depends on several Apache 2.0 components, attributed in the appliance's `LICENSES.md` and `NOTICES` files.

"CoreDNS" and other open-source project names referenced in product documentation are used in nominative-fair-use form to identify the underlying upstream software — no affiliation or endorsement is implied.
