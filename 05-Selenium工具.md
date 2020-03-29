# Selenium工具

### Selenium

##### 认识selenium

**Selenium是一个用于Web应用程序测试的工具**

通常各大网站的后台都会有一定的反爬机制，既为了数据安全，也为了减小服务器压力。反爬的手段的方向，就是识别非浏览器客户端，**而selenium恰恰是驱动真正的浏览器去执行请求和操作**，对于服务端来说，是没有任何差别的。

**用户能做的一切，selenium几乎都驱动浏览器取做**，无论是否有界面，包括输入、点击、滑动，等等

##### 网页数据来源

有时候用 requests 抓取页面的时候，得到的结果可能和在浏览器中看到的不一样：在浏览器中能看到的页面数据，但是在 requests 得到的结果中并没有。**这是因为 requests 获取的都是原始的 HTML 文档，而浏览器中的页面则是经过 JavaScript 处理数据后生成的结果**。

网页数据的来源大体有三种：

1. 有的页面的数据是放在HTML文档中
2. 有的页面的数据是通过Ajax请求加载的（即向服务器请求某个接口获取数据）
3. 有的页面的数据是通过JavaScript和特定算法计算后生成的

对于数据加密或放在不同地方的网页，**使用selenium驱动浏览器获取数据加载完成后的页面的源代码，做到可见即可爬**。选择selenium来做爬虫，也说明网站的反爬能力比较高（要不然直接上scrapy了），对网页之间的连贯性，cookies，用户状态等有较高的监测。

### 浏览器

Chrome（谷歌）、Firefox（火狐）、safari（苹果浏览器），流行组合基本是Selenium+Chrome或者Selenium+Firefox。

##### 有头模式

Head Browser(有头的浏览器)，**有图形用户界面(GUI)的web浏览器**。

##### 无头模式

Headless Browser(无头的浏览器)，**没有图形用户界面(GUI)的web浏览器，通常是通过编程或命令行界面来控制的**。

**注意：有头和无头只是一种运行模式，浏览器可以任选一种模式来运行。**

##### 模式区别

相同点：

1. **有头模式和无头模式都是由selenium来驱动浏览器**。
2. **浏览器在有头模式或无头模式下，都能完成所有操作**。

不同点：

1. **无头模式驱动浏览器，不会显示浏览器界面；有头模式驱动浏览器，会显示浏览器界面**。
2. **selenium驱动浏览器，默认有头模式，设置无头模式，需要设置options参数**。
3. 无头模式比有头模式在内存消耗，运行时间，CPU占用上面都有一定的优势

### Chromedriver

##### 简介

Chromedriver是驱动Chrome浏览器的驱动程序，主要作用启动Chrome浏览器，而Selenium就是操作浏览器的执行点击、输入等各种事件的库。打个比方，Chromedriver就是车钥匙，来启动汽车，Selenium就是车的主人，来驾驶汽车。

##### 安装Chromedriver

打开链接：http://npm.taobao.org/mirrors/chromedriver/2.41/ ，选择符合符合自己电脑操作系统的版本的Chromedriver，进行下载，完成后解压，**将解压文件拖入安装Python程序的Scripts文件夹中（相当于变相的加入了环境变量中，前提是Scripts文件夹路径已在环境变量中），或者在程序webdriver.Chrome()中添加executable_path参数来指定Chromedriver文件位置（这样稍显麻烦。）**

##### Chrome坑

驱动Chrome浏览器，去爬取网页会有许多坑，有时相同的代码驱动不同版本的Chrome浏览器都可能会产生错误，有时候访问页面需要delete_all_cookies()才能刷新出数据，有时候定位到元素进行点击出现资源加载错误，更别说还需要各种chrome_options配置，建议使用Firefox浏览器。

### geckodriver

geckodriver是驱动Firefox浏览器的驱动程序，主要作用启动Firefox浏览器。

##### 安装geckodriver

打开链接：https://github.com/mozilla/geckodriver/releases ，选择符合符合自己电脑操作系统的版本的geckodriver，进行下载，完成后解压，后面步骤和安装Chromedriver一样。

### Selenium基本操作

##### 声明浏览器对象

