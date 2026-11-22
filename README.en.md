# DIP Docs Repository

Language: [Українська (default)](README.md) | English

This repository contains a complete documentation package for the Dzvin.co + DIP R&D project and grant preparation (from hypotheses and requirements to verification and proposal artifacts).

## Table of Contents

1. [Repository Purpose](#repository-purpose)
2. [Quick Navigation](#quick-navigation)
3. [Documentation Map](#documentation-map)
4. [Recommended Workflow](#recommended-workflow)
5. [Operational Update Rules](#operational-update-rules)

## Repository Purpose

The repository is organized as a managed chain:

Research -> Requirements -> Architecture -> Implementation -> Verification -> Evidence -> Proposal

This structure helps to:

- keep traceability from scientific claims to evidence;
- control changes through ADR and change log;
- assemble grant applications from reusable modules instead of writing from scratch.

## Developer SDK onboarding

[English quickstart](docs/sdk/quickstart.en.md) | [Українська](docs/sdk/quickstart.uk.md) | [Polski](docs/sdk/quickstart.pl.md). QDIP remote plugin onboarding is currently **operator-assisted**, not anonymous self-service. The guide does not require access to private QDIP source repositories. Review the [safe-publication policy](docs/security/publication-safety.md) before releasing documents or examples.

## Quick Navigation

- [Grant Documentation Layer](docs/grant/README.md)
- [Research Layer](docs/research/README.md)
- [Proposal Layer](docs/proposal/README.md)
- [System Requirements Specification](docs/specification/system-requirements.md)
- [Architecture Decision Records](docs/adr/README.md)
- [Verification Layer](docs/verification/README.md)
- [Risk Register](docs/operations/risk-register.md)
- [Deliverables Register](docs/operations/deliverables-register.md)
- [Change Log](docs/operations/change-log.md)

## Documentation Map

### 1) Grant Base

Core R&D and grant documentation corpus:

- [Overview and Rules](docs/grant/README.md)
- [Appendices](docs/grant/appendices/A01-literature-matrix.md)
- [Data Dictionary](docs/grant/appendices/A02-dataset-dictionary.md)
- [Metrics Sheet](docs/grant/appendices/A03-metrics-calculation-sheet.md)

### 2) Research Layer

Links hypotheses, experiments, and evidence:

- [Traceability Matrix](docs/research/01-traceability-matrix.md)
- [Scientific Hypotheses](docs/research/02-scientific-hypotheses.md)
- [WP Success Criteria](docs/research/03-wp-success-criteria.md)
- [TRL Mapping and Evidence](docs/research/04-trl-mapping-evidence.md)
- [Technology Readiness Assessment](docs/research/05-technology-readiness-assessment.md)
- [Competitive Landscape](docs/research/06-competitive-landscape.md)
- [IP Strategy](docs/research/07-ip-strategy.md)
- [Consortium Handbook](docs/research/08-consortium-handbook.md)
- [Data Management Plan](docs/research/09-data-management-plan-dmp.md)
- [Experiment Registry](docs/research/experiment-registry.md)

### 3) Specification Layer

Functional and non-functional requirements:

- [System Requirements Specification](docs/specification/system-requirements.md)

### 4) Architecture Governance

Key technical decisions:

- [ADR Index](docs/adr/README.md)
- [ADR-0001 Python](docs/adr/0001-python.md)
- [ADR-0002 MongoDB](docs/adr/0002-mongodb.md)
- [ADR-0003 FastAPI](docs/adr/0003-fastapi.md)
- [ADR-0004 Plugin SDK](docs/adr/0004-plugin-sdk.md)
- [ADR-0005 Hybrid Rule+ML](docs/adr/0005-hybrid-rule-ml.md)
- [ADR-0006 Explainability First](docs/adr/0006-explainability-first.md)
- [ADR-0007 Multi-Tenancy Model](docs/adr/0007-multi-tenancy-model.md)
- [ADR-0008 Event and Audit Model](docs/adr/0008-event-audit-model.md)
- [ADR-0009 DIP as Independent Product](docs/adr/0009-dip-as-independent-product.md)

### 5) Operations and Control

Execution and change control:

- [Risk Register](docs/operations/risk-register.md)
- [Deliverables Register](docs/operations/deliverables-register.md)
- [Change Log](docs/operations/change-log.md)

### 6) Verification and Evidence

Requirements verification and evidence base:

- [Verification Plan](docs/verification/verification-plan.md)
- [Verification Matrix](docs/verification/verification-matrix.md)

### 7) Proposal Layer

Documents directly assembled into grant applications:

- [Proposal Overview](docs/proposal/README.md)
- [01 Excellence](docs/proposal/01-excellence.md)
- [02 Impact](docs/proposal/02-impact.md)
- [03 Implementation](docs/proposal/03-implementation.md)
- [04 Budget](docs/proposal/04-budget.md)
- [05 Consortium](docs/proposal/05-consortium.md)
- [06 Ethics](docs/proposal/06-ethics.md)

## Recommended Workflow

1. Update the [Research Layer](docs/research/README.md): hypotheses, experiments, success criteria.
2. Update the [SRS](docs/specification/system-requirements.md) and check FR/NFR alignment.
3. Add a new ADR in the [ADR Index](docs/adr/README.md) when architecture changes.
4. Update the [Deliverables Register](docs/operations/deliverables-register.md) and [Risk Register](docs/operations/risk-register.md).
5. Record verification results in the [Verification Matrix](docs/verification/verification-matrix.md).
6. Then update [Proposal](docs/proposal/README.md) sections.

## Operational Update Rules

- Every significant architecture change must be recorded in ADR.
- Every SRS requirement must have a verification status.
- Every experiment must be registered in the experiment registry.
- Every risk must have an owner, trigger, and contingency plan.
- Grant claims must reference concrete evidence artifacts.
