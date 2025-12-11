# Smart Online Shopping Assistant - Setup Guide

## Complete Installation and Configuration Guide

This guide provides step-by-step instructions to set up the Smart Online Shopping Assistant on your local machine or college computer.

## Prerequisites

Before starting, ensure you have the following installed:

### Required Software

- **Node.js** (v18.0.0 or higher)
  - Download from: https://nodejs.org/
  - Verify installation: `node --version`

- **npm** or **pnpm** (package manager)
  - npm comes with Node.js
  - Verify npm: `npm --version`
  - Install pnpm: `npm install -g pnpm`

- **MySQL** (v8.0 or higher)
  - Download from: https://dev.mysql.com/downloads/mysql/
  - Verify installation: `mysql --version`
  - Ensure MySQL service is running

- **Git** (optional, for version control)
  - Download from: https://git-scm.com/
  - Verify installation: `git --version`

### System Requirements

- **Operating System**: Windows, macOS, or Linux
- **RAM**: Minimum 4GB (8GB recommended)
- **Disk Space**: 2GB for project and dependencies
- **Internet Connection**: Required for API calls and npm packages

## Step 1: Obtain the Project Files

### Option A: Download from Manus Platform

1. Go to Management UI → Code panel
2. Click "Download all files" button
3. Extract the ZIP file to your desired location

### Option B: Clone from GitHub

```bash
git clone https://github.com/yourusername/smart-shopping-assistant.git
cd smart-shopping-assistant
```

### Option C: Transfer via USB Drive

1. Copy project folder to USB drive
2. Transfer to your computer
3. Extract if compressed

## Step 2: Install Dependencies

Navigate to the project directory and install all required packages:

```bash
# Navigate to project folder
cd smart-shopping-assistant

# Install dependencies using pnpm (recommended)
pnpm install

# Or using npm
npm install
```

**What this does:**

- Downloads all required npm packages
- Creates `node_modules` folder
- Generates `pnpm-lock.yaml` or `package-lock.json`
- Installs both frontend and backend dependencies

**Expected output:**

```
✔ Packages in scope: smart-shopping-assistant
✔ Lockfile is up-to-date
✔ Already up to date
Done in 2m 45s
```

## Step 3: Set Up Database

### Step 3.1: Create MySQL Database

Open MySQL command line or MySQL Workbench:

```sql
-- Create database
CREATE DATABASE shopping_assistant;

-- Create user (optional, for security)
CREATE USER 'shopping_user'@'localhost' IDENTIFIED BY 'secure_password';

-- Grant privileges
GRANT ALL PRIVILEGES ON shopping_assistant.* TO 'shopping_user'@'localhost';
FLUSH PRIVILEGES;
```

### Step 3.2: Configure Database Connection

Create a `.env` file in the project root:

```bash
# Windows
type nul > .env

# macOS/Linux
touch .env
```

Add the following content:

```env
# Database Configuration
DATABASE_URL=mysql://root:password@localhost:3306/shopping_assistant

# Or if using the user created above:
# DATABASE_URL=mysql://shopping_user:secure_password@localhost:3306/shopping_assistant
```

**Note**: Replace `root`, `password`, and `localhost` with your actual MySQL credentials.

### Step 3.3: Run Database Migrations

Execute the database schema migration:

```bash
# Using pnpm
pnpm db:push

# Or using npm
npm run db:push
```

**What this does:**

- Creates all database tables
- Sets up relationships and indexes
- Initializes the schema

**Expected output:**

```
✔ Database connection successful
✔ Creating tables...
✔ 8 tables created successfully
✔ Database ready
```

## Step 4: Configure Environment Variables

Edit the `.env` file and add the following variables:

### Essential Configuration

```env
# Application
VITE_APP_TITLE=Smart Online Shopping Assistant
VITE_APP_LOGO=https://your-logo-url.png

# Database (already added)
DATABASE_URL=mysql://root:password@localhost:3306/shopping_assistant

# Authentication
JWT_SECRET=your-secret-key-change-this-to-something-long-and-random
VITE_APP_ID=your-manus-app-id
OAUTH_SERVER_URL=https://api.manus.im
VITE_OAUTH_PORTAL_URL=https://portal.manus.im
```

### API Configuration (Optional)

If you have RapidAPI credentials:

```env
# RapidAPI Configuration
RAPIDAPI_KEY=your-rapidapi-key
FLIPKART_API_HOST=real-time-flipkart-data2.p.rapidapi.com
AMAZON_API_HOST=scout-amazon-data.p.rapidapi.com
```

**If you don't have these**, the system will use mock data automatically.

### Complete .env Template

```env
# ============================================
# Application Configuration
# ============================================
VITE_APP_TITLE=Smart Online Shopping Assistant
VITE_APP_LOGO=https://your-logo-url.png

# ============================================
# Database Configuration
# ============================================
DATABASE_URL=mysql://root:password@localhost:3306/shopping_assistant

# ============================================
# Authentication Configuration
# ============================================
JWT_SECRET=your-very-long-secret-key-at-least-32-characters-long
VITE_APP_ID=your-manus-app-id
OAUTH_SERVER_URL=https://api.manus.im
VITE_OAUTH_PORTAL_URL=https://portal.manus.im

# ============================================
# API Configuration (Optional)
# ============================================
RAPIDAPI_KEY=your-rapidapi-key-here
FLIPKART_API_HOST=real-time-flipkart-data2.p.rapidapi.com
AMAZON_API_HOST=scout-amazon-data.p.rapidapi.com

# ============================================
# Optional: Manus Built-in APIs
# ============================================
BUILT_IN_FORGE_API_URL=https://api.manus.im
BUILT_IN_FORGE_API_KEY=your-forge-api-key
```

