# Slack Clone PRD

## 1. Project Overview
This project aims to build a real-time messaging platform with channels, direct messages, file sharing, and AI-powered features (RAG-based chatbot and AI autoresponder). It leverages Supabase (PostgreSQL, Auth, Storage, Edge Functions) and pgvector for AI features.

---

## 2. Core Workflows

1. **Sign Up & Verification**: Users register and must verify email through Supabase Auth to gain full privileges.  
2. **Channel Creation & Membership**: Users create public or private channels, join or leave them, with real-time updates through Supabase subscriptions.  
3. **Real-Time Messaging**: Users post messages, react with emojis, and see updates instantly via Postgres NOTIFY/LISTEN.  
4. **File Sharing**: Users upload and preview files up to 10MB using Supabase Storage, with RLS policies controlling access.  
5. **Search**: Users query across messages and attachments using Postgres full-text search, then jump to exact message locations.  
6. **User Presence & Status**: Users appear online, idle, busy, or offline in real time through Supabase Presence.  
7. **Threaded Conversations**: Users reply to messages in threaded side panels (channels only).  
8. **AI: Chat with Jonathan**: Users interact in a dedicated AI chat page, which uses pgvector for similarity search (RAG).  
9. **AI Autoresponder**: Offline or always-on mode automatically replies via Edge Functions when mentioned, logging responses.

---

## 3. Technical Foundation

### 3.1 Data Models
1. **User**  
   - Fields: `id`, `email`, `username`, `display_name`, `photo_url`, `email_verified`, `status`, `auto_status_enabled`, `ai_responder_mode`, `ai_context`, timestamps.  
   - Relations: Referenced by messages (`sender_id`) and channel_membership.
   - Notes: Managed by Supabase Auth with RLS policies.

2. **Channel**  
   - Fields: `id`, `name`, `type` (public/private), `created_by`, timestamps, `is_deleted`.  
   - Relations: Linked to channel_membership table.
   - Notes: RLS policies control visibility of private channels.

3. **Channel Membership**  
   - Fields: `id`, `user_id`, `channel_id`, `role`, `joined_at`.  
   - Relations: Links users to channels for access control.
   - Notes: Used by RLS policies to enforce permissions.

4. **DM Thread**  
   - Fields: `id`, `participants[]`, `last_message`, `updated_at`.  
   - Relations: Referenced by messages table.
   - Notes: RLS ensures only participants can access.

5. **Message**  
   - Fields: `id`, `content`, `sender_id`, `channel_id`, `dm_id`, `thread_root_id`, `attachments`, timestamps, `is_deleted`, `is_ai`.  
   - Notes: Full-text search enabled, attachments in Supabase Storage.

6. **Reaction**  
   - Fields: `id`, `message_id`, `user_id`, `emoji`, `created_at`.  
   - Notes: Composite unique constraint, triggers maintain counts.

7. **Vector Embeddings**  
   - Fields: `id`, `content_id`, `embedding`, `content_type`, `created_at`.  
   - Notes: Uses pgvector for similarity search.

8. **AI Activity Log**  
   - Fields: `id`, `user_id`, `message_id`, `context`, `created_at`.  
   - Notes: Tracks AI-generated responses with context.

---

### 3.2 API Endpoints
1. **Auth**:  
   - `POST /auth/signup` (no auth), `POST /auth/login` (no auth).  

2. **Users & Settings**:  
   - `GET /users/{id}`, `PATCH /users/{id}` (auth required, RLS enforced).  

3. **Channels**:  
   - `POST /channels` (create), `GET /channels` (list), `GET /channels/{channelId}`, `DELETE /channels/{channelId}`, `POST /channels/{channelId}/join`, `POST /channels/{channelId}/leave`.  

4. **Direct Messages**:  
   - `POST /dms` (initiate DM), `GET /dms/{dmId}` (metadata).  

5. **Messages**:  
   - `POST /messages` (create in channel/DM), `GET /messages` (list/paginate), `PATCH /messages/{messageId}` (edit), `POST /messages/{messageId}/reactions`, `DELETE /messages/{messageId}/reactions/{reactionId}`.  

