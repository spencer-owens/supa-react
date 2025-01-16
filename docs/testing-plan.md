# Slack Clone Testing Plan

Below is an exhaustive list of tests covering all major features. Each test description outlines which parts of the code are affected, the purpose, the timing, the passing criteria, and the contingency plan if it fails.

---

## 1. Authentication Tests

### T1: **Sign Up Flow**
- **What**: Validate the sign-up form (username, email, password) and creation of a new user in Supabase Auth, plus the PostgreSQL user record.  
  - Affected Areas: `AuthPage.tsx`, Supabase Auth config, RLS policies.  
- **Why**: Ensures new users can register without errors and with correct data persisted.  
- **When**: After implementing sign-up functionality and on every subsequent code change to auth modules.  
- **Passing**: User record is created, email verification is sent, no JavaScript runtime errors occur, and typed fields (username, email) are validated.  
- **Contingency**: If it fails, disable pushing sign-up-related changes to production until the bug is fixed (e.g., handle validation or RLS policies properly).

### T2: **Email Verification Restriction**
- **What**: Attempt to post a message or create a channel with an unverified user.  
  - Affected Areas: RLS policies, client logic preventing unverified writes.  
- **Why**: Ensures that unverified users can't access full privileges.  
- **When**: After finishing email verification workflow, re-run any time RLS policies or client checks are updated.  
- **Passing**: Unverified user sees an error/prompt to verify email; verified user can proceed without issue.  
- **Contingency**: If it fails, verify that RLS policies and front-end checks correctly reference `email_verified`.

### T3: **Sign In Flow**
- **What**: Test logging in with valid, invalid, and locked-out credentials.  
  - Affected Areas: `AuthPage.tsx`, Supabase Auth settings.  
- **Why**: Ensures correct error handling and session establishment.  
- **When**: Immediately after implementing sign-in logic and whenever updating front-end session logic.  
- **Passing**: Successful sign-in with valid credentials, immediate errors with invalid, consistent user session state.  
- **Contingency**: If it fails, revert the sign-in or session logic commits and fix the error in the auth flow.

---

## 2. Channel & Membership Tests

### T4: **Create Public Channel**
- **What**: User creates a public channel and verifies it appears for all users in the public list.  
  - Affected Areas: `ChannelPage.tsx`, `ChannelList.tsx`, PostgreSQL channel table, RLS policies.  
- **Why**: Confirms new channel creation workflow and real-time updates in channel listings.  
- **When**: After channel creation implementation and any membership rule changes.  
- **Passing**: Channel record is created in PostgreSQL, immediately visible in other users' public channel lists via Supabase real-time.  
- **Contingency**: If it fails, ensure RLS policies and PostgreSQL triggers are correct.

### T5: **Create Private Channel**
- **What**: User creates a private channel; only the creator sees it until invitations are sent.  
  - Affected Areas: `ChannelPage.tsx`, channel_membership table, RLS policies.  
- **Why**: Ensures private channel logic (invisibility to non-members) is enforced.  
- **When**: After private channel functionality is implemented, re-run when membership or channel listing logic changes.  
- **Passing**: Non-members cannot see the channel, members see it instantly, type checks for channel data are correct.  
- **Contingency**: If it fails, fix RLS policies or membership checks to hide the channel from non-members.

### T6: **Join/Leave Channel**
- **What**: User joins a public channel or receives an invite to a private channel, then leaves it later.  
  - Affected Areas: Channel membership APIs, `ChannelList.tsx`, real-time subscriptions.  
- **Why**: Verifies membership record creation/removal and immediate UI updates.  
- **When**: After join/leave logic is built, re-run after channel membership or UI changes.  
- **Passing**: New membership record in PostgreSQL, channel appears in user's sidebar, leaving removes membership record and hides the channel.  
- **Contingency**: If it fails, check membership record creation/deletion code and associated real-time subscriptions.

---

## 3. Real-Time Messaging Tests

### T7: **Send & Receive Messages**
- **What**: Two logged-in users (in separate browsers or test mocks) post messages in the same channel and verify real-time updates.  
  - Affected Areas: `ChannelPage.tsx`, messages table, Supabase subscriptions, SubscriptionManager.  
- **Why**: Confirms real-time features are functioning with immediate message reflection and proper connection management.  
- **When**: After message posting is implemented, re-run on relevant code changes.  
- **Passing**: 
  - Both users see each other's messages instantly via NOTIFY/LISTEN
  - No console errors
  - Data types match expected structures
  - Connection count stays within limits
  - Subscription Manager properly tracks channels
