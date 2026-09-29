# CrowdFlow AI --- Product Requirements Document (PRD)

## Supabase + Agentic Crowd Management Prototype

### Build Specification for Google Antigravity

**Document version:** 1.0
**Product:** CrowdFlow AI
**Target:** Working prototype / hackathon-grade operational dashboard
**Primary build environment:** Google Antigravity
**Primary database/backend platform:** Supabase
**Frontend:** Next.js + React + TypeScript
**AI orchestration:** LangGraph-style stateful workflow
**Optional AI runtime:** Gemini/OpenAI-compatible model through a
server-side provider adapter

---

# 1. Product Summary

CrowdFlow AI is a station and platform crowd-management dashboard
designed to help an operator:

1. See the current crowd at a station/platform/bay.
2. Change live operational inputs such as current crowd, vehicle delay,
   arrival rate, departure rate, vehicle capacity, and forecast
   horizon.
3. Forecast the expected crowd level over the selected future horizon.
4. Identify whether the predicted crowd approaches or exceeds safe
   capacity.
5. Recommend exactly one immediate operational action.
6. Generate a short public announcement in the selected local language.
7. Persist scenarios, forecasts, actions, and announcements in
   Supabase.
8. Update the dashboard dynamically without hard-coded display values.
9. Provide a clear two-minute decision workflow: **Current state →**
   **Forecast → Risk → One action → Announcement.**

The prototype is intended to demonstrate an AI-assisted decision-support
workflow. It is not an autonomous emergency-control system and must not
claim certainty about future incidents.

---

# 2. Product Vision

## Vision

> Predict early. Move people safely. Respond clearly.

CrowdFlow AI should feel like a calm digital control room for safer
passenger flow.
The product combines:

- Live crowd monitoring
- Short-horizon forecasting
- Transport/delay context
- Capacity-risk detection
- AI-assisted operational reasoning
- Public communication generation
- Scenario simulation
- Persistent operational history

The interface should help an operator act early without overwhelming
them with data.

---

# 3. Problem Statement

Station crowd conditions can change rapidly when:

- A vehicle is delayed.
- A vehicle arrives already crowded.
- More passengers enter than expected.
- Boarding/departure capacity changes.
- A nearby event creates additional demand.
- A platform approaches its safe capacity.
- Multiple vehicles are delayed at once.

A static dashboard is not enough. The operator needs a compact decision
workflow that converts operational inputs into an understandable
forecast, risk level, recommended action, and announcement.

---

# 4. Prototype Goals

## Must-have goals

### G1 --- Dynamic inputs

The operator can change:

- Current crowd
- Platform/station capacity
- Arrival rate
- Departure rate
- Vehicle delay
- Next vehicle capacity
- Current vehicle occupancy
- Forecast horizon
- Optional event/weather impact

Changing an input must update the calculated result.

### G2 --- Forecast

The system calculates a crowd forecast for the selected horizon.
Example:
Current crowd = 420
Capacity = 600
Arrival rate = 18/min
Departure rate = 9/min
Horizon = 20 min
Estimated crowd:
`420 + (18 - 9) × 20 = 600`
The prototype should expose the calculation rather than hide it behind
an LLM.

### G3 --- Risk assessment

The system converts forecast occupancy into a risk state.
Suggested prototype thresholds:

- NORMAL: < 70%
- WATCH: 70--84%
- WARNING: 85--94%
- HIGH: 95--99%
- CRITICAL: >= 100%

Thresholds must be configurable rather than hard-coded into UI
components.

### G4 --- One recommended action

The system should return one primary recommended action, for example:

- No action
- Hold passengers at concourse
- Redirect passengers
- Open additional bay/platform
- Trigger public announcement
- Request additional transport
- Escalate to station operator

The UI must prominently show only one primary action.

### G5 --- Public announcement

Generate a short announcement based on:

- Station/platform
- Current risk
- Recommended action
- Vehicle status
- Selected language

Example:

> "Passengers are requested to remain in the concourse temporarily.
> Please follow staff instructions."

### G6 --- Persistence

Supabase stores:

- Stations
- Platforms
- Vehicles
- Crowd snapshots
- Scenarios
- Forecasts
- Risk assessments
- Recommended actions
- Announcements
- Operational events
- User profiles / operator metadata

### G7 --- Realtime updates

The dashboard should be capable of receiving updates when crowd
snapshots, alerts, or scenarios change.
Supabase provides a full PostgreSQL database and Realtime capabilities,
making it suitable for this prototype architecture.
citeturn0search0turn0search13

---

# 5. Non-Goals

The prototype does NOT need:

- Direct control of gates or physical barriers.
- Autonomous emergency dispatch.
- Automatic public broadcasting to real station speakers.
- Real-world passenger tracking.
- Facial recognition.
- Individual passenger identification.
- Guaranteed prediction accuracy.
- Fully autonomous operational decisions.
- Complex multi-city deployment.
- Production-grade transport integrations.

External transport feeds can be represented through a clean adapter
interface and demo data.

---

# 6. Primary User

## Station Control Operator

The primary user monitors one station and needs to answer:

1. How crowded is it now?
2. What will happen in the next N minutes?
3. Will capacity become unsafe?
4. What should I do right now?
5. What should I announce?

The product should answer these five questions in under two minutes.

---

# 7. Core User Journey

```
OPEN DASHBOARD
      ↓
SELECT STATION / PLATFORM
      ↓
VIEW LIVE CROWD
      ↓
CHANGE OPERATIONAL INPUTS IF NEEDED
      ↓
CLICK "RECALCULATE"
      ↓
FORECAST NEXT N MINUTES
      ↓
CALCULATE CAPACITY %
      ↓
CLASSIFY RISK
      ↓
AGENT REVIEWS STRUCTURED FACTS
      ↓
ONE RECOMMENDED ACTION
      ↓
GENERATE SHORT PA
      ↓
OPERATOR REVIEWS
      ↓
SAVE SCENARIO / ACTION
```

