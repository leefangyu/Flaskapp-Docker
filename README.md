# 簡介
使用Docker部署一個Flask App with Nginx and uWSGI
到AWS EC2 Instance上
使外部能透過public ip連線到該雲端server訪問Flask App

# Flask Nginx uWSGI  架構
![](https://hackmd.io/_uploads/BJZQ-ysla.png)

* Nginx：非同步Web Server，可做反向代理、負載平衡器和 HTTP 快取等
* WSGI (Web Server Gateway Interface)：負責溝通代理伺服器跟Flask，可以當成一個協議或規範
    uWSGI是WSGI的一種協議


# Build
先在開發端安裝Docker以建立應用程式與環境的Docker Image
此例使用DockerHub上
將Nginx與uWSGI包裝在一起的Docker Image做為Base Image
### Install Docker

Add Docker's official GPG key:
```shell=
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```


Add the repository to Apt sources:
```shell=
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  
  
sudo apt-get update
```

Install:
```shell
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Test:
```shell
sudo docker run hello-world
```
若安裝成功會顯示以下畫面
![](https://hackmd.io/_uploads/H1YRkFYla.png)


### Build Docker Image
建立一簡單Flask App作為範例:
先建立一目錄app
並在目錄中新增 main.py:
```python=
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello World from Flask"

if __name__ == "__main__":
    # Only for debugging while developing
    app.run(host='0.0.0.0', debug=True, port=80)
```

在與app同目錄的地方新增Dockerfile:
```dockerfile
FROM tiangolo/uwsgi-nginx-flask:python3.11
COPY ./app /app
```
FORM 指令為使用的Base Image 
COPY 代表複製檔案到Image裡面

此例的Base Image:
https://hub.docker.com/r/tiangolo/uwsgi-nginx-flask/?source=post_page-----b8bdc60d1dc7--------------------------------

Dockerfile建立完成後檔案結構為:
```
.
├── app
│   └── main.py
└── Dockerfile
```

接下來在與Dockerfile同目錄下執行Build指令:
```shell
sudo docker build -t myimage .
```
完成後即可使用docker images指令查看目前所有Image
確認建立完成
![](https://hackmd.io/_uploads/B1wPd6Yxp.png)

### 上傳至Docker Hub
為了讓部署的主機下載Docker Image
需上傳至Docker Hub提供Docker pull

Image建立完成後
先將Docker Image加上tag:
```shell
sudo docker tag <Image Name>  <DockerHub Id>/<Image Name>
```

登入Docker Hub:
```shell
sudo docker login
```

上傳至Docker Hub:
```shell
sudo docker push <DockerHub Id>/<Image Name>
```

即可到Docker Hub確認上傳完成
![](https://hackmd.io/_uploads/ryS1QgieT.png)



# Deploy
在被部署的主機上安裝完Docker
即可Pull開發端Build好的Docker Image下來
Run成Container即完成部署

### Pull Docker Image
登入Docker Hub到剛剛上傳的Repository公開頁面
即可複製該Image的Pull指令
![](https://hackmd.io/_uploads/Hy-QVejxp.png)

Pull指令格式:
```shell
sudo docker pull <DockerHub Id>/<ImageName>
```

執行sudo docker images確認有pull下來
![](https://hackmd.io/_uploads/ByoF4gie6.png)

### Run Container
將Image Run成Container :
```shell
sudo docker run -d --name mycontainer -p 8000:80 leefunyu/myimage
```
--name 後面為建立的Container名稱
-p 8000:80 
8000為主機對外的port 
80是container的port
leefunyu/myimage為Image名稱

Container run起來後可用指令確認目前執行中的Container:
```
sudo docker ps
```
![](https://hackmd.io/_uploads/BkjF8xola.png)

打開該EC2 Instance public ip加上指定port(該port需加入安全群組)
即可連到Flask App
完成部署
![](https://hackmd.io/_uploads/H1Ihc6txT.png)

若想停止該服務
把Container關掉即可:
```shell
sudo docker stop <containerID>
```

# 過程遇到的問題
不同AWS EC2 Instance若共用安全群組會導致port發生問題無法存取

# Reference
https://medium.com/%E5%B7%A5%E7%A8%8B%E9%9A%A8%E5%AF%AB%E7%AD%86%E8%A8%98/flask-app-%E5%8A%A0%E4%B8%8A-wsgi-%E5%8F%8A-nginx-%E6%9C%8D%E5%8B%99-b8bdc60d1dc7

https://hub.docker.com/r/tiangolo/uwsgi-nginx-flask/?source=post_page-----b8bdc60d1dc7--------------------------------
