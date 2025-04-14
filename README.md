import tkinter as tk
from tkinter import ttk, messagebox
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.chrome.options import Options
from webdriver_manager.chrome import ChromeDriverManager
from bs4 import BeautifulSoup
import requests
import time

# ========= Web Scrapers =========

def get_driver():
    options = Options()
    options.add_argument("--headless")
    options.add_argument("--disable-gpu")
    options.add_argument("--no-sandbox")
    return webdriver.Chrome(service=Service(ChromeDriverManager().install()), options=options)

def scrape_amazon(url):
    headers = {"User-Agent": "Mozilla/5.0"}
    r = requests.get(url, headers=headers)
    soup = BeautifulSoup(r.content, "lxml")
    price_tag = soup.select_one("#priceblock_ourprice") or soup.select_one("#priceblock_dealprice")
    if price_tag:
        return float(price_tag.text.replace("₹", "").replace(",", "").strip())
    return None

def scrape_flipkart(url):
    headers = {"User-Agent": "Mozilla/5.0"}
    r = requests.get(url, headers=headers)
    soup = BeautifulSoup(r.content, "lxml")
    price_tag = soup.select_one("._30jeq3")
    if price_tag:
        return float(price_tag.text.replace("₹", "").replace(",", "").strip())
    return None

def scrape_bigbasket(url):
    driver = get_driver()
    driver.get(url)
    time.sleep(3)
    try:
        price = driver.find_element("css selector", ".nk-price-final").text
        driver.quit()
        return float(price.replace("₹", "").replace(",", "").strip())
    except:
        driver.quit()
        return None

def scrape_blinkit(url):
    driver = get_driver()
    driver.get(url)
    time.sleep(3)
    try:
        price = driver.find_element("css selector", "span[class*='Price']").text
        driver.quit()
        return float(price.replace("₹", "").replace(",", "").strip())
    except:
        driver.quit()
        return None

# ========= Router =========

def get_price_by_url(url):
    if "amazon" in url:
        return scrape_amazon(url)
    elif "flipkart" in url:
        return scrape_flipkart(url)
    elif "bigbasket" in url:
        return scrape_bigbasket(url)
    elif "blinkit" in url:
        return scrape_blinkit(url)
    else:
        return None

# ========= GUI =========

def compare_prices():
    output_box.delete(*output_box.get_children())
    urls = [entry.get().strip() for entry in url_entries if entry.get().strip()]
    if not urls:
        messagebox.showwarning("Input Error", "Please enter at least one URL.")
        return

    store_prices = {}
    for url in urls:
        store = url.split("//")[1].split(".")[0].capitalize()
        price = get_price_by_url(url)
        store_prices[store] = price
        output_box.insert("", "end", values=(store, f"₹{price}" if price else "Not Found"))

    valid_prices = {k: v for k, v in store_prices.items() if v}
    if valid_prices:
        cheapest = min(valid_prices, key=valid_prices.get)
        messagebox.showinfo("Cheapest Option", f"{cheapest} is cheapest at ₹{valid_prices[cheapest]}")
    else:
        messagebox.showerror("No Prices Found", "Could not fetch any prices.")

# ========= UI Layout =========

app = tk.Tk()
app.title("🛒 Grocery Price Comparison Tool")
app.geometry("600x500")

tk.Label(app, text="Enter Product URLs (Amazon, Flipkart, BigBasket, Blinkit):", font=("Arial", 12)).pack(pady=10)

frame = tk.Frame(app)
frame.pack()

url_entries = []
for _ in range(5):  # 5 input boxes
    e = tk.Entry(frame, width=80)
    e.pack(pady=5)
    url_entries.append(e)

tk.Button(app, text="Compare Prices", command=compare_prices, font=("Arial", 12, "bold"), bg="green", fg="white").pack(pady=15)

output_box = ttk.Treeview(app, columns=("Store", "Price"), show="headings", height=8)
output_box.heading("Store", text="Store")
output_box.heading("Price", text="Price")
output_box.pack(pady=10)

app.mainloop()
Add main.py - GUI grocery price comparison script
