
---
title: 基于Selenium模拟浏览器爬虫详解
tags:
  - python
  - selenium
  

date: 2019-5-2 13:14:00


categories:
 - python

---
Selenium 是一个用于web应用程序自动化测试的工具，直接运行在浏览器当中，支持chrome、firefox等主流浏览器。可以通过代码控制与页面上元素进行交互（点击、输入等），也可以获取指定元素的内容。
<!--more-->




一.背景

Selenium 是一个用于web应用程序自动化测试的工具，直接运行在浏览器当中，支持chrome、firefox等主流浏览器。可以通过代码控制与页面上元素进行交互（点击、输入等），也可以获取指定元素的内容。

劣势：

相比于抓包→构造请求→解析返回值的爬虫，由于Selenium需要生成一个浏览器环境，所有操作（与元素交互、获取元素内容等）均需要等待页面加载完毕后才可以继续进行，所以速度相比构造请求的慢很多。对于为了反爬做了特殊处理的展示内容，如字体加密（参考猫眼）、图片替换数字（参考自如）等，可能取不到想要的数据。

使用图片替换数字的自如：

image-20190107215702089

优势：

a. 不需要做复杂的抓包、构造请求、解析数据等，开发难度相对要低一些。

b. 其访问参数跟使用浏览器的正常用户一模一样，访问行为也相对更像正常用户，不容易被反爬虫策略命中。

c.生成的浏览器环境可以自动运行 JS 文件，所以不用担心如何逆向混淆过的JS文件生成用作人机校验的参数，如马蜂窝酒店评论的人机校验参数_sn，网易云音乐评论的人机校验参数params、encSecKey。可以自行抓包查看。

d. 如果需要抓取同一个前端页面上面来自不同后端接口的信息，如OTA酒店详情页的酒店基础信息、价格、评论等，使用Selenium可以在一次请求中同时完成对三个接口的调用，相对方便。

二、实现

1.环境
python3.6
Macos
Selenium
3.浏览器驱动（webdriver）

加载浏览器环境需要下载对应的浏览器驱动，此处选择 Chrome。

下载地址：http://npm.taobao.org/mirrors/chromedriver/ ，

选择合适的版本下载解压后放在随便一个位置即可。

4.hello world

from selenium import webdriver

 这里填刚刚下载的驱动的路径 

path =  /Applications/Google Chrome.app/Contents/chromedriver 

driver = webdriver.Chrome(executable_path=path)

url =  http://hotel.qunar.com/city/beijing_city/ 

driver.get(url)

 运行上述代码，会打开一个浏览器，并且加载去哪儿的酒店列表页 

这时候可以通过webdriver自带的一些的一些方法获取元素内容或者与元素进行交互。

image-20190108224445170

#返回ID = js_block_beijing_city_7810的元素信息

hotel_info = driver.find_element_by_id( js_block_beijing_city_7810 )

print(hotel_info.text)

#返回 展示在列表页的酒店信息

#同理，可以find_element_by_[class_name|name] 等，均可完成查询。

也可以通过方法 find_elements查找符合某条件的一组元素，以列表的形式返回。

image-20190108225039418

#当需要查询的唯一标识带有空格时，可以使用find_elements_by_css_selector，否则会报错。

hotel_list = driver.find_elements_by_css_selector("[class= b_result_box js_list_block  b_result_commentbox ]")

print(hotel_list)

#返回酒店列表的全部信息。

5.关闭图片加载

在不需要抓取图片的情况下，可以设置不加载图片，节约时间，这样属于调整本地设置，在传参上并不会有异常。

from selenium import webdriver

chrome_opt = webdriver.ChromeOptions()

prefs={"profile.managed_default_content_settings.images":2}

chrome_opt.add_experimental_option("prefs",prefs)

path =   #驱动路径

browser_noPic = webdriver.Chrome(executable_path=path,chrome_options=chrome_opt)

三、使用webdriver与元素进行交互

1.模拟鼠标点击

image-20190108230139238

hotel_info = driver.find_element_by_id("js_plugin_tag_beijing_city_7810")

hotel.info.click()

