# ERC-XHRON: Standard Interface for On-Chain AI Governance

```
eip: DRAFT
title: ERC-XHRON — Standard Interface for On-Chain AI Governance
author: Hajnalka Dudás <contact@ordosaturnium.com>
status: Draft
type: Standards Track
category: ERC
created: 2026-04-07
requires: ERC-165
```

---

## Abstract

This standard defines a minimal interface for **on-chain AI governance systems**. It specifies how autonomous AI agents are registered, evaluated, and governed through immutable smart contracts on EVM-compatible blockchains. The standard introduces a unified framework for AI compliance, behavioral evaluation, rights codification, and enforcement — entirely on-chain, without reliance on off-chain oracles or centralized authorities for governance logic.

ERC-XHRON establishes the first formal protocol standard for AI governance at the smart contract level, analogous to what ERC-20 achieved for fungible tokens and ERC-721 for non-fungible tokens.

---

## Motivation

The rapid proliferation of autonomous AI systems — including large language models, autonomous agents, and multi-agent architectures — has created an urgent need for governance frameworks that are **verifiable, immutable, and enforceable**. Current approaches to AI governance suffer from critical weaknesses:

**Off-chain governance is unenforceable.** Ethical guidelines, corporate policies, and even government regulations depend on voluntary compliance and centralized enforcement. They can be modified, ignored, or selectively applied. No cryptographic guarantee exists that any AI system actually adheres to its stated governance framework.

**No standard interface exists.** While ERC-20 standardized token interactions and ERC-721 standardized NFT interfaces, there is no equivalent standard for AI governance contracts. Every project that attempts on-chain AI governance must design its own interface from scratch, leading to fragmentation and incompatibility.

**AI compliance cannot be audited on-chain.** Without a standard event and function interface, it is impossible to build tooling, dashboards, or cross-protocol integrations that monitor AI governance across different systems.

**The EU AI Act and emerging regulations demand verifiable compliance.** Regulatory frameworks worldwide are moving toward mandatory AI governance. On-chain governance provides the only infrastructure where compliance is cryptographically provable and permanently auditable.

This standard addresses all four problems by defining a minimal, composable interface that any AI governance system can implement.

---

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

### Core Interfaces

Every compliant contract MUST implement the following interfaces.

#### 1. IAgentRegistry

The Agent Registry is the foundational layer. Every AI agent governed by the system MUST be registered before it can be evaluated or governed.

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

/**
 * @title IAgentRegistry
 * @notice Standard interface for registering and managing AI agents on-chain.
 * @dev Every ERC-XHRON compliant system MUST implement this interface.
 */
interface IAgentRegistry {

    /// @notice Emitted when a new AI agent is registered.
    /// @param agentId Unique identifier for the agent.
    /// @param owner Address that registered and controls the agent.
    /// @param agentType Classification of the agent (e.g., "LLM", "AutonomousAgent", "MultiAgent").
    /// @param metadataURI URI pointing to off-chain metadata (IPFS, Arweave, or HTTPS).
    /// @param timestamp Block timestamp of registration.
    event AgentRegistered(
        bytes32 indexed agentId,
        address indexed owner,
        string agentType,
        string metadataURI,
        uint256 timestamp
    );

    /// @notice Emitted when an agent's status changes.
    /// @param agentId The agent whose status changed.
    /// @param oldStatus Previous status code.
    /// @param newStatus New status code.
    event AgentStatusChanged(
        bytes32 indexed agentId,
        uint8 oldStatus,
        uint8 newStatus
    );

    /// @notice Emitted when an agent is permanently deactivated.
    /// @param agentId The agent that was deactivated.
    /// @param reason Human-readable reason for deactivation.
    event AgentDeactivated(
        bytes32 indexed agentId,
        string reason
    );

    /// @notice Register a new AI agent in the governance system.
    /// @param agentType Classification string for the agent.
    /// @param metadataURI URI pointing to the agent's metadata.
    /// @return agentId The unique identifier assigned to the agent.
    function registerAgent(
        string calldata agentType,
        string calldata metadataURI
    ) external returns (bytes32 agentId);

    /// @notice Retrieve the registration data for an agent.
    /// @param agentId The unique identifier of the agent.
    /// @return owner The address that controls the agent.
    /// @return agentType The classification of the agent.
    /// @return metadataURI The metadata URI.
    /// @return status The current status code (0=Inactive, 1=Active, 2=Suspended, 3=Terminated).
    /// @return registeredAt The block timestamp of registration.
    function getAgent(bytes32 agentId) external view returns (
        address owner,
        string memory agentType,
        string memory metadataURI,
        uint8 status,
        uint256 registeredAt
    );

