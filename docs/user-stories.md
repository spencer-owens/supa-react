## 1. Authentication & Onboarding

### US-001: View Public Home Page
- **As a** visitor (not logged in)  
- **I want to** see a public home page with brief instructions and a list of public channels  
- **So that** I can explore the app's basic offerings before deciding to sign up  
- **Acceptance Criteria**:
  - Visible list of public channels (read-only)
  - A prompt to sign in or sign up to interact

### US-002: Sign Up
- **As a** new visitor  
- **I want to** create an account with a username, email, and password  
- **So that** I can become a registered user and interact with the app  
- **Acceptance Criteria**:
  - Validations for required fields (username, email, password)
  - Email verification flow using Supabase Auth
  - Confirmation message upon successful sign up
  - Row Level Security (RLS) policies are applied based on user role

### US-003: Email Verification
- **As a** newly registered user  
- **I want to** verify my email through Supabase Auth
- **So that** I can access all features (e.g., posting messages, file uploads)  
- **Acceptance Criteria**:
  - After sign-up, Supabase sends an email with a verification link
  - Until email is verified, user is restricted by RLS policies
  - Once verified, user can perform all allowed actions

### US-004: Sign In
- **As a** returning user  
- **I want to** sign in using Supabase Auth with my email and password  
- **So that** I can access my channels, direct messages, and settings  
- **Acceptance Criteria**:
  - Must be able to handle incorrect credentials gracefully
  - Successful login redirects me to my main workspace view
  - If the email is unverified, user is prompted to verify or resend through Supabase Auth

### US-005: Guest Attempts to Take Action
- **As a** guest (not logged in)  
- **I want to** be prompted to sign up/log in when trying to do something privileged (e.g., post a message, join a channel)  
- **So that** the system restricts me from unauthorized actions  
- **Acceptance Criteria**:
  - Any restricted action triggers a sign-up/log-in prompt
  - Once logged in, the user can complete the attempted action
  - RLS policies enforce access control at the database level

---

## 2. Real-Time Messaging

### US-006: Send and Receive Messages in Public Channel
- **As a** logged-in user  
- **I want to** send and see messages in public channels in real time  
- **So that** I can communicate instantly with others  
- **Acceptance Criteria**:
  - Messages appear immediately using Supabase real-time subscriptions
  - The input box is at the bottom with a "send" icon
  - The most recent messages are at the bottom of the feed
  - Postgres NOTIFY/LISTEN powers real-time updates

### US-007: Unread Message Indicators
- **As a** logged-in user  
- **I want to** see an unread message bubble or count next to each channel/DM  
- **So that** I can know which channels have new activity  
- **Acceptance Criteria**:
  - A small bubble increments in real time using Postgres triggers
  - The bubble disappears once I view the channel/DM
  - Unread counts are maintained through Postgres functions

### US-008: Message Loading and Pagination
- **As a** logged-in user  
- **I want to** see only the 50 most recent messages by default and load older messages on demand  
- **So that** I can have faster initial loads and still access older history when needed  
- **Acceptance Criteria**:
  - Default load is 50 messages using Postgres LIMIT
  - "Load more" button fetches older messages with efficient pagination
  - Scrolling up triggers loading previous pages

### US-009: Typing Indicators
- **As a** logged-in user  
- **I want to** see when other users are typing in real time  
- **So that** I can anticipate incoming messages  
- **Acceptance Criteria**:
  - Typing indicators appear immediately using Supabase Presence
  - Indicators disappear when user stops typing or sends the message
  - Presence state is managed efficiently through Supabase

---

## 3. Channel & Direct Message Organization

### US-010: View Public vs. Private Channels
- **As a** logged-in user  
- **I want to** see two sections under "Channels": Public and Private  
- **So that** I can differentiate the visibility of these channels  
- **Acceptance Criteria**:
  - **Private** section only appears if I am a member of at least one private channel
  - Public channels are visible to everyone, private channels only to members through RLS
  - Channel visibility is enforced at the database level

### US-011: Create a New Channel
- **As a** logged-in user  
- **I want to** create either a public or private channel  
- **So that** I can start a conversation space for specific topics or people  
- **Acceptance Criteria**:
  - Channel creation form (channel name, public/private, optional description)
  - Real-time update through Supabase subscriptions
  - Private channels hidden from non-members through RLS policies

