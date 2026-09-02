# hekara backend

The Hekara backend (control plane) is the central component that orchestrates the distributed infrastructure observability platform.

## Core Components

- **Agent Manager**: Handles agent registration, authentication, heartbeat monitoring, and configuration delivery
- **Topology Engine**: Builds and maintains the live infrastructure topology graph from agent observations and active measurements
- **Probe Scheduler**: Schedules and distributes network measurement probes across agents
- **Metrics/Events Engine**: Collects, stores, and processes metrics and event streams from all agents
- **Correlation Engine**: Correlates observations from multiple sources to identify probable fault domains
- **Diagnosis Engine**: Provides evidence-based diagnosis with confidence scoring and explanations
- **Alert Manager**: Manages alerting with deduplication, grouping, suppression, and cooldown policies

## Functions

- Agent registration and authentication via gRPC + Protobuf over TLS
- Secure control channel communication
- Inventory management and site mapping
- Probe scheduling and authorization
- Path history tracking and change detection
- Confidence-based diagnosis with evidence
- Alert generation, grouping, and suppression

## Technology

Implemented in Go for high performance, cross-platform support, and static binary deployment.

## License

[APACHE 2.0](./../../LICENSE)