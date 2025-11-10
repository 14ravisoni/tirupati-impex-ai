# Tirupati Impex - Complete System Documentation
## Jewelry Inventory Management System

---

## 🏢 Executive Summary

**Project Name**: Tirupati Impex Jewelry Inventory Management System  
**Purpose**: A comprehensive Django-based backend system for managing jewelry manufacturing, inventory, worker assignments, and sales workflows  
**Status**: Backend 100% Complete, Ready for Frontend Development  
**Architecture**: RESTful API with JWT Authentication  
**Tech Stack**: Django 5.0.1, PostgreSQL, Redis, Celery, Django REST Framework  

### System Overview
This system manages the complete lifecycle of jewelry manufacturing from raw materials to final sale. It handles two primary user types:
- **Clients**: Jewelry shop owners who manage inventory and workers
- **Workers**: Artisans who process jewelry through various manufacturing stages

---

## 📊 Core Business Workflow

### Jewelry Manufacturing Stages
Jewelry progresses through 7 distinct stages:

```
1. Ghatt (Raw Material) → 2. Jadai (First Processing) → 3. Chilai (Second Processing) 
→ 4. Pakai (Third Processing) → 5. Gold Ready → 6. Ready to Sell → 7. Sold
```

### Job Assignment Flow
1. Client creates a job for a specific stage (Jadai/Chilai/Pakai)
2. Worker receives notification of job assignment
3. Worker accepts/rejects the job
4. If accepted, worker completes the work and uploads before/after images
5. Worker submits for client review
6. Client approves (job completed) or sends back for corrections
7. Upon completion, jewelry automatically progresses to next stage

---

## 🗄️ Database Schema

### User Management Tables

#### users
```sql
- id: UUID (Primary Key)
- email: VARCHAR(255) UNIQUE
- password_hash: VARCHAR(255)
- user_type: ENUM('client', 'worker')
- phone: VARCHAR(20)
- is_active: BOOLEAN
- created_at: TIMESTAMP
- updated_at: TIMESTAMP
```

#### clients
```sql
- id: UUID (Primary Key)
- user_id: UUID (Foreign Key → users)
- business_name: VARCHAR(255)
- full_name: VARCHAR(255)
- address: TEXT
- metal_inventory: DECIMAL(10,3) [Gold in grams]
- points_inventory: DECIMAL(12,2) [Money/coins]
- created_at: TIMESTAMP
- updated_at: TIMESTAMP
```

#### workers
```sql
- id: UUID (Primary Key)
- user_id: UUID (Foreign Key → users)
- client_id: UUID (Foreign Key → clients)
- full_name: VARCHAR(255)
- nickname: VARCHAR(100)
- total_jobs_assigned: INT
- total_jobs_accepted: INT
- total_jobs_rejected: INT
- total_jobs_completed: INT
- total_jobs_sent_back: INT
- current_active_jobs: INT
- acceptance_rate: DECIMAL(5,2) [Percentage]
- created_at: TIMESTAMP
- updated_at: TIMESTAMP
```

### Jewelry Management Tables

#### jewels
```sql
- id: UUID (Primary Key)
- client_id: UUID (Foreign Key → clients)
- jewel_type: VARCHAR(100) [Ring, Set, Pendant, etc.]
- current_stage: ENUM('ghatt','jadai','chilai','pakai','gold_ready','ready_to_sell','sold')
- ghatt_weight: DECIMAL(10,3)
- stone_weight: DECIMAL(10,3)
- net_weight: DECIMAL(10,3)
- metal_weight: DECIMAL(10,3)
- is_active: BOOLEAN
- sold_at: TIMESTAMP
- [stage]_stage_at: TIMESTAMP (for each stage)
- created_at: TIMESTAMP
- updated_at: TIMESTAMP
```

