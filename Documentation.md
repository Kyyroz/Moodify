# Software Requirements Specification (SRS)
## Lo-Fi AI Mood-Based Music Recommendation Web Application

---

## 1. Introduction

### 1.1 Purpose
This document provides a complete Software Requirements Specification (SRS) for a Lo-Fi AI Mood-Based Music Recommendation Web Application. It defines the system architecture, functional and non-functional requirements, database schemas, user stories, and data flow required to build a production-ready application using the **MERN** stack (MongoDB, Express.js, React.js, Node.js) structured under the **MVC (Model-View-Controller)** design pattern.

### 1.2 System Overview
The application upgrades a base Lo-Fi aesthetic webpage into an interactive, AI-driven music discovery dashboard. Users authenticate via **OAuth**, interact with a customized dashboard, and receive hyper-personalized track recommendations based on their current mood. 

The mood engine operates via two primary inputs:
1. **Graphical UI (GUI):** Clickable mood cards and visual environment toggles (e.g., *Rainy Cafe, Late Night Study, Melancholy*).
2. **Natural Language Processing (NLP):** A descriptive text input where users describe their situation (e.g., *"I'm cramming for an exam and feeling stressed, play something calming with rain sounds"*). An integrated AI LLM parses this text into structured acoustic attributes and mood tags to query a public music API (such as the Spotify API or Jamendo API) and a curated MongoDB track cache.

---

## 2. System Architecture (MVC in MERN)

The backend adheres strictly to the **Model-View-Controller (MVC)** architectural pattern, separating data representation, business logic, and client-side rendering.

```text
+-----------------------------------------------------------------------+
|                             VIEW (React)                              |
|  +-------------------+  +--------------------+  +------------------+  |
|  |   Lo-Fi UI Base   |  | Interactive Moods  |  |  User Dashboard  |  |
|  +-------------------+  +--------------------+  +------------------+  |
+-----------------------------------^-----------------------------------+
                                    | (REST API / JSON / WebSockets)
+-----------------------------------v-----------------------------------+
|                         CONTROLLER (Express / Node)                   |
|  +-------------------+  +--------------------+  +------------------+  |
|  |  Auth Controller  |  |  Mood / AI Engine  |  |  Music API Sync  |  |
|  |     (OAuth 2.0)   |  |   (LLM Parser)     |  | (Spotify/Jamendo)|  |
|  +-------------------+  +--------------------+  +------------------+  |
+-----------------------------------^-----------------------------------+
                                    | (Mongoose ODM)
+-----------------------------------v-----------------------------------+
|                            MODEL (MongoDB)                            |
|  +-------------------+  +--------------------+  +------------------+  |
|  |    Users Schema   |  | Tracks & Metadata  |  | Mood Logs &      |  |
|  |                   |  |      Schema        |  | Playlists Schema |  |
|  +-------------------+  +--------------------+  +------------------+  |
+-----------------------------------------------------------------------+
```

### 2.1 Component Mapping
* **Model (MongoDB / Mongoose):** Defines data structures, schema validations, and database interactions for users, cached tracks, mood tags, listening history, and saved playlists.
* **View (React.js):** The client-side Single Page Application (SPA). It renders the Lo-Fi aesthetic UI, handles OAuth redirects, provides the mood selection interfaces (clicks & text), displays the user dashboard, and embeds the audio player.
* **Controller (Express.js / Node.js):** Acts as the brain of the application. It intercepts HTTP requests from the React View, executes business logic (such as sending user prompts to an AI API for tag extraction, querying public music APIs, and managing session state), and updates/reads from the MongoDB Models before returning JSON responses to the View.

---

## 3. Functional Requirements

