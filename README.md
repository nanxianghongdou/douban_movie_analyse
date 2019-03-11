# douban_movie_analyse
爬取豆瓣TOP250电影，并进行数据分析
#!/usr/bin/env python
# _*_coding:utf-8_*_
# Author: DDZZxiaohongdou
import requests
from bs4 import BeautifulSoup
from pandas import DataFrame
import pandas as pd
from matplotlib import pyplot as plt
import numpy as np
from tkinter import *
import os
import ch
import jieba
from wordcloud import WordCloud, STOPWORDS, ImageColorGenerator
from terminaltables import AsciiTable
from tkinter import scrolledtext

URL = 'http://movie.douban.com/top250'
path_result = 'E:/python code/douban_movie_analyse/douban_movie_analyse/result/'

#获取网页原始数据
def download_page(url):
    headers = {
        'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/47.0.2526.80 Safari/537.36'
    }
    data = requests.get(url, headers=headers).content
    return data

#从网页中提取感兴趣文本
def get_information(doc):
    soup = BeautifulSoup(doc, 'html.parser')
    ol = soup.find('ol', class_='grid_view')
    name = []                                   #电影名字
    film_reviewer = []                          #影评人数
    score = []                                  #影评分数
    quteo = []                                  #短评
    year = []                                  #电影上映年份
    country = []                               #拍摄电影国家
    for i in ol.find_all('li'):
        #1获取电影名字
        detail = i.find('div', attrs={'class': 'hd'})
        movie_name = detail.find( 'span', attrs={'class': 'title'}).get_text()
        name.append(movie_name)
        #2获取影评人数
        star = i.find('div', attrs={'class': 'star'})
        star_num = star.find(text=re.compile('评价'))
        film_reviewer.append(star_num)
        #3获取评分
        level_star = i.find('span', attrs={'class': 'rating_num'}).get_text()
        score.append(level_star)
        #4获取短评
        info = i.find('span', attrs={'class': 'inq'})
        if info:
            quteo.append(info.get_text())
        else:
            quteo.append('无')
        #56获取电影的上映年份和拍摄国家
        detail = i.find('div', attrs={'class': 'bd'})
        label = detail.find('p')
        information = label.find(text = re.compile(r'\d{4}'))
        information = information.split()
        new_information = []
        for j in information:
            if (j!='/'):
                new_information.append(j)
        year.append(new_information[0])
        country.append(new_information[1])
    # 获取下一页
    page = soup.find('span', attrs={'class': 'next'}).find('a')
    if page:
        return name, film_reviewer, score, quteo, year, country, URL + page['href']
    return name, film_reviewer, score, quteo, year, country, None

def get_all_information():
    url = URL
    name = []                                   #电影名字
    film_reviewer = []                          #影评人数
    score = []                                  #影评分数
    quteo = []                                  #短评
    year = []                                  #电影上映年份
    country = []                               #拍摄电影国家
    while url:
        doc = download_page(url)
        name1, film_reviewer1, score1, quteo1, year1, country1, url = get_information(doc)
        #进行数组合并
        name = name + name1
        film_reviewer = film_reviewer + film_reviewer1
        score = score + score1
        quteo = quteo + quteo1
        year = year + year1
        country = country + country1
    L = len(name)
    data = [name, film_reviewer, score, quteo, year, country]

    data_csv = DataFrame(data, index=['name', 'film_reviewer', 'score', 'quteo', 'year', 'country'],columns=np.array(range(L)))
    data_csv = data_csv.T
    if not os.path.exists(path_result):
        os.makedirs(path_result)
    filename = path_result + 'result.csv'
    data_csv.to_csv(filename, index=False,encoding='utf-8-sig')

def get_data():
    filename = path_result + 'result.csv'
    data_csv = pd.read_csv(filename, encoding='gbk')
    return data_csv