6. **Files**:  
   - `POST /storage/upload` (check size <= 10MB), `GET /storage/objects/{bucket}/{path}`.  

7. **AI**:  
   - `POST /functions/jonathan` (query RAG system), `POST /functions/autorespond` (trigger auto-reply on mention if offline/on).  

---

### 3.3 Real-time Features
1. **Message Updates**:
   - Instant delivery through Supabase subscriptions
   - Postgres NOTIFY/LISTEN for efficient updates
   - Cursor-based pagination for history
   - **Connection Management**:
     - Centralized subscription manager to prevent channel overflow
     - Automatic cleanup of inactive channels
     - Channel multiplexing for shared resources
     - Connection pooling for efficient resource usage

2. **Presence & Status**:
   - Online/offline tracking via Supabase Presence
   - Typing indicators in channels/DMs
   - Auto-idle detection after 15 minutes
   - **Connection Optimization**:
     - Single presence channel for all user statuses
     - Batched status updates to reduce overhead
     - Efficient reconnection strategies

3. **Reactions & Threads**:
   - Real-time reaction updates through triggers
   - Thread notifications and unread counts
   - Participant presence in threads
   - **Channel Management**:
     - Shared channels for related features
     - Automatic subscription cleanup
     - Resource usage monitoring

4. **Search & Files**:
   - Full-text search using Postgres capabilities
   - File upload progress and preview generation
   - Access control through RLS policies
   - **Performance Optimization**:
     - Efficient query patterns
     - Connection pooling
     - Resource monitoring and cleanup

### 3.4 Connection Management Strategy
1. **Subscription Lifecycle**:
   - Centralized subscription manager
   - Automatic cleanup of inactive subscriptions
   - Resource usage monitoring and limits
   - Connection pooling configuration

2. **Channel Organization**:
   - Multiplexed channels for shared resources
   - Efficient channel reuse patterns
   - Clear subscription boundaries
   - Automatic cleanup triggers

3. **Performance Monitoring**:
   - Real-time connection metrics
   - Resource usage tracking
   - Performance bottleneck detection
   - Automatic intervention for issues

4. **Error Recovery**:
   - Graceful degradation strategies
   - Automatic reconnection handling
   - Clear error messaging
   - User feedback mechanisms

---

### 3.4 Security Model
1. **Authentication**:
   - Email/password via Supabase Auth
   - Required email verification
   - JWT tokens for API access

2. **Authorization**:
   - Row Level Security (RLS) policies
   - Channel membership checks
   - DM participant verification
   - File access control

3. **AI Features**:
   - Edge Functions for processing
   - pgvector for similarity search
   - Access-controlled context retrieval

---

## 4. User Experience Requirements

1. **Performance**:
   - Message delivery < 100ms
   - Search results < 500ms
   - File upload feedback
   - Typing indicator < 50ms

2. **Accessibility**:
   - WCAG 2.1 compliance
   - Keyboard navigation
   - Screen reader support
   - High contrast mode

3. **Error Handling**:
   - Clear error messages
   - Offline indicators
   - Upload size warnings
   - Permission explanations

4. **Mobile Support**:
   - Responsive design
   - Touch optimization
   - File upload from camera
   - Push notifications

---

## 5. Future Considerations

1. **Scalability**:
   - Connection pooling
   - Read replicas
   - Edge Functions distribution
   - CDN integration

2. **Features**:
   - Voice/video calls
   - Screen sharing
   - Message scheduling
   - Advanced permissions

3. **Integration**:
   - OAuth providers
   - External services
   - API webhooks
   - Export tools

4. **Analytics**:
   - Usage metrics
   - Performance monitoring
   - Error tracking
   - AI effectiveness

---

This PRD outlines the core requirements for building a modern, real-time chat platform using Supabase's powerful features. The combination of PostgreSQL's capabilities, real-time subscriptions, and Edge Functions provides a solid foundation for scalable, secure messaging with AI enhancements. 