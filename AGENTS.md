# AGENTS.md - Hekara Contributor Guidelines

> **Welcome to Hekara development.** This document outlines the standards, procedures, and expectations for contributors and maintainers. Read this before opening a pull request.

```yaml
project: hekara
language: Go
domain: distributed infrastructure & network-path observability
target_users: SREs, network engineers, DevOps teams, cloud infrastructure
license: APACHE 2.0
```

---

## 1. Development Loop Engineering

### Local Development Cycle

```text
WRITE CODE
   |
   v
GO TEST    ───┬──────────────────────┐
   |           │                      │
   v           v                      v
  PASS        FAIL                 FIX BUG
   |           │                      |
   └───────┬───┘                      |
           v                          v
   REVIEW ───► COMMIT ───► CI/CD ───► DEPLOY
```

### CI/CD Pipeline

The CI/CD pipeline enforces quality gates on every PR:

| Stage                | Tool/Command                          | Pass Criteria                          |
|----------------------|---------------------------------------|----------------------------------------|
| **Lint**             | `golangci-lint run`                   | Zero new issues                        |
| **Unit Tests**       | `go test ./...`                       | 100% coverage on new code              |
| **Build**            | `go build ./...`                      | Compiles without errors                |
| **Mermaid Lint**     | `npx @mermaid-js/lint`                | Diagram syntax valid                   |
| **Security Scan**    | `govulncheck ./...`                   | No known vulnerabilities               |
| **Integration Tests**| `go test -tags=integration ./...`     | All integration tests pass             |

### Pre-commit Hooks

```bash
# Install pre-commit hooks
pre-commit install

# Run on demand
pre-commit run --all-files
```

Hooks include: `gofmt`, `golangci-lint`, `vulncheck`, `mermaid-lint`.

---

## 2. Testing Strategy

### Unit Tests

All functions must have unit tests following these conventions:

```go
// go test -run TestFunctionName -v
func TestAgent_Discover_CollectsAllFields(t *testing.T) {
    // Arrange
    agent := NewAgent("test-host", "linux")
    
    // Act
    discovered := agent.Discover()
    
    // Assert
    assert.Contains(t, discovered.OS.Family, "linux")
    assert.Contains(t, discovered.Arch, "amd64")
    assert.Contains(t, discovered.IPv4, "127.0.0.1")
}
```

### Coverage Requirements

- **New code**: 100% test coverage required
- **Existing code**: Maintain current coverage, no reduction allowed
- **Critical paths** (agent discovery, measurement, correlation): 100% coverage

### Integration Tests

Integration tests require a test infrastructure:

```bash
# Start test infrastructure
docker-compose -f test-infra.yml up -d

# Run integration tests
go test -tags=integration ./agent/... ./backend/...

# Tear down
docker-compose -f test-infra.yml down
```

Integration test scenarios:
- Agent registration and heartbeat
- Probe scheduling and execution
- Topology build from multiple agents
- Path change detection
- VPN detection and overlay representation

### Test Data Management

- Test fixtures stored in `internal/test/fixtures/`
- Fixtures include: agent inventories, measurement records, topology graphs, diagnosis results
- Fixtures versioned with the code; update when behavior changes

---

## 3. Code Standards

### Go Conventions

```bash
# Format code
gofmt -w .

# Vet code
go vet ./...

# Lint code
golangci-lint run
```

### Naming Conventions

- **Packages**: Lowercase, single-word names where possible
- **Functions**: camelCase, descriptive verbs (Discover, Measure, Correlate, Diagnose)
- **Types**: PascalCase, descriptive nouns (Agent, Topology, Path, Diagnosis)
- **Variables**: camelCase, concise but meaningful
- **Constants**: UPPER_SNAKE_CASE with `HEKARA_` prefix for configuration

### Error Handling

```go
// Always check errors, don't ignore them
func Discover() (Result, error) {
    // ... implementation
    if err != nil {
        return Result{}, fmt.Errorf("discovery failed: %w", err)
    }
    return Result{...}, nil
}
```

### No Silent Failures

- All errors must be propagated or properly logged
- Use sentinel errors for common failure patterns
- Include context in error messages (operation, component, inputs)

---

## 4. Component-Specific Guidelines

### Agent Development

```text
AGENT DEVELOPMENT CYCLE
1. Local Discovery → Implement platform-specific collectors
2. Measurements → Add ICMP/UDP/TCP probe support
3. Identity → Ensure stable identity across restarts
4. Telemetry → Send observations via gRPC
5. Configuration → Support probe policies and authorization
6. Upgrade → Hot-upgrade without service disruption
```

### Backend (Control Plane) Development

```text
BACKEND DEVELOPMENT CYCLE
1. API Gateway → REST/gRPC endpoints with authentication
2. Agent Manager → Registration, heartbeat, config delivery
3. Topology Engine → Graph build from agent observations
4. Probe Scheduler → Distribute measurements, rate limiting
5. Correlation Engine → Cross-agent fault domain identification
6. Diagnosis Engine → Confidence-based fault identification
7. Alert Manager → Deduplication, grouping, suppression
```

### CLI Development

```text
CLI DEVELOPMENT CYCLE
1. Commands → Follow established command structure
2. Output → Human-readable with JSON flag support
3. Error Handling → Clear error messages, exit codes
4. Autocomplete → bash completion support
5. Config → Environment variables and config file support
```

---

## 5. Contribution Workflow

### Pull Request Process

