IMPSAS: Intelligent Missing Person Search and Alert System

A crowdsourced, AI-powered web app for finding missing persons. Anyone can report a sighting, officers manage cases, and facial recognition matches new photos against registered missing-person records. When a match is found, alerts go out over WhatsApp.

Built at Vishwakarma Institute of Technology (VIT), Pune. guided by Mrs. Swati Atul Joshi.

Table of Contents

- Features Overview
- Architecture
- Project Structure
- Backend (Node.js)
- ML Service (Flask)
- Frontend (React)
- Roles and Authentication
- How Matching Works
- Getting Started
- Environment Variables
- Tech Stack
- Known Issues and Roadmap
- Team and Contributors


 Features Overview

What it does

- Crowdsourced reporting: the public can submit photos and locations of possible sightings
- Facial recognition: uploaded photos are compared against missing-person records (DeepFace)
- Confidence scoring: every match gets a score so officers can judge how strong it is
- Map view: case and sighting locations shown on an interactive Leaflet.js map (OpenStreetMap tiles)
- Location geocoding: addresses are converted to coordinates with Nominatim
- WhatsApp alerts: automated notifications through the Twilio WhatsApp API (3 message templates)
- Role-based access: separate views and permissions for Public, Officer, and Admin
- Admin panel: manage cases, users, and the database from one place
- Real-time updates: Socket.io for live alert delivery

Screens

- LoginScreen / RegisterScreen: sign in and sign up, phone number collected for alerts
- LocationScreen: map-based location selection and viewing
- MatchesScreen: matched results with confidence scores
- DashboardScreen: overview of cases and activity
- AlertsScreen: alert history and status
- AdminScreen: Cases, Users, and Database tabs


Architecture

┌──────────────────────────────────────────────────────────────┐
│                       USER'S BROWSER                         │
│                                                              │
│   React App (react-scripts 5.0.1)                            │
│   Login · Register · Location · Matches · Dashboard ·        │
│   Alerts · Admin                                             │
│   Leaflet.js + OpenStreetMap · Socket.io client              │
└───────────────┬──────────────────────────────────────────────┘
                │ REST + WebSocket
                ▼
┌──────────────────────────────────────────────────────────────┐
│                  Node.js Backend (Express)                   │
│                                                              │
│   JWT auth + RBAC (Public / Officer / Admin)                 │
│   Case + user routes · Socket.io server                      │
│   Twilio WhatsApp notifications                              │
└───────┬──────────────────────────────┬───────────────────────┘
        │                              │
        ▼                              ▼
┌──────────────────────┐    ┌──────────────────────────────────┐
│   MongoDB Atlas      │    │   Flask ML Service               │
│   users, cases,      │    │   DeepFace · YOLOv8 · Whisper    │
│   matches, alerts    │    │   match.py → confidence score    │
└──────────────────────┘    └──────────────────────────────────┘
                                            
        Twilio WhatsApp API  ◄── alerts sent from the backend

Project Structure

IMPSAS/
│
├── backend/                  # Node.js API server
│   ├── idx.js                # Entry point: server setup, MongoDB connection, routes
│   ├── .env                  # Secrets (never commit)
│   └── package.json
│
├── ml/                       # Flask service for recognition
│   ├── match.py              # Face matching + confidence scoring
│   └── requirements.txt
│
└── frontend/                 # React web app
    ├── public/
    ├── package.json
    └── src/
        ├── App.jsx           # Routing and screen navigation
        └── screens/
            ├── LoginScreen.jsx
            ├── RegisterScreen.jsx
            ├── LocationScreen.jsx
            ├── MatchesScreen.jsx
            ├── DashboardScreen.jsx
            ├── AlertsScreen.jsx
            └── AdminScreen.jsx

