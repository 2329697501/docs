---
layout: article
language: 'de-de'
version: '4.0'
title: 'Phalcon\Cache\BackendInterface'
---
# Interface **Phalcon\Cache\BackendInterface**

[Source on GitHub](https://github.com/phalcon/cphalcon/tree/v{{ page.version }}.0/phalcon/cache/backendinterface.zep)

## Methods

abstract public **start** (*mixed* $keyName, [*mixed* $lifetime])

...

abstract public **stop** ([*mixed* $stopBuffer])

...

abstract public **getFrontend** ()

...

abstract public **getOptions** ()

...

abstract public **isFresh** ()

...

abstract public **isStarted** ()

...

abstract public **setLastKey** (*mixed* $lastKey)

...

abstract public **getLastKey** ()

...

abstract public **get** (*mixed* $keyName, [*mixed* $lifetime])

...

abstract public **save** ([*mixed* $keyName], [*mixed* $content], [*mixed* $lifetime], [*mixed* $stopBuffer])

...

abstract public **delete** (*mixed* $keyName)

...

abstract public **queryKeys** ([*mixed* $prefix])

...

abstract public **exists** ([*mixed* $keyName], [*mixed* $lifetime])

...