### 3.1 Authentication & User Management
* **FR-1.1 (OAuth Integration):** The system must allow users to register and log in using OAuth 2.0 providers (Google and Discord) via Passport.js or Auth0.
* **FR-1.2 (Session Management):** Upon successful OAuth authentication, the backend must issue a secure, HTTP-only JSON Web Token (JWT) to maintain user sessions across routes.
* **FR-1.3 (Guest Mode):** Unauthenticated users may access the base Lo-Fi page and simple graphical mood clicks, but must be prompted to log in via OAuth to access the dashboard, descriptive AI recommendations, and playlist saving features.

### 3.2 Mood Determination & AI Recommendation Engine
* **FR-2.1 (Graphical UI Mood Selection):** The UI must present interactive graphical elements (e.g., icons for *Chill, Focus, Sleep, Sad, Energetic* and ambient toggles for *Rain, Vinyl Crackle, Coffee Shop Noise*). Clicking these elements immediately maps to hardcoded tag arrays.
* **FR-2.2 (Natural Language Mood Input):** The UI must provide a text area where users can input unstructured descriptions of their current state or desires.
* **FR-2.3 (AI Prompt Engineering & Parsing):** When a text description is submitted, the Controller must pass the prompt to an LLM API (e.g., OpenAI or Google Gemini) with a strict system instruction to return a JSON object containing:
  * Primary and secondary mood tags (e.g., `["calming", "ambient", "rain", "focus"]`).
  * Target acoustic attributes (e.g., `energy: 0.2`, `valence: 0.4`, `tempo: 60-80 BPM`).
* **FR-2.4 (Hybrid Track Retrieval):** The Controller must take the AI-generated tags/attributes and query the public music API (and local MongoDB cache) to return a curated playlist of 10–15 matching tracks.

### 3.3 Dashboard & Playlist Management
* **FR-3.1 (User Dashboard):** The dashboard must display user profile information retrieved via OAuth, recently played tracks, listening time statistics, and a history of past AI mood prompts.
* **FR-3.2 (Playlist Persistence):** Users must be able to save AI-generated tracklists as named playlists in their dashboard.
* **FR-3.3 (Track Feedback):** Users must be able to "Like" or "Dislike" tracks. The Controller must save this feedback to the User Model to weight future AI tag generations.

### 3.4 Audio Player & Public API Integration
* **FR-4.1 (Music API Sync):** The system must integrate with a public music API (e.g., Spotify Web API or Jamendo API) to fetch track metadata, album artwork, artist details, and playable audio streams/embeds.
* **FR-4.2 (Persistent Lo-Fi Player):** The React View must feature a persistent audio player bar that continues playing across route transitions (e.g., moving from the home page to the dashboard).
* **FR-4.3 (Ambient Sound Mixer):** The player must include an independent audio layer allowing users to overlay looping ambient sounds (rain, thunder, waves, white noise) over the streaming music.

---

## 4. Non-Functional Requirements

| Requirement Type | Specification |
| :--- | :--- |
| **Performance** | AI text parsing and track recommendation generation must complete and return a response to the View within **2.5 seconds**. API responses for static dashboard queries must take `< 300ms`. |
| **Scalability** | The backend Controller must implement rate-limiting (e.g., via `express-rate-limit`) on AI recommendation endpoints to prevent API quota exhaustion and DDoS attacks. |
| **Security** | All OAuth tokens, API keys (LLM & Music API), and database URIs must be stored securely in environment variables (`.env`). JWTs must use strict expiration times and `Secure/SameSite` cookie flags. |
| **Usability** | The interface must maintain a responsive, low-latency, aesthetic Lo-Fi theme (dark/warm color palettes, smooth CSS transitions) across desktop, tablet, and mobile devices. |
| **Reliability** | If the external AI API or Music API experiences downtime, the system must gracefully degrade by serving fallback Lo-Fi playlists cached directly in MongoDB. |

---

## 5. Database Architecture & Schema Design

The MongoDB database is designed from scratch to support custom tagging, API metadata caching, and user history tracking.

### 5.1 `User` Schema
Stores OAuth profile data, user preferences, and interaction history.