---

# 8. Key Product Principle

## Deterministic calculations first, AI reasoning second

Do NOT allow the LLM to invent crowd numbers, capacity calculations, or
safety thresholds.
The numerical pipeline must be deterministic:

```
Input
  ↓
Forecast Calculator
  ↓
Capacity Calculator
  ↓
Risk Engine
  ↓
Structured Situation
  ↓
Agent Orchestrator
  ↓
Recommended Action
  ↓
Announcement Generator
```

The AI agent reasons over structured outputs.

---

# 9. Recommended Technical Architecture

```
                         ┌─────────────────────┐
                         │     OPERATOR UI     │
                         │ Next.js + React     │
                         │ TypeScript          │
                         └──────────┬──────────┘
                                    │
                         Supabase JS / API
                                    │
                    ┌───────────────▼──────────────┐
                    │          SUPABASE            │
                    │                              │
                    │ PostgreSQL                   │
                    │ Auth                         │
                    │ Realtime                     │
                    │ Storage (optional)            │
                    │ Edge Functions (optional)     │
                    └───────────────┬──────────────┘
                                    │
                         Server-side API layer
                                    │
                    ┌───────────────▼──────────────┐
                    │       FORECAST SERVICE       │
                    │                              │
                    │ deterministic calculations   │
                    │ risk engine                  │
                    │ vehicle context              │
                    └───────────────┬──────────────┘
                                    │
                    ┌───────────────▼──────────────┐
                    │       AGENT ORCHESTRATOR     │
                    │          LangGraph            │
                    │                              │
                    │ Situation → Tools → Action   │
                    │ → Announcement               │
                    └───────────────┬──────────────┘
                                    │
                    ┌───────────────▼──────────────┐
                    │       MODEL PROVIDER         │
                    │ Gemini / compatible LLM      │
                    └──────────────────────────────┘
```

Supabase provides Postgres, Auth, Realtime, Storage, and Edge Functions
as integrated services. citeturn0search8turn0search3

---

# 10. Technology Stack

## Frontend

- Next.js
- React
- TypeScript
- Tailwind CSS
- Lucide React
- Recharts
- Supabase JavaScript client
- Zod
- React Hook Form where useful

## Database / Backend Platform

- Supabase
- PostgreSQL
- Supabase Auth
- Supabase Realtime
- Supabase Edge Functions only where useful

Supabase provides a real PostgreSQL database rather than a database
abstraction. citeturn0search0

## AI / Agent Layer

- LangGraph or equivalent stateful graph orchestration
- Server-side LLM provider
- Structured JSON outputs
- Tool calling

## Validation

- Zod on TypeScript boundaries
- Pydantic if a Python service is used

## Optional backend

Use FastAPI if the forecast/agent layer is implemented as a separate
Python service.
For the smallest prototype, the deterministic forecast service can
initially live in the Next.js server layer, while the agent
orchestration remains isolated behind a server-side service interface.

---

# 11. Why Supabase

Supabase is the selected persistence layer for this prototype because it
gives the project:

- Managed PostgreSQL
- Authentication
- Row Level Security
- Realtime database updates
- REST/Data API
- Optional Edge Functions
- A dashboard for inspecting data
- Easy integration with Next.js

Supabase Auth uses JWT-based authentication and integrates with
PostgreSQL Row Level Security for row-level authorization.
citeturn0search2turn0search3
The frontend should never contain a privileged Supabase service-role
key.

---

# 12. Environment Variables

Required frontend/server environment configuration:

```
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=

SUPABASE_SERVICE_ROLE_KEY=

AI_PROVIDER=
AI_API_KEY=
AI_MODEL=

NEXT_PUBLIC_APP_URL=
```

Rules:

- `NEXT_PUBLIC_*` values may be exposed to browser code.
- `SUPABASE_SERVICE_ROLE_KEY` must be server-side only.
- AI provider keys must be server-side only.
- Never commit `.env.local`.
- Create `.env.example`.

---

# 13. Supabase Database Schema

## 13.1 profiles

```
profiles
---------
id uuid primary key references auth.users(id)
full_name text
role text
created_at timestamptz
```

Roles:

```
operator
admin
viewer
```

---

## 13.2 stations

```
stations
---------
id uuid primary key
name text not null
code text unique
location text
capacity integer
status text
created_at timestamptz
updated_at timestamptz
```

Example:

```
Central Station
CEN-01
capacity = 2500
status = active
```

---

## 13.3 platforms

```
platforms
---------
id uuid primary key
station_id uuid references stations(id)
name text not null
capacity integer not null
status text
created_at timestamptz
updated_at timestamptz
```

---

## 13.4 vehicles

```
vehicles
---------
id uuid primary key
station_id uuid references stations(id)
platform_id uuid references platforms(id)
route_name text
vehicle_number text
scheduled_time timestamptz
expected_time timestamptz
capacity integer
current_occupancy integer
delay_minutes integer
status text
created_at timestamptz
updated_at timestamptz
```

Possible status:

```
scheduled
delayed
approaching
boarding
departed
cancelled
```

---

## 13.5 crowd_snapshots

```
crowd_snapshots
---------
id uuid primary key
station_id uuid references stations(id)
platform_id uuid references platforms(id)
crowd_count integer not null
captured_at timestamptz not null
source text
created_at timestamptz
```

Sources:

```
manual
sensor
camera
api
simulation
```

For the prototype, `manual` and `simulation` are acceptable.

---

## 13.6 operational_events

```
operational_events
---------
id uuid primary key
station_id uuid references stations(id)
event_type text
severity text
description text
start_time timestamptz
end_time timestamptz
impact_factor numeric
created_at timestamptz
```

Possible event types:

```
vehicle_delay
vehicle_cancellation
weather
festival
sports_event
school_event
station_issue
gate_closure
```

---

## 13.7 scenarios

This stores operator what-if calculations.

