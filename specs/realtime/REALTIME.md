# SkyMarket Messaging & Status Updates Documentation

Practical messaging and status update implementation for the SkyMarket hackathon project using simple, reliable patterns.

## Overview

SkyMarket implements messaging and status updates using server-side actions and page revalidation - a simple, robust approach perfect for hackathons and MVP development.

**Current Features**:
- Order/booking messaging between consumers and providers
- Status updates (pending → accepted → in_progress → completed)
- Simple form-based interactions with immediate feedback
- Server-side validation and database updates

**Key Principle**: Keep it simple - no complex WebSocket connections or real-time subscriptions needed for a functional marketplace.

## Architecture

### Simple Server Actions Pattern

SkyMarket uses Next.js Server Actions for reliable, server-side updates with automatic page revalidation.

**Technology Stack**:
- **Next.js Server Actions**: Server-side form handling
- **Supabase**: Database operations with RLS
- **revalidatePath**: Automatic page refresh with new data
- **Sonner**: Client-side toast notifications

### Current Implementation

**Location**: `/app/orders/[id]/page.tsx`

```typescript
// Server Action for sending messages
async function sendMessage(formData: FormData) {
  "use server"
  const supabase = await createClient()
  const { data: { user } } = await supabase.auth.getUser()
  if (!user) redirect("/auth/login")

  const bookingId = String(formData.get("bookingId"))
  const content = String(formData.get("content") || "").trim()
  if (!content) return

  // Fetch booking to determine recipient
  const { data: booking } = await supabase
    .from("bookings")
    .select("id, consumer_id, provider_id")
    .eq("id", bookingId)
    .single()

  if (!booking) return

  const { data: providerData } = await supabase
    .from("providers")
    .select("user_id")
    .eq("id", booking.provider_id)
    .single()

  const recipientId = user.id === booking.consumer_id ? providerData?.user_id : booking.consumer_id

  await supabase
    .from("messages")
    .insert({ booking_id: bookingId, sender_id: user.id, recipient_id: recipientId!, content })

  revalidatePath(`/orders/${bookingId}`) // Refresh the page with new messages
}

// Server Action for status updates
async function updateStatus(formData: FormData) {
  "use server"
  const supabase = await createClient()
  const { data: { user } } = await supabase.auth.getUser()
  if (!user) redirect("/auth/login")

  const id = String(formData.get("id"))
  const next = String(formData.get("next")) as Booking["status"]

  // RLS will enforce ownership (consumer or provider)
  await supabase.from("bookings").update({ status: next }).eq("id", id)

  revalidatePath(`/orders/${id}`) // Refresh with updated status
}
```

## Core Patterns

### 1. Simple Form-Based Messaging

**Basic messaging pattern with server actions**:
```typescript
// Simple message form in order details page
<form action={sendMessage} className="flex gap-2">
  <input type="hidden" name="bookingId" value={booking.id} />
  <Input name="content" placeholder="Type a message" />
  <Button type="submit">Send</Button>
</form>
```

### 2. Status Transition Buttons

**Provider status updates**:
```typescript
// Provider actions based on current status
const nextActions: { label: string; next: Booking["status"] }[] = []
if (isProvider) {
  if (booking.status === "pending") nextActions.push({ label: "Accept", next: "accepted" })
  if (booking.status === "accepted") nextActions.push({ label: "Start", next: "in_progress" })
  if (booking.status === "in_progress") nextActions.push({ label: "Complete", next: "completed" })
}

// Render action buttons
{nextActions.map((a) => (
  <form key={a.next} action={updateStatus} className="inline">
    <input type="hidden" name="id" value={booking.id} />
    <input type="hidden" name="next" value={a.next} />
    <Button>{a.label}</Button>
  </form>
))}
```

### 3. Message Display

**Simple message list with role identification**:
```typescript
// Display messages with sender identification
<div className="space-y-3 max-h-72 overflow-auto mb-4">
  {(messages ?? []).map((m) => (
    <div key={m.id} className="text-sm">
      <div className="text-gray-500">
        {m.sender_id === user.id ? "You" : "Them"} • {m.created_at ? new Date(m.created_at).toLocaleTimeString() : "Unknown time"}
      </div>
      <div>{m.content}</div>
    </div>
  ))}
</div>
```

## Database Schema

### Messages Table