1. **Fork** the repository
2. **Branch** from `main` with descriptive name: `feature/agent-discovery`, `fix/correlation-engine`, etc.
3. **Write code** following these guidelines
4. **Write tests** with at least 100% coverage on new code
5. **Run local CI** (`go test`, `golangci-lint`, build)
6. **Update PLAN.md** if behavior changes affect architecture
7. **Open PR** with:
   - Clear title following Conventional Commits
   - Description of what and why
   - Testing performed
   - Screenshots of mermaid diagram changes
   - Checklist completion

### Conventional Commits

```
feat: add ICMP probe support
fix: correct gateway IP in discovery
docs: update AGENTS.md with testing guidelines
refactor: simplify topology engine interface
chore: update dependencies
```

### Review Requirements

PRs require:
- ✅ All tests passing
- ✅ `golangci-lint` zero issues
- ✅ `go build` successful
- ✅ Mermaid diagrams valid
- ✅ At least one maintainer approval
- ✅ No merge conflicts
- ✅ CHANGELOG updated (if applicable)

---

## 6. Release Process

### Versioning

Following [SemVer](https://semver.org/): `MAJOR.MINOR.PATCH`

- **MAJOR**: Breaking API changes
- **MINOR**: New features, backward-compatible
- **PATCH**: Bug fixes, backward-compatible

### Release Checklist

```bash
# 1. Update version in go.mod and PLAN.md
go mod edit -v=vX.Y.Z

# 2. Update CHANGELOG.md
# (follow conventional changelog format)

# 3. Create release branch
git checkout -b release/vX.Y.Z

# 4. Run full test suite
go test ./... -count=1

# 5. Build binaries
go build -o hekara-agent ./agent/cmd/
go build -o hekara-backend ./backend/cmd/
go build -o hekara-cli ./cli/cmd/

# 6. Generate checksums
sha256sum hekara-agent hekara-backend hekara-cli > checksums.txt

# 7. Tag release
git tag vX.Y.Z
git push origin vX.Y.Z

# 8. Publish draft release on GitHub
# - Attach binaries
# - Summarize changes
# - Add installation instructions
```

---

## 7. Platform Tooling

### Development Environment

```bash
# Required tools
go version 1.23.x
pre-commit 3.x
golangci-lint 1.6.x
docker 24.x (for integration tests)
docker-compose 2.x
make (for common tasks)

# Optional but recommended
vscode with Go extension
delve (dlv) for debugging
prettier for config files
```

### Makefile Targets

Common tasks available via `make`:

```bash
make help          # Show all available targets
make test          # Run all unit tests
make lint          # Run golangci-lint
make vet           # Run go vet
make build         # Build all binaries
make docker-build  # Build Docker images
make generate      # Generate code/config
make diagrams      # Validate mermaid diagrams
make coverage      # Show test coverage report
```

Example Makefile entry:
```makefile
diagrams:
	@npx @mermaid-js/lint README.md
	@npx @mermaid-js/lint PLAN.md
```

---

## 8. Monitoring & Observability of the Project Itself

### Health Checks

The project includes self-monitoring:

```bash
# Check code quality score
make lint | tail -5

# Check test coverage
make coverage | tail -3

# Validate all mermaid diagrams
make diagrams

# Run security scan
govulncheck ./... | grep -i "high\|critical"
```

### Debugging Aids

```bash
# Verbose test output
go test -v ./...

# Race detector
go test -race ./...

# CPU profiling
go test -cpuprofile cpu.prof ./...
go tool pprof -http=:8080 cpu.prof

# Memory profiling
go test -memprofile mem.prof ./...
go tool pprof -http=:8080 mem.prof
```

---

## 9. Senior Engineer Notes

### Common Pitfalls to Avoid

1. **Assuming healthy control connection = healthy measured path**
   - Control plane TLS connection ≠ application path health
   - Always measure actual network behavior

2. **ICMP failure ≠ network failure**
   - ICMP may be blocked, rate-limited, or deprioritized
   - Consider multiple evidence sources before concluding failure

3. **Don't invent topology**
   - If relationship unknown, mark as `UNKNOWN` or `INFERRED`
   - Incomplete topology is better than false topology

4. **Mesh testing becomes expensive**
   - N agents → N*(N-1) relationships
   - Use probe groups, schedules, critical paths instead

5. **VPN detection requires multiple evidence**
   - Interface type alone is insufficient
   - Use: peer keys, endpoints, routes, tunnel addresses, handshakes

### Performance Considerations

- Agents: Low resource usage, minimal footprint
- Backend: Designed for horizontal scaling
- CLI: Fast startup, cached where possible
- Migrations: Use online schema changes where possible

### Security by Design

- TLS everywhere for control plane
- Probe authorization policies
- Rate-limited measurements
- No arbitrary remote execution
- Signed releases
- Least privilege principles

---

## 10. Quick Start for New Contributors

```bash
# 1. Clone the repository
git clone https://github.com/anomalyco/hekara.git
cd hekara

# 2. Install dependencies
go mod download

# 3. Install pre-commit hooks
pre-commit install

# 4. Run the full test suite
make test

# 5. Validate diagrams
make diagrams

# 6. Make your first change
# - Write code
# - Write tests
# - Run make lint
# - Open PR

# 7. For local development
make run-agent    # Run agent in dev mode
make run-backend  # Run control plane in dev mode
make run-cli      # Run CLI in dev mode
```

---

*This document is maintained as code. Updates follow the same review process as source code changes. Last updated: `git log -1 --format=%ci`.*

*For questions, discuss in the [GitHub Discussions](https://github.com/anomalyco/hekara/discussions) or reach out to maintainers.*