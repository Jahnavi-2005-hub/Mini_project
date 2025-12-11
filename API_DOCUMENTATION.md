# Smart Online Shopping Assistant - API Documentation

## API Overview

The Smart Online Shopping Assistant uses **tRPC** (TypeScript RPC) for end-to-end type-safe API communication. All API procedures are defined in `server/routers.ts` and consumed from the frontend using tRPC hooks.

## What is tRPC?

tRPC is a TypeScript-first framework for building type-safe APIs. It provides:

- **Type Safety**: Full TypeScript support from client to server
- **Auto-completion**: IDE autocomplete for all API calls
- **No Code Generation**: Types are inferred automatically
- **Validation**: Input validation using Zod schemas
- **Error Handling**: Type-safe error responses

## API Base URL

```
http://localhost:3000/api/trpc
```

## Authentication

All protected procedures require user authentication via Manus OAuth. The authentication context is automatically injected into procedures.

### Authentication Flow

```typescript
// Public procedure (no auth required)
publicProcedure.query(async ({ ctx }) => {
  // ctx.user is undefined
});

// Protected procedure (auth required)
protectedProcedure.query(async ({ ctx }) => {
  // ctx.user is available and contains user info
  // Throws error if not authenticated
});
```

## API Procedures

### 1. Authentication Procedures

#### `auth.me`

Get current authenticated user information.

**Type**: Query (Read-only)

**Authentication**: Public

**Input**: None

**Output**:

```typescript
{
  id: number;
  openId: string;
  name: string | null;
  email: string | null;
  role: 'user' | 'admin';
  createdAt: Date;
  lastSignedIn: Date;
} | null
```

**Example Usage**:

```typescript
const { data: user } = trpc.auth.me.useQuery();

if (user) {
  console.log(`Logged in as: ${user.name}`);
}
```

#### `auth.logout`

Logout the current user and clear session.

**Type**: Mutation (Write)

**Authentication**: Public

**Input**: None

**Output**:

```typescript
{ success: boolean }
```

**Example Usage**:

```typescript
const logout = trpc.auth.logout.useMutation();

logout.mutate(undefined, {
  onSuccess: () => {
    window.location.href = '/';
  },
});
```

---

### 2. Comparison Procedures

#### `comparison.search`

Search for a product and compare prices between Amazon and Flipkart.

**Type**: Query (Read-only)

**Authentication**: Protected

**Input**:

```typescript
{
  productName: string;  // Product to search for
}
```

**Output**:

```typescript
{
  productName: string;
  amazon?: {
    productId: string;
    productName: string;
    price: number;
    currency: string;
    url?: string;
    rating?: number;
    availability?: boolean;
  };
  flipkart?: {
    productId: string;
    productName: string;
    price: number;
    currency: string;
    url?: string;
    rating?: number;
    availability?: boolean;
  };
  cheaper?: 'amazon' | 'flipkart';
  savings?: number;
}
```

**Example Usage**:

```typescript
const { data: comparison, isLoading } = trpc.comparison.search.useQuery(
  { productName: 'iPhone 15 Pro' },
  { enabled: productName.length > 0 }
);

if (comparison) {
  console.log(`${comparison.cheaper} is cheaper by ₹${comparison.savings}`);
}
```

#### `comparison.create`

Save a price comparison to the database.

**Type**: Mutation (Write)

**Authentication**: Protected

**Input**:

```typescript
{
  productName: string;
  amazonPrice: number;
  flipkartPrice: number;
  amazonUrl?: string;
  flipkartUrl?: string;
  cheaperPlatform: 'amazon' | 'flipkart';
  savingsAmount: number;
}
```

**Output**:

```typescript
{
  id: number;
  userId: number;
  productName: string;
  amazonPrice: number;
  flipkartPrice: number;
  cheaperPlatform: string;
  savingsAmount: number;
  createdAt: Date;
}
```

**Example Usage**:

```typescript
const createComparison = trpc.comparison.create.useMutation();

createComparison.mutate(
  {
    productName: 'iPhone 15 Pro',
    amazonPrice: 99999,
    flipkartPrice: 104999,
    cheaperPlatform: 'amazon',
    savingsAmount: 5000,
  },
  {
    onSuccess: (result) => {
      console.log('Comparison saved:', result.id);
    },
  }
);
```

#### `comparison.list`

Get all comparisons made by the current user.

**Type**: Query (Read-only)

**Authentication**: Protected

**Input**: None

**Output**:

```typescript
Array<{
  id: number;
  productName: string;
  amazonPrice: number;
  flipkartPrice: number;
  cheaperPlatform: string;
  savingsAmount: number;
  createdAt: Date;
}>
```

**Example Usage**:

```typescript
const { data: comparisons } = trpc.comparison.list.useQuery();

comparisons?.forEach((comp) => {
  console.log(`${comp.productName}: Saved ₹${comp.savingsAmount}`);
});
```

#### `comparison.delete`

Delete a specific comparison.

**Type**: Mutation (Write)

**Authentication**: Protected

**Input**:

```typescript
{ id: number }
```

**Output**:

