# GET_weather_data

 ```
import requests
from bs4 import BeautifulSoup
import pandas as pd
import re
import time
```

#从首页获得各个城市的url
```
url = 'https://www.tianqi5.cn/guoneitianqi/'
result=requests.get(url)
soup = BeautifulSoup(result.text, 'html.parser')
news_links=[]
news_titles=[]
for link in soup.find_all('a'):
    news_links.append(link.get('href'))
    news_titles.append(link.text)
    
result = {
    'news_titles':news_titles,
    'news_links':news_links
}
result = pd.DataFrame(result)
result
result.to_excel('result.xlsx')
```

#根据抓到的城市url，来抓具体的数据
```
temps_2021_7 = []
for i in range (0,len(data_new)):
    lianjie = data_new['news_links'][i]+'202107.html'
    result=requests.get(lianjie)
    soup = BeautifulSoup(result.text, 'html.parser')
    temp = []
    for link in soup.find_all(class_='wdpfd'):
        temp.append(link.text)
    
    temps_2021_7.append(temp)
    print(i)
    print(len(temps_2021_7))
    print('='*50)
    time.sleep(2)
```

#和最初的表格合并,也可以取出单年的值
```
data_new['temps_2021_7']=temps_2021_7

data_2021 = {
    'news_titles':data_new['news_titles'],
    'tem':data_new['temps_2021_7']
}
data_2021 = pd.DataFrame(data_2021)

```
#因为每一个城市的是一个list，所以要拆开list
<img width="880" alt="屏幕快照 2022-07-13 下午4 23 15" src="https://user-images.githubusercontent.com/45219575/178687384-ad900d29-909e-4795-a2e7-83eff346e462.png">

```
def explode1(df, col):
    dftem1 = pd.DataFrame()
    for i in range(df.shape[0]):
        dftem = df.iloc[i].to_frame().T
        if isinstance(df.iloc[i][col], list):
            if len(df.iloc[i][col]) > 0:
                for j in range(len(df.iloc[i][col])):
                    dftem[col + "_"] = df.iloc[i][col][j]
                    dftem1 = pd.concat([dftem1, dftem])
            else:
                dftem1 = pd.concat([dftem1, dftem])
        else:
            dftem1 = pd.concat([dftem1, dftem])
    del dftem1[col]
    dftem1.rename(columns={col + "_": col}, inplace=True)
    return dftem1
```

#取极大值
```
def max_tep(data):
    data = data.dropna()
    data.index = range(len(data))
    max_aa = []
    for i in range(0,len(data)):
        ii=data['tem'][i].split('~')
        max_aa.append(ii[1])
        
    return max_aa
```
#行转列
```
def zhuan(data):
    two_level_index_series = data.set_index(["news_titles", "time"])["max"]
    new_df = two_level_index_series.unstack()
    new_df = new_df.rename_axis(columns=None)
    new_df = new_df.reset_index()
    
    return new_df
```

#复合
```
def jiehe(data):
    kk = explode1(data, "tem")  
    kk_max = max_tep(kk)
    num = len(data['tem'][0])
    time = list(range(1,num+1))
    num2 = len(kk_max)
    
    time2 = time *368
    kk['time']=time2
    kk['max']=kk_max
    
    temp_2022_7 = zhuan(kk)
    
    return temp_2022_7
```

temp_2021_7 = jiehe(data_2021)

<img width="880" alt="屏幕快照 2022-07-13 下午4 26 27" src="https://user-images.githubusercontent.com/45219575/178687340-f67f4f53-1075-498e-a8ed-31bf69cbf655.png">


