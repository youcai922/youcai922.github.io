

```python
from selenium import webdriver
from time import sleep

bro = webdriver.Chrome(executable_path='./chromedriver')

#标签定位
search_input = bro.find_element_by_id('q')
#标签交互
search_input.send_key('Iphone')

#执行一组js程序
bro.execute_script('window.scrollTo(0,document.body.scrollHeight)')
sleep(2)
#点击搜索按钮
btn=bro.find_element_by_css_selector('.btn-search')
btn.click()

#回退
bro.back()
#前进
bro.forward()

bro.quit()
```



#### iframe操作

```python
from selenium import webdriver
from time import sleep
#导入动作链对应的类
from selenium.webdriver import ActionChains
bro = webdriver.Chrome(executable_path='./chromedriver')

bro.get('https://www.runoob.com/try/try.php?filename=jqueryui-api-droppable')

#如果定位的标签是存在于iframe标签之中的，则必须通过如下操作机型标签定位
#切换浏览器标签定位的作用域
bro.switch_to.frame('iframeResult')
div = bro.find_element_by_id('draggable')

#动作链
action = ActionChains(bro)
action.click_and_hold(div)

from i in range(5):
    #perform()立即执行动作链操作
    #move_by_offset(x,y):x水平防线，y竖直方向
    action.move_by_offset(17,0).proform()
    sleep(0.5)
   
action.release()

bro.quit();
```



### 无可视化界面

```python
from selenium import webdriver
from time import sleep
from selenium.webdriver.chrome.options import Options

chrome_options = Options()
chrome_options.add_argument('--headless')
chrome_options.add_argument('--disable-gpu')

bro = webdriver.Chrome(executable_path='./chromedriver', chrome_options=chrome_options)

bro.get('https://www.baidu.com')

print(bro.page_source)
bro.quit()
```

### 规避检测

```python
from selenium.webdriver import ChromeOptions

option = ChromeOptions()
option.add_experimental_option('excludeSwitches',['enable-automation'])

bro = webdriver.Chrome(executable_path='./chromedriver', chrome_options=chrome_options,options=option) 
```



```python
from selenium import webdriver
from time import sleep
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.by import By

if __name__ == "__main__":
    username = 13026373922
    password = "2287360627yucan"

    headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36'
    }
    url = 'https://detail.damai.cn/item.htm?id=763755105403'

    chrome_driver_path = 'F:\software\chromedriver_win32\chromedriver.exe'
    s = Service(chrome_driver_path)

    # 创建ChromeOptions对象来设置浏览器选项
    options = webdriver.ChromeOptions()
    # 例如，设置无头模式（不显示浏览器界面）
    # options.add_argument('--headless')
    options.add_argument("--disable-gpu")  # 禁用GPU加速
    options.add_argument("--no-sandbox")  # 禁用沙箱模式（在Linux上需要）
    options.add_argument("--disable-dev-shm-usage")  # 禁用/dev/shm内存使用（在Linux上需要）
    options.add_argument("--disable-extensions")  # 禁用扩展
    options.add_argument("--disable-blink-features=AutomationControlled")  # 隐藏自动化控制标志
    options.add_argument(
        "user-agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36")  # 设置自定义User-Agent

    options.add_experimental_option('excludeSwitches', ['enable-automation'])
    # 禁用图片加载
    No_Image_loading = {"profile.managed_default_content_settings.images": 2}
    options.add_experimental_option("prefs", No_Image_loading)
    # 创建Chrome浏览器实例，传入Service和ChromeOptions
    driver = webdriver.Chrome(service=s, options=options)

    # 演出详情页
    driver.get(url)
    login_btn = driver.find_element(By.CLASS_NAME, "login-user")
    login_btn.click()
    # 登陆页面
    driver.switch_to.frame('alibaba-login-box')
    username_input = driver.find_element(By.ID, "fm-login-id")
    username_input.send_keys(username)
    password_input = driver.find_element(By.ID, "fm-login-password")
    password_input.send_keys(password)
    driver.find_element(By.CLASS_NAME, "fm-btn").click()
    sleep(4)
    # 演出详情页，驱动切换作用域
    driver.switch_to.default_content()
    driver.find_element(By.CLASS_NAME, 'cafe-c-input-number-handler-up').click()
    driver.find_element(By.CLASS_NAME, "buy-link").click()
    sleep(1)
    driver.execute_script("window.scrollTo(0, 70);")
    driver.find_element(By.XPATH, '//*[@id="dmViewerBlock_DmViewerBlock"]/div[2]/div/div[1]/div[2]/i').click()
    driver.find_element(By.XPATH, '//*[@id="dmViewerBlock_DmViewerBlock"]/div[2]/div/div[2]/div[2]/i').click()
    driver.find_element(By.XPATH, '//*[@id="dmOrderSubmitBlock_DmOrderSubmitBlock"]/div[2]/div/div[2]/div[2]/div[2]') \
        # .click()
    sleep(3)
    driver.quit()
```

