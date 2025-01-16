# rough-sketch.md

## Introduction

We’re building a Slack clone app. Below is a segmented outline of the initial feature set and the clarifications provided. The goal is to capture all the requirements, questions, and decisions in a structured way.

---

## Feature Outline

### 1. Authentication
- **Unauthenticated Access**  
  - Users landing on the main domain can see a home page and navigate to public channels, viewing messages without logging in.  
  - If they attempt to perform any action beyond viewing (e.g., send a message, create/join/leave a channel, react to a message, download or share a file, change their status), they must be prompted to log in or sign up.

- **Sign-up Process**  
  - Requires **username/display name**, **email**, and **password**.  
  - Uses **Supabase Auth** with built-in user management.  
  - **Email verification** is required before granting full, logged-in privileges.

### 2. Real-time Messaging
- **Immediate Updates**  
  - Messages in both channels and direct messages should be sent and received instantly via Supabase's real-time functionality (Postgres LISTEN/NOTIFY).  
  - The message input field should mirror Slack-like UIs (default text, paper airplane send icon, etc.).

- **Unread Indicators**  
  - In the sidebar, a small bubble shows the count of unread messages for each channel or direct message.  
  - Once a user navigates to that channel/DM, the count resets and the bubble disappears in real time.
  - Implemented using Postgres triggers and Supabase real-time subscriptions.

- **UI Layout**  
  - The message input field is at the bottom of the page.  
  - The most recent message appears just above the input field, with older messages above.  
  - Users can scroll up to see older messages.

### 3. Channel/DM Organization
- **Public vs. Private**  
  - Public channels are visible to everyone.  
  - Private channels are visible only to members. They do not appear at all to non-members.

- **Channel Management**  
  - A dedicated page to view all channels and manage membership.  
  - Users can create, join, or leave channels, with changes reflected in real time.  
  - Under “Channels” in the sidebar, two subsections: “Public” and “Private.”  
  - The “Private” subsection appears only if the user is a member of at least one private channel.

- **Direct Messages**  
  - A page where a user can see all other users and initiate a conversation.  
  - Shows the most recent conversations and the most recent message in each.  
  - Only the **most recent 50 messages** load by default for both channels and DMs.  
  - A **Load More** button can retrieve older messages.  
  - Scrolling up can eventually show the beginning of the conversation.

### 4. File Sharing
- **Upload, Store, Share, Download**  
  - Users can handle images, documents, videos, and audio in channels and direct messages in real time.  
  - An inline preview of the file should appear in the feed.

### 5. Search
- **Search Interface**  
  - A `/search` page exists, and a sidebar search bar redirects there with the query as a URL parameter.  
  - Search returns results from **both channels and direct messages** that the user has access to.  
  - Supports **pagination/scroll** for large result sets.  
  - Clicking a search result navigates to the channel/DM at the exact position of the message, not just a default view.

### 6. User Presence/Status
- **Four Statuses**  
  1. Online  
  2. Idle  
  3. Busy  
  4. Offline

- **Default Behavior**  
  - A user is set to **Online** by default when they log in.  
  - They move to **Idle** after 15 minutes of inactivity.  
  - They become **Offline** when they close the tab or window (via a performant approach using Firebase Realtime presence or a client event).  
  - **Busy** is always a manual user setting.

- **User Control**  
  - Users can opt out of automatic status updates in **Settings** and choose their own status.  
  - Status updates occur in real time.

- **Typing Indicators**  
  - Channels and direct messages detect when a user is typing and broadcast it immediately.

### 7. Thread Support
- **Channels Only**  
  - Messages in channels can receive “Reply in Thread.”  
  - The thread appears to the right of the main channel feed.  
  - The thread UI mirrors the main feed (its own message input, scrolling, file sharing, reactions in real time).

### 8. Emoji Reactions
- **Predefined Emoji Set**  
  - All message types can be reacted to with emojis.  
  - Users can apply multiple emojis to the same message.  
  - If multiple users use the same emoji, a count increments next to it.  
  - Reactions occur in real time.

### 9. Settings
- **Display Name/Username**  
  - Users can change these in Settings.

- **Status Configuration**  
  - Users can opt out of automatic status updates or set a custom status here.

