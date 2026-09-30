# Kuvik ADC Changelog

## v1.0.532 — 2026-09-30

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:1.0.532` (config `sha256:a186a81b9c20959c451f8f84997ede9d520270278473b5f82229823e14038997`)
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:1.0.532`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v1.0.532
- LB cluster build: kuvik-lb-cluster commit `3780977dcd9f`

#### Behaviour changes

- **The operator now verifies the LB cluster's controller by its URI identity, not by host name — and needs LB cluster 1.0.530 or newer.** The operator requires the controller's gRPC certificate to chain to the pinned CA **and** carry the URI SAN `kuvik://controller/grpc-server`. There is no fallback to a DNS name: a certificate that only names `kuvik-controller.<ns>.svc` is refused, because the LB cluster's CA signs DNS names that an operator-role REST user chooses, so a matching name is not proof of being the controller. LB clusters from 1.0.530 present the URI; against an older LB cluster the operator cannot connect (the log line says the certificate carries no `kuvik://controller/grpc-server` URI SAN). **Upgrade the LB cluster first, then the operators.** `grpc.serverName` remains a chart value but is now sent as SNI only. The CA-handover behaviour of `grpc.trustIssuedCAOnPinFailure` is unchanged: it still arms only on a genuine "unknown authority", never on a URI mismatch.
- **The registration token is no longer passed as a command-line argument.** The chart stores `grpc.operatorRegistrationToken` in a Secret (`<release>-registration-token`) and gives the operator and the pre-delete Job the environment variable `KUVIK_GRPC_OPERATOR_REGISTRATION_TOKEN` from it (`secretKeyRef`). Before, anyone who could read the Deployment or Pod in the workload cluster, and `ps` on the node, could read the token. Existing installations: upgrade with `helm upgrade --reset-then-reuse-values` (the stored `grpc.operatorRegistrationToken` is kept and the Secret is created; you do not need to type the token again). The `--grpc-operator-registration-token` flag is still accepted for one release for hand-written manifests; the chart no longer sets it. If the token was ever visible in a Pod spec, regenerate it in the Management UI (Settings, Operator Registration) and set the new value on upgrade.


## v1.0.531 — 2026-09-30

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:1.0.531` (config `sha256:a3c7764238335f94c8aadce1fb6ae83f1c10d4bd3b4005740ccc1c17ff736c31`)
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:1.0.531`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v1.0.531
- LB cluster build: kuvik-lb-cluster commit `4eb05c37f0b3`

#### Behaviour changes

- **ListenerSet `certificateRefs` without a namespace now resolve in the ListenerSet's own namespace** (Gateway API spec: "When unspecified, the local namespace is inferred"). Earlier operators resolved them in the parent Gateway's namespace. Set `namespace:` explicitly to keep referencing the Gateway's namespace — a ReferenceGrant is still required for a cross-namespace reference.

#### Fixes

- A Gateway's backend Service port is matched on (port, protocol), not on the port number alone, so a Service that exposes the same number over TCP and UDP resolves the right NodePort.
- ReferenceGrant verdicts are carried per reference (per listener slot), not per Secret, so one grant no longer admits or refuses another referrer's use of the same Secret.
- BackendPool names for a Gateway bundle are decided once by the operator; only colliding pools get identity-derived names, so two Gateways whose names fold to the same pool no longer share one.


## v1.0.530 — 2026-09-29

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:1.0.530` (config `sha256:773a5b403a117cde7e506994091a2a56314058231c5947824d7e2dc4e60fe882`)
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:1.0.530`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v1.0.530
- LB cluster build: kuvik-lb-cluster commit `8b0304822a8c`


## v1.0.529 — 2026-09-26

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:1.0.529` (config `sha256:3b00ade78de9582486441b9305b1d304f356b1ebd300962383e27f9116192929`)
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:1.0.529`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v1.0.529
- LB cluster build: kuvik-lb-cluster commit `9501b5138af8`


## v1.0.512 — 2026-09-22

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:1.0.512`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:1.0.512`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v1.0.512
- LB cluster build: kuvik-lb-cluster commit `3896b937`


## v1.0.511 — 2026-09-22

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:1.0.511`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:1.0.511`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v1.0.511
- LB cluster build: kuvik-lb-cluster commit `9d017df8`


## v1.0.509 — 2026-09-22

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:1.0.509`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:1.0.509`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v1.0.509
- LB cluster build: kuvik-lb-cluster commit `bd98b51d`


## v1.0.508 — 2026-09-21

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:1.0.508`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:1.0.508`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v1.0.508
- LB cluster build: kuvik-lb-cluster commit `e6c1a2f6`


