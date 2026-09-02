# hekara agent

The Hekara agent is a lightweight, cross-platform component deployed across network infrastructure including servers, VMs, containers, and Kubernetes nodes.

## Responsibilities

- **Local Discovery**: Collects hostname, OS, kernel, architecture, network interfaces, interface state, IP addresses, routing table, DNS configuration, and Kubernetes information
- **Active Measurements**: Performs controlled network probes using ICMP, UDP, and TCP to measure connectivity, latency, packet loss, and reachability
- **Telemetry**: Sends observations and measurements to the Hekara controller via secure gRPC connection
- **Identity**: Maintains stable agent identity surviving restarts with hostname, site, environment, and role metadata

## Features

- Lightweight with low resource usage
- Secure with TLS-encrypted control channel
- Cross-platform (Linux, Windows)
- Easy to install and upgrade
- Does not become a complete monitoring suite

## Installation

Please refer to the main [hekara README](../README.md) for overall project information.

## License

[APACHE 2.0](./../../LICENSE)