
# Challenge 6 — Collectibles Store Web Application
**Java 17 | Spark Framework | Mustache Templates | WebSockets | REST API | Scrum Methodology**

---

## Summary

The Collectibles Store Web Application is a professional-grade web system built with Java and the Spark framework. It provides a REST-first backend, server-rendered views powered by Mustache templates, and a real-time layer using WebSockets to broadcast price updates and auction activity. The solution is designed for incremental delivery through three Sprints, with clear acceptance criteria, peer review gates, and reproducible test evidence (Postman/Newman).

Stakeholders:
- **Rafael** : Full-stack developer responsible for architecture, implementation, and delivery.
- **Ramón**: Business owner and collector. Defines business rules and validates value.
- **Sofía**: Senior developer mentor. Advises on performance, architecture, and real-time features.

Business Goals:
1) Launch a clean, fast, and maintainable marketplace experience for collectibles.
2) Allow users to browse, filter, and inspect items; post offers; and see live price updates.
3) Document the system thoroughly for bootcamp evaluation and for future maintainers.

---

## Objectives & Scope

**Primary Objective**  
Deliver a modular, production-lean Java application that exposes a stable API, renders a pragmatic UI, and supports real-time price updates, following Scrum with three Sprints.

**Functional Scope**
- Collectibles CRUD (create, read, update, delete).
- Server-rendered pages for listing and item details.
- Offer creation form with validation and feedback.
- Filters: full-text query, category, price range; sorting and pagination.
- WebSockets channel for price updates during auctions.

**Non-Functional Scope**
- Clean, layered design (Routes/Controllers → Services → Repository → Domain).
- Reproducible testing via Postman and CLI automation with Newman.
- CI for build & test on each PR using GitHub Actions.
- Documentation-first workflow (this README, peer review template, evidence folders).

---

## Architecture

### High-Level Diagram
```text
┌───────────────────────────┐             HTTP + WS              ┌────────────────────────────┐
│         Browser           │  <──────────────────────────────>  │         Spark App          │
│  (HTML/CSS/JS + Mustache) │                                    │  Routes / Services / Repo  │
└─────────────┬─────────────┘                                    └─────────────┬──────────────┘
              │                                                                │
              │                                                                ▼
              │                                                      In-Memory Repository
              │                                                   (DB-ready future upgrade)
              ▼
        WebSocket Client
```

Key Decisions
- Use Spark for minimal overhead and fast routing.
- Server-side templates (Mustache) for simplicity, SEO, and low client JS.
- WebSockets for bi-directional, low-latency price updates.
- Start with in-memory storage; design for an easy transition to a DB later.

---

## Technology Stack

| Layer        | Technology                | Purpose                                  |
|--------------|---------------------------|------------------------------------------|
| Language     | Java 17                   | Runtime and language features            |
| Framework    | Spark (spark-core)        | HTTP routing and filters                 |
| Templates    | Mustache.java             | Server-side rendering                    |
| Frontend     | HTML5, CSS3, Vanilla JS   | UI, form handling, WebSocket client      |
| Real-time    | WebSockets (Jetty)        | Live price broadcasting                  |
| Build        | Maven                     | Dependency & build management            |
| Testing      | Postman / Newman          | API test automation                      |
| CI/CD        | GitHub Actions            | Build & test on push/PR                  |

---

## Repository Structure

```text
challenge6-collectibles-store/
├─ src/
│  ├─ main/
│  │  ├─ java/com/collectibles/
│  │  │  ├─ App.java
│  │  │  ├─ routes/ItemsRoutes.java
│  │  │  ├─ service/ItemService.java
│  │  │  ├─ repo/ItemRepository.java
│  │  │  ├─ domain/Item.java
│  │  │  ├─ ws/PriceWebSocket.java
│  │  │  └─ util/Json.java
│  │  └─ resources/
│  │     ├─ templates/
│  │     │  ├─ layout.mustache
│  │     │  ├─ index.mustache
│  │     │  └─ itemDetails.mustache
│  │     └─ static/
│  │        ├─ styles.css
│  │        └─ script.js
│  └─ test/ (optional for future Java tests)
├─ postman/
│  ├─ CollectiblesAPI.postman_collection.json
│  └─ Local.postman_environment.json
├─ docs/
│  ├─ architecture-diagram.png
│  ├─ ui-wireframes.png
│  ├─ peer-reviews.md
│  └─ decisions.md
└─ README.md
```

