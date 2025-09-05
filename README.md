# Install dependencies (only needed once per Colab runtime)
!pip install requests beautifulsoup4 lxml seaborn

# Import libraries
import requests
from bs4 import BeautifulSoup
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import re

# Base URL of the site
BASE_URL = "http://books.toscrape.com/catalogue/page-{}.html"

# Containers for scraped data
titles, prices, availability, ratings = [], [], [], []

print("Scraping data from Books to Scrape...")

# The site has 50 pages of books
for page in range(1, 51):
    url = BASE_URL.format(page)
    response = requests.get(url)
    if response.status_code != 200:
        print(f"Failed to fetch page {page}")
        continue

    soup = BeautifulSoup(response.text, "lxml")
    books = soup.find_all("article", class_="product_pod")

    for book in books:
        # Title
        title = book.h3.a["title"]
        titles.append(title)

        # Price (extract numbers only using regex)
        price_text = book.find("p", class_="price_color").text.strip()
        price_match = re.findall(r"[\d\.]+", price_text)
        price = float(price_match[0]) if price_match else None
        prices.append(price)

        # Availability
        avail = book.find("p", class_="instock availability").text.strip()
        availability.append(avail)

        # Rating (class contains star rating)
        rating_class = book.find("p", class_="star-rating")["class"]
        rating = rating_class[1] if len(rating_class) > 1 else None
        ratings.append(rating)

print("Scraping complete.")

# Create DataFrame
df = pd.DataFrame({
    "Title": titles,
    "Price (£)": prices,
    "Availability": availability,
    "Rating": ratings
})

# Handle missing values
df["Price (£)"].fillna(df["Price (£)"].mean(), inplace=True)
df["Rating"].fillna("Unknown", inplace=True)

# Save to CSV (downloadable in Colab)
df.to_csv("books_dataset.csv", index=False)
print("Cleaned dataset saved to 'books_dataset.csv'.")

# Show first few rows
df.head()
# Compute average price
avg_price = df["Price (£)"].mean()
print(f"\nAverage book price: £{avg_price:.2f}")

sns.set_style("whitegrid")

# Histogram of prices
plt.figure(figsize=(10, 6))
sns.histplot(df["Price (£)"], bins=20, kde=True, color="skyblue")
plt.title("Distribution of Book Prices", fontsize=16)
plt.xlabel("Price (£)")
plt.ylabel("Number of Books")
plt.show()

# Count of ratings
plt.figure(figsize=(8, 6))
sns.countplot(x="Rating", data=df, order=df["Rating"].value_counts().index, palette="viridis")
plt.title("Number of Books by Rating", fontsize=16)
plt.xlabel("Rating")
plt.ylabel("Count")
plt.show()

# Boxplot for outliers
plt.figure(figsize=(8, 6))
sns.boxplot(x=df["Price (£)"], color="orange")
plt.title("Boxplot of Book Prices (Outlier Detection)", fontsize=16)
plt.xlabel("Price (£)")
plt.show()