```
scenarios
---------
id uuid primary key
station_id uuid references stations(id)
platform_id uuid references platforms(id)
created_by uuid references auth.users(id)

current_crowd integer
capacity integer
arrival_rate numeric
departure_rate numeric

vehicle_delay_minutes integer
next_vehicle_capacity integer
next_vehicle_occupancy integer

forecast_horizon_minutes integer

weather_factor numeric
event_factor numeric

created_at timestamptz
```

---

## 13.8 forecasts

```
forecasts
---------
id uuid primary key
scenario_id uuid references scenarios(id)
station_id uuid references stations(id)
platform_id uuid references platforms(id)

horizon_minutes integer
predicted_crowd integer
predicted_percentage numeric

risk_level text

calculation_version text
created_at timestamptz
```

---

## 13.9 actions

```
actions
---------
id uuid primary key
forecast_id uuid references forecasts(id)

action_type text
reason text
priority text

recommended_at timestamptz
accepted_at timestamptz
status text

created_at timestamptz
```

Possible action types:

```
NO_ACTION
HOLD_CONCOURSE
OPEN_ADDITIONAL_BAY
REDIRECT_PASSENGERS
TRIGGER_PA
REQUEST_ADDITIONAL_TRANSPORT
ESCALATE_OPERATOR
```

---

## 13.10 announcements

```
announcements
---------
id uuid primary key
action_id uuid references actions(id)

language_code text
message text
status text

created_at timestamptz
```

Languages for prototype:

```
en
ta
hi
```

Additional languages can be added later.

---

# 14. Database Relationships

```
profiles
   │
   └── scenarios
          │
          └── forecasts
                 │
                 └── actions
                        │
                        └── announcements

stations
   ├── platforms
   ├── vehicles
   ├── crowd_snapshots
   ├── operational_events
   └── scenarios
```

---

# 15. Row Level Security

Enable RLS on all application tables.
Prototype policy:

### Operators

Can:

- Read stations
- Read platforms
- Read vehicles
- Read crowd snapshots
- Create scenarios
- Read own scenarios
- Read forecasts connected to their scenarios
- Read actions connected to their forecasts
- Create/read announcements connected to their actions

### Admin

Can read/write all operational records.

### Viewer

Read-only access.
Do not expose service-role operations to the browser.

---

# 16. Supabase Realtime

Subscribe to:

- `crowd_snapshots`
- `vehicles`
- `operational_events`
- `forecasts`
- `actions`

Use Realtime to update the dashboard when operational state changes.
Example:

```
New crowd snapshot
      ↓
Realtime event
      ↓
Dashboard updates
      ↓
Current crowd card changes
```

Supabase Realtime supports listening to PostgreSQL database changes.
citeturn0search13

---

# 17. Forecast Engine

## Base model

For the prototype, use a transparent short-horizon flow equation.

```
predicted_crowd =
current_crowd
+ expected_arrivals
- expected_departures
```

With constant rates:

```
expected_arrivals =
arrival_rate × horizon_minutes

expected_departures =
departure_rate × horizon_minutes
```

Therefore:

```
predicted_crowd =
current_crowd
+
(arrival_rate - departure_rate)
× horizon_minutes
```

Clamp the result at zero.

---

# 18. Vehicle Adjustment

The forecast engine should account for vehicle delay and vehicle
capacity.
If a delayed vehicle causes passengers to remain at the platform, the
delay should influence expected departure timing.
Prototype approach:

```
effective_departure_time =
scheduled_departure + delay_minutes
```

If the vehicle is not expected to depart within the selected horizon, do
not count its normal departure as occurring inside the horizon.
If a vehicle arrives during the horizon, calculate the expected number
of passengers that can leave the platform:

```
boarding_capacity =
vehicle_capacity - vehicle_current_occupancy
```

Then:

```
actual_boarding =
min(waiting_passengers, boarding_capacity)
```

The detailed boarding model can be simplified for the first prototype.

---

# 19. Optional Event/Weather Factors

Use simple multipliers rather than an opaque AI adjustment.
Example:

```
effective_arrival_rate =
base_arrival_rate × event_factor × weather_factor
```

Defaults:

```
normal event factor = 1.0
moderate event factor = 1.15
major event factor = 1.30

normal weather factor = 1.0
moderate disruption = 1.10
severe disruption = 1.20
```

These values are prototype assumptions and must be configurable.

---

# 20. Risk Engine

Calculate:

```
occupancy_percentage =
predicted_crowd / capacity × 100
```

Suggested configurable thresholds:

```
0–69%    NORMAL
70–84%   WATCH
85–94%   WARNING
95–99%   HIGH
100%+    CRITICAL
```

The risk engine returns:

```
{
  "predictedCrowd": 575,
  "capacity": 600,
  "occupancyPercentage": 95.83,
  "riskLevel": "HIGH"
}
```

The LLM must not calculate these values itself.

---

# 21. Action Recommendation Engine

Before calling the LLM, construct a structured situation object.
Example:

```
{
  "station": "Central Station",
  "platform": "Platform 2",
  "currentCrowd": 520,
  "capacity": 600,
  "forecastHorizonMinutes": 20,
  "predictedCrowd": 610,
  "occupancyPercentage": 101.7,
  "riskLevel": "CRITICAL",
  "nextVehicle": {
    "delayMinutes": 8,
    "capacity": 800,
    "currentOccupancy": 760,
    "availableCapacity": 40
  },
  "arrivalRate": 16,
  "departureRate": 4
}
```

---

# 22. Agentic Architecture

Use one orchestrator agent initially.
Do not build multiple autonomous agents for the prototype unless there
is a clear need.

## Agent graph

```
START
  ↓
LOAD CONTEXT
  ↓
VALIDATE INPUTS
  ↓
RUN FORECAST TOOL
  ↓
RUN RISK TOOL
  ↓
RUN VEHICLE CONTEXT TOOL
  ↓
REASON ABOUT ACTION
  ↓
SELECT ONE ACTION
  ↓
GENERATE ANNOUNCEMENT
  ↓
VALIDATE OUTPUT
  ↓
SAVE RESULT
  ↓
END
```