---

## Installation & Local Run 

Requirements
- Java JDK 17
- Maven 3.9+
- A modern browser

Quick Start
```bash
mvn clean install
mvn exec:java -Dexec.mainClass=com.collectibles.App -DENV=dev
# Open: http://localhost:4567
```

Optional environment variables (defaults shown):
```bash
export PORT=4567
export ENV=dev        # dev | test | prod
export WS_ENDPOINT=/ws/prices
```

---

## API Contracts (Planned)

Base URL: `http://localhost:4567/api/v1`

| Method | Route             | Description               | Request Body     | Responses                    |
|--------|-------------------|---------------------------|------------------|------------------------------|
| GET    | /items            | List items with filters   | —                | 200 OK `[Item]`              |
| GET    | /items/:id        | Get by id                 | —                | 200 OK `Item` • 404 NotFound |
| POST   | /items            | Create a new item         | `ItemRequest`    | 201 Created `Item` • 400     |
| PUT    | /items/:id        | Update an existing item   | `ItemRequest`    | 200 OK `Item` • 400 • 404    |
| DELETE | /items/:id        | Delete an item            | —                | 204 NoContent • 404          |
| POST   | /items/:id/offers | Create an offer for item  | `{ amount }`     | 201 Created • 400 • 404      |

### Query Parameters for `GET /items`
- `q`: full-text search
- `category`: string
- `minPrice` / `maxPrice`: numbers
- `sort`: `price` or `name`
- `order`: `asc` or `desc`
- `page`, `size`: pagination

### JSON Schema — Item (example)
```json
{
  "id": "uuid",
  "name": "Charizard Holo 1999",
  "description": "Base Set - shadowless",
  "price": 1200.00,
  "currency": "USD",
  "category": "cards",
  "tags": ["pokemon", "rare"],
  "onAuction": true,
  "updatedAt": "2025-10-27T14:05:00Z"
}
```

### Error Model (JSON)
```json
{
  "timestamp": "2025-10-27T14:06:00Z",
  "status": 404,
  "error": "Not Found",
  "message": "Item not found",
  "path": "/api/v1/items/abc-123",
  "requestId": "c141f8..."
}
```

---

## WebSockets — Real-Time Updates 
Endpoint: `ws://localhost:4567/ws/prices`

**Server → Client**
```json
{ "type": "PRICE_UPDATE", "itemId": "uuid", "newPrice": 1185.50 }
```

**Client → Server**
```json
{ "type": "SUBSCRIBE", "itemId": "uuid" }
```

Frontend bootstrap (example):
```html
<script>
  const ws = new WebSocket(`ws://${location.host}/ws/prices`);
  ws.onmessage = (e) => {
    const msg = JSON.parse(e.data);
    if (msg.type === "PRICE_UPDATE") {
      const el = document.querySelector(`[data-item="${msg.itemId}"] .price`);
      if (el) el.textContent = Number(msg.newPrice).toFixed(2);
    }
  };
