---
layout: article
language: 'nl-nl'
version: '4.0'
title: 'Phalcon\Annotations\Reader'
---
# Class **Phalcon\Annotations\Reader**

*implements* [Phalcon\Annotations\ReaderInterface](Phalcon_Annotations_ReaderInterface)

[Broncode op GitHub](https://github.com/phalcon/cphalcon/tree/v{{ page.version }}.0/phalcon/annotations/reader.zep)

Parses docblocks returning an array with the found annotations

## Methoden

public **parse** (*mixed* $className)

Reads annotations from the class dockblocks, its methods and/or properties

public static **parseDocBlock** (*mixed* $docBlock, [*mixed* $file], [*mixed* $line])

Parses a raw doc block returning the annotations found