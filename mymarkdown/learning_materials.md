

# python

## 虚拟环境

* Python3：python3 -m venv  ./venv
* Python2：使用virtualenv



## 日志(logging)

### 1. 什么情况下使用日志

* 显示控制台输出，用于命令行脚本或程序的普通使用
  * print：
* 用于诊断目的的详细输出，报告程序正常运行期间发生的事件(如状态监视或故障调查)
  * logging.info()
  * logging.debug()
* 发出关于特定运行时事件的警告，如果客户端应用程序对这种情况无能为力，那么logging.warning()，但是仍然应该注意该事件
  * logging.warning()
* 报告关于特定运行时事件的错误
  * 直接报错
* 在不引发异常的情况下报告错误抑制(例如长时间运行的服务器进程中的错误处理程序)
  * logging.error()
  * logging.exception()
  * logging.critical()

### 2. logging介绍

默认级别是WARNING，这意味着只有该级别及以上的事件才会被跟踪，除非日志包被配置为其他级别。

可以以不同的方式处理被跟踪的事件。处理跟踪事件的最简单方法是将它们打印到控制台。另一种常见的方法是将它们写入磁盘文件。

### 3. 记录到文件

#### 基本配置

对basicConfig()的调用应该在对debug()、info()等的调用之前。由于它是一种一次性的简单配置工具，因此只有第一个调用才会真正执行任何操作:后续调用实际上是无操作的。

```json
import logging
logging.basicConfig(filename='example.log', level=logging.DEBUG)
logging.debug('This message should go to the log file')
logging.info('So should this')
logging.warning('And this, too')
logging.error('And non-ASCII stuff, too, like Øresund and Malmö')
```

如果您多次运行上述脚本，则来自连续运行的消息将追加到文件example.log中。如果你希望每次运行都重新开始，而不记得以前运行时的消息，你可以指定filemode参数，通过将上面示例中的调用更改为:

```python
logging.basicConfig(filename='example.log', filemode='w', level=logging.DEBUG)
```

**在生产环境中，只要项目跑起来设置好级别后，后续可直接使用logging打日志**

### 4. 在多个模块中使用日志

```python
# myapp.py
import logging
import mylib

def main():
    logging.basicConfig(filename='myapp.log', level=logging.INFO)
    logging.info('Started')
    mylib.do_something()
    logging.info('Finished')

if __name__ == '__main__':
    main()
# mylib.py
import logging

def do_something():
    logging.info('Doing something')
```



### 5. 更改显示消息的格式

#### 5-1:显示日志等级

```python
import logging

logging.basicConfig(format='%(levelname)s:%(message)s', level=logging.DEBUG)
logging.debug('This message should appear on the console')
logging.info('So should this')
logging.warning('And this, too')
```

#### 5-1:显示时间

```python
import logging
logging.basicConfig(format='%(asctime)s %(message)s')
logging.warning('is when this event was logged.')
# 更改日期显示格式
import logging
logging.basicConfig(format='%(asctime)s %(message)s', datefmt='%m/%d/%Y %I:%M:%S %p')
logging.warning('is when this event was logged.')
```

### 6. 高级

Logging采用了一种模块化方法，并提供了几种类型的组件:

* 日志记录器Loggers：记录器公开应用程序代码直接使用的接口
* 处理程序Handlers：处理程序将日志记录(由记录器创建)发送到适当的目的地
* 过滤器Filters：筛选器为确定输出哪些日志记录提供了更细粒度的工具
* 格式化程序Formatters：格式化程序在最终输出中指定日志记录的布局

日志事件信息在LogRecord实例中的记录器、处理程序、过滤器和格式化程序之间传递。

日志记录是通过调用Logger类(以下称为Logger)实例上的方法来执行的。每个实例都有一个名称，它们在概念上使用点作为分隔符排列在名称空间层次结构中。

日志记录器的名称可以是您想要的任何名称，并指示日志消息起源于应用程序的区域。

在命名记录器时，一个好的约定是使用模块级别的记录器，在每个使用日志记录的模块中，命名如下:

```python
import logging
logger = logging.getLogger(__name__)
```

#### 6-1:logger对象

**logger对象有三重任务。**

* 它们向应用程序代码公开几个方法，以便应用程序可以在运行时记录消息。
* 其次，记录器对象根据严重性(默认的过滤功能)或过滤对象确定要对哪些日志消息进行操作。
* 第三，记录器对象将相关的日志消息传递给所有感兴趣的日志处理程序。

**logger对象上使用最广泛的方法分为两类:配置和消息发送。以下是最常见的配置方法:**

* logger . setlevel()指定记录器将处理的最低严重性日志消息，其中debug是最低的内置严重性级别，critical是最高的内置严重性级别。例如，如果严重级别为INFO，记录器将只处理INFO、警告、错误和关键消息，并将忽略调试消息。

* logger . addhandler()和logger . removehandler()从logger对象中添加和删除处理程序对象。处理程序在处理程序中有更详细的介绍

* logger . addfilter()和logger . removefilter()从logger对象中添加和删除过滤器对象。过滤器在过滤器对象中有更详细的介绍。

**配置了logger对象后，以下方法创建日志消息:**

* sLogger.debug()、Logger.info()、Logger.warning()、Logger.error()和Logger.critical()都创建日志记录，并使用与各自方法名称对应的消息和级别。消息实际上是一个格式字符串，它可能包含标准的字符串替换语法%s、%d、%f，等等。它们的其余参数是与消息中的替换字段对应的对象列表。对于**kwargs，日志记录方法只关注exc_info关键字，并使用它来决定是否记录异常信息。

* Logger.exception()创建类似于Logger.error()的日志消息。不同之处在于Logger.exception()同时转储了一个堆栈跟踪。只能从异常处理程序调用此方法。

* Logger.log()将日志级别作为显式参数。对于记录消息来说，这比使用上面列出的日志级别方便方法要详细一些，但这是在自定义日志级别上记录日志的方法。

**规则**

getLogger()如果提供了指定名称，则返回对logger实例的引用，如果没有提供，则返回root。名称是句点分隔的层次结构。使用相同名称多次调用getLogger()将返回对相同logger对象的引用。层次列表中较低的记录器是列表中较高的记录器的子记录器。例如，给定一个名为foo的记录器，foo.bar, foo.bar.bam都是foo的后代。

记录器有一个有效级别的概念。如果在记录器上没有显式设置级别，则使用其父级别作为其有效级别。如果父类没有显式设置级别，则检查其父类，依此类推——搜索所有祖先类，直到找到显式设置的级别为止。根日志记录器总是有一个显式的级别集(默认为警告)。在决定是否处理事件时，记录器的有效级别用于确定是否将事件传递给记录器的处理程序。

子记录器将消息传播到与其祖先记录器相关联的处理程序。因此，没有必要为应用程序使用的所有记录器定义和配置处理程序。为顶级记录器配置处理程序并根据需要创建子记录器就足够了。(但是，您可以通过将记录器的propagate属性设置为False来关闭传播。)

#### 6-2：handle

处理程序对象负责将适当的日志消息(基于日志消息的严重性)分派到处理程序指定的目的地。日志记录器对象可以使用addHandler()方法向自身添加零个或多个处理程序对象。作为一个示例场景，应用程序可能希望将所有日志消息发送到一个日志文件，将所有错误或更高级别的日志消息发送到stdout，将所有关键消息发送到一个电子邮件地址。此场景需要三个单独的处理程序，其中每个处理程序负责将特定严重性的消息发送到特定位置。

标准库包含了相当多的处理程序类型(参见有用的处理程序);本教程在其示例中主要使用StreamHandler和FileHandler。

应用程序开发人员需要关注的处理程序中的方法很少。与使用内置处理程序对象(即不创建自定义处理程序)的应用程序开发人员相关的唯一处理程序方法是以下配置方法:

setLevel()方法，就像在logger对象中一样，指定将被分派到适当目的地的最低严重性。为什么有两个setLevel()方法?记录器中设置的级别决定它将传递给其处理程序的消息的严重性。每个处理程序中设置的级别决定处理程序将发送哪些消息。

setFormatter()为这个处理程序选择一个Formatter对象。

addFilter()和removeFilter()分别配置和解构处理程序上的filter对象。

应用程序代码不应该直接实例化和使用Handler的实例。相反，Handler类是一个基类，它定义了所有处理程序都应该具有的接口，并建立了一些子类可以使用(或覆盖)的默认行为。

#### 6-3：Formatters

Formatter对象配置日志消息的最终顺序、结构和内容。与基本日志记录不同。处理程序类，应用程序代码可以实例化格式化程序类，不过如果应用程序需要特殊的行为，您可能会子类化格式化程序。构造函数接受三个可选参数——消息格式字符串、日期格式字符串和样式指示器。

logging.Formatter.__init__(fmt=None, datefmt=None, style='%')

