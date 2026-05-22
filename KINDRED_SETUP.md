# Kindred - Setup & Deployment Guide

## Overview

Kindred is a full-stack elder companionship platform built with:
- **Frontend**: React + TypeScript + Tailwind CSS + Vite
- **Backend**: Supabase (Database, Auth, Storage)
- **Payments**: Ready for Razorpay integration
- **Hosting**: Deploy-ready

## Features Implemented

### Landing Page
- Beautiful hero section with warm, trustworthy messaging
- How it works (4-step process)
- Session types pricing breakdown
- Companion types overview
- Safety & trust badges
- Footer with city expansion info

### Authentication
- Phone OTP login (via Supabase Auth)
- Email + password fallback
- User type selection (Elder / Companion / Admin)
- Profile management

### Elder Flow
1. **Registration (5-step form)**
   - Step 1: About you (name, relationship, mobile)
   - Step 2: About the elder (name, age, area, address, health)
   - Step 3: What they enjoy (interests, gender preference, age preference)
   - Step 4: What you need (session types, frequency, days, time)
   - Step 5: Emergency contact

2. **Dashboard**
   - Quick stats (upcoming sessions, companion status, session count)
   - SOS button for emergency contact
   - Book a Session button
   - View my Companion button
   - Elder information display
   - Session preferences view

3. **Booking Page**
   - Session type selector (Company visit / Assist outing / Hospital accompaniment / Video call)
   - Date & time picker
   - Duration selector (1-3 hours)
   - Special instructions
   - Price calculation
   - Payment summary
   - Razorpay integration ready

### Companion Flow
1. **Application (3-step form)**
   - Step 1: Basic details (name, age, gender, area, occupation)
   - Step 2: About you (languages, vehicle, bio, availability, sessions per week)
   - Step 3: Verification (Aadhaar front/back, selfie, reference person, consent)

2. **Dashboard**
   - Progress tracker (4-step verification process)
   - Quick stats (total sessions, rating, status, earnings)
   - Companion information
   - Training notification (when documents verified)
   - Link to training modules

3. **Training Modules**
   - Module 1: Understanding loneliness in elders (20 min)
   - Module 2: Active listening & conversation skills (25 min)
   - Module 3: Safety, boundaries & home visit protocol (30 min)
   - Module 4: Health observation & family reporting (20 min)
   - Progress tracking
   - Module completion markers

### Admin Dashboard
- Overview stats (elders, companions, sessions, revenue)
- Companion applications management
- Elder registrations management
- Bookings & sessions tracking
- Payout processing
- Quick action buttons for common admin tasks

## Database Schema

### Tables
- `profiles` - User profiles (elder, companion, admin)
- `elders` - Elder registration data
- `companions` - Companion application data
- `bookings` - Session bookings with payment tracking
- `session_notes` - Post-session notes from companions
- `payouts` - Companion payout requests
- `waitlist` - City expansion waitlist

### Security
- Row Level Security (RLS) enabled on all tables
- Restrictive policies ensuring users can only access their own data
- Admin-only access to management functions
- Companion verification workflow

## Environment Setup

### Required Environment Variables
```
VITE_SUPABASE_URL=your_supabase_url
VITE_SUPABASE_ANON_KEY=your_supabase_anon_key
```

### Optional (for Razorpay)
```
VITE_RAZORPAY_KEY_ID=your_razorpay_key
```

## File Structure
```
src/
  ├── contexts/
  │   └── AuthContext.tsx         # Auth state management
  ├── lib/
  │   └── supabase.ts             # Supabase client setup
  ├── pages/
  │   ├── Landing.tsx             # Public landing page
  │   ├── Login.tsx               # Auth login
  │   ├── ElderRegistration.tsx   # Elder 5-step registration
  │   ├── CompanionApplication.tsx # Companion 3-step application
  │   ├── ElderDashboard.tsx      # Elder main dashboard
  │   ├── BookingPage.tsx         # Session booking interface
  │   ├── CompanionDashboard.tsx  # Companion main dashboard
  │   ├── CompanionTraining.tsx   # Training modules
  │   └── AdminDashboard.tsx      # Admin management dashboard
  ├── App.tsx                      # Router setup
  ├── main.tsx                     # React entry point
  └── index.css                    # Tailwind styles
```