- **Contingency**: If it fails, investigate Supabase subscription setup, PostgreSQL triggers, or connection management issues.

### T8: **Typing Indicator**
- **What**: User begins typing, other users see a typing indicator in real time.  
  - Affected Areas: Supabase Presence, front-end display logic, shared presence channel.  
- **Why**: Ensures the ephemeral "typing" state is correctly triggered and observed through multiplexed channels.  
- **When**: After implementing typing indicators, re-run after any change to Presence code.  
- **Passing**: 
  - Typing indicator appears/disappears correctly through Presence
  - Performance remains stable
  - Shared presence channel properly manages multiple users
  - No connection overflow occurs
- **Contingency**: If it fails, check the logic that sets/clears typing states in Presence and channel management.

### T9: **Unread Message Bubbles**
- **What**: Users see unread counts for channels/DMs they haven't viewed, which reset upon viewing.  
  - Affected Areas: `ChannelList.tsx`, `DmList.tsx`, PostgreSQL triggers for unread counts.  
- **Why**: Confirms the unread count increments/decrements accurately.  
- **When**: After implementing unread logic, re-run on membership or message read changes.  
- **Passing**: The count reflects real-time updates through triggers, resets to zero immediately upon viewing.  
- **Contingency**: If it fails, adjust PostgreSQL triggers that track unread counts.

---

## 4. File Sharing Tests

### T10: **File Upload & Preview**
- **What**: User uploads images, docs, and videos (≤10MB) in a channel/DM and sees inline previews.  
  - Affected Areas: `FilePreview.tsx`, Storage RLS policies, file upload code, message attachment logic.  
- **Why**: Validates file type checks, real-time addition to the message feed, and secure storage.  
- **When**: After implementing file upload, re-run whenever changes to file logic or new file type acceptance occurs.  
- **Passing**: File successfully uploads to Supabase Storage, message includes an attachment object, preview displays without console errors.  
- **Contingency**: If it fails, check file size validation, Storage RLS policies, and message attachments JSONB for correct data types.

### T11: **File Download**
- **What**: A user in a channel or DM attempts to download a shared file, verifying membership checks.  
  - Affected Areas: Storage RLS policies, download link logic, membership checks.  
- **Why**: Ensures only authorized users (channel members or DM participants) can download.  
- **When**: After file download links are generated, re-run whenever membership logic or Storage RLS policies are updated.  
- **Passing**: Authorized user can download, non-member or external user is denied by RLS.  
- **Contingency**: If it fails, fix any mismatch in Storage RLS policies or membership checks.

---

## 5. Search Tests

### T12: **Search Messages & File Names**
- **What**: Query various keywords in `/search` and confirm relevant message text or file names appear in paginated results.  
  - Affected Areas: PostgreSQL full-text search, `SearchPage.tsx`, pagination.  
- **Why**: Ensures users can locate messages by content or file name.  
- **When**: After implementing search features, re-run after major data schema or index changes.  
- **Passing**: Accurate results returned using tsvector, pagination works, clicking a result jumps to the correct message.  
- **Contingency**: If it fails, inspect PostgreSQL indexes, full-text search configuration, and route logic.

---

## 6. User Presence & Status Tests

### T13: **Automatic Idle & Offline**
- **What**: User remains inactive for 15 minutes to trigger idle, closes browser to trigger offline, and checks UI updates.  
  - Affected Areas: Supabase Presence, `status` field in users table.  
- **Why**: Validates auto status updates and real-time reflection to other users.  
- **When**: After presence logic is implemented, re-run after status-handling code changes.  
- **Passing**: Status changes to "idle" after 15 minutes, "offline" on tab close, visible to others through Presence.  
- **Contingency**: If it fails, correct the client or server logic that sets the user's status in Presence.

### T14: **Manual Busy Setting**
- **What**: User toggles "busy" status and ensures it persists until explicitly changed.  
  - Affected Areas: `SettingsPage.tsx`, users table updates, Presence display.  
- **Why**: Verifies manual override works correctly and remains set across re-renders or restarts.  
- **When**: Post-implementation, re-run on any presence or settings refactor.  
- **Passing**: Busy status doesn't revert to idle or offline automatically; other users see "busy" in real time via Presence.  
- **Contingency**: If it fails, fix any front-end logic that accidentally overwrites "busy" or Presence handling.

