---
name: mermaid
description: Descriptive visualisations via inline Mermaid inside any md text. Should be used instead of the wall of a plain text while explaining complex things. Covers flowcharts, sequence/class/state/ER/C4 diagrams, mind maps, timelines, Gantt, Kanban, quadrant/pie/XY/radar/Sankey charts, packet diagrams, and more. Prefer a compact diagram when it can replace several prose steps; skip it when a table, tiny text structure, or polished standalone visual is better.
---

# Mermaid

Make the visualization part of the ordinary Markdown answer. Use a fenced `mermaid` block next to the short prose it clarifies; do not create a separate artifact unless the user asks for one.

## Choose by meaning

- Process, branching, dependencies, data flow → `flowchart`
- Calls and interactions over time → `sequenceDiagram` or `zenuml`
- Domain structure or data model → `classDiagram` or `erDiagram`
- Lifecycle and transitions → `stateDiagram-v2`
- Architecture → `C4Context`, `C4Container`, `architecture-beta`, or a flowchart with boundaries
- Roadmap and schedule → `timeline` or `gantt`
- Hierarchy and decomposition → `mindmap`, `treemap-beta`, or `block-beta`
- Work and prioritization → `kanban` or `quadrantChart`
- Quantitative comparison → `xychart`, `pie`, `radar-beta`, or `sankey-beta`
- Binary or protocol layout → `packet-beta`

Do not force every idea into a flowchart. Pick the grammar whose semantics match the relationship.

## Rules

- Show one useful idea per diagram. Split diagrams when crossings or density hide the point.
- Preserve truthful order, direction, cardinality, boundaries, units, and scale. Never invent data to make a chart look complete.
- Use real labels from the task. Add subgraphs, participants, states, series names, and notes only when they carry meaning.
- For charts, label units, choose an honest axis range, and name multiple series. Use color as reinforcement, not the only carrier of meaning.
- Keep surrounding prose short and do not narrate what the diagram already makes obvious.
- Renderer support varies for beta diagrams, C4, and ZenUML. Check the current syntax when uncertain and fail visibly rather than silently replacing the requested visualization with a weaker fallback.
- Mermaid is for compact inline explanation, not polished interactive charts, geographic maps, or spatial UI mockups.

## bb IDE renderer: mandatory frontmatter

When the diagram renders in bb chat (any bb thread), always start the mermaid block with this frontmatter. Verified root causes, do not drop pieces of it: bb initializes mermaid with font `Inter` while the app renders `Inter Variable`, so HTML labels are measured narrower than they render and get clipped (`htmlLabels: false` makes measurement and render use the same SVG font); mermaid's own state-diagram styles hardcode `opacity: 0.5` on edge-label plates, which only `themeCSS` can override (it is not in the strict-mode secure list).

```yaml
---
config:
  htmlLabels: false
  flowchart:
    htmlLabels: false
  themeCSS: ".edgeLabel rect, .edgeLabel .label rect, .stateLabel .box { opacity: 1 !important; fill: #ffffff; }"
  themeVariables:
    edgeLabelBackground: "#ffffff"
    labelBackgroundColor: "#ffffff"
---
```

Notes: `#ffffff` assumes bb light theme. `stateDiagram` transition labels have no auto-wrap even in SVG mode — keep each label line short and break long labels with `<br/>`; there is no knob for explicit label-box size. Font-family overrides (`Inter Variable` etc.) do NOT fix the clipping — tested.

## Examples

Process with boundaries:

```mermaid
flowchart LR
    subgraph Client
        U([User]) --> UI[Checkout]
    end
    subgraph Platform
        UI --> API[Create order]
        API --> V{Valid?}
        V -- Yes --> P[Take payment]
        V -- No --> E[Show errors]
    end
    P --> C([Confirmation])
    E --> UI
```

Interaction with alternatives:

```mermaid
sequenceDiagram
    actor User
    participant App
    participant Auth
    User->>App: Submit credentials
    App->>Auth: Create session
    alt Valid credentials
        Auth-->>App: Access token
        App-->>User: Open dashboard
    else Invalid credentials
        Auth-->>App: 401
        App-->>User: Show error
    end
```