---

# 23. Agent Tools

The agent should have access to explicit tools.

## Tool 1 --- get_station_context

Returns:

- Station
- Platform
- Capacity
- Current crowd
- Current vehicle state
- Recent crowd trend

## Tool 2 --- calculate_forecast

Input:

```
{
  "currentCrowd": 420,
  "capacity": 600,
  "arrivalRate": 18,
  "departureRate": 9,
  "horizonMinutes": 20
}
```

Output:

```
{
  "predictedCrowd": 600,
  "occupancyPercentage": 100,
  "riskLevel": "CRITICAL"
}
```

## Tool 3 --- get_vehicle_context

Returns:

- Next vehicle
- Scheduled time
- Expected time
- Delay
- Capacity
- Current occupancy
- Available capacity

## Tool 4 --- get_operational_events

Returns relevant:

- Delays
- Cancellations
- Events
- Weather conditions
- Station disruptions

## Tool 5 --- recommend_action

This can be deterministic first and agent-assisted second.
Input:

```
{
  "riskLevel": "HIGH",
  "occupancyPercentage": 96,
  "vehicleAvailableCapacity": 40,
  "arrivalRate": 16,
  "departureRate": 4
}
```

Output:

```
{
  "action": "HOLD_CONCOURSE",
  "reason": "Predicted platform occupancy is near capacity and the next vehicle has limited available capacity."
}
```

## Tool 6 --- generate_announcement

Input:

```
{
  "language": "ta",
  "station": "Central Station",
  "platform": "Platform 2",
  "action": "HOLD_CONCOURSE",
  "riskLevel": "HIGH"
}
```

Output:

```
{
  "message": "பயணிகள் தற்காலிகமாக கான்கோர்ஸில் காத்திருக்குமாறு கேட்டுக்கொள்ளப்படுகிறார்கள். பணியாளர்களின் அறிவுறுத்தல்களைப் பின்பற்றவும்."
}
```

---

# 24. Agent Output Contract

The agent must return strict structured JSON:

```
{
  "summary": "Platform 2 is approaching safe capacity.",
  "riskLevel": "HIGH",
  "recommendedAction": {
    "type": "HOLD_CONCOURSE",
    "label": "Hold at Concourse",
    "reason": "Projected occupancy reaches 96% within 20 minutes."
  },
  "announcement": {
    "language": "en",
    "text": "Passengers are requested to wait in the concourse temporarily."
  },
  "confidence": "medium",
  "dataFreshness": "current"
}
```

Do not allow free-form output to directly drive UI state.

---

# 25. Safety / Trust Rules

Never display:

> AI predicts a disaster with certainty.

Use:

- Estimated Risk: HIGH
- AI-assisted assessment
- Based on available data
- Projected crowd level

Never imply:

- certainty
- guaranteed safety
- replacement of official emergency services
- autonomous authority over station operations

For a production system, official operational procedures and responsible
human approval would be required.

---

# 26. Decision Logic

The prototype can combine deterministic rules with agent reasoning.
Example:

```
IF risk = NORMAL
    → NO_ACTION

IF risk = WATCH
    → MONITOR

IF risk = WARNING
    → TRIGGER_PA or HOLD_CONCOURSE
      depending on vehicle capacity and trend

IF risk = HIGH
    → HOLD_CONCOURSE / REDIRECT_PASSENGERS /
      OPEN_ADDITIONAL_BAY

IF risk = CRITICAL
    → immediate crowd-control action
      + public announcement
      + operator escalation
```

The exact action must be selected using available station capabilities.
Do not recommend opening a bay/platform that does not exist.

---

# 27. Dynamic What-If Simulator

The main dashboard must contain a "What-if" or "Scenario" panel.
Inputs:

```
Current Crowd
[ 420 ]

Capacity
[ 600 ]

Arrival Rate / min
[ 18 ]

Departure Rate / min
[ 9 ]

Vehicle Delay
[ +8 min ]

Next Vehicle Capacity
[ 800 ]

Next Vehicle Occupancy
[ 760 ]

Forecast Horizon
[ 20 min ]

Event Impact
[ Normal ]

Weather Impact
[ Normal ]
```

Primary button:

```
Recalculate
```

After clicking:

```
Forecast
575 people

Capacity
600

Occupancy
95.8%

Risk
HIGH

Recommended Action
Hold at Concourse

Announcement
Generated in selected language
```

---

# 28. Dashboard Information Architecture

## Navigation

```
Overview
Live Crowd
Forecast
Actions
Announcements
Settings
```

---

# 29. Overview Page

The Overview page is the primary decision page.
Layout:

```
TOP HEADER

SIDEBAR | HERO / LIVE SUMMARY | RIGHT STATUS PANEL

        CURRENT CROWD
        FORECAST
        RISK
        NEXT VEHICLE

        FEATURE CARDS

        LIVE ALERTS

        QUICK ACTIONS
```

---

# 30. Live Crowd Page

Show:

- Station selector
- Platform selector
- Current crowd count
- Capacity
- Occupancy percentage
- Crowd trend
- Recent snapshots
- Platform cards

Example:

```
Platform 1
420 / 600
70%
WATCH

Platform 2
510 / 600
85%
WARNING

Platform 3
210 / 500
42%
NORMAL
```

Use charts only where they clarify the trend.

---

# 31. Forecast Page

Show:

- Current crowd
- Forecast horizon selector
- Forecast line chart
- Predicted crowd
- Capacity line
- Risk band
- Scenario inputs
- Recalculate button

Suggested horizon choices:

```
5 min
10 min
15 min
20 min
30 min
```

The default should be 20 minutes.

---

# 32. Actions Page

Show:

- Current recommended action
- Reason
- Risk
- Time generated
- Accept action button
- Dismiss/review button
- Action history

