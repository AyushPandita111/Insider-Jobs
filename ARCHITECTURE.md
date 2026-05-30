# Insider Jobs - Architecture Documentation

## 📋 Table of Contents
1. [Project Overview](#project-overview)
2. [Technology Stack](#technology-stack)
3. [System Architecture](#system-architecture)
4. [Database Schema](#database-schema)
5. [API Architecture](#api-architecture)
6. [Authentication & Authorization](#authentication--authorization)
7. [File Upload & Storage](#file-upload--storage)
8. [Frontend Architecture](#frontend-architecture)
9. [Key Features](#key-features)
10. [Deployment](#deployment)

---

## 🎯 Project Overview

**Insider Jobs** is a full-stack job portal application that connects job seekers with companies. The platform supports two types of users:
- **Job Seekers**: Can browse jobs, apply for positions, and manage their applications
- **Companies/Recruiters**: Can post jobs, manage listings, and review applications

---

## 🛠 Technology Stack

### Frontend
- **React 18.3.1** - UI library
- **Vite 6.0.1** - Build tool and dev server
- **React Router DOM 7.0.1** - Client-side routing
- **Clerk React 5.17.1** - User authentication (job seekers)
- **Axios 1.7.9** - HTTP client
- **Tailwind CSS 3.4.15** - Utility-first CSS framework
- **Quill 2.0.3** - Rich text editor for job descriptions
- **React Toastify 10.0.6** - Toast notifications
- **Moment.js 2.30.1** - Date formatting

### Backend
- **Node.js** with **Express 4.21.1** - Server framework
- **MongoDB** with **Mongoose 8.8.3** - Database and ODM
- **Clerk Express 1.3.21** - Authentication middleware
- **JWT (jsonwebtoken 9.0.2)** - Token-based auth for companies
- **Bcrypt 5.1.1** - Password hashing
- **Cloudinary 2.5.1** - Cloud storage for images and resumes
- **Multer 1.4.5** - File upload middleware
- **Sentry** - Error monitoring and tracking
- **CORS 2.8.5** - Cross-origin resource sharing
- **Svix 1.42.0** - Webhook verification

---

## 🏗 System Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        CLIENT LAYER                          │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  React SPA (Vite)                                     │   │
│  │  - React Router (Navigation)                          │   │
│  │  - Context API (State Management)                     │   │
│  │  - Clerk (User Auth)                                  │   │
│  │  - Axios (API Calls)                                  │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                            ↕ HTTPS
┌─────────────────────────────────────────────────────────────┐
│                        SERVER LAYER                          │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Express.js API Server                                │   │
│  │  - REST API Endpoints                                 │   │
│  │  - JWT Auth Middleware (Companies)                    │   │
│  │  - Clerk Middleware (Users)                           │   │
│  │  - Multer (File Uploads)                              │   │
│  │  - Sentry (Error Tracking)                            │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                            ↕
┌─────────────────────────────────────────────────────────────┐
│                     EXTERNAL SERVICES                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   MongoDB    │  │  Cloudinary  │  │    Clerk     │      │
│  │   Database   │  │  (Storage)   │  │   (Auth)     │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

### Request Flow

#### Job Seeker Flow
```
User → Clerk Auth → React App → API (Clerk Middleware) → MongoDB → Response
```

#### Company/Recruiter Flow
```
Company → Login → JWT Token → React App → API (JWT Middleware) → MongoDB → Response
```

---

## 💾 Database Schema

### Collections

#### 1. **User** (Job Seekers)
```javascript
{
  _id: String,              // Clerk User ID
  name: String,             // Full name
  email: String,            // Unique email
  resume: String,           // Cloudinary URL
  image: String             // Profile image URL
}
```

#### 2. **Company** (Recruiters)
```javascript
{
  _id: ObjectId,            // Auto-generated
  name: String,             // Company name
  email: String,            // Unique email
  image: String,            // Company logo URL
  password: String          // Bcrypt hashed password
}
```

#### 3. **Job**
```javascript
{
  _id: ObjectId,
  title: String,            // Job title
  description: String,      // Rich text description
  location: String,         // Job location
  category: String,         // Job category
  level: String,            // Experience level
  salary: Number,           // Salary amount
  date: Number,             // Timestamp
  visible: Boolean,         // Job visibility (default: true)
  companyId: ObjectId       // Reference to Company
}
```

#### 4. **JobApplication**
```javascript
{
  _id: ObjectId,
  userId: String,           // Reference to User (Clerk ID)
  companyId: ObjectId,      // Reference to Company
  jobId: ObjectId,          // Reference to Job
  status: String,           // 'Pending', 'Accepted', 'Rejected'
  date: Number              // Application timestamp
}
```

### Entity Relationships

```
Company (1) ──────< (N) Job
   │                   │
   │                   │
   └────< (N) JobApplication (N) >────┘
                      │
                      │
                   User (1)
```

---

## 🔌 API Architecture

### Base URL Structure
```
Backend: http://localhost:5000
Frontend: http://localhost:5173
```

### API Endpoints

#### **Company Routes** (`/api/company`)
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | `/register` | None | Register new company |
| POST | `/login` | None | Company login |
| GET | `/company` | JWT | Get company profile |
| POST | `/post-job` | JWT | Create new job posting |
| GET | `/applicants` | JWT | Get all applicants |
| GET | `/list-jobs` | JWT | Get company's posted jobs |
| POST | `/change-status` | JWT | Update application status |
| POST | `/change-visiblity` | JWT | Toggle job visibility |

#### **Job Routes** (`/api/jobs`)
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/` | None | Get all visible jobs |
| GET | `/:id` | None | Get single job by ID |

#### **User Routes** (`/api/users`)
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/user` | Clerk | Get user profile |
| POST | `/apply` | Clerk | Apply for a job |
| GET | `/applications` | Clerk | Get user's applications |
| POST | `/update-resume` | Clerk | Upload/update resume |

#### **Webhook Routes**
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | `/webhooks` | Svix | Clerk user sync webhook |

---

## 🔐 Authentication & Authorization

### Dual Authentication System

#### 1. **Job Seekers (Clerk Authentication)**
- **Provider**: Clerk
- **Flow**: OAuth-based authentication
- **Token**: Managed by Clerk SDK
- **Middleware**: `@clerk/express` middleware
- **User Sync**: Webhook-based sync with MongoDB

**Webhook Events Handled:**
- `user.created` - Create user in MongoDB
- `user.updated` - Update user data
- `user.deleted` - Remove user from MongoDB

#### 2. **Companies (JWT Authentication)**
- **Provider**: Custom JWT implementation
- **Flow**: Email/password login
- **Token**: JWT stored in localStorage
- **Middleware**: Custom `protectCompany` middleware
- **Password**: Bcrypt hashed (10 salt rounds)

### Authentication Flow Diagram

```
┌─────────────────────────────────────────────────────────┐
│                    JOB SEEKER AUTH                       │
├─────────────────────────────────────────────────────────┤
│  1. User signs up/in via Clerk                          │
│  2. Clerk webhook triggers                              │
│  3. User data synced to MongoDB                         │
│  4. Clerk token used for API requests                   │
│  5. Backend validates via Clerk middleware              │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                    COMPANY AUTH                          │
├─────────────────────────────────────────────────────────┤
│  1. Company registers with email/password               │
│  2. Password hashed with bcrypt                         │
│  3. JWT token generated on login                        │
│  4. Token stored in localStorage                        │
│  5. Token sent in headers for protected routes          │
│  6. Backend validates JWT and attaches company to req   │
└─────────────────────────────────────────────────────────┘
```

---

## 📁 File Upload & Storage

### Cloudinary Integration

**Supported File Types:**
- **Images**: Company logos, user profile pictures (PNG, JPG)
- **Documents**: User resumes (PDF)

**Upload Flow:**
```
Client → Multer (temp storage) → Cloudinary API → Secure URL → MongoDB
```

**Configuration:**
- Files temporarily stored via Multer
- Uploaded to Cloudinary cloud storage
- Secure URLs stored in database
- Original files deleted after upload

---

## ⚛️ Frontend Architecture

### Component Structure

```
src/
├── components/
│   ├── Navbar.jsx              # Navigation bar
│   ├── Hero.jsx                # Landing page hero
│   ├── JobListing.jsx          # Job list display
│   ├── JobCard.jsx             # Individual job card
│   ├── Footer.jsx              # Footer component
│   ├── AppDownload.jsx         # App download section
│   ├── RecruiterLogin.jsx      # Company login modal
│   └── Loading.jsx             # Loading spinner
├── pages/
│   ├── Home.jsx                # Landing page
│   ├── ApplyJob.jsx            # Job application page
│   ├── Applications.jsx        # User's applications
│   ├── Dashboard.jsx           # Company dashboard
│   ├── AddJob.jsx              # Create job posting
│   ├── ManageJobs.jsx          # Manage company jobs
│   └── ViewApplications.jsx    # View applicants
├── context/
│   └── AppContext.jsx          # Global state management
├── assets/                     # Images, icons, SVGs
├── App.jsx                     # Main app component
└── main.jsx                    # Entry point
```

### State Management (Context API)

**AppContext provides:**
- `searchFilter` - Job search filters
- `jobs` - All job listings
- `userData` - Current user data
- `userApplications` - User's job applications
- `companyData` - Company profile data
- `companyToken` - JWT token for companies
- `showRecruiterLogin` - Modal visibility state

**Key Functions:**
- `fetchJobs()` - Load all jobs
- `fetchUserData()` - Load user profile
- `fetchUserApplications()` - Load user applications
- `fetchCompanyData()` - Load company profile

### Routing Structure

```
/ (Home)
├── /apply-job/:id (Apply for specific job)
├── /applications (User's applications)
└── /dashboard (Company dashboard)
    ├── /dashboard/add-job (Create job)
    ├── /dashboard/manage-jobs (Manage jobs)
    └── /dashboard/view-applications (View applicants)
```

**Protected Routes:**
- Dashboard routes require `companyToken`
- User routes require Clerk authentication

---

## ✨ Key Features

### For Job Seekers
1. **Browse Jobs** - View all available job listings
2. **Search & Filter** - Search by title and location
3. **Apply for Jobs** - One-click application process
4. **Track Applications** - View application status
5. **Profile Management** - Upload and update resume
6. **Clerk Authentication** - Secure OAuth login

### For Companies
1. **Company Registration** - Create company account
2. **Post Jobs** - Create detailed job listings with rich text
3. **Manage Jobs** - Edit, hide/show job postings
4. **View Applicants** - See all applications with candidate details
5. **Application Management** - Accept/reject applications
6. **Dashboard Analytics** - View applicant counts per job

### Technical Features
1. **Rich Text Editor** - Quill.js for job descriptions
2. **File Uploads** - Resume and image uploads via Cloudinary
3. **Real-time Notifications** - Toast notifications for user actions
4. **Error Monitoring** - Sentry integration for error tracking
5. **Responsive Design** - Tailwind CSS for mobile-first design
6. **Webhook Integration** - Clerk webhook for user synchronization

---

## 🚀 Deployment

### Environment Variables

#### Backend (.env)
```env
MONGODB_URI=mongodb://...
JWT_SECRET=your_jwt_secret
CLERK_WEBHOOK_SECRET=whsec_...
CLOUDINARY_NAME=your_cloud_name
CLOUDINARY_API_KEY=your_api_key
CLOUDINARY_API_SECRET=your_api_secret
SENTRY_DSN=your_sentry_dsn
PORT=5000
```

#### Frontend (.env)
```env
VITE_BACKEND_URL=http://localhost:5000
VITE_CLERK_PUBLISHABLE_KEY=pk_test_...
```

### Deployment Configuration

**Backend (Vercel):**
- `vercel.json` configured for Express deployment
- Serverless function setup

**Frontend (Vercel):**
- `vercel.json` configured for SPA routing
- Vite build optimization

---

## 🔄 Data Flow Examples

### Job Application Flow
```
1. User clicks "Apply" on job
2. Frontend sends POST /api/users/apply with jobId
3. Backend validates:
   - User is authenticated (Clerk)
   - Job exists
   - User hasn't already applied
4. Creates JobApplication record
5. Returns success response
6. Frontend updates UI and shows toast
```

### Company Job Posting Flow
```
1. Company fills job form with rich text editor
2. Frontend sends POST /api/company/post-job
3. Backend validates:
   - Company is authenticated (JWT)
   - All required fields present
4. Creates Job record with companyId
5. Returns new job data
6. Frontend redirects to manage jobs page
```

---

## 🎨 Design Patterns Used

1. **MVC Pattern** - Controllers handle business logic
2. **Repository Pattern** - Mongoose models as data layer
3. **Middleware Pattern** - Authentication and file upload
4. **Context Pattern** - React Context for state management
5. **HOC Pattern** - Protected routes wrapper
6. **Webhook Pattern** - Clerk user synchronization

---

## 🔧 Development Setup

### Prerequisites
- Node.js 16+
- MongoDB instance
- Clerk account
- Cloudinary account

### Installation

```bash
# Clone repository
git clone <repo-url>

# Install backend dependencies
cd server
npm install

# Install frontend dependencies
cd ../client
npm install

# Setup environment variables
# Create .env files in both client and server directories

# Run backend
cd server
npm run server

# Run frontend
cd client
npm run dev
```

---

## 📊 Performance Considerations

1. **Database Indexing** - Email fields indexed for faster lookups
2. **Pagination** - Can be implemented for large job lists
3. **Lazy Loading** - Images optimized via Cloudinary
4. **Code Splitting** - Vite handles automatic code splitting
5. **Caching** - LocalStorage for company tokens

---

## 🔒 Security Measures

1. **Password Hashing** - Bcrypt with 10 salt rounds
2. **JWT Tokens** - Secure token-based authentication
3. **CORS Configuration** - Controlled cross-origin access
4. **Input Validation** - Server-side validation for all inputs
5. **Webhook Verification** - Svix signature verification
6. **Environment Variables** - Sensitive data in .env files
7. **Clerk Security** - OAuth-based secure authentication

---

## 📈 Scalability Considerations

1. **Microservices Ready** - Separate client/server architecture
2. **Cloud Storage** - Cloudinary for file storage
3. **Database Scaling** - MongoDB Atlas for horizontal scaling
4. **CDN Integration** - Cloudinary CDN for assets
5. **Serverless Deployment** - Vercel serverless functions

---

## 🐛 Error Handling

1. **Sentry Integration** - Automatic error tracking
2. **Try-Catch Blocks** - All async operations wrapped
3. **User-Friendly Messages** - Toast notifications for errors
4. **Logging** - Console logs for debugging
5. **Graceful Degradation** - Fallback UI for errors

---

## 📝 API Response Format

### Success Response
```json
{
  "success": true,
  "data": { ... },
  "message": "Operation successful"
}
```

### Error Response
```json
{
  "success": false,
  "message": "Error description"
}
```

---

## 🎯 Future Enhancements

1. Real-time notifications (WebSockets)
2. Advanced search filters
3. Job recommendations (ML-based)
4. Chat system between recruiters and candidates
5. Email notifications
6. Analytics dashboard
7. Multi-language support
8. Mobile app (React Native)

---

**Last Updated:** 2024
**Version:** 1.0.0