```typescript
{ success: boolean }
```

**Example Usage**:

```typescript
const deleteComparison = trpc.comparison.delete.useMutation();

deleteComparison.mutate({ id: 123 });
```

---

### 3. Dashboard Procedures

#### `dashboard.getSavings`

Get savings statistics for the current user.

**Type**: Query (Read-only)

**Authentication**: Protected

**Input**: None

**Output**:

```typescript
{
  totalSavings: number;        // In paise (₹)
  comparisonsCount: number;
  averageSavingsPerComparison: number;
  lastUpdated: Date;
}
```

**Example Usage**:

```typescript
const { data: savings } = trpc.dashboard.getSavings.useQuery();

if (savings) {
  console.log(`Total Savings: ₹${savings.totalSavings / 100}`);
  console.log(`Comparisons: ${savings.comparisonsCount}`);
}
```

#### `dashboard.getRecentComparisons`

Get the most recent comparisons made by the user.

**Type**: Query (Read-only)

**Authentication**: Protected

**Input**:

```typescript
{
  limit?: number;  // Default: 10
}
```

**Output**:

```typescript
Array<{
  id: number;
  productName: string;
  amazonPrice: number;
  flipkartPrice: number;
  cheaperPlatform: string;
  savingsAmount: number;
  createdAt: Date;
}>
```

**Example Usage**:

```typescript
const { data: recent } = trpc.dashboard.getRecentComparisons.useQuery(
  { limit: 5 }
);

recent?.forEach((comp) => {
  console.log(`${comp.productName}: ${comp.createdAt}`);
});
```

---

### 4. Prediction Procedures

#### `prediction.getPrediction`

Get AI-powered price prediction for a product.

**Type**: Query (Read-only)

**Authentication**: Protected

**Input**:

```typescript
{
  productName: string;
  platform: 'amazon' | 'flipkart';
}
```

**Output**:

```typescript
{
  productName: string;
  platform: string;
  currentPrice: number;
  predictedPrice: number;
  predictedDate: Date;
  trend: 'up' | 'down' | 'stable';
  confidence: number;  // 0-100
  reasoning: string;
}
```

**Example Usage**:

```typescript
const { data: prediction } = trpc.prediction.getPrediction.useQuery({
  productName: 'iPhone 15 Pro',
  platform: 'amazon',
});

if (prediction) {
  console.log(`Predicted Price: ₹${prediction.predictedPrice}`);
  console.log(`Confidence: ${prediction.confidence}%`);
  console.log(`Trend: ${prediction.trend}`);
}
```

#### `prediction.listPredictions`

Get all price predictions for the current user.

**Type**: Query (Read-only)

**Authentication**: Protected

**Input**: None

**Output**:

```typescript
Array<{
  id: number;
  productName: string;
  platform: string;
  currentPrice: number;
  predictedPrice: number;
  trend: string;
  confidence: number;
  createdAt: Date;
}>
```

**Example Usage**:

```typescript
const { data: predictions } = trpc.prediction.listPredictions.useQuery();

predictions?.forEach((pred) => {
  console.log(`${pred.productName}: ${pred.trend} trend`);
});
```

---

### 5. Budget Alert Procedures

#### `budgetAlerts.create`

Create a new budget price alert.

**Type**: Mutation (Write)

**Authentication**: Protected

**Input**:

```typescript
{
  productName: string;
  platform: 'amazon' | 'flipkart' | 'both';
  budgetPrice: number;
  currentPrice: number;
}
```

**Output**:

```typescript
{
  id: number;
  userId: number;
  productName: string;
  platform: string;
  budgetPrice: number;
  currentPrice: number;
  isActive: boolean;
  createdAt: Date;
}
```

**Example Usage**:

```typescript
const createAlert = trpc.budgetAlerts.create.useMutation();

createAlert.mutate({
  productName: 'iPhone 15 Pro',
  platform: 'amazon',
  budgetPrice: 95000,
  currentPrice: 99999,
});
```

#### `budgetAlerts.list`

Get all budget alerts for the current user.

**Type**: Query (Read-only)

**Authentication**: Protected

**Input**: None

**Output**:

```typescript
Array<{
  id: number;
  productName: string;
  platform: string;
  budgetPrice: number;
  currentPrice: number;
  isActive: boolean;
  notificationSent: boolean;
  createdAt: Date;
}>
```

**Example Usage**:

```typescript
const { data: alerts } = trpc.budgetAlerts.list.useQuery();

alerts?.forEach((alert) => {
  const dropNeeded = alert.currentPrice - alert.budgetPrice;
  console.log(`${alert.productName}: Need ₹${dropNeeded} drop`);
});
```

#### `budgetAlerts.delete`

Delete a budget alert.

**Type**: Mutation (Write)

**Authentication**: Protected

**Input**:

```typescript
{ id: number }
```

**Output**:

```typescript
{ success: boolean }
```

**Example Usage**:

```typescript
const deleteAlert = trpc.budgetAlerts.delete.useMutation();

deleteAlert.mutate({ id: 456 });
```

---

## Error Handling