</script>
```

---

# Sprint-by-Sprint Plan

## Sprint 1 — API Foundation & Project Setup

**Goals**
- Initialize Maven project; add Spark, Mustache, Jackson, SLF4J.
- Create Item domain, repository (in-memory), and service.
- Implement CRUD routes and JSON serialization helpers.
- Prepare Postman collection and environment files.

**Step-by-Step**
1. Create Maven skeleton and `pom.xml` dependencies:
   - `com.sparkjava:spark-core`
   - `com.github.spullara.mustache.java:compiler`
   - `com.fasterxml.jackson.core:jackson-databind`
   - `org.slf4j:slf4j-simple`
2. Implement `App.java` to:
   - Read `PORT` and `ENV` from environment
   - Register global before/after filters (JSON headers, timing)
   - Mount routes under `/api/v1`
3. Add `ItemRepository` (Map/UUID) and `ItemService` with validations.
4. Implement `ItemsRoutes` with CRUD and correct HTTP codes.
5. Export Postman collection `postman/CollectiblesAPI.postman_collection.json`.

**Definition of Done**
- CRUD endpoints pass manual tests in Postman.
- README endpoints table matches actual behavior.
- Evidence: screenshots in `docs/screenshots/` (placeholders now).

---

## Sprint 2 — UI Templates & Exception Handling

**Goals**
- Create Mustache templates: `layout.mustache`, `index.mustache`, `itemDetails.mustache`.
- Implement server routes for HTML pages (`/`, `/items/:id/view`).
- Create offer form posting to `/api/v1/items/:id/offers`.
- Add global error handling for 400/404/500 with friendly pages.

**Step-by-Step**
1. Add `resources/templates` and `resources/static` with CSS/JS.
2. Add a view-render helper to pass models to Mustache.
3. Render item list with filters (reusing service methods).
4. Add offer form with server-side validation and flash message pattern.
5. Style minimal UI for readability and accessibility.

**Definition of Done**
- Pages render list and detail with live data from repository.
- Offer form posts to API and shows feedback.
- Error pages return user-friendly messages.

---

## Sprint 3 — Filters & WebSockets

**Goals**
- Add query filters, sorting, and pagination to `GET /items`.
- Implement WebSocket server and client for price updates.
- Optimize broadcast logic and request processing.

**Step-by-Step**
1. Extend `ItemService.find(...)` to support `q`, `category`, `minPrice`, `maxPrice`, `sort`, `order`, `page`, `size`.
2. Add `PriceWebSocket` with `onConnect`, `onMessage`, `onClose`.
3. Create subscription model and broadcast price changes on updates/offers.
4. Add small demo button to simulate price change for evidence.

**Definition of Done**
- Filters operational; responses stable under typical list sizes.
- WS protocol verified with a demo video/gif.
- Evidence uploaded to `docs/videos/` and `docs/screenshots/`.

---

## Testing & Quality

**Postman Collection**
- `postman/CollectiblesAPI.postman_collection.json`
- `postman/Local.postman_environment.json`

**Run with Newman (CLI)**
```bash
newman run postman/CollectiblesAPI.postman_collection.json       -e postman/Local.postman_environment.json --reporters cli --bail
```

**Test Matrix (minimum)**
| Area  | Case                                  | Expected |
|-------|----------------------------------------|----------|
| CRUD  | Create with valid payload              | 201      |
| CRUD  | Create with missing fields             | 400      |
| CRUD  | Get existing item                      | 200      |
| CRUD  | Get non-existing item                  | 404      |
| CRUD  | Update with valid payload              | 200      |
| CRUD  | Delete existing item                   | 204      |
| Filter| List with `q`, `minPrice`, `maxPrice`  | 200      |
| WS    | Receive PRICE_UPDATE after change      | Message  |

**Evidence Placeholders**
- `docs/screenshots/postman_create.png`
- `docs/screenshots/postman_list.png`
- `docs/videos/ws-demo.mp4`

---

## CI/CD — GitHub Actions 
`.github/workflows/ci.yml`
```yaml
name: CI
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 17
      - name: Build
        run: mvn -B -DskipTests=false verify
