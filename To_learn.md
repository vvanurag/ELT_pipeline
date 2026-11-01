I will be starting with Data engineering responsibilities.
I need to learn the below. 
5 Architectural patterns to learn
- ETL
- ELT
- Lamda
- Kapa
- Madallion

FASTAPI
ELT -> Snowflakes, Databricks, Apache Kafka, Microsoft Azure datafactory
Also learn Different LLM Models and their specialities, choice of LLM according to Usecase and model training, finetuning ect.
Github -> GIthub actions -> Check out first step
CI/CD pipeline

Some other terms that came up are
SLA Service level 
SOA -> Service oriented architecture
non-functionals - > What are software's non-functional necessities?

Bronze Silver Gold
Python basics

Eventually I will grow towards being a AI engineer or higher roles like AI Specialist, AI researcher and higher.


Let us start with ELT, ETL, Databricks, Snowflake, FASTAPI, Data engineering required behind RAG systems.



# Live coding session
---------------------
How to build fact tables, Datamarts, Snowflake RBAC, how to deploy a model on Airflow.
Tools:
DBT for transformation
Snowflake for data warehousing
Airflow for orchestration
Snowflake TPC-H dataset 

#Contanerization
----------------
pip, python env, Homebrew, Docker,




# NEILSEN IQ Tech stack(googled)
-----------------------
As a massive consumer intelligence company handling over 3.2 trillion data attributes and trillions of transactions annually, NielsenIQ (NIQ) uses a modern, cloud-native tech stack centered on big data processing, microservices, and enterprise AI.
Following its spin-off and multi-billion dollar replatforming initiative, the core components of NIQ’s engineering architecture break down into several distinct tiers:

1. Cloud Infrastructure & Hybrid Orchestration

• Google Cloud Platform (GCP): Designated as NIQ's primary AI and cloud services provider, powering their global data pipelines and hosting their NIQ Connect enterprise CPG analytics platform.
• Microsoft Azure: Heavily utilized across various engineering verticals alongside GCP.
• HashiCorp Terraform: Used as the primary Infrastructure-as-Code (IaC) tool to provision and scale multi-cloud environments.

2. Big Data & Real-Time Stream Processing

• Apache Kafka: Acts as the central nervous system for real-time event streaming and data ingestion across multi-market retail metrics.
• Databricks: Extensively deployed for large-scale data lakehouse management, collaborative data science, and ETL optimization.
• Apache Hadoop: Maintained for heavy legacy on-premise and distributed data warehouse operations.
• Elastic Stack (ELK): Leveraged for log aggregation, data management, observability, and real-time operational search capabilities.

3. Core Development & Backend Services

• Java & .NET: The backbone of NIQ’s enterprise backend services and secure microservice architectures.
• Python: The default language powering their data science, statistical modeling, and ML engineering pipelines.
• C++: Utilized in high-performance execution blocks where processing speeds for massive transactional matrices are critical.

4. Advanced AI & Analytics Layer

• Google Gemini & OpenAI Claude / ChatGPT: Integrated deeply into their business workflow layer via Optiq Bridge, enabling conversational query layers over petabytes of consumer analytics.
• Google Meridian Framework: Integrated directly into the NIQ Cadence platform to handle location-based open-source marketing mix modeling (MMM) and real-time spending optimizations.

5. Frontend & Application Delivery

• JavaScript Frameworks: Used to build intuitive web application dashboards like NIQ Discover for real-time customer data visualizations.
• UserWay: Implemented across consumer-facing SaaS tools to enforce strict digital accessibility web standards.

6. DevOps & Enterprise Middleware

• Jenkins: Serves as the primary automation engine driving continuous integration and continuous delivery (CI/CD) pipelines.
• Boomi: Used as a low-code Integration Platform as a Service (iPaaS) to link siloed internal APIs and cloud workflows seamlessly.
• Salesforce & Salesloft: Form the foundation of their enterprise CRM, customer data integration, and sales pipeline mechanics.