# Smart Online Shopping Assistant - Architecture Documentation

## System Architecture Overview

The Smart Online Shopping Assistant follows a modern three-tier architecture with a React frontend, Express backend, and MySQL database. The system uses tRPC for type-safe API communication and integrates with external APIs for real-time price data.

```
┌─────────────────────────────────────────────────────────────┐
│                    Client Layer (React 19)                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  Home Page   │  │ Comparison   │  │  Dashboard   │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│  ┌──────────────┐  ┌──────────────┐                        │
│  │ Prediction   │  │ Budget Alerts│                        │
│  └──────────────┘  └──────────────┘                        │
└────────────────────────────────────────────────────────────┘
                            ↓
                    tRPC Type-Safe API
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                  API Layer (Express + tRPC)                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ Comparison   │  │ Dashboard    │  │ Prediction   │      │
│  │ Router       │  │ Router       │  │ Router       │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│  ┌──────────────┐  ┌──────────────┐                        │
│  │ Auth Router  │  │ Budget Alerts│                        │
│  └──────────────┘  └──────────────┘                        │
└────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│              Business Logic Layer (Services)                │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ Price API    │  │ Prediction   │  │ Product DB   │      │
│  │ Service      │  │ Service      │  │ Service      │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                  Data Layer (MySQL + Drizzle)               │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Users | Products | Comparisons | Predictions       │   │
│  │  Alerts | History | Cache | Savings                 │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│              External Services (RapidAPI)                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ Amazon API   │  │ Flipkart API │  │ OAuth Server │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

## Frontend Architecture

### Technology Stack

- **Framework**: React 19 with TypeScript
- **Styling**: Tailwind CSS 4 with custom theme
- **UI Components**: shadcn/ui component library
- **State Management**: React Query (via tRPC)
- **Routing**: wouter (lightweight router)
- **Icons**: lucide-react

### Component Structure

```
src/
├── pages/
│   ├── Home.tsx              # Landing page with feature overview
│   ├── Comparison.tsx        # Price comparison interface
│   ├── Dashboard.tsx         # Savings analytics
│   ├── Prediction.tsx        # Price prediction display
│   ├── BudgetAlerts.tsx      # Alert management
│   └── NotFound.tsx          # 404 page
├── components/
│   ├── DashboardLayout.tsx   # Main layout wrapper
│   ├── ErrorBoundary.tsx     # Error handling
│   └── ui/                   # shadcn/ui components
├── contexts/
│   └── ThemeContext.tsx      # Theme management
├── hooks/
│   └── useAuth.ts            # Authentication hook
├── lib/
│   └── trpc.ts               # tRPC client setup
├── App.tsx                   # Main app component
├── main.tsx                  # Entry point
└── index.css                 # Global styles
```

### Data Flow

1. **User Interaction**: User enters product name in search box
2. **Component State**: React component updates local state
3. **tRPC Query**: Component calls `trpc.comparison.search.useQuery()`
4. **Backend Processing**: Server processes request and fetches data
5. **Response**: Data returned to client with type safety
6. **UI Update**: React re-renders with new data
7. **Caching**: React Query caches results for performance

### Authentication Flow

```
User clicks Login
    ↓
Redirected to Manus OAuth Portal
    ↓
User authenticates
    ↓
OAuth callback to /api/oauth/callback
    ↓
Session cookie created
    ↓
User redirected to dashboard
    ↓
useAuth() hook retrieves user info
    ↓
Protected pages render
```

## Backend Architecture

### Technology Stack

- **Runtime**: Node.js with TypeScript
- **Framework**: Express.js
- **API Layer**: tRPC 11
- **Database ORM**: Drizzle ORM
- **Authentication**: Manus OAuth
- **Validation**: Zod (via tRPC)

### API Layer (tRPC)

tRPC provides end-to-end type safety for API calls. Each procedure is defined with input validation and output types.

**Procedure Types:**

- **publicProcedure**: Accessible without authentication
- **protectedProcedure**: Requires user authentication
- **adminProcedure**: Requires admin role

**Example Procedure:**

```typescript
comparison: router({
  search: protectedProcedure
    .input(z.object({ productName: z.string() }))
    .query(async ({ ctx, input }) => {
      return compareProductPrices(input.productName);
    }),
});
```

### Service Layer

**priceApi.ts** - Price fetching and comparison

- `compareProductPrices()`: Fetches prices from both platforms
- `fetchAmazonPrice()`: Calls Amazon RapidAPI endpoint
- `fetchFlipkartPrice()`: Calls Flipkart RapidAPI endpoint
- `findMatchingProduct()`: Fuzzy matching algorithm
- Fallback to mock data when APIs fail

**prediction.ts** - Price prediction algorithm

- `predictPrice()`: Generates price forecast
- `calculateTrend()`: Analyzes price direction
- `calculateConfidence()`: Determines prediction accuracy
- Uses moving averages and volatility metrics

**productDatabase.ts** - Product management

- `getProduct()`: Retrieves product by name
- `searchProducts()`: Fuzzy search across 200+ products
- `generateProductPrice()`: Creates realistic prices
- Supports 12 product categories

### Router Structure

```
appRouter
├── auth
│   ├── me (query)
│   └── logout (mutation)
├── comparison
│   ├── search (query)
│   ├── create (mutation)
│   ├── list (query)
│   └── delete (mutation)
├── dashboard
│   ├── getSavings (query)
│   └── getRecentComparisons (query)
├── prediction
│   ├── getPrediction (query)
│   └── listPredictions (query)
└── budgetAlerts
    ├── create (mutation)
    ├── list (query)
    └── delete (mutation)