```
# 导入selenium的浏览器驱动接口webdriver
from selenium import webdriver

# selenium支持非常多的浏览器，如Chrome、Firefox、Safari等（运行前提是环境变量中存在该浏览器的驱动）
browser = webdriver.Chrome()
browser = webdriver.Firfox()
browser = webdriver.Safari()
注意：浏览器首字母大写
```

##### 获取网页代码

```
# 注释：因为默认为有头模式，运行代码后，会有谷歌浏览器弹出，访问页面

# 导入selenium库
from selenium import webdriver

# 声明初始化的浏览器为Chrome（谷歌浏览器）
browser = webdriver.Chrome()

# get方法访问传入url
browser.get('https://www.taobao.com')

# page_source方法输出网页的源代码
print(browser.page_source)

# 关闭浏览器
browser.close()

输出：
<title>淘宝网 - 淘！我喜欢</title>
<meta name="spm-id" content="a21bo" />
<meta name="description" content="淘宝网 - 亚洲较大的网上交易平台，提供各类服饰、美容、家居、数码、话费/点卡充值… 数亿优质商品，同时提供担保交易(先收货后付款)等安全交易保障服务，并由商家提供退货承诺、破损补寄等消费者保障服务，让你安心享受网上购物乐趣！" />...
```

##### Chrome配置参数

在使用selenium + Chromedriver驱动chrome浏览器爬取网站信息时，默认情况下就是一个普通的纯净的chrome浏览器，还需要对这个chrome做一些特殊的配置，以满足爬虫的行为。

常用的行为有：

- **禁止图片和视频的加载：提升网页加载速度。**
- **添加代理：用于翻墙访问某些页面，或者应对IP访问频率限制的反爬技术。**
- **使用移动头：访问移动端的站点，一般这种站点的反爬技术比较薄弱。**
- **设置编码：应对中文站，防止乱码。**
- 阻止JavaScript执行。

Options 就是一个配置 chrome 启动是属性的类。通过这个类，我们可以为chrome配置如下参数：

- 设置 chrome 二进制文件位置 (binary_location)
- **添加启动参数 (add_argument)**
- 添加扩展应用 (add_extension, add_encoded_extension)
- 添加实验性质的设置参数 (add_experimental_option)
- 设置调试器地址 (debugger_address)

```
# 导入chrome选项
from selenium.webdriver.chrome.options import Options

# 建立Option类对象
chrome_options = Options()

# '--headless'设置为无头模式
chrome_options.add_argument('--headless')

# 设置默认编码为 utf-8，也就是中文
chrome_options.add_argument('lang=zh_CN.UTF-8')

# 设置头部user-agent
chrome_options.add_argument('user-agent="Mozilla/5.0 (iPod; U; CPU iPhone OS 2_1 like Mac OS X; ja-jp) AppleWebKit/525.18.1 (KHTML, like Gecko) Version/3.1.1 Mobile/5F137 Safari/525.20"')
# 移动版网站的反爬虫的能力比较弱，可以模拟移动设备
# 模拟android QQ浏览器
chrome_options.add_argument('user-agent="MQQBrowser/26 Mozilla/5.0 (Linux; U; Android 2.3.7; zh-cn; MB200 Build/GRJ22; CyanogenMod-7) AppleWebKit/533.1 (KHTML, like Gecko) Version/4.0 Mobile Safari/533.1"')
# 模拟iPhone 6
chrome_options.add_argument('user-agent="Mozilla/5.0 (iPhone; CPU iPhone OS 9_1 like Mac OS X) AppleWebKit/601.1.46 (KHTML, like Gecko) Version/9.0 Mobile/13B143 Safari/601.1"')

# 设置代理（一定要注意，等号两边不能有空格）
chrome_options.add_argument("--proxy-server=http://202.20.16.82:10152")
# 添加代理
# 静态IP：102.23.1.105:2005
# 阿布云动态IP：http://D37EPSERV96VT4W2:CERU56DAEB345HU90@proxy.abuyun.com:9020
# 尽量选择静态IP，才能提升爬取的稳定性。如果使用动态匿名IP，每个IP的存活时间是很短的（1~3分钟）。
PROXY = "proxy_host:proxy:port"
desired_capabilities = chrome_options.to_capabilities()
desired_capabilities['proxy'] = {
    "httpProxy": PROXY,
    "ftpProxy": PROXY,
    "sslProxy": PROXY,
    "noProxy": None,
    "proxyType": "MANUAL",
    "class": "org.openqa.selenium.Proxy",
    "autodetect": False
}

# 设置图片不加载
chrome_options.add_argument('blink-settings=imagesEnabled=false')

# 携带Cookie
chrome_options.add_argument("user-data-dir=selenium") 

# 防屏蔽检测
chrome_options.add_experimental_option('excludeSwitches', ['enable-automation'])

# 阻止js
prefs = {'profile.default_content_setting_values': {'javascript': 2,}}
chrome_options.add_experimental_option('prefs', prefs)

# 禁止加载图片
prefs = {'profile.default_content_setting_values': {'images': 2,}}
chrome_options.add_experimental_option('prefs', prefs)

# 启用配置（上面的配置不启动，就无效）
browser = webdriver.Chrome(chrome_options=chrome_options)
# 但这样启动配置会报警告DeprecationWarning，原因是chrome_options已经被弃用，使用options来代替chrome_options，即可
browser = webdriver.Chrome(options=chrome_options)
```

