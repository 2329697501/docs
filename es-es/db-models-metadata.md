---
layout: article
language: 'es-es'
version: '4.0'
---
**Este artículo refleja la v3.4 y todavía no ha sido revisado**

<a name='models-metadata'></a>

# Metadatos de modelos

Para acelerar el desarrollo, [Phalcon\Mvc\Model](api/Phalcon_Mvc_Model) ayuda a consultar campos y restricciones de las tablas relacionadas con los modelos. Para lograr esto, está disponible [Phalcon\Mvc\Model\MetaData](api/Phalcon_Mvc_Model_MetaData) para administrar y almacenar en caché los metadatos de las tablas.

Muchas veces es necesario obtener estos atributos cuando trabajamos con modelos. Puede obtener una instancia de metadatos, de la siguiente forma:

```php
<?php

$robot = new Robots();

// Obtener instancia de Phalcon\Mvc\Model\Metadata
$metadata = $robot->getModelsMetaData();

// Obtener nombres de los campos de los robots
$attributes = $metadata->getAttributes($robot);
print_r($attributes);

// Obtener tipos de datos de los campos de los robots
$dataTypes = $metadata->getDataTypes($robot);
print_r($dataTypes);
```

<a name='caching-metadata'></a>

## Almacenamiento en caché de metadatos

Una vez que la aplicación está en una etapa de producción, no es necesario consultar los metadatos de la tabla del sistema de base de datos cada vez que use la tabla. Esto podría hacerse almacenando en caché los metadatos, utilizando cualquiera de los siguientes adaptadores:

