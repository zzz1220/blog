---
title: elasticsearch使用
abbrlink: 5e47ac86
tags: Elasticsearch 
date: 2021-04-11 23:38:41
---

> 记录一次将千万级别的数据库表迁移到Elasticsearch 的过程

## 背景
我们有一张千万级别，并且每天都新增将近万条数据，更新也在十几万的业务表，这个业务表可以说是我们的核心业务，因为历史问题，表字段除了基本的字段外，还存了json， 一个字符串可代表多业务， 甚至存了二进制的对象， 同时，我们的运营页面需要根据将近20个查询条件对这个表进行查询，一方面因为数据量大，多个查询条件组合起来，查出数据有时也要十几秒，甚至在一些极端情况下，页面直接超时，用户体验很不好，另一方面，有些字段隐藏在json内部，或者二进制的对象里面，限制了其作为查询条件。

## 预研
于是我花了两天的时间了解了Elasticsearch 是什么，可以做什么，大概的操作步骤，查询api，以及配套的生态，又花了一天在测试环境搭建了一套elk，就开始了迁移的测试。
ps：在这里说明一下   
Elasticsearch是个开源分布式搜索引擎，它的特点有：分布式，零配置，自动发现，索引自动分片，索引副本机制，restful风格接口，多数据源，自动搜索负载等。
Logstash是一个完全开源的工具，他可以对你的日志进行收集、过滤，并将其存储供以后使用（如，搜索）。
Kibana 也是一个开源和免费的工具，它Kibana可以为 Logstash 和 ElasticSearch 提供的日志分析友好的 Web 界面，可以帮助您汇总、分析和搜索重要数据日志
elk当然就是这三部分的组合了。

## 用logstash 同步数据
logstash 本身的配置也不复杂，分为`input`，`filter`，`output` 三部分。
`input` 和 `output`支持各种插件。
比如`input` 支持 `mysql`，`file`，`http`，等等，最新的`input` 甚至支持`Twitter `。
`output` 支持 `elasticsearch`，`email`，`rabbitmq`等等。
查看所有支持的插件可以点击[这里](https://www.elastic.co/guide/en/logstash/7.12/introduction.html)


这一步本来是很简单的，因为logstash 本身就支持同步mysql数据到es，只需要简单的配置，加上json的中间filter，就可以完成了，但是就是二进制的数据对象要解析出来有点麻烦，还好logstash支持用ruby作为脚本，于是就需要编写一个ruby脚本，把从mysql里读出来的二进制转化成对象，我们存的二进制是protobuf转化的对象，因此，需要用一个protobuf的库 把二进制转成ruby对象，然后在解析这个对象，分割成不同的字段存入es。
首先，我从内部的平台，下载了原始的protobuf文件，然后在公司的编译机上，便成ruby的class文件，但是怎么从二进制解码到对象呢？我又翻看了下logstash的文档，发现他有一个codec的插件是直接支持解码pb对象的，但是这个插件只支持每次解析一行，但是我从mysql读出来的数据，是一行sql插叙结果，里面只有几个字段是二进制的，于是我找到了这个插件`logstash-codec-protobuf`的github[地址](https://github.com/logstash-plugins/logstash-codec-protobuf)，简单看了下源码和`api`，源码不多加上注释也才700多行，虽然没学过ruby但也大概看出来一些逻辑，看了下他的依赖，
```ruby
require 'logstash/codecs/base'
require 'logstash/util/charset'
require 'google/protobuf' # for protobuf3
require 'protocol_buffers' # https://github.com/codekitchen/ruby-protocol-buffers, for protobuf2
```
主要起作用的应该是
```ruby
require 'google/protobuf' # for protobuf3
require 'protocol_buffers
```

整个代码是一个面向对象的语法， 定义了一个class
```ruby
class LogStash::Codecs::Protobuf < LogStash::Codecs::Base
```
里面有`register`,`decode`,`encode` 等方法，看来起作用的就是`decode` 了，马上拿来放在脚本里面跑了一下，成功从二进制解析得到了ruby对象。

接下来就简单了，注意细节，对照字段开始同步。大概花了25个小时，就把所有的数据同步到了es,增量的数据，我用`updatetime`作为查询条件更新。

## 用php客户端查询数据

最后，我用php客户端，通过http调用的形式，使用es查询语法，对数据进行查询，果然快了很多，复杂的查询条件，也能在1s左右查询出来。

这里要说明的是`kibana`里面有个devtool的页面，可以在里面写查询语句验证结果，使用起来十分方便。

## 遇到的坑

主要是分页查询的时候，很大的数据量下不能以一般的查询语法，即from to 这种，因为在深度分页的情况下，这种使用方式效率是非常低的，比如from = 5000, size=10， es 需要在各个分片上匹配排序并得到5000*10条有效数据，然后在结果集中取最后10条数据返回。

我们就遇到了这个问题，先把`max_result_window `调大了一些，后来用深度分页代替：
es 提供了 `Scroll API` 的方式进行分页读取。原理上是对某次查询生成一个游标 scroll_id ， 后续的查询只需要根据这个游标去取数据，直到结果集中返回的 hits 字段为空，就表示遍历结束。scroll_id 的生成可以理解为建立了一个临时的历史快照，在此之后的增删改查等操作不会影响到这个快照的结果。
但是这个也不允许随机跳页，适合类似微博那种下拉翻页的场景。

ps: 关于深度分页优化可以看下[这里](https://juejin.cn/post/6850037275456339975)


## elk很复杂的
虽然基本符合了使用场景，但是关于elk我感觉还是只了解了皮毛，如何优化，怎么关联查询，怎么设置索引结构更高效，更复杂的查询语句，这些都还没接触到。
ps: 网上很多关于性能优化的文章，比如[这篇](https://zhuanlan.zhihu.com/p/67362440)