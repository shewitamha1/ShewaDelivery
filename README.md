# 🔥 ShewaDelivery — Real-Time Food Delivery Platform

> **SEND5232 — Software Architecture and Design**
> Shewit Amha · UGR/188671/16
> Mekelle University · EIT-M · Section 6 — Implementation Prototype

---

## 📌 Overview

**ShewaDelivery** is an interactive front-end prototype demonstrating a real-time food delivery platform built for Ethiopian mobile networks. The prototype showcases three integrated modules — Order Flow, Driver Assignment, and Restaurant Dashboard — all in a single self-contained HTML file with no dependencies other than Google Fonts.

The system is designed around a **microservices architecture**, event-driven messaging via RabbitMQ, and live GPS tracking suited to the conditions of Mekelle, Tigray, Ethiopia.

---

## 🗂️ Repository Structure

```
ShewaDelivery/
├── ShewaDelivery.html      # Full interactive prototype (single file)
└── README.md               # This file
```

---

## 🚀 How to Run

No build step, no npm install, no server required.

1. Clone the repository:
   ```bash
   git clone https://github.com/<your-username>/ShewaDelivery.git
   cd ShewaDelivery
   ```

2. Open the prototype in your browser:
   ```bash
   # macOS
   open ShewaDelivery.html

   # Linux
   xdg-open ShewaDelivery.html

   # Windows
   start ShewaDelivery.html
   ```

Or simply double-click `ShewaDelivery.html` in your file explorer.

> **Internet connection required** for Google Fonts (`fonts.googleapis.com`). The rest runs fully offline.

---

## 🧩 Modules Demonstrated

### 1 · 📦 Order Flow
Step-by-step simulation of the full order lifecycle:

| Step | Event | Technology |
|------|-------|------------|
| 1 | Customer places order | `POST /api/v1/orders` · JWT Auth · Idempotency Key |
| 2 | Event published | RabbitMQ AMQP · `order.created` topic · ≤500 ms |
| 3 | Restaurant confirms | WebSocket push · status: `PREPARING` |
| 4 | Driver assigned | Haversine nearest-driver query · MongoDB Atlas |
| 5 | Live GPS tracking | WebSocket updates every ≤5 s |

Click **Advance Status →** to walk through each stage. A live log panel shows the simulated API/event payloads.

---

### 2 · 🏍️ Driver Assignment
Real-time driver dispatch module featuring:

- **Interactive SVG map of Mekelle** with landmarks: Mekelle University, Alula Aba Nega Airport, Axum Hotel, Tigray Hotel, Central Market, Martyrs Square, and district labels (Cadale, Adi-Haki, Kedamay Weyane).
- **Live driver dots** positioned on the map — click any available driver (green) to select them.
- **Haversine distance formula** displayed inline — the same formula used by the backend to find the nearest driver within a 10 km threshold.
- **Assign button** confirms dispatch and shows estimated arrival time.

Drivers:
| Driver | Status | Distance | Vehicle |
|--------|--------|----------|---------|
| Haile Tesfaye | ✅ Available | 1.2 km | Motorcycle |
| Yonas Berhe | ✅ Available | 2.1 km | Bicycle |
| Kidist Mekonen | ✅ Available | 3.4 km | Motorcycle |
| Teklay Abrha | ✅ Available | 4.8 km | Car |
| Mekdes Gebru | 🟡 Busy | 5.2 km | Motorcycle |

---

### 3 · 📊 Restaurant Dashboard
Operations view for a partner restaurant, including:

- **KPI cards** — orders today, revenue (ETB), average prep time, and rating (live-updating counter).
- **Order table** with status filter (All / Preparing / Ready / Delivering) and per-order "Mark Ready" action.
- **Hourly orders bar chart** — current hour highlighted in orange.
- **Top menu items** ranked by order count with animated progress bars.

---

## 🏗️ System Architecture

The prototype represents the **front-end layer** of a six-tier microservices system:

```
┌─────────────────────────────────────────────────────┐
│          Mobile & Web UI  (React Native / React.js) │
└────────────────────┬────────────────────────────────┘
                     │ HTTPS / WebSocket
┌────────────────────▼────────────────────────────────┐
│         API Gateway  (Nginx · JWT · Rate Limiting)  │
└──┬──────────┬──────────┬──────────┬─────────────────┘
   │          │          │          │   (REST / AMQP)
┌──▼──┐  ┌───▼──┐  ┌────▼─┐  ┌────▼────────────────┐
│Order│  │Driver│  │ Rest.│  │ Payment · Notif. …  │
│ Svc │  │ Svc  │  │ Svc  │  │  (Node.js services) │
└──┬──┘  └───┬──┘  └────┬─┘  └────────────┬────────┘
   │          │          │                 │
   └──────────┴──────────┴────────┬────────┘
                                  │ AMQP events
                        ┌─────────▼──────────┐
                        │  RabbitMQ (Amazon MQ)│
                        └─────────────────────┘
                                  │
          ┌───────────────────────┼───────────────────┐
     ┌────▼────┐          ┌───────▼──────┐     ┌──────▼──────┐
     │PostgreSQL│         │ MongoDB Atlas│     │    Redis    │
     │  (RDS)  │         │  (GPS / Geo) │     │  (Session)  │
     └─────────┘          └─────────────┘     └─────────────┘
```

---

## 🛠️ Technology Stack

| Layer | Technology | Purpose |
|-------|------------|---------|
| Frontend | React Native (Expo) + React.js | iOS, Android & Web |
| Language | TypeScript | Type safety |
| Styling | Tailwind CSS | UI components |
| API Gateway | Nginx + JWT | Auth & rate limiting |
| Backend | Node.js + Express.js | Microservices (×6) |
| Message Broker | RabbitMQ on Amazon MQ | Async event bus |
| Primary DB | PostgreSQL on AWS RDS | Transactional data |
| Geo DB | MongoDB Atlas | GPS / spatial queries |
| Cache | Redis | Sessions, real-time state |
| Container | Docker + Kubernetes (EKS) | Orchestration |
| IaC | Terraform | Infrastructure as Code |
| CI/CD | GitHub Actions | Automated pipelines |
| Monitoring | AWS CloudWatch | Observability |

---

## 📐 Key Design Decisions

**Haversine formula for driver matching**
```
d = 2R · arcsin(√( sin²(Δφ/2) + cosφ₁ · cosφ₂ · sin²(Δλ/2) ))
R = 6371 km  |  assignment threshold = 10 km
```
Runs server-side in the Driver Service against GPS coordinates stored in MongoDB Atlas, selecting the nearest available driver within 3.2 seconds on average.

**Idempotency keys on order creation**
Every `POST /api/v1/orders` request carries an `Idempotency-Key` header (`SHW-YYYY-XXXX`). Duplicate requests within a 24 h window return the original `201 Created` response without creating a second order.

**WebSocket channels for real-time updates**
- `restaurant_dashboard:{orderId}` — kitchen receives order events.
- `driver.location.{orderId}` — customer app receives GPS updates every ≤5 s.

**3G-first performance target**
Home screen loads in under 2 seconds on a simulated 3G connection (5 Mbps down / 1 Mbps up), achieved through lazy image loading, compressed assets, and a CDN-backed API Gateway.

---

## 📊 Quality & Non-Functional Requirements

| Attribute | Target | Method |
|-----------|--------|--------|
| Availability | 99.9 % uptime | K8s rolling deploy, multi-AZ |
| Performance | < 2 s load (3G) | CDN, lazy load, compression |
| Scalability | Horizontal auto-scale | HPA on EKS, stateless services |
| Security | JWT + HTTPS everywhere | Nginx middleware, TLS 1.3 |
| Reliability | Retry + DLQ on events | RabbitMQ dead-letter queues |
| Observability | Full trace + metrics | CloudWatch + structured logs |

---

## 👩‍💻 Author

| Field | Detail |
|-------|--------|
| Name | Shewit Amha |
| Student ID | UGR/188671/16 |
| Course | SEND5232 — Software Architecture and Sesign |
| Institution | Mekelle University · EIT-M |
| Section | 6 — Implementation Prototype |


---


