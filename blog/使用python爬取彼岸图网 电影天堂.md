> 前端时间学习Python随手写了两个爬虫，记录一下

### 爬取彼岸图网略缩图
```python
import requests
import re


class Getpictur:
    url = 'http://pic.netbian.com/'
    headers = {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.88 Safari/537.36"}
    img = []


    def getdata(self):
        for i in range(2, 6):
            zongurl = self.url + 'index_{}.html'.format(i)
            response = requests.get(zongurl,self.headers)
            response.encoding = 'gbk'
            regular_path = re.compile(r'(?<=<a href=")/tupian/.*?.html(?=")')
            child_path = regular_path.findall(response.text)
            for item in child_path:
                child_url = self.url + item
                child_response = requests.get(child_url, self.headers)
                child_response.encoding = 'gbk'
                self.getchild_data(child_response)


    def getchild_data(self, response):
        regular_img = re.compile(r'(?<=<img src=").*(?=" data-pic)')
        child_child_path = regular_img.findall(response.text)
        for item in child_child_path:
            print(self.url + item)
            self.img.append(self.url + item)


    def downloadImg(self):
        for item in self.img:
            print("开始下载" + item)
            DownloadPath = 'tupian/' + str(item).split('-')[1]
            r = requests.get(item)
            with open(DownloadPath, 'wb') as f:
                f.write(r.content)
            print(item + "下载完成")
        f.close()


if __name__ == '__main__':
    x = Getpictur()
    x.getdata()
    x.downloadImg()
```

### 爬取电影天堂种子文件
```python
import requests
import re
from lxml import etree


class GetMovie:
    headers = {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.88 Safari/537.36"}
    url = 'https://www.dytt8.net/'
    magnetlist = []
    moviename = []


    def start_request(self):
        # for i in range(1, 207):
        #     url = 'https://www.dytt8.net/html/gndy/dyzz/list_23_{}.html'.format(i)
        url = 'https://www.dytt8.net/html/gndy/dyzz/list_23_1.html'
        response = requests.get(url, self.headers)
        response.encoding = 'gbk'
        regular_one = re.compile(r'(?<=<a href=").*(?=" class="ulink">)')
        regular_two = re.compile(r'(?<=class="ulink">).*(?=</a>)')
        for j in regular_two.findall(response.text):
            self.moviename.append('magnet/' + j + '.txt')
        path = regular_one.findall(response.text)
        for item in path:
            child_url = self.url + item
            self.getSave(child_url)


    def getSave(self, url):
        response = requests.get(url, self.headers)
        response.encoding = 'gbk'
        html = etree.HTML(response.text)
        result = html.xpath('//a/@href')
        for i in result:
            regular = re.compile(r'magnet.*')
            if regular.findall(i):
                self.magnetlist.append(regular.findall(i)[0])


    def Savemagnet(self):
        list = {k:v for k,v in zip(self.moviename, self.magnetlist)}
        for item in list.keys():
            with open(item, 'a+') as f:
                f.write(list[item])
        f.close()


if __name__ == '__main__':
    x = GetMovie()
    x.start_request()
    x.Savemagnet()
```