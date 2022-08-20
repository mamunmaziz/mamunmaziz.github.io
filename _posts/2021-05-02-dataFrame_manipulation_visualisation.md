---
title: "Python Code: DataFrame manipulation and visualisation"
Date: 2021-05-02
tags: [Data Science Projects with Python]
header:
  image: "/images/CF.jpg"
excerpt: "Data Science, Data Analysis, R"
mathjax: "true"
---

## Introduction

A simple python code was written for find out the cryptocurrenbcy real time price from webpage .


```python
import re
import urllib.request
url = "https://digitalcoinprice.com/coins/"
coin = input("Enter cryto coin full name: ")
url = url + coin
data= urllib.request.urlopen(url).read()
data1 = data.decode("utf-8")

m = re.search('<span class="price" data-usd="', data1)
start = m.start()
end = start + 50
newString = data1[start:end]

m = re.search('data-usd="',newString)
start = m.end()
newString1 = newString[start:]


m = re.search('data-btc="',newString1)
start = 0
end = m.end()-11
final = newString1[0:end]
print("The value of " + coin.upper() + " is USD $" + final)
```

