# Smart Online Shopping Assistant - User Guide

## Website Overview

**Purpose:** Compare prices between Amazon and Flipkart, track your savings, and predict future product prices to make smarter shopping decisions.

**Access:** Login required. Sign in with your Manus account to get started.

---

## Powered by Manus

This application is built with a cutting-edge technology stack designed for performance and reliability:

**Frontend:** React 19 with TypeScript and Tailwind CSS 4 for a responsive, modern user interface

**Backend:** Express.js with tRPC for type-safe API communication

**Database:** MySQL with Drizzle ORM for robust data management

**Authentication:** Manus OAuth integration for secure user authentication

**Deployment:** Auto-scaling infrastructure with global CDN for lightning-fast performance worldwide

---

## Using Your Website

### 1. Price Comparison

Compare product prices between Amazon and Flipkart to find the best deals:

1. Click "Compare Now" on the homepage or navigate to the "Compare" page
2. Enter the product name (e.g., "iPhone 15 Pro")
3. Fill in the current price on Amazon and Flipkart
4. Optionally add product URLs for quick reference
5. Click "Add Comparison" to save

The system automatically calculates which platform is cheaper and how much you save. Your comparison history is displayed below, showing all past comparisons with savings amounts.

### 2. Dashboard

View your total savings and shopping analytics:

1. Click "Dashboard" in the navigation menu
2. See your "Total Savings" - the cumulative amount saved using the platform
3. View "Comparisons Made" - total number of price comparisons you've performed
4. Check "Average Savings Per Comparison" - how much you save on average per comparison
5. Review "Recent Comparisons" table showing your last 5 comparisons with details

The dashboard updates automatically as you add new comparisons, giving you real-time insights into your savings.

### 3. Price Prediction

Get AI-powered predictions for future product prices:

1. Click "Predict" in the navigation menu
2. Enter the product name and select the platform (Amazon or Flipkart)
3. Enter the current price and your predicted price
4. Select the predicted date when you expect this price
5. Choose the price trend: "Going Up", "Going Down", or "Stable"
6. Set your confidence level (0-100%) in this prediction
7. Click "Add Prediction" to save

Your predictions are displayed as cards showing the current price, predicted price, trend indicator (↑ ↓ →), and confidence level with a visual progress bar.

---

## Managing Your Website

### Settings & Configuration

Access the Management UI by clicking the settings icon in the top right corner:

**General Settings:** Customize your website name and logo

**Database:** View and manage your stored comparisons and predictions

**Secrets:** Manage any API keys or credentials (if applicable)

**Notifications:** Configure notification preferences for important updates

---

## Next Steps

Talk to Manus AI anytime to request changes or add features to your shopping assistant. You can enhance the platform with features like:

- Real-time price tracking across multiple products
- Automatic price drop alerts
- Integration with more e-commerce platforms
- Advanced analytics and spending reports
- Mobile app for on-the-go shopping

Start comparing prices now and begin saving money on every purchase. Click "Compare Now" to make your first price comparison today!

---

## Tips for Best Results

**For Comparisons:** Always verify prices on both platforms before making a purchase, as prices update frequently.

**For Predictions:** Base your predictions on historical price trends and seasonal sales patterns for better accuracy.

**For Savings Tracking:** Regularly check your dashboard to monitor your total savings and stay motivated to find better deals.


## Real-Time API Integration

### How It Works

Your Smart Online Shopping Assistant now integrates with real-time price APIs from RapidAPI:

**Amazon Integration:** Uses the Scout Amazon Data API to fetch real product prices, ratings, availability, and product images from Amazon India.

**Flipkart Integration:** Uses the Real-time Flipkart Data API to fetch live prices, ratings, and availability information from Flipkart.

### Supported Products

The system supports searching for any product, including:
- Electronics (iPhones, Samsung Galaxy, laptops)
- Audio devices (AirPods, headphones)
- Cameras and photography equipment
- Smart home devices
- And thousands more!

### How Predictions Work

The trend-based prediction algorithm analyzes:
- **Historical Price Data:** Tracks price changes over the last 30 days
- **Moving Averages:** Calculates 7-day and 14-day moving averages to identify trends
- **Volatility:** Measures price stability to adjust confidence levels
- **Trend Direction:** Determines if prices are going up, down, or staying stable

The system then predicts future prices and provides a confidence percentage (0-100%) indicating how reliable the prediction is.

### API Limits

Be aware of RapidAPI rate limits on your subscription plan:
- Free tier: Limited requests per month
- Paid tier: Higher request limits

The system includes caching to minimize API calls and respect rate limits.

### Troubleshooting

**No prices showing?**
- Verify your RapidAPI key is configured correctly
- Check if the product name matches available products on the platforms
- Ensure you have remaining API quota for the month

**Predictions not available?**
- The product needs at least a few price comparisons to generate predictions
- Try comparing the product multiple times to build price history
- Wait a few moments for the system to calculate trends

### Future Enhancements

Planned features include:
- Real-time price drop notifications
- Integration with more e-commerce platforms
- Advanced ML-based price prediction
- Price history charts and analytics
- Wishlist and price alert system
