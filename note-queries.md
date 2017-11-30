# SQL构造器-注，既SQL构建方法索引

> 作者：[韩忠康](http://hellokang.net)
> 原文：[https://laravel.com/docs/5.5/queries](https://laravel.com/docs/5.5/queries)
> 中文（laravel-china翻译）：[https://d.laravel-china.org/docs/5.5/queries](https://d.laravel-china.org/docs/5.5/queries)
> Laravel版本：5.5
> 本文是Laravel文档中查询构建器章节的整理，建议与文档对照阅读。
---

[TOC]

[# 概述](#概述)
[# SQL构建方法索引](#SQL构建方法索引)
[# 结语](#结语)


## 概述
框架执行数据库操作，有 SQL构建 和 ORM 两种方式。其中SQL构建更直接些，而ORM更OOP一些。本文对SQL构建中用到的操作进行总结。
本文的主要总结，是SQL构建的语法。

## SQL构建方法索引

|方法|作用|详细|
|----|----||
|**通用**|
|table()|指定表名|[详细](#table)|
|from()|from子句|[详细](#from)|
|DB::raw()|原生字符串|[详细](https://d.laravel-china.org/docs/5.5/queries#raw-expressions)|
|**插入**|
|insert()|插入单条或多条记录|[详细](https://d.laravel-china.org/docs/5.5/queries#Inserts)|
|insertGetId()|插入后返回自增ID|[详细](https://d.laravel-china.org/docs/5.5/queries#Inserts)|
|**更新**|
|update()|更新，设置类更新|[详细](https://d.laravel-china.org/docs/5.5/queries#Updates)|
|increment()|递增更新|[详细](https://d.laravel-china.org/docs/5.5/queries#%E8%87%AA%E5%A2%9E%E6%88%96%E8%87%AA%E5%87%8F)|
|decrement()|递减更新|[详细](https://d.laravel-china.org/docs/5.5/queries#%E8%87%AA%E5%A2%9E%E6%88%96%E8%87%AA%E5%87%8F)|
|**删除**|
|delete()|删除记录|[详细](https://d.laravel-china.org/docs/5.5/queries#Deletes)|
|truncate()|清空|[详细](https://d.laravel-china.org/docs/5.5/queries#Deletes)|
|**查询**|
|select()|设置查询字段|[详细](https://d.laravel-china.org/docs/5.5/queries#%E6%8C%87%E5%AE%9A%E4%B8%80%E4%B8%AA-Select-%E5%AD%90%E5%8F%A5)|
|addSelect()|添加查询字段|[详细](https://d.laravel-china.org/docs/5.5/queries#%E6%8C%87%E5%AE%9A%E4%B8%80%E4%B8%AA-Select-%E5%AD%90%E5%8F%A5)|
|join()|连接查询，为inner join|[详细](https://d.laravel-china.org/docs/5.5/queries#joins)|
|leftJoin()|左外连接|[详细](https://d.laravel-china.org/docs/5.5/queries#joins)|
|rightJoin()|右外连接|[详细](https://d.laravel-china.org/docs/5.5/queries#joins)|
|crossJoin()|交叉连接|[详细](https://d.laravel-china.org/docs/5.5/queries#joins)|
|groupBy()|分组|[详细](https://d.laravel-china.org/docs/5.5/queries#Ordering-Grouping-Limit-%E5%8F%8A-Offset)|
|groupBy()|分组|[详细](https://d.laravel-china.org/docs/5.5/queries#Ordering-Grouping-Limit-%E5%8F%8A-Offset)|
|having()|分组后过滤|[详细](#having)|
|groupBy()|分组|[详细](https://d.laravel-china.org/docs/5.5/queries#Ordering-Grouping-Limit-%E5%8F%8A-Offset)|
|havingRaw()|原始字符串having|[详细](#whereRaw)|
|orderBy()|排序|[详细](https://d.laravel-china.org/docs/5.5/queries#Ordering-Grouping-Limit-%E5%8F%8A-Offset)|
|latest()|最新|[详细](https://d.laravel-china.org/docs/5.5/queries#Ordering-Grouping-Limit-%E5%8F%8A-Offset)|
|oldest()|最旧|[详细](https://d.laravel-china.org/docs/5.5/queries#Ordering-Grouping-Limit-%E5%8F%8A-Offset)|
|inRandomOrder()|随机排序|[详细](https://d.laravel-china.org/docs/5.5/queries#Ordering-Grouping-Limit-%E5%8F%8A-Offset)|
|skip()->take()|限定记录|[详细](https://d.laravel-china.org/docs/5.5/queries#Ordering-Grouping-Limit-%E5%8F%8A-Offset)|
|offset()->limit()|限定记录，同上|[详细](https://d.laravel-china.org/docs/5.5/queries#Ordering-Grouping-Limit-%E5%8F%8A-Offset)|
|get()|获取多条|[详细](https://d.laravel-china.org/docs/5.5/queries#%E4%BB%8E%E6%95%B0%E6%8D%AE%E8%A1%A8%E4%B8%AD%E8%8E%B7%E5%8F%96%E6%89%80%E6%9C%89%E7%9A%84%E6%95%B0%E6%8D%AE%E5%88%97)|
|first()|查询单条|[详细](https://d.laravel-china.org/docs/5.5/queries#%E4%BB%8E%E6%95%B0%E6%8D%AE%E8%A1%A8%E4%B8%AD%E8%8E%B7%E5%8F%96%E5%8D%95%E4%B8%AA%E5%88%97%E6%88%96%E8%A1%8C)|
|value()|查询标量值|[详细](https://d.laravel-china.org/docs/5.5/queries#%E4%BB%8E%E6%95%B0%E6%8D%AE%E8%A1%A8%E4%B8%AD%E8%8E%B7%E5%8F%96%E5%8D%95%E4%B8%AA%E5%88%97%E6%88%96%E8%A1%8C)|
|find()|利用主键查询一条|[详细]()|
|pluck()|查询列|[详细](https://d.laravel-china.org/docs/5.5/queries#%E8%8E%B7%E5%8F%96%E4%B8%80%E5%88%97%E7%9A%84%E5%80%BC)|
|count()|记录数|[详细](https://d.laravel-china.org/docs/5.5/queries#aggregates)|
|max()|最大值|[详细](https://d.laravel-china.org/docs/5.5/queries#aggregates)|
|min()|最小值|[详细](https://d.laravel-china.org/docs/5.5/queries#aggregates)|
|avg()|平均值|[详细](https://d.laravel-china.org/docs/5.5/queries#aggregates)|
|sum()|求和|[详细](https://d.laravel-china.org/docs/5.5/queries#aggregates)|
|union()|联合|[详细](https://d.laravel-china.org/docs/5.5/queries#unions)|
|unionAll()|全部联合|[详细](https://d.laravel-china.org/docs/5.5/queries#unions)|
|distinct()|去重|[详细](#distinct)|
|sharedLock()|共享悲观锁|[详细](https://d.laravel-china.org/docs/5.5/queries#pessimistic-locking)|
|lockForUpdate()|独占悲观锁|[详细](https://d.laravel-china.org/docs/5.5/queries#pessimistic-locking)|
|paginate()|分页|[详细](https://d.laravel-china.org/docs/5.5/pagination#%E5%AF%B9%E6%9F%A5%E8%AF%A2%E8%AF%AD%E5%8F%A5%E6%9E%84%E9%80%A0%E5%99%A8%E8%BF%9B%E8%A1%8C%E5%88%86%E9%A1%B5)|
|simplePaginate()|简单分页|[详细](https://d.laravel-china.org/docs/5.5/pagination#%22%E7%AE%80%E5%8D%95%E5%88%86%E9%A1%B5%22)|
|**条件**|
|where()|条件设置|[详细](https://d.laravel-china.org/docs/5.5/queries#where-clauses)|
|orWhere()|OR逻辑条件|[详细](https://d.laravel-china.org/docs/5.5/queries#where-clauses)|
|where(function(){})|复杂逻辑条件|[详细](https://d.laravel-china.org/docs/5.5/queries#where-clauses)|
|whereBetween()|between and 条件|[详细](https://d.laravel-china.org/docs/5.5/queries#%E5%85%B6%E5%AE%83-Where-%E5%AD%90%E5%8F%A5)|
|whereNotBetween()|not between and 条件|[详细](https://d.laravel-china.org/docs/5.5/queries#%E5%85%B6%E5%AE%83-Where-%E5%AD%90%E5%8F%A5)|
|whereIn()|in 条件|[详细](https://d.laravel-china.org/docs/5.5/queries#%E5%85%B6%E5%AE%83-Where-%E5%AD%90%E5%8F%A5)|
|whereNotIn()|not in 条件|[详细](https://d.laravel-china.org/docs/5.5/queries#%E5%85%B6%E5%AE%83-Where-%E5%AD%90%E5%8F%A5)|
|whereNull()|is null 条件|[详细](https://d.laravel-china.org/docs/5.5/queries#%E5%85%B6%E5%AE%83-Where-%E5%AD%90%E5%8F%A5)|
|whereNotNull()|is not null 条件|[详细](https://d.laravel-china.org/docs/5.5/queries#%E5%85%B6%E5%AE%83-Where-%E5%AD%90%E5%8F%A5)|
|whereDate()|年月日比较|[详细](https://d.laravel-china.org/docs/5.5/queries#%E5%85%B6%E5%AE%83-Where-%E5%AD%90%E5%8F%A5)|
|whereDay()|日比较|[详细](https://d.laravel-china.org/docs/5.5/queries#%E5%85%B6%E5%AE%83-Where-%E5%AD%90%E5%8F%A5)|
|whereMonth()|月比较|[详细](https://d.laravel-china.org/docs/5.5/queries#%E5%85%B6%E5%AE%83-Where-%E5%AD%90%E5%8F%A5)|
|whereYear()|年比较|[详细](https://d.laravel-china.org/docs/5.5/queries#%E5%85%B6%E5%AE%83-Where-%E5%AD%90%E5%8F%A5)|
|whereColumn()|比较两个字段|[详细](https://d.laravel-china.org/docs/5.5/queries#%E5%85%B6%E5%AE%83-Where-%E5%AD%90%E5%8F%A5)|
|whereExists()|exists条件|[详细](https://d.laravel-china.org/docs/5.5/queries#Where-Exists-%E8%AF%AD%E6%B3%95)|
|whereRaw()|原生字符串where|[详细](https://d.laravel-china.org/docs/5.5/queries#%E5%8E%9F%E5%A7%8B%E8%A1%A8%E8%BE%BE%E5%BC%8F)|
|**直接执行SQL**|
|DB::select()|直接执行select语句|[详细](https://d.laravel-china.org/docs/5.5/database#%E8%BF%90%E8%A1%8C-Select)|
|DB::update()|直接执行update语句|[详细](https://d.laravel-china.org/docs/5.5/database#%E8%BF%90%E8%A1%8C-Update)|
|DB::delete()|直接执行delete语句|[详细](https://d.laravel-china.org/docs/5.5/database#%E8%BF%90%E8%A1%8C-Delete)|
|DB::insert()|直接执行insert语句|[详细](https://d.laravel-china.org/docs/5.5/database#%E8%BF%90%E8%A1%8C-Insert)|
|DB::statement()|直接执行非CRUD语句|[详细](https://d.laravel-china.org/docs/5.5/database#%E8%BF%90%E8%A1%8C%E4%B8%80%E8%88%AC%E5%A3%B0%E6%98%8E)|
|**事务支持**|
|DB::transaction(function () {})|事务处理|[详细](https://d.laravel-china.org/docs/5.5/database#%E6%95%B0%E6%8D%AE%E5%BA%93%E4%BA%8B%E5%8A%A1)|
|DB::beginTransaction()|开启事务|[详细](https://d.laravel-china.org/docs/5.5/database#%E6%89%8B%E5%8A%A8%E6%93%8D%E4%BD%9C%E4%BA%8B%E5%8A%A1)|
|DB::rollBack()|回滚|[详细](https://d.laravel-china.org/docs/5.5/database#%E6%89%8B%E5%8A%A8%E6%93%8D%E4%BD%9C%E4%BA%8B%E5%8A%A1)|
|DB::commit()|提交|[详细](https://d.laravel-china.org/docs/5.5/database#%E6%89%8B%E5%8A%A8%E6%93%8D%E4%BD%9C%E4%BA%8B%E5%8A%A1)|

详细引用的是laravel-china翻译文档，点击可以查看详请。章节划分有所不同，上下翻找即可。

## 结语
SQL构建，就是将SQL由方法拼凑出来，便于维护更新。laravel提供了大量的方法来使用，尤其是条件拼凑处。这样的目的是多方法，少参数。从方法名既可以判断出我们要做什么。像写文章一样写代码，这就是优雅的代码？反问ing。