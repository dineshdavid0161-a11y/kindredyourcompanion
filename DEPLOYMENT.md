# Kindred - Deployment Guide

## Pre-Deployment Checklist

- [x] Database schema created with RLS policies
- [x] Authentication configured (Supabase Auth)
- [x] All pages built and tested
- [x] Environment variables configured
- [x] Build succeeds with no errors
- [ ] Set up custom domain
- [ ] Configure email notifications
- [ ] Set up Razorpay integration
- [ ] Configure WhatsApp notifications (optional)

## Environment Variables

### Development (`.env.local`)
```
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_ANON_KEY=your-anon-key
```

### Production
Set the same variables in your hosting provider's dashboard.

## Build & Deploy to Vercel (Recommended)

### Step 1: Push to GitHub
```bash
git init
git add .
git commit -m "Initial Kindred commit"
git branch -M main
git remote add origin https://github.com/yourusername/kindred.git
git push -u origin main
```

### Step 2: Connect to Vercel
1. Go to https://vercel.com/new
2. Import your GitHub repository
3. Select React/Vite as framework
4. Configure environment variables
5. Deploy

### Step 3: Add Custom Domain
1. In Vercel dashboard, go to Settings → Domains
2. Add your domain (e.g., kindred.co)
3. Follow DNS configuration instructions
4. Wait for SSL certificate (auto-provisioned)

## Deploy to Netlify

### Step 1: Connect Repository
1. Go to https://app.netlify.com
2. Click "New site from Git"
3. Choose GitHub
4. Select your repository

### Step 2: Configure Build
- Build command: `npm run build`
- Publish directory: `dist`

### Step 3: Set Environment Variables
1. Go to Site settings → Build & deploy → Environment
2. Add variables:
   - `VITE_SUPABASE_URL`
   - `VITE_SUPABASE_ANON_KEY`

### Step 4: Deploy
Netlify auto-deploys on push to main branch

## Self-Hosted Deployment

### Option 1: Docker

Create `Dockerfile`:
```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .
RUN npm run build

FROM node:18-alpine
WORKDIR /app
RUN npm install -g serve
COPY --from=0 /app/dist ./dist

EXPOSE 3000
CMD ["serve", "-s", "dist", "-l", "3000"]
```

Build and run:
```bash
docker build -t kindred .
docker run -p 3000:3000 -e VITE_SUPABASE_URL=... -e VITE_SUPABASE_ANON_KEY=... kindred
```

### Option 2: Traditional Server (Ubuntu)

```bash
# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Clone repository
git clone https://github.com/yourusername/kindred.git
cd kindred

# Install dependencies
npm install

# Build
npm run build

# Install serve (static server)
sudo npm install -g serve

# Run
serve -s dist -l 3000
```

Set up systemd service:
```bash
sudo nano /etc/systemd/system/kindred.service
```

Add:
```ini
[Unit]
Description=Kindred Service
After=network.target

[Service]
Type=simple
User=ubuntu
WorkingDirectory=/home/ubuntu/kindred
ExecStart=/usr/bin/serve -s dist -l 3000
Restart=always

[Install]
WantedBy=multi-user.target
```

Start service:
```bash
sudo systemctl enable kindred
sudo systemctl start kindred
```

## Database Setup

### 1. Create Supabase Project
- Go to https://supabase.com
- Click "New Project"
- Select region (closest to your users)
- Choose authentication method

### 2. Get Credentials
- Copy Project URL
- Copy Anon Key
- Add to environment variables

### 3. Run Migrations
All migrations have been created in the project:
- `01_create_profiles_table`
- `02_create_elders_table`
- `03_create_companions_table`
- `04_create_bookings_and_notes`
- `05_create_payouts_waitlist`
- `06_setup_storage_bucket`

They run automatically in Supabase when deployed.

### 4. Configure Storage Bucket
1. Go to Supabase Dashboard → Storage
2. Create bucket: `documents` (private)
3. Add RLS policies for uploads

## SSL Certificate Setup

### Let's Encrypt (Free)
```bash
sudo apt install certbot python3-certbot-nginx

sudo certbot certonly --nginx -d kindred.com -d www.kindred.com

# Auto-renewal
sudo certbot renew --dry-run
```

### Vercel/Netlify
SSL is automatically provisioned for custom domains.

## Domain Setup

### Namecheap / GoDaddy
1. Buy domain
2. Go to DNS settings
3. Add CNAME record:
   - If Vercel: `vercel.com`
   - If Netlify: Your Netlify domain
4. Wait for DNS propagation (up to 48 hours)

## Monitoring & Maintenance

### Uptime Monitoring
```bash
# Using UptimeRobot (free)
1. Go to https://uptimerobot.com
2. Add monitor for your domain
3. Set alerts to email
```

