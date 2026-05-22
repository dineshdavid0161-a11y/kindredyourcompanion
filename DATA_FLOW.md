# Kindred - Data Flow & Architecture

## Authentication Flow

```
User visits app
├─ Check existing session
│  ├─ Session exists → Load profile → Redirect to dashboard
│  └─ No session → Show landing/login
└─ User selects user type → Choose auth method → Authenticate
```

### Login Paths
1. **Phone OTP** (Elders & Companions)
   - User enters phone → Supabase sends OTP
   - Verify OTP → Create auth session
   - Redirect to registration page

2. **Email + Password** (All users, especially admin)
   - User enters credentials
   - Supabase verifies
   - Create auth session

## User Registration Flow

### Elder Registration
```
Elder Login Complete
└─ /register page (5 steps)
   ├─ Step 1: About you
   │  └─ Name, relationship, mobile, location
   ├─ Step 2: About the elder
   │  └─ Name, age, area, address, health notes
   ├─ Step 3: Preferences
   │  └─ Interests, gender preference, age preference, avoid topics
   ├─ Step 4: Session needs
   │  └─ Session types, frequency, days, time
   ├─ Step 5: Emergency contact
   │  └─ Name, mobile
   └─ Submit → Create profiles + elders table entries
      └─ Redirect to /elder/dashboard
```

**Database Operations:**
1. Insert into `profiles` table
   ```sql
   INSERT INTO profiles (user_id, user_type, full_name, mobile)
   VALUES (auth_uid, 'elder', ...)
   ```

2. Insert into `elders` table
   ```sql
   INSERT INTO elders (user_id, elder_name, elder_age, ...)
   VALUES (auth_uid, ...)
   ```

### Companion Registration
```
Companion Login Complete
└─ /apply page (3 steps)
   ├─ Step 1: Basic details
   │  └─ Name, age, gender, area, occupation
   ├─ Step 2: About you
   │  └─ Languages, vehicle, bio, availability, sessions/week
   ├─ Step 3: Verification
   │  └─ Upload Aadhaar (front/back), selfie, reference person
   └─ Submit → Create profiles + companions table
      ├─ Upload documents to storage
      ├─ Create profiles entry
      ├─ Create companions entry (status: pending)
      └─ Redirect to /companion/dashboard
```

**Database Operations:**
1. Upload documents to storage
   ```
   /documents/companions/{user_id}/aadhaar_front
   /documents/companions/{user_id}/aadhaar_back
   /documents/companions/{user_id}/selfie
   ```

2. Insert into `profiles` table

3. Insert into `companions` table with status='pending'

## Elder Dashboard Flow

```
/elder/dashboard
├─ Load elder data from elders table
├─ Display:
│  ├─ Welcome message with name
│  ├─ Quick stats:
│  │  ├─ Upcoming sessions count
│  │  ├─ Companion status
│  │  └─ Total sessions completed
│  ├─ SOS button → Call emergency contact
│  ├─ "Book a Session" button → /booking
│  ├─ Elder information display
│  └─ Session preferences
└─ User can:
   ├─ View profile
   ├─ Edit profile (future enhancement)
   └─ Logout
```

**Query Pattern:**
```typescript
const { data: elder } = await supabase
  .from('elders')
  .select('*')
  .eq('user_id', userId)
  .maybeSingle();
```

## Booking Flow

```
/booking page
├─ Load elder data
├─ User selects:
│  ├─ Session type (Company visit / Assist outing / Hospital / Video call)
│  ├─ Date (date picker, min tomorrow)
│  ├─ Time (morning/afternoon/evening)
│  ├─ Duration (hours based on session type)
│  └─ Special instructions (optional)
├─ Display:
│  ├─ Price calculation
│  └─ Booking summary
└─ "Proceed to Payment" → Razorpay integration
   ├─ On success:
   │  ├─ Insert booking into bookings table
   │  ├─ Update payment_status = 'completed'
   │  └─ Notify companion
   └─ On failure:
      └─ Show error message
```

**Booking Table Entry:**
```typescript
await supabase.from('bookings').insert({
  elder_id: elderId,
  companion_id: companionId,
  session_type: 'Company visit',
  date: '2024-06-15',
  time_slot: 'morning',
  duration_hours: 2,
  price: 400,
  payment_status: 'completed',
  razorpay_payment_id: 'pay_xxx',
  status: 'confirmed'
});
```

## Companion Dashboard Flow

```
/companion/dashboard
├─ Load companion data from companions table
├─ Check status:
│  ├─ status = 'pending'
│  │  └─ Show "Application under review"
│  ├─ status = 'verified'
│  │  └─ Show "Training notification" with link to /companion/training
│  └─ status = 'active'
│     └─ Show sessions, earnings, availability
├─ Display:
│  ├─ Progress tracker (4 steps)
│  ├─ Quick stats:
│  │  ├─ Total sessions
│  │  ├─ Rating
│  │  ├─ Current status
│  │  └─ This month earnings
│  └─ Companion information
└─ User can:
   ├─ View/edit profile
   ├─ Access training (if verified)
   ├─ View upcoming sessions (if active)
   └─ Logout
```