```javascript
const mongoose = require('mongoose');

const UserSchema = new mongoose.Schema({
  oauthProvider: { type: String, required: true, enum: ['google', 'discord', 'spotify'] },
  oauthId: { type: String, required: true, unique: true },
  displayName: { type: String, required: true },
  email: { type: String, required: true, unique: true },
  avatarUrl: { type: String },
  preferences: {
    defaultAmbientSound: { type: String, default: 'none' },
    preferredBpmRange: {
      min: { type: Number, default: 60 },
      max: { type: Number, default: 90 }
    }
  },
  likedTracks: [{ type: mongoose.Schema.Types.ObjectId, ref: 'Track' }],
  dislikedTracks: [{ type: mongoose.Schema.Types.ObjectId, ref: 'Track' }],
  listeningHistory: [{
    track: { type: mongoose.Schema.Types.ObjectId, ref: 'Track' },
    playedAt: { type: Date, default: Date.now },
    contextMood: { type: String }
  }]
}, { timestamps: true });

module.exports = mongoose.model('User', UserSchema);
```

### 5.2 `Track` Schema (Metadata Cache & Tagging Structure)
Caches tracks fetched from the public music API to reduce latency and API calls, enriching them with a proprietary mood-tagging structure.

```javascript
const TrackSchema = new mongoose.Schema({
  externalApiId: { type: String, required: true, unique: true }, // e.g., Spotify Track ID
  apiSource: { type: String, default: 'spotify' },
  title: { type: String, required: true, index: true },
  artist: { type: String, required: true },
  album: { type: String },
  artworkUrl: { type: String },
  streamUrl: { type: String, required: true },
  durationMs: { type: Number, required: true },
  
  // Custom Tagging & Acoustic Structure
  acousticAttributes: {
    bpm: { type: Number },
    energy: { type: Number, min: 0, max: 1 },       // 0.0 (calm) to 1.0 (intense)
    valence: { type: Number, min: 0, max: 1 },      // 0.0 (sad/melancholic) to 1.0 (happy/uplifting)
    instrumentalness: { type: Number, min: 0, max: 1 },
    acousticness: { type: Number, min: 0, max: 1 }
  },
  moodTags: [{ 
    type: String, 
    index: true, 
    lowercase: true 
    // Examples: 'rain', 'study', 'sleep', 'late-night', 'nostalgic', 'piano', 'chillhop'
  }],
  cachedAt: { type: Date, default: Date.now, expires: 604800 } // 7-day TTL cache renewal
}, { timestamps: true });

module.exports = mongoose.model('Track', TrackSchema);
```

### 5.3 `Playlist` Schema
Allows users to save playlists generated by the AI or created manually.

```javascript
const PlaylistSchema = new mongoose.Schema({
  userId: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true, index: true },
  title: { type: String, required: true, default: 'My Lo-Fi Vibe' },
  description: { type: String },
  generatedByAi: { type: Boolean, default: false },
  originalPrompt: { type: String }, // e.g., "Cramming for an exam with rain sounds"
  tracks: [{ type: mongoose.Schema.Types.ObjectId, ref: 'Track' }],
  isPublic: { type: Boolean, default: false },
  coverImage: { type: String }
}, { timestamps: true });

module.exports = mongoose.model('Playlist', PlaylistSchema);
```

### 5.4 `MoodLog` Schema
Tracks AI recommendations to refine future suggestions and display mood analytics on the user dashboard.

```javascript
const MoodLogSchema = new mongoose.Schema({
  userId: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
  inputType: { type: String, enum: ['gui_click', 'text_description'], required: true },
  rawInput: { type: String, required: true }, // Button name or raw text prompt
  parsedTags: [{ type: String }],             // Tags extracted by the AI LLM
  recommendedTracks: [{ type: mongoose.Schema.Types.ObjectId, ref: 'Track' }],
  userFeedbackScore: { type: Number, min: -1, max: 1, default: 0 } // -1 (poor), 0 (neutral), 1 (great)
}, { timestamps: true });

module.exports = mongoose.model('MoodLog', MoodLogSchema);
```

