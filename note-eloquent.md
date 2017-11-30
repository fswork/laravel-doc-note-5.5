# Eloquent模型-注，既ORM方法索引

> 作者：[韩忠康](http://www.hellokang.net)
> 原文：[https://laravel.com/docs/5.5/eloquent](https://laravel.com/docs/5.5/eloquent)
> 中文（laravel-china翻译）：[https://d.laravel-china.org/docs/5.5/eloquent](https://d.laravel-china.org/docs/5.5/eloquent)
> Laravel版本：5.5
> 本文是Laravel文档中Eloquent ORM路由章节的解析，建议与文档对照阅读。
---

[TOC]

[# 概述](#概述)
[# 基础操作索引](#基础操作索引)
[# 模型关联](#模型关联)
[# 访问器&修改器](#访问器&修改器)
[# 序列化](#序列化)
[# 查询作用域](#查询作用域)
[# 事件](#事件)

## 概述

ORM，对象关系映射，使用OOP思想和语法操作关系型数据库的解决方案。Eloquent，就是Laravel使用的漂亮、简洁的 ActiveRecord 实现来和数据库进行交互的ORM。映射的关系为：

|关系|映射为|对象|
|----|:---:|----|
|表|映射为|模型类|
|记录|映射为|模型对象|
|字段|映射为|模型对象属性|
|关联|映射为|模型关联|


也就是说，如果需要操作表，则需要使用模型类操作，如果需要操作记录，则需要使用模型对象操作，如果需要操作字段，则通过模型对象属性完成，如果操作记录之间关联，则通过模型关联实现。以上就是典型的ORM，对象关系映射实现。

本文，总结Eloquent操作。

以模型开始的构建器查询，查询结果是模型对象或者模型对象的集合。对应的查询构建器，查询的结果是stdClass对象或者stdClass对象集合。

利用模型完成更新，可以利用模型完善功能，例如，时间戳，软删除，修改器，访问器，事件等。

## 基础操作索引

|操作|说明|详细|
|----|---|----|
|>php artisan make:model Model|定义模型命令|[详细](https://d.laravel-china.org/docs/5.5/eloquent#%E5%AE%9A%E4%B9%89%E6%A8%A1%E5%9E%8B)|
|**约定**|
|$model->table|指定表名，可选。默认ModelName对应table_name|[详细](https://d.laravel-china.org/docs/5.5/eloquent#eloquent-model-conventions)|
|$model->primaryKey|指定主键字段，可选。默认id|[详细](https://d.laravel-china.org/docs/5.5/eloquent#eloquent-model-conventions)|
|$model->incrementing|指定主键字段是否为自增，可选。默认true|[详细](https://d.laravel-china.org/docs/5.5/eloquent#eloquent-model-conventions)|
|$model->timestamps|指定是否维护created_at和updated_at时间戳字段，可选。默认true|[详细](https://d.laravel-china.org/docs/5.5/eloquent#eloquent-model-conventions)|
|$model->dateFormat|指定时间戳字段格式，可选|[详细](https://d.laravel-china.org/docs/5.5/eloquent#eloquent-model-conventions)|
|Model::CREATED_AT|指定创建时间字段，可选。默认为created_at|[详细](https://d.laravel-china.org/docs/5.5/eloquent#eloquent-model-conventions)|
|Model::UPDATED_AT|指定更新时间字段，可选。默认为updated_at|[详细](https://d.laravel-china.org/docs/5.5/eloquent#eloquent-model-conventions)|
|$model->connection|指定使用的数据库连接，可选。默认由配置文件定|[详细](https://d.laravel-china.org/docs/5.5/eloquent#eloquent-model-conventions)|
|**CRUD**|
|Model::all()|查询全部|[详细](https://d.laravel-china.org/docs/5.5/eloquent#retrieving-models)|
|Model::查询构造方法|每个Eloquent模型都可以当作一个查询构造器|[详细](https://d.laravel-china.org/docs/5.5/eloquent#retrieving-models)|
|Model::chunk()|分块结果查询，|[详细](https://d.laravel-china.org/docs/5.5/eloquent#retrieving-models)|
|Model::cursor()|游标获取，一条条的获取。构建器也有该方法|[详细](https://d.laravel-china.org/docs/5.5/eloquent#retrieving-models)|
|Model::find()|依据主键获取，支持主键数组和单个主键|[详细](https://d.laravel-china.org/docs/5.5/eloquent#retrieving-single-models)|
|Model::findOrFail()|利用主键或主键主组获取，失败抛出异常，未捕获，响应404|[详细](https://d.laravel-china.org/docs/5.5/eloquent#retrieving-single-models)|
|Model::firstOrFail()|获取第一条，失败抛出异常，未捕获，响应404，查询构建器也有该方法|[详细](https://d.laravel-china.org/docs/5.5/eloquent#retrieving-single-models)|
|(new Model())->save()|插入，实例化模型对象，save()存储|[详细](https://d.laravel-china.org/docs/5.5/eloquent#inserting-and-updating-models)|
|Model::find()->save()|更新，获取已存在模型对象，save()存储|[详细](https://d.laravel-china.org/docs/5.5/eloquent#inserting-and-updating-models)|
|Model::where()->update()|更新，条件更新|[详细](https://d.laravel-china.org/docs/5.5/eloquent#inserting-and-updating-models)|
|$model->fill()|批量填充，需要配置$fillable或$guarded|[详细](https://d.laravel-china.org/docs/5.5/eloquent#mass-assignment)|
|Model::create()|创建，创建模型对象，自动入库，参数为关联数组|[详细](https://d.laravel-china.org/docs/5.5/eloquent#inserting-and-updating-models)|
|Model::::firstOrCreate()|未找到，则创建模型对象，参数为关联数组|[详细](https://d.laravel-china.org/docs/5.5/eloquent#inserting-and-updating-models)|
|Model::::firstOrNew()|未找到，则实例化新的模型对象，参数为关联数组|[详细](https://d.laravel-china.org/docs/5.5/eloquent#inserting-and-updating-models)|
|Model::::updateOrCreate()|存在则更新，否则创建，参数为关联数组|[详细](https://d.laravel-china.org/docs/5.5/eloquent#inserting-and-updating-models)|
|$model->delete()|删除|[详细](https://d.laravel-china.org/docs/5.5/eloquent#deleting-models)|
|Model::destroy()|通过主键或主键数组删除|[详细](https://d.laravel-china.org/docs/5.5/eloquent#deleting-models)|
|Model {use SoftDeletes;}|支持软删除|[详细](https://d.laravel-china.org/docs/5.5/eloquent#deleting-models)|
|$model->trashed()|判断该模型是否被软删除|[详细](https://d.laravel-china.org/docs/5.5/eloquent#deleting-models)|
|Model::withTrashed()|查询时包含被软删除的，支持关联|[详细](https://d.laravel-china.org/docs/5.5/eloquent#deleting-models)|
|Model::onlyTrashed()|查询时仅包含被软删除的|[详细](https://d.laravel-china.org/docs/5.5/eloquent#deleting-models)|
|$model->restore()|恢复被软删除的，支持关联|[详细](https://d.laravel-china.org/docs/5.5/eloquent#deleting-models)|
|$model->forceDelete()|强制删除|[详细](https://d.laravel-china.org/docs/5.5/eloquent#deleting-models)|

## 模型关联

|操作|说明|详细|
|----|---|----|
|return $this->hasOne('Model', 'foreign_key', 'local_key')|1:1关联，逻辑上表示拥有一个|[详细](https://d.laravel-china.org/docs/5.5/eloquent-relationships#%E4%B8%80%E5%AF%B9%E4%B8%80)|
|return $this->hasMany('Model', 'foreign_key', 'local_key')|1:n关联，逻辑上表示拥有多个|[详细](https://d.laravel-china.org/docs/5.5/eloquent-relationships#%E4%B8%80%E5%AF%B9%E5%A4%9A)|
|return $this->belongsTo('Model', 'foreign_key', 'local_key')|1:n关联，逻辑上表示属于某个，用于1:1, 1:n的反向逻辑|[详细](https://d.laravel-china.org/docs/5.5/eloquent-relationships#%E4%B8%80%E5%AF%B9%E5%A4%9A%E5%8F%8D%E5%90%91)|
|return $this->belongsToMany('Model', $table, $foreignPivotKey, $relatedPivotKey, $parentKey, $relatedKey, $relation)|m:n关联，多对多关联。|[详细](https://d.laravel-china.org/docs/5.5/eloquent-relationships#%E5%A4%9A%E5%AF%B9%E5%A4%9A)|
|return $this->hasManyThrough($related, $through, $firstKey, $secondKey, $localKey, $secondLocalKey)|远层一对多。|[详细](https://d.laravel-china.org/docs/5.5/eloquent-relationships#%E8%BF%9C%E5%B1%82%E4%B8%80%E5%AF%B9%E5%A4%9A)|
|$model->pivot|多对多关联中间表获取|[详细](https://d.laravel-china.org/docs/5.5/eloquent-relationships#%E8%8E%B7%E5%BE%97%E4%B8%AD%E9%97%B4%E8%A1%A8%E5%AD%97%E6%AE%B5)|
|$model->relateAttribute|关联属性，用于获取关联数据|[详细](https://d.laravel-china.org/docs/5.5/eloquent-relationships#%E6%9F%A5%E8%AF%A2%E5%85%B3%E8%81%94)|
|$model->relateMethod()|关联方法，用于在关联上，完成其他业务逻辑|[详细](https://d.laravel-china.org/docs/5.5/eloquent-relationships#%E6%9F%A5%E8%AF%A2%E5%85%B3%E8%81%94)|
|Model::has()|存在关联查询|[详细](https://d.laravel-china.org/docs/5.5/eloquent-relationships#%E5%9F%BA%E4%BA%8E%E5%AD%98%E5%9C%A8%E7%9A%84%E5%85%B3%E8%81%94%E6%9F%A5%E8%AF%A2)|
|Model::doesntHave()|不存在关联查询|[详细](https://d.laravel-china.org/docs/5.5/eloquent-relationships#%E5%9F%BA%E4%BA%8E%E5%AD%98%E5%9C%A8%E7%9A%84%E5%85%B3%E8%81%94%E6%9F%A5%E8%AF%A2)|
|Model::whereHas()|条件存在关联查询|[详细](https://d.laravel-china.org/docs/5.5/eloquent-relationships#%E5%9F%BA%E4%BA%8E%E5%AD%98%E5%9C%A8%E7%9A%84%E5%85%B3%E8%81%94%E6%9F%A5%E8%AF%A2)|
|Model::whereDoesntHave()|条件不存在关联查询|[详细](https://d.laravel-china.org/docs/5.5/eloquent-relationships#%E5%9F%BA%E4%BA%8E%E5%AD%98%E5%9C%A8%E7%9A%84%E5%85%B3%E8%81%94%E6%9F%A5%E8%AF%A2)|
|Model::withCount()|统计关联查询，支持统计一个或多个关联，支持过滤|[详细](https://d.laravel-china.org/docs/5.5/eloquent-relationships#%E5%85%B3%E8%81%94%E6%95%B0%E6%8D%AE%E8%AE%A1%E6%95%B0)|
|Model::with()|预加载关联查询，支持一个或多个，支持设置过滤|[详细](https://d.laravel-china.org/docs/5.5/eloquent-relationships#%E9%A2%84%E5%8A%A0%E8%BD%BD)|
|$collection->load()|延迟预加载关联查询，支持一个或多个，支持设置过滤|[详细](hhttps://d.laravel-china.org/docs/5.5/eloquent-relationships#%E5%BB%B6%E8%BF%9F%E9%A2%84%E5%8A%A0%E8%BD%BD)|
|$model->relateMethod()->save($relateModel)|关联存储，多对多关联时支持设置中间表，拥有关联更新|[详细](https://d.laravel-china.org/docs/5.5/eloquent-relationships#save-%E6%96%B9%E6%B3%95)|
|$model->relateMethod()->saveMany([$relateModel, ...])|关联存储,多个，拥有关联更新|[详细](https://d.laravel-china.org/docs/5.5/eloquent-relationships#save-%E6%96%B9%E6%B3%95)|
|$model->relateMethod()->create([assocArray])|关联存储，支持关联数组，拥有关联更新|[详细](https://d.laravel-china.org/docs/5.5/eloquent-relationships#create-%E6%96%B9%E6%B3%95)|
|$model->relateMethod()->createMany([[assocArray], ...])|关联存储，支持关联数组，拥有关联更新|[详细](https://d.laravel-china.org/docs/5.5/eloquent-relationships#create-%E6%96%B9%E6%B3%95)|
|$model->relateMethod()->associate($relateModel)|关联存储，属于关联更新|[详细](https://d.laravel-china.org/docs/5.5/eloquent-relationships#%E6%9B%B4%E6%96%B0-belongsTo-%E5%85%B3%E8%81%94)|
|$model->relateMethod()->dissociate()|取消关联，属于关联更新|[详细](https://d.laravel-china.org/docs/5.5/eloquent-relationships#%E6%9B%B4%E6%96%B0-belongsTo-%E5%85%B3%E8%81%94)|
|$model->relateMethod()->attach(relate-id)|附加关联，支持数组，支持设置关联字段，多对多关联更新|[详细](https://d.laravel-china.org/docs/5.5/eloquent-relationships#%E5%A4%9A%E5%AF%B9%E5%A4%9A%E5%85%B3%E8%81%94)|
|$model->relateMethod()->detach(relate-id)|解除关联，支持数组，多对多关联更新|[详细](https://d.laravel-china.org/docs/5.5/eloquent-relationships#%E5%A4%9A%E5%AF%B9%E5%A4%9A%E5%85%B3%E8%81%94)|
|$model->relateMethod()->sync()|同步关联，支持数组，支持设置其他字段，多对多关联更新|[详细](https://d.laravel-china.org/docs/5.5/eloquent-relationships#%E5%A4%9A%E5%AF%B9%E5%A4%9A%E5%85%B3%E8%81%94)|
|$model->relateMethod()->sync()|同步关联，排除移除支持，多对多关联更新|[详细](https://d.laravel-china.org/docs/5.5/eloquent-relationships#%E5%A4%9A%E5%AF%B9%E5%A4%9A%E5%85%B3%E8%81%94)|
|$model->relateMethod()->updateExistingPivot()|更新已存在关联中间表属性，多对多关联更新|[详细](https://d.laravel-china.org/docs/5.5/eloquent-relationships#%E5%A4%9A%E5%AF%B9%E5%A4%9A%E5%85%B3%E8%81%94)|
|$model->touches|更新属于关系的关联模型时间戳，属于关联更新|[详细](https://d.laravel-china.org/docs/5.5/eloquent-relationships#%E6%9B%B4%E6%96%B0%E7%88%B6%E7%BA%A7%E6%97%B6%E9%97%B4%E6%88%B3)|



## 操作表示说明
以>开头，表示命令
$model->property，表示属性
$model->method()，表示对象方法
Model::$property，表示静态属性
Model::method()，表示静态方法
Model::const，表示常量
详细引用laravel-china。如有侵权，请告知。

## 访问器&修改器
未完待续...

## 序列化
未完待续...

## 查询作用域
未完待续...

## 事件
未完待续...

## 结语
ORM会不会慢？执行肯定比SQL构建要慢。但是开发效率和语法语义要有优势。