# Leiningen 文档
### [ClojureCLR](https://github.com/clojure/clojure-clr/wiki/)

---

```
dotnet tool install --global Clojure.Main
Clojure.Main
```

### 安装
```
1 安装leiningen
leiningen-win-installer.exe
2 安装clojure
https://github.com/technomancy/leiningen/releases
后缀改为jar，放到 ~/.lein/self-installs
3 lein repl 自动安装
```

### 使用
```
lein help
lein help task

lein new my-stuff

鼓励多端命名空间my-stuff.core
JVM加载含有-的文件会遇到问题，会用_替代

checkouts目录 略

lein repl
=> (require 'my-stuff.core)
=> (my-stuff.core/-main)
=> (require -[clj-http.client :as http])
=> (def response (http/get "http://leiningen.org"))
=> (keys response)

=> (doc reduce)
=> (user/clojuredocs pprint)
=> (source my-stuff.core/-main)

运行-main
设置project.clj 里的 :main关键字 可以省略-m
lein run -m my-stuff.core

对于长时间运行的lein run进程，可能希望利用高阶的trampoline任务来节省内存
允许leiningen的JVM进程在启动项目的JVM之前退出
lein help trampoline
lein trampoline run -m my-stuff.server 5000
```

```
测试
lein test
lein test my.text.stuff
```

```
可以分发给终端用户的应用
构建一个uberjar
lein new app myapp
必须设置命名空间，作为project.clj里的 :main

#project.clj
(defproject my-stuff "0.1.0-SNAPSHOT"
     :description "FIXME: write description"
     :url "http://example.com/FIXME"
     :license {:name "Eclipse Public License"
               :url "http://www.eclipse.org/legal/epl-v10.html"}
     :dependencies [[org.clojure/clojure "1.3.0"]
                    [org.apache.lucene/lucene-core "3.0.2"]
                    [clj-http "0.4.1"]]
     :profiles {:dev {:dependencies midje "1.3.1"}}
     :test-selectors {:default (complement :integration)
                     :integration :integration
                     :all (fn [_] true)}
     :main my.stuff)

#src/my/stuff.clj
   (ns my.stuff
     (:gen-class))
   (defn -main [& args]
     (println "Welcome to my project! These are your args:" args))

lein uberjar
创建一个单一jar文件，可以用java运行
java -jar my-stuff-0.1.0-standalone.jar Hello world.
输出：Welcome to my project! These are your args: (Hello world.)

也可以用lein run
```

```
发行类库 略
```

---