---

## 6. User Stories

### 6.1 Authentication & Profile
* **US-1:** *As a new visitor*, I want to sign in using my Google or Discord account via OAuth, so that I don't have to create and remember a new password.
* **US-2:** *As a returning user*, I want my session to be remembered securely, so that I am automatically taken to my custom dashboard when I visit the site.

### 6.2 Music Discovery & Mood Engine
* **US-3:** *As a casual listener*, I want to click simple graphical icons (like a cloud or coffee cup), so that I can instantly play music matching that vibe without typing anything.
* **US-4:** *As a stressed student*, I want to type *"I'm cramming for an exam and feeling stressed, play something calming with rain sounds"*, so that the AI understands my complex emotional state and customizes a soundtrack specifically for focus and stress relief.
* **US-5:** *As a listener*, I want to toggle background rain or vinyl crackle sounds over the music, so that I can create a deeper ambient immersion.

### 6.3 Dashboard & Interaction
* **US-6:** *As an authenticated user*, I want to view my recent listening history and most frequent moods on my dashboard, so that I can track my study/relaxation habits.
* **US-7:** *As a user who received a perfect AI recommendation*, I want to click a "Save Playlist" button, so that I can easily listen to that exact sequence of tracks again later from my dashboard.

---

## 7. Data Flow Diagrams (DFD)

### 7.1 High-Level System Data Flow (Level 0 DFD)

```text
 [ User / Browser ]
    |         ^
    |         |
    | (1)     | (6)
    | Input   | Return Playlist
    | Mood    | & Stream Audio
    v         |
+-------------------------------------------------------+
|              MERN APPLICATION (MVC Core)              |
|                                                       |
|  [ View: React UI ] <---> [ Controller: Express ]     |
+-------------------------------------------------------+
    |         ^                       |          ^
    | (2)     | (3)                   | (4)      | (5)
    | Send    | Return                | Query    | Return
    | Prompt  | JSON Tags             | Tracks   | Metadata & Audio
    v         |                       v          |
+-----------------------+     +-------------------------+
|     External AI       |     |  Public Music API &     |
|   (LLM / NLP API)     |     |  MongoDB Model Cache    |
+-----------------------+     +-------------------------+
```

### 7.2 Step-by-Step Execution Flow (Natural Language Prompt)

1. **User Input (View):** The user enters `"I'm cramming for an exam and feeling stressed, play something calming with rain sounds"` into the React frontend input bar and submits.
2. **API Request (View -> Controller):** The React View sends a secure `POST /api/mood/recommend` request containing the text string and the user's JWT authorization header to the Express Controller.
3. **AI Parsing (Controller -> LLM API):** The `MoodController` formats the prompt with system instructions and calls the LLM API.
   * *LLM Output Received:* `{ "tags": ["study", "calm", "rain", "lofi"], "attributes": { "energy": 0.2, "valence": 0.4 }, "ambient": "rain" }`
4. **Cache Check & Query (Controller -> Model):** The Controller queries the MongoDB `Track` Model for cached songs matching `"study"` and `"calm"` with energy <= 0.3.
5. **External Fallback/Enrichment (Controller -> Music API):** If MongoDB has fewer than 10 matches, the Controller queries the public Music API (e.g., Spotify API search endpoint with genre/tag filters), retrieves new tracks, and asynchronously caches them into the MongoDB `Track` collection.
6. **Logging (Controller -> Model):** The Controller creates a new document in the `MoodLog` collection linking the user, the prompt, the extracted tags, and the selected track IDs.
7. **Response & Playback (Controller -> View):** The Controller sends a JSON response back to the React View containing the track metadata array and an instruction to auto-enable the `"rain"` ambient audio loop. The React audio player begins playback.