### Error Types

tRPC returns standardized error responses:

```typescript
{
  code: 'UNAUTHORIZED' | 'FORBIDDEN' | 'NOT_FOUND' | 'BAD_REQUEST' | 'INTERNAL_SERVER_ERROR';
  message: string;
  data?: {
    code: string;
    httpStatus: number;
  };
}
```

### Error Handling Example

```typescript
const { data, error, isLoading } = trpc.comparison.search.useQuery(
  { productName: 'iPhone' },
  {
    onError: (err) => {
      if (err.code === 'UNAUTHORIZED') {
        console.log('Please login first');
      } else if (err.code === 'BAD_REQUEST') {
        console.log('Invalid input:', err.message);
      } else {
        console.log('Error:', err.message);
      }
    },
  }
);
```

## Rate Limiting

API calls are rate-limited to prevent abuse:

- **Public endpoints**: 100 requests per minute
- **Protected endpoints**: 1000 requests per minute
- **Rate limit headers**: Included in response

## Pagination

Large result sets support pagination:

```typescript
// Example with pagination
const { data: comparisons } = trpc.comparison.list.useQuery({
  limit: 10,
  offset: 0,
});
```

## Caching

tRPC uses React Query for automatic caching:

```typescript
// Data is cached by default
const query1 = trpc.comparison.search.useQuery({ productName: 'iPhone' });
const query2 = trpc.comparison.search.useQuery({ productName: 'iPhone' });
// query2 uses cached data from query1

// Invalidate cache and refetch
const utils = trpc.useUtils();
utils.comparison.search.invalidate();
```

## WebSocket Support

Real-time updates via WebSocket (planned feature):

```typescript
// Future: Real-time price updates
trpc.priceUpdates.subscribe({ productName: 'iPhone' }, {
  onData: (price) => {
    console.log('Price updated:', price);
  },
});
```

## API Response Examples

### Successful Comparison Query

```json
{
  "productName": "iPhone 15 Pro",
  "amazon": {
    "productId": "B0CL6KQFSY",
    "productName": "iPhone 15 Pro",
    "price": 99999,
    "currency": "INR",
    "url": "https://www.amazon.in/s?k=iPhone+15+Pro",
    "rating": 4.7,
    "availability": true
  },
  "flipkart": {
    "productId": "FK-IPHONE15PRO",
    "productName": "iPhone 15 Pro",
    "price": 104999,
    "currency": "INR",
    "url": "https://www.flipkart.com/search?q=iPhone+15+Pro",
    "rating": 4.5,
    "availability": true
  },
  "cheaper": "amazon",
  "savings": 5000
}
```

### Successful Prediction Query

```json
{
  "productName": "iPhone 15 Pro",
  "platform": "amazon",
  "currentPrice": 99999,
  "predictedPrice": 97999,
  "predictedDate": "2024-12-17T00:00:00Z",
  "trend": "down",
  "confidence": 78,
  "reasoning": "Price has been declining over the past 7 days. Expected to drop by ₹2,000 in the next week."
}
```

### Error Response

```json
{
  "code": "BAD_REQUEST",
  "message": "Product not found in database",
  "data": {
    "code": "PRODUCT_NOT_FOUND",
    "httpStatus": 400
  }
}
```

## Testing API Endpoints

### Using cURL

```bash
# Get current user
curl -X GET http://localhost:3000/api/trpc/auth.me

# Search for product
curl -X GET "http://localhost:3000/api/trpc/comparison.search?input=%7B%22productName%22:%22iPhone%22%7D"
```

### Using Postman

1. Create new request
2. Set method to GET or POST
3. URL: `http://localhost:3000/api/trpc/[procedure-name]`
4. Add input as query parameter or body
5. Send request

### Using tRPC Client

```typescript
import { trpc } from '@/lib/trpc';

// Query
const { data } = await trpc.comparison.search.useQuery({
  productName: 'iPhone',
});

// Mutation
await trpc.comparison.create.useMutation().mutateAsync({
  productName: 'iPhone',
  amazonPrice: 99999,
  flipkartPrice: 104999,
  cheaperPlatform: 'amazon',
  savingsAmount: 5000,
});
```

## Best Practices

1. **Always handle errors**: Use `onError` callbacks
2. **Use loading states**: Show spinners while fetching
3. **Invalidate cache**: After mutations, invalidate related queries
4. **Optimize queries**: Use `enabled` to skip unnecessary queries
5. **Type safety**: Leverage TypeScript for autocomplete
6. **Error messages**: Show user-friendly error messages

## API Limits

| Metric | Limit |
|--------|-------|
| Max request size | 10MB |
| Max response size | 50MB |
| Request timeout | 30 seconds |
| Concurrent requests | 10 per user |
| Daily API calls | 10,000 |

## Deprecation Policy

Deprecated endpoints will be marked with `@deprecated` and supported for 6 months before removal.

## Support

For API issues:

- Check error message and code
- Review this documentation
- Check GitHub issues
- Contact support

---

**Document Version**: 1.0

**Last Updated**: December 2024

**API Status**: Production Ready ✅
