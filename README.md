# Kuvik ADC Distribution

Public distribution repository for **Kuvik ADC** (Application Delivery Controller).

This repository holds **user-facing release artifacts** for the workload-cluster operator. Helm charts and container images are published to GitHub Container Registry; binary tarballs are attached to GitHub Releases for airgap installs. Source code lives in private repositories.

## Latest release: v1.0.5

| Artifact | Reference |
|---|---|
| Container image | `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:1.0.5` (also `:latest`) |
| Helm chart (OCI) | `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:1.0.5` |
| Chart tarball | [kuvik-operator-1.0.5.tgz](https://github.com/Kuvik-io/kuvik-adc/releases/download/v1.0.5/kuvik-operator-1.0.5.tgz) |
| Image tarball (airgap) | [kuvik-operator-image-1.0.5.tar.gz](https://github.com/Kuvik-io/kuvik-adc/releases/download/v1.0.5/kuvik-operator-image-1.0.5.tar.gz) |

Full release notes: [v1.0.5](https://github.com/Kuvik-io/kuvik-adc/releases/tag/v1.0.5). Earlier versions: [all releases](https://github.com/Kuvik-io/kuvik-adc/releases).

## What is this?

The **Kuvik Operator** runs on a workload Kubernetes cluster and registers `LoadBalancer`-type Services with a Kuvik LB control plane via gRPC. One operator per workload cluster. The Kuvik LB control plane allocates a VIP, provisions an LB Pod, and announces the route — without changing your application's Service manifest.

The Kuvik LB control plane is a separate product — what this operator talks to. See the Kuvik LB Management UI's **Clusters → Add Workload Cluster** wizard for the values to plug into the install command below.

## Install (online — recommended)

```bash
export KUBECONFIG=<your_kubeconfig>

helm upgrade --install kuvik-operator \
  oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator \
  --version 1.0.5 \
  --namespace kuvik-operator-system --create-namespace \
  --set controllerGRPCAddress=<LB-VIP>:19000 \
  --set clusterID=<your-cluster-id> \
  --set site=<site-label> \
  --set grpc.operatorRegistrationToken=<token-from-UI> \
  --set-string grpc.caCert="<base64-CA-from-UI>"
```

No cluster-level prerequisites — the chart installs cleanly on any Kubernetes 1.27+ cluster. Gateway API support is opt-in (see [Optional: Gateway API support](#optional-gateway-api-support) below).

`controllerGRPCAddress`, `clusterID`, `site`, `grpc.operatorRegistrationToken`, `grpc.caCert` are emitted by the Kuvik LB Management UI's **Add Workload Cluster** wizard — copy them from there.

If the operator runs in the same Kubernetes cluster as the controller, use `kuvik-controller.kuvik-system.svc:19000` as the dial target instead of a VIP.

## Install (airgap / offline)

Works without any registry access — download both tarballs from this release and import locally.

```bash
curl -fLo /tmp/op.tgz \
  https://github.com/Kuvik-io/kuvik-adc/releases/download/v1.0.5/kuvik-operator-1.0.5.tgz
curl -fLo /tmp/img.tar.gz \
  https://github.com/Kuvik-io/kuvik-adc/releases/download/v1.0.5/kuvik-operator-image-1.0.5.tar.gz

# Import image into your container runtime's local image store.
# Pick the command matching your runtime — examples:
#   sudo ctr -n=k8s.io images import /tmp/img.tar.gz   # containerd
#   docker load < <(gunzip -c /tmp/img.tar.gz)         # docker
gunzip -c /tmp/img.tar.gz | sudo ctr -n=k8s.io images import -

# Install from tarball
export KUBECONFIG=<your_kubeconfig>
helm upgrade --install kuvik-operator /tmp/op.tgz \
  --namespace kuvik-operator-system --create-namespace \
  --set controllerGRPCAddress=<LB-VIP>:19000 \
  --set clusterID=<your-cluster-id> \
  --set site=<site-label> \
  --set grpc.operatorRegistrationToken=<token-from-UI> \
  --set-string grpc.caCert="<base64-CA-from-UI>"
```

For a private mirror, retag the image and override `image.repository` / `image.tag` accordingly.

## Optional: Gateway API support

The operator can manage [Gateway API](https://gateway-api.sigs.k8s.io/) resources (`GatewayClass`/`Gateway`/`HTTPRoute`) in addition to plain LoadBalancer-type Services. Disabled by default — opt in only if your workloads use Gateway API.

```bash
# 1. Install Gateway API CRDs (one-time, cluster-wide)
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.2.0/standard-install.yaml

# 2. Re-run the operator helm upgrade with the flag set
helm upgrade --install kuvik-operator \
  oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator \
  --version 1.0.5 \
  --namespace kuvik-operator-system --reuse-values \
  --set gatewayAPI.enabled=true
```

The chart auto-skips the `GatewayClass` resource when the Gateway API CRDs are absent, and the operator binary auto-skips the Gateway controller — so leaving `gatewayAPI.enabled=true` on a cluster without Gateway API CRDs installs cleanly and runs in Service-only mode.

## Uninstall

```bash
helm uninstall kuvik-operator --namespace kuvik-operator-system
```

The chart ships a Helm `pre-delete` hook that calls `FullSync(empty)` against the controller so all LBService records for this cluster are removed automatically. If the controller is unreachable, the hook times out after 60 seconds and uninstall proceeds; orphaned records can be cleaned up from the Management UI. To skip the graceful deregister entirely:

```bash
helm uninstall kuvik-operator --namespace kuvik-operator-system --no-hooks
```

## Compatibility

| Operator version | Kuvik LB control plane |
|---|---|
| 0.11.x | 0.10.x, 0.11.x |

v0.11.0 was a breaking minor on the LB-cluster side (file-path gRPC TLS removed). The operator itself has no breaking changes.

## License & trademarks

Operator binary is proprietary (Kuvik © 2026, all rights reserved). It depends on several Apache 2.0 components, attributed in the appliance's `LICENSES.md` and `NOTICES` files.

"CoreDNS" and other open-source project names referenced in product documentation are used in nominative-fair-use form to identify the underlying upstream software — no affiliation or endorsement is implied.