Backend (Node.js)
- Entry point: `idx.js`
- Connects to MongoDB Atlas using the connection string from `.env`
- Issues and verifies JWTs, enforces role-based access on routes
- Sends WhatsApp alerts through Twilio using three message templates 
- Runs a Socket.io server for live updates
- Talks to the Flask ML service for face matching

 API endpoints
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/auth/register` | Create account (role + Email) |
| POST | `/auth/login` | Returns JWT |
| GET | `/cases` | List cases |
| POST | `/cases` | Register a missing person |
| POST | `/match` | Submit a photo for matching |

ML Service (Flask)
- match.py receives a photo, compares it with stored missing-person faces, and returns a confidence score
- DeepFace: face embedding and comparison
-YOLOv8: detection 
Frontend (React)
-React with `react-scripts` 5.0.1
- Leaflet.js + OpenStreetMap for maps. Google Maps was dropped to avoid API cost and access problems.
- Nominatim for geocoding case locations, so the map centers on the case instead of defaulting to a fixed city
- User session kept in localStorage
- Role decides which screens a user sees



Roles and Authentication

| Role | Can do |
|------|--------|
| Public | Register, report sightings, view public case info |
| Officer | Manage assigned cases, review matches, trigger alerts |
| Admin | Everything above, plus manage users and the database |

Flow:
Register (name, email, role)
        │
        ▼
Login → backend verifies credentials → returns JWT
        │
        ▼
JWT stored in localStorage → sent with every API request
        │
        ▼
Backend checks role on each protected route

How Matching Works
Photo uploaded (sighting or new case)
        │
        ▼
Face detected and cropped
        │
        ▼
DeepFace generates face embedding
        │
        ▼
Compared against stored missing-person embeddings
        │
        ▼
match.py returns confidence score per candidate
        │
        ├── high confidence → match saved + WhatsApp alert sent
        └── low confidence  → no alert, can still be reviewed manually
Getting Started
Prerequisites

- Node.js 18+
- Python 3.10+
- A MongoDB Atlas cluster (must be active, not paused, with your IP allowed under Network Access)
- A Twilio account with the WhatsApp sandbox or an approved sender

1. Clone

bash
git clone https://github.com/<your-username>/IMPSAS.git
cd IMPSAS
2. Backend
bash
cd backend
npm install
# create .env (see Environment Variables)
node idx.js
3. ML service
bash
cd ml
python -m venv venv
# Windows
venv\Scripts\activate
# macOS / Linux
source venv/bin/activate

pip install -r requirements.txt
python match.py
 4. Frontend
bash
cd frontend
npm install
npm start
Environment Variables
Create a `.env` file in `backend/`. Format is `KEY=VALUE` with an equals sign, no colons, and no hardcoded credentials in code.

MONGODB_URI=your-mongodb-atlas-connection-string
JWT_SECRET=your-secret
TWILIO_ACCOUNT_SID=your-twilio-sid
TWILIO_AUTH_TOKEN=your-twilio-token

| Variable | Description |
|----------|-------------|
| `MONGODB_URI` | Atlas connection string |
| `JWT_SECRET` | Secret used to sign tokens |
| `TWILIO_ACCOUNT_SID` | Twilio account ID |
| `TWILIO_AUTH_TOKEN` | Twilio auth token |


 Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React, Leaflet.js, OpenStreetMap |
| Backend | Node.js, Socket.io, JWT |
| Database | MongoDB Atlas |
| ML | DeepFace, YOLOv8, Whisper, Flask |
| Notifications | Twilio WhatsApp API |
| Auth | JWT, role-based access control |

Known Issues and Roadmap
Known issues

- MongoDB Atlas connection can fail after a paused cluster resumes. A timeout config was added in `idx.js`, but the root fix is still open. Check that the cluster is running and your IP is allowlisted.
Planned
- Social media auto-posting for new cases
- Live heatmaps of sightings
- Sponsorship and outreach material

License
For educational and academic purposes. 
Team and Contributors

Group SEDA_06, VIT Pune. Guide: Mrs. Swati Atul Joshi.

Member	Role	Github	Linkedln
Prajkta Patil	Lead Developer · Full Stack (React, Node.js, MongoDB, Twilio, auth)	
https://github.com/PrajktaSPatil
	https://www.linkedin.com/in/prajkta-patil-7233a5262/
Srushti Patel	AI/ML Engineer (face recognition and matching pipeline)		https://www.linkedin.com/in/srushti-patel-b03585319/
Sanchita Mote	Backend Engineer (API routes and database)		https://www.linkedin.com/in/sanchita-mote16/
Samarth Patil	Documentation and Testing Lead (research paper, testing, evaluation)		https://www.linkedin.com/in/samarth-patil-26a7aa31b/

