# Automation-Testing
## Flipkart
### Code
```
import re
import subprocess
import time
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.common.exceptions import TimeoutException, StaleElementReferenceException

URL = "https://www.flipkart.com/login"
MOBILE = "7845346177"

NON_SEARCH_INPUTS = "//input[not(@name='q') and not(@type='hidden') and not(@disabled)]"
CONTINUE_BTN = (By.XPATH, "//button[contains(.,'Continue') or contains(.,'Request OTP')]")
VERIFY_BTN = (By.XPATH, "//button[contains(.,'Verify') or contains(.,'Login') or contains(.,'Continue')]")


def adb_ready():
    try:
        out = subprocess.run(["adb", "devices"], capture_output=True, text=True).stdout
    except FileNotFoundError:
        return False
    return any(line.endswith("device") for line in out.splitlines()[1:])


def otp_from_phone(after_ts, timeout=60):
    end = time.time() + timeout
    while time.time() < end:
        out = subprocess.run(
            ["adb", "shell", "content", "query", "--uri", "content://sms/inbox",
             "--projection", "body,date", "--sort", "date DESC"],
            capture_output=True, text=True
        ).stdout
        for line in out.splitlines():
            if "flipkart" in line.lower():
                d = re.search(r"date=(\d+)", line)
                m = re.search(r"\b(\d{6})\b", line)
                if d and m and int(d.group(1)) / 1000 >= after_ts:
                    return m.group(1)
        time.sleep(2)
    return None


def get_otp(after_ts):
    if adb_ready():
        return otp_from_phone(after_ts)
    value = input("Type the 6-digit OTP from your phone: ").strip()
    return value if re.fullmatch(r"\d{6}", value) else None


def visible_inputs(driver):
    return [e for e in driver.find_elements(By.XPATH, NON_SEARCH_INPUTS) if e.is_displayed()]


def dump_inputs(driver):
    for e in driver.find_elements(By.TAG_NAME, "input"):
        print(e.get_attribute("outerHTML"))


options = webdriver.ChromeOptions()
options.add_argument("--disable-blink-features=AutomationControlled")

driver = webdriver.Chrome(options=options)
driver.maximize_window()
wait = WebDriverWait(driver, 20, ignored_exceptions=(StaleElementReferenceException,))
results = {}

try:
    driver.get(URL)

    wait.until(lambda d: len(visible_inputs(d)) > 0)
    phone = visible_inputs(driver)[0]
    driver.execute_script("arguments[0].scrollIntoView({block:'center'});", phone)
    phone.click()
    phone.send_keys(MOBILE)

    search_value = driver.find_element(By.NAME, "q").get_attribute("value")
    results["mobile entered"] = phone.get_attribute("value") == MOBILE and search_value == ""

    if not results["mobile entered"]:
        print("Inputs found on page:")
        dump_inputs(driver)
    else:
        requested_at = time.time()
        wait.until(EC.element_to_be_clickable(CONTINUE_BTN)).click()

        try:
            wait.until(lambda d: [e for e in visible_inputs(d) if e.get_attribute("value") == ""])
            otp_field = [e for e in visible_inputs(driver) if e.get_attribute("value") == ""][0]
            results["otp requested"] = True
        except TimeoutException:
            results["otp requested"] = False

        if results["otp requested"]:
            otp = get_otp(requested_at)
            results["otp received"] = otp is not None

            if otp:
                otp_field.click()
                otp_field.send_keys(otp)
                try:
                    wait.until(EC.element_to_be_clickable(VERIFY_BTN)).click()
                except TimeoutException:
                    pass
                try:
                    wait.until(lambda d: "/login" not in d.current_url)
                    results["login verified"] = True
                except TimeoutException:
                    results["login verified"] = False

except Exception as e:
    results["error"] = str(e).splitlines()[0]
    try:
        dump_inputs(driver)
    except Exception:
        pass

finally:
    driver.quit()

for step, status in results.items():
    print(f"{step}: {status}")

print("TEST PASSED" if results.get("login verified") is True else "TEST FAILED")
```
### Output
<img width="652" height="544" alt="image" src="https://github.com/user-attachments/assets/c4327069-02ca-47ec-8304-fbc6d4c0c946" />

<img width="619" height="482" alt="image" src="https://github.com/user-attachments/assets/8205efb9-4d2d-4866-b4e1-7ff579a4171e" />

## Amazon
### Code
```
from selenium import webdriver
from selenium.webdriver.common.by import By
import time

driver = webdriver.Chrome()

driver.get("https://www.amazon.in/")

time.sleep(3)

# Click Account & Lists
login = driver.find_element(By.ID, "nav-link-accountList")
login.click()

time.sleep(3)

# Enter email or mobile
email = driver.find_element(By.ID, "ap_email_login")
email.send_keys("your_email_id")

# Click Continue
continue_button = driver.find_element(
    By.XPATH, "//input[@type='submit']"
)
continue_button.click()

time.sleep(3)

# Enter password
password = driver.find_element(By.ID, "ap_password")
password.send_keys("Your Password")

print("Password entered")

# Click Sign-In
sign_in = driver.find_element(By.ID, "signInSubmit")
sign_in.click()

print("Sign-in clicked")

time.sleep(5)

print("Page title:", driver.title)

driver.quit()
```

### Output

<img width="1282" height="971" alt="image" src="https://github.com/user-attachments/assets/c4a811bc-2f1e-483f-b0df-362a6e0ddeac" />

<img width="1878" height="733" alt="image" src="https://github.com/user-attachments/assets/4e62ce57-5f48-4e98-bcda-e7038663f214" />
<img width="1333" height="990" alt="image" src="https://github.com/user-attachments/assets/49e7b102-27ba-43e4-9db0-0135b2829082" />
<img width="940" height="303" alt="image" src="https://github.com/user-attachments/assets/b16e67e9-bd05-4797-8f40-aeff5e013ac7" />
<img width="1677" height="962" alt="image" src="https://github.com/user-attachments/assets/9ae4b4fa-8c64-4a85-91bf-4d3ad4d7a05a" />