## Step 5: Start the Development Server

Start the application:

```bash
# Using pnpm
pnpm dev

# Or using npm
npm run dev
```

**Expected output:**

```
  VITE v5.0.0  ready in 234 ms

  ➜  Local:   http://localhost:3000/
  ➜  press h to show help
```

## Step 6: Access the Application

1. Open your web browser
2. Navigate to: `http://localhost:3000`
3. You should see the Smart Online Shopping Assistant home page
4. Click "Login" to authenticate

## Troubleshooting

### Issue 1: "Cannot find module" error

**Cause**: Dependencies not installed properly

**Solution**:

```bash
# Clear node_modules
rm -rf node_modules

# Reinstall dependencies
pnpm install
```

### Issue 2: "Database connection failed" error

**Cause**: MySQL not running or credentials incorrect

**Solution**:

```bash
# Check if MySQL is running
mysql -u root -p

# If connection fails, start MySQL service:

# Windows
net start MySQL80

# macOS
brew services start mysql

# Linux
sudo systemctl start mysql
```

### Issue 3: "Port 3000 already in use" error

**Cause**: Another application is using port 3000

**Solution**:

```bash
# Option 1: Kill process on port 3000
# Windows
netstat -ano | findstr :3000
taskkill /PID <PID> /F

# macOS/Linux
lsof -ti:3000 | xargs kill -9

# Option 2: Use different port
PORT=3001 pnpm dev
```

### Issue 4: "Cannot GET /" error

**Cause**: Frontend build not generated

**Solution**:

```bash
# Rebuild frontend
pnpm build

# Or restart dev server
pnpm dev
```

### Issue 5: API keys not working

**Cause**: Invalid or missing API credentials

**Solution**:

- System automatically uses mock data if APIs fail
- To use real APIs, obtain credentials from RapidAPI
- Update `.env` file with correct keys
- Restart dev server

## Verification Checklist

After setup, verify everything is working:

- [ ] Node.js installed: `node --version`
- [ ] npm/pnpm installed: `pnpm --version`
- [ ] MySQL running: `mysql -u root -p`
- [ ] Database created: `SHOW DATABASES;`
- [ ] `.env` file created with all variables
- [ ] Dependencies installed: `node_modules` folder exists
- [ ] Dev server running: `http://localhost:3000` accessible
- [ ] Login page loads
- [ ] Can search for products
- [ ] Dashboard displays data

## Development Commands

### Common Commands

```bash
# Start development server
pnpm dev

# Build for production
pnpm build

# Run tests
pnpm test

# Database migrations
pnpm db:push

# Generate database types
pnpm db:generate

# Format code
pnpm format

# Lint code
pnpm lint
```

### Project Structure Commands

```bash
# View project structure
tree -L 2

# Count lines of code
find . -name "*.ts" -o -name "*.tsx" | xargs wc -l

# Check dependencies
pnpm list
```

## Deployment to College Computer

### Via USB Drive

1. **On Development Machine**:
   - Complete all setup steps
   - Create `.env` file with all variables
   - Run `pnpm build`
   - Copy entire project folder to USB drive

2. **On College Computer**:
   - Copy project from USB drive
   - Install Node.js and MySQL
   - Follow Steps 2-6 above
   - Run `pnpm dev`

### Via Git

1. **Push to GitHub**:

```bash
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/yourusername/smart-shopping-assistant.git
git push -u origin main
```

2. **On College Computer**:

```bash
git clone https://github.com/yourusername/smart-shopping-assistant.git
cd smart-shopping-assistant
pnpm install
# Configure .env
pnpm db:push
pnpm dev
```

### Via Email/Cloud Storage

1. Compress project: `zip -r smart-shopping-assistant.zip smart-shopping-assistant/`
2. Upload to Google Drive, OneDrive, or email
3. Download on college computer
4. Extract and follow Steps 2-6

## Performance Tips

### Optimize Development Experience

```bash
# Use pnpm for faster installations
npm install -g pnpm

# Enable caching
pnpm config set store-dir ~/.pnpm-store

# Use faster database queries
# Add indexes to frequently queried columns
```

### Optimize Database

```sql
-- Create indexes for faster queries
CREATE INDEX idx_userId ON priceComparisons(userId);
CREATE INDEX idx_productName ON priceComparisons(productName);
CREATE INDEX idx_platform ON priceHistory(platform);
```

## Security Best Practices

1. **Never commit `.env` file** to Git
2. **Use strong JWT_SECRET** (at least 32 characters)
3. **Change default passwords** for MySQL
4. **Keep dependencies updated**: `pnpm update`
5. **Use HTTPS** in production
6. **Validate all user inputs**
7. **Sanitize database queries**

## Next Steps

After successful setup:

1. **Explore the Application**:
   - Test price comparison
   - Create budget alerts
   - View predictions

2. **Customize**:
   - Update `VITE_APP_TITLE` and logo
   - Add more products to database
   - Customize colors in `index.css`

3. **Deploy**:
   - Run `pnpm build`
   - Deploy to hosting platform
   - Configure custom domain

4. **Monitor**:
   - Check application logs
   - Monitor database performance
   - Track API usage

## Support & Resources

- **Documentation**: See README.md, ARCHITECTURE.md, FEATURES.md
- **GitHub Issues**: Report bugs and request features
- **Stack Overflow**: Search for common issues
- **Official Docs**: https://nodejs.org/, https://dev.mysql.com/

---

**Document Version**: 1.0

**Last Updated**: December 2024

**Setup Status**: Ready for Production ✅
