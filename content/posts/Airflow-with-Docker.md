---
title: "Airflow With Docker-Part 2"
date: 2019-08-25T14:25:56+08:00
draft: false
description:
categories:
 -
featured_image:
author: "Leo"
---

![](https://cdn-images-1.medium.com/max/3200/1*bF-mCJh4TCh9MvY1M97DRQ.jpeg)

## Airflow with Docker 容器部署 — part 2

在前一篇文章從頭到尾的安裝了Airflow, 這次基於未來考量的擴展性跟管控性，決定也試試使用Docker 安裝Airflow。網路上已有現成的Docker包，在這邊我們只做些微的調整。(安裝環境依然是在Ubuntu上)

This is the setting I already changed and update to the git that you can just download to use it.
>  git clone [https://github.com/cchangleo/docker-airflow.git](https://github.com/cchangleo/docker-airflow.git)

Here are few steps we will follow:

 1. Installing Docker

 2. Install Docker-Compose

 3. Getting Docker-Airflow from git

 4. Dockerfile setting

 5. docker-compose setting

 6. Start all service

 7. Testing
<br>
<iframe src="https://giphy.com/embed/gVoBC0SuaHStq" width="470" height="480" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/republican-bill-oreilly-ron-paul-gVoBC0SuaHStq">via GIPHY</a></p>
<br>

If you don’t know what is Docker before, just like me, here are few websites can let you know more about Docker.

[Docker 基本教學 - 從無到有 Docker-Beginners-Guide](https://github.com/twtrubiks/docker-tutorial)

[Docker — 从入门到实践](https://yeasy.gitbooks.io/docker_practice/content/)





OK, let’s install Docker
<br>
<iframe src="https://giphy.com/embed/xT5LMyWnCCvfKyD3PO" width="480" height="366" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/season-3-the-simpsons-3x18-xT5LMyWnCCvfKyD3PO">via GIPHY</a></p>
<br>

**安裝Docker CE**

    sudo apt-get update
    
    sudo apt-get install docker-ce

    *#將當前用户加入docker組*
    sudo gpasswd -a ${USER} docker  
    sudo systemcl enable docker 
    sudo systemcl restart docker  
    docker ps

**安裝Docker-Compose**

    sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

    *#給予執行權限*

    sudo chmod +x /usr/local/bin/docker-compose

    sudo pip install docker-compose  

    *#檢查版本*
    docker-compose version

So far so good, now we need to pull the image from the Docker repository.

    docker pull puckel/docker-airflow

    #or get from git

    git clone https://github.com/puckel/docker-airflow.git

由於希望Docker airflow 的架構可以像這樣擴展，在這邊需要先修改下原本的dockerfile 與剛剛我們clone下來的docker-compose-CeleryExecutor.yml

![](https://cdn-images-1.medium.com/max/2000/0*LkVLDdgd3nb1lE6P.png)

![](https://cdn-images-1.medium.com/max/2000/1*3p92WDmnf94QINq4ZX_bjw.png)

### Docker部署Airflow CeleryExecutor模式

    cd docker-airflow

**修改Dockerfile**
*主要修改了redis安裝版本與幾個會用到的pip包，還有airflow 需要的資源*
<script src="https://gist.github.com/cchangleo/ecc13e04424c75ac3028ada5fc1f421b.js"></script>


**修改 docker-compose-CeleryExecutor.yml**

這邊主要把airflow run job時會要用到的目錄都先掛載上去，可以在worker/scheduler 那邊添加，如果DB資料需要留存 也建議把command拿掉 才不會當刪除Docker時, docke裡db資料也不見。另外在這邊將使用postgres + redis + celery +flower。

默认的compose是抓原本版本的鏡像包，這邊記得都改成latest: 這樣才會抓取我們剛剛修改過後的Docker file

![](https://cdn-images-1.medium.com/max/2000/1*qwsrQ4NHNy6C46rdzERyHQ.png)



**Airflow config 設定**

由於我們是要佈署Celery 在airflow, 這邊也需要修改airflow.config

    cd config
    vi airflow.cfg

![executor 改為CeleryExecutor](https://cdn-images-1.medium.com/max/2000/1*4yQwYJ4ka9s1a6PfaLWCKQ.png)


Now looks great, we can run dockerfile first, after need to run compose file.

    docker build --rm -t puckel/docker-airflow .

    docker-compose -f docker-compose-CeleryExecutor.yml up -d

Test it, now you should see like this:

![](https://cdn-images-1.medium.com/max/2630/1*FGMV3URO8mHNJVb2qF5p5w.png)

Here are some scripts let you control your docker easily. And if you need to delete the docker and change some setting, you need to stop docker compose then clean docker compose then clean the docker images.

    # 啟動
    docker-compose -f docker-compose-CeleryExecutor.yml up -d

    # 停止
    docker-compose -f ./docker-compose-CeleryExecutor.yml stop

    # Celery and worker 擴展
    docker-compose -f docker-compose-CeleryExecutor.yml scale worker=3 docker-compose -f docker-compose-CeleryExecutor.yml scale scheduler=3

使用airflow 命令行的範例

    docker-compose -f docker-compose-CeleryExecutor.yml run --rm webserver airflow list_dags

Now we can change the connection setting on airflow browser and check if celery and flower are working. (default setting for flower is localhost:5555)

![localhost:8080](https://cdn-images-1.medium.com/max/2236/1*KbTBRPXn21XUJKsocetrBw.png)

![localhost:5555](https://cdn-images-1.medium.com/max/5086/1*fwOQX74_t3PzCp2p9BdjcQ.png)

Finally 大功告成，之後我們只需要把要跑得dag放在設定下的目錄就可以跑了~~ 在這邊我用的DAG範例來源如下:
[**一段 Airflow 與資料工程的故事：談如何用 Python 追漫畫連載**
*經過前面的幾個章節，我們已經對 comic_app_v2 DAG 的測試及開發下了不少功夫： 利用 airflow test 指令分別測試每個 Airflow 工作執行如預期 python dags/comic_app_v2.py 確保…*leemeng.tw](https://leemeng.tw/a-story-about-airflow-and-data-engineering-using-how-to-use-python-to-catch-up-with-latest-comics-as-an-example.html)

(已修改上述文中的爬蟲方式及某些設定)

![](https://cdn-images-1.medium.com/max/2536/1*RCUUKwnbdd9_chuZr5pNkQ.png)

![成功跑在兩台workers](https://cdn-images-1.medium.com/max/5060/1*D6VXm8EyEJ97yCIcYeNWcw.png)

<iframe src="https://giphy.com/embed/3o7TKnvDNYADdLYZIQ" width="480" height="480" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/art-design-illustration-3o7TKnvDNYADdLYZIQ">via GIPHY</a></p>

Refreence:

[https://github.com/puckel/docker-airflow](https://github.com/puckel/docker-airflow)

[https://medium.com/bluecore-engineering/were-all-using-airflow-wrong-and-how-to-fix-it-a56f14cb0753](https://medium.com/bluecore-engineering/were-all-using-airflow-wrong-and-how-to-fix-it-a56f14cb0753)

[http://site.clairvoyantsoft.com/setting-apache-airflow-cluster/](http://site.clairvoyantsoft.com/setting-apache-airflow-cluster/)

[https://github.com/apachecn/airflow-doc-zh/blob/master/zh/26.md](https://github.com/apachecn/airflow-doc-zh/blob/master/zh/26.md)
