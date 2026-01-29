# PiAware Helm Chart

A Helm chart for deploying [PiAware](https://flightaware.com/adsb/piaware/) - FlightAware's ADS-B feeder software using the [sdr-enthusiasts/docker-piaware](https://github.com/sdr-enthusiasts/docker-piaware) container image.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.0+
- A node with RTL-SDR hardware attached
- A FlightAware account and Feeder ID

## Installation

### Add the Helm repository

```bash
helm repo add piaware https://jvhaarst.github.io/docker-piaware-helm
helm repo update
```

### Install the chart

```bash
# Install with your FlightAware Feeder ID
helm install piaware piaware/piaware \
  --namespace piaware \
  --set flightaware.feederId=YOUR_FEEDER_ID

# Or use an existing Kubernetes secret
helm install piaware piaware/piaware \
  --namespace piaware \
  --set flightaware.existingSecret=my-piaware-secret
```

### Label your node

The chart uses a node selector to schedule on nodes with RTL-SDR hardware:

```bash
kubectl label node <node-name> rtlsdr=true
```

## Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `namespace.create` | Create the namespace | `true` |
| `namespace.name` | Namespace name | `piaware` |
| `image.repository` | Container image | `ghcr.io/sdr-enthusiasts/docker-piaware` |
| `image.tag` | Image tag | `latest-build-640` |
| `flightaware.feederId` | Your FlightAware Feeder ID | `""` |
| `flightaware.existingSecret` | Use existing secret for Feeder ID | `""` |
| `general.timezone` | Timezone | `UTC` |
| `receiver.type` | Receiver type: `rtlsdr` or `relay` | `rtlsdr` |
| `receiver.device` | RTL-SDR device serial number | `""` |
| `receiver.gain` | RTL-SDR gain setting | `""` (max) |
| `mlat.enabled` | Enable multilateration | `true` |
| `mlat.results` | Return MLAT results | `true` |
| `uat.receiverType` | UAT receiver: `""`, `rtlsdr`, or `relay` | `""` |
| `service.type` | Kubernetes service type | `ClusterIP` |
| `service.webPort` | Web interface port | `8080` |
| `nodeSelector` | Node selector labels | `rtlsdr: "true"` |

See [values.yaml](piaware/values.yaml) for all configuration options.

## Accessing the Web Interface

```bash
kubectl port-forward -n piaware svc/piaware 8080:8080
```

Then open http://localhost:8080 in your browser.

## Getting a Feeder ID

1. Install and run PiAware without a Feeder ID
2. Visit https://flightaware.com/adsb/piaware/claim
3. Follow the instructions to claim your feeder
4. Update your Helm release with the assigned Feeder ID

## Relay Mode

If you have an external ADS-B receiver (e.g., running dump1090), you can use relay mode:

```bash
helm install piaware piaware/piaware \
  --set flightaware.feederId=YOUR_FEEDER_ID \
  --set receiver.type=relay \
  --set relay.beastHost=your-receiver-host \
  --set relay.beastPort=30005
```

## Uninstallation

```bash
helm uninstall piaware -n piaware
```

## License

This Helm chart is provided as-is. The PiAware software is developed by FlightAware. The container image is maintained by [sdr-enthusiasts](https://github.com/sdr-enthusiasts).
