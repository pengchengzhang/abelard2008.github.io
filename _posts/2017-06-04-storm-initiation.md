---
published: true
title: "storm在Local Storm Cluster模式下的初始化过程"
author: Dillon Peng
date: Thu Feb 16 11:17:10 CST 2017
comments: []
---
从源代码分析storm的初始化流程

### 概括
对于local storm cluster，由用户调用testing/mk-local-storm-cluster来构建运行环境，因此对local storm cluster的初始化分析，从这里开始。
（当构建了运行环境以后，通过StormSubmitter/submitTopology用户定义的topology到storm集群）
```clojure
testing/mk-local-storm-cluster
```
返回*cluster-map*，在*cluster-map*里包含了
```clojure
 nimbus nimbus
 port-counter
 daemon-conf
 supervisors 
 state (mk-distributed-cluster-state daemon-conf)
 storm-cluster-state (mk-storm-cluster-state daemon-conf)
 tmp-dirs (atom [nimbus-tmp zk-tmp])
 zookeeper zk-handle
 shared-context context
```

### storm中的宏

```clojure
### util/fast-list-iter 
	(defmacro fast-list-iter [pairs & body]
		(let [pairs (partition 2 pairs)
			lists (map second pairs)
			elems (map first pairs)
			iters (map (fn [_] (gensym)) lists)
			bindings (->> (map (fn [i l] [i `(get-iterator ~l)]) iters lists) (apply concat))
			tests (map (fn [i] `(iter-has-next? ~i)) iters)
			assignments (->> (map (fn [e i] [e `(iter-next ~i)]) elems iters) (apply concat))]
			`(let [~@bindings]
				(while (and ~@tests)
					(let [~@assignments]
						~@body
					)))))
```
- 目的 
对列表中的元素进行迭代，由用户提供元素操作。以worker/mk-transfer-fn中
扩展该宏为例：
#+BEGIN_SRC java
	(fast-list-iter [[task tuple :as pair] tuple-batch]
        (if (local-tasks task)
            (.add local pair)
				(.add remote pair)
			))
#+END_SRC
使用macroexpand的结果如下：
#+BEGIN_SRC java
	(let* [G__1345 (backtype.storm.util/get-iterator tuple-batch)] 
		(clojure.core/while (clojure.core/and (backtype.storm.util/iter-has-next? G__1345)) 
			(clojure.core/let [[task tuple :as pair]
		(backtype.storm.util/iter-next G__1345)] 
			(if (local-tasks task) (.add local pair) (.add remote pair)))))
#+END_SRC
为了更详细了解扩展细节，将所有中间变量打印出来：
#+BEGIN_SRC java
	pairs:  (([task tuple :as pair] tuple-batch))
	lists:  (tuple-batch)
	elems:  ([task tuple :as pair])
	iters:  (G__1345)
	bindings:  (G__1345 (backtype.storm.util/get-iterator tuple-batch))
	tests:  ((backtype.storm.util/iter-has-next? G__1345))
	assignments:  ([task tuple :as pair] (backtype.storm.util/iter-next G__1345))
#+END_SRC
其中：

 - G__1345 通过 /gensym/ 函数生成。
 - tuple-batch 为 *ArrayList* 类型，因此使用 *get-iterator* 生成 *Iterator* ，然后使用 *util/iter-next* , 与java中直接使用 *for( elem: ArrayList)* 不同。举例：
  #+BEGIN_SRC java
	  user> (def al (ArrayList. [[1 {1 5241700977694457705}], [3 {2 963084339025074184}], [2 {3 -6622099668460348678}]]))
	  #'user/al
	  user> (let [al (get-iterator al)] (while (and (iter-has-next? al))
				    (println (iter-next al))))
					      
     [1 {1 5241700977694457705}]
	 [3 {2 963084339025074184}]
	 [2 {3 -6622099668460348678}]
	 nil
	 user>   
  #+END_SRC

* spout的emit
用户spout类，与emit相关的代码，以 *storm.starter.spout.RandomSentenceSpout* 为例：
#+BEGIN_SRC java
	public class RandomSentenceSpout extends BaseRichSpout {
		SpoutOutputCollector _collector;
			    

    @Override
    public void open(Map conf, TopologyContext context, SpoutOutputCollector collector) {
        _collector = collector;
    }

    @Override
    public void nextTuple() {
       ....
        _collector.emit(new Values(sentence));
    }        
#+END_SRC
在每个supervisor中，由worker进程，executor线程和task组成，对spout来说，会创建spout线程，由executor/mk-threads调用async-loop完成。

open()函数将在executor/mk-threads所产生的线程中被调用一次，完成一些初始化的工作，(a) 创建SpoutOutputCollector实例，open()函数完成之后，上面的__collector，就可以使用了。

#+BEGIN_SRC java
    (defmethod mk-threads :spout [executor-data task-datas]
	...
	(.open spout-obj
		storm-conf
			(:user-context task-data)
				(SpoutOutputCollector.
					(reify ISpoutOutputCollector
						(^List emit [this ^String stream-id ^List tuple ^Object message-id]
							(send-spout-msg stream-id tuple message-id nil)
				  
#+END_SRC
(b) 在调用open()和完成SpoutOutputCollector实例创建之前，实现了ISpoutOutputCollector接口，这里只给出通常需要的emit函数，它调用send-spout-msg来完成。

nextTuple()一旦被调用，spout消息就可以通过__collect.emit()发送出去了，而nextTuple()在哪里被调用呢：
  #+BEGIN_SRC java
     (defmethod mk-threads :spout [executor-data task-datas]
	   ...
	   (async-loop
	     (fn []
		    ...
			(fn []
				(fast-list-iter [^ISpout spout spouts] (.nextTuple
	   spout)))
	   ...
  #+END_SRC
通过对async-loop的函数的分析，可以知道，mk-threads中，传递给 async-loop的嵌套的(fn ... (fn ...)), 在内层的fn将会在每次线程被调度中执行，而内层的fn之前到外层fn定义之间的代码，只执行一次。  
  
  
