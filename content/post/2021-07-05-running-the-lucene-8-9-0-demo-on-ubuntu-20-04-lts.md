---
author: jet
categories:
- Homelab
- Linux
- Programming
- Server
- Tech
date: "2021-07-05T08:59:35Z"
guid: https://jetvillegas.com/?p=20
id: 20
title: Running the Lucene 8.9.0 Demo on Ubuntu 20.04 LTS
url: /2021/07/05/running-the-lucene-8-9-0-demo-on-ubuntu-20-04-lts/
---

I’ve been teaching people about Search Advertising lately, and I use the Lucene project to demonstrate indexing, searching, relevance, ranking, scoring, and boosting. It’s a little tricky to set up with some gotchas with build and runtime setup. I’m posting here so I can tell students where to look.

This tutorial assumes a clean Ubuntu Linux 20.04 LTS system. We’ll put developer dependencies into a `dev` folder on the user’s home directory:

`mkdir ~/dev<br></br>cd ~/dev`

Set up [JAVA](https://www.digitalocean.com/community/tutorials/how-to-install-java-with-apt-on-ubuntu-20-04).

```
sudo apt install default-jre
sudo apt install default-jdk
```

Set up [Ant](https://ant.apache.org/bindownload.cgi)

```
wget https://ftp.jaist.ac.jp/pub/apache//ant/binaries/apache-ant-1.10.10-bin.tar.gz<br></br>tar -zxvf ./apache-ant-1.10.10-bin.tar.gz
```

Edit our shell environment for Java. Add these to your shell’s configuration

```
JAVA_HOME="/usr/lib/jvm/java-11-openjdk-amd64"<br></br>ANT_HOME="/home/jet/dev/apache-ant-1.10.10"
path+=(${ANT_HOME}'/bin')<br></br>export PATH
```

We’ll set up [Lucene](https://lucene.apache.org/core/downloads.html) into a `src` folder in the user’s home directory:

```
mkdir ~/src<br></br>cd ~/src<br></br>wget https://ftp.riken.jp/net/apache/lucene/java/8.9.0/lucene-8.9.0-src.tgz<br></br>tar -zxvf lucene-8.9.0-src.tgz 
```

Build Lucine. Note the specific `ant ivy` setup it requires:

```
cd ~/src/lucene-8.9.0
ant ivy-bootstrap<br></br>ant jar -Dversion=8.9.0
```

Run the Lucene [Demo](https://lucene.apache.org/core/8_9_0/demo/index.html). Note the classpath requires 4 specific paths to include. Mistakes here will cause the `org.apache.lucene.analysis.standard.StandardAnalyzer` not found error:

```
#replace user with your username  
java -cp "/home/user/src/lucene-8.9.0/build/core/:/home/jet/src/lucene-8.9.0/build/queryparser/:/home/user/src/lucene-8.9.0/build/analysis/common/:/home/user /src/lucene-8.9.0/build/demo/" java org.apache.lucene.demo.IndexFiles -docs ~/src/lucene-8.9.0
```

Run the Lucene search engine through its own source files:

```
java -cp "/home/user/src/lucene-8.9.0/build/core/:/home/jet/src/lucene-8.9.0/build/queryparser/:/home/`user`/src/lucene-8.9.0/build/analysis/common/:/home/`user`/src/lucene-8.9.0/build/demo/"``` org.apache.lucene.demo.SearchFiles`
```