## v1.0.367 — 2026-09-08

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:1.0.367`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:1.0.367`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v1.0.367
- LB cluster build: kuvik-lb-cluster commit `5de7ef7a`


## v1.0.331 — 2026-09-05

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:1.0.331`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:1.0.331`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v1.0.331
- LB cluster build: kuvik-lb-cluster commit `620df4a4`


## v1.0.321 — 2026-09-04

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:1.0.321`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:1.0.321`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v1.0.321
- LB cluster build: kuvik-lb-cluster commit `2f59495b`


## v1.0.316 — 2026-09-02

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:1.0.316`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:1.0.316`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v1.0.316
- LB cluster build: kuvik-lb-cluster commit `4c5bee18`


## v1.0.5 — 2026-06-11

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:1.0.5`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:1.0.5`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v1.0.5
- LB cluster build: kuvik-lb-cluster commit `39a8525f`


## v1.0.4 — 2026-05-30

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:1.0.4`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:1.0.4`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v1.0.4
- LB cluster build: kuvik-lb-cluster commit `38a7d2ea`


## v1.0.3 — 2026-05-30

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:1.0.3`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:1.0.3`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v1.0.3
- LB cluster build: kuvik-lb-cluster commit `40bb398a`


## v1.0.2 — 2026-05-30

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:1.0.2`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:1.0.2`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v1.0.2
- LB cluster build: kuvik-lb-cluster commit `f9132928`


## v1.0.1 — 2026-05-30

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:1.0.1`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:1.0.1`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v1.0.1
- LB cluster build: kuvik-lb-cluster commit `df2ffc16`


## v0.13.80 — 2026-05-28

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.13.80`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.13.80`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.13.80
- LB cluster build: kuvik-lb-cluster commit `b9833ae`


## v0.13.5 — 2026-05-24

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.13.5`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.13.5`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.13.5
- LB cluster build: kuvik-lb-cluster commit `baf3f02`


## v0.13.2 — 2026-05-24

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.13.2`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.13.2`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.13.2
- LB cluster build: kuvik-lb-cluster commit `5936c1a`


## v0.13.1 — 2026-05-23

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.13.1`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.13.1`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.13.1
- LB cluster build: kuvik-lb-cluster commit `1345a6e`


## v0.13.0 — 2026-05-23

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.13.0`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.13.0`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.13.0
- LB cluster build: kuvik-lb-cluster commit `369ee7d`


## v0.12.4 — 2026-05-23

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.12.4`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.12.4`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.12.4
- LB cluster build: kuvik-lb-cluster commit `870ba18`


## v0.12.3 — 2026-05-23

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.12.3`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.12.3`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.12.3
- LB cluster build: kuvik-lb-cluster commit `f1b7976`


## v0.12.2 — 2026-05-23

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.12.2`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.12.2`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.12.2
- LB cluster build: kuvik-lb-cluster commit `52cedf6`


## v0.12.1 — 2026-05-23

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.12.1`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.12.1`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.12.1
- LB cluster build: kuvik-lb-cluster commit `7c150f2`


## v0.12.0 — 2026-05-23

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.12.0`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.12.0`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.12.0
- LB cluster build: kuvik-lb-cluster commit `6f16fc5`


## v0.11.100 — 2026-05-23

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.11.100`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.11.100`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.11.100
- LB cluster build: kuvik-lb-cluster commit `4a4493b`


## v0.11.98 — 2026-05-23

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.11.98`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.11.98`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.11.98
- LB cluster build: kuvik-lb-cluster commit `c101f88`


## v0.11.97 — 2026-05-23

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.11.97`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.11.97`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.11.97
- LB cluster build: kuvik-lb-cluster commit `6f37ec1`


## v0.11.96 — 2026-05-23

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.11.96`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.11.96`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.11.96
- LB cluster build: kuvik-lb-cluster commit `f5ea795`


## v0.11.95 — 2026-05-23

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.11.95`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.11.95`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.11.95
- LB cluster build: kuvik-lb-cluster commit `b80551b`


## v0.11.93 — 2026-05-23

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.11.93`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.11.93`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.11.93
- LB cluster build: kuvik-lb-cluster commit `09b2c79`


## v0.11.92 — 2026-05-23

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.11.92`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.11.92`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.11.92
- LB cluster build: kuvik-lb-cluster commit `c9ea6e7`


## v0.11.91 — 2026-05-23

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.11.91`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.11.91`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.11.91
- LB cluster build: kuvik-lb-cluster commit `5433b4f`


## v0.11.88 — 2026-05-23

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.11.88`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.11.88`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.11.88
- LB cluster build: kuvik-lb-cluster commit `ad3fcf8`


## v0.11.86 — 2026-05-23

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.11.86`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.11.86`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.11.86
- LB cluster build: kuvik-lb-cluster commit `fe0d8ba`


