from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
import time
import os
import re


options = webdriver.ChromeOptions()
options.add_argument('--ignore-certificate-errors')
options.add_experimental_option('prefs', {'download.default_directory': r'D:\downloads\知网文献'})

current_page_xpath = '//*[contains(@id,"gridTable")]/div[2]/span'
next_page_xpath = '//*[contains(@id,"PageNext")]'

# 初始化浏览器
driver = webdriver.Chrome(chrome_options=options)
download_path = r'D:\downloads\知网文献'

# 如果路径不存在,创建目录
if not os.path.exists(download_path):
    os.makedirs(download_path)
# 打开首页
driver.get('https://fsso.cnki.net/')
print('打开首页')
time.sleep(2)
# 输入框可见
WebDriverWait(driver, 10).until(EC.visibility_of_element_located((By.XPATH, '//*[@id="o"]')))
print('输入框已出现')
time.sleep(1)
# 点击输入框
input_box = driver.find_element(By.XPATH, '//*[@id="o"]')
input_box.click()
print('已点击输入框')
time.sleep(1)
# 选择学校
school_btn = driver.find_element(By.XPATH, '//*[@id="auto"]/div[102]')
school_btn.click()
print('已选择学校')
time.sleep(2)
# 点击前往按钮
transfer_btn = driver.find_element(By.XPATH, '/html/body/div[2]/div[1]/div[2]/div[2]')
transfer_btn.click()
time.sleep(2)
print(driver.current_url)
# 跳转登录页
login_url = 'https://idp.fjtcm.edu.cn/idp/profile/SAML2/Redirect/SSO?execution=e1s1'
driver.get(login_url)
# 账号输入框
username_input = driver.find_element(By.XPATH, '//*[@id="username"]')
print('输入账号:')
username = input()
# 密码输入框
password_input = driver.find_element(By.XPATH, '//*[@id="password"]')
print('输入密码:')
password = input()
# 输入账号密码
username_input.send_keys(username)
password_input.send_keys(password)
time.sleep(5)
# 点击登录按钮
login_btn = driver.find_element(By.XPATH, '/html/body/div/div/div/div[1]/form/div[5]/button')
login_btn.click()
print('点击登录')
time.sleep(5)
# 不刷新,在当前页面上操作
# 点击按钮
btn = driver.find_element(By.XPATH, '//*[@id="DBFieldBox"]/div[1]/i')
btn.click()
# 点击篇关摘

paguangzhai = WebDriverWait(driver, 10).until(EC.element_to_be_clickable((By.XPATH, '//*[@id="DBFieldList"]/ul/li[2]')))
paguangzhai.click()
# 定位输入框
search_input = driver.find_element(By.XPATH, '//*[@id="txt_SearchText"]')
# 打印提示语
print('请输入篇关摘:')
# 获取用户输入
kw = input()
# 输入篇关摘
search_input.send_keys(kw)
# 单击搜索按钮
search_btn = driver.find_element(By.XPATH, '/html/body/div[2]/div[2]/div/div[1]/input[2]')
search_btn.click()
time.sleep(3)
count = driver.find_element(By.XPATH, '//*[@id="countPageDiv"]/span[1]/em').text
print(f'找到结果条数:{count}')
time.sleep(1)
start_page = int(input('请输入起始页(>=1):'))
end_page = int(input('请输入结束页:'))

time.sleep(2)

urls = []
for page in range(start_page, end_page + 1):
    if page > 1:
        driver.find_element(By.XPATH, f'//*[contains(@id, "page{page}")]').click()
        time.sleep(2)

    for i in range(1, 21):
        try:
            xpath = f'//*[@id="gridTable"]/table/tbody/tr[{i}]/td[2]/a'
            url = driver.find_element(By.XPATH, xpath).get_attribute('href')
            urls.append(url)
        except:
            print(f'获取第{i}条url失败,跳过')
            continue

    # 打印url总数
    print(f'获取url条数:{len(urls)}')

print('获取urls成功')

# 创建文件夹
folder_name = kw
download_dir = f'D:\\downloads\\知网文献\\{folder_name}'
options.add_experimental_option('prefs', {'download.default_directory': download_dir})
if not os.path.exists(folder_name):
  os.mkdir(folder_name)

retry_times = 3

for url in urls:

    while retry_times > 0:
        try:
            driver.get(url)
            driver.find_element(By.XPATH, '//*[@id="pdfDown"]').click()
            print(f'{url}下载成功')
            time.sleep(1)  # 暂停1秒
            break
        except:
            retry_times -= 1

    if retry_times == 0:
        print(f'{url}下载失败')

    retry_times = 3  # 重置重试次数