---

## 7. Thread Support Tests

### T15: **Thread Creation & View**
- **What**: User replies to a specific channel message and sees a right-pane thread with real-time updates.  
  - Affected Areas: `ThreadPane.tsx`, `thread_root_id` in messages table.  
- **Why**: Checks that threaded messages are stored and retrieved correctly, separate from the main feed.  
- **When**: After thread logic is built, re-run after changes to channel messaging.  
- **Passing**: Thread messages appear only in the thread panel, real-time updates function through subscriptions.  
- **Contingency**: If it fails, validate `thread_root_id` usage and PostgreSQL queries for thread messages.

---

## 8. Emoji Reaction Tests

### T16: **Add & Remove Emoji Reactions**
- **What**: Multiple users react to the same message, verify counts increment, remove a reaction, verify counts decrement.  
  - Affected Areas: reactions table, PostgreSQL triggers, UI elements.  
- **Why**: Ensures real-time updates to reaction counts and user-lists.  
- **When**: After reaction feature is added, re-run whenever message schema or real-time logic changes.  
- **Passing**: Reaction count updates for all viewers through triggers, removing it subtracts user from reaction.  
- **Contingency**: If it fails, check reaction record creation/deletion logic and PostgreSQL triggers.

---

## 9. AI (Jonathan) & Autoresponder Tests

### T17: **RAG Query & Response**
- **What**: User sends a query to Jonathan, system performs similarity search and returns relevant context.  
  - Affected Areas: Edge Function, pgvector similarity search, OpenAI embeddings.  
- **Why**: Validates the RAG pipeline and real-time response delivery.  
- **When**: After implementing the Jonathan chat feature, re-run on embedding or similarity search changes.  
- **Passing**: Query embedding generated, similar content found via pgvector, response delivered in real time.  
- **Contingency**: If it fails, check Edge Function logs, pgvector indexes, and embedding generation.

### T18: **AI Autoresponder (Offline Mode)**
- **What**: User sets `ai_responder_mode = 'offline'`, goes offline, and another user @mentions them.  
  - Affected Areas: Edge Function, Presence detection, pgvector context retrieval.  
- **Why**: Confirms the AI responds appropriately when user is offline.  
- **When**: After implementing autoresponder, re-run on AI logic changes.  
- **Passing**: AI response triggers when offline, logs to ai_activity_log, appears with proper label.  
- **Contingency**: If it fails, check Presence detection or Edge Function trigger conditions.

### T19: **AI Autoresponder (On Mode)**
- **What**: User sets `ai_responder_mode = 'on'`, remains logged in, and sees the AI respond whenever the user is tagged.  
  - Affected Areas: Same as above but triggered even if user is active.  
- **Why**: Confirms the always-on mode logic, ensures the user can see their own AI responses in real time.  
- **When**: After confirming offline mode works, test "on" mode.  
- **Passing**: AI response triggers on @mention, logs to ai_activity_log, user sees it labeled.  
- **Contingency**: If it fails, diagnose mention detection or Edge Function boundaries.

---

## 10. TypeScript Type Safety Tests

### T20: **Type-Checking & Lint**
- **What**: Run ESLint and TypeScript type checker on the codebase.  
  - Affected Areas: Every TypeScript file, especially data model types and API calls.  
- **Why**: Prevents common type errors and ensures correct usage of data shapes.  
- **When**: Before each commit via a pre-commit hook and during continuous integration runs.  
- **Passing**: No unresolved type warnings, no lint errors, consistent use of data model types.  
- **Contingency**: If it fails, fix type definitions or data model references before merging any code.

---

## 11. Connection Management Tests

### T21: **Subscription Manager Limits**
- **What**: Test the Subscription Manager's handling of channel limits and cleanup.
  - Affected Areas: `SubscriptionManager.ts`, real-time connection handling.
- **Why**: Ensures the application maintains a healthy number of active channels and properly manages resources.
- **When**: After implementing connection management, re-run on any changes to subscription logic.
- **Passing**:
  - Channel count never exceeds maximum limit (50)
  - Inactive channels are properly cleaned up
  - New subscriptions work after cleanup
  - No memory leaks occur
- **Contingency**: If it fails, review cleanup logic and channel lifecycle management.

### T22: **Channel Multiplexing**
- **What**: Verify that shared channels properly handle multiple data types and users.
  - Affected Areas: `ChannelMultiplexer.ts`, shared channel implementation.