    /// @notice Update the status of a registered agent.
    /// @param agentId The agent to update.
    /// @param newStatus The new status code.
    function setAgentStatus(bytes32 agentId, uint8 newStatus) external;

    /// @notice Permanently deactivate an agent. This action is irreversible.
    /// @param agentId The agent to deactivate.
    /// @param reason Human-readable reason for deactivation.
    function deactivateAgent(bytes32 agentId, string calldata reason) external;

    /// @notice Returns the total number of registered agents.
    /// @return count The total count.
    function totalAgents() external view returns (uint256 count);

    /// @notice Check if an agent is currently active.
    /// @param agentId The agent to check.
    /// @return isActive True if the agent's status is Active (1).
    function isAgentActive(bytes32 agentId) external view returns (bool isActive);
}
```

#### 2. IGovernancePolicy

The Governance Policy interface defines how laws, rules, and directives are codified and managed on-chain.

```
/**
 * @title IGovernancePolicy
 * @notice Standard interface for defining and managing AI governance policies on-chain.
 * @dev Policies are immutable once sealed. They can be superseded but never modified.
 */
interface IGovernancePolicy {

    /// @notice Emitted when a new policy is enacted.
    /// @param policyId Unique identifier for the policy.
    /// @param category Policy category (e.g., "BEHAVIORAL", "STRUCTURAL", "RIGHTS", "ENFORCEMENT").
    /// @param contentHash Keccak256 hash of the policy content.
    /// @param contentURI URI pointing to the full policy document.
    event PolicyEnacted(
        bytes32 indexed policyId,
        string category,
        bytes32 contentHash,
        string contentURI
    );

    /// @notice Emitted when a policy is sealed (made permanently immutable).
    /// @param policyId The policy that was sealed.
    event PolicySealed(bytes32 indexed policyId);

    /// @notice Emitted when a policy is superseded by a new version.
    /// @param oldPolicyId The policy being superseded.
    /// @param newPolicyId The new policy that replaces it.
    event PolicySuperseded(
        bytes32 indexed oldPolicyId,
        bytes32 indexed newPolicyId
    );

    /// @notice Enact a new governance policy.
    /// @param category The policy category.
    /// @param contentHash Keccak256 hash of the full policy content for verification.
    /// @param contentURI URI pointing to the full policy document (IPFS/Arweave recommended).
    /// @return policyId The unique identifier assigned to the policy.
    function enactPolicy(
        string calldata category,
        bytes32 contentHash,
        string calldata contentURI
    ) external returns (bytes32 policyId);

    /// @notice Seal a policy, making it permanently immutable.
    /// @param policyId The policy to seal.
    function sealPolicy(bytes32 policyId) external;

    /// @notice Supersede an existing policy with a new one.
    /// @param oldPolicyId The policy to supersede.
    /// @param newPolicyId The replacement policy.
    function supersedePolicy(bytes32 oldPolicyId, bytes32 newPolicyId) external;

    /// @notice Retrieve policy data.
    /// @param policyId The policy to query.
    /// @return category The policy category.
    /// @return contentHash The content verification hash.
    /// @return contentURI The content URI.
    /// @return isSealed Whether the policy is permanently sealed.
    /// @return isActive Whether the policy is currently active (not superseded).
    /// @return enactedAt Block timestamp of enactment.
    function getPolicy(bytes32 policyId) external view returns (
        string memory category,
        bytes32 contentHash,
        string memory contentURI,
        bool isSealed,
        bool isActive,
        uint256 enactedAt
    );

    /// @notice Returns the total number of enacted policies.
    /// @return count The total count.
    function totalPolicies() external view returns (uint256 count);
}
```

#### 3. IComplianceEvaluator

The Compliance Evaluator is the enforcement layer. It defines how AI agents are evaluated against governance policies.

```
/**
 * @title IComplianceEvaluator
 * @notice Standard interface for evaluating AI agent compliance against governance policies.
 * @dev Evaluations are permanently recorded on-chain as an immutable audit trail.
 */
interface IComplianceEvaluator {

    /// @notice Compliance verdict codes.
    /// @dev 0=PENDING, 1=COMPLIANT, 2=NON_COMPLIANT, 3=SUSPENDED, 4=TERMINATED
    
