# Biyo Dhawr 💧

> *Centralized Water Intelligence and AI Platform for Drought Resilience* — **"Xog la helayaa, Talo la helaa"** (data you can get, advice you can act on)

**Biyo Dhawr** ("water nearby" in Somali) is an open-source water-source monitoring and drought early-warning platform for rural Somaliland/Somalia, starting with the **Awdal region**. It connects three groups that today rarely talk to each other in time: the **nomads and villagers** who depend on a well, the **government workers and village leaders** who can dispatch a repair, and the **data** (SWIMS water points, community reports, sensor readings) needed to act before a source fails.

Built by a student team from Amoud University, where it won **1st place** in the university competition, and released under the MIT licence.

## The problem

Water management in rural areas is **reactive**: authorities act only after a source has dried up or a pump has failed, because there is no channel for communities to report problems and no central place where water data is collected and analysed. Biyo Dhawr turns this into **proactive, intelligence-based** management.

More than 60 % of rural households in the region depend on boreholes, dug wells, berkads and dams that fail silently: a pump breaks or a well runs dry, nobody with authority hears about it for weeks, and families and livestock walk for days to the next source. Most affected people have a basic phone but no smartphone or internet.

## How it works

```
 Nomad / villager                 Government dashboard                    Field team
 dials *999# (USSD, Somali) ──▶  report appears, staff verify  ──▶  repair dispatched
 no internet, no account         map + AI risk + priorities            well works again
```

1. **Report** — dial `*999#`, pick district → village → water source → problem (broken / dry). The report reaches the API in seconds. A smartphone app simulates the same flow.
2. **Verify and prioritise** — staff see every water point on a satellite map, triage reports, and let the risk engine rank villages by drought risk and sources by estimated days to failure.
3. **Act** — alerts are raised for High/Severe villages, repairs are logged, analytics show what is working across the region.

## Repositories

| Repo | What it is | Stack |
|---|---|---|
| [`backend`](https://github.com/biyo-dhawr/backend) | REST API, PostgreSQL schema, Socket.IO, SWIMS importer, and the **drought-risk service** (rule-based scoring + LLM reports) | Node 18, Express 5, Drizzle ORM, PostgreSQL, Python FastAPI, Groq |
| [`web`](https://github.com/biyo-dhawr/web) | Government / NGO dashboard: live map, field-report triage, AI intelligence center, analytics, exports | Next.js 14, TypeScript, Tailwind, Leaflet, Recharts, SWR |
| [`mobile`](https://github.com/biyo-dhawr/mobile) | USSD `*999#` simulator for community reporting, fully in Somali | Expo / React Native |

## Architecture

```
┌──────────────┐   HTTP/JSON + Socket.IO   ┌──────────────────────┐        ┌────────────────────┐
│  web (Next)  │ ────────────────────────▶ │  backend (Express)   │ ─────▶ │ PostgreSQL         │
└──────────────┘                           │  /api/*              │        └────────────────────┘
┌──────────────┐   HTTP/JSON (public)      │                      │        ┌────────────────────┐
│ mobile (USSD)│ ────────────────────────▶ │  Socket.IO events    │ ─────▶ │ drought-risk-service│
└──────────────┘                           └──────────────────────┘        │ FastAPI + Groq LLM │
                                                                           └────────────────────┘
```

- **Data**: 604 real water points from the SWIMS live map (UNICEF/Somalia Water Information Management System) seeded into `regions → districts → villages → water_sources`.
- **Risk engine**: explainable, rule-based scoring (water level, infrastructure state, community reports, maintenance age) returning a level (Low → Severe), a confidence score and human-readable reasons. No black box.
- **Narrative reports**: on demand, an LLM turns a source's context into an executive summary, concerns, evidence and recommended actions.
- **Real-time**: dashboards refresh over Socket.IO when sources or predictions change.

## Roles

| Role | Channel | Can |
|---|---|---|
| Community member | USSD `*999#` / mobile | Report a broken or dry source, no account needed |
| Village leader | Web | Triage reports for their district |
| Government worker | Web | Everything: sources, reports, predictions, alerts, analytics, user management |

## Run the whole stack

```bash
git clone https://github.com/biyo-dhawr/backend && cd backend
npm install && cp .env.example .env            # set DATABASE_URL, JWT_SECRET, PORT=4000
npx drizzle-kit push && npm run import:swims   # schema + SWIMS seed
node scripts/reset_admin.js                    # staff login
npm run dev                                    # API on :4000
# second terminal: cd drought-risk-service && pip install -r requirements.txt && uvicorn app.main:app --port 8000

git clone https://github.com/biyo-dhawr/web && cd web
npm install && npm run dev                     # dashboard on :3000
```

Each repository README has the detailed steps; the backend has a full [API reference](https://github.com/biyo-dhawr/backend/blob/main/docs/API.md).

## Roadmap

- Real USSD/SMS gateway integration (the API already accepts URL-encoded payloads and has a mock SMS endpoint).
- IoT sensor ingestion for `sensor_readings` (water level, soil moisture).
- NGO interventions tracking and alert resolution.
- Expand beyond Awdal to all regions in the SWIMS dataset.
- Somali-language dashboard.

## Team

[@Ibrahim-Abdirashid](https://github.com/Ibrahim-Abdirashid) · [@abdilahi-fullstack-dev](https://github.com/abdilahi-fullstack-dev) · [@AyoubKilwe](https://github.com/AyoubKilwe) · [@Abdulahia-39](https://github.com/Abdulahia-39)

## License

All repositories are released under the [MIT License](https://opensource.org/licenses/MIT).
