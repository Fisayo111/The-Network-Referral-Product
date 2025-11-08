
# System Architecture - Community Referral Network

## Table of Contents
1. [Overview](#overview)
2. [Technology Stack](#technology-stack)
3. [System Components](#system-components)
4. [Data Flow](#data-flow)
5. [Database Schema](#database-schema)
6. [API Endpoints](#api-endpoints)
7. [Security Considerations](#security-considerations)
8. [Scalability Plan](#scalability-plan)

---

## Overview

Community Referral Network is a web-based platform that digitizes the informal service worker referral system found in Nigerian estate WhatsApp groups. The architecture prioritizes trust verification, community context, and relationship continuity over generic marketplace mechanics.

### Architecture Principles
- **Community-First Design**: All features reinforce neighborhood trust networks
- **Mobile-Responsive**: Primary access via mobile devices
- **Offline Capability**: Core features accessible with poor connectivity
- **Data Integrity**: References immutable once submitted

---

## Technology Stack

### Frontend
**Framework**: React.js (v18+)

**Why React?**
- Component reusability (worker profiles, reference cards)
- Large ecosystem for mobile responsiveness
- Virtual DOM for fast rendering on low-end devices
- React Native path for future mobile apps

**Key Libraries**:
- **React Router**: Client-side routing
- **Axios**: HTTP requests to backend
- **Tailwind CSS**: Utility-first styling for rapid UI development
- **React Hook Form**: Form validation and management
- **React Query**: Data fetching and caching

**State Management**: Context API + React Query (avoids Redux complexity)

---

### Backend
**Framework**: Node.js with Express.js

**Why Node.js?**
- JavaScript across full stack (easier developer onboarding)
- Non-blocking I/O handles concurrent users efficiently
- npm ecosystem for rapid development
- JSON-native (matches MongoDB)

**Key Dependencies**:
- **Express.js**: Web server framework
- **Mongoose**: MongoDB object modeling
- **bcrypt**: Password hashing
- **jsonwebtoken**: JWT authentication
- **express-validator**: Input validation
- **multer**: File upload handling (for before/after photos)
- **node-cron**: Scheduled tasks (notification reminders)

---

### Database
**Primary Database**: MongoDB (NoSQL)

**Why MongoDB?**
- Flexible schema for evolving reference structures
- Handles nested data (references within worker profiles)
- Horizontal scalability for growing user base
- JSON-like documents match JavaScript objects

**Database Structure**:
```
Collections:
- users (service seekers + validators)
- workers (service providers)
- communities (estates/neighborhoods)
- references (vouches from neighbors)
- bookings (service requests)
```

---

### Authentication
**Method**: JWT (JSON Web Tokens)

**Flow**:
1. User logs in with credentials
2. Server validates and issues JWT
3. Client stores JWT in httpOnly cookie
4. JWT included in subsequent requests
5. Server verifies JWT on protected routes

**Security**:
- Passwords hashed with bcrypt (12 salt rounds)
- JWT expires after 7 days (refresh token for 30 days)
- Rate limiting on login attempts (5 max per 15 minutes)

---

### Payment Integration
**Provider**: Paystack

**Why Paystack?**
- Nigerian market leader
- Supports local payment methods (cards, bank transfers, USSD)
- Webhook support for real-time updates
- Split payment capability (platform fee + worker payment)

**Payment Flow**:
1. User requests booking
2. Paystack checkout initiated
3. User completes payment
4. Webhook confirms payment
5. Booking status updated
6. Worker notified

---

### File Storage
**Provider**: Cloudinary

**Why Cloudinary?**
- Free tier sufficient for MVP
- Automatic image optimization
- CDN delivery for fast loading
- Supports transformations (thumbnails, compression)

**Use Cases**:
- Worker before/after photos
- User profile pictures
- Community verification documents

---

### Hosting

**Frontend**: Vercel
- Free tier for MVP
- Automatic deployments from Git
- Global CDN
- Custom domains

**Backend**: Railway
- Free tier with $5 monthly credit
- PostgreSQL option for future migration
- Easy environment variable management
- Auto-scaling

**Database**: MongoDB Atlas
- Free M0 cluster (512MB storage)
- Automatic backups
- Built-in monitoring

---

## System Components

### 1. User Interface Layer (React Frontend)

**Components**:
```
/src
  /components
    /auth
      - LoginForm.jsx
      - SignupForm.jsx
      - CommunityVerification.jsx
    /worker
      - WorkerProfile.jsx
      - WorkerCard.jsx
      - WorkerList.jsx
    /reference
      - ReferenceForm.jsx
      - ReferenceCard.jsx
      - ReferenceList.jsx
    /booking
      - BookingForm.jsx
      - BookingHistory.jsx
    /community
      - CommunitySelector.jsx
      - CommunityFeed.jsx
  /pages
    - Home.jsx
    - Search.jsx
    - Profile.jsx
    - Dashboard.jsx
  /hooks
    - useAuth.js
    - useWorkers.js
    - useReferences.js
  /services
    - api.js (Axios configuration)
  /utils
    - validators.js
    - formatters.js
```

---

### 2. Application Server Layer (Node.js/Express Backend)

**Structure**:
```
/server
  /routes
    - auth.js (login, signup, verify)
    - workers.js (worker CRUD, search)
    - references.js (create, read references)
    - bookings.js (create, manage bookings)
    - communities.js (list, verify communities)
  /controllers
    - authController.js
    - workerController.js
    - referenceController.js
    - bookingController.js
  /models
    - User.js
    - Worker.js
    - Reference.js
    - Booking.js
    - Community.js
  /middleware
    - authMiddleware.js (JWT verification)
    - validationMiddleware.js
    - errorHandler.js
  /utils
    - emailService.js (notifications)
    - paystackService.js
    - cloudinaryService.js
  server.js (entry point)
```

---

### 3. Data Layer (MongoDB)

**Collections Design**:

**users**:
```json
{
  "_id": ObjectId,
  "name": String,
  "email": String,
  "password": String (hashed),
  "phone": String,
  "communityId": ObjectId (reference to communities),
  "role": String ("seeker" | "validator"),
  "verifiedBy": ObjectId (reference to users),
  "createdAt": Date,
  "updatedAt": Date
}
```

**workers**:
```json
{
  "_id": ObjectId,
  "name": String,
  "phone": String, // Add unique constraint
  "email": String, // Add unique constraint
  "services": [String],
  "communityIds": [ObjectId],
  "profilePhoto": String,
  "totalReferences": Number,
  "averageRating": Number,
  "availability": { 
    "monday": [{"start": "09:00", "end": "17:00"}],
    "tuesday": [...],

  },
  "isActive": Boolean, 
  "responseRate": Number,
  "averageResponseTime": Number, 
  "createdAt": Date,
  "updatedAt": Date
}
```

**references**:
```json
{
  "_id": ObjectId,
  "workerId": ObjectId,
  "authorId": ObjectId,
  "communityId": ObjectId,
  "bookingId": ObjectId,
  "serviceType": String,
  "rating": Number,
  "description": String,
  "priceCharged": Number,
  "photos": [String],
  "isRepeatCustomer": Boolean,
  "timesUsed": Number,
  "workQuality": { 
    "professionalism": Number, // 1-5
    "punctuality": Number,
    "cleanliness": Number,
    "valueForMoney": Number
  },
  "disputeStatus": String, //none" | "disputed" | "resolved"
  "disputeReason": String, 
  "isEdited": Boolean, 
  "editHistory": [{
    "editedAt": Date,
    "oldRating": Number,
    "oldDescription": String
  }],
  "visibility": String, 
  "createdAt": Date,
  "updatedAt": Date
}
```

**bookings**:
```json
{
  "_id": ObjectId,
  "seekerId": ObjectId,
  "workerId": ObjectId,
  "serviceType": String,
  "status": String ("pending" | "accepted" | "completed" | "cancelled"),
  "scheduledDate": Date,
  "paymentStatus": String,
  "paystackReference": String,
  "amount": Number,
  "createdAt": Date
}
```

**communities**:
```json
{
  "_id": ObjectId,
  "name": String ("Lekki Phase 1 Estate"),
  "location": {
    "city": String,
    "state": String,
    "coordinates": {
      "lat": Number,
      "lng": Number
    }
  },
  "verificationMethod": String ("whatsapp" | "admin"),
  "whatsappGroupLink": String,
  "totalMembers": Number,
  "createdAt": Date
}
```

---

## Data Flow

### User Registration & Community Verification
```
1. User submits signup form (name, email, password, community)
   ↓
2. Frontend validates input
   ↓
3. POST /api/auth/signup
   ↓
4. Backend validates + checks if community exists
   ↓
5. User created with "unverified" status
   ↓
6. Verification code sent to community WhatsApp group
   ↓
7. User enters verification code
   ↓
8. POST /api/auth/verify
   ↓
9. User status updated to "verified"
   ↓
10. JWT token issued → User logged in
```

---

### Worker Search & Profile View
```
1. User enters search query + filters (service type, location)
   ↓
2. GET /api/workers/search?service=plumber&communityId=xyz
   ↓
3. Backend queries workers collection
   ↓
4. Workers ranked by:
   - References from user's community (highest priority)
   - Total references
   - Average rating
   - Repeat customer count
   ↓
5. Response includes worker basic info + reference summary
   ↓
6. User clicks worker → GET /api/workers/:id
   ↓
7. Full profile returned with:
   - All references (paginated)
   - Before/after photos
   - Repeat usage indicators
   ↓
8. Frontend renders WorkerProfile component
```

---

### Reference Submission (After Completed Job)
```
1. User completes booking
   ↓
2. System prompts: "How did it go with [Worker Name]?"
   ↓
3. User fills reference form (rating, description, photos)
   ↓
4. Photos uploaded to Cloudinary
   ↓
5. POST /api/references
   ↓
6. Backend validates:
   - User is verified community member
   - Booking was completed
   - No duplicate reference for this booking
   ↓
7. Reference saved to database
   ↓
8. Worker's totalReferences and averageRating updated
   ↓
9. If repeat customer: isRepeatCustomer flag set + timesUsed incremented
   ↓
10. Worker notified of new reference
```

---

### Booking Flow with Payment
```
1. User clicks "Request [Worker]" button
   ↓
2. BookingForm modal opens (date, service details)
   ↓
3. POST /api/bookings
   ↓
4. Booking created with "pending_payment" status
   ↓
5. Paystack checkout initialized
   ↓
6. User redirected to Paystack payment page
   ↓
7. User completes payment
   ↓
8. Paystack webhook → POST /api/webhooks/paystack
   ↓
9. Backend verifies webhook signature
   ↓
10. Booking status updated to "paid"
   ↓
11. Worker notified (email + SMS)
   ↓
12. Worker accepts → status = "confirmed"
   ↓
13. After scheduled date → status = "completed"
   ↓
14. Reference submission prompt triggered
```

---

## API Endpoints

### Authentication
- `POST /api/auth/signup` - Create new user
- `POST /api/auth/login` - User login (returns JWT)
- `POST /api/auth/verify` - Verify community membership
- `POST /api/auth/logout` - Invalidate JWT
- `GET /api/auth/me` - Get current user info

### Workers
- `GET /api/workers/search` - Search workers (query params: service, communityId, page)
- `GET /api/workers/:id` - Get worker full profile
- `POST /api/workers` - Create worker profile (admin only)
- `PATCH /api/workers/:id` - Update worker info

### References
- `GET /api/references?workerId=xyz` - Get all references for a worker
- `POST /api/references` - Submit new reference (requires completed booking)
- `GET /api/references/:id` - Get single reference details

### Bookings
- `POST /api/bookings` - Create new booking + initiate payment
- `GET /api/bookings` - Get user's booking history
- `PATCH /api/bookings/:id/accept` - Worker accepts booking
- `PATCH /api/bookings/:id/complete` - Mark booking as completed

### Communities
- `GET /api/communities` - List all communities
- `GET /api/communities/search?city=Lagos` - Search communities
- `POST /api/communities` - Create new community (admin)

### Webhooks
- `POST /api/webhooks/paystack` - Handle Paystack payment notifications

---

## Security Considerations

### 1. Authentication Security
- Passwords hashed with bcrypt (cost factor: 12)
- JWTs signed with secret key (256-bit minimum)
- Refresh token rotation every 7 days
- Rate limiting: 5 login attempts per 15 minutes per IP

### 2. Input Validation
- All user inputs sanitized (express-validator)
- SQL injection prevention (Mongoose ODM)
- XSS prevention (Content Security Policy headers)
- File upload restrictions: max 5MB, images only

### 3. Data Protection
- HTTPS enforced (TLS 1.3)
- Sensitive data encrypted at rest (MongoDB encryption)
- Personal info (phone, email) not exposed in public APIs
- GDPR-compliant data deletion on user request

### 4. Community Verification
- Two-factor verification for community join:
  1. Verification code sent to estate WhatsApp admin
  2. Code expires after 24 hours
- Prevents fake accounts from accessing community data

### 5. Reference Integrity
- References immutable after 24 hours
- Only verified community members can reference
- Workers cannot delete negative references
- Dispute resolution mechanism for false references

---

## Scalability Plan

### Phase 1: MVP (0-1,000 users)
- Single MongoDB Atlas M0 cluster (free tier)
- Vercel free tier (frontend)
- Railway free tier (backend)
- Cloudinary free tier (25GB storage)
- **Cost**: $0/month

### Phase 2: Growth (1,000-10,000 users)
- Upgrade to MongoDB M10 cluster ($0.08/hour)
- Railway Pro plan ($20/month)
- Cloudinary Plus ($99/month)
- Add Redis for caching frequently accessed workers
- **Cost**: ~$180/month

### Phase 3: Scale (10,000-100,000 users)
- MongoDB M30 cluster with replica sets
- Multiple backend instances (horizontal scaling)
- CDN for static assets (Cloudflare)
- Elasticsearch for advanced search
- Microservices architecture (split booking + payment services)
- **Cost**: ~$1,500/month

### Performance Optimizations
- Database indexing on frequently queried fields (communityId, workerId)
- API response caching (60-second TTL for worker lists)
- Image lazy loading and progressive JPEGs
- Server-side pagination (20 items per page)
- WebSocket for real-time notifications (Socket.io)

---

## Why This Architecture Works

### 1. Aligns with Product Vision
- **Community-first design**: CommunityId is a first-class citizen in data models
- **Trust emphasis**: References immutable, verification required, repeat usage tracked
- **Not a generic marketplace**: Architecture prevents self-promotion, prioritizes social proof

### 2. Technically Feasible for MVP
- **Proven stack**: MERN (MongoDB, Express, React, Node) widely documented
- **Free tiers available**: Can launch with $0 infrastructure cost
- **Single developer capable**: No complex distributed systems required
- **Fast iteration**: JavaScript across full stack enables rapid development

### 3. Scalable Foundation
- **Horizontal scaling path**: Node.js clusters, MongoDB sharding
- **Microservices ready**: Clear service boundaries (auth, workers, bookings)
- **Database flexibility**: NoSQL allows schema evolution as product learns

### 4. Maintainable
- **Clear separation of concerns**: Frontend ↔ API ↔ Database
- **RESTful API**: Standard conventions, easy to document
- **Modular code structure**: Components reusable, testable
- **Type safety path**: Can migrate to TypeScript without rewrite

---

## Next Steps (Post Stage 4A)

1. **Database Setup**: Create MongoDB Atlas cluster + seed initial communities
2. **Backend Scaffold**: Set up Express server with basic auth routes
3. **Frontend Scaffold**: Create React app with routing structure
4. **Authentication Flow**: Implement JWT login/signup
5. **Community Verification**: Build verification code system
6. **Worker Profile MVP**: Create basic profile view with static data
7. **Reference Submission**: Build form + backend storage
8. **Search Functionality**: Implement community-proximity ranking
9. **Payment Integration**: Connect Paystack sandbox
10. **Deployment**: Push to Vercel + Railway

---

**Architecture Version**: 1.0  
**Last Updated**: 08/11/2025  
**Author**: Fisayo Tokede