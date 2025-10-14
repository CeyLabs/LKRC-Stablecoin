# LKRC Stablecoin Token

LKRC is an Ethereum-based ERC20 stablecoin that layers pausability, blacklist enforcement, and controlled mint/burn operations on top of OpenZeppelin security primitives.

## Documentation

- [Documentation Index](docs/index.md)
- [Getting Started Guide](docs/getting-started.md)
- [Architecture: Contract Components](docs/architecture/contract-components.md)
- [Operations: Token Lifecycle](docs/operations/token-lifecycle.md)
- [Compliance: Security Controls](docs/compliance/security-controls.md)

### Persona-Specific Use Cases

- [Stablecoin Issuer Journey](docs/use-cases/stablecoin-issuer.md)
- [Bank Integration Playbook](docs/use-cases/bank-integration.md)
- [Enterprise Treasury Runbook](docs/use-cases/enterprise-treasury.md)

## Quick Start

```bash
npm install
npx hardhat compile
```

The full smart contract implementation lives in [`LKRC.sol`](LKRC.sol).
