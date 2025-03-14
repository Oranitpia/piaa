from selenium import webdriver
from selenium.webdriver.common.keys import Keys
import time
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.by import By
import os
import requests as r
import gspread
import pandas as pd
from oauth2client.service_account import ServiceAccountCredentials
import subprocess
import logging
import sys

# Set up logging
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

######################################### Input zone ###################################################
json = "API-key.json"
main_sheet = "https://docs.google.com/spreadsheets/d/1sb0KdlViCAnusxoO21LNiI7N36Qm-1AZqCrcE9bBAV0/edit#gid=0"


#######################################################################################################

def read_googlesheet(json, main_sheet):
    logging.info("Reading Google Sheets data")
    scope = ["https://www.googleapis.com/auth/spreadsheets"]
    credentials = ServiceAccountCredentials.from_json_keyfile_name(json, scope)
    gc = gspread.authorize(credentials)

    sheet = gc.open_by_url(main_sheet)
    worksheet = sheet.get_worksheet(0)
    sheet = pd.DataFrame(worksheet.get_all_values())

    new_header = sheet.iloc[0]
    sheet.columns = new_header
    sheet = sheet.iloc[1:]
    sheet = sheet.reset_index().drop(columns=["index"])

    return sheet


logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

def send_script(taskname, new_data_url):
    logging.info(f"Sending task: {taskname} with data URL: {new_data_url}")
    print("Task URL:", taskname)  # แสดง URL ของ task
    print("New Data URL:", new_data_url)  # แสดง URL ของข้อมูลใหม่

    if "prinventures" in taskname:
        url = "https://script.google.com/macros/s/AKfycbxfrp5n3jhIRFoZ8XUv7U35ARD_acXkDML_o6w5Y34qG_0VerP_6a2WKquOORrNrf9v/exec"
        # ถ้าต้องการใช้บัญชีอื่นสเครปให้เปลี่ยนเจ้าของในสคริปต์นี้ด้วย
        # https://script.google.com/u/0/home/projects/1aB41V2n0qUa2edJ0YHW6aXRvqN5iuwve0LHNmUag5-iYDPvW4TzMiLp6/edit
    else:
        url = "https://script.google.com/macros/s/AKfycbzO2wMBZ00wkik5AvujmDZ4Z0qpZSNlkN63JTvfD-WPt1bnwAZjtUMxiOXU1lWEWqmx4g/exec"
        # ถ้าต้องการใช้บัญชีอื่นสเครปให้เปลี่ยนเจ้าของในสคริปต์นี้ด้วย
        # https://script.google.com/home/projects/1T3DJixivnGnaKF3BZR27N4FZnZkaFFvuHsgiA305mr0v6l8ZUJUPJs3a/projecthistory

    myobj = {
        "Task_name": taskname,
        "NewData_sheeturl": new_data_url
    }

    try:
        r.post(url, data=myobj, timeout=50)  # ใช้ timeout สั้น ๆ
        logging.info(f"Request sent for task: {taskname}")
    except r.exceptions.Timeout:
        logging.warning(f"Timeout while sending task: {taskname}, but continuing...")
    except Exception as e:
        logging.error(f"Error sending task: {taskname}, error: {e}")



def kill_chrome_and_driver():
    logging.info("Killing existing Chrome and ChromeDriver processes")
    subprocess.call("taskkill /f /im chromedriver.exe", shell=True)
    subprocess.call("taskkill /f /im chrome.exe", shell=True)


def is_password_required(driver):
    """Check if password input is required."""
    logging.info("Checking if password is required")
    try:
        WebDriverWait(driver, 5).until(
            EC.presence_of_element_located((By.XPATH, "//*[@id='password']/div[1]/div/div[1]/input")))
        return True
    except Exception:
        return False


def enter_password(driver, company):
    """Enter password based on company credentials."""
    password = "V<bjE6yh&2R=8aW%V<bjE6yh&2R=8aW%" if company == "prinsiri" else "#asdfkl123pia8935"
    #rePirinasdfk@
    logging.info(f"Enter128ing password for {company}")
    try:
        box2 = WebDriverWait(driver, 10).until(
            EC.presence_of_element_located((By.XPATH, "//*[@id='password']/div[1]/div/div[1]/input")))
        box2.send_keys(password)
        time.sleep(2)
        button1 = driver.find_element_by_xpath("//*[@id='passwordNext']/div/button")
        button1.click()
        time.sleep(60)
    except Exception as e:
        logging.error(f"Error entering password for {company}: {e}")
        sys.exit(f"Exiting script due to password entry error: {e}")