#### jobs
```sql
- id: UUID (Primary Key)
- job_number: VARCHAR(50) UNIQUE [JOB-2024-00001]
- jewel_id: UUID (Foreign Key → jewels)
- client_id: UUID (Foreign Key → clients)
- worker_id: UUID (Foreign Key → workers)
- job_stage: ENUM('jadai','chilai','pakai')
- job_description: TEXT
- status: ENUM('pending_acceptance','accepted','rejected','in_progress',
              'submitted_for_review','sent_back_by_worker','sent_back_by_client','completed')
- before_weight: DECIMAL(10,3)
- after_weight: DECIMAL(10,3)
- worker_send_back_count: INT
- client_send_back_count: INT
- last_send_back_reason: TEXT
- assigned_at, accepted_at, rejected_at, started_at, submitted_at, completed_at: TIMESTAMP
- created_at: TIMESTAMP
- updated_at: TIMESTAMP
```

### Supporting Tables

#### stone_types
- Master table for gemstone types (Diamond, Emerald, Ruby, Sapphire, Pearl)

#### jewel_stones
- Links stones to jewels with quantity, carat, weight, and pricing

#### jewel_images & job_images
- Store image URLs for jewels and jobs at different stages

#### sales
- Records jewelry sales with payment method (metal/points)
- Tracks buyer information and transaction details

#### inventory_transactions
- Audit trail for all metal and points inventory changes

#### notifications
- User notifications for job events and system updates

#### job_history
- Complete audit trail of job status changes

---

## 🔑 API Endpoints (60+ Total)

### Authentication APIs
```
POST /api/v1/auth/login/                     # Login (JWT)
POST /api/v1/auth/refresh/                   # Refresh JWT token
POST /api/v1/auth/logout/                    # Logout (blacklist token)
POST /api/v1/auth/register/client/           # Register new client
POST /api/v1/auth/change-password/           # Change password
```

### User Management APIs
```
GET  /api/v1/users/me/                       # Get current user profile
PUT  /api/v1/users/me/update/                # Update profile
```

### Worker Management APIs (Client Only)
```
GET  /api/v1/workers/                        # List all workers
POST /api/v1/workers/create/                 # Create new worker account
GET  /api/v1/workers/{id}/                   # Get worker details
PUT  /api/v1/workers/{id}/update/            # Update worker
DEL  /api/v1/workers/{id}/deactivate/        # Deactivate worker
GET  /api/v1/workers/{id}/jobs/              # Get worker's job history
GET  /api/v1/workers/{id}/stats/             # Get worker statistics
```

### Jewelry Management APIs
```
GET  /api/v1/jewels/                         # List jewels (with filters)
POST /api/v1/jewels/create/                  # Create new jewel
GET  /api/v1/jewels/{id}/                    # Get jewel details
PUT  /api/v1/jewels/{id}/update/             # Update jewel
DEL  /api/v1/jewels/{id}/delete/             # Soft delete jewel
POST /api/v1/jewels/{id}/upload-image/       # Upload jewel image
GET  /api/v1/jewels/{id}/images/             # Get jewel images
POST /api/v1/jewels/{id}/add-stone/          # Add stone to jewel
GET  /api/v1/jewels/{id}/stones/             # Get jewel stones
PUT  /api/v1/jewels/{id}/update-weight/      # Update jewel weight
GET  /api/v1/jewels/by-stage/                # Get jewels grouped by stage
```

### Job Management APIs

#### Client APIs
```
GET  /api/v1/jobs/                           # List all jobs
POST /api/v1/jobs/create/                    # Create new job
GET  /api/v1/jobs/{id}/                      # Get job details
PUT  /api/v1/jobs/{id}/reassign/             # Reassign job to different worker
POST /api/v1/jobs/{id}/approve/              # Approve completed job
POST /api/v1/jobs/{id}/send-back/            # Send job back for corrections
GET  /api/v1/jobs/pending-review/            # Get jobs pending review
```

#### Worker APIs
```
GET  /api/v1/jobs/my-jobs/                   # Get assigned jobs
GET  /api/v1/jobs/my-jobs/pending/           # Get pending acceptance jobs
POST /api/v1/jobs/{id}/accept/               # Accept job
POST /api/v1/jobs/{id}/reject/               # Reject job
POST /api/v1/jobs/{id}/start/                # Start working on job
POST /api/v1/jobs/{id}/upload-before-image/  # Upload before image
POST /api/v1/jobs/{id}/upload-after-image/   # Upload after image
POST /api/v1/jobs/{id}/submit/               # Submit job for review
POST /api/v1/jobs/{id}/send-back/            # Send back (weight issues, etc.)
```