Example:

```
RECOMMENDED ACTION

Hold passengers at concourse

WHY

Projected occupancy reaches 96% in 20 minutes,
while the next vehicle has limited available capacity.

[ Review Action ]
```

---

# 33. Announcements Page

Show:

- Generated announcements
- Language
- Associated action
- Timestamp
- Copy button
- Play button (prototype text-to-speech optional)
- Regenerate button

Languages:

```
English
Tamil
Hindi
```

---

# 34. Settings Page

Settings should include:

- Station configuration
- Capacity thresholds
- Default forecast horizon
- Supported languages
- Action availability
- Notification preferences
- User role

---

# 35. Visual Design System

The existing CrowdFlow AI Lavender/Purple design system is authoritative
for visual implementation.
The UI should be:

- Dark
- Premium
- Calm
- Modern
- Friendly
- Slightly dreamy
- Visually rich without being cluttered

The existing design system specifies the deep navy-black background,
purple/lavender/neon-pink palette, rounded cards, glowing borders, soft
shadows, and lavender typography. fileciteturn1file0

---

# 36. Exact Color Tokens

```
Primary Background: #07071F
Secondary Background: #0B0928
Surface Background: #100C35
Elevated Surface: #151044

Primary Purple: #7C3AED
Bright Purple: #8B5CF6
Lavender: #A78BFA
Light Lavender: #C4B5FD

Primary Pink: #EC4899
Bright Pink: #F472B6
Soft Pink: #F9A8D4

Primary Text: #F8F7FF
Secondary Text: #C4B5FD
Muted Text: #8F88B5

Default Border: #29205C
Purple Border: #5B3BBF
Active Border: #8B5CF6

Progress Purple: #8B5CF6
Progress Pink: #EC4899
Progress Track: #211A4B
```

These colors must not be replaced by a generic blue dashboard palette.

---

# 37. Typography

Preferred:

```
Inter
```

Fallback:

```
system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif
```

Sizes:

```
Page Heading: 32–42px / 700–800
Section Heading: 20–24px / 600–700
Card Heading: 16–18px / 600
Body: 14–15px / 400–500
Small Label: 12–13px / 500
```

---

# 38. Layout

Desktop:

```
Sidebar: 220–250px
Main: flexible
Right panel: 300–340px
```

Mobile:

```
Top header
Main content
Bottom navigation
```

Tablet:

```
Sidebar
Main content
Right panel collapsed into content
```

---

# 39. Core UI Components

Create reusable components:

```
AppShell
Sidebar
TopHeader
HeroCard
LiveStatusCard
CrowdMetricCard
CrowdProgress
RiskBadge
ForecastChart
CapacityGauge
VehicleStatusCard
ScenarioPanel
RecommendationCard
AnnouncementCard
AlertList
QuickActionGrid
StationSelector
PlatformSelector
LanguageSelector
LoadingState
EmptyState
ErrorState
```

---

# 40. Risk Visual Language

Use status chips:

```
NORMAL
WATCH
WARNING
HIGH
CRITICAL
```

Do not communicate risk only through color.
Every risk state must include:

- Label
- Icon
- Text explanation
- Accessible status

Example:

```
HIGH
Projected occupancy is 96% in 20 minutes.
```

---

# 41. Hero Section

Use the existing hero hierarchy:

```
SMART MONITORING, SAFER FLOW

Crowd levels
under control

Monitor the crowd. Predict the next 20 minutes. Respond early.

[ View Live Crowd → ]
```

Hero illustration:

- Modern station platform
- Passenger flow
- Digital display
- Platform/bay markers
- Purple lighting
- Calm organized atmosphere
- Editorial illustration

Do not use photorealistic imagery.

---

# 42. Feature Cards

Four cards:

### Live Crowd

```
See current platform capacity.
[ View Crowd → ]
```

### Crowd Forecast

```
Predict the next 20 minutes.
[ View Forecast → ]
```

### Recommended Action

```
One immediate response.
[ Take Action → ]
```

### Public Announcement

```
Generate a short local-language PA.
[ Play Announcement → ]
```

---

# 43. Live Alerts

Example alerts:

```
Platform 1
62%
NORMAL

Platform 2
84%
WARNING

Bay 3
Next vehicle full

Gate area
Crowd rising
```

These are examples only. The implementation must load actual rows from
Supabase or calculated state.

---

# 44. Quick Actions

```
Open Bay
Hold at Concourse
Generate PA
View Forecast
```

Only enable actions supported by the selected station configuration.

---

# 45. API / Service Contracts

## POST /api/scenarios

Create a scenario.
Request:

```
{
  "stationId": "...",
  "platformId": "...",
  "currentCrowd": 420,
  "capacity": 600,
  "arrivalRate": 18,
  "departureRate": 9,
  "vehicleDelayMinutes": 8,
  "nextVehicleCapacity": 800,
  "nextVehicleOccupancy": 760,
  "forecastHorizonMinutes": 20
}
```

---

## POST /api/forecast

Returns:

```
{
  "predictedCrowd": 575,
  "occupancyPercentage": 95.8,
  "riskLevel": "HIGH"
}
```

---

## POST /api/recommendation

Returns:

```
{
  "actionType": "HOLD_CONCOURSE",
  "label": "Hold at Concourse",
  "reason": "Projected occupancy is near safe capacity."
}
```

---

## POST /api/announcement

Request:

```
{
  "actionType": "HOLD_CONCOURSE",
  "language": "ta",
  "stationName": "Central Station",
  "platformName": "Platform 2"
}
```

Response:

```
{
  "message": "..."
}
```

---

# 46. Error Handling

The UI must handle:

### Database failure

```
Unable to load station data.
Retry
```

### Forecast failure

```
Forecast unavailable.
Check inputs and retry.
```

### AI failure

The system must still display deterministic forecast/risk.

```
AI recommendation unavailable.
Manual operational review required.
```

### Missing data

