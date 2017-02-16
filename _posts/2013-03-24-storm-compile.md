---
layout: post
status: publish
published: true
title: "storm 0.9.0-wip16: compile "
author: Dillon Peng
date: Thu Feb 16 11:17:10 CST 2017
comments: []
---

### 从源代码0.9.0-wip16编译和运行storm

 使用git clone 版本 0.9.0-wip16的storm
```sh
   [abelard@abelard storm-source3]$ git clone https://github.com/nathanmarz/storm.git
   [abelard@abelard storm-source3] cd storm
```

### 执行lein compile

 - 第一个错误 
```sh
    Exception in thread "main" java.lang.ClassNotFoundException: backtype.storm.LocalDRPC, compiling:(testing.clj:1)
```
	
 - 解答： 将storm.trident.testing ns
```clojure
	(ns storm.trident.testing
	(:import [storm.trident.testing FeederBatchSpout FeederCommitterBatchSpout MemoryMapState MemoryMapState$Factory TuplifyArgs])
	(:import [backtype.storm LocalDRPC])
	(:import [backtype.storm.tuple Fields])
	(:import [backtype.storm.generated KillOptions])
	(:require [backtype.storm [testing :as t]])
	(:use [backtype.storm util])
	)
```
 - 改为：
```clojure
    (ns storm.trident.testing
	(:require [backtype.storm.LocalDRPC])
	(:import [storm.trident.testing FeederBatchSpout FeederCommitterBatchSpout MemoryMapState MemoryMapState$Factory TuplifyArgs])
	(:import [backtype.storm LocalDRPC])
	(:import [backtype.storm.tuple Fields])
	(:import [backtype.storm.generated KillOptions])
	(:require [backtype.storm [testing :as t]])
	(:use [backtype.storm util])
	)
```
	
### lein install

将storm打包后的jar文件安装到用户主目录(ex.,/home/abelard/)的.m2（mvn的
目录）下。

运行storm-starter的local mode
```sh
	lein run -m storm.starter.WordCountTopology
```
	
### 运行storm-starter的distributed mode

```sh
	cd storm-starter
	lein clean;lein deps;lein compile;lein uberjar
	storm jar target/storm-starter-0.0.1-SNAPSHOT-standalone.jar	storm.starter.ExclamationTopology exclamation-topology 
```
	
	
### 可能的错误

提示storm包重复，是因为project.clj文件写的不对，所以, *uberjar*将storm包，也打入到storm-starter-0.0.1-SNAPSHOT-standalone.jar中了。