### Sales APIs
```
GET  /api/v1/sales/                          # List all sales
POST /api/v1/sales/create/                   # Record new sale
GET  /api/v1/sales/{id}/                     # Get sale details
GET  /api/v1/sales/report/                   # Generate sales report
```

### Inventory APIs
```
GET  /api/v1/inventory/transactions/         # List inventory transactions
POST /api/v1/inventory/add-metal/            # Add metal to inventory
POST /api/v1/inventory/add-points/           # Add points to inventory
POST /api/v1/inventory/withdraw-metal/       # Withdraw metal
POST /api/v1/inventory/withdraw-points/      # Withdraw points
GET  /api/v1/inventory/current/              # Get current inventory levels
```

### Notification APIs
```
GET  /api/v1/notifications/                  # List user notifications
GET  /api/v1/notifications/unread/           # Get unread notifications
POST /api/v1/notifications/{id}/mark-read/   # Mark notification as read
POST /api/v1/notifications/mark-all-read/    # Mark all as read
```

### Dashboard/Analytics APIs
```
GET  /api/v1/dashboard/summary/              # Dashboard summary stats
GET  /api/v1/dashboard/jewels-by-stage/      # Jewel distribution by stage
```

---

## 🔄 Automatic System Behaviors (Django Signals)

### Worker Statistics Auto-Updates
When job status changes:
- **Job Accepted**: Increments `total_jobs_accepted`, `current_active_jobs`, recalculates `acceptance_rate`
- **Job Rejected**: Increments `total_jobs_rejected`, recalculates `acceptance_rate`
- **Job Completed**: Increments `total_jobs_completed`, decrements `current_active_jobs`
- **Job Sent Back**: Increments `total_jobs_sent_back`

### Automatic Notifications
System automatically creates notifications for:
- New job assignment to worker
- Job acceptance/rejection
- Job submission for review
- Job approval/send-back
- Job completion

### Inventory Updates
When a sale is recorded:
- Updates client's `metal_inventory` or `points_inventory` based on payment method
- Creates inventory transaction record
- Marks jewel as sold and sets `is_active = false`

### Jewel Stage Progression
When a job is completed:
- Automatically progresses jewel to next stage
- Updates stage timestamp

---

## 🛠️ Technical Implementation Details

### Django Project Structure
```
backend/
├── config/
│   ├── settings/
│   │   ├── base.py         # Base settings
│   │   ├── development.py  # Dev environment
│   │   └── production.py   # Production settings
│   └── urls.py             # Main URL configuration
├── apps/
│   ├── authentication/     # JWT auth implementation
│   ├── users/             # User models & APIs
│   ├── workers/           # Worker management
│   ├── jewels/           # Jewelry CRUD
│   ├── jobs/             # Job workflow
│   ├── inventory/        # Inventory tracking
│   ├── sales/           # Sales recording
│   └── notifications/   # Notification system
├── media/               # Uploaded images
├── static/             # Static files
└── manage.py
```

### Key Django Settings
```python
# Authentication
AUTH_USER_MODEL = 'users.User'
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': ('rest_framework_simplejwt.authentication.JWTAuthentication',),
    'DEFAULT_PERMISSION_CLASSES': ('rest_framework.permissions.IsAuthenticated',),
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 20,
}

# JWT Configuration
SIMPLE_JWT = {
    'ACCESS_TOKEN_LIFETIME': timedelta(hours=24),
    'REFRESH_TOKEN_LIFETIME': timedelta(days=30),
    'ROTATE_REFRESH_TOKENS': True,
    'BLACKLIST_AFTER_ROTATION': True,
}

# Celery (for background tasks)
CELERY_BROKER_URL = 'redis://localhost:6379/0'

# Channels (for WebSocket support)
CHANNEL_LAYERS = {
    'default': {
        'BACKEND': 'channels_redis.core.RedisChannelLayer',
        'CONFIG': {'hosts': [('127.0.0.1', 6379)]},
    },
}
```

### Required Dependencies
```txt
Django==5.0.1
djangorestframework==3.14.0
djangorestframework-simplejwt==5.3.1
django-cors-headers==4.3.1
psycopg2-binary==2.9.9
Pillow==10.2.0
python-dotenv==1.0.0
django-filter==23.5
channels==4.0.0
channels-redis==4.2.0
redis==5.0.1
celery==5.3.6
```