#进入酒店详情页

2.模拟键盘输入

hotel_search = driver.find_element_by_id("jxQ")

hotel_search.send_keys("如")

hotel_search.send_keys("如家")

#由于搜索框输入的第一个字会被选中，所以需要第二次才能完整输入，当然也可以模拟按键盘的 →(右键)取消选中后再次输入。

3.模拟下拉

webdriver中对鼠标的操作的方法封装在ActionChains类中 ，使用前要先导入ActionChains类：

from selenium.webdriver.common.action_chains import  ActionChains

"""在页面顶部、底部个找了一个元素，并模拟鼠标从顶到底的滑动"""

start = driver.find_element_by_class_name( e_above_header )

target = driver.find_element_by_class_name( qn_footer )

ActionChains(driver).drag_and_drop(start,target).perform()

此外，webdiver还提供丰富的交互功能，比如鼠标悬停、双击、按住左键等等，此处不展开介绍。

四、一个完整的模拟浏览器爬虫

from selenium import webdriver

from selenium.webdriver.common.action_chains import  ActionChains

import time

 这里填刚刚下载的驱动的路径 

path =  /Users/./Desktop/chromedriver 

driver = webdriver.Chrome(executable_path=path)

url =  http://hotel.qunar.com/city/beijing_city/ 

driver.get(url) 


time.sleep(6) #等待页面加载完再进行后续操作

"""在页面顶部、底部个找了一个元素，并模拟鼠标从顶到底的滑动"""

start = driver.find_element_by_class_name( e_above_header )

target = driver.find_element_by_class_name( qn_footer )

ActionChains(driver).drag_and_drop(start,target).perform()

time.sleep(5) #等待页面加载完再进行后续操作

hotel_link_list = driver.find_elements_by_css_selector("[class= item_price js_hasprice ]")

print("在此页面共有酒店",len(hotel_link_list),"家")

windows = driver.window_handles

#此处可以爬整个页面任何想要想要的元素

list_hotel_info=[]

def hotel_info_clawer():

    list_hotel_info.append([driver.find_element_by_class_name("info").text,

                            driver.find_element_by_class_name("js-room-table").text,

                            driver.find_element_by_class_name("dt-module").text])


for i in range(len(hotel_link_list)):

    hotel_link_list[i].click()

    driver.switch_to.window(windows[-1]) #切换到刚打开的酒店详情页

    hotel_info_clawer()

    driver.close() #关闭已经爬完的酒店详情页    

    print("已经抓取酒店",i,"家")


#后面可以补充翻页继续抓取的部分

五、使用截图+OCR抓取关键数据

对于做了特殊处理的信息，如上述的猫眼电影的票房信息、自如的价格等，不适用于直接获取制定元素的信息进行抓取，可以使用截图+OCR的方式抓取此类数据。

以自如的房租为例：

image-20190112201939908

from selenium import webdriver

 这里填刚刚下载的驱动的路径 

path =  /Applications/Google Chrome.app/Contents/chromedriver 

driver = webdriver.Chrome(executable_path=path)

url =  http://www.ziroom.com/z/vr/61715463.html 

driver.get(url)

price = diver.find_element_by_class_name( room_price )

print(price.text)#由于自如的价格用图片做了替换，这样并不能获取到实际价格，需要获取图片再做ocr处理

"对指定元素部分截图再保存"

price.screenshot( /Users/./Desktop/price.png )

安装ocr工具：

Tesseract是一个开源的OCR引擎，能识别100多种语言（中，英，韩，日，德，法…等等），但是Tesseract对手写的识别能力较差，仅适用于打印字体。

//仅安装tesseract，不安装训练工具和其他语音包，需要识别中文的话得额外下载

//下载地址：https://github.com/tesseract-ocr/tessdata

brew install tesseract

使用Tesseract：

tesseract ~/price.png result

//识别图片并将结果存在result里面

在python下使用Tesseract：

首先安装依赖包：pip install pytesseract

import pytesseract

from PIL import Image

# open image

image = Image.open( price.png )

code = pytesseract.image_to_string(image)

print(code)