##### 无头获取代码

```
# 注释：因为是无头模式，运行程序，不会有浏览器界面的显示

# 导入selenium的浏览器驱动接口
from selenium import webdriver

# 导入chrome选项
from selenium.webdriver.chrome.options import Options

# 建立Option类对象
chrome_options = Options()

# '--headless'设置为无头模式
chrome_options.add_argument('--headless')

# 启用配置（上面的配置不启动，就无效）
browser = webdriver.Chrome(options=chrome_options)

# get方法访问传入url（淘宝网址）
browser.get('https://www.taobao.com')

# page_source方法输出网页的源代码
print(browser.page_source)

# 关闭浏览器
browser.close()

输出：
<title>淘宝网 - 淘！我喜欢</title>
<meta name="spm-id" content="a21bo" />
<meta name="description" content="淘宝网 - 亚洲较大的网上交易平台，提供各类服饰、美容、家居、数码、话费/点卡充值… 数亿优质商品，同时提供担保交易(先收货后付款)等安全交易保障服务，并由商家提供退货承诺、破损补寄等消费者保障服务，让你安心享受网上购物乐趣！" />...
```

### 结点

​	selenium 可以驱动浏览器完成填充表单、模拟点击、输入框输入文字等各种操作，完成这些操作就需要知道节点位置。Selenium 提供了一系列查找节点的方法，利用这些方法来获取想要的节点。

##### 单节点

节点有许多其他属性，就可以通过属性方法获取它

```
# 选择name为q的节点
浏览器对象.find_element_by_name('q')

# 选择id为q的节点
浏览器对象.find_element_by_id('q')

# 利用css选择器选择id为q的节点
浏览器对象.find_element_by_css_selector('#q')

# 利用xpath选择器选择id为q的节点
浏览器对象.find_element_by_xpath('//*[@id="q"]')

# 利用q节点文本内容来查找q节点
浏览器对象.find_element_by_link_text('下一页')
```

单节点内容获取

```
from selenium import webdriver
browser = webdriver.Chrome()
browser.get('https://www.taobao.com')

# 选择id为q的节点
first = browser.find_element_by_id('q')

# 利用css选择器选择id为q的节点
second = browser.find_element_by_css_selector('#q')

# 利用xpath选择器选择id为q的节点
third = browser.find_element_by_xpath('//*[@id="q"]')

print(first)
print(second)
print(third)

输出：
<selenium.webdriver.remote.webelement.WebElement (session="50d72640dfc78b86f03126fe055de764", element="0.9212112373187529-1")>
<selenium.webdriver.remote.webelement.WebElement (session="50d72640dfc78b86f03126fe055de764", element="0.9212112373187529-1")>
<selenium.webdriver.remote.webelement.WebElement (session="50d72640dfc78b86f03126fe055de764", element="0.9212112373187529-1")>
# 可以看出结果一样，说明根据ID、css选择器和XPath选择器都选到了相同的节点。
```

##### 多节点

**注意：单个节点是find_element...，多个节点是find_elements...(多一个s)**

**坑：有时候需要定位两个有相同id、class的元素的第二个元素内容，这时候就要browser.find_elements_by_id('...')[1]通过最后列表定位**