---

## 🚀 Setup Instructions

### 1. Database Setup
```sql
CREATE DATABASE tirupati_impex_dev;
```

### 2. Environment Variables (.env)
```env
DJANGO_ENVIRONMENT=development
SECRET_KEY=your-secret-key-here
DB_NAME=tirupati_impex_dev
DB_USER=postgres
DB_PASSWORD=your_password
DB_HOST=localhost
DB_PORT=5432
```

### 3. Django Setup
```bash
# Install dependencies
pip install -r requirements/development.txt

# Run migrations
python manage.py makemigrations
python manage.py migrate

# Create superuser
python manage.py createsuperuser

# Load initial data (stone types)
python manage.py shell
>>> from apps.jewels.models import StoneType
>>> for name in ['Diamond', 'Emerald', 'Ruby', 'Sapphire', 'Pearl']:
...     StoneType.objects.create(name=name)

# Run server
python manage.py runserver
```

---

## 📝 API Usage Examples

### 1. Client Registration & Login
```bash
# Register Client
POST /api/v1/auth/register/client/
{
  "email": "client@test.com",
  "password": "testpass123",
  "business_name": "Test Jewelers",
  "full_name": "Test Client",
  "phone": "+91-9876543210"
}

# Login
POST /api/v1/auth/login/
{
  "email": "client@test.com",
  "password": "testpass123"
}

# Response includes JWT token and user profile
{
  "access": "eyJ0eXAiOiJKV1QiLCJhbGc...",
  "refresh": "eyJ0eXAiOiJKV1QiLCJhbGc...",
  "user": {
    "id": "uuid-here",
    "email": "client@test.com",
    "userType": "client",
    "profile": {
      "businessName": "Test Jewelers",
      "fullName": "Test Client",
      "phone": "+91-9876543210"
    }
  }
}
```

### 2. Create Worker Account (Client Only)
```bash
POST /api/v1/workers/create/
Headers: Authorization: Bearer {token}
{
  "email": "worker@test.com",
  "password": "workerpass123",
  "full_name": "Test Worker",
  "nickname": "TW",
  "phone": "+91-9876543211"
}
```

### 3. Create Jewel
```bash
POST /api/v1/jewels/create/
Headers: Authorization: Bearer {token}
{
  "jewel_type": "Ring",
  "ghatt_weight": 10.5,
  "stone_weight": 0.5,
  "net_weight": 10.0
}
```

### 4. Create Job for Worker
```bash
POST /api/v1/jobs/create/
Headers: Authorization: Bearer {token}
{
  "jewel_id": "jewel-uuid",
  "worker_id": "worker-uuid",
  "job_stage": "jadai",
  "job_description": "Please complete jadai work carefully",
  "before_weight": 10.0
}
```

### 5. Worker Accepts Job
```bash
POST /api/v1/jobs/{job-id}/accept/
Headers: Authorization: Bearer {worker-token}
```

### 6. Worker Submits Completed Job
```bash
POST /api/v1/jobs/{job-id}/submit/
Headers: Authorization: Bearer {worker-token}
{
  "after_weight": 9.8,
  "notes": "Completed successfully"
}
```

### 7. Client Approves Job
```bash
POST /api/v1/jobs/{job-id}/approve/
Headers: Authorization: Bearer {client-token}
```

### 8. Record Sale
```bash
POST /api/v1/sales/create/
Headers: Authorization: Bearer {token}
{
  "jewel_id": "jewel-uuid",
  "buyer_name": "Customer Name",
  "buyer_phone": "+91-9876543212",
  "buyer_type": "retailer",
  "payment_method": "metal",
  "metal_amount": 8.5,
  "final_amount": 45000,
  "gold_price_per_gram": 5294
}
```

---

## 🔐 Security Features

1. **JWT Authentication** with token refresh mechanism
2. **Password Hashing** using Django's default (PBKDF2)
3. **CORS Configuration** for cross-origin requests
4. **Permission Classes** ensuring proper access control
5. **UUID Primary Keys** for better security and distribution
6. **Audit Logs** for tracking all critical changes
7. **Soft Deletes** for jewels (maintaining data integrity)

