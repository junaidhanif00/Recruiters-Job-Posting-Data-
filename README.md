# Recruiters-Job-Posting-Data-

from selenium import webdriver
from selenium.webdriver.common.by import By
from bs4 import BeautifulSoup
import time
import pandas as pd

# Creating a webdriver instance
driver = webdriver.Chrome("chromedriver.exe")

# Opening LinkedIn's login page
driver.get("https://linkedin.com/uas/login")

# waiting for the page to load
time.sleep(5)

# entering username
username = driver.find_element("id", "username")
username.send_keys("")

# entering password
pword = driver.find_element("id", "password")
pword.send_keys("")

# Clicking on the login button
login_btn = driver.find_element(By.XPATH, '/html/body/div/main/div[2]/div[1]/form/div[3]/button')
time.sleep(10)
login_btn.click()
time.sleep(60)

# Prompting the user to enter keywords for job search
keywords = input("Enter keywords for job search:")

# updating the LinkedIn job search URL with the entered keywords
main_url = f"https://www.linkedin.com/jobs/search/?keywords={keywords}&currentJobId=3354274229&f_JT=F&f_TPR=r86400&f_WT=2"

# Scrape all pages of job search

for page_num in range(1, 5):
    # Updating the LinkedIn job search URL with the page number
    page_url = f"{main_url}&start={25 * (page_num - 1)}"
    # Opening the job search page
    driver.get(page_url)
    time.sleep(30)
    while True:
        last_height = driver.execute_script('return document.body.scrollHeight')
        driver.execute_script('window.scrollTo(0, document.body.scrollHeight);')
        time.sleep(30)  # Wait for 5 seconds for the page to load more job postings
        new_height = driver.execute_script('return document.body.scrollHeight')
        if new_height == last_height:
            break

    # Getting the page source
    page_source = driver.page_source

    # Parsing the page source using Beautiful Soup
    soup = BeautifulSoup(page_source, 'html.parser')

    data = []

    job_postings = soup.find_all('li', {'class': 'jobs-search-results__list-item'})

    for job_posting in job_postings:
        try:
            job_title = job_posting.find('a', class_='job-card-list__title').get_text().strip()
        except AttributeError:
            job_title = None

        try:
            job_title_url = 'https://www.linkedin.com' + job_posting.find('a', class_='job-card-list__title').get('href').strip()
        except AttributeError:
            job_title_url = None

        try:
            company_name = job_posting.find('a', class_='job-card-container__company-name').get_text().strip()
        except AttributeError:
            company_name = None

        try:
            company_name_url = 'https://www.linkedin.com' + job_posting.find('a', class_='job-card-container__company-name').get('href').strip()
        except AttributeError:
            company_name_url = None

        try:
            location = job_posting.find('li', class_='job-card-container__metadata-item').get_text().strip()
        except AttributeError:
            location = None

        try:
            post_hours = job_posting.find('li', class_='job-card-container__listed-time').get_text().strip()
        except AttributeError:
            post_hours = None

        try:
            remote = job_posting.find('li', class_='job-card-container__metadata-item--workplace-type').get_text().strip()
        except AttributeError:
            remote = None

        #print(remote)
       # print(job_title)
       # print(job_title_url)
       # print(company_name)
       # print(company_name_url)
       # print(location)
       # print(post_hours)

        data.append({'job_title':job_title, 'title_url':job_title_url, 'company_name':company_name, 'company_url':company_name_url,
                    'location':location,'post_hours':post_hours, 'remote':remote})
      #Save the data to a CSV file
        df = pd.DataFrame(data)
        df.to_csv('dataanalyst.csv', mode='a', header=False)