- **Profile Picture**  
  - Users can add or change their profile picture in Settings.

### 10. RAG AI (“Jonathan”)
- **Dedicated Page**  
  - Accessible from the sidebar via a `/jonathan` route, which behaves like a DM interface.  
  - Message input at the bottom; the most recent message is just above it.  
  - Users can scroll up for older messages.

- **Backend Logic**  
  - When a user enters a query, the front-end sends it to a Supabase Edge Function.
  - All previous messages are vectorized with OpenAI embeddings and stored in pgvector.
  - A Retrieval-Augmented Generation (RAG) process runs in TypeScript, querying pgvector for relevant messages.
  - The LLM response is sent back to the UI and displayed in the feed.

### 11. RAG-Powered AI Response
- **Auto-Responder Logic**  
  - Users can enable an AI avatar to respond on their behalf when they are offline.  
  - Upon activation, users can provide extra info in Settings for more context.  
  - The AI responds on behalf of the user in channels when mentioned with `@username`, or in direct messages, using the same RAG process described above.

- **Three Settings Modes**  
  1. **AI activates when offline** (default is offline on tab/window close or logout).  
  2. **AI never activates** (completely disabled).  
  3. **AI on** (the AI responds even when the user is online).

- **Real-Time Updates**  
  - This setting change takes effect immediately.

---

## Clarifications & Decisions

### 1. User Status Behavior
> **Q:** Users go Idle after 15 minutes. Is this front-end only or server-side?  
> **A:** We'll use a combination of client-side detection and Supabase real-time presence for optimal performance.

> **Q:** When users close the tab/window, do we rely on Supabase real-time presence or a client event to set them Offline?  
> **A:** We'll use Supabase's real-time presence system with a fallback to client events for better reliability.

### 2. Private Channel Visibility
> **Q:** Are private channels completely hidden from non-members or just inaccessible?  
> **A:** They are completely hidden. The channel name and details do not appear at all unless you are a member.

### 3. Search Scope and Behavior
> **Q:** Should search results include attachments (file names, text in attachments)?  
> **A:** Yes to file names, but only message text is indexed—no need to index content *within* attachments. We'll use Postgres full-text search with proper indexing.

> **Q:** Any preference on the search engine approach?  
> **A:** **Simplicity** is key; use Postgres's built-in full-text search capabilities with proper indexing.

### 4. File Size/Type Limitations
> **Q:** Any max file size or type restrictions?  
> **A:** A maximum file size of **10MB**. No virus scanning or moderation at this point.

### 5. AI Avatar Offline Response
> **Q:** Should the AI respond immediately when the user is “Offline,” or is there a grace period?  
> **A:** It responds **as soon as** the user is offline.  

> **Q:** Does the AI respond if the user is Busy or Idle?  
> **A:** By default, only when Offline. However, users can choose from three modes in Settings:  
  1. AI triggers when Offline  
  2. AI never triggers  
  3. AI is always on

### 6. Thread Support in Direct Messages
> **Q:** Should direct messages support threads, or is that only for channels?  
> **A:** Threading is **only** for channels.

### 7. Admin/Workspace Owner Roles
> **Q:** Any admin features like deleting channels, seeing all private channels, or user management?  
> **A:** Out of scope for now. Only the channel creator can delete that channel.

### 8. Advanced or Future Features
> **Q:** Are we focusing on core features, or do we plan to add extras like message pinning, calls, etc.?  
> **A:** Focus on **listed features** for now.

### 9. Search UI “Jump To Location”
> **Q:** Should we use a fancy smooth scroll, or is a simple load that jumps to the correct position acceptable?  
> **A:** A simple load and jump is fine.

### 10. Other Edge Cases
> **Q:** If a user tries to reply in a thread in a channel they’re not a member of (perhaps from a direct link), how do we handle that?  
> **A:** A user cannot post in any channel (thread or main feed) if they’re not a member. If it’s a **public** channel, they can see the messages but must join to post. If they try to post before joining, they get an **error message** with a link to join the channel. Channel joining reflects in real time.

---

## Next Steps
This document outlines the **features** and **clarifications** for our Slack clone. The next phase might include creating detailed **user stories** for each feature to guide development, testing, and eventual production.