---

## 📈 Business Logic & Rules

### Worker Assignment Rules
- Workers can only be created by their assigned client
- Workers can only see/accept jobs assigned to them
- Workers cannot modify jewel or job details directly

### Job Lifecycle Rules
- Jobs must be accepted before work can begin
- Jobs can be sent back multiple times by both worker and client
- Completed jobs automatically progress jewel to next stage
- Same job continues even after send-backs (no new job created)

### Inventory Rules
- Metal and points inventory updated automatically on sales
- All inventory changes create transaction records
- Inventory cannot go negative

### Weight Tracking Rules
- Before and after weights recorded for each job
- Weight discrepancies can trigger job send-backs
- Net weight = Ghatt weight - Stone weight

---

## 🎯 Current Project Status

### ✅ Completed
- Database schema design
- All Django models
- Authentication system with JWT
- Complete CRUD APIs for all modules
- Worker statistics auto-update system
- Notification system
- Image upload functionality
- Sales and inventory management
- Dashboard analytics APIs
- Django signals for workflow automation

### 🔄 In Progress / Planned
- React web application for clients
- React Native mobile app for workers
- API documentation (Swagger/OpenAPI)
- WebSocket implementation for real-time updates
- Email/SMS notifications
- Report generation (PDF)
- Data export functionality

---

## 💡 Key Design Decisions

1. **UUID Primary Keys**: Better for distributed systems and more secure than sequential IDs
2. **Separate Client/Worker Models**: Clear separation of concerns and permissions
3. **Job-based Workflow**: Each processing stage is a separate job for better tracking
4. **Auto-updating Statistics**: Reduces database queries and ensures data consistency
5. **Soft Deletes**: Maintains referential integrity and allows data recovery
6. **Stage-based Images**: Visual documentation at each manufacturing stage
7. **Metal/Points Dual Currency**: Supports traditional jewelry industry practices

---

## 🔧 Troubleshooting Guide

### Common Issues

1. **Migration Errors**
```bash
# Reset migrations
find . -path "*/migrations/*.py" -not -name "__init__.py" -delete
python manage.py makemigrations
python manage.py migrate
```

2. **CORS Issues**
```python
# In development.py
CORS_ALLOW_ALL_ORIGINS = True  # Only for development
```

3. **Image Upload Issues**
```bash
# Ensure media directories exist
mkdir -p media/jewels media/jobs
```

4. **Permission Denied Errors**
- Verify user type (client vs worker)
- Check JWT token is valid and included in headers
- Ensure user has appropriate permissions for the endpoint

---

## 📚 Additional Notes

### Performance Considerations
- Database indexes on all foreign keys and frequently filtered fields
- Pagination enabled (20 items per page default)
- Selective field loading in serializers
- Efficient query optimization with `select_related` and `prefetch_related`

### Scalability Preparations
- Redis configured for caching and Celery
- WebSocket support via Django Channels
- Database designed for sharding by client_id if needed
- Stateless authentication (JWT) for horizontal scaling

### Development Workflow
1. Backend API development (Complete)
2. API testing with Postman/curl
3. Frontend development (React/React Native)
4. Integration testing
5. User acceptance testing
6. Production deployment

---

## 📞 Contact & Support

This system was developed for Tirupati Impex jewelry business to modernize their inventory management and worker assignment processes. The backend is fully functional and ready for frontend integration.

### System Capabilities Summary
- **Users**: 2 types (Clients and Workers)
- **Stages**: 7 jewelry manufacturing stages
- **APIs**: 60+ RESTful endpoints
- **Authentication**: JWT with refresh tokens
- **Real-time**: WebSocket ready
- **Database**: PostgreSQL with comprehensive schema
- **Files**: Image upload support
- **Automation**: Signal-based workflow updates

---

*This document serves as a complete reference for the Tirupati Impex Jewelry Inventory Management System. Any AI model or developer can understand the entire system architecture, implementation details, and business logic from this single comprehensive document.*

---

**Version**: 1.0.0  
**Last Updated**: November 2024  
**Backend Status**: 100% Complete  
**Next Phase**: Frontend Development