```
from selenium import webdriver

browser = webdriver.Chrome()
browser.get('https://www.taobao.com')

# css多节点选择
last1 = browser.find_elements_by_css_selector('.service-bd li')
# xpath多节点选择
last2 = browser.find_elements_by_xpath('//*[@class="service-bd"]/li')

print(last1)
print(last2)

输出：
[<selenium.webdriver.remote.webelement.WebElement (session="ab0c34c539d914394c1e51d789349796", element="0.02681066407470678-3")...]
[<selenium.webdriver.remote.webelement.WebElement (session="ab0c34c539d914394c1e51d789349796", element="0.02681066407470678-3")...]
```

##### 节点属性

```
from selenium import webdriver

browser = webdriver.Chrome()
browser.get('https://www.zhihu.com/explore')

# 在页面中找到id为zh-top-link-logo的节点
logo = browser.find_element_by_id('zh-top-link-logo')
# 输出logo节点对象
print(logo)							# <selenium.webdriver...>
# 输出logo节点的class属性
print(logo.get_attribute('class'))	# zu-top-link-logo

# 在页面中找到class为zu-top-add-question的节点
input = browser.find_element_by_class_name('zu-top-add-question')
# 通过text属性获取文本内容
print(input.text)					# 提问
# 获取div块里面的内容
div = browser.find_element_by_xpath('...')
print(div.get_attribute("innerHTML"))
# 通过id属性获取到节点的id
print(input.id)						# 0.2681338057896978-2
# 通过location属性获取到节点在页面中的相对位置
print(input.location)				# {'x': 761, 'y': 7}
# 通过tag_name属性可以获取到标签名称
print(input.tag_name)				# button
# 通过size属性可以获取到节点的大小
print(input.size)					# {'height': 32, 'width': 66}
```

### 其他操作

##### 刷新退出

```
# 刷新当前窗口
browser.refresh()

# 获取当前页面的url
browser.current_url

# 退出浏览器，关闭全部窗口
browser.quit()
```

##### 活动窗口

当我们使用page_source输出网页源码时，输出的是browser.get()中url的页面源码，假若selenium在页面中打开了新的窗口，再使用page_source输出网页源码，仍然是旧窗口页面的源码，这是因为selenium的活动窗口在旧窗口，要输出新窗口的页面源码就需要切换活动窗口。

而如果是打开多个网页。首先打开网页1，再点击网页1上一个链接，在新窗口打开新网页2，同理点击新网页2上一个链接，新窗口打开新网页3等等，这是按打开顺序为网页编号。实际网页对应句柄如下图所示，即第一个打开网页一直是句柄0，而最新打开的网页记为句柄1，依次序编号。

标签页顺序（**按照打开顺序**）：1 2 3 4 5，对应的句柄 ：0 4 3 2 1

```
# 句柄
n = browser.window_handles
# 获取当前窗口列表n,切换第一个标签n[0]为活动窗口
browser.switch_to.window(n[0])
# 输出第活动窗口网页源码
print(browser.page_source)

# 点击产生新窗口新页面
browser.find_element_by_xpath(...).click()

# 句柄
n = browser.window_handles
# 切换最后一个标签n[1]为活动窗口，就是新打开的窗口为活动窗口
browser.switch_to.window(n[1])
# 输出活动窗口网页源码
print(browser.page_source)

# 关闭活动窗口
browser.close()

# 退出浏览器，关闭全部窗口
driver.quit()
```

##### 截屏

**注意：有头模式和无头模式都能进行截屏**

