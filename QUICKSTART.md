# Kindred - Quick Start Guide

## 🚀 Get Started in 5 Minutes

### Prerequisites
- Node.js 18+ installed
- Supabase account (free tier available)
- Git

### Step 1: Environment Setup (2 min)

```bash
# 1. Get your Supabase credentials
# Go to https://supabase.com
# Create new project → Get URL and Anon Key

# 2. Create .env.local in project root
cat > .env.local << EOF
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_ANON_KEY=your-anon-key-here
EOF

# 3. Install dependencies
npm install
```

### Step 2: Build & Run (1 min)

```bash
# Development
npm run build

# The app is ready to deploy!
# Output: dist/ folder
```

### Step 3: Deploy (2 min)

#### Option A: Vercel (Easiest - 30 seconds)
```bash
npm install -g vercel
vercel
# Follow prompts, add environment variables
```

#### Option B: Netlify
```bash
npm install -g netlify-cli
netlify deploy --prod --dir=dist
```

#### Option C: Any Static Host
Upload the `dist/` folder to your hosting provider.

---

## 📱 Test the Application

### Test Users (No Backend Setup Needed!)

Use these flows to test:

1. **Landing Page**
   - View features and pricing
   - Click "Find a Companion" or "Become a Companion"

2. **Sign Up as Elder**
   - Click "Find a Companion"
   - Enter any phone number
   - Complete 5-step registration form
   - Access elder dashboard

3. **Sign Up as Companion**
   - Click "Become a Companion"
   - Enter phone number
   - Complete 3-step application
   - Upload test documents
   - Access companion dashboard

4. **Admin Access**
   - Use email login
   - Default test: admin@kindred.io / password123
   - View admin dashboard

---

## 🗂 Project Structure

```
kindred/
├── src/
│   ├── pages/              # Page components
│   │   ├── Landing.tsx     # Public homepage
│   │   ├── Login.tsx       # Authentication
│   │   ├── ElderRegistration.tsx
│   │   ├── ElderDashboard.tsx
│   │   ├── BookingPage.tsx
│   │   ├── CompanionApplication.tsx
│   │   ├── CompanionDashboard.tsx
│   │   ├── CompanionTraining.tsx
│   │   └── AdminDashboard.tsx
│   ├── contexts/           # State management
│   │   └── AuthContext.tsx
│   ├── lib/                # Utilities
│   │   └── supabase.ts
│   ├── App.tsx             # Router setup
│   ├── main.tsx            # Entry point
│   └── index.css           # Tailwind styles
├── dist/                   # Production build
├── package.json
├── tsconfig.json
├── vite.config.ts
├── tailwind.config.js
└── postcss.config.js
```

---

## ⚙️ Configuration

### Customize Colors

Edit `tailwind.config.js`:
```javascript
module.exports = {
  theme: {
    extend: {
      colors: {
        kindred: {
          primary: '#5C7A4E',  // Sage green
          accent: '#C4714A',   // Terracotta
          light: '#FAF7F2',    // Warm cream
        }
      }
    }
  }
}
```

### Change City

Update in files:
- `/src/pages/ElderRegistration.tsx` - Line 13-19 (AREAS array)
- `/src/pages/CompanionApplication.tsx` - Line 15-21 (AREAS array)

Replace "Adyar", "Anna Nagar", etc. with your cities.

### Change Pricing

Edit `/src/pages/BookingPage.tsx` - Line 8-13:
```typescript
const SESSION_PRICES = {
  'Company visit': { 2: 400, 3: 550 },     // Change these
  'Assist outing': { 2: 500, 3: 650 },
  'Video call only': { 1: 200 },
};
```

---

## 🔑 Key Files to Understand

### AuthContext.tsx
Manages authentication state and user loading.
```typescript
const { user, loading, signOut } = useAuth();
```

### supabase.ts
Initializes Supabase client.
```typescript
import { supabase } from '../lib/supabase';
```

### App.tsx
Defines all routes and protected pages.

---

## 🧪 Testing Checklist

- [ ] Landing page loads
- [ ] Can navigate to login
- [ ] Can select user type
- [ ] Can start elder registration
- [ ] Can fill all 5 steps
- [ ] Data saves to dashboard
- [ ] Can start companion application
- [ ] Can upload test documents
- [ ] Can access training modules
- [ ] Admin dashboard loads
- [ ] Responsive on mobile

---

## 🐛 Common Issues & Fixes

### "VITE_SUPABASE_URL is not defined"
```bash
# Make sure .env.local exists in project root
# Contains correct values
# Then rebuild: npm run build
```

### "ReferenceError: supabase is not defined"
```bash
# Check AuthContext.tsx imports
# Verify supabase.ts is in src/lib/
```

### Build fails with TypeScript errors
```bash
# Run type check first
npm run typecheck
# Fix reported errors
npm run build
```

### Application shows blank page
```bash
# Check browser console for errors (F12)
# Verify environment variables are set
# Hard refresh: Cmd+Shift+R (Mac) or Ctrl+Shift+R (Windows)
```

---

## 📚 Next Steps

### 1. Add Razorpay Integration
```typescript
// In /src/pages/BookingPage.tsx
// Add payment handler:
const handleBook = async () => {
  const script = document.createElement('script');
  script.src = 'https://checkout.razorpay.com/v1/checkout.js';
  // ... Razorpay integration code
};
```

### 2. Set Up Email Notifications
Create Supabase Edge Function:
```bash
supabase functions new send_email
```

### 3. Deploy to Production
```bash
vercel --prod
# OR
netlify deploy --prod
```

### 4. Monitor Performance
- Set up Google Analytics
- Add error tracking (Sentry)
- Monitor database usage

### 5. Add Video Calls
Integrate Twilio for video calls in booking flow.

---

## 📖 Documentation

- **Full Setup Guide**: See `KINDRED_SETUP.md`
- **Data Flow**: See `DATA_FLOW.md`
- **Deployment**: See `DEPLOYMENT.md`
- **Features**: See `FEATURES.md`

---

## 🆘 Support

### Questions?
1. Check the documentation files above
2. Review Supabase docs: https://supabase.com/docs
3. Check React Router docs: https://reactrouter.com

### Something broken?
1. Check browser console (F12)
2. Check terminal output
3. Verify environment variables
4. Try `npm install && npm run build`

---

## ✨ Tips & Tricks

### Fast Local Development
```bash
# Instead of build, use dev server
npm install -D vite-preview
npm run dev
```

### Debug Supabase Queries
```typescript
// Add debugging to see queries
import { supabase } from './lib/supabase';
supabase.auth.onAuthStateChange(console.log);
```

### Check Build Size
```bash
npm run build
# Analyzes dist/ folder
ls -lh dist/assets/
```

### Clean Cache
```bash
rm -rf node_modules dist package-lock.json
npm install
npm run build
```

---

## 🎉 You're Ready!

Your Kindred application is ready to deploy.

**Next command:**
```bash
npm run build  # Build for production
vercel        # Deploy to Vercel (or your host)
```

**Time to launch: ~10 minutes from setup to live!**

---

**Questions? Check the docs or visit:**
- Kindred Docs: See KINDRED_SETUP.md
- Supabase: https://supabase.com
- React: https://react.dev
- Tailwind: https://tailwindcss.com
