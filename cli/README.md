# hekara cli

The hekara CLI is the command-line interface for interacting with the Hekara observability platform.

## Commands

### Agent Operations

- `hekara agent status` - Check agent status and connectivity
- `hekara agent info` - Display agent information and identity

### Network Probes

- `hekara ping <target>` - Test ICMP reachability to a target
- `hekara probe --source <agent> --destination <agent>` - Run network probes between agents
- `hekara trace <target>` - Trace network path to a target

### Diagnosis

- `hekara diagnose --source <agent> --destination <target>` - Diagnote connectivity issues between source and destination
- `hekara watch <target>` - Continuously monitor a target for changes
- `hekara path <source> <destination>` - Display path information between source and destination

### Topology & VPN

- `hekara topology` - View the infrastructure topology graph
- `hekara topology neighbors <agent>` - Show agent neighbors in the topology
- `hekara vpn <agent>` - VPN status and tunnel information for an agent
- `hekara route <agent> <destination>` - Route analysis to a destination from an agent

### Diff & History

- `hekara diff <path-id>` - Compare path observations over time
- `hekara incidents` - List active incidents and issues

## Usage Examples

```bash
hekara diagnose --source app01 --destination api01

hekara probe \
  --source agent-a \
  --destination agent-b

hekara topology

hekara vpn <agent>
```

## Design Principles

- Simple and intuitive command structure
- Output focused on actionable information
- Supports JSON output for pipeline integration
- Designed for both interactive use and scripting

## License

[APACHE 2.0](./../../LICENSE)