如果没有消息格式字符串，默认情况下使用原始消息。如果没有日期格式字符串，则默认日期格式为:

% Y - % m - H % d %: % m: % S
最后加上了毫秒。样式是%，'{'或' $ '之一。如果其中一个未指定，则使用' % '。

如果样式是' % '，消息格式字符串使用%(<字典键>)s样式字符串替换;可能的键记录在LogRecord属性中。如果样式是'{'，则假定消息格式字符串与str.format()兼容(使用关键字参数)，而如果样式是' $ '，则消息格式字符串应符合string. template .substitute()所期望的。

在3.2版更改:添加了style形参。

下面的消息格式字符串将以人类可读的格式记录时间、消息的严重性和消息的内容，顺序如下:

'%(asctime)s - %(levelname)s - %(message)s'
格式化程序使用用户可配置的函数将记录的创建时间转换为元组。默认情况下，使用time.localtime();要为特定的formatter实例更改此属性，请将实例的converter属性设置为具有与time.localtime()或time.gmtime()相同签名的函数。要为所有格式化程序更改它，例如，如果您希望所有的日志记录时间都以GMT显示，请在Formatter类中设置converter属性(为time)。GMT显示的gmtime)。

### 7. 配置

程序员可以用三种方式配置日志:

使用调用上面列出的配置方法的Python代码显式创建记录器、处理程序和格式化程序。

创建一个日志配置文件并使用fileConfig()函数读取它。

创建配置信息的字典，并将其传递给dictConfig()函数。

有关最后两个选项的参考文档，请参见配置函数。下面的示例使用Python代码配置了一个非常简单的记录器、一个控制台处理程序和一个简单的格式化程序:

```python
import logging

# create logger
logger = logging.getLogger('simple_example')
logger.setLevel(logging.DEBUG)

# create console handler and set level to debug
ch = logging.StreamHandler()
ch.setLevel(logging.DEBUG)

# create formatter
formatter = logging.Formatter('%(asctime)s - %(filename)s - %(relativeCreated)s - %(levelname)s - %(message)s')

# add formatter to ch
ch.setFormatter(formatter)

# add ch to logger
logger.addHandler(ch)

# 'application' code
logger.debug('debug message')
logger.info('info message')
logger.warning('warn message')
logger.error('error message')
logger.critical('critical message')

```



# GIT

## config

* git config -e  :将打开项目目录下的配置文件进行修改
* git  config -e --global:打开用户主目录下的配置文件修改
* git config -e --system：打开全局 配置文件

## log

* git log 
  * --pertty=fuller：较全面的信息
  * --pertty=oneline：只显示commit_id和message
  * --stat:看到每次提交的文件变更 统计

## commit

*  git commit --amend：有两个用法
   *  重新修改提交信息，当你对提交信息不满意时使用
   *  当发现少提交了文件的时候，你可以再往暂存区add一个文件，然后再使用本命令，但如果多提交的话就得使用reset了

## diff

* git diff:查看工作区和暂存区的差异 
* git diff HEAD：查看工作区和版本库head指针指向版本的比较
* git diff --cached:查看暂存区和版本库head指针指向版本的比较

## checkout

1. git  checkout [commit_id] [path], commit默认为暂存区，

   * git checkout filename:将工作区文件恢复到和暂存区一样

   * git checkout HEAD filename:指向不变，将工作区文件恢复到和HEAD一样，注意如果此时暂存区有该文件，暂存区也会被覆盖掉
2. git  checkout [commit_id]   切换分支 ，切换指向
3. git  checkout   [-b] [new_branch] ：以当前分支为基础新建分支并切换
4. git checkout -b new_branch origin/branch_name: 以远程分支为基础新建分支并切换

## clean

* git clean -n:查看工作区没有加入版本库的文件和目录
* git clean -fd:清除工作区没有加入版本库的文件和目录
* git checkout .:用暂存区的文件刷新工作区，执行以上两步操作后，工作区会变得干净

## rm

* git rm filename:不再跟踪某文件，从暂存区删除文件，工作区文件也消失，然后提交
* git rm --cache filename:不再跟踪某文件，从暂存区删除文件，但保留工作区文件，在忘记设置ignore时尤其有用，然后提交

## mv

* git mv sourcename targername:修改文件名称

## stash

没有被版本库跟踪的文件是无法保存进度的,就是说必须是add的，在object中生成了的

* git stash  [save message] :将当前分支的所有修改封存，可自己命名
* git stash list:显示保存列表，可输出别名
* git stash pop [别名],应用并且删除
* git  stash apply:与pop功能类似，但是不删除保存信息
* git   stash drop [stash]:删除保存
* git stash clear:清空保存
* git stash branch \<branchname> \<stash>:基于保存的信息创建新分支，这是在新分支轻松恢复贮藏工作并继续工作的一个很不错的途径

## reset

一些概念

* HEAD：指向当前分支的最后一次提交，每次提交后， .git/refs/head/master文件的指向会变化为最新的提交id
* INDEX：暂存区
* 工作区

reset过程

* 修改HEAD 的指向，我们可以当成一个游标，人为修改工作区指向
* 当然，这是一个比较危险的操作，因为会将提交历史丢弃，此时已无法通过浏览提交历史查看被丢弃的提交

用法

* git reset --hard:可以认为此命令最危险，会将所有修改丢弃，保持和指向一致
* git reset --soft:替换master的引用，不改变暂存区和工作区，这个功能就是更改提交和压缩体积用的
  * git reset --soft HEAD^:替换master的引用，当对提交信息不满意时，可重新提交，还记得amend吗？上一次的提交会回到暂存区,因此用此命令也可实现压缩提交的作用
* git reset --mixed: 默认为mixed，替换master的引用，用版本库的内容刷新暂存区，暂存区的内容重新回到工作区
  * git reset [HEAD] filename：等于说指向无变化，只是将暂存区文件删除，相当于撤销add,与使用git rm --cache filename效果一样
* git reset --merge:
* git reset --keep:

总结

* hard：更改head指向，index与指向保持一样，工作区保持和index一样，等于说三者一样
* soft：更改head指向，所有的提交退回index，工作区保持不变
* mixed：更改head指向，index与指向保持一样，所有提交内容退回工作区

## revert

作用：当合并分支时，如果出问题想回滚使用reset --hard固然可以，但是如果提交已经被人拉取了，这样无疑历史被修改了，如果此时使用

git revert -m 1 HEAD会重新生成一个提交还原

## reflog

挽留被错误reset丢弃的提交

查看提交日志可以发现所有的提交历史和reset历史都被记录下来了：cat .git/logs/refs/heads/master

* git reflog show master:按照时间倒序查看commit和reset历史，而且git给了一个操作的表达式，我们可以应用这个表达式再进行返回
* git reset --hard master@{1}:我们先用上面的命令输出表达式，找到之后应用，再将被丢弃的提交找回来

## ignore

* 可以在gitignore忽略自身
* 可以常见独享的忽略文件，方便自己做测试

## 归档：方便的生成文件

## 单步悔棋

* 先 使用git checkout HEAD^  -- filename，将某文件还原到暂存区
* git commit --amend修改提交信息

## 多步悔棋

当程序员接到任务，在本地任务进行开发时，已经进行了多次提交，在push到远程合并时，想合并提交为一个完整的提交时

* git reset --soft commit_id，回到某个提交，文件回到暂存区
* git commit -m,重新提交 ，既能压缩为一个提交

## rebase

语法：git rebase --onto [newbranch] [since] [till]

将since到till的内容写到临时文件，然后嫁接到newbranch分支上

* 合并提交记录 git rebase - HEAD~3，压缩最近三次，如果已经push到远程仓库就不建议合并了
* 让提交历史更加整洁
  * git rebase master:将master的最新修改应用到本分支上
  * git checkout master,git merge dev:切换回master，将dev开发的新功能合并过来，这样这两个分支都指向rebase后的那次提交

遇到rebase冲突

* 解决冲突
* git add filename:标记文件冲突已解决
* git  rebase --continue：继续rebase直到成功

什么时候不要使用rebase

* 当你的代码已经提交到远程服务器上了，再提交会更改提交历史，如果别人在使用本分支，就会对别人造成影响

