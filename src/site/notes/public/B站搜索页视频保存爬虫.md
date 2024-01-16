---
{"dg-publish":true,"permalink":"/public/b/","title":"B站搜索页视频保存爬虫"}
---






```
from selenium import webdriver

from selenium.webdriver.common.by import By

from bs4 import BeautifulSoup

from selenium.webdriver.support.ui import WebDriverWait

from selenium.webdriver.support import expected_conditions as EC

import xlwt

import time

import os

n = 1

word = input('请输入要搜索的关键词：')

driver = webdriver.Chrome()

wait = WebDriverWait(driver,20)

excl = xlwt.Workbook(encoding='utf-8', style_compression=0)

sheet = excl.add_sheet('b站视频：'+word, cell_overwrite_ok=True)

sheet.write(0, 0, '名称')

sheet.write(0, 1, 'up主')

sheet.write(0, 2, '播放量')

sheet.write(0, 3, '视频时长')

sheet.write(0, 4, '链接')

sheet.write(0, 5, '发布时间')

def search():

    driver.get("https://www.bilibili.com/")

    input = wait.until(EC.presence_of_element_located((By.CLASS_NAME, 'nav-search-input')))

    button = wait.until(EC.element_to_be_clickable((By.CLASS_NAME, 'nav-search-btn')))

    input.send_keys(word)

    button.click()

    print('开始搜索:'+word)

    windows = driver.window_handles

    driver.switch_to.window(windows[-1])

    wait.until(EC.presence_of_element_located((By.CSS_SELECTOR,

                                               '#i_cecream > div > div:nth-child(2) > div.search-content--gray.search-content > div > div > div > div.brand-ad-list.search-all-list')))

    get_source()

    print('开始下一页：')

    button_next = driver.find_element(By.CSS_SELECTOR,

                                      '#i_cecream > div > div:nth-child(2) > div.search-content--gray.search-content > div > div > div > div.flex_center.mt_x50.mb_x50 > div > div > button:nth-child(11)')

    button_next.click()

    #time.sleep(2)

    wait.until(EC.presence_of_element_located((By.CSS_SELECTOR, '#i_cecream > div > div:nth-child(2) > div.search-content > div > div > div.video-list.row > div:nth-child(1) > div > div.bili-video-card__wrap.__scale-wrap > div > div > a > h3')))

    get_source()

    print("完成")

def next_page():

    button_next = wait.until(EC.presence_of_element_located((By.CSS_SELECTOR,

                                      '#i_cecream > div > div:nth-child(2) > div.search-content > div > div > div.flex_center.mt_x50.mb_lg > div > div > button:nth-child(11)')))

    button_next.click()

    print("开始下一页")

    #time.sleep(5)

    wait.until(EC.presence_of_element_located((By.CSS_SELECTOR,

                                               '#i_cecream > div > div:nth-child(2) > div.search-content > div > div > div.video-list.row > div:nth-child(1) > div > div.bili-video-card__wrap.__scale-wrap > div > div > a > h3')))

    get_source()

    print("完成")

def save_excl(soup):

    list = soup.find(class_='video-list row').find_all(class_="bili-video-card")

    for item in list:

        # print(item)

        video_name = item.find(class_='bili-video-card__info--tit').text

        video_up = item.find(class_='bili-video-card__info--author').string

        video_date = item.find(class_='bili-video-card__info--date').string

        video_play = item.find(class_='bili-video-card__stats--item').text

        video_times = item.find(class_='bili-video-card__stats__duration').string

        video_link = item.find('a')['href'].replace('//', 'https://')

  

        cmdmingling="bbdown -tv "+video_link

  

        os.system(cmdmingling)

        time.sleep(30)

        print(video_name, video_up, video_play, video_times, video_link, video_date)

        global n

        sheet.write(n, 0, video_name)

        sheet.write(n, 1, video_up)

        sheet.write(n, 2, video_play)

        sheet.write(n, 3, video_times)

        sheet.write(n, 4, video_link)

        sheet.write(n, 5, video_date)

        n = n +1

def get_source():

    html = driver.page_source

    soup = BeautifulSoup(html, 'lxml')

    save_excl(soup)

def main():

    search()

    for i in range(1,1):

        next_page()

        i = i + 1

    driver.close()

if __name__ == '__main__':

    main()

    excl.save('b站'+word+'视频.xls')
    
    
    ```