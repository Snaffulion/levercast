# Levercast Software Requirements Specification

## System Design

- **Client-Server Architecture**: Web-based application with a separate frontend and backend
- **Responsive Design**: Priority on desktop experience with mobile responsiveness
- **Real-time Processing**: LLM integration for content transformation
- **Multi-platform Publishing**: API integrations with social media platforms
- **Scalable Infrastructure**: Support for growing user base and additional platforms
- **Secure Authentication**: OAuth integration for both user accounts and social media platforms

## Architecture Pattern

- **Frontend**: React Single Page Application (SPA) with Next.js framework
  - Server-side rendering for initial page loads and SEO
  - Client-side rendering for dynamic user interactions
- **Backend**: API-driven architecture with RESTful endpoints
  - Modular service-based approach for different functionalities
  - Serverless functions for event-driven operations (content transformation, scheduled publishing)
- **Design Pattern**: Feature-based modular architecture
  - Core modules: Authentication, Content Creation, Template Management, Publishing, Analytics
  - Separation of concerns between UI components, business logic, and data access

## State Management

- **Global State**: React Context API for application-wide state
  - User authentication status
  - Theme preferences (dark/light mode)
  - Global application settings
- **Local Component State**: React useState for component-specific states
- **Server State**: React Query for API data fetching, caching, and synchronization
  - Optimistic UI updates for improved perceived performance
  - Background refetching for data freshness
- **Form State**: React Hook Form for efficient form handling
- **Persistence**: LocalStorage for user preferences and draft content

## Data Flow

- **User Authentication Flow**:
  1. User signs up/logs in via Clerk Auth
  2. Authentication tokens stored securely in HTTP-only cookies
  3. Protected routes check authentication status
  4. Social media account connections managed through OAuth

- **Content Creation Flow**:
  1. User inputs raw content in the Idea Pad
  2. Input sent to backend API
  3. Backend processes with LLM integration for transformation
  4. Transformed content returned to frontend
  5. User reviews and edits platform-specific versions
  6. Final content stored in database with user association

- **Publishing Flow**:
  1. User selects platforms and scheduling options
  2. Request sent to backend publishing service
  3. Content either published immediately or stored with scheduled timestamp
  4. Scheduled publisher function checks database for pending publications
  5. Social media API calls executed at scheduled times
  6. Status updates returned to frontend

## Technical Stack

- **Frontend**:
  - Framework: React with Next.js
  - Styling: Tailwind CSS with shadcn/ui components
  - Icons: Lucide Icons
  - State Management: React Context API + React Query
  - Form Handling: React Hook Form
  - TypeScript for type safety

- **Backend**:
  - API Framework: Next.js API routes
  - Serverless Functions: Vercel Functions
  - Authentication: Clerk Auth
  - LLM Integration: OpenAI API or similar

- **Database**:
  - Primary Database: Supabase (PostgreSQL)
  - Real-time Features: Supabase Realtime

- **DevOps**:
  - Hosting: Vercel
  - CI/CD: GitHub Actions
  - Monitoring: Vercel Analytics + custom logging

- **Third-party Services**:
  - Social Media APIs: Twitter API, LinkedIn API
  - Payment Processing: Stripe
  - Email Notifications: SendGrid or Resend

## Authentication Process

- **User Authentication**:
  - Clerk Auth integration for user management
  - Email/password and social login options
  - JWT tokens for session management
  - Role-based access control (User, Admin)

- **Social Media Authentication**:
  - OAuth 2.0 flow for each platform
  - Secure token storage in database
  - Scheduled refresh of access tokens
  - Revocation process for removed connections

- **API Authentication**:
  - JWT-based API authentication
  - Role-based endpoint access
  - Rate limiting to prevent abuse

## Route Design

- **Public Routes**:
  - `/` - Landing page
  - `/login` - User authentication
  - `/signup` - New user registration
  - `/pricing` - Subscription options

- **Protected Routes**:
  - `/app` - Main application shell
  - `/app/create` - Content creation workspace
  - `/app/posts` - Recent posts view
  - `/app/templates` - Template management
  - `/app/settings` - User and application settings
  - `/app/settings/connections` - Social media account management

- **API Routes**:
  - `/api/auth/*` - Authentication endpoints
  - `/api/content` - Content management
  - `/api/templates` - Template management
  - `/api/publish` - Content publishing
  - `/api/platforms` - Social media platform operations
  - `/api/analytics` - Performance data

## API Design

- **Authentication Endpoints**:
  - `POST /api/auth/register` - Create new user
  - `POST /api/auth/login` - Authenticate user
  - `POST /api/auth/logout` - End user session
  - `GET /api/auth/me` - Get current user info
  - `POST /api/auth/platforms/{platform}` - Connect social platform

- **Content Endpoints**:
  - `POST /api/content` - Create new content
  - `GET /api/content` - List user content with pagination
  - `GET /api/content/{id}` - Get specific content
  - `PUT /api/content/{id}` - Update content
  - `DELETE /api/content/{id}` - Remove content
  - `POST /api/content/transform` - Process raw content with templates

- **Publishing Endpoints**:
  - `POST /api/publish` - Publish or schedule content
  - `GET /api/publish/scheduled` - Get scheduled content
  - `PUT /api/publish/{id}` - Update scheduled content
  - `DELETE /api/publish/{id}` - Cancel scheduled publication

- **Template Endpoints**:
  - `POST /api/templates` - Create template
  - `GET /api/templates` - List user templates
  - `GET /api/templates/{id}` - Get specific template
  - `PUT /api/templates/{id}` - Update template
  - `DELETE /api/templates/{id}` - Remove template

- **Platform Endpoints**:
  - `GET /api/platforms` - List connected platforms
  - `POST /api/platforms/{platform}/disconnect` - Remove platform connection
  - `GET /api/platforms/{platform}/analytics` - Get platform metrics

## Database Design ERD

- **Users Table**:
  - id (PK)
  - email
  - name
  - created_at
  - updated_at
  - last_login
  - preferences (JSONB)

- **PlatformConnections Table**:
  - id (PK)
  - user_id (FK → Users.id)
  - platform_name
  - access_token (encrypted)
  - refresh_token (encrypted)
  - expiry_date
  - platform_user_id
  - created_at
  - updated_at

- **Content Table**:
  - id (PK)
  - user_id (FK → Users.id)
  - title
  - raw_content
  - images (JSONB array)
  - created_at
  - updated_at
  - status (draft, published, scheduled)

- **ContentVersions Table**:
  - id (PK)
  - content_id (FK → Content.id)
  - platform_name
  - transformed_content
  - character_count
  - media_urls (JSONB array)
  - created_at
  - updated_at

- **PublishRecords Table**:
  - id (PK)
  - content_version_id (FK → ContentVersions.id)
  - platform_name
  - scheduled_time
  - published_time
  - status (scheduled, published, failed)
  - platform_post_id
  - response_data (JSONB)
  - created_at
  - updated_at

- **Templates Table**:
  - id (PK)
  - user_id (FK → Users.id)
  - name
  - description
  - prompt_template
  - platform_specific (boolean)
  - platform_name
  - created_at
  - updated_at

- **Analytics Table**:
  - id (PK)
  - publish_record_id (FK → PublishRecords.id)
  - views
  - likes
  - comments
  - shares
  - clicks
  - other_metrics (JSONB)
  - last_updated
  - created_at

- **Subscriptions Table**:
  - id (PK)
  - user_id (FK → Users.id)
  - stripe_customer_id
  - stripe_subscription_id
  - plan_level
  - status
  - current_period_start
  - current_period_end
  - created_at
  - updated_at