![image-20210220134914661](/Users/shenshaoxuan/Library/Application Support/typora-user-images/image-20210220134914661.png

## 丢弃历史

## 反向提交

## 克隆

* git clone \<repo> \<directory>:克隆项目并创建工作区
* git clone --bare \<repo> \<directory.git>：克隆项目，直接克隆版本库内容，不创建工作区
* git clone --mirror \<repo> \<directory.git> ：克隆项目，直接克隆版本库内容，不创建工作区，但是可以fetch同步

## pull

基本命令：**git pull origin** 远程分支/本地分支(可以省略，默认和当前分支合并)

1. 自动合并
   * 修改不同的文件
   * 修改相同文件的不同区域
   * 同时更改文件名和文件内容
2. 逻辑冲突，指的是修改了文件名导致引入失败，或者修改了函数返回值等，隐藏着 巨大BUG,所以需要单元测试
3. 冲突解决, merge时失败，手动解决后add，commit, 等于说这次commit就是我们手动代替自动的过程，然后push
4. 树冲突，同时修改文件名字，需要商量确定文件名字，使用git rm删除不需要的文件即可

## merge

Git merge

## tag

命令：

* git tag:显示标签列表
* git log --decorate:可以看到标签

标签类型：

* 轻量级:git tag tagname commit, 无记录，无法知道谁创建的
* 带说明的里程碑:git tag -m 'message' tagname commit, 有记录

推送标签

* git push origin tagname:推送单次标签
* git push origin --tags:推送所有标签

删除标签

* git push origin --delete \<tagname>

## branch

* git branch
  * -r:查看本地已存在的远程分支
  * --no-merged
  * --merged:查看哪些分支已经合并到当前分支
  * -vv:列出所有本地分支和本地分支跟踪的远程分支
* 创建分支：git branch [branchname] [point]
* 删除分支: git branch -d/-D branchname
* 修改分支名:git branch -M/-m branchname
* 删除远程分支：git push origin --delete xxx

## push

* git push origin --delete branch_name:删除远程分支

## remote

* git remote:获取配置的远程分支
  * -v : 获取远程分支的名称和url
* git remote add \<shortname> \<url>: 给本地版本库添加一个远程仓库
* git remote show origin:展示远程仓库详情，需要联网
* git remote rename origin new_orign:远程仓库的重命名与移除
* git remote update origin --prune 清理本地分支和远程同步
* git remote set-url origin url:修改 git仓库地址

## gitflow

## 其他问题解决

* 解决git status显示中文乱码：git config --global core.quotepath false 

  

# Golang

## proto

### 安装

* 下载对应版本，版本如果对应不上，编译也许会出现问题
* [下载链接](https://github.com/protocolbuffers/protobuf/releases/tag/v3.7.1)
  * ./configure 
  * make
  * make check 
  * sudo make install 
  * sudo ldconfig ＃刷新共享库缓存。
* 查看版本：protoc --version 

### 编译命令

* -I 就是如果多个proto文件之间有互相依赖，生成某个proto文件时，需要import其他几个proto文件，这时候就要用-I来指定搜索目录
* --go_out,指的是编译成go文件，不同的语言有不同的指定

```shell
protoc -I/usr/local/include -I. \
  -I$GOPATH/src \
  -I$GOPATH/src/github.com/grpc-ecosystem/grpc-gateway/third_party/googleapis \
  --go_out=plugins=grpc:. \
  ./proto/supplier-platform/indent.proto

protoc -I/usr/local/include -I. \
  -I$GOPATH/src \
  -I$GOPATH/src/github.com/grpc-ecosystem/grpc-gateway/third_party/googleapis \
  --grpc-gateway_out=logtostderr=true:. \
  ./proto/supplier-platform/indent.proto
```

## 语言语法

### 疑问点

* "hexcloud.cn/histore/supplier-platform/pkg/logger"，导入时前面的前缀是什么情况？如果项目可以放在任何目录，那么是如何确定导入路径的？
  * module hexcloud.cn/histore/supplier-platform, go.mod中的第一句话，可能是这个起的作用，待确定，我猜可能是为了让我们把项目放在任何路径，所以提供了一个设置告诉go他本来的位置

* 声明完未赋值，整型和float默认为0， 字符串默认为空字符串，布尔值默认为false

### 编译

* go build:在当前目录生成一个可执行文件
* go  run: 编译到 临时目录，然后执行
* go install:先执行build，然后拷贝到bin目录
* 也可以跨平台编译，比如在windows开发，在linux运行，需要设置一下环境变量进行编译

### 注释

```go
//单行注释
/*
多行注释
*/
```

### 声明常量和变量

```go
package main

import (
	"fmt"
	"reflect"
)

//定义常量
const name = 1

//iota:常量计数器，只能在常量定义中使用，
//在const出现时被重置为0，每一行加1，在定义枚举值时比较有用
const (
	KB = 1 << (10 * iota)
	MB = 1 << (10 * iota)
	GB = 1 << (10 * iota)
	TB = 1 << (10 * iota)
	PB = 1 << (10 * iota)
)

func main() {
	//声明变量
	var name1, name2 string

	//声明变量并赋值
	var name3 string="3"

	//声明多个变量并赋值
	var name4, name5 string = "4", "5"

	//批量声明的另外一种写法
	var (
		name6 string = "6"
		name7 int = 7
	)

	//简短声明，但是只能声明在函数内部，在函数外部使用会报错，在外部是为了声明全局变量
	name8, name9 := 8, 9

	//匿名变量，不占用命名空间，使用 _ 丢弃赋值
	_, name10 := 1,10

	//变量声明必须使用，否则编译失败
	fmt.Println(name1, name2, name3, name4, name5, name6, name7, name8, name9, name10)
	fmt.Println(reflect.TypeOf(name))
	fmt.Println(KB, MB, GB, TB, PB)
}
```

### fmt.print

```go
package main

import "fmt"

//定义常量
const name = 1
const s = "hello"

func main() {
	//打印，但是打印完毕不换行
	fmt.Print(name)
	//打印，并换行
	fmt.Println(name)
	//格式化输出
	fmt.Printf("变量是:%d\n",  name)
	fmt.Printf("类型是:%T\n",  name)
	fmt.Printf("value是:%v\n",  name)
	fmt.Printf("value是:%v\n",  s)
	fmt.Printf("value是:%#v\n",  s)
}
```

### 数据类型

#### byte

字节类型，一个字节是int8

编码

* ascii：最早计算机定义的字符的对应关系,当时字符比较少，256个够了，所以只需要一个字节
* unicode：发现不够使用时，发明了万国码，32位，一个字符要占4个字节
* utf-8：优化了存储，根据unicode字符集，自动选择字符的存储，中文是占3个字节

```go
package main

import (
	"fmt"
	"reflect"
)

func main(){
	s1 := 1
	s2 := "沈少轩"
	s3 := "ssx"
	fmt.Println(s1, s2, s3)
	fmt.Println(reflect.TypeOf(s1))
	fmt.Println(reflect.TypeOf(s2))
	fmt.Println(reflect.TypeOf(s3))
	fmt.Println(s2[0])
	fmt.Println(s2[1])
	fmt.Println(s2[2])
	fmt.Println(s2[3])
	fmt.Println(s2[4])
	fmt.Println(s2[5])
	fmt.Println(s2[6])
	fmt.Println(s2[7])
	fmt.Println(s2[8])
	for i:=range(s2)  {
		//发现这里的i分别是0， 3， 6，按照中文算的话，一个占三个byte
		fmt.Println(i)
		fmt.Println(s2[i])
	}
}
```

#### rune

当需要处理中文，韩文时，需要用到

rune就是int32

```go
package main

import (
	"fmt"
	"reflect"
)

func main() {
	s1 := "沈少轩"
	fmt.Println(len(s1))
	//转换为切片类型，元素是rune类型，rune是int32
	s2 := []rune(s1)
	fmt.Println(len(s2))
	fmt.Println(reflect.TypeOf(s2))
	//转换为切片类型，元素是byte类型， byte是int8
	s3 := []byte(s1)
	fmt.Println(s3)
	fmt.Println(reflect.TypeOf(s3))
}

```



#### 整型

| uint8  | 无符号 8位整型 (0 到 255)                                    |
| :----- | ------------------------------------------------------------ |
| uint16 | 无符号 16位整型 (0 到 65535)                                 |
| uint8  | 无符号 32位整型 (0 到 4294967295)                            |
| uint8  | 无符号 64位整型 (0 到 18446744073709551615)                  |
| Int8   | 有符号 8位整型 (-128 到 127)                                 |
| Int16  | 有符号 16位整型 (-32768 到 32767)                            |
| int32  | 有符号 32位整型 (-2147483648 到 2147483647)                  |
| Int64  | 有符号 64位整型 (-9223372036854775808 到 9223372036854775807) |

| 特殊类型 |                          描述                          |
| :------: | :----------------------------------------------------: |
|   uint   | 32位操作系统上就是`uint32`，64位操作系统上就是`uint64` |
|   int    |  32位操作系统上就是`int32`，64位操作系统上就是`int64`  |
| uintptr  |              无符号整型，用于存放一个指针              |

```go
package main

import (
	"fmt"
	"reflect"
)

func main(){
	//无符号整型，  uint8, uint16, uint32, uint64
	//有符号整型, int8, int16,  int32, int64
	//如果不声明类型，那么go语言会认为是什么类型呢
	i1 := 1
	i2 := 257
	i3 := 65536
	//可以 看到都是int类型，在64位 系统上默认是int64
	fmt.Println(reflect.TypeOf(i1))
	fmt.Println(reflect.TypeOf(i2))
	fmt.Println(reflect.TypeOf(i3))
	i4 := 077
	i5 := 0xe2
	//可以看到go无法声明8进制和16进制数，go都会把他们转换为int类型
	fmt.Println(i4, i5)
	fmt.Println(reflect.TypeOf(i4))
	fmt.Println(reflect.TypeOf(i5))
	//明确指定类型
	var (
		i6 int8
		i7 int16
	)
	i6, i7 = 1000, 200
	//如果溢出定义是会报错的
	fmt.Println(i6, i7)
	fmt.Println(reflect.TypeOf(i6))
	fmt.Println(reflect.TypeOf(i7))
}
```

#### 浮点数

```go
package main

import (
	"fmt"
	"reflect"
)

func main(){
	f1 := 1.234
	f2 := 257.32
	//可以 看到都是float64类型
	fmt.Println(f1, f2)
	fmt.Println(reflect.TypeOf(f1))
	fmt.Println(reflect.TypeOf(f2))
	//明确指定类型
	var (
		f3 float32
		f4 float64
	)
	f3, f4 = 1000, 200
	fmt.Println(f3, f4)
	fmt.Println(reflect.TypeOf(f3))
	fmt.Println(reflect.TypeOf(f4))
}
```

#### bool

```go
package main

import (
	"fmt"
)

func main() {
	b1 := false
	fmt.Println(b1)
	var b2 bool
	b2 = true
	fmt.Println(b2)
	fmt.Println(b1 == b2)
	//不同类型不能比较
	//fmt.Println(b1 == 1)
	//布尔也不能转为整型
	//fmt.Println(int(b1))
  //布尔不能参与运算，也永远无法和其他类型进行转换
}
```

#### 字符串

```go
package main

import (
	"fmt"
	"reflect"
	"strings"
)

func main() {
	//单引号包裹的是字符，双引号才是字符串
  //在go中占4个字节，是rune(int32)类型
	i := 'A'
	fmt.Println(i)
	fmt.Println(reflect.TypeOf(i))
	s1 := "A"
	fmt.Println(s1)
	fmt.Println(reflect.TypeOf(s1))
	//多行字符串，原样输出，这样就不用转义了，比如打印路径
	s2 := `
宝剑锋从磨砺出`
	fmt.Println(s2)
	fmt.Println(reflect.TypeOf(s2))
	//字符串相关操作
	//长度,求的是byte字节的数量，大部分中文占3个
	fmt.Println(len(s2))
	//拼接
	fmt.Println(s1 + s2)
	fmt.Printf("%s%s\n",  s1,  s2)
	s3 := fmt.Sprintf("%s%s\n",  s1,  s2)
	fmt.Println(s3)
	//切割
	s4 := "ssx"
	fmt.Println(strings.Split(s4, ""))
	//包含
	fmt.Println(strings.Contains(s4, "s"))
	//以开始
	fmt.Println(strings.HasPrefix(s4, "s"))
	//以结束
	fmt.Println(strings.HasSuffix(s4, "s"))
	//获取第一个index的位置
	fmt.Println(strings.Index(s4, "x"))
	//获取第一个index的位置
	l1 := []string{"沈", "少", "轩"}
	fmt.Println(strings.Join(l1, ""))
	fmt.Println(len(strings.Join(l1, "")))
	//还有很多，可以在string里看
}

```

```go
package main

import (
	"fmt"
)

func main() {
	//修改字符串
	s1 := "沈少轩"
	//转换为切片类型，元素是rune类型，rune是int32
	s2 := []rune(s1)
	s2[0] = '王'
	fmt.Println(string(s2))
}
```



#### 数组(值类型)

数组的局限性

* 长度是一定的
* 数组中元素类型是相同的

```go
  package main

  import "fmt"

  func main() {
    //声明数组，说明数组的长度和数据的类型
    var a1 [5]int
    //数组的初始化，不初始化默认为0值
    fmt.Println(a1)
    //初始化方式1
    a1 = [5]int{1, 2, 3, 4, 11}
    fmt.Println(a1)
    //初始化方式2,根据初始值，自动推断长度
    a2 := [...]int{1, 2, 3, 4, 11}
    fmt.Println(a2)
    //初始化方式3，指定索引赋值，不赋值则为0
    a3 := [3]int{0: 1, 2: 13}
    fmt.Println(a3)

    //不能超过索引范围
    //a1[10] = 1
    //修改数组
    a3[1] = 11
    fmt.Println(a3)

    //多维数组的两种声明方式
    a4 := [...][2]int{[2]int{1, 2}, [2]int{3, 4}}
    fmt.Println(a4)
    a5 := [...][2]int{{1, 2}, {3, 4}}
    fmt.Println(a5)

    //数组不能改变长度，通过函数传入的数组是数组的副本，如果需要修改原数组，需要用到指针，用的slice类型
    //测试数据是否会改变呢，答案是不会，传递的是副本，数组是值类型，不是引用类型
    test1(a3)
    fmt.Println("函数运行完成后", a3)
  }

  func test1(aaa [3]int) {
    aaa[0] = 100
    fmt.Println("函数内改变数组", aaa)
  }
```

```go
package main

import "fmt"

func main() {
	//1. 求数组的和
	a1 := [...]int{1, 3, 5, 7, 8}
	i := 0
	for _, v := range a1 {
		i += v
	}
	fmt.Println("数组的和为", i)
	var result [][2]int
	//2. 求两个元素的和为8的索引，比如(0,3), (1,2)
	for index, value := range a1 {
		for inIndex, InValue := range a1[index+1:] {
			if value+InValue == 8 {
				result = append(result, [2]int{index, inIndex + index + 1})
				break
			}
		}
	}
	fmt.Println(result)
}
```

#### 切片

* 长度是可变的
* 底层也是一个数组
* 切片不能直接比较，唯一的合法比较是和nil比较，nil值的切片长度和容量是0，但是长度和容量是0的切片不一定是nil,所以要判断切片是不是空，应该用len判断 ，而不是与 nil比较

```go
package main

import (
	"fmt"
	"sort"
)

func main() {
	//1. 声明一个切片
	var s1 []int
	//只声明不赋值，指向的是nil对象
	fmt.Println(s1 == nil)

	//声明切片并初始化
	s2 := []bool{true, false}
	s3 := []int{1, 2, 3, 4}
	s4 := []int{1, 2, 3, 4}
	fmt.Println(s1, s2, s3, s4)
	//slice can only be compared to nil, slice是引用类型，不支持slice之间比较
	//fmt.Println(s4 == s3)
	//长度和容量
	fmt.Println("数组的长度是：", len(s4), "数组的容量是：", cap(s4))
	fmt.Println("************************************************由数组得到切片")
	//2. 由数组得到切片
	a1 := [5]int{1, 2, 3, 4, 5}
	s5 := a1[:2]
	s6 := a1[2:]
	fmt.Printf("类型是%T, value是%v\n", s5, s5)
	fmt.Printf("类型是%T, value是%v\n", s6, s6)

	//更新切片查看原数组的情况, 经查看原数组随之改变
	//数组的切片的容量是指从切片的第一个元素到数组的最后一个元素
	s5[0] = 1000
	fmt.Printf("s5类型是%T, value是%v, 容量是%d\n", s5, s5, cap(s5))
	fmt.Printf("a1类型是%T, value是%v, 容量是%d\n", a1, a1, cap(a1))
	//TODO:
	s5 = append(s5, 11,12,13,14)
	fmt.Printf("s5类型是%T, value是%v, 容量是%d\n", s5, s5, cap(s5))
	fmt.Printf("a1类型是%T, value是%v, 容量是%d\n", a1, a1, cap(a1))
	//更改原数组，查看切片的情况,切片也变了，印证了切片是引用类型
	a1[0] = 1000
	fmt.Println(s5)
	fmt.Println("************************************************由切片得到切片")
	//3. 由切片得到切片
	s8 := s3[:1]
	s9 := s3[1:]
	//经查看，切片的切片容量根数组的切片规则一样
	fmt.Printf("s8类型是%T, value是%v, 容量是%d\n", s8, s8, cap(s8))
	fmt.Printf("s9类型是%T, value是%v, 容量是%d\n", s9, s9, cap(s9))
	fmt.Printf("s3类型是%T, value是%v, 容量是%d\n", s3, s3, cap(s3))
	//尝试更改切片，然后观察原切片，切片是对象引用
	fmt.Println("原切片是：", s3)
	s9[0] = 100
	fmt.Println("切片更新后原切片更新为：", s3)
	//尝试append超过原切片长度时,原切片并不会变长，而是被侵占直到无法侵占位置
	//而slice对象的空间不足时，将动态分配新的空间，而原对象容量不变
	//TODO:
	s8 = append(s8, 101, 102, 103, 104)
	fmt.Printf("切片append后原切片更新为:%v, 容量是%d\n", s3, cap(s3))
	//更改原切片，查看切片的切片,随之改变
	s3[0] = 1001
	fmt.Println(s8)
	//为什么切片的容量是5呢
	array := [10]int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
	s10 := array[2:4:7]
	fmt.Println(cap(s10), s10)

	fmt.Println("************************************************使用make创建切片")
	//3. 使用make创建切片
	//会有默认的容量，并且初始化了值，不再是nil类型
	s11 := make([]int, 3, 8)
	fmt.Println(s11 == nil)
	fmt.Printf("s11类型是%T, value是%v, 长度是%d, 容量是%d\n", s11, s11, len(s11), cap(s11))
	//切片不能直接比较，唯一的合法比较是和nil比较，nil值的切片长度和容量是0，但是长度和容量是0的切片不一定是nil
	s12 := make([]int, 0, 0)
	fmt.Println(s12 == nil)
	fmt.Printf("s12类型是%T, value是%v, 长度是%d, 容量是%d\n", s12, s12, len(s12), cap(s12))
	fmt.Println("************************************************")
	//4，append和copy和sort
	var s13 []int
	s14 := [3]int{1, 2, 3}
	s13 = append(s13, s14[:]...)
	fmt.Printf("s13类型是%T, value是%v, 长度是%d, 容量是%d\n", s13, s13, len(s13), cap(s13))
	s15 := make([]int, 3, 3)
	copy(s15, s13)
	fmt.Println(s15)
	//copy的对象更改不会影响原切片
	s13[0] = 100
	fmt.Println(s13, s15)
	s16 := [5]int{12, 1, 15, 18, 3}
	s17 := s16[:4]
	sort.Ints(s17)
	fmt.Println(s16, s17)
}
```

#### map

```go
package main

import (
	"fmt"
	"math/rand"
	"sort"
	"strings"
)

func main() {
	//创建map
	m1 := make(map[int]string)
	m1[1] = "a"
	fmt.Println(m1)
	//印证map是引用类型
	m2 := m1
	m2[1] = "xx"
	fmt.Println(m1, m2)

	//初始化
	m3 := make(map[string]string, 3)
	m3["name"] = "ssx"
	m3["age"] = "15"
	fmt.Println(m3)

	//取值, 约定俗成用ok接收返回的布尔值
	class, ok := m3["class"]
	if !ok {
		fmt.Println(ok)
	} else {
		fmt.Println(class)
	}
	//删除
	delete(m3, "name")
	fmt.Println(m3)

	//对map进行排序
	scoreMap := make(map[string]int, 200)
	for i := 1; i < 100; i++ {
		key := fmt.Sprintf("stu%02d", i)
		value := rand.Intn(100)
		scoreMap[key] = value
	}
	//fmt.Println(scoreMap)
	key := make([]string, 0, 200)
	for k := range scoreMap {
		key = append(key, k)
	}
	sort.Strings(key)
	for _, value := range key {
		fmt.Println(value, scoreMap[value])
	}

	//类型为map的切片
	s1 := make([]map[string]string, 3, 10)
	m4 := make(map[string]string, 1)
	m4["name"] = "ssx"
	s1 = append(s1, m4)
	fmt.Println(s1)

	//统计字符串中单次出现的次数
	string1 := "how do you do"
	slice1 := strings.Split(string1, " ")
	fmt.Println(slice1)
	wordMap := make(map[string]int, 10)
	for _, value := range slice1 {
		if v, ok := wordMap[value]; !ok {
			wordMap[value] = 1
		} else {
			wordMap[value] = v + 1
		}
	}
	fmt.Println(wordMap)
}
```

#### 指针

* &：取地址
* *：取值

```go
package main

import (
	"fmt"
)

func main() {
	//1. 声明一个指针类型
	//var p1 *int
	//注意，这种情况下会panic，因为只是声明并未申请内存地址
	//*p1=100
	//fmt.Println(*p1)

	//使用new申请内存地址
	p2 := new(int)
	*p2 = 100
	fmt.Println(*p2)

	i1 := 100
	p3 := &i1
	fmt.Println(&i1)
	fmt.Println(p3)
	fmt.Println(&p3)
	*p3 = 200
	fmt.Println(i1)
}
```

#### new和make

go中的对于引用类型的变量，除了生命它，还要为它分配内存，否则无法存储，所以用到make和new用来分配内存

**new**

* 传入类型，申请内存地址，返回一个指针，返回类型的零值
* string或者int才需要使用这种方式，而切片，map等引用类型本身就认为是指针类型了

**make**

* make也是用来分配内存，区别于new，它只用于slice， map，chanel的创建，而且返回的是这三个类型的本身，并不返回指针类型，因为他们就是引用类型
* make是不能替代的，我们创建上面三种类型时，必须使用make

#### error

#### 类型转换

### 流程控制

#### if-else

```go
package main

import (
	"fmt"
)

func main() {
	s1 := 16
	if s1 > 5 {
		fmt.Println("baby")
	} else if s1 > 18 {
		fmt.Println("成年")
	} else {
		fmt.Println("人")
	}
	//if的条件中可以声明变量
	if s2 := 16;s2 > 5 {
		fmt.Println("baby")
	} else if s2 > 18 {
		fmt.Println("成年")
	} else {
		fmt.Println("人")
	}
}
```

#### switch

```go
package main

import "fmt"

func main() {
	n := 1
	switch n {
	case 1:
		fmt.Println("大拇指")
	case 2:
		fmt.Println("食指")
	case 3:
		fmt.Println("食指")
	case 4:
		fmt.Println("食指")
	case 5:
		fmt.Println("食指")
	default:
		fmt.Println("无效")
	}
	n = 7
	switch n {
	case 1, 3, 5, 7, 9:
		fmt.Println("奇数")
	case 2, 4, 6, 8, 10:
		fmt.Println("偶数")
	default:
		fmt.Println("无效数字")
	}
}
```

#### for

```go
package main

import "fmt"

func main() {
	//s1 := "string"
	s1 := "沈少轩"
	//基本格式，循环字符串是byte类型
	for i := 0; i < len(s1); i++ {
		fmt.Println(s1[i])
	}
	fmt.Println("************************")
	//键值循环，使用range，返回index和value，用来循环slice，array，map等类型
	s2 := [3]int{10, 21, 13}
	s3 := []int{14, 12, 3}
	s4 := make(map[int]string)
	s4[1] = "a"
	//字符串迭代出来是int32,可以看出go用的是rune处理的,用string转换
	for _, v := range s1 {
		fmt.Printf("%c:%T\n", v, v)
	}
	fmt.Println("************************array")
	for _, v := range s2 {
		fmt.Println(v)
	}
	fmt.Println("************************slice")
	for _, v := range s3 {
		fmt.Println(v)
	}
	fmt.Println("************************map")
	for key, value := range s4 {
		fmt.Println(key, value)
	}
}
```

```go
package main

import "fmt"

func main() {
	//打印99乘法表
	for i := 1; i < 10; i++ {
		for j := 1; j <= i; j++ {
			fmt.Printf("%d x %d = %d\t", i, j, i*j)
		}
		fmt.Println()
	}
}

```

### 函数

#### 基本定义

```go
package main

import (
	"errors"
	"fmt"
)

//函数定义
//无参数
func f1() {
	fmt.Println("无参数，无返回值的函数")
}

//无返回值
func f2(x int, y int) {
	fmt.Println("无返回值的函数", x+y)
}

//有返回值
func f3(x int, y int) int {
	return x + y
}

//设置返回参数名的用处是可以在函数内使用，并直接返回
func f4(x int, y int) (result int) {
	result = x + y
	return
}

//多个返回值的函数
func f5(x int, y int) (result int, error error) {
	result = x + y
	if result > 100 {
		error = errors.New("结果太大啦")
		return
	}
	return
}

//不定长参数
func f6(args ...int) {
	fmt.Printf("类型是%T， value是%v\n", args, args)
}

//go的函数参数是不能设置默认值的

func main() {
	f1()
	f2(1, 2)
	fmt.Println(f3(1, 2))
	fmt.Println(f4(1, 2))
	result, err := f5(100, 2)
	if err != nil {
		fmt.Println(err)
	} else {
		fmt.Println(result)
	}
	f6(1, 2, 3, 4, 100)

	s1 := []int{3, 5, 7, 2, 1, 87, 6}
	fmt.Println(max(s1))

  //值类型传递的都是副本
	//测试slice这种引用对象是否会被函数改变，答案是会，因为他们是引用类型
	test2(s1)
	fmt.Println(s1)

	//如果真的需要改变原值，不使用副本，我们需要传递指针对象，channel，slice，map这三种类型的实现机制类似指针
	//传指针比较轻量，当传递大数据时用指针比较好，如果用参数值，每次copy花费时间较多
	s2 := 4
	test3(&s2)
	fmt.Println(s2)
}

//定义一个函数排序
func max(values []int) int {
	for i := 0; i < len(values)-1; i++ {
		for j := i + 1; j < len(values); j++ {
			if values[i] > values[j] {
				values[i], values[j] = values[j], values[i]
			}
		}
	}
	return values[0]
}

func test2(s []int) {
	s[0] = 1000
	fmt.Println(s)
}

func test3(s *int) {
	*s = *s + 1
}
```

### defer

* 注意运行到defer时，如果发现defer定义的函数内有嵌套的，会先把内部的给执行掉
* 注意运行到defer时，变量的值已经压栈，不会随着变量的更改影响到defer运行的参数值

```go
package main

import "fmt"

func main() {
	//查看defer的执行顺序
	deferDemo()
	//四种defer的面试题
	fmt.Println(f1())
	fmt.Println(f2())
	fmt.Println(f3())
	fmt.Println(f4())
}

func f1() int {
	x := 5
	defer func() {
		x++
	}()
	return x
}

func f2() (result int) {
	x := 5
	defer func() {
		x++
	}()
	return x
}

func f3() (x int) {
	defer func() {
		x++
	}()
	return 5
}

func f4() (x int) {
	defer func(x int) {
		x++
	}(x)
	return 5
}

func deferDemo() {
	fmt.Println("before defer")
	defer fmt.Println("defer1.......")
	defer fmt.Println("defer2.......")
	defer fmt.Println("defer3.......")
	fmt.Println("return result")
}
```

### 函数类型,作为参数传递

```go
package main

import "fmt"

func main() {
	//函数也可以作为参数传递
	fmt.Println(f(f1))
	//函数也可以作为返回值
	ff := f2()
	fmt.Printf("%T", ff)
}

func f(f func()int) int {
	ret := f()
	return ret
}

func f1() int {
	return 1
}

func f2() func()int {
	return f1
}
```

### 匿名函数

一般用在函数内部

```go
package main

import "fmt"

func main() {
	//命名
	f1 := func() {
		fmt.Println("我是一个匿名函数")
	}
	f1()
	//一次性的
	func(x, y int) {
		fmt.Println("我是一个匿名函数")
		fmt.Println(x + y)
	}(1, 2)
}

```

### 闭包

```go
package main

import "fmt"

func main() {
	r := outer(f, 1000)(1,2)
	fmt.Println(r)
}
func outer(f func(int, int) int, xxx int) func(m,n int) int{
	tem := func(x,y int) int {
		fmt.Println("start")
		fmt.Println("我可以在函数内引用装饰器的参数", xxx)
		result := f(x, y)
		fmt.Println("end")
		return result
	}
	return tem
}

func f(x int,y int) int{
	return x + y
}
```

### panic和recover

```go
package main

import "fmt"

func main() {
	f1()
	f2()
	f3()
}
func f1() {
	fmt.Println("f1")
}

func f2() {
	//如果是刚启动service就发生了致命错误，就释放掉数据库连接
	defer func() {
		if r := recover(); r != nil {
			fmt.Println("释放数据库连接")
		}
	}()
	panic("f2出现了严重的错误") //程序崩溃退出
}

func f3() {
	fmt.Println("f3")
}
```

### 递归

### 自定义类型

### 结构体

#### 定义结构体

```go
package main

import "fmt"

//定义结构体
type Person struct {
	name  string
	age   int
	hobby []string
}

func main() {
	var p1 Person
	p1.name = "ssx"
	p1.age = 15
	p1.hobby = []string{"eat", "drink", "play"}
	fmt.Printf("%T, %v\n", p1, p1)
	
	p5 := Person{
		name:  "张三",
		age:   15,
		hobby: []string{"1"},
	}
	fmt.Println(p5)
}
```

#### 结构体是值类型

```go
package main

import "fmt"

//定义结构体
type Person struct {
	name  string
	age   int
	hobby []string
}

func main() {
	var p1 Person
	p1.name = "ssx"
	p1.age = 15
	p1.hobby = []string{"eat", "drink", "play"}
	fmt.Printf("%T, %v\n", p1, p1)
	
	fmt.Println("*******************************验证结构体是值类型")
	p3 := p1
	p3.name = "沈少轩"
	fmt.Println(p3, p1)
	fmt.Println("*******************************使用指针修改")
	p4 := new(Person)
	p4 = &p1
	p4.name = "沈少轩"
	fmt.Println(*p4, p1)
	update(&p1)
	fmt.Println(p1)
}

func update(p *Person) {
	p.name = "王大锤"
}

```

#### 构造函数

```go
package main

import "fmt"

//定义结构体
type Person struct {
	name  string
	age   int
	hobby []string
}

//构造函数，约定以new开头,根据业务选择返回值或者指针，当结构体比较大是选择指针
func newPerson(name string, age int, hobby []string) Person {
	return Person{
		name:  name,
		age:   age,
		hobby: hobby,
	}
}
```



#### 方法

```go
package main

import "fmt"

//定义结构体
type Person struct {
	name  string
	age   int
	hobby []string
}

//方法和接收者,p表示调用者，类似python的self，GO推荐用类型首字母的小写表示
func (p Person) eat(){
	fmt.Printf("%s可以吃饭", p.name)
}

func main() {
	p5 := Person{
		name:  "张三",
		age:   15,
		hobby: []string{"1"},
	}
	fmt.Println(p5)
	p5.eat()
}
```

#### 值接收和指针接收的区别

* 当需要对原对象进行修改
* 当拷贝代价比较大时
* 保证一致性，如果有方法使用指针接收，那么其他方法也使用指针

```go
package main

import (
	"fmt"
)

type person struct {
	name string
	city string
	age  int8
}

func main() {
	//返回结构体的指针,go中结构体指针也可以使用.来操作数据
	var p1 = new(person)
	p1.name = "ssx"
	fmt.Println(p1.name)
	fmt.Println(p1.age)
	fmt.Println(p1)
	
	//在go中直接使用&操作符，结果就是一个结构体指针
	p2 := &person{
		name: "ssx1",
		age: 15,
	}
	fmt.Println(p2)
}
```

#### 任意类型添加方法

有点类似继承

在go中任何类型都可以自定义方法，比如内建类型的 int，string

```go
package main

import "fmt"

type myInt int

func(m myInt) hello(){
	fmt.Println("say hello")
}


func main() {
	i := myInt(100)
	i.hello()
	fmt.Println(i)
}

```

#### 结构体的嵌套

```go
package main

import "fmt"

type Person struct {
	name  string
	age   int
	order Order
	Class //不定义名称的时候默认是类型名
}

type Order struct {
	num      int
	price    float64
	quantity int
}

type Class struct {
	name  string
	count int
}

func main() {
	p1 := Person{
		name: "ssx",
		age:  15,
		order: Order{
			num:      1,
			price:    12.1,
			quantity: 100,
		},
		Class: Class{
			name:  "三年二班",
			count: 45,
		},
	}
	fmt.Println(p1)
	fmt.Println(p1.order.quantity)
	fmt.Println(p1.Class.name)  //取值的时候可以指定匿名的结构体
	fmt.Println(p1.count)
}
```

#### 模拟继承

```go
package main

import "fmt"

type animal struct {
	name string
}

func (a animal) move() {
	fmt.Printf("%s在移动了\n", a.name)
}

type dog struct {
	feet int
	animal
}

func (d dog) run() {
	fmt.Printf("用%d只脚在跑步呢\n", d.feet)
}

func main() {
	d1 := dog{
		feet: 4,
		animal: animal{
			name: "heihei",
		},
	}
	fmt.Println(d1)
	d1.move()
	d1.run()
}
```

#### 结构体与json

```go
package main

import (
	"encoding/json"
	"fmt"
)

type animal struct {
	Name string `json:"name"` //序列化和反序列化使用的字段
}

func (a animal) move() {
	fmt.Printf("%s在移动了\n", a.Name)
}

type dog struct {
	Feet int `json:"feet"`
	animal
}

func (d dog) run() {
	fmt.Printf("用%d只脚在跑步呢\n", d.Feet)
}

func main() {
	d1 := dog{
		Feet: 4,
		animal: animal{
			Name: "heihei",
		},
	}
	fmt.Println(d1)
	d1.move()
	d1.run()
	//序列化
	d, err := json.Marshal(d1)
	if err != nil {
		fmt.Println(err)
	} else {
		fmt.Println(string(d))
	}
	//反序列化,注意这里需要传入指针类型
	var d2 dog
	s := `{"name":"ssx", "feet":100}`
	json.Unmarshal([]byte(s), &d2)
	fmt.Println(d2, d2.Feet, d2.Name)
}
```

### 接口interface



## go mod

### 写入环境变量

```bash
go env -w GOBIN=/Users/shenshaoxuan/go/bin //创建目录存放
go env -w GO111MODULE=on //开启后只会使用modules，不会到gopath下查找包
go env -w GOPROXY=https://goproxy.cn,direct // 使用七牛云的
# 环境变量写入到GOENV
```

项目运行后，go mod会自动查找依赖下载

此时查看

* go.mod，会发现新增了require，自动拉取最新的发布tag，如果没有则拉取最新commit
* go.sum,记录依赖树

到$gopath/pkg/mod下可以查看到依赖的包

* go mod download：下载所有依赖
* go mod tidy：移除不需要的依赖包
* go mod init：初始化 go.mod
* go mod vender：将依赖复制到 vendor 下



# sql优化

## 数据类型的创建优化

* 尽量使用符合的类型，比如定长的使用char，不定长使用varchar，并估算好长度，将定长的数据放在数据表前面
* 不推荐使用枚举类型，枚举的特点：
  * 底层使用数字存储，省空间
  * 如果需要增加枚举值，需要修改字段，代价比较大
  * 不太推荐使用数字作为枚举值，容易混淆，就使用字符串好了
* 在b/s的web架构中，不推荐使用外键，性能是一个原因，并发问题还容易造成死锁

## 大数据量的sql同步脚本

## Left  join容易出现的问题

## in exists join的性能对比

## index

索引实现方式

* hash索引：使用哈希表，取单值很快，但是取区间无法快速
* BTREE

索引类型

* 主键索引
* 唯一索引
* 普通索引
* 组合索引

创建索引

* 查询快
* 插入，更新，需要维护索引，会变慢
* 索引散列值少，不适合创建索引，比如性别

索引命中

* 组合索引的最左前缀匹配(经过测试，发现mysql会自动优化的，不用管顺序)， 组合索引效率大于索引合并
* like的百分号在前面的无法命中，在后面可以
* where条件经过函数处理的无法命中
* or
* 类型不一致的，如果是字符串，而没加引号
* !=,  主键索引可以，其他不会走
* \>是主键索引或者类型是整数的，会走索引，比如>1001
* Order_by，排序时需要使用索引才可以

其他

* 覆盖索引：在索引表就可以查询到数据
* 索引合并：把多个索引合并使用，通过业务确定使用组合索引或者单个索引

查询计划

*  Explain

慢sql日志

* 查看配置：show variables like '%query%';

* 开启慢日志记录：set global  slow_query_log=on;

分页





# mysql

## 导入



# Grpc

## 名词解释

* rpc：远程过程调用，就像在本机调用函数一样调用远程机器的接口
* grpc：google提供的rpc开源框架，基于HTTP/2开发，支持多种语言 
* protobuf：高效的数据转换协议，比json快3-5倍

## 简单示例

### 定义proto

```protobuf
syntax = "proto3";

package hello;

service HelloService {
  rpc Hello(HelloRequest) returns (HelloResponse) {}
}

message HelloRequest {
  string name = 1;
  int32 age = 2;
}

message HelloResponse {
  string result = 1;
}
```



### 编译

```python
python -m grpc_tools.protoc -I. --python_out=. --grpc_python_out=. hello.proto
```



### 定义service

```python
import time

import grpc
import hello_pb2
import hello_pb2_grpc

from concurrent import futures


class HelloService(hello_pb2_grpc.HelloServiceServicer):

    def Hello(self, request, context):
        name = request.name
        age = request.age
        return hello_pb2.HelloResponse(result=name)


def run():
    grpc_server = grpc.server(
        futures.ThreadPoolExecutor(max_workers=4)
    )
    hello_pb2_grpc.add_HelloServiceServicer_to_server(HelloService(), grpc_server)
    grpc_server.add_insecure_port('0.0.0.0:5000')
    grpc_server.start()
    print('server_start...........')
    try:
        while True:
            time.sleep(3600)
    except KeyboardInterrupt:
        grpc_server.stop(0)

run()
```



### 定义client

```python
import grpc
import hello_pb2
import hello_pb2_grpc

conn = grpc.insecure_channel('0.0.0.0:5000')
stub = hello_pb2_grpc.HelloServiceStub(conn)

request = hello_pb2.HelloRequest()
request.name = 'ssx'
request.age = 11

result = stub.Hello(request)
print(result)s
```

## protobuf数据类型

* String:字符串，必须是UTF8字符或者ascci字符集
* byte：比特类型
* bool：布尔类型
* int32
* int64
* repeat：数组
* float
* map

## 如何抛出错误

server

```python
context.set_details('error ssx')
        context.set_code(grpc.StatusCode.NOT_FOUND)
        raise context
```



client

```python
try:
    result = stub.Hello(request)
except Exception as e:
    print(e.code())
    print(e.details())
```



## 如何传输header

在rpc中不是header的概念，而是metadata元数据

server

```python
headers = context.invocation_metadata()
print(dict(headers))
context.set_trailing_metadata((('token', '123'), ('user', 'ssx')))
return hello_pb2.HelloResponse(result=name)
```

Client

```python
try:
    # 向服务端传递header数据
    # 注意metadata必须都为字符串，否则请求失败
    result, call = stub.Hello.with_call(request, metadata=(('a', 1),))
    # 接收服务端传递的headers数据
    headers = call.trailing_metadata()
    print(dict(headers))
except Exception as e:
    print(e)
```



## 数据压缩和解压缩

server压缩，客户端会自动解压

```python
grpc_server = grpc.server(
        futures.ThreadPoolExecutor(max_workers=4),
        compression=grpc.Compression.Gzip
    )
```

client

```python
result, call = stub.Hello.with_call(request, metadata=(('a', 'b'),),  compression=grpc.Compression.Gzip)s
```



## 解决传输的数据量大小限制

```python
grpc_server = grpc.server(
        futures.ThreadPoolExecutor(max_workers=4),
        compression=grpc.Compression.Gzip,
        options=[
            ('grpc.max_send_message_length', 50 * 1024 * 1024),
            ('grpc.max_receive_message_length', 50 * 1024 * 1024)

        ]
    )
```



## service拦截器

```python

```



## 多进程启动python grpc服务

# gRPC Testing







# Flask



# 测试用例(unittest)

[官方文档](https://docs.python.org/3/library/unittest.html)

## 介绍

#### 测试fixture

测试fixture表示执行一个或多个测试需要的准备工作或者事后清除工作，例如连接数据库、目录或启动服务器进程。

#### 测试用例

测试用例是单独的测试单元。它检查特定输入集合的特定响应。unittest提供了一个基类TestCase，可以用来创建新的测试用例。

#### 测试套件

测试套件是测试用例、测试套件或两者的集合。它用于聚合应该一起执行的测试。

#### 测试运行器

测试运行程序是编排测试执行并向用户提供结果的组件。运行程序可以使用图形界面、文本界面或返回一个特殊值来指示执行测试的结果。

## 基本示例

unittest模块为构造和运行测试提供了一套丰富的工具。本节演示了一小部分工具就足以满足大多数用户的需求。

下面是一个用于测试三个字符串方法的简短脚本:

```python
import unittest

class TestStringMethods(unittest.TestCase):

    def test_upper(self):
        self.assertEqual('foo'.upper(), 'FOO')

    def test_isupper(self):
        self.assertTrue('FOO'.isupper())
        self.assertFalse('Foo'.isupper())

    def test_split(self):
        s = 'hello world'
        self.assertEqual(s.split(), ['hello', 'world'])
        # check that s.split fails when the separator is not a string
        with self.assertRaises(TypeError):
            s.split(2)

if __name__ == '__main__':
    unittest.main()
```

* 一个测试用例是通过子类unittest.TestCase来创建的
* 测试方法名称以字母test开头的方法定义， 此命名约定通知测试运行程序哪些方法表示测试
* 检查结果，以便测试运行程序可以累积所有测试结果并生成报告。
  * assertEqual()来检查预期的结果
  * assertFalse()来验证一个条件
  * assertRaises()来验证是否引发了特定的异常

* 最后一个块显示了一种运行测试的简单方法。unittest.main()为测试脚本提供了一个命令行接口

## 命令行接口

unittest模块可以从命令行使用来运行模块、类甚至单个测试方法的测试:

```shell
python -m unittest test_module1 test_module2 # 运行模块
python -m unittest test_module.TestClass  # 运行测试类
python -m unittest test_module.TestClass.test_method # 运行测试方法
```

* 您可以传入一个包含模块名、完全限定类名或方法名的任意组合的列表
* 测试模块也可以通过文件路径指定:python -m unittest test /test_something.py

你可以通过传递-v标志来运行更详细(更详细)的测试

## 测试发现

3.2新版功能。

Unittest支持简单的测试发现。为了与测试发现兼容，所有的测试文件都必须是可以从项目的顶级目录导入的模块或包(包括命名空间包)(这意味着它们的文件名必须是有效的标识符)。

测试发现在TestLoader.discover()中实现，但也可以从命令行使用。基本的命令行用法是:

```shell
cd project_directory
python -m unittest discover
```


说明作为快捷方式，python -m unittest等价于python -m unittest discover。如果要向测试发现传递参数，则必须显式地使用discover子命令。

## 组织测试代码

单元测试的基本构建块是测试用例——必须设置并检查正确性的单个场景。在unittest中，测试用例由unittest表示。TestCase实例。要制作自己的测试用例，必须编写TestCase的子类或使用FunctionTestCase。

TestCase实例的测试代码应该是完全自包含的，这样它可以独立运行，也可以与任何数量的其他测试用例任意组合运行。

最简单的TestCase子类将简单地实现一个测试方法(即一个名称以test开头的方法)，以便执行特定的测试代码:

```python
import unittest

class DefaultWidgetSizeTestCase(unittest.TestCase):
    def test_default_widget_size(self):
        widget = Widget('The widget')
        self.assertEqual(widget.size(), (50, 50))
```

测试可以是很多的，他们的设置可以是重复的。幸运的是，我们可以通过实现一个名为setUp()的方法来分解设置代码，测试框架会自动调用我们运行的每个测试:

```python
import unittest

class WidgetTestCase(unittest.TestCase):
    def setUp(self):
        self.widget = Widget('The widget')

    def test_default_widget_size(self):
        self.assertEqual(self.widget.size(), (50,50),
                         'incorrect default size')

    def test_widget_resize(self):
        self.widget.resize(100,150)
        self.assertEqual(self.widget.size(), (100,150),
                         'wrong size after resize')

```

> 注意，各种测试的运行顺序是根据字符串的内置顺序对测试方法名进行排序来确定的。
> 如果setUp()方法在测试运行时抛出异常，框架将认为测试发生了错误，测试方法将不会被执行。



类似地，我们可以提供一个tearDown()方法，在测试方法运行后进行整理:

如果setUp()成功，无论测试方法成功与否，都会运行tearDown()。

**注意：执行每个测试方法，都会运行一遍setUp()、tearDown()**



建议您使用TestCase实现根据测试的特性将测试分组在一起。unittest为此提供了一种机制:测试套件，由unittest的TestSuite类表示。在大多数情况下，调用unittest.main()会做正确的事情，并为你收集所有模块的测试用例并执行它们。

然而，如果你想自定义你的测试套件的构建，你可以自己做:

```python
def suite():
    suite = unittest.TestSuite()
    suite.addTest(WidgetTestCase('test_default_widget_size'))
    suite.addTest(WidgetTestCase('test_widget_resize'))
    return suite

if __name__ == '__main__':
    runner = unittest.TextTestRunner()
    runner.run(suite())
```

请将模块代码和测试代码分开，将测试代码放在单独的模块中有几个好处，比如test_widget.py:

* 测试模块可以从命令行独立运行。
* 测试代码可以更容易地从发布代码中分离出来
* 经过测试的代码可以更容易地进行重构。

## 跳过测试和预期失败

Unittest支持跳过单个测试方法，甚至跳过整个测试类。此外，它还支持将测试标记为“预期失败”，这是一个被破坏并将失败的测试，但不应该被算作测试结果上的失败。

跳过测试只需使用skip()装饰器或其条件变体之一，在setUp()或测试方法中调用TestCase.skipTest()，或直接引发SkipTest。

#### 基本示例:

```python
import unittest
import sys
from mymodel import Widget


class MyTestCase(unittest.TestCase):

    # 一定会跳过的测试
    @unittest.skip("demonstrating skipping")
    def test_nothing(self):
        self.fail("shouldn't happen")

    # 符合某种条件时跳过，如果条件为true，则返回第二字段，并跳过
    @unittest.skipIf(Widget.__version__ < (1, 3), "not supported in this library version")
    def test_format(self):
        # Tests that work for only a certain version of the library.
        print('test_formatxxx')

    # 不符合某种条件时跳过，如果符合执行 
    @unittest.skipUnless(sys.platform.startswith("win"), "requires Windows")
    def test_windows_support(self):
        # windows specific testing code
        print('test_windows_support')
    
    # 某种条件自行判断是否跳过
    def test_maybe_skipped(self):
        if not external_resource_available():
            self.skipTest("external resource not available")
        # test code that depends on the external resource
        pass

```

#### 跳过class

```python
@unittest.skip("showing class skipping")
class MySkippedTestCase(unittest.TestCase):
    def test_not_run(self):
        pass
```

TestCase.setUp()也可以跳过测试。当需要设置的资源不可用时，这是很有用的。

#### 预期失败使用expectedFailure()装饰器

```python
class ExpectedFailureTestCase(unittest.TestCase):
    @unittest.expectedFailure
    def test_fail(self):
        self.assertEqual(1, 0, "broken")
```

#### 通过创建一个decorator，在需要跳过测试时调用skip()，可以很容易地滚动您自己的跳过decorator。这个装饰器跳过测试，除非传递的对象有特定的属性:

```python
def skipUnlessHasattr(obj, attr):
    if hasattr(obj, attr):
        return lambda func: func
    return unittest.skip("{!r} doesn't have {!r}".format(obj, attr))
```

```以下装饰器和异常实现了跳过测试和预期的失败:
@unittest.skip(原因)
无条件地跳过修饰测试。理由应该描述为什么要跳过测试。

@unittest。skipIf(条件、原因)
如果condition为真，跳过修饰测试。

@unittest。skipUnless(条件、原因)
除非condition为真，否则跳过修饰测试。

@unittest.expectedFailure
将测试标记为预期失败或错误。如果测试失败或出错，则认为测试成功。如果测试通过，将被视为失败。

异常unittest.SkipTest(原因)
抛出此异常以跳过测试。

通常，你可以使用TestCase.skipTest()或其中一个跳过装饰器，而不是直接引发这个。

跳过的测试将不会运行setUp()或tearDown()。被跳过的类将不会运行setUpClass()或tearDownClass()。被跳过的模块将不会运行setUpModule()或tearDownModule()。

```

## 使用子测试区分测试迭代

当测试之间有非常小的差异时，例如一些参数，unittest允许您使用subTest()上下文管理器在测试方法体中区分它们。

例如，以下测试:

```python
class NumbersTest(unittest.TestCase):

    def test_even(self):
        """
        Test that numbers between 0 and 5 are all even.
        """
        for i in range(0, 6):
            with self.subTest(i=i):
                self.assertEqual(i % 2, 0)

```

如果不使用子测试，执行将在第一次失败后停止，错误将不太容易诊断，因为i的值不会显示:

## 类和函数

测试用例
类unittest.TestCase (methodName =“矮子”)
TestCase类的实例代表了unittest范围内的逻辑测试单元。该类被用作基类，具体的子类实现特定的测试。该类实现了测试运行程序所需的接口，以允许它驱动测试，以及测试代码可以用来检查和报告各种失败的方法。

TestCase的每个实例都将运行一个基本方法:名为methodName的方法。在大多数使用TestCase的情况下，您既不会更改methodName，也不会重新实现默认的runTest()方法。

在3.2版更改:TestCase可以在不提供methodName的情况下成功实例化。这使得在交互式解释器中试验TestCase更加容易。

TestCase实例提供了三组方法:一组用于运行测试，另一组用于测试实现检查条件并报告失败，还有一些查询方法允许收集关于测试本身的信息。

第一组中的方法(运行测试)是:

设置()
方法来准备测试夹具。这在调用test方法之前被立即调用;除了AssertionError或SkipTest之外，此方法引发的任何异常都将被视为错误，而不是测试失败。默认的实现不做任何事情。

tearDown ()
方法在测试方法被调用后立即调用，并记录结果。即使测试方法引发了异常，也会调用它，因此子类中的实现可能需要特别小心检查内部状态。除了AssertionError或SkipTest之外，由该方法引发的任何异常都将被视为附加错误，而不是测试失败(因此增加了报告的错误总数)。只有在setUp()成功时，才会调用这个方法，而不管测试方法的结果如何。默认的实现不做任何事情。

setUpClass ()
在运行单个类中的测试之前调用的类方法。setUpClass被调用时，类作为唯一的参数，并且必须装饰为一个classmethod():





# gunicorn

## 基本示例

创建myapp.py

```python
def app(environ, start_response):
    """Simplest possible application object"""
    data = b'Hello, World!\n'
    status = '200 OK'
    response_headers = [
        ('Content-type', 'text/plain'),
        ('Content-Length', str(len(data)))
    ]
    start_response(status, response_headers)
    return iter([data])
```

启动命令：gunicorn --workers=2 myapp:app

需要将app对象传递给gunicorn

## 命令行参数

-c :指定配置文件

-b：可指定主机和端口

-w：指定进程数，每个cpu核心指定2-4的worker比较合适

-k：要运行的工作进程的类型 ， sync, eventlet, gevent, tornado, gthread。同步是默认值。

--paste：可以指定一个ini配置文件启动