## Key Design Decisions

### Color Scheme
- **Primary**: Sage Green (#5C7A4E) - Trust, growth, care
- **Accent**: Terracotta (#C4714A) - Warmth, connection
- **Background**: Warm Cream (#FAF7F2) - Comfort, home-like

### UX Patterns
- Progress bars for multi-step forms
- Warm, friendly success/error messages
- Large text and touch targets for elder accessibility
- Mobile-first responsive design
- Clear visual hierarchy with generous spacing

### Security
- All user data protected by RLS policies
- Document uploads stored securely
- Verification workflows before companion activation
- Separate admin access controls

## Next Steps for Deployment

### 1. Razorpay Integration
Update `/src/pages/BookingPage.tsx` with:
```typescript
// Add Razorpay checkout integration
const handleBook = async () => {
  const response = await fetch('/api/create-razorpay-order', {
    method: 'POST',
    body: JSON.stringify(bookingData)
  });
  // Handle Razorpay checkout
};
```

### 2. Email Notifications
Create Supabase Edge Functions for:
- Elder registration confirmation
- Booking confirmation
- Session reminders
- Companion application status

### 3. WhatsApp Integration (Optional)
- Session check-in notifications
- Family updates after sessions
- Emergency contact notifications

### 4. Database Backups
- Set up automated Supabase backups
- Test restoration procedures

### 5. Domain Setup
- Configure custom domain (e.g., kindred.com)
- SSL certificate setup
- Email domain verification

## Testing Checklist

### Authentication
- [ ] Phone OTP login flow
- [ ] Email login fallback
- [ ] User type selection
- [ ] Profile creation and loading

### Elder Flow
- [ ] Complete registration form (all 5 steps)
- [ ] View dashboard after registration
- [ ] Book a session (date/time selection)
- [ ] View booking summary

### Companion Flow
- [ ] Complete application form (all 3 steps)
- [ ] Document upload and validation
- [ ] View dashboard after application
- [ ] Access training modules
- [ ] Mark training modules complete

### Admin
- [ ] View all stats and overview
- [ ] See pending applications
- [ ] View elder and companion data
- [ ] Access management tools

## Production Deployment

### Build
```bash
npm run build
```

Output: `dist/` folder ready for deployment

### Deploy Options
1. **Vercel** (Recommended for React/Vite)
   - Push to GitHub
   - Connect repository to Vercel
   - Auto-deploys on push

2. **Netlify**
   - Build: `npm run build`
   - Publish: `dist/`

3. **Self-hosted**
   - Build project
   - Serve `dist/` with any static server
   - Ensure environment variables are set

### Environment Variables (Production)
Set these in your hosting provider's dashboard:
- `VITE_SUPABASE_URL`
- `VITE_SUPABASE_ANON_KEY`

## Support & Troubleshooting

### Common Issues

**"Not authenticated" errors**
- Check Supabase credentials
- Verify RLS policies are set correctly
- Check auth session in browser console

**Build failures**
- Run `npm install` to ensure dependencies are installed
- Check for TypeScript errors: `npm run typecheck`

**Database connectivity**
- Verify Supabase project is active
- Check database URL in environment variables
- Test connection in Supabase console

## Future Enhancements

1. **Video Call Integration** (Twilio/WebRTC)
2. **Advanced Matching Algorithm**
3. **In-app Messaging**
4. **Rating & Reviews System**
5. **Analytics Dashboard**
6. **Mobile App** (React Native)
7. **Multi-language Support**
8. **AI-powered Recommendations**

## Support

For questions or issues:
- Check the troubleshooting section above
- Review Supabase documentation: https://supabase.com/docs
- Contact development team

---

**Kindred** - Warm, trustworthy elder companionship platform
Built with care for India's elderly community.