## v0.11.85 — 2026-05-23

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.11.85`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.11.85`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.11.85
- LB cluster build: kuvik-lb-cluster commit `742e38d`


## v0.11.84 — 2026-05-23

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.11.84`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.11.84`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.11.84
- LB cluster build: kuvik-lb-cluster commit `6de0cf0`


## v0.11.83 — 2026-05-23

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.11.83`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.11.83`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.11.83
- LB cluster build: kuvik-lb-cluster commit `be01dff`


## v0.11.82 — 2026-05-22

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.11.82`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.11.82`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.11.82
- LB cluster build: kuvik-lb-cluster commit `c9a6aea`


## v0.11.81 — 2026-05-22

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.11.81`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.11.81`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.11.81
- LB cluster build: kuvik-lb-cluster commit `76f10c1`


## v0.11.80 — 2026-05-22

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.11.80`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.11.80`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.11.80
- LB cluster build: kuvik-lb-cluster commit `d7111d8`


## v0.11.78 — 2026-05-22

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.11.78`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.11.78`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.11.78
- LB cluster build: kuvik-lb-cluster commit `39cfaab`


## v0.11.65 — 2026-05-22

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.11.65`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.11.65`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.11.65
- LB cluster build: kuvik-lb-cluster commit `b6ea9f5`


## v0.11.63 — 2026-05-21

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.11.63`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.11.63`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.11.63
- LB cluster build: kuvik-lb-cluster commit `8b75c96`


## v0.11.61 — 2026-05-21

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.11.61`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.11.61`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.11.61
- LB cluster build: kuvik-lb-cluster commit `e1104ac`


## v0.11.60 — 2026-05-21

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.11.60`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.11.60`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.11.60
- LB cluster build: kuvik-lb-cluster commit `688e73f`


## v0.11.57 — 2026-05-21

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.11.57`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.11.57`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.11.57
- LB cluster build: kuvik-lb-cluster commit `c4f9f66`


## v0.11.55 — 2026-05-21

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.11.55`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.11.55`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.11.55
- LB cluster build: kuvik-lb-cluster commit `4de5356`


## v0.11.54 — 2026-05-21

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.11.54`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.11.54`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.11.54
- LB cluster build: kuvik-lb-cluster commit `0412115`


## v0.11.53 — 2026-05-21

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.11.53`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.11.53`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.11.53
- LB cluster build: kuvik-lb-cluster commit `4230a67`


## v0.11.52 — 2026-05-21

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.11.52`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.11.52`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.11.52
- LB cluster build: kuvik-lb-cluster commit `fae0036`


## v0.11.51 — 2026-05-21

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.11.51`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.11.51`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.11.51
- LB cluster build: kuvik-lb-cluster commit `880f8a4`


## v0.11.50 — 2026-05-21

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.11.50`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.11.50`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.11.50
- LB cluster build: kuvik-lb-cluster commit `1329b7a`


## v0.11.49 — 2026-05-21

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.11.49`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.11.49`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.11.49
- LB cluster build: kuvik-lb-cluster commit `5c34b5b`


## v0.11.13 — 2026-05-19

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.11.13`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.11.13`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.11.13
- LB cluster build: kuvik-lb-cluster commit `b88eaee`


## v0.11.12 — 2026-05-19

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.11.12`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.11.12`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.11.12
- LB cluster build: kuvik-lb-cluster commit `3b74c03`


## v0.11.11 — 2026-05-19

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.11.11`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.11.11`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.11.11
- LB cluster build: kuvik-lb-cluster commit `96be241`


## v0.11.10 — 2026-05-19

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.11.10`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.11.10`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.11.10
- LB cluster build: kuvik-lb-cluster commit `92a20ad`


## v0.11.9 — 2026-05-19

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.11.9`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.11.9`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.11.9
- LB cluster build: kuvik-lb-cluster commit `3330914`


## v0.11.8 — 2026-05-19

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.11.8`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.11.8`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.11.8
- LB cluster build: kuvik-lb-cluster commit `e7cbe53`


## v0.11.7 — 2026-05-19

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.11.7`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.11.7`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.11.7
- LB cluster build: kuvik-lb-cluster commit `6f6fc69`


## v0.11.6 — 2026-05-19

- Image: `ghcr.io/kuvik-io/kuvik-adc/kuvik-operator:0.11.6`
- Chart: `oci://ghcr.io/kuvik-io/kuvik-adc/charts/kuvik-operator:0.11.6`
- Release: https://github.com/Kuvik-io/kuvik-adc/releases/tag/v0.11.6
- LB cluster build: kuvik-lb-cluster commit `e53149a`