def data_analyse_button1(data):
    country_count = []
    country_name = []
    country_dict = {'美国': 0, '中国大陆': 0, '法国': 0, '意大利': 0, '日本': 0, '印度': 0, '香港': 0, '韩国': 0, '德国': 0,
                 '英国':0, '新西兰':0, '台湾':0,'西班牙':0, '澳大利亚':0, '丹麦':0, '巴西':0, '阿根廷':0, '伊朗':0, '爱尔兰':0,
                 '瑞典':0, '泰国':0, '博茨瓦纳':0 }
    country_list = ['美国', '中国大陆', '法国', '意大利', '日本', '印度', '香港', '韩国', '德国',
                 '英国', '新西兰', '台湾','西班牙', '澳大利亚', '丹麦', '巴西', '阿根廷', '伊朗', '爱尔兰',
                 '瑞典', '泰国', '博茨瓦纳']
    for i in data.country:
        if i in country_list:
            country_dict[i] = country_dict[i] + 1
    for i in range(len(country_list)):
        country_count.append(country_dict[country_list[i]])
    for i in country_dict:
        country_name.append(i)
    return country_name, country_count
def data_analyse_button1_figure(country_name, country_count):
    fig = plt.figure(1)
    ax1 = plt.subplot(111)
    data = country_count
    width = 0.5
    x_bar = np.arange(len(country_count))
    rect = ax1.bar(left=x_bar, height=data, width=width, color="lightblue")
    for rec in rect:
        x = rec.get_x()
        height = rec.get_height()
        ax1.text(x + 0.1, 1.02 * height, str(height))
    ax1.set_xticks(x_bar)
    ax1.set_xticklabels(country_name)
    ax1.set_ylabel("num")
    ax1.set_title("TOP 250 movie country distribution")
    ax1.grid(True)
    ax1.set_ylim(0, 150)
    plt.show()
def data_analyse_button2(data):
    movie_year = []
    movie_count = []
    movie_dict =  {1931:0, 1936:0, 1939:0, 1940:0, 1942:0, 1950:0, 1952:0, 1953:0, 1954:0, 1957:0, 1960:0, 1961:0, 1965:0, 1966:0, 1971:0,
                  1972:0, 1974:0, 1975:0, 1979:0, 1980:0, 1982:0, 1986:0, 1987:0, 1988:0, 1989:0, 1990:0, 1991:0, 1992:0, 1993:0, 1994:0,
                  1995:0, 1996:0, 1997:0, 1998:0, 1999:0,2000:0, 2001:0, 2002:0, 2003:0, 2004:0, 2005:0, 2006:0, 2007:0, 2008:0, 2009:0,
                  2010:0, 2011:0, 2012:0, 2013:0, 2014:0, 2015:0, 2016:0, 2017:0}
    movie_list = [1931, 1936, 1939, 1940, 1942, 1950, 1952, 1953, 1954, 1957, 1960, 1961, 1965, 1966, 1971, 1972, 1974,
                  1975, 1979, 1980,1982, 1986, 1987, 1988, 1989, 1990, 1991, 1992, 1993, 1994, 1995, 1996, 1997, 1998,
                  1999, 2000, 2001,2002, 2003, 2004, 2005,2006, 2007, 2008, 2009, 2010, 2011, 2012, 2013, 2014, 2015, 2016,
                  2017]
    for i in data.year:
        if i in movie_list:
            movie_dict[i] = movie_dict[i] + 1
    for i in range(len(movie_list)):
        movie_count.append(movie_dict[movie_list[i]])
    for i in movie_dict:
        movie_year.append(i)
    return movie_year, movie_count
def data_analyse_button2_figure(movie_year, movie_count):
    fig = plt.figure(1)
    ax1 = plt.subplot(111)
    data = movie_count
    width = 0.5
    x_bar = np.arange(len(movie_count))
    rect = ax1.bar(left=x_bar, height=data, width=width, color="lightblue")
    for rec in rect:
        x = rec.get_x()
        height = rec.get_height()
        ax1.text(x + 0.1, 1.02 * height, str(height))
    ax1.set_xticks(x_bar)
    ax1.set_xticklabels(movie_year, rotation=30)
    ax1.set_ylabel("num")
    ax1.set_title("TOP 250 movie year distribution")
    ax1.grid(True)
    ax1.set_ylim(0, 50)
    fig.get_tight_layout()
    plt.show()
