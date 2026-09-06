# Diagrams (D1–D4)

All four diagrams describe the same system and the same one core workflow as
`feature-list.md`/`user-journey.md`: actors **Formulator** and **Domain Expert**, system
**AI Perfumery Engine**. Rendered as Mermaid (GitHub and most Markdown viewers render these
natively); source diagram tool alternatives (draw.io/Figma/PlantUML) can replace these later
without changing what they show.

## D1 — System Context

What it shows: the system as one box, plus the actors and data store outside it. No internal
detail — just the boundary.

```mermaid
flowchart LR
    Formulator[Formulator]:::actor
    DomainExpert[Domain Expert]:::actor
    System[AI Perfumery Engine]
    Dataset[(Material & Rule Dataset)]

    Formulator -->|logs in, views & evaluates formulas| System
    DomainExpert -->|approves rule/threshold/group changes| System
    System -->|reads/writes| Dataset

    classDef actor fill:#fff,stroke:#333,stroke-width:2px,font-weight:bold;
```

## D2 — Use Case

What it shows: which actor can do what. The core use case ("View & evaluate a formula") is
central and includes Login, matching the journey's step order.

```mermaid
flowchart LR
    Formulator[Formulator]:::actor
    DomainExpert[Domain Expert]:::actor
    subgraph SYS["AI Perfumery Engine"]
        UC1([Login])
        UC2([View formula list])
        UC3([View & evaluate a formula])
        UC4([Adjust value & recalculate])
        UC5([Export formula view])
        UC6([Approve rule/threshold change])
    end

    Formulator --- UC1
    Formulator --- UC2
    Formulator --- UC3
    Formulator --- UC4
    Formulator --- UC5
    DomainExpert --- UC6

    UC3 -.include.-> UC1
    UC2 -.include.-> UC1

    classDef actor fill:#fff,stroke:#333,stroke-width:2px,font-weight:bold;
```

## D3 — High-Level Architecture

What it shows: layers and data direction, kept deliberately generic (no framework named) since
no technology stack has been decided yet (`project-context.md` §29) — this diagram must not be
read as a stack decision.

```mermaid
flowchart LR
    subgraph Client
        UI[Web UI]
    end
    subgraph Server
        API[REST API]
        AUTH[Login + Access Log]
        ENGINE[Calculation Engine]
    end
    subgraph Database
        DB[(Formulas, Materials, Rules,<br/>Accounts, Consent, Access Log)]
    end

    UI -->|HTTPS| API
    API --> AUTH
    API --> ENGINE
    AUTH --> DB
    ENGINE --> DB
```

## D4 — Activity

What it shows: one scenario, start to end — the same steps as `user-journey.md`, including the
two optional branches (what-if edit loops back to review; export is optional before end).

```mermaid
flowchart TD
    Start((Start)) --> Login[Log in]
    Login --> List[Open formula list]
    List --> Select[Select a formula]
    Select --> Calc[System calculates weights/percentages<br/>and highlights applicable rules]
    Calc --> Review[Formulator reviews formula view]
    Review --> Decision1{Adjust a value?}
    Decision1 -->|yes| Recalc[Recalculate in place]
    Recalc --> Review
    Decision1 -->|no| Decision2{Export?}
    Decision2 -->|yes| Export[Export formula view]
    Decision2 -->|no| End((End))
    Export --> End
```