Do not invent values.
Display:

```
Data unavailable
```

---

# 47. Loading States

Use skeleton loaders for:

- Crowd metrics
- Forecast chart
- Recommendation card
- Announcement card

Avoid blank screens.

---

# 48. Demo Mode

The prototype should include a Demo Mode.
Purpose:
Judges/operators can demonstrate the full workflow without waiting for
real transport events.
Demo Mode should allow:

```
Normal
Busy
Vehicle Delayed
Near Capacity
Critical
```

These presets must populate input controls, but the resulting forecast
must still be calculated through the real application logic.
Do not hard-code the final result into UI components.

---

# 49. Example Demo Scenario

Preset:

```
Current crowd: 510
Capacity: 600
Arrival rate: 16/min
Departure rate: 5/min
Vehicle delay: 8 min
Next vehicle capacity: 800
Next vehicle occupancy: 760
Horizon: 20 min
```

Base projection:

```
510 + (16 - 5) × 20
= 730
```

Then vehicle timing/capacity logic should adjust the scenario if the
next vehicle is expected to board passengers within the horizon.
The final result must come from the forecast engine, not from a static
demo string.

---

# 50. City / External Cooperation Model

The architecture should leave room for external operational context.
Potential future integrations:

- City transport authority
- Bus APIs
- Train APIs
- Station management systems
- Event calendars
- Weather services

Represent external sources using adapters:

```
TransportProvider
EventProvider
WeatherProvider
StationProvider
```

For the prototype, implement:

```
MockTransportProvider
MockEventProvider
MockWeatherProvider
```

The application should not be tightly coupled to one external API.

---

# 51. External Data Criteria

When adding real integrations later, consider:

### Transport

- Scheduled arrival
- Expected arrival
- Delay
- Cancellation
- Vehicle capacity
- Current occupancy if available

### Passenger flow

- Current crowd
- Arrival rate
- Departure rate
- Boarding rate
- Alighting rate

### Station

- Platform capacity
- Concourse capacity
- Open/closed platforms
- Gate status
- Additional bay availability

### City context

- Major event
- Road disruption
- Weather
- Public transport disruption
- Nearby station congestion

---

# 52. Folder Structure

Recommended:

```
crowdflow-ai/
│
├── app/
│   ├── (dashboard)/
│   │   ├── page.tsx
│   │   ├── live-crowd/
│   │   ├── forecast/
│   │   ├── actions/
│   │   ├── announcements/
│   │   └── settings/
│   │
│   ├── login/
│   ├── api/
│   │   ├── forecast/
│   │   ├── scenarios/
│   │   ├── recommendation/
│   │   └── announcement/
│   │
│   ├── layout.tsx
│   └── globals.css
│
├── components/
│   ├── layout/
│   ├── dashboard/
│   ├── crowd/
│   ├── forecast/
│   ├── actions/
│   ├── announcements/
│   ├── charts/
│   └── ui/
│
├── lib/
│   ├── supabase/
│   ├── forecast/
│   ├── risk/
│   ├── actions/
│   ├── agents/
│   ├── providers/
│   └── validation/
│
├── types/
│   ├── database.ts
│   ├── forecast.ts
│   ├── action.ts
│   └── agent.ts
│
├── supabase/
│   ├── migrations/
│   ├── seed.sql
│   └── config.toml
│
├── public/
│
├── tests/
│   ├── forecast/
│   ├── risk/
│   ├── api/
│   └── components/
│
├── .agents/
│   ├── rules/
│   └── agents/
│
├── AGENTS.md
├── GEMINI.md
├── .env.example
├── package.json
└── README.md
```

---

# 53. Antigravity Development Rules

Antigravity supports persistent project rules through files such as
`AGENTS.md`, `GEMINI.md`, and `.agents/rules/`. citeturn0search15
Create `AGENTS.md` containing:

```
# CrowdFlow AI Engineering Rules

1. Build incrementally.
2. Do not rewrite working features unnecessarily.
3. Never hard-code operational results into UI components.
4. All crowd calculations must come from forecast/risk services.
5. Never let the LLM directly calculate safety-critical numerical values.
6. Validate all API inputs.
7. Keep Supabase service-role credentials server-side.
8. Use Row Level Security.
9. Use TypeScript strict mode.
10. Use reusable components.
11. Follow the CrowdFlow lavender/purple design system exactly.
12. Use Lucide icons instead of emoji UI icons.
13. Do not introduce a blue corporate dashboard theme.
14. Make every page responsive.
15. Add loading, error, and empty states.
16. Test calculations before connecting AI.
17. Prefer deterministic tools over free-form AI logic.
18. Never claim prediction certainty.
19. Do not invent missing data.
20. After each implementation stage, run tests and verify the browser UI.
```

---

# 54. Antigravity Build Strategy

Do NOT ask Antigravity to generate the entire application in one
uncontrolled step.
Use this sequence.

## Phase 1 --- Requirements and architecture

Antigravity should:

- Read PRD
- Inspect repository
- Propose implementation plan
- Confirm dependencies
- Create architecture artifacts

Deliverables:

```
architecture.md
folder structure
data flow diagram
```

---

## Phase 2 --- Project scaffold

Build:

- Next.js
- TypeScript
- Tailwind
- Supabase client
- ESLint
- basic route structure
- environment validation

Acceptance:

```
npm run dev
```

works.

---

## Phase 3 --- Supabase setup

Create:

- migrations
- tables
- indexes
- RLS
- seed data
- typed database definitions

Acceptance:

- Supabase connects
- station data loads
- sample platform data loads

---

## Phase 4 --- Design system

Implement:

- colors
- typography
- spacing
- cards
- buttons
- inputs
- status badges
- navigation
- responsive shell

Acceptance:

- UI visually follows the existing CrowdFlow Lavender/Purple design
  system.

---

## Phase 5 --- Dashboard shell

Build:

- sidebar
- header
- hero
- right panel
- live status
- feature cards
- responsive layout

