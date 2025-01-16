# AI Implementation Guide - Supabase Version

## Overview
This document outlines our RAG (Retrieval Augmented Generation) implementation for Taut using Supabase Edge Functions, pgvector for similarity search, and real-time features through PostgreSQL NOTIFY/LISTEN. This implementation leverages Supabase's native features for efficient, scalable AI interactions.

## Current Architecture

### Frontend Components

1. **AI Context (`src/contexts/AIContext.tsx`)**
   - Manages global AI state using React Context
   - Handles communication with Supabase Edge Functions
   - Provides hooks for question asking and response management
   - Manages loading and error states
   - Subscribes to real-time updates via Supabase

2. **AI Components**
   - `AskAIInput.tsx`: Search-like input for questions
   - `AIResponse.tsx`: Displays answers with collapsible sources
   - `AIStatus.tsx`: Shows processing state and errors
   - All built with ShadcN UI components

### Backend Services

1. **Edge Functions (`functions/`)**
   ```typescript
   // Example Edge Function setup
   import { createClient } from '@supabase/supabase-js'
   import OpenAI from 'openai'

   const supabase = createClient(
     Deno.env.get('SUPABASE_URL')!,
     Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!
   )

   const openai = new OpenAI({
     apiKey: Deno.env.get('OPENAI_API_KEY')
   })
   ```

2. **Vector Store & Embeddings**
   - Uses pgvector for storing and querying embeddings
   - Implements periodic embedding updates via Edge Functions
   - Maintains content-embedding relationships in PostgreSQL

3. **Real-time Updates**
   - Leverages PostgreSQL NOTIFY/LISTEN for instant updates
   - Uses Supabase subscriptions for client-side updates
   - Handles errors and reconnections gracefully

## Implementation Details

### 1. Database Schema
```sql
-- Content tables with vector embeddings
create extension if not exists vector;

-- Messages table with embeddings
create table messages (
  id uuid primary key default gen_random_uuid(),
  content text not null,
  embedding vector(1536),
  created_at timestamp with time zone default timezone('utc'::text, now())
);

-- AI activity log
create table ai_activity_log (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references auth.users(id),
  query text not null,
  response text not null,
  context_messages jsonb,
  created_at timestamp with time zone default timezone('utc'::text, now())
);
```

### 2. Edge Functions

1. **Embedding Generation (`/functions/generate-embeddings.ts`)**
```typescript
async function generateEmbedding(content: string) {
  const response = await openai.embeddings.create({
    model: "text-embedding-3-small",
    input: content,
  });
  return response.data[0].embedding;
}

// Process content without embeddings
async function fetchContentWithoutEmbeddings(): Promise<ContentItem[]> {
  const results: ContentItem[] = [];
  
  // Fetch messages without embeddings
  const { data: messages, error: messagesError } = await supabase
    .rpc('messages_without_embeddings');
  if (messagesError) throw messagesError;
  
  return results;
}
```

2. **RAG Query Processing (`/functions/bot-messages.ts`)**
```typescript
async function processQuery(query: string, userId: string) {
  // Generate embedding for query
  const embedding = await generateEmbedding(query);
  
  // Find similar content
  const { data: similarContent } = await supabase
    .rpc('match_messages', {
      query_embedding: embedding,
      match_threshold: 0.78,
      match_count: 5
    });
    
  // Generate response with context
  const response = await generateResponse(query, similarContent);
  
  // Save to activity log
  await supabase.from('ai_activity_log').insert({
    user_id: userId,
    query,
    response,
    context_messages: similarContent
  });
  
  return response;
}
```

### 3. Real-time Features

1. **Client Subscriptions**
```typescript
// Subscribe to AI responses
supabase
  .channel('ai_responses')
  .on(
    'postgres_changes',
    {
      event: 'INSERT',
      schema: 'public',
      table: 'messages',
      filter: 'is_ai = true'
    },
    (payload) => {
      // Handle new AI message
    }
  )
  .subscribe()
```

2. **Database Triggers**
```sql
-- Notify clients of new AI responses
create function notify_ai_response()
returns trigger as $$
begin
  perform pg_notify(
    'ai_responses',
    json_build_object(
      'id', NEW.id,
      'content', NEW.content
    )::text
  );
  return NEW;
end;
$$ language plpgsql;

create trigger ai_response_notify
after insert on messages
for each row
when (NEW.is_ai = true)
execute function notify_ai_response();
```

## Current Limitations
- Processing time for large context windows
- Rate limits on OpenAI API calls
- Embedding update latency
- Basic error recovery strategies

## Future Enhancements
1. **Performance Optimization**
   - Implement caching for frequent queries
   - Batch embedding updates
   - Optimize similarity search thresholds

2. **User Experience**
   - Add streaming responses
   - Improve error handling and recovery
   - Enhance source attribution
   - Add feedback mechanism

3. **Context Management**
   - Implement conversation history
   - Add user preference consideration
   - Improve context relevance scoring

## Setup Guide

1. **Prerequisites**
   - Supabase project with pgvector enabled
   - OpenAI API key
   - TypeScript and Edge Function development environment

2. **Installation Steps**
   ```bash
   # Install dependencies
   npm install @supabase/supabase-js openai

   # Deploy Edge Functions
   supabase functions deploy generate-embeddings
   supabase functions deploy bot-messages
   ```

3. **Environment Configuration**
   ```bash
   # Set up Edge Function secrets
   supabase secrets set OPENAI_API_KEY=your_key
   supabase secrets set SUPABASE_URL=your_url
   supabase secrets set SUPABASE_SERVICE_ROLE_KEY=your_key
   ```

## Security Considerations
- Secure API key management in Edge Functions
- RLS policies for AI feature access
- Rate limiting on Edge Functions
- Input validation and sanitization
- Error handling and logging

## Monitoring
- Edge Function execution metrics
- Token usage tracking
- Error rate monitoring
- Response time tracking
- User feedback collection

## Resources
- [Supabase Edge Functions Documentation](https://supabase.com/docs/guides/functions)
- [pgvector Documentation](https://github.com/pgvector/pgvector)
- [OpenAI API Documentation](https://platform.openai.com/docs/api-reference)
- [TypeScript Documentation](https://www.typescriptlang.org/docs/) 