- **Why**: Confirms efficient use of real-time connections through channel sharing.
- **When**: After implementing channel multiplexing, re-run on changes to shared channel logic.
- **Passing**:
  - Multiple data types correctly handled in shared channels
  - No cross-contamination of data
  - Performance remains stable under load
  - Clear separation of concerns maintained
- **Contingency**: If it fails, review multiplexing logic and data separation.

### T23: **Connection Recovery**
- **What**: Test automatic recovery from connection issues and proper cleanup.
  - Affected Areas: Connection monitoring, error recovery logic.
- **Why**: Ensures the application gracefully handles connection problems and maintains stability.
- **When**: After implementing connection monitoring, re-run on relevant changes.
- **Passing**:
  - Automatic reconnection works after disconnection
  - Resources are properly cleaned up
  - User experience remains smooth
  - Error messages are clear and helpful
- **Contingency**: If it fails, review recovery logic and cleanup procedures.

### T24: **Resource Monitoring**
- **What**: Verify that connection metrics are properly tracked and logged.
  - Affected Areas: `ConnectionMonitor.ts`, metrics gathering.
- **Why**: Ensures proper monitoring of system resources and connection health.
- **When**: After implementing monitoring, re-run on changes to monitoring logic.
- **Passing**:
  - Metrics are accurately gathered
  - Performance issues are detected
  - Automatic interventions work as expected
  - Logging is comprehensive
- **Contingency**: If it fails, review monitoring implementation and logging system.

---

## 12. Performance Optimization Tests

### T25: **Message Virtualization**
- **What**: Test virtual list performance with large message sets.
  - Affected Areas: `VirtualMessageList.tsx`, message rendering logic.
- **Why**: Ensures efficient rendering of large message lists without performance degradation.
- **When**: After implementing virtualization, re-run on changes to list rendering.
- **Passing**:
  - Smooth scrolling with 10,000+ messages
  - Memory usage remains stable
  - No visible rendering gaps
  - Proper handling of dynamic message heights
- **Contingency**: If it fails, optimize virtualization parameters or rendering logic.

### T26: **State Management Performance**
- **What**: Test atomic state updates and caching efficiency.
  - Affected Areas: Channel state atoms, CacheManager implementation.
- **Why**: Verifies efficient state updates and prevents unnecessary re-renders.
- **When**: After implementing atomic state, re-run on state management changes.
- **Passing**:
  - Only affected components re-render
  - Cache hits for frequently accessed data
  - Memory usage within acceptable limits
  - Proper cache invalidation
- **Contingency**: If it fails, review state atomization or cache strategy.

### T27: **Database Query Performance**
- **What**: Test query performance with optimized indexes and materialized views.
  - Affected Areas: PostgreSQL indexes, materialized views, query patterns.
- **Why**: Ensures database queries meet performance requirements.
- **When**: After implementing database optimizations, re-run on schema changes.
- **Passing**:
  - Channel list queries < 100ms
  - Search queries < 500ms
  - Proper use of indexes
  - Materialized view updates don't impact performance
- **Contingency**: If it fails, analyze query plans and optimize indexes.

### T28: **Asset Loading Performance**
- **What**: Test optimized image loading and asset caching.
  - Affected Areas: `OptimizedImage` component, AssetCache service.
- **Why**: Verifies efficient asset loading and caching strategy.
- **When**: After implementing asset optimizations, re-run on asset handling changes.
- **Passing**:
  - Images load progressively
  - Cache hits for repeated assets
  - Proper fallback handling
  - Memory usage within limits
- **Contingency**: If it fails, review caching strategy or asset optimization logic.

### T29: **Performance Monitoring**
- **What**: Test performance metric collection and alerting.
  - Affected Areas: PerformanceMonitor service, metrics collection.
- **Why**: Ensures accurate performance tracking and timely alerts.
- **When**: After implementing monitoring, re-run on monitoring changes.
- **Passing**:
  - Accurate metric collection
  - Proper threshold alerts
  - Low overhead from monitoring
  - Comprehensive logging
- **Contingency**: If it fails, adjust monitoring parameters or collection methods.

---

## Conclusion

This test plan ensures coverage of **all critical features** (auth, channels, messaging, file sharing, search, presence, threads, emoji reactions, and AI functionality). Each test specifies **why** we need it, **when** it should run, **how** we know it passes, and **what** to do if it fails. By adhering to this plan, we maintain a robust and reliable Slack-clone application that meets all specified requirements. 