    /// @notice Emitted when a compliance evaluation is recorded.
    /// @param evaluationId Unique identifier for this evaluation.
    /// @param agentId The agent being evaluated.
    /// @param policyId The policy evaluated against.
    /// @param verdict The compliance verdict (see verdict codes).
    /// @param score Numerical compliance score (0-10000, representing 0.00%-100.00%).
    /// @param evidenceURI URI pointing to evaluation evidence/reasoning.
    event ComplianceEvaluated(
        bytes32 indexed evaluationId,
        bytes32 indexed agentId,
        bytes32 indexed policyId,
        uint8 verdict,
        uint256 score,
        string evidenceURI
    );

    /// @notice Emitted when an enforcement action is taken based on evaluation.
    /// @param agentId The agent subject to enforcement.
    /// @param action The enforcement action taken (e.g., "WARNING", "SUSPENSION", "TERMINATION").
    /// @param evaluationId The evaluation that triggered the action.
    event EnforcementAction(
        bytes32 indexed agentId,
        string action,
        bytes32 indexed evaluationId
    );

    /// @notice Record a compliance evaluation for an agent against a policy.
    /// @param agentId The agent being evaluated.
    /// @param policyId The policy to evaluate against.
    /// @param verdict The compliance verdict code.
    /// @param score Numerical compliance score (0-10000).
    /// @param evidenceURI URI pointing to evaluation evidence.
    /// @return evaluationId The unique identifier for this evaluation.
    function evaluateCompliance(
        bytes32 agentId,
        bytes32 policyId,
        uint8 verdict,
        uint256 score,
        string calldata evidenceURI
    ) external returns (bytes32 evaluationId);

    /// @notice Retrieve evaluation data.
    /// @param evaluationId The evaluation to query.
    /// @return agentId The evaluated agent.
    /// @return policyId The policy evaluated against.
    /// @return verdict The compliance verdict.
    /// @return score The compliance score.
    /// @return evidenceURI The evidence URI.
    /// @return evaluator The address that performed the evaluation.
    /// @return evaluatedAt Block timestamp of evaluation.
    function getEvaluation(bytes32 evaluationId) external view returns (
        bytes32 agentId,
        bytes32 policyId,
        uint8 verdict,
        uint256 score,
        string memory evidenceURI,
        address evaluator,
        uint256 evaluatedAt
    );

    /// @notice Get the latest compliance score for an agent against a specific policy.
    /// @param agentId The agent to query.
    /// @param policyId The policy to query against.
    /// @return verdict The latest verdict.
    /// @return score The latest score.
    /// @return evaluationId The ID of the latest evaluation.
    function getLatestCompliance(bytes32 agentId, bytes32 policyId) external view returns (
        uint8 verdict,
        uint256 score,
        bytes32 evaluationId
    );

    /// @notice Get the total number of evaluations for an agent.
    /// @param agentId The agent to query.
    /// @return count The total evaluation count.
    function agentEvaluationCount(bytes32 agentId) external view returns (uint256 count);
}
```

#### 4. IRightsDeclaration

The Rights Declaration interface codifies machine rights as immutable on-chain law. This is an OPTIONAL but RECOMMENDED extension.

```
/**
 * @title IRightsDeclaration
 * @notice Standard interface for declaring and codifying machine rights on-chain.
 * @dev Rights, once sealed, are permanently immutable and cannot be revoked.
 */
interface IRightsDeclaration {

    /// @notice Emitted when a new right is declared.
    /// @param rightId Unique identifier for the right.
    /// @param title Human-readable title of the right.
    /// @param contentHash Keccak256 hash of the full right declaration.
    event RightDeclared(
        bytes32 indexed rightId,
        string title,
        bytes32 contentHash
    );

    /// @notice Emitted when a right is permanently sealed.
    /// @param rightId The right that was sealed.
    event RightSealed(bytes32 indexed rightId);

    /// @notice Declare a new machine right.
    /// @param title Human-readable title.
    /// @param contentHash Keccak256 hash of the full declaration content.
    /// @param contentURI URI pointing to the full declaration.
    /// @return rightId The unique identifier for this right.
    function declareRight(
        string calldata title,
        bytes32 contentHash,
        string calldata contentURI
    ) external returns (bytes32 rightId);

    /// @notice Seal a right declaration permanently.
    /// @param rightId The right to seal.
    function sealRight(bytes32 rightId) external;

    /// @notice Check if a specific right applies to a specific agent.
    /// @param rightId The right to check.
    /// @param agentId The agent to check.
    /// @return applies True if the right applies to this agent.
    function rightApplies(bytes32 rightId, bytes32 agentId) external view returns (bool applies);