Chart with named series and non-zero axes:

```mermaid
---
config:
  themeVariables:
    xyChart:
      plotColorPalette: "#2563eb, #dc2626, #16a34a"
---
xychart
    title "API latency under load"
    x-axis "Requests per second" 200 --> 1200
    y-axis "Latency, ms" 50 --> 700
    line "p50" [85, 95, 115, 150, 205, 290]
    line "p95" [145, 170, 220, 310, 455, 640]
    line "SLO" [300, 300, 300, 300, 300, 300]
```

Protocol layout:

```mermaid
packet-beta
    0-3: "Version"
    4-7: "IHL"
    8-13: "DSCP"
    14-15: "ECN"
    16-31: "Total Length"
    32-47: "Identification"
    48-50: "Flags"
    51-63: "Fragment Offset"
```

Lifecycle with explicit transitions:

```mermaid
stateDiagram-v2
    [*] --> Draft
    Draft --> Review : submit
    Review --> Draft : changes requested
    Review --> Approved : approve
    Approved --> Published : publish
    Published --> Archived : retire
    Archived --> [*]
```

Relational data model:

```mermaid
erDiagram
    CUSTOMER ||--o{ ORDER : places
    ORDER ||--|{ ORDER_ITEM : contains
    PRODUCT ||--o{ ORDER_ITEM : references

    CUSTOMER {
        uuid id PK
        string email UK
    }
    ORDER {
        uuid id PK
        uuid customer_id FK
        string status
    }
    ORDER_ITEM {
        uuid order_id PK, FK
        uuid product_id PK, FK
        int quantity
    }
    PRODUCT {
        uuid id PK
        string name
    }
```

Schedule with dependencies and a milestone:

```mermaid
gantt
    title Release plan
    dateFormat YYYY-MM-DD
    excludes weekends

    section Discovery
        Specification :done, spec, 2025-03-03, 3d
    section Build
        Backend API   :crit, api, after spec, 5d
        Web UI        :ui, after spec, 7d
    section Release
        Launch        :milestone, after ui, 0d
```

Hierarchy and decomposition:

```mermaid
mindmap
    root((Product launch))
        Product
            Research
            MVP
        Go-to-market
            Content
            Sales
        Operations
            Support
            Reliability
```

Prioritization by two dimensions:

```mermaid
quadrantChart
    title Feature priorities
    x-axis Low effort --> High effort
    y-axis Low impact --> High impact
    quadrant-1 Strategic bets
    quadrant-2 Quick wins
    quadrant-3 Fill-ins
    quadrant-4 Avoid
    Better search: [0.25, 0.82]
    Enterprise SSO: [0.55, 0.76]
    New UI themes: [0.35, 0.28]
    Custom chat: [0.82, 0.22]
```

Work moving through stages:

```mermaid
kanban
    backlog[Backlog]
        task1[Define API contract]
        task2[Prepare migration]
    doing[In progress]
        task3[Implement backend]
    review[Review]
        task4[Security review]
    done[Done]
        task5[Create dashboard]
```

Part-to-whole comparison:

```mermaid
pie showData
    title Infrastructure cost
    "Compute" : 42
    "Databases" : 24
    "Network" : 14
    "Storage" : 11
    "Monitoring" : 9
```

Architecture with explicit boundaries:

```mermaid
architecture-beta
    group edge(cloud)[Edge]
    group app(cloud)[Application]

    service client(internet)[Client]
    service gateway(server)[Gateway] in edge
    service api(server)[API] in app
    service db(database)[Database] in app

    client:R --> L:gateway
    gateway:R --> L:api
    api:R --> L:db
```

## Documentation

- [Mermaid documentation](https://mermaid.js.org/intro/)
- [Diagram syntax reference](https://mermaid.js.org/intro/syntax-reference.html)
- [Mermaid Live Editor](https://mermaid.live/)
