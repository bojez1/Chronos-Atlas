# 🌊 DATA_FLOW_AND_SCALING.md

## "Chronos Atlas" Data Flow and Scaling Rationale

This document clarifies how data moves through the system and justifies the database choice for scaling.

### 3.1 Data Flow Diagram (Operational Logic)

| Cycle | Step | Node/Component | Action/Flow |
| :--- | :--- | :--- | :--- |
| **Ingestion** | **1. Schedule** | Celery Beat → Redis | Sends the ETL task to the message queue. |
| (Async ETL) | **2. Extract** | Celery Worker → SPARQLWrapper | Runs Python code to query Wikidata. |
| | **3. Transform** | Django Command | Cleans data, calculates `normalized_year`. |
| | **4. Load** | Django Command → PostgreSQL | Upserts clean data; PG applies time-series indexes. |
| **Serving** | **1. Request** | User Browser → Django Server | Sends specific GraphQL query (e.g., `timelineFigures(startYear: 1600)`). |
| (Real-time) | **2. Cache Check**| Graphene Resolver → Redis | Checks if the query result exists in Redis. |
| | **3. Database Query**| Graphene Resolver → PostgreSQL | **(Cache Miss)** Executes fast indexed query for the timeline range. |
| | **4. Cache Write**| Graphene Resolver → Redis | **(Cache Miss)** Stores the result back into Redis. |
| | **5. Response** | Graphene Resolver → User Browser | Returns the final JSON payload. |

---

### 3.2 Scaling Rationale: PostgreSQL is Key

The choice of **PostgreSQL** over NoSQL (like MongoDB) is a deliberate scaling strategy for this project.

| Scaling Challenge | PostgreSQL Solution | Rationale for Choice |
| :--- | :--- | :--- |
| **Timeline Query Speed** | Uses **Indexed Integer (`normalized_year`)** fields with **GiST/BRIN** indexing. | **Future-Proofing:** Guarantees timeline queries scale efficiently (logarithmically) even with massive datasets. |
| **Relational Complexity** | **Native, Fast JOINs** and Foreign Key constraints. | **Integrity:** Ensures relationships are never broken. **Performance:** Efficiently links `Figure` to `Work` and `Influence` in a single GraphQL query. |
| **ETL Management** | Separated the long-running ETL process using **Celery/Redis**. | **Stability:** Keeps the web server fast and responsive by offloading heavy, periodic tasks to background workers. |
| **Read Scaling**| Easily scaled by deploying multiple **Read Replicas**. | Meets high traffic demands by distributing read-heavy load across multiple database instances. |