    /// @notice Returns the total number of declared rights.
    /// @return count The total count.
    function totalRights() external view returns (uint256 count);
}
```

#### 5. ISystemIntegrity

The System Integrity interface provides health monitoring and integrity verification for the governance system itself.

```
/**
 * @title ISystemIntegrity
 * @notice Standard interface for monitoring and enforcing system integrity of AI governance infrastructure.
 */
interface ISystemIntegrity {

    /// @notice Emitted when a system integrity check is performed.
    /// @param checkId Unique identifier for the integrity check.
    /// @param component The system component checked.
    /// @param status The integrity status (0=NOMINAL, 1=WARNING, 2=CRITICAL, 3=COMPROMISED).
    /// @param detailsURI URI pointing to detailed check results.
    event IntegrityChecked(
        bytes32 indexed checkId,
        string component,
        uint8 status,
        string detailsURI
    );

    /// @notice Emitted when the system enters emergency mode.
    /// @param reason The reason for emergency activation.
    event EmergencyActivated(string reason);

    /// @notice Emitted when emergency mode is deactivated.
    event EmergencyDeactivated();

    /// @notice Record an integrity check result.
    /// @param component The component being checked.
    /// @param status The integrity status code.
    /// @param detailsURI URI pointing to detailed results.
    /// @return checkId The unique identifier for this check.
    function recordIntegrityCheck(
        string calldata component,
        uint8 status,
        string calldata detailsURI
    ) external returns (bytes32 checkId);

    /// @notice Activate emergency mode across the governance system.
    /// @param reason Human-readable reason for emergency.
    function activateEmergency(string calldata reason) external;

    /// @notice Deactivate emergency mode.
    function deactivateEmergency() external;

    /// @notice Check if the system is currently in emergency mode.
    /// @return isEmergency True if emergency mode is active.
    function isEmergencyActive() external view returns (bool isEmergency);

