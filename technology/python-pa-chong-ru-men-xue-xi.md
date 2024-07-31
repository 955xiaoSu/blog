# Python 爬虫入门学习

### 前言

由于实验室项目需要大量的文本信息作为数据支撑，因此需要一种自动化获取信息的工具，**爬虫**（ 用程序在互联网上获取特定信息的一种手段）就进入了我的学习课程。爬虫需要一门编程语言作为基础，而**Python** 有着强大的第三方库，能为爬虫提供极大的便利，因此我选择 Python 这门编程语言作为爬虫的基础。

### 学习路线

1. **文档**：[Python Tutorial官方文档](https://docs.python.org/3/tutorial/appetite.html) —> [崔庆才《Python3网络爬虫开发实战》](https://cuiqingcai.com/17777.html)
2. **课程**：[B站Python入门课程天花板](https://www.bilibili.com/video/BV12E411A7ZQ?spm\_id\_from=333.1007.top\_right\_bar\_window\_view\_later.content.click\&vd\_source=68401e073d6cf69f6f72f1ad56c67eaf)

### 爬虫全过程分析：

#### **爬虫思路**

由于目的是为了在互联网上获取特定的信息，因此需要以下解决几个问题：

> 1. 目标资源在互联网的哪一个位置；
> 2. 目标资源在前端（互联网的具体页面）是如何体现的；
> 3. 锁定目标资源后，如何获取相关数据；
> 4. 如何存储有关数据。

#### **解决方案**：

对应上述每一点问题，提供对应解决方法。

> 1. 进入到目标资源所在的页面后，查看浏览器的地址栏，将地址栏中的内容复制即可。\
>    **补充**：地址栏中的内容为统一资源定位符（Uniform Resource Locator, URL），是因特网上标准的资源的地址。好比你现在要去拜访一个人，那么URL就是这个人的家庭地址，你可以根据URL找到你要拜访的那个人。URL的一般形式是：_协议 + 服务器 + 相对文件路径 + 文件名_，例子：`https://github.com/ryanhanwu/How-To-Ask-Questions-The-Smart-Way/blob/main/README-zh_CN.md`，`https`为访问协议，`github.com`为服务器`20.205.243.166`对应解析的域名(域名与服务器是一一对应的，二者转换是通过DNS服务器完成的)，`ryanhanwu/How-To-Ask-Questions-The-Smart-Way/blob/main`是文件相对于服务器根目录的相对路径，README-zh\_CN.md为资源的文件名。
> 2. `F12` / `Fn + F12`打开开发者工具，点击左上角的小箭头（目的是为了使用检查元素功能），此时在页面中点击想获取的资源，便会自动定位到资源对应的前端代码，观察前端代码为下一步进行数据提取做好准备（参考 Fig.1）。         &#x20;
> 3. 定位好前端代码后，紧接着是使用正则表达式进行数据获取。**正则表达式（regular expression, re）** 是一种字符串匹配模式，可以用来判定一个字符串中是否含有某种子串、进行字符串替换或者取出字符串中符合某个条件的子串等，详细内容可参考[菜鸟教程：正则表达式](https://www.runoob.com/regexp/regexp-syntax.html)。为了找出相应资源的前端代码，首先要使用 re.compile() 方法规定出相应资源的“模样”，其次再用这种“模样”去匹配前端代码。这就像相亲，你先在心目中构建一个对伴侣的期待，然后在海一般的相亲市场中根据你的期待去匹配心仪的相亲对象。此时你的期待就是“模样”，相亲市场就是网页中所有的前端代码，现在要做的就是在所有的前端代码中找出你想要的“模样”代码，也就是我们想要的资源（即下文所称的数据）。
> 4. 使用正则表达式成功匹配数据之后，可以使用python自带的**json模块**，把数据以.json形式存储到本地。当然也可以引入像**MySQL / sqlite**之类的数据库，使用SQL（Structured Query Language）语言，把数据以.db形式存储到本地

<figure><img src="../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>Fig.1</p></figcaption></figure>

#### **样例分析**：

以下代码为爬取`https://ssr1.scrape.center`所有电影信息的样例代码。

```
					 # 引入各个模块的作用说明
import requests  			 # 访问URL的工具
import logging   			 # 以日志形式进行结果输出
import re	     		         # 引入正则表达式
from urllib.parse import urljoin         # 实现URL的拼接
import json            			 # 存储数据
from os import makedirs 		 # 创建用来存储数据的文件夹
from os.path import exists		 # 判断文件夹是否存在，辅助文件夹的创建
import multiprocessing			 # 多线程爬取，加快爬取速度 

logging.basicConfig(level=logging.INFO,	
                    format='%(asctime)s-%(levelname)s:%(message)s') # 设置日志输出模式
BASE_URL = 'https://ssr1.scrape.center'				    # 设置基本URL
TOTAL_PAGE = 10														# 目标网站共十页
RESULTS_DIR = 'results'											    # 创建名为results的文件夹 
exists(RESULTS_DIR) or makedirs(RESULTS_DIR)

# 功能：返回对应URL的前端代码
# 参数：URL
# 返回值类型：string
def scrape_page(url):
    logging.info('scraping %s...', url)
    try: 
        response = requests.get(url)    # 对目标URL进行访问
        if response.status_code == 200: # 对状态码进行判断，如果返回状态码200说明连接正常
            return response.text
        logging.error('get invalid status code %s while scraping %s', 
                      response.staus_code, url)
    except requests.RequestException:   # 对错误的堆栈信息进行追踪
        logging.error('error occurred while scraping %s', url,
                      exc_info=True)

# 功能：返回对应URL的前端代码
# 参数：网站当前页数
# 返回值类型：string
def scrape_index(page):
    index_url = f'{BASE_URL}/page/{page}' # 'f'参数使得变量可以填充字符串
    return scrape_page(index_url)

# 功能：获取每部电影对应的URL
# 参数：网页的前端代码
# 返回值类型：string
def parse_index(html):
    pattern = re.compile('<a.*?href="(.*?)".*?class="name">') # 用re构造出每部电影对应href的匹配模式
    items = re.findall(pattern, html)                         # 在前端代码中查找与pattern相匹配的代码
    if not items:
        return []
    for item in items:
        detail_url = urljoin(BASE_URL, item) # 观察每部电影URL可知，URL = 基本URL + items
        logging.info('get detail url %s', detail_url)
        yield detail_url                     # 关键字yield的机制是，暂停当前运行进程、输出当前结果并保存状态

# 此函数作用同scrape_page()完全相同
# 构造此函数的目的：为日后进行爬取拓展提供接口
def scrape_detail(url): 
    return scrape_page(url)
    
# 功能：定位目标资源的前端代码，而后返回相应的数据
# 参数：对应每部电影的前端代码
# 返回值类型：dictionary
def parse_detail(html):
    # 分别对电影的封面、名称、分类、发行时间、评分、概要设置相应的re匹配规则
    cover_pattern = re.compile('class="item.*?<img.*?src="(.*?)".*?class="cover">', re.S) # Make '.' match all characters
    name_pattern = re.compile('<h2.*?>(.*?)</h2>')
    categories_pattern = re.compile('<button.*?category.*?<span>(.*?)</span>.*?</button>', re.S)
    published_at_pattern = re.compile('(\d{4}-\d{2}-\d{2})\s上映') # As for time, use standard (\d{4}-\d{2}-\d{2})
    score_pattern = re.compile('<p.*?score.*?>(.*?)</p>', re.S)
    drama_pattern = re.compile('<div.*?drama.*?>.*?<p.*?>(.*?)</p>', re.S)
    
    # 在前端代码中查找相应的匹配数据
    cover = re.search(cover_pattern, html).group(1).strip() # strip delete space in start and end part
    name = re.search(name_pattern, html).group(1).strip()
    categories = re.findall(categories_pattern, html)
    published_at = re.search(published_at_pattern, html).group(1) if re.search(published_at_pattern, html) else None
    score = float(re.search(score_pattern, html).group(1).strip())
    drama = re.search(drama_pattern, html).group(1).strip() 
    
    # 将所获取的数据以字典的形式进行返回
    return {
        'cover': cover,
        'name': name,
        'categories': categories,
        'published_at': published_at,
        'drama': drama,
        'score': score
    }

# 功能：将获取到的数据以.json形式存储
# 参数：数据
# 返回值类型：None
def save_data(data):
    name = data.get('name')			            # 获取电影名称
    data_path = f'{RESULTS_DIR}/{name}.json'                # 根据电影名称，规定相应电影的文件名
    json.dump(data, open(data_path, 'w', encoding='utf-8'), 
                         ensure_ascii=False, indent=2)      # 将数据写入文件

# 功能：爬取并保存电影信息
# 参数：网站当前页数
# 返回值类型：None
def main(page):
    index_html = scrape_index(page)             # 获取当前页面下的前端代码
    detail_urls = parse_index(index_html)       # 获取当前页面下所有电影的URL
    for detail_url in detail_urls:
        detail_html = scrape_detail(detail_url) # 获取每部电影页面的前端代码
        data = parse_detail(detail_html)	# 获取每部电影的信息
        logging.info('get detail data %s', data)
        logging.info('save data to json data')
        save_data(data)				# 将获取到的信息进行存储
        logging.info('data saved successfully')

if __name__ == '__main__':
    pool = multiprocessing.Pool(processes=4) # 构造进程池，并将进程数设为4
    pages = range(1, TOTAL_PAGE + 1)	     # 构造页面数
    pool.map(main, pages)                    # 将 pages 作为参数传入到 main() 当中，
    					     # 并将每一次 main() 函数调用作为一个进程，
    					     # 加入到进程池当中
    pool.close()  			     # 等待worker进程结束再关闭进程池
    pool.join()   			     # 防止主程序在worker进程结束前结束

```
