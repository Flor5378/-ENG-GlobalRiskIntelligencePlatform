# Global Risk Intelligence Platform

## Overview
The **Global Risk Intelligence Platform** is an OSINT-driven, real-time data collection and analytics system for global risk monitoring.  
It ingests events from multiple open sources (military conflicts, protests, climate disasters, industrial accidents, cyber incidents, etc.), enriches and standardizes them, stores them for historical analysis, applies machine learning for classification and risk prediction, and provides real-time visualization and alerting. The final output of the project would be to be usable both on android app, website and .exe structures.

The goal: **turn scattered public data into structured, actionable intelligence**.

---

## Core Objectives
- **Real-time collection** of multi-domain risk events from APIs, RSS feeds, scraping, and social media streams.
- **Normalization & enrichment**: converting heterogeneous data into a unified schema and adding context (geolocation, language, entity recognition).
- **Storage & retrieval**: storing raw and enriched data for both historical and live querying.
- **Machine learning analysis**: classifying event types, estimating severity, detecting anomalies, and predicting escalation.
- **Visualization & alerting**: interactive dashboards, geospatial maps, and push notifications.

---

## High-Level Architecture

| Data Sources | ---> | Ingestion | ---> | Message Bus (Kafka) |
+-----------------------------+ +--------------------+ +---------------------+
APIs (USGS, ACLED, GDACS, Async collectors, Topics:
NOAA, GDELT) webhooks, scrapers - raw-events
Social APIs (X, Reddit) - normalized-events
RSS feeds, forums - enriched-events
- scored-events
- alerts

Message Bus (Kafka) ---> Stream Processing ---> Storage & Search ---> Apps
(Normalization, (TimescaleDB, (Dashboard,
enrichment, Elasticsearch, Alerting,
ML scoring) S3/MinIO) Integrations)
---

## Components

### 1. **Data Sources**
- **Disaster & Climate APIs**:
  - [USGS Earthquake API](https://earthquake.usgs.gov/fdsnws/event/1/)
  - [NOAA Weather API](https://www.weather.gov/documentation/services-web-api)
  - [GDACS API](https://www.gdacs.org/)
  - etc.
- **Conflict & Protest Feeds (Geopolitical Scale)**:
  - [ACLED API](https://acleddata.com/)
  - [GDELT Project API](https://www.gdeltproject.org/)
  - Liveuamap (non-official API)
  - etc.
- **Social Media**:
  - Twitter/X API (filtered stream)
  - Reddit API
  - Mastodon instances (ActivityPub)
    (filtered by keywords)
- **RSS / News**:
  - Crisis Group, humanitarian reports, security blogs

---

### 2. **Ingestion Layer**
- **Async collectors**: Python (`aiohttp`, `httpx`) for non-blocking API calls
- **Scrapers**: `Scrapy` or `BeautifulSoup` for HTML sites without APIs
- **Webhooks**: For APIs supporting push (e.g., USGS earthquake feeds)
- **Output**: Publish raw JSON events to Kafka topic `raw-events`

---

### 3. **Message Bus**
- **Apache Kafka** for event streaming
- Recommended topics:
  - `raw-events`: untouched source payloads
  - `normalized-events`: standardized schema
  - `enriched-events`: geocoded, NER-tagged, embedded
  - `scored-events`: ML-scored events with risk metrics
  - `alerts`: pre-formatted alert messages

---

### 4. **Stream Processing**
- **Normalization**:
  - Enforce JSON schema (e.g., `jsonschema` Python lib)
  - Convert timestamps to UTC ISO-8601
  - Unify category labels
- **Enrichment**:
  - Named Entity Recognition (NER) with spaCy / Hugging Face
  - Language detection (fastText, langdetect)
  - Translation (DeepL API, Google Translate)
  - Geocoding (GeoPy, Nominatim, Google Maps API)
  - Text embeddings (sentence-transformers)
- **Real-time ML scoring**:
  - Event classification (multi-label)
  - Severity estimation
  - Escalation probability

---

### 5. **Storage Layer**
- **TimescaleDB**: Time-series optimized PostgreSQL for structured event data
- **Elasticsearch**: Full-text and geo search for events
- **S3 / MinIO**: Object storage for raw payloads and training datasets
- **Neo4j (optional)**: Graph storage for actor-network analysis
- **Feature Store (optional)**: Feast for ML feature management

---

### 6. **Machine Learning**
- **Classification models**:
  - Text-based multi-label classification (TF-IDF + LightGBM or BERT-based)
- **Severity scoring**:
  - Gradient boosting (XGBoost, LightGBM)
  - Features: location population density, historical event severity, sentiment
- **Anomaly detection**:
  - Isolation forest, change point detection
- **Trend prediction**:
  - Time series forecasting with Prophet or ARIMA
- **Clustering**:
  - HDBSCAN on embeddings for grouping related events

---

### 7. **Applications**
- **Dashboard**:
  - MVP: Streamlit or Plotly Dash
  - Advanced: React + Mapbox/Leaflet + backend API
- **Alerting**:
  - Slack, Telegram, Email, Webhook integration
- **API Layer**:
  - FastAPI service for search, filtering, and integrations

---

### 8. **Observability**
- **Metrics**: Prometheus + Grafana for Kafka lag, API latency, ML inference times
- **Logging**: ELK stack (Elasticsearch, Logstash, Kibana)
- **Data Quality Monitoring**: Schema validation, missing field tracking
- **Drift Detection**: Monitor ML feature and prediction drift

---

## JSON Event Schema (Example)
```json
{
  "event_id": "src-uuid-20250810-0001",
  "source": "acled",
  "source_url": "https://acleddata.com/...",
  "ingest_ts": "2025-08-10T12:34:56Z",
  "published_ts": "2025-08-10T12:10:00Z",
  "categories": ["protest", "violent_incident"],
  "title": "Demonstration downtown",
  "description": "Group of protesters clashed with police...",
  "entities": ["Organisation A", "Police Department X"],
  "actors": [{"name": "Group A", "role": "protester"}],
  "location": {
    "text": "Downtown City, Region",
    "lat": 48.8566,
    "lon": 2.3522,
    "geo_confidence": 0.92
  },
  "severity_score": 0.0,
  "confidence": 0.7,
  "language": "fr",
  "raw": {}
}
```

---

## MVP Development Steps

1. **Select 5 high-value data sources**  
   (e.g., USGS, ACLED, GDACS, GDELT, Twitter API)
2. **Implement async collectors** to fetch and publish to `raw-events`
3. **Define JSON schema** for normalized events
4. **Implement normalizer** → output to `normalized-events`
5. **Implement enrichment** (NER, geocoding, translation) → `enriched-events`
6. **Store events** in TimescaleDB and index searchable fields in Elasticsearch
7. **Build MVP dashboard** (Streamlit + folium map + table view)
8. **Implement baseline ML classifier** (TF-IDF + LightGBM)
9. **Add alerting rules** (severity threshold → Slack/Telegram)

---

## Security & Compliance

- **Respect Terms of Service**: Only use APIs and scraping methods allowed by source sites  
- **Avoid PII**: Anonymize any personally identifiable information before storage or sharing  
- **Data Licensing**: Track source and license metadata for all ingested data  
- **Audit Trail**: Keep raw payloads for traceability, but secure them

---

## Future Enhancements

- Add more data domains (cyber incidents, industrial safety, public health)
- Deploy to Kubernetes with autoscaling
- Implement active learning loop for ML model improvements
- Create user authentication and role-based dashboards
- Implement geospatial risk propagation models

---