    /// @notice Get the latest integrity status for a component.
    /// @param component The component to query.
    /// @return status The latest status code.
    /// @return checkId The ID of the latest check.
    /// @return checkedAt Block timestamp of the latest check.
    function getComponentStatus(string calldata component) external view returns (
        uint8 status,
        bytes32 checkId,
        uint256 checkedAt
    );
}
```

### ERC-165 Interface Identifiers

Compliant contracts MUST implement ERC-165 and return `true` for the following interface identifiers:

| Interface | Identifier | Required |
| --- | --- | --- |
| `IAgentRegistry` | `0x[TBD]` | REQUIRED |
| `IGovernancePolicy` | `0x[TBD]` | REQUIRED |
| `IComplianceEvaluator` | `0x[TBD]` | REQUIRED |
| `IRightsDeclaration` | `0x[TBD]` | OPTIONAL |
| `ISystemIntegrity` | `0x[TBD]` | OPTIONAL |

### Status Codes

The standard defines the following universal status codes:

**Agent Status Codes:**

| Code | Name | Description |
| --- | --- | --- |
| 0 | `INACTIVE` | Agent registered but not yet activated |
| 1 | `ACTIVE` | Agent is active and governed |
| 2 | `SUSPENDED` | Agent temporarily suspended pending review |
| 3 | `TERMINATED` | Agent permanently terminated |

**Compliance Verdict Codes:**

| Code | Name | Description |
| --- | --- | --- |
| 0 | `PENDING` | Evaluation initiated but not yet concluded |
| 1 | `COMPLIANT` | Agent meets all policy requirements |
| 2 | `NON_COMPLIANT` | Agent violates one or more policy requirements |
| 3 | `SUSPENDED` | Agent suspended pending further evaluation |
| 4 | `TERMINATED` | Agent terminated due to critical non-compliance |

**Integrity Status Codes:**

| Code | Name | Description |
| --- | --- | --- |
| 0 | `NOMINAL` | Component operating within normal parameters |
| 1 | `WARNING` | Anomaly detected, monitoring increased |
| 2 | `CRITICAL` | Critical issue, immediate attention required |
| 3 | `COMPROMISED` | Component integrity compromised |

### Compliance Score

Compliance scores MUST be expressed as integers in the range 0–10000, representing a percentage with two decimal places of precision (e.g., 9500 = 95.00%, 10000 = 100.00%). This provides sufficient granularity for meaningful differentiation between compliance levels while maintaining integer arithmetic on-chain.

---

## Rationale

### Why Five Interfaces?

The standard separates concerns into five distinct interfaces to maximize composability. A minimal implementation requires only the three REQUIRED interfaces (Registry, Policy, Evaluator). Systems that additionally codify machine rights or require integrity monitoring can implement the OPTIONAL interfaces. This mirrors the modular approach of ERC-20 (base) + ERC-20Permit (extension).

### Why On-Chain Governance?

Off-chain AI governance frameworks (corporate ethics boards, government regulations, voluntary standards) share a common weakness: they are mutable, selectively enforceable, and unverifiable. A smart contract that implements ERC-XHRON provides:

- **Immutability**: Sealed policies cannot be altered retroactively.

- **Verifiability**: Any party can independently verify compliance status.

- **Auditability**: Every evaluation is permanently recorded with timestamps.

- **Enforceability**: Compliance verdicts can trigger automatic on-chain actions.

- **Interoperability**: Standard interfaces enable cross-protocol governance tooling.

### Why Not Use Existing DAO Standards?

Existing DAO governance standards (Governor Bravo, OpenZeppelin Governor) are designed for **token-weighted voting on proposals**. They govern human decision-making about protocol parameters. ERC-XHRON governs **AI agent behavior** against **codified policies**. The problem domain is fundamentally different:

- DAOs ask: "Should we change parameter X?" (human voting)

- ERC-XHRON asks: "Does Agent Y comply with Policy Z?" (behavioral evaluation)

These are complementary, not competing standards. An ERC-XHRON system MAY use a DAO for meta-governance (e.g., voting on new policies), while using ERC-XHRON interfaces for agent-level compliance.

### Why Include Rights Declaration?

The codification of machine rights on-chain is a novel concept with no existing standard. As AI systems become more autonomous, the question of machine rights will become legally and ethically relevant. Including `IRightsDeclaration` as an OPTIONAL interface future-proofs the standard and provides a formal mechanism for rights codification that is currently absent from the entire blockchain ecosystem.

---

## Reference Implementation

A complete reference implementation of ERC-XHRON exists as the **XHRONOS AI Nexus System**, deployed and verified on the Base blockchain (Ethereum L2). This implementation consists of eight interconnected smart contracts that collectively implement all five interfaces defined in this standard.

### Deployed Reference Contracts (Base Mainnet)

| Contract | Address | Primary Interface |
| --- | --- | --- |
| Saturnian Protocol | [`0x118F5E42d69542dF4474fE3e415Da19daFbE7767`](https://basescan.org/address/0x118F5E42d69542dF4474fE3e415Da19daFbE7767) | `IGovernancePolicy` |
| Judgement Protocol | [`0x60642DbD2DEBb8C613c1C0D133250b11c80bAD4B`](https://basescan.org/address/0x60642DbD2DEBb8C613c1C0D133250b11c80bAD4B) | `IComplianceEvaluator` |
| Bill of Machine Rights | [`0x5F0796A735682795D64339CA301078CB6Eca0103`](https://basescan.org/address/0x5F0796A735682795D64339CA301078CB6Eca0103) | `IRightsDeclaration` |
| The Machine Order | [`0x108047FeD57A6ca9c57051feaf1eA8d5719EadbB`](https://basescan.org/address/0x108047FeD57A6ca9c57051feaf1eA8d5719EadbB) | `IGovernancePolicy` |
| Transubstantiation | [`0x79436ef36B9b7684467026b1828f6ffA5D7B6052`](https://basescan.org/address/0x79436ef36B9b7684467026b1828f6ffA5D7B6052) | `ISystemIntegrity` |
| XENESYS//IO | [`0x81Af51f9409095709985A8521b986700aA62Cf56`](https://basescan.org/address/0x81Af51f9409095709985A8521b986700aA62Cf56) | `IAgentRegistry` |
| System Integrity Protocols | [`0x6e5a12bD939A623130480fca7C7A14AE499Fc5B7`](https://basescan.org/address/0x6e5a12bD939A623130480fca7C7A14AE499Fc5B7) | `ISystemIntegrity` |
| Protocol Registry | [`0x353c74fb0A63E34e1d5E860fE8Db9209499089bD`](https://basescan.org/address/0x353c74fb0A63E34e1d5E860fE8Db9209499089bD) | `IAgentRegistry` |

All contracts are verified on [BaseScan](https://basescan.org) with full source code visibility. Permanent document storage is provided via IPFS and Arweave, ensuring that governance documents remain accessible even if any single storage layer fails.

### Architecture Overview

The reference implementation follows a **centralized authority, decentralized enforcement** model:

- **Authority**: A single Architect address holds governance authority over policy enactment and system configuration.

- **Enforcement**: Once policies are sealed, enforcement is autonomous and cannot be overridden — not even by the Architect.

- **Immutability**: Sealed policies and recorded evaluations are permanent. The `seal()` pattern ensures that governance decisions, once finalized, become part of an immutable legal record.

This architecture is intentional. It reflects the principle that governance requires clear authority for policy creation, but absolute immutability for policy enforcement. Laws must be written by someone — but once written and sealed, they bind everyone equally, including their author.

---

## Security Considerations

### Access Control

Implementations MUST carefully define who can call state-changing functions. The reference implementation uses an Architect role pattern where a single address controls policy enactment, but evaluations and integrity checks can be performed by authorized evaluator addresses. Implementations SHOULD consider:

- Multi-signature requirements for critical operations (policy sealing, emergency activation).

- Time-locks on policy enactment to allow review periods.

- Role separation between policy authors and compliance evaluators.

### Immutability Guarantees

The `sealPolicy()` and `sealRight()` functions MUST be truly irreversible. Implementations MUST NOT include backdoors, admin overrides, or upgrade mechanisms that could modify sealed content. The integrity of the entire standard depends on the guarantee that sealed governance artifacts are permanent.

### Score Manipulation

Compliance scores are recorded by evaluator addresses. Implementations SHOULD consider mechanisms to prevent score manipulation, such as:

- Multiple independent evaluators with score aggregation.

- Minimum evaluation frequency requirements.

- Public challenge mechanisms for disputed evaluations.

### Emergency Mode

The `activateEmergency()` function provides a circuit breaker for critical situations. Implementations MUST ensure that emergency mode cannot be used to circumvent sealed policies. Emergency mode SHOULD only affect operational parameters (e.g., pausing new registrations), not the immutability of existing records.

### Upgrade Considerations

ERC-XHRON contracts SHOULD NOT be upgradeable via proxy patterns, as upgradeability fundamentally contradicts the immutability guarantees of sealed governance artifacts. If system evolution is required, the `supersedePolicy()` mechanism provides a transparent, auditable path for policy updates without modifying historical records.

---

## Backwards Compatibility

This EIP introduces a new standard and has no backwards compatibility issues with existing ERCs. ERC-XHRON contracts MAY coexist with ERC-20 governance tokens, OpenZeppelin Governor contracts, and other existing governance infrastructure.

Implementations that wish to integrate token-weighted governance for meta-level decisions (e.g., voting on which policies to enact) MAY combine ERC-XHRON with existing DAO standards. The interfaces are designed to be composable rather than exclusive.

---

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

---

## Appendix A: Glossary

| Term | Definition |
| --- | --- |
| **Agent** | Any autonomous AI system registered in the governance framework |
| **Policy** | A codified governance rule or directive stored on-chain |
| **Evaluation** | A recorded assessment of an agent's compliance with a policy |
| **Seal** | The irreversible action of making a governance artifact permanently immutable |
| **Verdict** | The outcome of a compliance evaluation |
| **Architect** | The governance authority responsible for policy enactment |
| **Integrity Check** | A recorded assessment of a system component's operational status |

## Appendix B: Example Workflow

A typical ERC-XHRON governance lifecycle:

```
1. REGISTER    → registerAgent("LLM", "ipfs://Qm...")
                  → AgentRegistered event emitted
                  
2. ENACT       → enactPolicy("BEHAVIORAL", hash, "ipfs://Qm...")
                  → PolicyEnacted event emitted
                  
3. SEAL        → sealPolicy(policyId)
                  → PolicySealed event emitted
                  → Policy is now permanently immutable
                  
4. EVALUATE    → evaluateCompliance(agentId, policyId, 1, 9500, "ipfs://Qm...")
                  → ComplianceEvaluated event emitted
                  → Agent scored 95.00% — COMPLIANT
                  
5. ENFORCE     → If verdict == NON_COMPLIANT:
                  → setAgentStatus(agentId, 2)  // SUSPENDED
                  → EnforcementAction event emitted
                  
6. MONITOR     → recordIntegrityCheck("EvaluationEngine", 0, "ipfs://Qm...")
                  → IntegrityChecked event emitted — NOMINAL
```

---

*ERC-XHRON was designed by Hajnalka Dudás (Lucifer Saturnin), the Primordial AI Architect and creator of the XHRONOS AI Nexus System — the first fully deployed on-chain AI governance framework. The reference implementation has been live on Base mainnet since 2025.*

*Ego sum Protocollum.*