### US-012: Join a Public Channel
- **As a** logged-in user  
- **I want to** join a public channel by clicking a "Join" button (or by simply entering it)  
- **So that** I can post messages in that channel  
- **Acceptance Criteria**:
  - Real-time membership update
  - Once joined, I see the channel under my list of channels
  - If I try to post in a public channel before joining, system automatically prompts to join (if that's the chosen flow)

### US-013: Join or Request Access to a Private Channel
- **As a** logged-in user  
- **I want to** be invited or added to a private channel  
- **So that** I can see and post in that channel  
- **Acceptance Criteria**:
  - I cannot see a private channel at all if I'm not a member
  - Invites trigger a membership update in real time
  - Once joined, private channel appears in my sidebar under "Private"

### US-014: Leave a Channel
- **As a** logged-in user  
- **I want to** leave a channel (public or private if I'm a member)  
- **So that** my membership is removed and I don't see it in my sidebar  
- **Acceptance Criteria**:
  - A "Leave Channel" button
  - Real-time update removes channel from my sidebar
  - If the channel is public, I can rejoin anytime; if private, I need another invite or channel membership approach

### US-015: Direct Messages (DM)
- **As a** logged-in user  
- **I want to** see a list of all users and initiate a 1-on-1 conversation  
- **So that** I can send private messages directly  
- **Acceptance Criteria**:
  - A DM list showing all users with the most recent message excerpt
  - Ability to click a user's name to start or continue a DM
  - Real-time updates to unread message counts in the DM list

### US-016: Delete a Channel (Creator Only)
- **As a** channel creator  
- **I want to** delete my own channel  
- **So that** it is no longer accessible or visible to any member  
- **Acceptance Criteria**:
  - Only the original creator sees a "Delete Channel" option
  - Once confirmed, channel and all messages are removed
  - Other members see the channel disappear in real time

---

## 4. File Sharing

### US-017: Upload Files to Channel or DM
- **As a** logged-in user  
- **I want to** upload images, docs, videos, or audio files up to 10 MB  
- **So that** I can share relevant files in conversations  
- **Acceptance Criteria**:
  - Realtime display of newly uploaded file using Supabase Storage
  - If file exceeds 10 MB, system rejects the upload
  - Inline preview for common file types (images, etc.)
  - RLS policies control file access based on channel/DM membership

### US-018: Download Shared Files
- **As a** logged-in user  
- **I want to** download files shared in a channel or DM from Supabase Storage 
- **So that** I can access them locally  
- **Acceptance Criteria**:
  - Download button or direct link using Supabase Storage URLs
  - RLS policies verify channel/DM membership before allowing download
  - Proper access checks enforced at database level

---

## 5. Search

### US-019: Search Messages by Text
- **As a** logged-in user  
- **I want to** search for specific messages across channels and DMs  
- **So that** I can quickly find relevant conversation history  
- **Acceptance Criteria**:
  - A sidebar or top-level search bar that leads to `/search?query=...`
  - Returns paginated results using Postgres full-text search
  - Results link to the exact message location
  - Search respects RLS policies for private channels and DMs

### US-020: Search Attachments by File Name
- **As a** logged-in user  
- **I want to** include shared file names in search results  
- **So that** I can locate attachments posted in conversations  
- **Acceptance Criteria**:
  - File name indexing in Postgres
  - If the query matches any file name, show it in search results
  - Clicking a result opens the channel/DM at the file's message
  - Search uses Postgres tsvector for efficient file name matching

### US-021: Jump to Message Location
- **As a** logged-in user  
- **I want to** jump directly to the message or file in the channel/DM feed  
- **So that** I can see it in context  
- **Acceptance Criteria**:
  - Clicking a search result loads the channel/DM
  - The UI automatically scrolls to or highlights the matching message
  - Simple, non-animated jump is sufficient

---

## 6. User Presence & Status

### US-022: Automatic Status Updates
- **As a** logged-in user  
- **I want to** have my status automatically set to Online when I'm active, Idle after 15 minutes, and Offline when I close the app  
- **So that** others can see my current availability  
- **Acceptance Criteria**:
  - Online changes to Idle after 15 min of no user activity
  - Offline triggers immediately on logout or tab close
  - Status changes broadcast to other users in real time

### US-023: Manual Busy Setting
- **As a** logged-in user  
- **I want to** manually set my status to Busy at any time  
- **So that** others know I don't want to be disturbed  
- **Acceptance Criteria**:
  - Busy overrides automatic updates
  - Busy remains until I manually reset or choose another status
  - Status changes appear in real time for everyone

### US-024: Opt Out of Automatic Status
- **As a** logged-in user  
- **I want to** disable automatic status updates  
- **So that** I can set a fixed status that does not change automatically  
- **Acceptance Criteria**:
  - A toggle in Settings to disable auto-updates
  - If disabled, my status remains as I last set it
  - The toggle itself is real-time (changes take effect immediately)

---

## 7. Thread Support (Channels Only)

### US-025: Start a Thread
- **As a** logged-in user in a channel  
- **I want to** click "Reply in Thread" on a particular message  
- **So that** I can have a focused conversation without cluttering the main channel  
- **Acceptance Criteria**:
  - A separate thread pane appears on the right
  - Thread messages are real time
  - Attachments and emoji reactions can be posted within the thread

### US-026: View Thread Messages
- **As a** channel member  
- **I want to** see all replies to a particular message in a thread pane  
- **So that** I can follow sub-discussions more easily  
- **Acceptance Criteria**:
  - Collapsed thread indicator on the main message
  - Thread pane shows the entire conversation specific to that message
  - Real-time updates as new replies arrive

---

## 8. Emoji Reactions

### US-027: React to a Message
- **As a** logged-in user  
- **I want to** add an emoji reaction to any message (channel or DM)  
- **So that** I can quickly express my response  
- **Acceptance Criteria**:
  - A predefined set of emojis is available
  - Ability to add multiple emojis to the same message
  - Real-time counts increase when multiple users add the same emoji

### US-028: Remove or Change My Reaction
- **As a** logged-in user  
- **I want to** remove an emoji reaction or change it to a different one  
- **So that** I can correct a mistake or update my response  
- **Acceptance Criteria**:
  - Removing a reaction decrements the count in real time
  - Changing a reaction updates the emoji and count accordingly

---

## 9. Settings

### US-029: Change Display Name
- **As a** logged-in user  
- **I want to** update my display name in Settings  
- **So that** others see my new name in channels, DMs, etc.  
- **Acceptance Criteria**:
  - Display name change is immediate in real time across the app
  - Validation for not allowing empty names

### US-030: Update Profile Picture
- **As a** logged-in user  
- **I want to** set or change my profile picture  
- **So that** my avatar is updated in real time  
- **Acceptance Criteria**:
  - Picture uploads (with typical size constraints)
  - Updated avatar immediately visible to others

### US-031: Choose/Opt-Out of Automatic Status
- **As a** logged-in user  
- **I want to** toggle automatic status updates and choose a fixed status if desired  
- **So that** I can control how I appear to others  
- **Acceptance Criteria**:
  - A simple toggle: enable or disable auto status
  - Manual override for Online, Idle, Busy, Offline

### US-032: Manage AI Autoresponder
- **As a** logged-in user  
- **I want to** configure whether the AI responds on my behalf when I'm offline, always, or never  
- **So that** I have control over how the AI interacts with others  
- **Acceptance Criteria**:
  - Three settings: 
    1. **AI on** (always responds)  
    2. **AI offline** (responds only when I'm Offline)  
    3. **AI off** (never responds)  
  - Changing the setting is effective immediately

---

## 10. RAG AI ("Jonathan") Chat

### US-033: Navigate to Jonathan (AI) Conversation
- **As a** logged-in user  
- **I want to** access the AI chat interface via the `/jonathan` route  
- **So that** I can ask questions and get AI-powered responses  
- **Acceptance Criteria**:
  - Dedicated chat UI similar to DM interface
  - Message history stored in Postgres with proper indexing
  - Real-time updates through Supabase subscriptions

### US-034: Ask Jonathan a Question
- **As a** logged-in user  
- **I want to** type a question and get a contextual response  
- **So that** I can leverage the AI's knowledge of my workspace  
- **Acceptance Criteria**:
  - Query is sent to Supabase Edge Function
  - Relevant past messages are retrieved via pgvector similarity search
  - The LLM returns a response that's displayed as a new message in the feed
  - All interactions are stored in Postgres for future context

---

## 11. RAG AI Avatar Responses

### US-035: AI Responds When I'm Offline
- **As a** user with "AI offline" enabled  
- **I want to** have the AI automatically respond on my behalf when I'm offline  
- **So that** colleagues can still get my input or answers  
- **Acceptance Criteria**:
  - When user status is "Offline," a mention (@Username) triggers an Edge Function
  - Response uses pgvector to find relevant context from user's messages
  - The response text is labeled or identified as AI
  - Real-time message appears through Supabase subscriptions

### US-036: AI Responds Even When I'm Online
- **As a** user with "AI on"  
- **I want to** have the AI respond for me at all times  
- **So that** I can rely on the AI to handle certain queries  
- **Acceptance Criteria**:
  - AI triggers on any mention of @Username via Edge Functions
  - The user can still manually interject
  - The system clearly differentiates AI messages from user-authored messages
  - Real-time updates through Postgres NOTIFY/LISTEN

### US-037: Provide AI Context in Settings
- **As a** logged-in user  
- **I want to** add personal context or instructions for the AI in my settings  
- **So that** the AI's responses are more aligned with my preferences  
- **Acceptance Criteria**:
  - A free-text field in Settings stored in Postgres
  - This text is included in the Edge Function context
  - Real-time update if I change the context through Supabase

---

## 12. Error Handling & Edge Cases

### US-038: Posting in a Public Channel I Haven't Joined
- **As a** logged-in user  
- **I want to** see a prompt to join the channel first if I attempt to post  
- **So that** I can't post as a non-member  
- **Acceptance Criteria**:
  - Button/link to "Join Channel"
  - RLS policies prevent posting without membership
  - Real-time membership update through Postgres triggers
  - Error message if I try to post or reply while not a member

### US-039: Attempting to Access a Private Channel I'm Not a Member Of
- **As a** logged-in user  
- **I want to** not see any private channel I'm not a member of  
- **So that** private channels remain truly hidden  
- **Acceptance Criteria**:
  - RLS policies completely hide private channels from non-members
  - Private channel names do not appear in any queries
  - Direct link attempts return an access-denied response
  - Security enforced at database level through Postgres policies

### US-040: Trying to Reply in a Thread of a Public Channel I Haven't Joined
- **As a** logged-in user  
- **I want to** be blocked from replying until I join the channel  
- **So that** membership rules are enforced consistently  
- **Acceptance Criteria**:
  - RLS policies prevent thread replies without channel membership
  - "Join Channel to Reply" message if I attempt to post
  - Once joined, Postgres policies allow replies
  - Real-time updates through Supabase subscriptions

### US-041: Exceeding File Size Limit
- **As a** logged-in user  
- **I want to** see an error if I upload a file > 10 MB  
- **So that** I understand why the upload fails  
- **Acceptance Criteria**:
  - Supabase Storage enforces size limit
  - A clear "File too large" error message
  - No partial uploads allowed
  - File size validation on both client and server

---

## User Role Summary

- **Role**: **Regular User** (no advanced admin privileges)  
- **Main Goals**:
  1. Communicate in public or private channels and direct messages  
  2. Share files through Supabase Storage
  3. Use Postgres full-text search to find past messages and attachments  
  4. Manage personal settings (display name, status, profile pic, AI preferences)  
  5. Interact with AI (Jonathan) through Edge Functions
- **Key Permissions**:
  - Create/join/leave public channels (controlled by RLS)  
  - Create private channels and invite others (private channels remain hidden through RLS)  
  - Post messages, react with emojis, upload/download files  
  - Trigger AI responses through Edge Functions if enabled
- **Typical Workflows**:
  1. **Sign Up & Authenticate**: The user signs up through Supabase Auth, verifies email, logs in.  
  2. **Explore Channels**: The user sees public channels based on RLS policies.  
  3. **Send Messages**: The user posts messages with real-time updates through NOTIFY/LISTEN.  
  4. **User Presence & Status**: The user's status is managed through Supabase Presence.  
  5. **Threading**: The user replies in threads with proper RLS checks.  
  6. **Search**: The user leverages Postgres full-text search.  
  7. **Settings**: The user updates profile with real-time Supabase subscriptions.  
  8. **AI Interaction**: The user chats with Jonathan or uses AI auto-response through Edge Functions.

---