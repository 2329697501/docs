---
layout: article
language: 'nl-nl'
version: '4.0'
title: 'Phalcon\Annotations\ReaderInterface'
---
# Interface **Phalcon\Annotations\ReaderInterface**

[Broncode op GitHub](https://github.com/phalcon/cphalcon/tree/v{{ page.version }}.0/phalcon/annotations/readerinterface.zep)

## Methoden

abstract public **parse** (*mixed* $className)

Reads annotations from the class dockblocks, its methods and/or properties

abstract public static **parseDocBlock** (*mixed* $docBlock, [*mixed* $file], [*mixed* $line])

Parses a raw doc block returning the annotations found