**Core schema for order messaging**:
```sql
CREATE TABLE messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  booking_id UUID NOT NULL REFERENCES bookings(id),
  sender_id UUID NOT NULL REFERENCES profiles(id),
  recipient_id UUID NOT NULL REFERENCES profiles(id),
  content TEXT NOT NULL,
  read BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

### Booking Status Flow

**Simple status enum with clear transitions**:
```sql
CREATE TYPE booking_status AS ENUM (
  'pending',     -- Initial state after booking creation
  'accepted',    -- Provider accepted the job
  'in_progress', -- Service is being performed
  'completed',   -- Service finished successfully
  'cancelled'    -- Cancelled by either party
);
```

### Row Level Security

**Secure access to messages and bookings**:
```sql
-- Messages: only sender and recipient can access
CREATE POLICY "message_access_policy" ON messages
  FOR ALL USING (
    sender_id = auth.uid() OR recipient_id = auth.uid()
  );

-- Bookings: consumer and provider can access
CREATE POLICY "booking_access_policy" ON bookings
  FOR ALL USING (
    consumer_id = auth.uid() OR
    provider_id IN (SELECT id FROM providers WHERE user_id = auth.uid())
  );
```

## Future Enhancement Ideas

*These are potential future features - not currently implemented. Focus on the simple patterns above for hackathon development.*

### 1. Real-time Notifications

*Future enhancement: Could add Supabase Realtime subscriptions*

```typescript
// FUTURE: Real-time message notifications
// This would require additional complexity - stick to simple approach for now
// useEffect(() => {
//   const supabase = createClient()
//   const channel = supabase
//     .channel('messages')
//     .on('postgres_changes', {
//       event: 'INSERT',
//       schema: 'public',
//       table: 'messages',
//       filter: `recipient_id=eq.${user.id}`
//     }, (payload) => {
//       toast.message('New message', { description: payload.new.content })
//     })
//     .subscribe()
//
//   return () => supabase.removeChannel(channel)
// }, [])
```

### 2. Live Status Updates

*Future enhancement: Dashboard with live booking status changes*

```typescript
// FUTURE: Live provider dashboard updates
// Simple approach: Use periodic refresh with setInterval
// Or manual refresh buttons on key pages
```

## Implementation Guide

### Step 1: Set up the messages table

**Run this SQL in Supabase**:
```sql
CREATE TABLE messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  booking_id UUID NOT NULL REFERENCES bookings(id),
  sender_id UUID NOT NULL REFERENCES profiles(id),
  recipient_id UUID NOT NULL REFERENCES profiles(id),
  content TEXT NOT NULL,
  read BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Enable RLS
ALTER TABLE messages ENABLE ROW LEVEL SECURITY;

-- Policy: users can only access their own messages
CREATE POLICY "message_access_policy" ON messages
  FOR ALL USING (
    sender_id = auth.uid() OR recipient_id = auth.uid()
  );
```

### Step 2: Add messaging to order pages

**Update `/app/orders/[id]/page.tsx`**:
```typescript
// Add the server actions shown above
// Add the messaging UI in the order details page

// Simple message display
<div className="space-y-3 max-h-72 overflow-auto mb-4">
  {(messages ?? []).map((m) => (
    <div key={m.id} className="text-sm">
      <div className="text-gray-500">
        {m.sender_id === user.id ? "You" : "Them"} •
        {new Date(m.created_at).toLocaleTimeString()}
      </div>
      <div>{m.content}</div>
    </div>
  ))}
</div>

// Simple message form
<form action={sendMessage} className="flex gap-2">
  <input type="hidden" name="bookingId" value={booking.id} />
  <Input name="content" placeholder="Type a message" />
  <Button type="submit">Send</Button>
</form>
```

### Step 3: Add status update buttons

**Provider status transitions**:
```typescript
// In your order page, add status action buttons
const nextActions = []
if (isProvider) {
  if (booking.status === "pending") nextActions.push({ label: "Accept", next: "accepted" })
  if (booking.status === "accepted") nextActions.push({ label: "Start", next: "in_progress" })
  if (booking.status === "in_progress") nextActions.push({ label: "Complete", next: "completed" })
}

{nextActions.map((a) => (
  <form key={a.next} action={updateStatus}>
    <input type="hidden" name="id" value={booking.id} />
    <input type="hidden" name="next" value={a.next} />
    <Button>{a.label}</Button>
  </form>
))}
```

### Why This Simple Approach Works

**Advantages**:
- ✅ **Reliable**: Server actions are robust and handle errors well
- ✅ **Simple**: No complex WebSocket state management
- ✅ **SEO-friendly**: Server-rendered content works with search engines
- ✅ **Low complexity**: Perfect for hackathons and MVP development
- ✅ **Progressive enhancement**: Works without JavaScript

**When to consider real-time features**:
- When you need instant notifications (beyond form submissions)
- When you have live data that changes frequently (like GPS tracking)
- When you need to support many concurrent users with live updates

For a marketplace MVP, the simple form-based approach handles 90% of communication needs effectively.