```

---

## Product Backlog

| ID  | Epic        | User Story                                                    | Priority | Acceptance Criteria |
|-----|-------------|----------------------------------------------------------------|:--------:|--------------------|
| B1  | Catalog     | As a user, I want to browse collectibles with filters         | High     | Query by text/category/price; paginate; sort |
| B2  | CRUD        | As an admin, I want to manage collectibles                    | High     | Create/Update/Delete with validations        |
| B3  | Offers      | As a user, I want to submit offers on auction items           | High     | Form accepted; rules validated; feedback     |
| B4  | UI          | As a user, I want a clean server-rendered UI                  | Medium   | Mustache templates with layout               |
| B5  | Real-time   | As a user, I want instant price updates                       | High     | WS push on price changes                     |
| B6  | Errors      | As a user, I want clear error messages                        | Medium   | 400/404/500, consistent JSON and pages       |
| B7  | Observability| As a dev, I want request correlation IDs in logs             | Low      | requestId in error model & access logs       |

---

## Roadmap

| Sprint | Scope                          | Deliverables                                  |
|--------|--------------------------------|-----------------------------------------------|
| 1      | API foundation & setup         | CRUD endpoints, service, repo, Postman        |
| 2      | UI + exception handling        | Mustache views, offer form, error pages       |
| 3      | Filters + WebSockets           | Advanced filters, WS server/client, demo      |
| Future | Persistence + deployment       | DB integration, Docker, cloud deploy          |

---

## Definition of Done (Evaluation Criteria)
- API endpoints implemented, documented, and tested (Postman/Newman).
- UI templates render real data and accept offers.
- WebSockets broadcast price updates; demo evidence recorded.
- CI pipeline builds and runs tests on PR.
- README, backlog, roadmap, and peer review logs present.

---

## Peer Reviews

Process
- Open a PR from `feature/*` branches to `main`.
- Attach screenshots or short video for the feature/evidence.
- Use `docs/peer-reviews.md` to log reviewer comments and decisions.

Checklist
- [ ] Unit/Manual tests executed or described
- [ ] API responses and errors match README
- [ ] No hard-coded secrets; env vars documented
- [ ] UI usability sanity check
- [ ] Code structure: routes → service → repo separation

Template (`docs/peer-reviews.md`)
```markdown
# Peer Reviews Log
- Date:
- Reviewer:
- PR Link:
- Notes:
  1. ...
  2. ...
- Status: Approved / Changes Requested
```

---

## 🌱 Sustainability

Sustainability isn’t a footnote. It’s a design principle stitched into every sprint of this project:

### Technical Sustainability
- Architecture follows SOLID principles: maintainable code grows like a bonsai, not a wild weed.
- Minimal dependencies reduce security risk and simplify long-term updates.
- Reusable components + clean separation of concerns = future developers won’t curse our names.

### Economic Sustainability
- Entirely open-source stack: Spring Boot, Spark Java, Maven, MongoDB/PostgreSQL (depending on final architecture).
- CI pipelines and automated testing catch bugs early, lowering the cost of future enhancements.
- Scalable infrastructure avoids overspending during low-traffic periods.

### Operational Sustainability
- Server-side rendering optimizes page load even in low-bandwidth environments.
- Robust error handling minimizes support needs and user frustration.
- Documentation-first approach reduces onboarding time for new team members.

### Environmental Sustainability
- Reduced client-side script execution saves battery and processing power for users on mobile.
- Efficient query strategies shrink server workload and carbon footprint.
- Hosting flexibility allows deployment in data centers powered by renewable energy.

> A sustainable design ensures Ramón grows a long-lasting marketplace rather than a disposable gimmick.

---

## 🚀 Innovation & Business Impact

This platform isn’t just a storefront. It’s an evolution in collector culture.

### Real-Time Market Reactions
- Live pricing powered by WebSockets transforms shopping into an event: users witness supply, demand, and excitement unfold instantly.

### Search & Discoverability Enhancements
- Server-rendered content improves SEO visibility, bringing in throngs of curious enthusiasts.
- Filter-rich UI empowers collectors to navigate rare and niche items with ease.

### Future-Proof Extensibility
Micro-focused design enables rapid addition of new features:
- Secure online payments
- AI-powered recommendation engine
- Sales analytics for sellers
- Multi-language and regional catalog expansion

### Brand Reputation & Inclusivity
- Accessibility-first interface meets WCAG guidelines and gives every collector a seat at the table.
- Great UX invites casual fans to join the fun, expanding Ramón’s business beyond hardcore fans.

> This isn’t a website. It’s a marketplace with personality, built to spark joy and profits.

---

## 💰 Estimated Costs  
*(Replace numbers with your final rates when finished)*

A clearer breakdown showing sprints, complexity, and professional polish.

| Task / Deliverable                             | Description | Hours | Rate (USD/hr) | Cost (USD) |
|-----------------------------------------------|-------------|-----:|--------------:|-----------:|
| Requirements & Architecture Design            | Domain discovery, UX workflows, data modeling | 6 | 45 | 270 |
| Backend Resource Development (API & CRUD)     | Create, read, update, delete for collectibles, users, offers | 18 | 45 | 810 |
| UI Screens & Server-Side Templates            | SSR pages, dynamic listing, pagination, responsive design | 14 | 45 | 630 |
| Real-Time Features (WebSockets)               | Live pricing updates, offer notifications | 10 | 45 | 450 |
| Database Integration & Index Optimization     | Queries, schema refinement, performance indexing | 10 | 45 | 450 |
| Authentication & Validation (Optional)        | Login, signup, rules for secure interactions | 8 | 45 | 360 |
| Comprehensive Testing & Debugging             | Postman collections, integration, end-to-end | 14 | 45 | 630 |
| CI/CD Integration & Deployment Prep           | GitHub setup, workflows, staging environment | 8 | 45 | 360 |
| Documentation, Demos & Delivery               | README, run instructions, stakeholder presentation | 10 | 45 | 450 |
| **Total Estimated**                            |             | **98** |              | **4,410** |



---

## Risks & Mitigations

| Risk                                   | Impact | Mitigation                                  |
|----------------------------------------|:------:|---------------------------------------------|
| WebSocket instability under load       | High   | Throttle/batch updates; consider backoff     |
| In-memory store data loss              | Med    | Add DB persistence in a future iteration     |
| Validation gaps in offers              | Med    | Centralize validation in service layer       |
| Cross-browser WebSocket quirks         | Low    | Test on major browsers; fallback messaging   |

---

## Contribution Guidelines

Branching
- `main`: stable
- `feature/B1-filters`
- `feature/B2-crud`
- `feature/B3-offers`
- `feature/B5-websockets`

Commit style
- `feat: add item creation endpoint`
- `fix: correct 400 error payload`
- `docs: update README endpoints table`

---

## License

MIT License © 2025 Iván Kaleb Ramírez Torres

---

## Glossary

- **CRUD**: Create, Read, Update, Delete operations.
- **SSR**: Server-Side Rendering using templates.
- **WebSocket**: Full-duplex communication channel for real-time updates.
- **Postman/Newman**: Tools to validate APIs manually/automatically.
- **DoD**: Definition of Done for sprint acceptance.

---

## References & Acknowledgements

- Spark Framework official docs: https://sparkjava.com/
- Mustache.java: https://github.com/spullara/mustache.java
- WebSockets (RFC 6455): https://www.rfc-editor.org/rfc/rfc6455
- Project structure and documentation cadence inspired by prior challenge deliverables.

---

## Evidence Placeholders
```text
docs/
├─ screenshots/
│  ├─ postman_create.png
│  ├─ postman_list.png
│  ├─ ui_index.png
│  └─ ui_item_details.png
├─ videos/
│  └─ ws-demo.mp4
└─ architecture/
   ├─ sequence-ws.png
   └─ deployment.png
```
---

## 👨‍💻 Author

**Iván Kaleb Ramírez Torres**  
PhD in Materials Science & Engineering | Full-Stack & Backend Developer  

- GitHub: [@rtkaleb](https://github.com/rtkaleb)  
- LinkedIn: [Iván Kaleb Ramírez Torres](https://linkedin.com/in/ivan-kaleb-ramirez-torres)