```

## Database Architecture

### Database Design

The database uses a normalized schema with 8 tables optimized for read and write performance.

### Entity Relationship Diagram

```
┌─────────────────┐
│     users       │
├─────────────────┤
│ id (PK)         │
│ openId (UNIQUE) │
│ name            │
│ email           │
│ role            │
│ createdAt       │
└────────┬────────┘
         │
         ├─────────────────────┬──────────────────┬──────────────────┐
         │                     │                  │                  │
    ┌────▼──────────────┐  ┌──▼──────────────┐  ┌─▼────────────────┐
    │ priceComparisons  │  │  budgetAlerts   │  │  pricePredictions│
    ├───────────────────┤  ├─────────────────┤  ├──────────────────┤
    │ id (PK)           │  │ id (PK)         │  │ id (PK)          │
    │ userId (FK)       │  │ userId (FK)     │  │ userId (FK)      │
    │ productName       │  │ productName     │  │ productName      │
    │ amazonPrice       │  │ budgetPrice     │  │ predictedPrice   │
    │ flipkartPrice     │  │ currentPrice    │  │ trend            │
    │ amazonUrl         │  │ isActive        │  │ confidence       │
    │ flipkartUrl       │  │ createdAt       │  │ createdAt        │
    │ cheaperPlatform   │  └─────────────────┘  └──────────────────┘
    │ savingsAmount     │
    │ createdAt         │
    └───────────────────┘
         │
         ├──────────────────┬──────────────────┐
         │                  │                  │
    ┌────▼──────────────┐  ┌─▼────────────────┐
    │  priceHistory     │  │  userSavings     │
    ├───────────────────┤  ├──────────────────┤
    │ id (PK)           │  │ id (PK)          │
    │ userId (FK)       │  │ userId (FK)      │
    │ productName       │  │ totalSavings     │
    │ platform          │  │ comparisonsCount │
    │ price             │  │ lastUpdated      │
    │ fetchedAt         │  └──────────────────┘
    └───────────────────┘

