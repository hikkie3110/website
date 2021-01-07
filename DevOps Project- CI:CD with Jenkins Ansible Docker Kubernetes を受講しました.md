DevOpsやCI/CDに対する理解を深めるため
ハンズオントレーニングのある講座を受講しました。

Udemyの```DevOps Project: CI/CD with Jenkins Ansible Docker Kubernetes```
です。

[講座のURL](https://www.udemy.com/course/valaxy-devops/)
※セールを狙って購入した方が良いかもしれません。



### 講座内容
```
Do you want to build a CI/CD pipeline using various DevOps tools? then you are at the right place.

Here you can see a CI/CD pipeline by using tools using Git, Jenkins, Ansible, Docker, and Kubernetes. This gives some light on how the IT industry uses DevOps.
```

〜〜〜となります。

### 対象者
```
#### Who is this course for?
* Anyone who wants to build CI/CD pipeline tools on Various DevOps tools
* Anyone who wants to Enhance their skills in DevOps domain
```

```
* Who wants to know how DevOps does work, who completed DevOps training and want to do a project hands-on project
```
と記載があります。

# 紹介

セクションは6つに分かれています。

{{{表で示す？？？}}}


## Section1:Introduction
CI/CDとは何か？
この講座がカバーする範囲など概要を説明しています。

## Section2:CI/CD pipeline using Git,Jenkins and Maven
本講座では、CI/CDパイプラインを構築するためにGit,Jenkins,Mavenを利用します。
本セクションではこれらのインストールを説明しています。

環境変数やjenkinsでのconfigurationの設定方法を学べます。

環境の前提としては
* awsのEC2インスタンスを作成し、これらをインストールする
　（後続に出てくる、tomcat,dockerなども同様です）
があるため注意してください。


## Section3:Integrating Tomcat server in CI/CD pipeline
* tomcatサーバにアプリ(warファイル）をCI/CDでデプロイするシナリオをハンズオンします。
* git管理しているソースコードにmergeする事で、自動的にCI/CDが回ることを体験します。

※疑問に思っている事
VM上のtomcatにデプロイするのに、なぜ deploy to containerのpluginを入れたのか。

### 仕組み上やる事
* pollSCM
    * Build triggersにPoll SCMを設定する
    * 設定の仕方はcronみたいに　```* * * * *```とか（左記だと1分おきのポーリング）
    * 
* どういう設定をしないと行けないかがわかる。
    * jenkins上でtomcatのcredential情報を入力しないと行けない事(roleはmanagement-script)
    * Goal: "clean install package"
    * Post-build actionの設定(deploy war/ear to container)
    * 
### 使用したplugin
* GitHub
* Maven Invoker
* Maven Integration
* Deploy to Container




## Section4::Integrating Docker in CI/CD pipeline
今度はDockerを利用したCI/CDです。

### やった事
* Dockerホストの構築（EC2インスタンスを立てて、dockerのインストール）
* latestタグ、特定のバージョンのタグでtomcatのdockerイメージをデプロイし動作確認をする。
* Publish Over SSHにdockerホストのユーザなどSSHに必要な設定を行う。
* アーティファクトをdockerhostへコピーする。
    * remove prefixの機能
    * Dockerfileの作成
        ```
        FROM tomcat:latest
        MAINTAINER AR Shankar
        COPY ./webapp.war /usr/local/tomcat/webapps
        ```
* 直接コンテナをデプロイする。
    * Post-build actionsの```exec command``に以下をいれる
        ```
        cd /home/dockeradmin;
        docker build -t devops-image　.
        docker run -d --name devops-container -p 8080:8080 devops-image;
        ```
### コマンド
* ```docker images```
* ```docker run```
* ```docker ps```、```docker ps -a```
* ```docker rm```
* ```docker delete```
* ```docker run -d --name tomcat -p 8080:8080 tomcat```
* ```docker exec -it tomcat-container /bin/bash```
* ```docker build```



### 使用したplugin
* Publish Over SSH




## seciton5:ansible

## やった事
* ansibleのインストール
* Jenkinsの設定でansibleをセッティング
* ansible-serverにDockerファイルとplaybook.ymlを作成
* jenkinsにansible-playbookを実行コマンドとして設定しジョブ実行
* 上記の状態からPollSCMを設定した上でwebapp.warを```git push```し更新を実行する（結果はUNSTABLEになります。すでに動いているコンテナを停止する必要がある）
* playbookを編集する（コンテナを停止、コンテナを削除、dockerイメージを削除）
    * この停止の時、ignore_errors: yesをタスクに入れないとfailする。これはなぜか理解はしておきたい。。
* DockerHubの登録
* DockerHubにbuildしたイメージをpushするplaybookを作成する（create-simple-devops-image.yml）
* DockerHubからpullしてデプロイするようにplaybookを修正する。（create-simple-devops-project.yml）
    ```
    ---
- hosts: all
  become: true

  tasks:
  - name: stop current running container
    command: docker stop simple-devops-container
    ignore_errors: yes 

  - name: remove stopped container
    command: docker rm simple-devops-container
    ignore_errors: yes 

  - name: remove docker image
    command: docker rmi hikkie3110/simple-devops-image:latest
    ignore_errors: yes

#  - name: build docker image using war file
#    command: docker build -t simple-devops-image .
#    args:
#      chdir: /opt/docker

  - name: pull docker image from docker hub
    command: docker pull hikkie3110/simple-devops-image:latest

  - name: create container using simple-devops-image
    command: docker run -d --name simple-devops-container -p 8080:8080 hikkie3110/simple-devops-image:latest
    ```
* ```--limit```タグを利用して限定的にplaybookを実行する。



## 有用なコマンド
* ```ssh-copy-id ansadmin@<target-server>```こんなコマンドあったのか・・・。
* remote directoryセットする時は、//opt//dockerとする。　これはなぜかよくわからん。
* ```docker tag simple-devops-image hikkie3110/simple-devops-image```
* ```docker push```
* ```docker build```


## 困った事
* 手順通りインストールしたが、/etc/ansibleディレクトリが作成されなかった。。なぜ？？？
* docker loginが切れていると、playbookでdocker pushする所がうまく動かない（当たり前） →これ、どうやって回避するの？？？



## seciton6:Integrating Kubernetes in CI/CD pipeline

### やる事
* kopsを利用してk8s clusterをaws上に構築する。
* 上記のk8s上で```deployment```や```servie```リソースをデプロイする。
* jenkinsからansibleで```deployment```や```service```を```kubectl apply```するjobを記載する
    →　webapp.warを書き換えても稼働しているアプリケーションは新しくならない！！(No.45)


### ハマった事
* kopsで構築したマスターノードにsshできない。
    * 原因は不明
    * 鍵のパーミッション見直し、実行ユーザubuntuにした上でもう一度クラスター作り直した。
* Ansibleから```kubectl apply -f```をplaybookで流そうとすると
    ```
    "The connection to the server localhost:8080 was refused - did you specify the right host or port?", "stderr_lines": ["The connection to the server localhost:8080 was refused - did you specify the right host or port?"], "stdout": "", "stdout_lines": []}
    ```
    と表示されてしまい実行できない。
    （playbook出なく、当該ノードで直接実行すると普通にうまくいく）

    ```わかった
    playbookのbecomeはrootの事。そのあとのuserは何？？？？ubuntuを指定してもダメだったやんか
    ```

    ```yml NGパターン
    become: yes
    user: ubuntu
    ```

    ```yml OKパターン
    become: yes
    user: root
    ```
    * 確かにrootではmaster nodeでもkubectlを実行できない（同じメッセージ```localhost:8080 connection refused```）が発生する。
    * これはrootのホームディレクトリ配下に.kube/configがなかったため
    * 多分kopsコマンドをubuntuで実行してしまったために発生した問題だと考えられる。
    * もしかしたらansibleでうまくubuntuユーザで実行するように指定できれば良かったのかもしれない？？
        ```yml
        - name: Create pods using deployment 
          hosts: kubernetes 
          become: true
          user: root   # ←この意味って何！？
        ```


* deploymentのマニフェストファイルに```imagepullpolicy:always```がある。この意味は
* dockerイメージ更新された場合にデプロイしたい場合、
    ```kubectl rollout restart deployment.v1.apps/valaxy-deployment```を利用する。
    出ないと、マニフェストファイルが変更された訳ではないので反映されないのだ。


### 参考リポジトリ
* [Simple-DevOps-Project](https://github.com/yankils/Simple-DevOps-Project)
* [hello-world](https://github.com/yankils/hello-world)


---
* [フローと使うツールのマッピングの例：きた山さん！](https://www.slideshare.net/ShingoKitayama/ansible-73396071)
* [NTTデータのIaC](https://www.slideshare.net/nttdata-tech/infrastructure-as-code-2019-nttdata-sasaki-takai)
* [CI/CD パイプライン（例が載っている）：きたやまさん！](https://www.slideshare.net/ShingoKitayama/openstackdaystokyo4b13-cicd)
* 