def scrape_start(username, password, adminurl, task, company):
    logging.info(f"Starting scrape for {task} with URL: {adminurl} and company: {company}")
    driver = None
    kill_chrome_and_driver()

    chrome_profile_path = r'C:/Users/admin/AppData/Local/Google/Chrome/User Data/'
    profile_name = "Profile 4" if company == "prinsiri" else "Profile 2"

    try:
        options = webdriver.ChromeOptions()
        options.add_argument(f"user-data-dir={chrome_profile_path}")
        options.add_argument(f'--profile-directory={profile_name}')
        options.add_argument("--disable-dev-shm-usage")
        options.add_argument("--no-sandbox")

        driver = webdriver.Chrome(executable_path=r'C:\Users\admin\Desktop\Line\chromedriver.exe', options=options)
        driver.get(adminurl)

        # ตรวจสอบว่าต้องใส่รหัสผ่านหรือไม่
        if is_password_required(driver):
            enter_password(driver, company)
            logging.info("Password entered, proceeding with download.")
        else:
            logging.info("No password required. Proceeding to task actions and download.")

        logging.info(f"Logged in automatically for {company}")
        time.sleep(10)

        # ดำเนินการ task-specific logic
        if task == "Approvals":
            logging.info("Performing 'Approvals' task...")
            try:
                driver.find_element_by_xpath(
                    "//*[@id='yDmH0d']/c-wiz/div/div[1]/div/div/div[2]/div/c-wiz/div/div/div/div[3]/div/div/div[2]/div/div/div/div/div/div/div/div[1]/div[2]/span/div/div[1]/div/div[2]/div[1]"
                ).click()
                time.sleep(10)
                logging.info("Approval task - Step 1 complete")

                driver.find_element_by_xpath(
                    "/html/body/div[7]/c-wiz/div/div[1]/div/div/div[2]/div/c-wiz/div/div/div/div[3]/div/div/div[2]/div/div/div/div/div/div/div/div[1]/div[2]/span/div/div[1]/div/div[2]/div[1]/div/div/div[2]/div/span/div[1]/div/label/input"
                ).send_keys("drive log event")
                time.sleep(2)
                logging.info("Approval task - Entered search term")

                driver.find_element_by_xpath(
                    "/html/body/div[7]/c-wiz/div/div[1]/div/div/div[2]/div/c-wiz/div/div/div/div[3]/div/div/div[2]/div/div/div/div/div/div/div/div[1]/div[2]/span/div/div[1]/div/div[2]/div[1]/div/div/div[2]/div/span/div[2]/div/div/span/div/span/div[2]/div"
                ).click()
                time.sleep(2)
                logging.info("Approval task - Search submitted")
            except Exception as e:
                logging.error(f"Error during 'Approvals' task: {e}")
                return  # ข้าม Task นี้ แต่ไม่หยุดโปรแกรม

        elif task == "datastudio_usage":
            logging.info("Performing 'datastudio_usage' task...")
            try:
                time.sleep(20)
                driver.find_element_by_xpath(
                    "/html/body/div[8]/c-wiz/div/div[1]/div/div/div[1]/div/div[2]/div/c-wiz/div/div/div/div[3]/div/div/div[2]/div[2]/div/div/div/div/div/div/div[1]/div[2]/span/div/div[3]/div/div/span"
                ).click()
                logging.info("Clicked on search button")


                time.sleep(18)
                driver.find_element_by_xpath(
                    "/html/body/div[8]/c-wiz/div/div[1]/div/div/div[1]/div/div[2]/div/c-wiz/div/div/div/div[3]/div/div/div[2]/div[2]/div/div/div/div/div/div/div[1]/div[2]/span/div/div[5]/div/div/div[1]/div[2]/div/div[1]/div[1]/div/span"
                ).click()
                logging.info("Clicked on export button")

                time.sleep(20)
                driver.find_element_by_xpath(
                    "/html/body/div[8]/div[5]/div/div[2]/span/div/span/div/div[1]/div/div[1]/div/div[1]/input"
                ).send_keys("datastudio sheet")
                logging.info("Entered 'datastudio sheet' for export")

                time.sleep(2)
                driver.find_element_by_xpath(
                    "/html/body/div[8]/div[5]/div/div[2]/span/div/div[2]/div[2]/span/span"
                ).click()
                logging.info("Clicked on confirm export")

                driver.execute_script('window.scrollBy(0, 1000);')
                logging.info("Scroll down for further actions")
                time.sleep(500)  # Wait for download to complete

                driver.find_element_by_xpath(
                    "/html/body/div[8]/c-wiz/div/div[1]/div/div/div[1]/div/div[2]/div/c-wiz/div/div/div/div[3]/div/div/div[2]/div[2]/div/div/div[2]/div/div/div/div/div/div[2]/span/div/div/div[3]/div[1]/table/tbody/tr/td[2]/div/div/div"
                ).click()
                time.sleep(60)
                logging.info("Final action complete in 'datastudio_usage' task")

            except Exception as e:
                logging.error(f"Error during 'datastudio_usage' task: {e}")
                return  # ข้าม Task นี้ แต่ไม่หยุดโปรแกรม

        else:
            logging.info(f"Performing general task: {task}")
            try:
                # General case: assume that you need to download some file or perform other actions
                for download_button in [
                    "/html/body/div[8]/c-wiz/div/div[1]/div/div/div[1]/div/div[2]/div/div[2]/div/div/div/div/div[1]/div[2]/div[4]/div/span/span",
                    "/html/body/div[8]/div[5]/div/div[2]/span/div/span/c-wiz/div[1]/div/div[2]/div[1]/label",
                    "/html/body/div[8]/div[5]/div/div[2]/span/div/div[2]/div[2]"
                ]:
                    try:
                        button1 = driver.find_element_by_xpath(download_button)
                        button1.click()
                        time.sleep(3)
                        logging.info(f"Clicked on download button: {download_button}")

                        # Select "All Columns"
                        for choice in [
                            "/html/body/div[8]/div[5]/div/div[2]/span/div/span/c-wiz/div[1]/div/div[2]/div[1]/label",
                            "/html/body/div[8]/div[5]/div/div[2]/span/div/div[2]/div[2]"
                        ]:
                            try:
                                driver.find_element_by_xpath(choice).click()
                                logging.info(f"Selected all columns for export: {choice}")
                                time.sleep(10)
                            except Exception as e:
                                logging.error(f"Error selecting column: {e}")
                                return

                        logging.info("File download initiated")
                        time.sleep(50)
                        download1 = driver.find_element_by_xpath("//*[@id='tabUser']/div/div[1]/div/div[2]/div[1]/a")
                        download1.click()
                        time.sleep(12)
                        break
                    except Exception as e:
                        logging.error(f"Error clicking download button: {e}")
                        return

            except Exception as e:
                logging.error(f"Error during general task: {e}")
                return  # ข้าม Task นี้ แต่ไม่หยุดโปรแกรม
            # Handle success and data capture
            driver.switch_to.window(driver.window_handles[1])
            time.sleep(2)
            sheet_url = driver.current_url
            logging.info(f"Task {task} completed, sheet URL: {sheet_url}")

            driver.close()
            driver.switch_to.window(driver.window_handles[0])
            driver.close()

            send_script(task, sheet_url)
        driver.switch_to.window(driver.window_handles[1])
        time.sleep(10)

        # Handle success and data capture
        sheet_url = driver.current_url
        logging.info(f"Task {task} completed, sheet URL: {sheet_url}")
        send_script(task, sheet_url)

    except Exception as e:
        logging.error(f"Error during scrape for {task}: {e}")
    finally:
        # ตรวจสอบและปิด driver เพื่อป้องกัน invalid session id
        if driver:
            try:
                driver.quit()
            except Exception as quit_error:
                logging.warning(f"Error quitting WebDriver: {quit_error}")


data = read_googlesheet(json, main_sheet)

for i in range(len(data)):
    try:
        # อ่านข้อมูลแต่ละแถว
        input_task = data["Task_name"].iloc[i]
        input_adminUrl = data["AdminUrl"].iloc[i]
        input_company = data["Access"].iloc[i]

        username = ""
        password = ""

        # ลองรัน scrape_start
        scrape_start(username, password, input_adminUrl, input_task, input_company)

    except Exception as row_error:
        logging.error(f"Error processing row {i + 1}: {row_error}")
        continue  # ข้ามไปยังแถวถัดไป

logging.info("Script execution completed.")

