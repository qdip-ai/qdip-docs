# DIP Docs Repository

Language: Українська (default) | [English](README.en.md)

Цей репозиторій містить керований комплект документації QDIP/DIP: research, requirements, architecture, verification, evidence та матеріали для грантових заявок.

> **Поточний canonical grant/R&D package:** [QDIP Universal Runtime — Grant Evidence Pack](docs/grant-evidence-pack/README.md)
>
> Центральна thesis: зберегти non-inferior decision quality відносно specialized implementation при скороченні total engineering effort щонайменше на 50% у кожному mandatory validation domain.

## Зміст

1. [Призначення репозиторію](#призначення-репозиторію)
2. [Швидка навігація](#швидка-навігація)
3. [Карта документації](#карта-документації)
4. [Рекомендований порядок роботи](#рекомендований-порядок-роботи)
5. [Операційні правила оновлення](#операційні-правила-оновлення)

## Призначення репозиторію

Репозиторій організований як керований ланцюг:

Research -> Requirements -> Architecture -> Implementation -> Verification -> Evidence -> Proposal

Це дозволяє:

- зберігати простежуваність від наукових тверджень до доказів;
- контролювати зміни через ADR і change log;
- збирати заявку з готових модулів, а не писати з нуля;
- відділяти pre-registered primary evidence від exploratory результатів.

## Розробникам плагінів

[SDK quickstart — English](docs/sdk/quickstart.en.md) | [Українська](docs/sdk/quickstart.uk.md) | [Polski](docs/sdk/quickstart.pl.md). Підключення наразі **лише за допуском оператора**, не публічний self-service. Посібник не потребує доступу до приватного репозиторію Core. Перед публікацією файлів діють [правила безпечної публікації](docs/security/publication-safety.md).

## Швидка навігація

- [Universal Runtime Grant Evidence Pack](docs/grant-evidence-pack/README.md)
- [Шар Grant Documentation](docs/grant/README.md)
- [Шар Research](docs/research/README.md)
- [Шар Proposal](docs/proposal/README.md)
- [System Requirements Specification](docs/specification/system-requirements.md)
- [Architecture Decision Records](docs/adr/README.md)
- [Verification Layer](docs/verification/README.md)
- [Risk Register](docs/operations/risk-register.md)
- [Deliverables Register](docs/operations/deliverables-register.md)
- [Change Log](docs/operations/change-log.md)

## Карта документації

### 0) Canonical Grant Evidence Pack

Поточний application-facing пакет для Universal Runtime thesis:

- [Evidence Pack Index](docs/grant-evidence-pack/README.md)
- [Executive Summary](docs/grant-evidence-pack/01-executive-summary.md)
- [State of the Art](docs/grant-evidence-pack/02-state-of-the-art.md)
- [Research Hypothesis](docs/grant-evidence-pack/03-research-hypothesis.md)
- [Methodology](docs/grant-evidence-pack/04-methodology.md)
- [Architecture + Canonical Decision IR](docs/grant-evidence-pack/05-architecture-canonical-ir.md)
- [Test A Pre-registration](docs/grant-evidence-pack/06-test-a-protocol.md)

### 1) Grant Base

Базовий корпус R&D і грантової документації:

- [Огляд і правила](docs/grant/README.md)
- [Додатки](docs/grant/appendices/A01-literature-matrix.md)
- [Data Dictionary](docs/grant/appendices/A02-dataset-dictionary.md)
- [Metrics Sheet](docs/grant/appendices/A03-metrics-calculation-sheet.md)

### 2) Research Layer

Зв'язок між гіпотезами, експериментами і доказами:

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

Функціональні та нефункціональні вимоги:

- [System Requirements Specification](docs/specification/system-requirements.md)

### 4) Architecture Governance

Ключові технічні рішення:

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

Контроль виконання і змін:

- [Risk Register](docs/operations/risk-register.md)
- [Deliverables Register](docs/operations/deliverables-register.md)
- [Change Log](docs/operations/change-log.md)

### 6) Verification and Evidence

Верифікація вимог і доказова база:

- [Verification Plan](docs/verification/verification-plan.md)
- [Verification Matrix](docs/verification/verification-matrix.md)

### 7) Proposal Layer

Матеріали, які напряму збираються у заявку:

- [Proposal Overview](docs/proposal/README.md)
- [01 Excellence](docs/proposal/01-excellence.md)
- [02 Impact](docs/proposal/02-impact.md)
- [03 Implementation](docs/proposal/03-implementation.md)
- [04 Budget](docs/proposal/04-budget.md)
- [05 Consortium](docs/proposal/05-consortium.md)
- [06 Ethics](docs/proposal/06-ethics.md)

## Рекомендований порядок роботи

1. Зафіксувати [Test A pre-registration](docs/grant-evidence-pack/06-test-a-protocol.md).
2. Стабілізувати [Canonical Decision IR](docs/grant-evidence-pack/05-architecture-canonical-ir.md).
3. Freeze Core / IR semantics / extension API та benchmark artifacts.
4. Виконати Test A — Readiness Recovery без post-hoc зміни thesis.
5. Виконати Test B — Procurement через той самий runtime.
6. Виконати blind Test C на незалежному domain із frozen Core/IR/API.
7. Оновлювати Research/Verification/Proposal layers лише на основі traceable evidence.

## Операційні правила оновлення

- Кожна суттєва зміна архітектури має бути відображена в ADR.
- Кожна вимога SRS повинна мати verification-статус.
- Кожен primary experiment повинен бути pre-registered і зареєстрований в experiment registry.
- Кожен ризик повинен мати owner, trigger і contingency plan.
- Дані для грантових тверджень повинні посилатися на evidence-артефакти.
- Після freeze зміна Core/IR/API semantics робить відповідний primary generalization evidence недійсним і має бути явно задокументована.