No AI yet.

---

## Phase 6 --- Live Crowd

Build:

- station selector
- platform selector
- crowd cards
- capacity indicators
- crowd trend
- Supabase queries
- Realtime subscription

---

## Phase 7 --- Forecast Engine

Implement deterministic functions.
Unit tests:

```
current crowd only
positive net flow
negative net flow
zero net flow
capacity exceeded
horizon changes
delay changes
vehicle boarding
```

---

## Phase 8 --- Risk Engine

Implement configurable thresholds.
Test:

```
69%
70%
84%
85%
94%
95%
99%
100%
```

---

## Phase 9 --- What-if Simulator

Connect inputs to forecast engine.
Acceptance:
Changing any input changes the calculated output where mathematically
relevant.

---

## Phase 10 --- Agent Orchestrator

Add:

```
context tool
forecast tool
risk tool
vehicle tool
recommendation tool
announcement tool
```

Use structured outputs.

---

## Phase 11 --- Recommendation UI

Display:

```
Risk
Forecast
One Action
Reason
```

Add:

```
Review
Accept
Dismiss
```

---

## Phase 12 --- Announcement UI

Add:

- language selector
- generate
- regenerate
- copy
- optional play

---

## Phase 13 --- Demo Mode

Add presets:

```
Normal
Busy
Delayed
Near Capacity
Critical
```

Verify that presets still run through the same real calculation
pipeline.

---

## Phase 14 --- Realtime

Connect:

```
crowd snapshots
vehicle updates
alerts
forecast updates
```

to Supabase Realtime.

---

## Phase 15 --- Testing

Run:

```
unit tests
integration tests
type checking
lint
build
browser tests
```

---

## Phase 16 --- Final Polish

Check:

- responsive layout
- keyboard navigation
- loading states
- errors
- empty states
- animations
- visual consistency
- database security
- environment variables
- README

---

# 55. Acceptance Criteria

## Product

- [ ] Operator can select a station.
- [ ] Operator can select a platform.
- [ ] Current crowd loads dynamically.
- [ ] Operator can edit scenario inputs.
- [ ] Forecast horizon can be changed.
- [ ] Recalculate updates forecast.
- [ ] Risk is calculated from forecast/capacity.
- [ ] Vehicle delay influences the scenario.
- [ ] Vehicle capacity is considered.
- [ ] One recommendation is displayed.
- [ ] Announcement can be generated.
- [ ] Announcement language can be changed.
- [ ] Scenario can be saved.
- [ ] Historical scenarios can be viewed.

## Database

- [ ] Supabase PostgreSQL connected.
- [ ] Migrations exist.
- [ ] Seed data exists.
- [ ] RLS enabled.
- [ ] Service-role key never reaches browser.
- [ ] Realtime subscription works.

## AI

- [ ] AI receives structured data.
- [ ] AI cannot directly modify database records without server
  validation.
- [ ] AI does not calculate core numeric forecast values.
- [ ] AI returns structured output.
- [ ] AI failures do not break deterministic forecasting.
- [ ] Generated announcements are reviewable.

## UI

- [ ] Lavender/purple design system preserved.
- [ ] Responsive desktop/tablet/mobile.
- [ ] No emoji interface icons.
- [ ] Status is not conveyed only through color.
- [ ] Keyboard focus states exist.
- [ ] Loading states exist.
- [ ] Error states exist.

---

# 56. Testing Strategy

## Forecast unit tests

Example:

```
Input:
current = 100
arrival = 10/min
departure = 5/min
horizon = 10

Expected:
150
```

## Risk tests

```
420 / 600 = 70%
→ WATCH
```

```
510 / 600 = 85%
→ WARNING
```

```
600 / 600 = 100%
→ CRITICAL
```

## Scenario tests

Change:

```
horizon 20 → 10
```

Expected:
Forecast decreases if net flow is positive.
Change:

```
delay 0 → 10
```

Expected:
Vehicle departure timing changes where the vehicle falls inside the
forecast horizon.

## UI tests

Verify:

- Recalculate button
- Station selector
- Platform selector
- Forecast chart
- Risk badge
- Action card
- Announcement generation
- Responsive navigation

---

# 57. Performance Requirements

For prototype:

- Initial dashboard render target: under 2 seconds on normal
  development/deployed conditions.
- Forecast calculation should feel immediate.
- Realtime updates should update visible metrics without full-page
  refresh.
- Avoid unnecessary database polling.
- Debounce rapid input changes if auto-preview is added later.

---

# 58. Security Requirements

- Never expose service-role keys.
- Enable RLS.
- Validate all user input.
- Validate agent output.
- Do not allow arbitrary action types from client payloads.
- Server must verify station/platform/action compatibility.
- Sanitize generated announcement text before display.
- Keep audit history for accepted recommendations.

---

# 59. Observability

Store:

- scenario ID
- forecast ID
- agent execution ID where available
- model name
- calculation version
- timestamps
- selected action
- operator acceptance/dismissal

This allows later evaluation of the prototype.

---

# 60. Future ML Upgrade Path

The first prototype uses deterministic flow equations.
Later, the forecast service can be upgraded to:

```
Historical data
      ↓
Feature engineering
      ↓
Time-series model
      ↓
Prediction interval
      ↓
Risk engine
      ↓
Agent
```

Possible future models:

- Exponential smoothing
- ARIMA
- Prophet
- Gradient boosting
- LSTM/temporal neural model
- Specialized time-series model

The UI and database contracts should remain stable so the forecast
implementation can change independently.

---

# 61. Evaluation Metrics

For future real-data validation:

### Forecast

- MAE
- RMSE
- MAPE where appropriate
- Capacity breach recall
- Capacity breach precision

### Decision support

- Correct risk classification
- Action appropriateness
- Time-to-warning
- Operator acceptance rate

### Communication

- Announcement correctness
- Language correctness
- Length
- Action clarity