def data_analyse_button3(data):
    movie_score = []
    movie_score_count = []
    movie_dict = {8:0, 8.1:0, 8.2:0, 8.3:0, 8.4:0, 8.5:0, 8.6:0, 8.7:0, 8.8:0, 8.9:0, 9.0:0, 9.1:0, 9.2:0, 9.3:0, 9.4:0,
                  9.5:0, 9.6:0, 9.7:0, 9.8:0, 9.9:0, 10:0}
    movie_list = [8, 8.1, 8.2, 8.3, 8.4, 8.5, 8.6, 8.7, 8.8, 8.9, 9.0, 9.1, 9.2, 9.3, 9.4, 9.5, 9.6, 9.7, 9.8, 9.9, 10]
    for i in data.score:
        if i in movie_list:
            movie_dict[i] = movie_dict[i] + 1
    for i in range(len(movie_list)):
        movie_score_count.append(movie_dict[movie_list[i]])
    for i in movie_dict:
        movie_score.append(i)
    return movie_score, movie_score_count
def data_analyse_button3_figure(movie_score, movie_score_count):
    fig = plt.figure(1)
    ax1 = plt.subplot(111)
    data = movie_score_count
    width = 0.5
    x_bar = np.arange(len(movie_score))
    rect = ax1.bar(left=x_bar, height=data, width=width, color="lightblue")
    for rec in rect:
        x = rec.get_x()
        height = rec.get_height()
        ax1.text(x + 0.1, 1.02 * height, str(height))
    ax1.set_xticks(x_bar)
    ax1.set_xticklabels(movie_score)
    ax1.set_ylabel("num")
    ax1.set_title("TOP 250 movie score distribution")
    ax1.grid(True)
    ax1.set_ylim(0, 50)
    fig.get_tight_layout()
    plt.show()
def work_cloud_visualization(data):
    words = ''
    word_list = []
    girl_image = plt.imread('girl.jpg')
    wc = WordCloud(background_color='white',  # 背景颜色
                   max_words=1000,  # 最大词数
                   mask= girl_image,  # 以该参数值作图绘制词云，这个参数不为空时，width和height会被忽略
                   max_font_size=100,  # 显示字体的最大值
                   font_path="C:/Windows/Fonts/STFANGSO.ttf",  # 解决显示口字型乱码问题，可进入C:/Windows/Fonts/目录更换字体
                   random_state=42,  # 为每个词返回一个PIL颜色
                   # width=1000,  # 图片的宽
                   # height=860  #图片的长
                   )
    for i in data.quteo:
        words = words + i
    word_generator = jieba.cut(words, cut_all=False)
    for word in word_generator:
        word_list.append(word)
    text = ' '.join(word_list)
    wc.generate(text)
    # 基于彩色图像生成相应彩色
    image_colors = ImageColorGenerator(girl_image)
    # 显示图片
    plt.imshow(wc)
    # 关闭坐标轴
    plt.axis('off')
    # 绘制词云
    plt.figure()
    plt.imshow(wc.recolor(color_func=image_colors))
    plt.axis('off')
    wc.to_file('19th.png')
    plt.show()
def data_analyse_button5(data, country_name, country_count):
    score = []
    for i in range(len(country_count)):
        score.append(0)
    score_average = []
    for i in range(len(country_count)):
        for j in data.index:
            if country_name[i] == data.loc[j, 'country']:
                score[i] = score[i] + data.loc[j, 'score']
            else:
                continue
        score_average.append(round(score[i]/country_count[i],2))
    return country_name, score_average
def data_analyse_button5_figure(country_name, score_average):
    fig = plt.figure(1)
    ax1 = plt.subplot(111)
    data = score_average
    width = 0.5
    x_bar = np.arange(len(country_name))
    rect = ax1.bar(left=x_bar, height=data, width=width, color="lightblue")
    for rec in rect:
        x = rec.get_x()
        height = rec.get_height()
        ax1.text(x + 0.1, 1.02 * height, str(height))
    ax1.set_xticks(x_bar)
    ax1.set_xticklabels(country_name)
    ax1.set_ylabel("num")
    ax1.set_title("TOP 250 movie country score distribution")
    ax1.grid(True)
    ax1.set_ylim(0, 10)
    fig.get_tight_layout()
    plt.show()
#以上是数据分析模块，以下是数据查询模块
def data_inquire_entry1(event=None):
    filename = path_result + 'result.csv'
    data = pd.read_csv(filename, encoding='gbk')
    temp = e1.get()
    temp = int(temp)
    inquire = data[data['year'] == temp]
    print(inquire)
    head = list(inquire)
    data_inquire = [head]
    content = inquire.values.tolist()
    for i in range(len(content)):
        data_inquire.append(content[i])
    data_inquire = AsciiTable(data_inquire)
    text.insert(INSERT, data_inquire.table)