```
# 窗口截屏（只对窗口显示的部分网页进行截屏）
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
# 创建设置类
chrome_options = Options()
# 设置无头模式
chrome_options.add_argument('--headless')
driver = webdriver.Chrome(options=chrome_options)
# 访问百度页面
driver.get('https://www.baidu.com')
# 生成当前页面快照并保存
driver.save_screenshot("baidu.png")

# 网页截屏（对整个网页进行截屏）
# 方法一：使用Chrome浏览器自带的网页截屏功能
# 导入键盘操作
from pynput.keyboard import Key,Controller
# 访问网页
driver.get('...')
# 生成控制对象
keyboard = Controller()
time.sleep(3)
# 按下释放f12
keyboard.press(Key.f12)
keyboard.release(Key.f12)
time.sleep(2)
# 快捷键ctrl+shift+p
with keyboard.pressed(Key.ctrl, Key.shift):
keyboard.press('p')
keyboard.release('p')
time.sleep(2)
# 输入功能进行截图
keyboard.type('capture full size screenshot')
time.sleep(2)
keyboard.press(Key.enter)
keyboard.release(Key.enter)
time.sleep(2)
# 退出截屏
keyboard.press(Key.f12)
keyboard.release(Key.f12)
time.sleep(2)

# 方法二：使用phantomjs无头浏览器执行js代码来网页截屏
# 下载phantomjs无头浏览器，将执行文件phantomjs.exe文件放到环境变量中（最好直接放入Python文件夹中的Script文件夹下，前提是该文件夹已在环境变量中）。
# 将下面js代码保存为WebToImage.js文件
var page = require('webpage').create();
var system = require('system');
page.viewportSize = { width: 1366, height: 100};
page.open(system.args[1], function(status){
    console.log('Status:' + status);
    if(status === "success") {
        page.render(system.args[2]);
    }
    phantom.exit();
});
# 获取当前网页url
url = browser.current_url
h = hashlib.md5()
tmd5 = url.rstrip()
h.update(tmd5.encode('utf-8'))
# 执行DOS命令
command = 'phantomjs WebToImage.js {} {}'.format(tmd5, 'F:\\Image\\' + h.hexdigest() + '.jpg')
os.system(command)
```

##### 输入点击操作

```
from selenium import webdriver
from selenium.webdriver.chrome.options import Options

driver = webdriver.Chrome()

driver.get('https://www.baidu.com')

# id="kw"是百度搜索输入框，输入字符串"程序猿"
driver.find_element_by_id('kw').send_keys('程序猿')

# id="su"是百度搜索按钮，click() 是模拟点击
driver.find_element_by_id("su").click()
```

##### 动作链

```
from selenium import webdriver
# 导入链式动作
from selenium.webdriver import ActionChains

# 浏览器对象
browser = webdriver.Chrome()
# 滑块对象
slider = find_element_by_id('...')
# 滑动链表
track = [0, 0, 0, 1, 1, 1, 1, 1, 1, 2, 2, 2, 2, 2, 2, 2]

# click_and_hold保持按住
ActionChains(browser).click_and_hold(slider).perform()
for x in track:
    # xoffse水平位移, yoffset垂直位移
    ActionChains(browser).move_by_offset(xoffset=x, yoffset=0).perform()
time.sleep(0.5)
ActionChains(browser).release().perform()

# move_to_element移动到该元素
# 有些内容是隐藏在<ul><li>标签下的，用xpath直接定位是定位不到的，只能一层一层的指向
# 移动到button1
button1 = wait.until(EC.presence_of_element_located((By.XPATH, '//*[@id="wholeMenu"]/li[4]/a')))
ActionChains(browser).move_to_element(button1).perform()
# 移动到button1下的button2
button2 = wait.until(EC.element_to_be_clickable((By.XPATH, '//*[@id="company_secondNavs"]/li[1]/a')))
ActionChains(browser).move_to_element(button2).perform()
# 移动到button1下的button2下的button3
button3 = wait.until(EC.element_to_be_clickable((By.XPATH, '//*[@id="company_secondNavs"]/li[1]/ul/li[1]/a')))
ActionChains(browser).move_to_element(button3).perform()
# 点击button3
button3.click()
```

##### 键盘操作

```
from selenium import webdriver
from selenium.webdriver.common.by import By
# 调用键盘按键操作引入keys包
from selenium.webdriver.common.keys import Keys

driver = webdriver.Chrome()

driver.get('https://www.baidu.com')

# id="kw"是百度搜索输入框，输入字符串"程序猿"
driver.find_element_by_id('kw').send_keys('程序猿')

# ctrl+a 全选输入框内容
driver.find_element_by_id("kw").send_keys(Keys.CONTROL, 'a')

# ctrl+x 剪切输入框内容
driver.find_element_by_id("kw").send_keys(Keys.CONTROL, 'x')

# 输入框重新输入"美女"
driver.find_element_by_id("kw").send_keys("美女")

# id="su"是百度搜索按钮，模拟Enter回车键
driver.find_element_by_id("su").send_keys(Keys.ENTER)
```

##### 前进后退

浏览器都有前进和后退功能， Selenium 也可以完成这个操作，它使用 back （）方法后退，使用forward（）方法前进。

