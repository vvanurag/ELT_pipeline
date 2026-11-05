# Running & Monitoring Airflow DAGs in Docker

Instructions for executing and observing orchestrated pipelines via Docker and Astronomer CLI.

---

## ⚡ Starting Airflow
From the `airflow/` project directory:

```bash
cd airflow
astro dev start
```

This starts:
- Airflow Webserver: `http://localhost:8080` (Default credentials: `admin` / `admin`)
- Airflow Scheduler
- PostgreSQL metadata database
- Airflow Triggerer

---

## 🛑 Stopping Airflow
```bash
astro dev stop
```

