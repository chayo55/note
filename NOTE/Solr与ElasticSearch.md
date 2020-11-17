## Lucene
Lucene是apache软件基金会4 jakarta项目组的一个子项目，是一个开放源代码的**全文检索引擎工具包**。Lucene不是一个完整的全文检索引擎，是一个**全文检索引擎的架构**。

## Solr
Solr是一个基于Lucene的Java搜索引擎服务器。

## Elasticsearch
Elasticsearch是一个基于Apache Lucene(TM)的开源搜索引擎。

## Solr与Elasticsearch优缺点对比
Solr 利用 Zookeeper 进行分布式管理，而 Elasticsearch 自身带有分布式协调管理功能;

Solr 支持更多格式的数据，而 Elasticsearch 仅支持json文件格式；

Solr 官方提供的功能更多，而 Elasticsearch 本身更注重于核心功能，高级功能多有第三方插件提供；

Solr 在传统的搜索应用中表现好于 Elasticsearch，但在处理实时搜索应用时效率明显低于 Elasticsearch。

Solr 是传统搜索应用的有力解决方案，但 Elasticsearch 更适用于新兴的实时搜索应用。