```
import time 
from selenium import webdriver

browser = webdriver.Chrome() 
browser.get('https://www.baidu.com/')
browser.get('https://www.taobao.com/')
browser.get('https://www.python.org/')

# 从python页面返回到淘宝页面
browser.back()
# 休息2两秒
time.sleep(2)
# 再从淘宝页面前进到python页面
browser.forward()
browser.close()
```

##### Cookies操作

获取、添加、删除 Cookies等。

```
from selenium import webdriver

browser = webdriver.Chrome() 
browser. get('https://www.zhihu.com/explore')

# get_cookies()方法获取所有的cookies
print(browser.get_cookies())

# 添加一个cookie，传入一个字典，有name、domain等内容
browser.add_cookie({'name':'name','domain':'www.zhihu.com','value':'germey'})
print(browser.get_cookies())

# delete_all_cookies()删除所有的cookies
browser.delete_all_cookies()
print(browser.get_cookies())


结果：
[{'domain': ...]
[{'domain': ... {'domain': 'www.zhihu.com', 'name': 'name', 'value': 'germey'}]
[]
```

##### 模拟JS

selenium还可以使用execute_script方法来模拟运行JavaScript。

```
# 将进度条下拉到最底部
浏览器对象.execute_script('window.scrollTo(0, document.body.scrollHeight)')

# 弹出alert提示框，内容是（To Bottom）
浏览器对象.execute_script('alert("To Bottom")')
```

### 延时等待

##### 举例说明

```
import time
# 导入浏览器驱动
from selenium import webdriver

# 驱动谷歌浏览器
browser = webdriver.Chrome()
url = 'https://www.hapag-lloyd.cn/zh/home.html#hal-map'
browser.get(url)

# 获取网页代码
print(browser.page_source)
print('=============================================================================')
# 暂停10秒
time.sleep(10)
# 获取网页代码
print(browser.page_source)

输出：
...<script>
(function(){
window["bobcmn"] = "111110101010102000000022000000052000000002d35d0af5200000096300000000300000000300000006/TSPD/300000008TSPD_101300000005https3000000b0081ecde62cab200043318e928c7db9eaaef388790629486f29c67cb3597ba57c3aa683042dc0876c082c083c2c0a2800cfed5cff81ca51b0470f7b9b40b2135d1901feff635b795bc44083cf2d481ece37ef1687633e22a820000000020000000...
</script>
<script type="text/javascript" src="/TSPD/081ecde62cab2000dcdb0f891d541a52e60a746a73e0298daad7b3af10474d6685159c974748d9cd?type=7"></script>
<noscript>Please enable JavaScript to view the page content.&lt;br/&gt;Your support ID is:  5824692628514319806.</noscript>
</head><body>
</body></html>

================================================================================

...</script>
<div id="hal-cookieconsent" class="hal-cookienotice" style="display: block;">
	<div class="hal-cookienotice-info">
		<p class="hal-cookienotice-info-text">我们使用Cookies来确保带给您网站的最佳体验。如果选择继续，将代表您同意接收这些Cookies。有关Cookie的详细信息，请参阅我们的<a href="/zh/meta/cookie-policy.html" class="hal-link hal-cookienotice-info-link" btattached="true">Cookie 政策</a>和 
 <a class="hal-link  hal-cookienotice-info-link" href="/zh/meta/privacy-statement.html" btattached="true">数据保护政策.</a></p>
	</div>...
```

##### 分析

​	上面的打印结果不一样，是因为当Selenium加载网页后马上获取的网页代码是未加载Js的，在等待10秒后Js加载完毕，这时再执行获取网页代码的方法，获取的就是加载Js的网页代码。这是因为Js加载是需要时间的。

​	**浏览器对象 . get方法会在网页框架加载结束后结束执行，此时获取 page_source ，可能并不是浏览器完全加载完成的页面，如有额外的 Ajax 请求、js加载，在网页源代码中也不一定能成功获取到。所以，这里需要延时等待，确保节点已经加载出来。**

##### 延时等待分类

​	在selenium中常用的等待分为**显式等待WebDriverWait()**、**隐式等待implicitly_wait()**、**强制等待sleep()**

​	WebDriverWait()：**显式等待，配合该类的until()和until_not()方法方法。在设置时间内，默认每隔0.5s检测一次，如果条件成立了，则执行下一步，否则继续等待，直到超过设置时间，然后抛出TimeoutException。**

