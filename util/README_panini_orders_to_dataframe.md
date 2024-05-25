# Web Scraping Panini Orders with Selenium and Pandas

This script automates the process of logging into the Panini website, navigating to the order history page, extracting order information, and saving it as a CSV file using Selenium and Pandas.

## Requirements

- Python 3.x
- Selenium
- Pandas
- ChromeDriver
- Google Chrome browser
- Environment variables for username and password

## Setup

1. **Install Required Packages:**

   ```bash
   pip install selenium pandas
   ```

2. **Download ChromeDriver:**

   Download ChromeDriver from [here](https://sites.google.com/a/chromium.org/chromedriver/downloads) and place it in a directory included in your system's PATH or specify its path directly in the script.

3. **Set Environment Variables:**

   Set the environment variables for your Panini account's username and password:

   ```bash
   export USERNAME="your_username"
   export PASSWORD="your_password"
   ```

## Script Overview

The script is organized into several functions for better readability and maintainability.

### Imports

```python
import pandas as pd
import re
import time
import os
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from datetime import datetime
```

### Functions

#### `setup_driver()`

Sets up the Chrome WebDriver.

```python
def setup_driver():
    service = Service(executable_path="chromedriver.exe")
    driver = webdriver.Chrome(service=service)
    return driver
```

#### `accept_cookies(driver)`

Accepts cookies on the Panini website.

```python
def accept_cookies(driver):
    try:
        wait = WebDriverWait(driver, 20)
        element = wait.until(EC.element_to_be_clickable((By.CLASS_NAME, 'amgdprcookie-button')))
        buttons = driver.find_elements(By.CLASS_NAME, 'amgdprcookie-button')
        for btn in buttons:
            if '-settings' in btn.get_attribute('class') and '-save' in btn.get_attribute('class'):
                btn.click()
                break
        else:
            print("Cookies button not found.")
    except Exception as e:
        print(f"Error finding element: {e}")
```

#### `login(driver, username, password)`

Logs into the Panini website.

```python
def login(driver, username, password):
    try:
        wait = WebDriverWait(driver, 20)
        username_field = wait.until(EC.visibility_of_element_located((By.ID, 'gigya-loginID-20503411558167264')))
        password_field = wait.until(EC.visibility_of_element_located((By.ID, 'gigya-password-102120096138430450')))
        username_field.send_keys(username)
        password_field.send_keys(password)
        password_field.send_keys(Keys.RETURN)
    except Exception as e:
        print(f"Error finding element: {e}")
```

#### `get_order_ids(driver)`

Retrieves all order IDs from the order history table.

```python
def get_order_ids(driver):
    order_ids = []
    try:
        wait = WebDriverWait(driver, 20)
        wait.until(EC.visibility_of_element_located((By.ID, 'my-orders-table')))
        orders = driver.find_elements(By.CSS_SELECTOR, '#my-orders-table tbody tr')
        for order in orders:
            order_id_element = order.find_element(By.CSS_SELECTOR, 'td.col.id span')
            order_ids.append(order_id_element.text)
    except Exception as e:
        print(f"Error finding order IDs: {e}")
    return order_ids
```

#### `find_and_click_order(driver, target_order_id)`

Finds and clicks on a specific order ID to view its details.

```python
def find_and_click_order(driver, target_order_id):
    try:
        wait = WebDriverWait(driver, 20)
        wait.until(EC.visibility_of_element_located((By.ID, 'my-orders-table')))
        orders = driver.find_elements(By.CSS_SELECTOR, '#my-orders-table tbody tr')
        for order in orders:
            order_id_element = order.find_element(By.CSS_SELECTOR, 'td.col.id span')
            if order_id_element.text == target_order_id:
                view_link = order.find_element(By.CSS_SELECTOR, 'a.action.view')
                view_link.click()
                break
    except Exception as e:
        print(f"Error finding or clicking order: {e}")
```

#### `extract_volume(input_string)`

Extracts the volume number from a product title.

```python
def extract_volume(input_string):
    volume_match = re.search(r"Vol\. (\d+)", input_string)
    return volume_match.group(1) if volume_match else None
```

#### `get_order_info_value(order_info, search_string)`

Extracts specific order information based on a search string.

```python
def get_order_info_value(order_info, search_string):
    text = order_info.text
    value_match = re.search(f"{search_string}: (.+)", text)
    return value_match.group(1)
```

#### `convert_date_pt_to_en(date_str)`

Converts a date string from Portuguese to English format.

```python
def convert_date_pt_to_en(date_str):
    months_pt_to_en = {
        "janeiro": "January",
        "fevereiro": "February",
        "março": "March",
        "abril": "April",
        "maio": "May",
        "junho": "June",
        "julho": "July",
        "agosto": "August",
        "setembro": "September",
        "outubro": "October",
        "novembro": "November",
        "dezembro": "December"
    }
    for pt_month, en_month in months_pt_to_en.items():
        if pt_month in date_str:
            date_str = date_str.replace(pt_month, en_month)
            break
    return datetime.strptime(date_str, "%d de %B de %Y")
```

#### `extract_order_data(driver, target_order_id)`

Extracts data for a specific order and returns it as a DataFrame.

```python
def extract_order_data(driver, target_order_id):
    product_list = []
    wait = WebDriverWait(driver, 20)
    wait.until(EC.visibility_of_element_located((By.CLASS_NAME, 'table-wrapper.order-items')))
    products = driver.find_elements(By.CLASS_NAME, 'product')
    order_info_element = driver.find_element(By.CLASS_NAME, 'order-title__info')
    
    for product in products:
        order_id = target_order_id
        title = product.find_element(By.CLASS_NAME, 'name').text
        quantity = product.find_element(By.CLASS_NAME, 'product-buy__qty').text.split(' ')[1]
        price = product.find_element(By.CLASS_NAME, 'price').text
        volume = extract_volume(title)
        status_text = get_order_info_value(order_info_element, 'Status')
        order_date = get_order_info_value(order_info_element, 'Data do pedido')
        order_date_obj = convert_date_pt_to_en(order_date)
        order_value = get_order_info_value(order_info_element, 'Total')
        order_value_obj = re.search(r'R\$\d+,\d{2}', order_value).group(0)
        
        product_list.append({
            'Order_id': order_id, 
            'Title': title, 
            'Quantity': quantity, 
            'Price': price, 
            'Volume': volume, 
            'Status': status_text, 
            'Order Date': order_date_obj,
            'Order Value': order_value_obj
        })
    
    return pd.DataFrame(product_list)
```

### Main Function

The `main` function orchestrates the entire process.

```python
def main():
    driver = setup_driver()
    url = 'https://panini.com.br/sales/order/history/'
    driver.get(url)
    
    username = os.getenv('USERNAME')
    password = os.getenv('PASSWORD')
    
    accept_cookies(driver)
    login(driver, username, password)
    
    time.sleep(10)  # Wait for redirection after login
    
    order_ids = get_order_ids(driver)
    
    all_dataframes = []
    for order_id in order_ids:
        find_and_click_order(driver, order_id)
        df = extract_order_data(driver, order_id)
        all_dataframes.append(df)
        driver.back()  # Go back to the orders list page
    
    final_df = pd.concat(all_dataframes, ignore_index=True)
    
    print(final_df)
    final_df.to_csv('orders.csv', index=False)
    
    driver.quit()

if __name__ == "__main__":
    main()
```

## How to Run

1. Ensure all dependencies are installed.
2. Set your environment variables for `USERNAME` and `PASSWORD`.
3. Run the script:

   ```bash
   python script_name.py
   ```

This will log into the Panini website, navigate to the order history, extract all order data, and save it to `orders.csv`.