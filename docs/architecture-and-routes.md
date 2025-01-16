# Slack Clone: System Architecture

Below is an expanded overview of our Slack clone's system architecture. It incorporates **Vite/React** on the frontend, **Supabase** (PostgreSQL, Auth, Storage, Edge Functions) for core services, and **pgvector** for AI-based features. The goal is to provide an **exhaustive** reference that explains each segment in clear detail.

---

## Table of Contents
- [Slack Clone: System Architecture](#slack-clone-system-architecture)
  - [Table of Contents](#table-of-contents)
  - [1. Overview of Architecture Layers](#1-overview-of-architecture-layers)
  - [2. API Routes](#2-api-routes)
    - [2.1 Authentication Routes](#21-authentication-routes)
    - [2.2 User Profile & Settings](#22-user-profile--settings)
    - [2.3 Channels](#23-channels)
    - [2.4 Direct Messages](#24-direct-messages)
    - [2.5 Messages](#25-messages)
    - [2.6 File Upload & Download](#26-file-upload--download)
    - [2.7 AI (RAG) Endpoints](#27-ai-rag-endpoints)
  - [3. Page Structure & React Components](#3-page-structure--react-components)
  - [4. Key Middleware Functions & Explanation](#4-key-middleware-functions--explanation)

---

## 1. Overview of Architecture Layers

1. **Frontend Layer (Vite/React + Tailwind + ShadcN)**  
   - Renders UI components for channels, DMs, settings, and the AI chatbot page.  
   - Uses **React Router** to define routes such as `/`, `/channel/:id`, `/dm/:id`, `/settings`, `/jonathan`, etc.  
   - Interacts with Supabase SDK for Auth, real-time data, and Storage (file uploads).

2. **Backend Layer (Supabase Edge Functions)**  
   - **Edge Functions**:  
     - TypeScript-based serverless functions for AI processing and complex operations.
     - Handles RAG queries using pgvector for similarity search.
     - Processes AI responses and stores them in PostgreSQL.

3. **Storage & Database Layer**  
   - **PostgreSQL**:  
     - Primary data store with tables for users, channels, messages, DMs, membership, and AI logs.
     - Real-time updates via NOTIFY/LISTEN and Supabase subscriptions.
     - Vector similarity search using pgvector extension.
   - **Supabase Auth**:  
     - Manages user identity, including sign-up, email verification, and login.
     - JWT tokens for secure API calls and RLS policies.
   - **Supabase Storage**:  
     - Stores user-uploaded files (images, documents, videos, audio) up to 10MB.
     - RLS policies ensure only channel members/DM participants can upload/download.

---

## 2. API Routes

This section expands on the key endpoints and clarifies how each route works, including authentication requirements and relevant parameters.

### 2.1 Authentication Routes

1. **`POST /auth/signup`**  
   - **Auth**: None (public).  
   - **Purpose**: Creates a new Supabase Auth user, sets `email_verified = false` until verification.  
   - **Data**: `{ username, email, password }`.  
   - **Behavior**:  
     - Inserts basic info into users table after Supabase Auth user is created.  
     - Returns a success response prompting email verification.  

2. **`POST /auth/login`**  
   - **Auth**: None (public).  
   - **Purpose**: Authenticates with Supabase Auth (via email/password).  
   - **Data**: `{ email, password }`.  
   - **Behavior**:  
     - Returns success if credentials are valid.
     - Provides JWT token for subsequent requests.

---

### 2.2 User Profile & Settings

1. **`GET /users/{id}`**  
   - **Auth**: Supabase Auth JWT required.  
   - **Purpose**: Fetches user's profile data.  
   - **Rules**:  
     - RLS policy ensures users can only access their own full details.
     - Public fields (display_name, photo_url) readable by all.

2. **`PATCH /users/{id}`**  
   - **Auth**: Supabase Auth JWT required.  
   - **Purpose**: Updates display name, status preferences, AI settings (`ai_responder_mode`, `ai_context`).  
   - **Rules**: RLS policy ensures users can only update their own data.

---

### 2.3 Channels

1. **`POST /channels`**  
   - **Auth**: Supabase Auth required.  
   - **Purpose**: Creates a new channel (public or private).  
   - **Data**: `{ name, type: "public"|"private" }`.  
   - **Rules**:  
     - `created_by` is set to the user's ID.
     - RLS policy enforces creation permissions.

2. **`GET /channels`**  
   - **Auth**: Supabase Auth required.  
   - **Purpose**: Returns public channels plus private channels where user is a member.  
   - **Rules**: RLS policy filters private channels automatically.

3. **`GET /channels/{channelId}`**  
   - **Auth**: Supabase Auth required.  
   - **Purpose**: Retrieves channel info.  
   - **Rules**:  
     - Public channels readable by all.
     - Private channels require membership (enforced by RLS).

4. **`DELETE /channels/{channelId}`**  
   - **Auth**: Supabase Auth required.  
   - **Purpose**: Soft deletes if `created_by` matches authenticated user.  
   - **Rules**: RLS policy ensures only creator can delete.

5. **`POST /channels/{channelId}/join`**  
   - **Auth**: Supabase Auth required.  
   - **Purpose**: Adds membership record for public channels or processes invite for private.

6. **`POST /channels/{channelId}/leave`**  
   - **Auth**: Supabase Auth required.  
   - **Purpose**: Removes user membership from channel.

---

### 2.4 Direct Messages

1. **`GET /dms/{dmId}`**  
   - **Auth**: Supabase Auth required.  
   - **Purpose**: Fetches DM metadata (participants, last_message, updated_at).  
   - **Rules**: RLS ensures only participants can view.

2. **`POST /dms`**  
   - **Auth**: Supabase Auth required.  
   - **Data**: `{ recipient_id }`.  
   - **Purpose**: Creates or retrieves existing DM conversation.
   - **Rules**: RLS ensures valid participant access.

---

### 2.5 Messages

1. **`POST /messages`**  
   - **Auth**: Supabase Auth required.  
   - **Data**: `{ channel_id or dm_id, content, attachments, thread_root_id }`.  
   - **Purpose**: Creates message in specified channel or DM.  
   - **Rules**:  
     - RLS policies enforce channel membership or DM participation.
     - Triggers update last_message and notify subscribers.

2. **`GET /messages?channel_id=...&limit=...&start_after=...`**  
   - **Auth**: Supabase Auth required.  
   - **Purpose**: Retrieves up to 50 messages with pagination.
   - **Rules**: RLS automatically filters based on access rights.

3. **`PATCH /messages/{messageId}`**  
   - **Auth**: Supabase Auth required.  
   - **Purpose**: Edits message text or attachments.  
   - **Rules**: RLS ensures only sender can edit.

4. **`POST /messages/{messageId}/reactions`**  
   - **Auth**: Supabase Auth required.  
   - **Data**: `{ emoji }`.  
   - **Purpose**: Creates/updates reaction count.
   - **Rules**: RLS inherits message access permissions.

5. **`DELETE /messages/{messageId}/reactions/{reactionId}`**  
   - **Auth**: Supabase Auth required.  
   - **Purpose**: Removes reaction if user added it.
   - **Rules**: RLS ensures user can only remove own reactions.

---

### 2.6 File Upload & Download

1. **`POST /storage/upload`**  
   - **Auth**: Supabase Auth required.  
   - **Data**: Binary file payload, up to 10MB.  
   - **Purpose**: Stores file in Supabase Storage and returns metadata.  
   - **Rules**:  
     - RLS checks channel membership or DM participation.
     - Size limit enforced by Storage configuration.

2. **`GET /storage/objects/{bucket}/{path}`**  
   - **Auth**: Supabase Auth required.  
   - **Purpose**: Downloads file from Supabase Storage.  
   - **Rules**: RLS ensures user has access rights.

---

### 2.7 AI (RAG) Endpoints

1. **`POST /functions/jonathan`**  
   - **Auth**: Supabase Auth required.  
   - **Data**: `{ query }`.  
   - **Purpose**: Edge Function that:
     - Generates embeddings using OpenAI.
     - Performs similarity search with pgvector.
     - Returns AI response stored as message.

2. **`POST /functions/autorespond`**  
   - **Auth**: Supabase Auth required.  
   - **Data**: `{ mention_data, channel_id or dm_id, user_id }`.  
   - **Purpose**: Edge Function that:
     - Checks user's `ai_responder_mode`.
     - Retrieves context via pgvector.
     - Posts AI response if conditions met.

---

## 3. Page Structure & React Components

The frontend uses **React Router** to define pages. Each page is composed of modular components.

1. **`App.tsx` (Root Layout)**  
   - Sets up global styles (Tailwind, ShadcN), top-level error boundaries, and side navigation.  
   - Initializes Supabase client, checks authentication state.

2. **`HomePage.tsx`**  
   - Displays landing area for non-authenticated users.  
   - Lists public channels (read-only) and invites sign-up/login.

3. **`AuthPage.tsx`**  
   - Houses sign-up form (email, password, display name).  
   - Houses login form.  
   - Handles email verification reminders.

4. **`ChannelList.tsx`** (Sidebar)  
   - Lists channels filtered by RLS policies.  
   - Shows unread counts via real-time subscriptions.

5. **`ChannelPage.tsx`**  
   - Displays main chat feed for a channel.  
   - Shows last 50 messages by default with pagination.  
   - Contains message input at bottom.

6. **`DmList.tsx`** (Sidebar)  
   - Displays user's direct message conversations.  
   - Shows participant's name, photo, and preview.

7. **`DmPage.tsx`**  
   - Like `ChannelPage.tsx` but for private DMs.  
   - Real-time message feed, attachments, and reactions.

8. **`ThreadPane.tsx`**  
   - Appears alongside `ChannelPage.tsx`.  
   - Displays replies for a specific "root" message.  
   - Includes input, real-time feed, attachments, reactions.

9. **`SearchPage.tsx`**  
   - Contains search bar.  
   - Lists matching messages using Postgres full-text search.  
   - Clicking result jumps to specific message.

10. **`SettingsPage.tsx`**  
    - Manages display name, profile picture, auto status, AI responder mode.  
    - Includes text field for AI context.

11. **`JonathanPage.tsx`**  
    - Chat interface for RAG-based AI queries.  
    - Real-time updates via Supabase subscriptions.

12. **`FilePreview.tsx`**  
    - Renders inline previews or file icons.  
    - Download button if user has permission.

13. **`EmojiPicker.tsx`**  
    - Grid of predefined emojis for reactions.  
    - Attaches to messages or thread messages.

---

## 4. Key Middleware Functions & Explanation

Below are key helper functions and policies that ensure proper access control and data integrity.

1. **Row Level Security (RLS)**  
   - **Purpose**: Enforces access control at the database level.  
   - **Implementation**: PostgreSQL policies on each table.
   - **Examples**:
     - Users can only read/write their own data.
     - Private channels hidden from non-members.
     - Message access requires channel membership.

2. **Real-time Subscriptions**  
   - **Purpose**: Delivers instant updates to clients.  
   - **Implementation**: PostgreSQL NOTIFY/LISTEN + Supabase real-time.
   - **Examples**:
     - New messages in current channel.
     - Reaction updates.
     - Presence changes.

3. **File Access Control**  
   - **Purpose**: Manages file upload/download permissions.  
   - **Implementation**: Storage bucket RLS policies.
   - **Examples**:
     - Size limit checks.
     - Channel/DM membership verification.
     - Preview generation rules.

4. **AI Processing**  
   - **Purpose**: Handles RAG queries and auto-responses.  
   - **Implementation**: Edge Functions + pgvector.
   - **Examples**:
     - Embedding generation.
     - Similarity search.
     - Response formatting.

5. **Presence System**  
   - **Purpose**: Tracks user online status.  
   - **Implementation**: Supabase Presence.
   - **Examples**:
     - Online/offline detection.
     - Typing indicators.
     - Auto-idle updates.

By combining these components, we create a secure, scalable, and real-time system leveraging Supabase's features and PostgreSQL's capabilities.

--- 