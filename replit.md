# Azam Max AI Hub - Next-Gen Entertainment Platform

## Overview
AI-powered video analysis platform for Azam Media (Bakhresa Group). Supports 3 genres: Football (Mpira), Film (Filam), Boxing (Ngumi). Features AI scene detection with timestamps, Swahili subtitle generation, and an interactive AI narrator that users can query about video content. Branded with Azam Blue (#003366) and Azam Orange (#FF8C00).

## Architecture
- **Frontend**: React + TypeScript + Vite + TailwindCSS + shadcn/ui + Framer Motion
- **Backend**: Express.js + TypeScript
- **Database**: PostgreSQL (via Drizzle ORM)
- **AI**: OpenAI via Replit AI Integrations (gpt-4o-mini-transcribe for STT, gpt-4o/gpt-4o-mini for vision/translation/chat), Google Gemini TTS (gemini-2.5-flash-preview-tts, Kore voice)
- **Media**: FFmpeg for frame extraction and audio processing, yt-dlp for YouTube

## Key Features
1. **AI Video Analysis** - Upload video with genre → AI extracts frames every 8s → vision model detects genre-specific events
   - Football: goals, shots, fouls, celebrations, corners, saves, penalties
   - Film: action, emotion, dialogue, dramatic, fight, chase
   - Boxing: KO, knockdown, punch, round start/end, celebration
2. **Swahili Subtitle Generation** - Audio transcription + 3-way translation:
   - Kiswahili Sanifu (Standard)
   - Kiswahili cha Mtaani (Street/Tanzania slang)
   - English
3. **AI Narrator/Chat with Voice** - Interactive chat where users ask about video content:
   - "Nionyeshe goli la kwanza" → AI responds with timestamp + narration
   - "Nini kinatokea dakika ya 5?" → AI describes the scene
   - Clicking timestamp in response seeks video to that point
   - Responds in chosen language style (Sanifu/Mtaani/English)
   - **Text-to-Speech**: AI reads response aloud using genre-specific voices:
     - Football: onyx (deep male commentator)
     - Film: nova (expressive narrator)
     - Boxing: echo (energetic commentator)
   - Voice toggle button to enable/disable, stop button during playback
4. **Scene Timeline** - Clickable scene cards with icons, timestamps, descriptions
5. **Live Subtitles** - Overlay on video player synced to playback position
6. **SRT/VTT Export** - Download subtitles in standard formats
7. **Dark/Light Theme** - Toggle with Azam brand colors

## App Layout
- **Header**: Azam Max AI Hub branding with theme toggle
- **Left Sidebar**: 3 genre sections (Mpira/Filam/Ngumi) each with own "Pakia" upload button + video list per genre
- **Center**: Video player with subtitle overlay, analyze button, AI insights card, progress
- **Right Panel**: Tabs for Matukio (Scenes), Manukuu (Subtitles), AI Chat

## Project Structure
```
shared/schema.ts             - Database schema (videos, subtitles, scenes, videoAnalysis, conversations, messages)
server/db.ts                 - Database connection
server/storage.ts            - Data access layer with full CRUD
server/routes.ts             - API routes (upload, analyze SSE, scenes, chat, export)
server/video-analysis-service.ts - Frame extraction, AI vision analysis, transcription + translation
server/subtitle-service.ts   - Audio extraction, getVideoDuration, SRT/VTT formatting
server/youtube-service.ts    - YouTube URL parsing, yt-dlp audio download
server/replit_integrations/  - OpenAI integration code
client/src/App.tsx           - Main app with routing and providers
client/src/pages/home.tsx    - Main page: 3-column layout with upload, player, panels
client/src/components/
  ai-narrator.tsx            - AI chat component with quick prompts, timestamp links
  scene-timeline.tsx         - Clickable scene cards with genre-specific icons
  theme-provider.tsx         - Dark/light theme context
uploads/                     - Uploaded video files (created at runtime)
```

## Database Schema
- `videos` - id, title, filename, originalName, genre, duration, status, createdAt
- `subtitles` - id, videoId, startTime, endTime, originalText, swahiliStandard, swahiliStreet, englishTranslation
- `scenes` - id, videoId, startTime, endTime, sceneType, description, descriptionSwahili, confidence, metadata
- `video_analysis` - id, videoId, summary, summarySwahili, analysisData (JSON), status
- `conversations` - id, videoId, title, createdAt
- `messages` - id, conversationId, role, content, createdAt

## API Endpoints
- `GET /api/videos` - List all videos
- `GET /api/videos/:id` - Get single video
- `POST /api/videos/upload` - Upload video with genre (multipart/form-data)
- `POST /api/videos/:id/analyze` - Full AI analysis (SSE stream: scenes + subtitles + summary)
- `GET /api/videos/:id/scenes` - Get detected scenes
- `GET /api/videos/:id/subtitles` - Get subtitles
- `GET /api/videos/:id/analysis` - Get analysis summary
- `POST /api/videos/:id/chat` - AI narrator chat (returns reply + clickable timestamps)
- `POST /api/tts` - Text-to-speech (body: text, genre → returns audio/mpeg)
- `GET /api/videos/:id/export/:format?style=` - Export subtitles as SRT/VTT
- `DELETE /api/videos/:id` - Delete video and all related data

## Technical Notes
- Transcription: gpt-4o-mini-transcribe (NOT whisper-1, NOT verbose_json format)
- Vision: gpt-4o with base64 frames, detail: "low", batch 4 frames at a time (gpt-5.2 broken — use gpt-4o everywhere)
- Video streaming: proper Range request support (206 Partial Content) for reliable pause/seek
- Frame extraction: every 15 seconds via ffmpeg
- Audio chunking: 8-second segments for transcription with volume boost (3.5x + compressor)
- Subtitle timing: character-proportional with 300ms delay for sync
- Gap filling: silent sections get "[♪ Muziki / Sauti ♪]" placeholder subtitles
- TTS: Gemini gemini-2.5-flash-preview-tts with Kore voice via @google/genai (GEMINI_API_KEY)
- Chat: video context (scenes + subtitles + analysis) injected into system prompt
- SSE routes use res.on("close") not req.on("close") for disconnect detection
- yt-dlp installed via pip (python3 -m yt_dlp) — Nix version is outdated
- Colors: Azam Blue (#003366), Orange (#FF8C00)

## Environment
- Replit AI Integrations (AI_INTEGRATIONS_OPENAI_BASE_URL, AI_INTEGRATIONS_OPENAI_API_KEY)
- DATABASE_URL for PostgreSQL
- FFmpeg available in system PATH