​	implicitly_wait()：隐式等待，也叫智能等待，是 webdirver 提供的一个超时等待。隐式的等待一个元素被发现，或一个命令完成。如果超出了设置时间的则抛出异常。

​	sleep()： **强制等待，设置固定休眠时间。 python 的 time 包提供了休眠方法 sleep() ， 导入 time 包后就可以使用 sleep()，进行脚本的执行过程进行休眠。**

```
from selenium import webdriver
from selenium.webdriver.common.by import By
# 导入WebDriverWait，设置延时等待
from selenium.webdriver.support.ui import WebDriverWait
# 将导入的expected_conditions（期望的情况，判断是否）重命名为EC
from selenium.webdriver.support import expected_conditions as EC

browser = webdriver.Chrome()
browser.get(' https://www.taobao.com/')

# 设置等待时间为10秒
wait = WebDriverWait(browser, 10)

# 调用until（）方法，传入要等待的条件EC
# presence_of_element_located 代表定位节点，参数是节点的定位元组，也就是ID为q的节点搜索框。
# 在 10 秒内如果 ID 的节点（搜索框）成功加载出来，就返回该节点；如果超过 10 秒还没有加载出来，就抛出异常
input = wait.until(EC.presence_of_element_located((By . ID, 'q')))

# element_to_be_clickable代表可点击的，所以 css 选择器查找的为.btn-search的按钮
# 10 秒内它是可点击的，也就是成功加载出来了，就返回这个按钮节点；如果超过 10 秒还不可点击，也就是没有加载出来，就抛出异常。
button = wait.until(EC.element_to_be_clickable((By.CSS_SELECTOR, '.btn-search')))
print(input, button)

结果：
<selenium.webdriver.remote.webelement.WebElement (session="93257f160f06ca43cdf970e6e4d7ef34", element="0.24301452802066237-1")> <selenium.webdriver.remote.webelement.WebElement (session="93257f160f06ca43cdf970e6e4d7ef34", element="0.24301452802066237-2")>
```

### Iframe

有时候明明元素就在那，chrome浏览器中的Elements也可以看到，但是无论怎么定位还是抓取不到，而且page_source输出的源码中也没有该元素，这个时候就需要检查下页面是否含有iframe，如果你需要定位的元素是否在iframe之中，有可能就是因为iframe的问题。

##### Iframe介绍

iframe是内嵌的网页元素，也可以说是内嵌的框架，可以将一个HTML文档嵌入在另一个HTML中显示，网页中嵌入<Iframe>包含的内容与整个页面是一个整体

为什么定位元素是否在iframe中就抓取不到？

**这是因为iframe实际上是嵌入了另一个页面，而webdriver每次只能在一个页面识别**，因此需要先定位到相应的iframe，对那个页面里的元素进行定位。

##### Iframe定位抓取

```
# 先跳转到iframe框架
# reference参数用来定位frame，可以传入id、name、index，有动态id的情况，可以使用xpath定位，然后传入selenium的WebElement对象
browser.switch_to.frame('reference')

# 再通过xpath进行定位元素
browser.find_element_by_xpath('')

# 返回到上一个文档
browser.switch_to.parent_frame()

# 退回到主文档
# 切换进入frame之中后，我们就不能对主文档的元素进行操作了，如果需要操作，那么我们只能再切换回去
browser.switch_to.default_content()
```

##### Selenium设置代理

```
from selenium import webdriver

proxy = '127.0.0.1:9743'
chrome_options = webdriver.ChromeOptions()
chrome_options.add_argument('--proxy-server=http://' + proxy)
browser = webdriver.Chrome(chrome_options=chrome_options)
browser.get('http://httpbin.org/get')

结果：
{
  "args": {}, 
  "headers": {
    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8", 
    "Accept-Encoding": "gzip, deflate", 
    "Accept-Language": "zh-CN,zh;q=0.9", 
    "Cache-Control": "max-age=0", 
    "Connection": "close", 
    "Host": "httpbin.org", 
    "Upgrade-Insecure-Requests": "1", 
    "User-Agent": "Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.132 Safari/537.36"
  }, 
  "origin": "117.139.208.10", 
  "url": "http://httpbin.org/get"
}
```

### 