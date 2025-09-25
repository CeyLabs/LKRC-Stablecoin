# Governance Stakeholder Roles

LKRC stablecoin governance depends on a coordinated set of stakeholders. Each role carries explicit permissions, decision thresholds, and communication responsibilities to ensure safe operation of the protocol.

## Stakeholder Overview

| Stakeholder | Core Responsibilities | Key Permissions | Decision Thresholds | Communication Channels |
| --- | --- | --- | --- | --- |
| Governance Council | Strategic oversight, policy setting, emergency actions | Upgrade governance parameters, ratify new partners, authorize emergency circuit-breakers | Supermajority (≥ 67%) for upgrades or partner onboarding, simple majority for routine policy updates | Multi-sig forum, weekly governance sync, incident bridge (Signal/Matrix) |
| Partner Banks | Fiat liquidity provisioning, customer KYC/AML | Mint/burn requests within assigned credit lines, propose liquidity adjustments | Dual approval with custodian for mint/burn above daily limit, unanimous consent of partner banks for shared risk pools | Secure partner portal, monthly treasury calls |
| Custodians | Safekeeping of fiat reserves and token collateral | Freeze/unfreeze accounts under compliance flags, attest reserve balances | Joint approval with governance council for asset reallocations, 24h SLA for freeze requests | Custodian dashboard, incident hotline |
| Auditors | Independent verification of reserves and controls | Schedule on-chain/off-chain audits, access read-only financial records | Majority vote to escalate discrepancies; quorum (≥ 3 auditors) required for publishing reports | Audit workspace (secure data room), quarterly audit briefings |

## RACI Matrix

The following RACI matrix clarifies accountability for critical governance activities.

| Activity | Governance Council | Partner Banks | Custodians | Auditors |
| --- | --- | --- | --- | --- |
| Set monetary policy parameters | **A** | C | C | I |
| Onboard new partner bank | **A/R** | **R** | C | I |
| Approve mint request > credit line | **A/R** | **R** | **C** | I |
| Execute emergency freeze | **A/R** | I | **R** | C |
| Publish quarterly reserve report | C | C | **R** | **A** |
| Smart-contract upgrade | **A/R** | C | C | I |

**Legend:** **R** = Responsible, **A** = Accountable, **C** = Consulted, **I** = Informed.

## Communication Cadence

- **Governance Council:** weekly governance sync for parameter review, ad-hoc emergency bridge for incident response, quarterly town halls with community observers.
- **Partner Banks:** monthly treasury operations call with council representatives, daily alerts from liquidity monitoring bots, ticketing system for mint/burn requests.
- **Custodians:** bi-weekly custody reconciliation meeting with council compliance subcommittee, immediate incident hotline for security breaches.
- **Auditors:** quarterly audit briefings, secure data room updates in real time, annual joint presentation of audit outcomes to broader stakeholders.

## Decision Threshold Notes

- Supermajority thresholds rely on a 7-of-10 governance multi-signature wallet implemented in the `GovernanceCouncil` smart contract.
- Dual approvals for partner banks and custodians leverage the `MintingController` contract's requirement for both `PartnerBankSigner` and `CustodianSigner` signatures on high-value transactions.
- Auditor escalation requires an on-chain vote recorded in the `AuditRegistry` contract before public release of discrepancies.
