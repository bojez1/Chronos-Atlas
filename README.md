# ⏳ Chronos Atlas: The Historical Timeline Project

**A modern, data-driven visualization of history powered by Python, GraphQL, and PostgreSQL.**

This project aims to transform the consumption of biographical and historical data into an interactive, visual experience. By linking figures, works, and influences across time, Chronos Atlas provides unprecedented chronological context to history.

---

## 🎯 Core Value Proposition

The primary goal is to launch a scalable web application featuring a stunning **Master Timeline View**. Users can explore the overlapping lives and events of historical figures, filtering by field, occupation, and influence.

| Feature | Description |
| :--- | :--- |
| **Interactive Timeline** | Visually displays the lifespan of historical figures on a scrollable, zoomable axis. |
| **Relational Data** | Utilizes a graph-like model to clearly show **who influenced whom** and their contemporaries. |
| **Optimized Speed** | Built on a high-performance backend stack designed to handle complex chronological queries at scale. |

---

## 🛠️ Technology Stack

Chronos Atlas is built on a **Python/PostgreSQL/GraphQL** foundation, chosen specifically for its power in handling complex relational and time-series data.

| Layer | Primary Technology | Rationale |
| :--- | :--- | :--- |
| **Backend/API** | **Python (Django)** + **GraphQL (Graphene)** | Fast development, structured framework, and efficient data transfer. |
| **Database** | **PostgreSQL** 🐘 | **Core performance engine.** Superior indexing for time-series and relational integrity. |
| **Performance** | **Redis** + **Celery** | Caching API responses and handling heavy, background data ingestion (ETL). |
| **Data Source** | **Wikidata (SPARQL)** | Structured, open source for reliable historical data. |

---

## 🧭 Project Blueprint Navigation

This project is fully documented across three detailed blueprint files. Start with the Architecture Blueprint to understand the technical foundation.

| Document | Focus | Key Topics |
| :--- | :--- | :--- |
| **1. `ARCHITECTURE_BLUEPRINT.md`** | **Technical Foundation** | Stack justification, **Refined PostgreSQL Schema**, and indexing strategy. |
| **2. `DATA_FLOW_AND_SCALING.md`** | **Operational Logic & Stability** | **Data Flow** (ETL and Serving Cycles), **PostgreSQL Scaling Rationale**, and performance requirements. |
| **3. `EXECUTION_ROADMAP.md`** | **Implementation Plan** | Detailed breakdown of all phases: **Foundation**, **MVP**, **Performance Tuning**, and **Production Launch**. |

### [🔗 **Read the Architecture Blueprint**](./ARCHITECTURE_BLUEPRINT.md)

### [🔗 **View the Execution Roadmap**](./EXECUTION_ROADMAP.md)

### [🔗 **Explore Data Flow and Scaling**](./DATA_FLOW_AND_SCALING.md)

---

## 📈 Current Status: Planning & Foundation

| Phase | Status | Next Major Milestone |
| :--- | :--- | :--- |
| **Phase 0: Foundation** | 🟢 **Complete** | **Start Phase 1: Build the MVP Timeline.** |
| **Phase 1: MVP** | ⚪️ Pending | Complete the Master Timeline and Figure Detail Views. |
| **Phase 2: Polish** | ⚪️ Pending | Implement Redis Caching and Celery ETL. |
| **Phase 3: Production** | ⚪️ Pending | Full data load and cloud deployment. |

*Last Updated: September 2025*
