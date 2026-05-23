# Kuvik ADC Distribution

Public distribution repository for **Kuvik ADC** (Application Delivery Controller).

This repository holds **user-facing release artifacts** for the workload-cluster operator. Helm charts and container images are published to GitHub Container Registry; binary tarballs are attached to GitHub Releases for airgap installs. Source code lives in private repositories.

## Latest release: v0.11.92

| Artifact | Reference |
|---|---|
| Container image | `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.11.92` (also `:latest`) |
| Helm chart (OCI) | `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.11.92` |
| Chart tarball | [kuvik-operator-0.11.92.tgz](https://github.com/Kuvik-io/kuvik-adc/releases/download/v0.11.92/kuvik-operator-0.11.92.tgz) |
| Image tarball (airgap) | [kuvik-operator-image-0.11.92.tar.gz](https://github.com/Kuvik-io/kuvik-adc/releases/download/v0.11.92/kuvik-operator-image-0.11.92.tar.gz) |

Full release notes: [v0.11.92](https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.11.92). Earlier versions: [all releases](https://github.com/Kuvik-io/kuvik-adc/releases).

## What is this?

The **Kuvik Operator** runs on a workload Kubernetes cluster and registers `LoadBalancer`-type Services with a Kuvik LB control plane via gRPC. One operator per workload cluster. The Kuvik LB control plane allocates a VIP, provisions an LB Pod, and announces the route — without changing your application's Service manifest.

The Kuvik LB control plane is a separate product — what this operator talks to. See the Kuvik LB Management UI's **Clusters → Add Workload Cluster** wizard for the values to plug into the install command below.

## Install (online — recommended)

```bash
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml   # or your kubeconfig path

helm upgrade --install kuvik-operator \
  oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator \
  --version 0.11.92 \
  --namespace kuvik-operator-system --create-namespace \
  --set controllerGRPCAddress=<LB-VIP>:19000 \
  --set clusterID=<your-cluster-id> \
  --set site=<site-label> \
  --set grpc.operatorRegistrationToken=<token-from-UI> \
  --set-string grpc.caCert="<base64-CA-from-UI>"
```

`controllerGRPCAddress`, `clusterID`, `site`, `grpc.operatorRegistrationToken`, `grpc.caCert` are emitted by the Kuvik LB Management UI's **Add Workload Cluster** wizard — copy them from there.

If the operator runs in the same Kubernetes cluster as the controller, use `kuvik-controller.kuvik-system.svc:19000` as the dial target instead of a VIP.

## Install (airgap / offline)

Works without any registry access — download both tarballs from this release and import locally.

```bash
curl -fLo /tmp/op.tgz \
  https://github.com/Kuvik-io/kuvik-adc/releases/download/v0.11.92/kuvik-operator-0.11.92.tgz
curl -fLo /tmp/img.tar.gz \
  https://github.com/Kuvik-io/kuvik-adc/releases/download/v0.11.92/kuvik-operator-image-0.11.92.tar.gz

# Load image into local containerd (k3s)
gunzip -c /tmp/img.tar.gz | sudo k3s ctr images import -

# Install from tarball
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
helm upgrade --install kuvik-operator /tmp/op.tgz \
  --namespace kuvik-operator-system --create-namespace \
  --set controllerGRPCAddress=<LB-VIP>:19000 \
  --set clusterID=<your-cluster-id> \
  --set site=<site-label> \
  --set grpc.operatorRegistrationToken=<token-from-UI> \
  --set-string grpc.caCert="<base64-CA-from-UI>"
```

For a private mirror, retag the image and override `image.repository` / `image.tag` accordingly.

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
