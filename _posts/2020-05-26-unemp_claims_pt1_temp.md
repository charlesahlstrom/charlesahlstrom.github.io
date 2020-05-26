---
title:  "Digging into Unemployment Claims Data & Reasons"
date:   2020-05-26
tags: [Macroeconomics, Covid, PDF scraping, R]

header:
  image: "placeholder.png"
  caption: "placeholder"

excerpt: "State variations and explanations in unemployment claims"
---

Introduction

# A Simple Example


```R
import requests
from bs4 import BeautifulSoup

BASE_URL = "https://news.ycombinator.com/"
STORY_LINKS = []

for i in range(10):
    resp = requests.get(f"{BASE_URL}news?p={i}")
    soup = BeautifulSoup(resp.content, "html.parser")
    stories = soup.find_all("a", attrs={"class":"storylink"})
    links = [x["href"] for x in stories if "http" in x["href"]]
    STORY_LINKS += links
    time.sleep(0.25)

print(len(STORY_LINKS))

for url in STORY_LINKS[:3]:
    print(url)
```

    

```R
import time

def download_url(url):
    t0 = time.time()
    resp = requests.get(url)
    t1 = time.time()
    print(f"Request took {round(t1-t0, 2)} seconds.")
    
    title = "".join(x for x in url if x.isalpha()) + "html"
    
    with open(title, "wb") as fh:
        fh.write(resp.content)

download_url("https://beckernick.github.io/what-blogging-taught-me-about-software/")
```






# Conclusion



<sub><sub>*Note: This is a placeholder; see [Timmy's page](google.com).*</sub></sub>