┌──────────────────┐  ┌──────────────────┐
│    products      │  │ productCache     │
├──────────────────┤  ├──────────────────┤
│ id (PK)          │  │ id (PK)          │
│ productName      │  │ productName      │
│ category         │  │ amazonProductId  │
│ createdAt        │  │ flipkartProductId│
└──────────────────┘  └──────────────────┘
```

### Table Definitions

**users** - User accounts

```sql
CREATE TABLE users (
  id INT PRIMARY KEY AUTO_INCREMENT,
  openId VARCHAR(64) UNIQUE NOT NULL,
  name TEXT,
  email VARCHAR(320),
  loginMethod VARCHAR(64),
  role ENUM('user', 'admin') DEFAULT 'user',
  createdAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  lastSignedIn TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**priceComparisons** - Comparison history

```sql
CREATE TABLE priceComparisons (
  id INT PRIMARY KEY AUTO_INCREMENT,
  userId INT NOT NULL,
  productId INT,
  productName VARCHAR(255) NOT NULL,
  amazonPrice INT,
  flipkartPrice INT,
  amazonUrl TEXT,
  flipkartUrl TEXT,
  cheaperPlatform VARCHAR(50),
  savingsAmount INT,
  comparisonDate TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  createdAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (userId) REFERENCES users(id),
  INDEX idx_userId (userId),
  INDEX idx_productName (productName)
);
```

**budgetAlerts** - Price alerts

```sql
CREATE TABLE budgetAlerts (
  id INT PRIMARY KEY AUTO_INCREMENT,
  userId INT NOT NULL,
  productName VARCHAR(255) NOT NULL,
  platform VARCHAR(50) NOT NULL,
  budgetPrice INT NOT NULL,
  currentPrice INT NOT NULL,
  isActive INT DEFAULT 1,
  notificationSent INT DEFAULT 0,
  createdAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (userId) REFERENCES users(id),
  INDEX idx_userId_active (userId, isActive)
);
```

**pricePredictions** - Price forecasts

```sql
CREATE TABLE pricePredictions (
  id INT PRIMARY KEY AUTO_INCREMENT,
  userId INT NOT NULL,
  productName VARCHAR(255) NOT NULL,
  platform VARCHAR(50) NOT NULL,
  currentPrice INT NOT NULL,
  predictedPrice INT NOT NULL,
  predictedDate TIMESTAMP NOT NULL,
  trend VARCHAR(50) NOT NULL,
  confidence INT NOT NULL,
  createdAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (userId) REFERENCES users(id),
  INDEX idx_userId_productName (userId, productName)
);
```

**priceHistory** - Historical prices

```sql
CREATE TABLE priceHistory (
  id INT PRIMARY KEY AUTO_INCREMENT,
  userId INT NOT NULL,
  productName VARCHAR(255) NOT NULL,
  platform VARCHAR(50) NOT NULL,
  price INT NOT NULL,
  productId VARCHAR(255),
  fetchedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  createdAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (userId) REFERENCES users(id),
  INDEX idx_productName_platform (productName, platform),
  INDEX idx_fetchedAt (fetchedAt)
);
```

## Integration Architecture

### RapidAPI Integration

The system integrates with two RapidAPI endpoints for real-time price data:

**Amazon API** (`scout-amazon-data.p.rapidapi.com`)

- Endpoint: `/Amazon-Product-Data`
- Parameters: `asin`, `region`
- Returns: Product details, price, rating, availability

**Flipkart API** (`real-time-flipkart-data2.p.rapidapi.com`)

- Endpoint: `/product-search`
- Parameters: `q` (query), `pincode`
- Returns: Product list, prices, ratings, availability

### Authentication Integration

Manus OAuth provides secure user authentication:

1. User clicks login
2. Redirected to Manus OAuth portal
3. User authenticates with credentials
4. OAuth callback to `/api/oauth/callback`
5. Session cookie created with JWT
6. User authenticated for protected routes

### Fallback Mechanism

When external APIs are unavailable:

1. System detects API failure
2. Logs error for monitoring
3. Falls back to mock data from product database
4. Generates realistic prices based on product category
5. User experience remains seamless

## Performance Optimizations

### Caching Strategy

- **Product Cache**: Caches product metadata for 24 hours
- **Price Cache**: Caches prices for 1 hour
- **Query Cache**: React Query caches API responses
- **Database Indexing**: Optimized indexes on frequently queried columns

### Database Optimization

- **Indexes**: Composite indexes on userId and productName
- **Query Optimization**: Efficient SELECT queries with proper WHERE clauses
- **Connection Pooling**: Reuses database connections
- **Pagination**: Limits result sets for large queries

### Frontend Optimization

- **Code Splitting**: Lazy loading of page components
- **Image Optimization**: Compressed and responsive images
- **CSS Optimization**: Tailwind CSS purges unused styles
- **Lazy Loading**: Components load on demand

## Security Architecture

### Authentication & Authorization

- **OAuth 2.0**: Secure authentication via Manus
- **Session Management**: Secure cookie-based sessions
- **Role-Based Access**: Admin and user roles
- **Protected Routes**: tRPC procedures validate authentication

### Data Security

- **Input Validation**: Zod validation on all inputs
- **SQL Injection Prevention**: Parameterized queries via Drizzle ORM
- **CORS Protection**: Configured for secure cross-origin requests
- **Environment Variables**: Sensitive data in `.env` file

### API Security

- **Rate Limiting**: Prevents API abuse
- **Error Handling**: Generic error messages to prevent information leakage
- **HTTPS**: All communications encrypted
- **API Keys**: Securely stored in environment variables

## Deployment Architecture

### Manus Platform Deployment

The application is deployed on the Manus platform with:

- **Auto Scaling**: Handles traffic spikes
- **Load Balancing**: Distributes requests across servers
- **CDN**: Content delivery network for static assets
- **SSL/TLS**: Automatic HTTPS certificates
- **Database Backups**: Automated daily backups
- **Monitoring**: Real-time performance monitoring
- **Logging**: Centralized log aggregation

### Deployment Pipeline

```
Code Changes
    ↓
Create Checkpoint
    ↓
Run Tests
    ↓
Build Application
    ↓
Deploy to Staging
    ↓
Smoke Tests
    ↓
Deploy to Production
    ↓
Monitor Performance
```

## Scalability Considerations

### Horizontal Scaling

- **Stateless Backend**: Can run multiple instances
- **Database Replication**: Read replicas for scaling reads
- **CDN**: Distributes static content globally

### Vertical Scaling

- **Database Optimization**: Improved queries and indexing
- **Caching**: Reduces database load
- **API Optimization**: Efficient data fetching

### Future Scaling

- **Microservices**: Separate services for price fetching, predictions, etc.
- **Message Queues**: Async processing for price updates
- **Search Engine**: Elasticsearch for product search
- **Cache Layer**: Redis for distributed caching

## Monitoring & Observability

### Metrics Tracked

- API response times
- Database query performance
- Error rates and types
- User authentication success rates
- Price API success rates
- Cache hit rates

### Logging

- Application logs
- Database query logs
- API request/response logs
- Error stack traces
- User action audit logs

### Alerting

- High error rates
- API failures
- Database connection issues
- Performance degradation
- Unusual user activity

---

**Document Version**: 1.0

**Last Updated**: December 2024

**Architecture Status**: Production Ready ✅