## Training Flow

```
/companion/training
├─ Load companion progress
├─ Display 4 modules:
│  ├─ Module 1: Understanding loneliness
│  ├─ Module 2: Active listening
│  ├─ Module 3: Safety & protocols
│  └─ Module 4: Health observation
├─ For each module:
│  ├─ Show completion status (checkbox)
│  ├─ Show duration
│  ├─ On click → Show module details in modal
│  └─ "Mark as Complete" → Update companions table
└─ After all 4 complete:
   ├─ Show success banner
   ├─ Admin notified
   └─ Status changes to 'active'
```

**Training Update:**
```typescript
await supabase.from('companions').update({
  training_1: true,
  training_2: true,
  training_3: true,
  training_4: true
}).eq('user_id', userId);
```

## Admin Dashboard Flow

```
/admin
├─ Load stats:
│  ├─ Count elders
│  ├─ Count active companions
│  ├─ Count bookings this month
│  └─ Sum revenue from completed bookings
├─ Display:
│  ├─ Overview cards (4 metrics)
│  ├─ Quick action buttons:
│  │  ├─ Verify companion documents
│  │  ├─ Match elder with companion
│  │  ├─ Review pending applications
│  │  └─ Process payouts
│  └─ Management sections (expandable future)
└─ Admin can:
   ├─ Access companion applications
   ├─ Approve/reject companions (updates status)
   ├─ Match companions to elders (updates matched_companion_user_id)
   ├─ Process payouts
   └─ View all data
```

## Data Access Patterns

### Row Level Security (RLS)

**Elders can see:**
- Their own elder profile
- Their own matched companion (if any)
- Their own bookings
- Their own session notes

**Companions can see:**
- Their own companion profile
- Their own bookings
- Their own session notes
- Matched elder details (if matched)

**Admin can see:**
- All elders
- All companions
- All bookings
- All session notes
- All profiles

### Query Examples

**Get current user's profile:**
```typescript
const { data } = await supabase
  .from('profiles')
  .select('*')
  .eq('user_id', currentUserId)
  .maybeSingle();
```

**Get elder's bookings:**
```typescript
const { data } = await supabase
  .from('bookings')
  .select(`*,
    companion:companions(full_name, rating)
  `)
  .eq('elder_id', elderId)
  .order('date', { ascending: false });
```

**Get companion's upcoming sessions:**
```typescript
const { data } = await supabase
  .from('bookings')
  .select(`*,
    elder:elders(elder_name, address, health_notes)
  `)
  .eq('companion_id', companionId)
  .gte('date', today)
  .order('date', { ascending: true });
```

## Session Lifecycle

```
Booking Created (status: confirmed, payment_status: pending)
├─ Elder & companion notified
│
Razorpay Payment Processed
├─ payment_status → completed
├─ Confirmation sent to both
│
Session Day Arrives
├─ Companion checks in (GPS)
├─ Companion checks out (GPS)
│
Companion Submits Notes
├─ Insert into session_notes table
├─ Notes include:
│  ├─ Elder mood
│  ├─ Note for family
│  ├─ Health observations
│  └─ Timestamp
│
booking.status → completed
├─ Companion can request rating
└─ Family sees session notes
```

## Notifications (Future Implementation)

### Email Notifications
- Registration confirmation
- Booking confirmations
- Session reminders (24h before)
- Companion verified notification
- Payout processed confirmation

### WhatsApp Notifications (Optional)
- Session check-in reminder
- Post-session family update
- Emergency contact alert (if SOS pressed)

### In-app Notifications
- Message notifications
- Session reminders
- New booking alerts

## Error Handling Patterns

```typescript
try {
  // Database operation
  const { data, error } = await supabase
    .from('table')
    .select('*');
    
  if (error) throw error;
  
  // Process data
  return data;
} catch (error) {
  console.error('Operation failed:', error);
  // Show user-friendly error message
  return null;
}
```

## Performance Optimizations

1. **Lazy Loading**
   - Load user profile only when needed
   - Load bookings on demand

2. **Caching**
   - Use browser localStorage for session data
   - React query for data management (future)

3. **Database Indexing**
   - Index on user_id for all user-specific queries
   - Index on date for booking queries

4. **Images**
   - Compress document uploads
   - Use CDN for profile images

## Security Considerations

1. **Data Isolation**
   - RLS ensures users can only see their data
   - Admin role required for admin operations

2. **Document Security**
   - Files stored in private bucket
   - Only accessible to authenticated users via RLS

3. **Payment Security**
   - Never handle payment directly (use Razorpay)
   - Store only payment ID, not card details

4. **Session Management**
   - Auto logout after 30 minutes of inactivity (future)
   - Secure session tokens

---

**Data Flow Visualization:**
```
Landing Page ─→ Login ─→ Registration ─→ Dashboard ─→ Actions
              (Auth)                     (User-specific)
                ↓
           Supabase Auth
                ↓
          RLS Enforcement
                ↓
        User Data Access
```