| Adaptador    | Descripción                                                                                                                                                                                                                                                                                                                                                                 | API                                                                                        |
| ------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| Apc          | Este adaptador utiliza [Alternative PHP Cache (APC)](https://secure.php.net/manual/en/book.apc.php) para almacenar los metadatos de tabla. Puede especificar la duración de los metadatos con opciones. (Recomendado para producción).                                                                                                                                      | [Phalcon\Mvc\Model\MetaData\Apc](api/Phalcon_Mvc_Model_MetaData_Apc)                   |
| Files        | Este adaptador utiliza archivos planos para almacenar los metadatos. Este adaptador reduce las consultas a la base de datos, pero puede incrementar el acceso al sistema de archivos.                                                                                                                                                                                       | [Phalcon\Mvc\Model\MetaData\Files](api/Phalcon_Mvc_Model_MetaData_Files)               |
| Libmemcached | Este adaptador utiliza el [Servidor Memcached](https://www.memcached.org/) para almacenar los metadatos de la tabla. Los parámetros de servidor así como la duración de la caché se especifica en las opciones. (Recomendado para producción)                                                                                                                               | [Phalcon\Mvc\Model\MetaData\Libmemcached](api/Phalcon_Mvc_Model_MetaData_Libmemcached) |
| Memcache     | Este adaptador utiliza [Memcache](https://php.net/manual/en/book.memcache.php) para almacenar los metadatos de la tabla. Puede especificar la duración de los metadatos con opciones. (Recomendado para producción)                                                                                                                                                         | `Phalcon\Mvc\Model\MetaData\Memcache`                                                  |
| Memory       | Este adaptador es el predeterminado. Se almacena en caché los metadatos sólo durante la solicitud. Cuando se haya completado la solicitud, los metadatos son liberados como parte de la memoria normal de la solicitud. (Recomendado para el desarrollo)                                                                                                                    | [Phalcon\Mvc\Model\MetaData\Memory](api/Phalcon_Mvc_Model_MetaData_Memory)             |
| Redis        | Este adaptador utiliza [Redis](https://redis.io/) para almacenar los metadatos de la tabla. Los parámetros de servidor así como la duración de la caché se especifica en las opciones. (Recomendado para producción).                                                                                                                                                       | [Phalcon\Mvc\Model\MetaData\Redis](api/Phalcon_Mvc_Model_MetaData_Redis)               |
| Session      | Este adaptador almacena metadatos en la variable global `$_SESSION`. Este adaptador sólo se recomienda cuando la aplicación está utilizando realmente un pequeño número de modelos. Los metadatos se actualizan cada vez que inicie una nueva sesión. Esto también requiere el uso de `session_start()` para iniciar la sesión antes de utilizar cualquiera de los modelos. | [Phalcon\Mvc\Model\MetaData\Session](api/Phalcon_Mvc_Model_MetaData_Session)           |
| XCache       | Este adaptador utiliza [XCache](https://xcache.lighttpd.net/) para almacenar los metadatos de la tabla. Puede especificar la duración de los metadatos con opciones. Esta es una de las maneras recomendadas para almacenar metadatos cuando la aplicación está en producción.                                                                                              | [Phalcon\Mvc\Model\MetaData\Xcache](api/Phalcon_Mvc_Model_MetaData_Xcache)             |

Como otras dependencias del ORM, se solicita el administrador de metadatos al contenedor de servicios:

```php
<?php

use Phalcon\Mvc\Model\MetaData\Apc as ApcMetaData;

$di['modelsMetadata'] = function () {
    // Creamos un gestor de metadatos con APC
    $metadata = new ApcMetaData(
        [
            'lifetime' => 86400,
            'prefix'   => 'my-prefix',
        ]
    );

    return $metadata;
};
```

<a name='metadata-strategies'></a>

## Estrategias de metadatos

Como se mencionó anteriormente la estrategia por defecto para obtener metadatos del modelo es introspección de la base de datos. En esta estrategia, el esquema de información se utiliza para conocer los campos de una tabla, su clave primaria, campos que aceptan valores `NULL`, los tipos de datos, etcétera.

Se puede cambiar la introspección de metadatos por defecto de la siguiente manera:

```php
<?php

use Phalcon\Mvc\Model\MetaData\Apc as ApcMetaData;

$di['modelsMetadata'] = function () {
    // Instanciar un adaptador de metadata
    $metadata = new ApcMetaData(
        [
            'lifetime' => 86400,
            'prefix'   => 'my-prefix',
        ]
    );

    // Configurar una estratégia personalizada de instrospección
    $metadata->setStrategy(
        new MyIntrospectionStrategy()
    );

    return $metadata;
};
```

<a name='strategies-database-introspection'></a>

### Estrategia de introspección de la base de datos

Esta estrategia no requiere ninguna personalización y es utilizada implícitamente por todos los adaptadores de metadatos.

<a name='strategies-annotations'></a>

### Estrategia de anotaciones

Esta estrategia hace uso de anotaciones `<anotations>` para describir las columnas en un modelo:

```php
<?php

use Phalcon\Mvc\Model;

class Robots extends Model
{
    /**
     * @Primary
     * @Identity
     * @Column(type='integer', nullable=false)
     */
    public $id;

    /**
     * @Column(type='string', length=70, nullable=false)
     */
    public $name;

    /**
     * @Column(type='string', length=32, nullable=false)
     */
    public $type;

    /**
     * @Column(type='integer', nullable=false)
     */
    public $year;
}
```

Las anotaciones deben colocarse en las propiedades que se asignan a columnas en la fuente asignada. Las propiedades sin la anotación `@Column` se tratan como atributos simples de la clase.

Son soportadas las siguientes anotaciones:

| Nombre   | Descripción                                                 |
| -------- | ----------------------------------------------------------- |
| Primary  | Marcar el campo como parte de la clave primaria de la tabla |
| Identity | El campo es una columna auto_increment/serial               |
| Column   | Esto marca un atributo como una columna mapeada             |

La anotación `@Column` admite los siguientes parámetros:

| Nombre               | Descripción                                                                                                                                                                      |
| -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| column               | Nombre real de la columna                                                                                                                                                        |
| type                 | Tipos de columnas: varchar/string (por defecto), text, char, json, tinyblob, blob, mediumblob, longblob, integer, biginteger, float, decimal, date, datetime, timestamp, boolean |
| length               | Longitud de la columna, si lo hubiere                                                                                                                                            |
| nullable             | Si la columna acepta valores null o no                                                                                                                                           |
| skip_on_insert     | Omitir esta columna al insertar                                                                                                                                                  |
| skip_on_update     | Omitir esta columna al actualizar                                                                                                                                                |
| allow_empty_string | Esta columna permite cadenas vacías                                                                                                                                              |
| default              | Valor por defecto                                                                                                                                                                |

La estrategia de anotaciones podría ser configurada de esta manera:

```php
<?php

use Phalcon\Mvc\Model\MetaData\Apc as ApcMetaData;
use Phalcon\Mvc\Model\MetaData\Strategy\Annotations as StrategyAnnotations;

$di['modelsMetadata'] = function () {
    // Instancia del adaptador de metadata 
    $metadata = new ApcMetaData(
        [
            'lifetime' => 86400,
            'prefix'   => 'my-prefix',
        ]
    );

    // Configurar un introspección personalizada para la base de datos
    $metadata->setStrategy(
        new StrategyAnnotations()
    );

    return $metadata;
};
```

<a name='strategies-manual'></a>

## Metadata manual

Usando las estrategias de introspección presentadas anteriormente, Phalcon puede obtener los metadatos para cada modelo automáticamente sin que el desarrollador tenga que configurarlos manualmente.

Sin embargo, el desarrollador también tiene la opción de definir los metadatos manualmente. Esta estrategia sobrescribe cualquier estrategia establecida en el gestor de metadatos. Las nuevas columnas añadidas/modificadas/eliminadas de/hacia la tabla asignada deben agregarse/modificarse/eliminarse también para que todo funcione correctamente.

En el ejemplo siguiente se muestra cómo definir los metadatos manualmente:

```php
<?php

use Phalcon\Mvc\Model;
use Phalcon\Db\Column;
use Phalcon\Mvc\Model\MetaData;

class Robots extends Model
{
    public function metaData()
    {
        return array(
            // Cada columna en la tabla asignada
            MetaData::MODELS_ATTRIBUTES => [
                'id',
                'name',
                'type',
                'year',
            ],

            // Cada columna que es parte de la clave primaria
            MetaData::MODELS_PRIMARY_KEY => [
                'id',
            ],

            // Columnas que no son parte de la clave primaria
            MetaData::MODELS_NON_PRIMARY_KEY => [
                'name',
                'type',
                'year',
            ],

            // Columnas que no permiten valores nulos
            MetaData::MODELS_NOT_NULL => [
                'id',
                'name',
                'type',
            ],

            // Cada columna con su tipo de datos
            MetaData::MODELS_DATA_TYPES => [
                'id'   => Column::TYPE_INTEGER,
                'name' => Column::TYPE_VARCHAR,
                'type' => Column::TYPE_VARCHAR,
                'year' => Column::TYPE_INTEGER,
            ],

            // Columnas que tienen tipos de datos numéricos
            MetaData::MODELS_DATA_TYPES_NUMERIC => [
                'id'   => true,
                'year' => true,
            ],

            // Columna identidad, utilizar false si el modelo no tiene una columna identidad
            MetaData::MODELS_IDENTITY_COLUMN => 'id',

            // Como deben enlazarse y clasificarse las columnas
            MetaData::MODELS_DATA_TYPES_BIND => [
                'id'   => Column::BIND_PARAM_INT,
                'name' => Column::BIND_PARAM_STR,
                'type' => Column::BIND_PARAM_STR,
                'year' => Column::BIND_PARAM_INT,
            ],

            // Campos que deben ser ignorados en las instrucciones SQL INSERT
            MetaData::MODELS_AUTOMATIC_DEFAULT_INSERT => [
                'year' => true,
            ],

            // Campos que deben ser ignorados en instrucciones SQL UPDATE
            MetaData::MODELS_AUTOMATIC_DEFAULT_UPDATE => [
                'year' => true,
            ],

            // Valores por defecto de las columnas
            MetaData::MODELS_DEFAULT_VALUES => [
                'year' => '2015',
            ],

            // Campos que permiten string vacios
            MetaData::MODELS_EMPTY_STRING_VALUES => [
                'name' => true,
            ],
        );
    }
}
```