### Error Tracking
```bash
# Using Sentry (free tier available)
1. Go to https://sentry.io
2. Create project for React
3. Add Sentry to React app
4. Track errors automatically
```

### Performance Monitoring
```bash
# Vercel Analytics (built-in)
# Netlify Analytics (in site settings)
# Or Google Analytics
```

## Scaling Considerations

### Database
- Supabase auto-scales with usage
- Monitor query performance in dashboard
- Add indexes if needed

### File Storage
- Supabase provides 1GB free storage
- Upgrade plan for more space
- Consider CDN for frequent downloads

### Concurrent Users
- Supabase handles concurrent connections
- Each plan has limits
- Scale plan as needed

## Backup Strategy

### Automated Backups
Supabase includes backups:
- Daily backups (free tier, 7 days retention)
- Weekly backups (pro tier, 30 days)

### Manual Backups
```bash
# Export database via Supabase console
# Or use pg_dump for PostgreSQL
pg_dump --host supabase-host --username postgres dbname > backup.sql
```

## Rollback Procedure

### If Deployment Fails
1. **Vercel**: Automatic rollback to previous version
   - Go to Deployments → Select previous
   - Click "Redeploy"

2. **Netlify**: Same process
   - Go to Deploys → Select previous
   - Click "Publish"

3. **Self-hosted**: 
   ```bash
   git revert <commit-hash>
   npm run build
   systemctl restart kindred
   ```

## Post-Deployment

### 1. Verify Everything Works
- Visit landing page
- Test elder registration flow
- Test companion application flow
- Verify authentication
- Check admin dashboard
- Test booking flow

### 2. Set Up Email Notifications
Create Supabase Edge Functions for:
- Welcome emails
- Booking confirmations
- Reminder emails

### 3. Configure Razorpay
1. Get test keys from Razorpay dashboard
2. Add to environment as `VITE_RAZORPAY_KEY_ID`
3. Update BookingPage.tsx with actual integration
4. Test payment flow
5. Switch to live keys when ready

### 4. Add WhatsApp Notifications (Optional)
1. Set up Twilio account
2. Create Edge Function for WhatsApp messages
3. Send notifications on key events

### 5. Analytics & Monitoring
- Set up Google Analytics
- Configure error tracking
- Set up uptime monitoring
- Monitor database performance

## Troubleshooting Deployment Issues

### Build Fails
```bash
# Clear cache and reinstall
rm -rf node_modules package-lock.json
npm install
npm run build
```

### Environment Variables Not Working
- Verify variable names match exactly
- Check for typos
- Redeploy after adding variables
- Hard refresh browser (Cmd+Shift+R)

### Database Connection Issues
- Verify Supabase URL is correct
- Check anon key is valid
- Test connection in Supabase console
- Check RLS policies are set correctly

### Page Shows Blank
- Check browser console for errors
- Verify environment variables are set
- Clear browser cache
- Check network tab for failed requests

## Performance Optimization

### Frontend
- Static files cached by CDN
- Code splitting automatic with Vite
- Images optimized via Tailwind

### Backend
- Database queries indexed
- RLS policies optimized
- Storage bucket for files instead of database

### Monitoring
- Vercel/Netlify provide performance metrics
- Use Web Vitals for real-world metrics

## Scaling Timeline

### Phase 1 (Launch)
- Single region deployment
- Free/starter database plan
- Basic monitoring

### Phase 2 (100+ users)
- Add uptime monitoring
- Upgrade database plan
- Set up CDN

### Phase 3 (1000+ users)
- Multi-region deployment
- Advanced caching
- Database optimization
- Dedicated database instance

### Phase 4 (10000+ users)
- Load balancing
- Microservices (if needed)
- Dedicated infrastructure

## Security Checklist

- [x] HTTPS/SSL enabled
- [x] RLS policies configured
- [x] Authentication enabled
- [ ] Rate limiting configured
- [ ] CORS properly set
- [ ] Secrets not in code
- [ ] Database backups enabled
- [ ] Admin password strong
- [ ] 2FA enabled for critical accounts

## Support & Documentation

### Useful Links
- Supabase Docs: https://supabase.com/docs
- Vercel Docs: https://vercel.com/docs
- React Docs: https://react.dev
- Tailwind Docs: https://tailwindcss.com/docs

### Getting Help
- Check error messages first
- Search issues in GitHub
- Review documentation
- Contact support team

---

**Next Steps:**
1. Choose deployment platform
2. Configure environment variables
3. Deploy to staging first
4. Test all flows
5. Deploy to production
5. Monitor for issues
6. Implement additional features

**Go live in 1-2 hours!**