For the prototype, these metrics can be demonstrated using test
scenarios rather than claimed as real-world performance.

---

# 62. Example End-to-End Run

Initial state:

```
Station: Central Station
Platform: 2

Current crowd: 420
Capacity: 600

Arrival rate: 18/min
Departure rate: 9/min

Next vehicle:
Capacity: 800
Occupancy: 760
Delay: +8 min

Horizon: 20 min
```

Operator clicks:

```
RECALCULATE
```

Forecast engine calculates the projected state.
Risk engine calculates:

```
occupancy %
risk level
```

Agent receives:

```
current state
forecast
risk
vehicle context
station capabilities
```

Agent returns:

```
Recommended Action:
Hold at Concourse

Reason:
Projected platform occupancy is approaching/exceeding
safe capacity while the next vehicle has limited available
boarding capacity within the forecast window.
```

Announcement:

```
Passengers are requested to remain in the concourse temporarily.
Please follow staff instructions.
```

Operator reviews and accepts.
The scenario, forecast, action, and announcement are saved to Supabase.

---

# 63. Judge / Demo Story

The complete demonstration should take approximately two minutes.

### Scene 1 --- Calm

Show:

```
Current Crowd: 420 / 600
Risk: NORMAL
```

### Scene 2 --- Change conditions

Operator changes:

```
Arrival rate
Vehicle delay
Forecast horizon
```

### Scene 3 --- Recalculate

Show:

```
Forecast rising
Capacity line approached
Risk changes
```

### Scene 4 --- AI decision support

Show:

```
ONE RECOMMENDED ACTION
```

### Scene 5 --- Communication

Generate:

```
English / Tamil / Hindi
```

### Scene 6 --- Save

Show:

```
Scenario saved
Action logged
Announcement logged
```

The story should be immediately understandable:

```
EARLY WARNING
      ↓
PREDICT CROWD
      ↓
IDENTIFY RISK
      ↓
RECOMMEND ONE ACTION
      ↓
COMMUNICATE CLEARLY
```

---

# 64. What Antigravity Must NOT Do

Do not:

- Build a static mockup only.
- Hard-code forecast results into cards.
- Use fake AI text disconnected from calculations.
- Put database credentials in frontend code.
- Create dozens of unnecessary agents.
- Use an LLM to perform arithmetic that can be deterministic.
- Replace the purple/lavender visual system with a generic dashboard.
- Add excessive glassmorphism.
- Add excessive neon.
- Use emoji as navigation icons.
- Make every UI element glow.
- Claim real-world prediction accuracy without evidence.
- Pretend external transport APIs are connected when they are not.

---

# 65. Definition of Done

The prototype is complete when:

```
SUPABASE
   ↓
station/platform data loads
   ↓
operator changes inputs
   ↓
forecast recalculates
   ↓
risk updates
   ↓
vehicle context is considered
   ↓
agent recommends one action
   ↓
announcement is generated
   ↓
operator reviews it
   ↓
result is saved
   ↓
dashboard reflects the saved state
```

And:

```
npm run lint
npm run typecheck
npm test
npm run build
```

complete successfully.

---

# 66. Final Implementation Instruction for Antigravity

Build CrowdFlow AI as a working, polished prototype rather than a static
design mockup.
Start by reading this PRD and the existing CrowdFlow AI design-system
file.
Then:

1. Inspect the repository.
2. Create the implementation plan.
3. Create the folder structure.
4. Create `AGENTS.md`.
5. Scaffold the Next.js application.
6. Connect Supabase.
7. Create migrations and seed data.
8. Implement the design system.
9. Implement the dashboard shell.
10. Implement live crowd.
11. Implement deterministic forecast.
12. Implement risk engine.
13. Implement scenario simulator.
14. Implement agent orchestration.
15. Implement recommendation.
16. Implement announcements.
17. Implement Realtime.
18. Implement Demo Mode.
19. Add tests.
20. Run browser verification.
21. Fix all obvious issues.
22. Produce a final README explaining setup and environment variables.

After each major stage, report:

```
Implemented:
Files changed:
Tests run:
Result:
Next stage:
```

Do not move forward when the current stage has obvious build, type,
runtime, or UI errors.

---

# 67. Antigravity First Prompt

Use the following as the first implementation prompt after placing this
PRD in the repository:

> Read `CrowdFlow_AI_PRD.md` and the existing CrowdFlow AI design-system
> documentation completely before changing code.
>
> First inspect the repository and create a detailed implementation
> plan. Do not generate the entire application in one step.
>
> The application must be a real dynamic prototype backed by Supabase
> PostgreSQL. Do not hard-code forecast results or operational states
> into UI components.
>
> Use Next.js + React + TypeScript for the frontend, Supabase for
> database/auth/realtime, and a server-side agent orchestration layer
> for AI-assisted recommendation and announcement generation.
>
> Keep all safety-critical numerical calculations deterministic. The AI
> agent must reason over structured forecast/risk/vehicle outputs rather
> than inventing numbers.
>
> Preserve the exact CrowdFlow AI dark lavender/purple visual language
> from the supplied design system.
>
> Begin with:
>
> 1. Repository inspection.
> 2. Architecture plan.
> 3. Folder structure.
> 4. `AGENTS.md`.
> 5. Supabase schema/migrations plan.
>
> Do not start building later stages until the architecture and schema
> are internally consistent.

---

# 68. Reference Documentation

Supabase documentation confirms that each project includes a full
PostgreSQL database and that Supabase also provides Auth, Realtime,
Storage, and Edge Functions. citeturn0search0turn0search8
Google Antigravity documentation describes Antigravity as an agentic
development environment capable of working across the editor, terminal,
browser, tasks, artifacts, and parallel agents.
citeturn0search9turn0search4
Antigravity supports persistent workspace rules through `AGENTS.md`,
`GEMINI.md`, and `.agents/rules/`, which is why this PRD includes a
dedicated engineering-rules section. citeturn0search15