def data_inquire_entry2(event=None):
    filename = path_result + 'result.csv'
    data = pd.read_csv(filename, encoding='gbk')
    temp = e2.get()
    inquire = data[data['country'] == temp]
    print(inquire)
    head = list(inquire)
    data_inquire = [head]
    content = inquire.values.tolist()
    for i in range(len(content)):
        data_inquire.append(content[i])
    data_inquire = AsciiTable(data_inquire)
    text.insert(INSERT, data_inquire.table)
def data_inquire_entry3(event=None):
    filename = path_result + 'result.csv'
    data = pd.read_csv(filename, encoding='gbk')
    temp = e3.get()
    temp = float(temp)
    inquire = data[data['score'] == temp]
    print(inquire)
    head = list(inquire)
    data_inquire = [head]
    content = inquire.values.tolist()
    for i in range(len(content)):
        data_inquire.append(content[i])
    data_inquire = AsciiTable(data_inquire)
    text.insert(INSERT, data_inquire.table)
def data_inquire_entry4(event=None):
    filename = path_result + 'result.csv'
    data = pd.read_csv(filename, encoding='gbk')
    temp = e4.get()
    inquire = data[data['name'] == temp]
    print(inquire)
    head = list(inquire)
    data_inquire = [head]
    content = inquire.values.tolist()
    for i in range(len(content)):
        data_inquire.append(content[i])
    data_inquire = AsciiTable(data_inquire)
    text.insert(INSERT, data_inquire.table)

#download_page(URL)
#get_all_information()
ch.set_ch()
data= get_data()
country_name, country_count = data_analyse_button1(data)
movie_year, movie_count = data_analyse_button2(data)
movie_score, movie_score_count = data_analyse_button3(data)
country_name, score_average = data_analyse_button5(data, country_name, country_count)

root = Tk()
root.title("豆瓣分析系统")
root.minsize(800, 700)
# 数据分析模块界面设计
label_analyse = Label(root, text='数据分析', background='red').grid(row=1,column=0,padx=50)
anaylse_button1 = Button(root, text='国家分布',command = lambda: data_analyse_button1_figure(country_name,country_count)).grid(row=2,column=1)
anaylse_button2 = Button(root, text='年代分布',command=  lambda: data_analyse_button2_figure(movie_year, movie_count)).grid(row=3,column=1)
anaylse_button3 = Button(root, text='评分分布',command=  lambda: data_analyse_button3_figure(movie_score, movie_score_count)).grid(row=4,column=1)
anaylse_button4 = Button(root, text='词云可视化',command=  lambda: work_cloud_visualization(data)).grid(row=5,column=1)
anaylse_button5 = Button(root, text='平均评分国家对比', command=lambda: data_analyse_button5_figure(country_name, score_average)).grid(row=6,column=1)
# 数据查询模块
label_inquire = Label(root, text='数据查询', background='yellow').grid(row=1,column=2,padx=50)
inquire_label1 = Label(root, text='按年代查询').grid(row=2,column=3)
inquire_label2 = Label(root, text='按国家查询').grid(row=3,column=3)
inquire_label3 = Label(root, text='按评分查询').grid(row=4,column=3)
inquire_label4 = Label(root, text='按片名查询').grid(row=5,column=3)
#数据查询输入模块
e1 = StringVar()
en1 = Entry(root, validate='key', textvariable=e1)
en1.grid(row=2, column=4)
en1.bind('<Return>', data_inquire_entry1)
e2 = StringVar()
en2 = Entry(root, validate='key', textvariable=e2)
en2.grid(row=3, column=4)
en2.bind('<Return>', data_inquire_entry2)
e3 = StringVar()
en3 = Entry(root, validate='key', textvariable=e3)
en3.grid(row=4, column=4)
en3.bind('<Return>', data_inquire_entry3)
e4 = StringVar()
en4 = Entry(root, validate='key', textvariable=e4)
en4.grid(row=5, column=4)
en4.bind('<Return>', data_inquire_entry4)
# 数据显示模块
text = scrolledtext.ScrolledText(root, width=120, height=20)
text.grid(row=6, columnspan=5,padx=20,pady=10)
root.mainloop()



