# Recruiters-Job-Posting-Data-


from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.options import Options
from bs4 import BeautifulSoup as bs
import pandas as pd
import time

# Setting up Chrome options
chrome_options = Options()
chrome_options.add_argument("--start-maximized")  # Start browser maximized
chrome_options.add_argument("--disable-blink-features=AutomationControlled")
chrome_options.add_argument("--headless")  # Run in headless mode for efficiency

# Specifying the path to chromedriver
service = Service("chromedriver.exe")
driver = webdriver.Chrome(service=service, options=chrome_options)

data_list = []

# Loop through pages 11 to 12
for page in range(1, 1):
    #url = f'https://www.daraz.pk/womens-socks-stockings/?page={page}spm=a2a0e.searchlistcategory.cate_8_4.3.1d094eadEYvvVn'
    url = f'https://www.daraz.pk/catalog/?_keyori=ss&from=search_history&page=102&q=Storage%20Bag&spm=a2a0e.searchlistcategory.search.4.337514a2D3VMqP&sugg=Storage%20Bag_2_1'
    driver.get(url)
    time.sleep(5)  # Allow time for page to load

    # Get page source and parse with BeautifulSoup
    page_source = driver.page_source
    soup = bs(page_source, 'html.parser')

    # Find product blocks
    blocks = soup.find_all('div', {'class': 'Bm3ON'})

    for data in blocks:
        product_name_element = data.find('div', {'class': 'RfADt'})
        product_link_element = data.find('a', href=True)
        image_link_element = data.find('img', src=True)
        product_price_element = data.find('span', {'class': 'ooOxS'})
        product_review_element = data.find('div', {'class': '_6uN7R'})  # Find correct class for reviews    
        seller_location_element = data.find('span', {'class': 'oa6ri'})

        # Extract text or default to 'Not Found' if element is None
        product_name = product_name_element.get_text(strip=True) if product_name_element else 'Not Found'
        product_link = product_link_element['href'] if product_link_element else 'Not Found'
        image_link = image_link_element['src'] if image_link_element else 'Not Found'
        product_price = product_price_element.get_text(strip=True) if product_price_element else 'Not Found'
        product_review = product_review_element.get_text(strip=True) if product_review_element else 'Not Found'
        seller_location = seller_location_element.get_text(strip=True) if seller_location_element else 'Not Found'

        # Append data to the list
        data_list.append({
            'Product Name': product_name,
            'Product Link': product_link,
            'Image Link': image_link,
            'Product Price': product_price,
            'Product Review': product_review,
            'Seller Location': seller_location
        })

# Create a DataFrame from the list of dictionaries
df = pd.DataFrame(data_list)

# Save DataFrame to CSV file
df.to_csv('darazstoragebags.csv', index=False)

# Closing the webdriver
driver.quit()

print("Scraping completed and data saved to daraz_health_beauty_tools.csv.")
