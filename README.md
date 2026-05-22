# Clinical-Voice-A
# 2Care.ai: Multilingual Real-Time Voice Appointment Agent

2Care.ai is an intelligent, real-time, multilingual voice assistant tailored for clinical appointment booking. Operating via a low-latency pipeline, the agent allows patients to seamlessly schedule, cancel, view, or reschedule appointments using voice commands in English, Hindi, and Tamil.

---

## 🚀 Setup Instructions

### Prerequisites
* **Node.js** (v18+ recommended)
* **Supabase CLI** (for local edge function development)
* **Google Cloud Project** with Gemini API access enabled
* **Google Chrome Browser** (required for native Web Speech API support)

### Installation

1. **Clone the repository and install dependencies:**
   ```bash
   npm install

2. Configure Environment Variables:
  Create a .env file in your root/backend directory:
Code snippet
SUPABASE_URL=your_supabase_url
SUPABASE_ANON_KEY=your_supabase_anon_key
GEMINI_API_KEY=your_gemini_api_key
3. Database Setup & Migrations:
Apply the database schema and seed data via Supabase to initialize the doctors, appointments, and memory tables:

Bash
supabase db pus
4. Run the Application Locally:

Bash
npm run dev

Open http://localhost:5173 in Google Chrome, click the microphone icon, and grant mic permissions to start testing.

Architecture Explanation
The system uses an event-driven, streaming pipeline optimized to bypass complex server architectures by combining native browser APIs with robust edge compute functions.

[Patient Mic] ──(Web Speech API)──> [STT Text & Lang Detect] 
                                            │
                                     (JSON Payload)
                                            ▼
[Supabase Edge Function] ◄───► [Gemini 2.5 Flash Agent] ◄───► [PostgreSQL DB]
                                            │                      (Schema / Tools)
                                   (Text Response)
                                            ▼
[Patient UI] ◄──────(Web Speech Synthesis)──┘
Frontend Capture & STT: The client-side browser captures microphone input via the Web Speech API for rapid, zero-cost Speech-to-Text conversion.

Language Detection Layer: A hybrid script-based analyzer combines with LLM parsing to instantly lock onto the speaker's language context (en-IN, hi-IN, ta-IN).

Orchestration Edge Function: The text is securely forwarded to a Supabase Edge Function that manages the session state and communicates directly with the LLM.

Brain (Gemini 2.5 Flash): Acts as the central reasoning engine. Based on the user's intent, it fires native Tool Calls to modify or read the PostgreSQL database.

Text-to-Speech (TTS): The agent's generated response is channeled back to the frontend, utilizing the browser's native SpeechSynthesis API to vocalize the reply instantly.

🧠 Memory Design
To behave like a human receptionist, the system operates on a dual-layer memory paradigm:

1. Short-Term Memory (voice_sessions)
Purpose: Tracks the active conversational state during a single call session.

Mechanism: Temporarily holds the chat history array, slots currently discussed, and transient language flags. This allows the user to say context-dependent phrases like "No, the other one" or "Change that to 12 instead".

2. Long-Term Memory (patient_memory)
Purpose: Permanently persists user identifiers, historical configurations, and personal traits.

Mechanism: Stores user identity details (patient_id), language preferences, preferred doctors, or scheduling constraints. If a patient routinely books with a specific cardiologist, the edge function injects this preference into the system prompt on subsequent calls (remember_preference tool).

⏱️ Latency Breakdown
The architecture targets an ultra-responsive threshold of <450 ms per conversational turn. The frontend monitors performance live via the /metrics panel across several milestones:

Pipeline Stage	Technology Used	Target Latency	Performance Impact
STT & Lang Detect	Web Speech API	~50–100 ms	Local execution avoids massive network audio payload uploads.
LLM Execution & Tools	Gemini 2.5 Flash	~200–350 ms	Flash is selected explicitly for high token processing speeds and rapid function invocation.
Database Operations	PostgreSQL Indexing	~10–30 ms	Unique-slot constraints and validation triggers verify schedules rapidly.
TTS Generation	Browser SpeechSynthesis	~20–50 ms	Zero-latency text chunk parsing avoids cloud audio synthesis bottlenecks.
Total Target	End-to-End Pipeline	<450 ms	Highlighted green in live telemetry metrics dashboard.
⚖️ Trade-offs
Web Speech API vs. Dedicated Cloud Whisper/Deepgram:

Trade-off: We use native browser engines instead of server-side audio streams.

Pros: Virtually zero latency for STT, no heavy bandwidth overhead sending raw audio files, and complete cost savings on audio APIs.

Cons: Transcription reliability is highly dependent on the user's browser, client hardware quality, and specific OS configurations.

Gemini 2.5 Flash vs. Gemini 2.5 Pro:

Trade-off: We prioritized Flash for core reasoning.

Pros: Significantly faster execution speeds and highly cost-effective execution boundaries perfect for low-latency voice turns.

Cons: Marginally lower accuracy on ultra-complex multi-intent reasoning streams compared to Pro models.

⚠️ Known Limitations
Browser Restrictions: Native Web Speech API functionality performs optimally in Google Chrome. Performance and multilingual support may degrade or fall back unpredictably on Safari, Firefox, or Brave.

Ambient Noise & Overlapping Speech: Lacking a dedicated custom server-side voice activity detector (VAD), background noises or pauses can occasionally cause early speech cut-offs or skipped phrases.

Database Race Conditions: While protected by unique slot constraint indexes, concurrent users hitting the identical doctor's slot at the precise millisecond depend on standard database lock resolution handling.
