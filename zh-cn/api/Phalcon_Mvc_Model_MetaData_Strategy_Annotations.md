---
layout: article
language: 'zh-cn'
version: '4.0'
title: 'Phalcon\Mvc\Model\MetaData\Strategy\Annotations'
---
# Class **Phalcon\Mvc\Model\MetaData\Strategy\Annotations**

*implements* [Phalcon\Mvc\Model\MetaData\StrategyInterface](Phalcon_Mvc_Model_MetaData_StrategyInterface)

[源码在GitHub](https://github.com/phalcon/cphalcon/tree/v{{ page.version }}.0/phalcon/mvc/model/metadata/strategy/annotations.zep)

## 方法

final public **getMetaData** ([Phalcon\Mvc\ModelInterface](Phalcon_Mvc_ModelInterface) $model, [Phalcon\DiInterface](Phalcon_DiInterface) $dependencyInjector)

The meta-data is obtained by reading the column descriptions from the database information schema

final public **getColumnMaps** ([Phalcon\Mvc\ModelInterface](Phalcon_Mvc_ModelInterface) $model, [Phalcon\DiInterface](Phalcon_DiInterface) $dependencyInjector)

Read the model's column map, this can't be inferred