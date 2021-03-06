---
layout: article
language: 'fr-fr'
version: '4.0'
title: 'Phalcon\Mvc\Model\Validator\Exclusionin'
---
# Class **Phalcon\Mvc\Model\Validator\Exclusionin**

*extends* abstract class [Phalcon\Mvc\Model\Validator](Phalcon_Mvc_Model_Validator)

*implements* [Phalcon\Mvc\Model\ValidatorInterface](Phalcon_Mvc_Model_ValidatorInterface)

[Source on GitHub](https://github.com/phalcon/cphalcon/tree/v{{ page.version }}.0/phalcon/mvc/model/validator/exclusionin.zep)

Phalcon\Mvc\Model\Validator\ExclusionIn

Check if a value is not included into a list of values

This validator is only for use with Phalcon\Mvc\Collection. If you are using Phalcon\Mvc\Model, please use the validators provided by Phalcon\Validation.

```php
<?php

use Phalcon\Mvc\Model\Validator\ExclusionIn as ExclusionInValidator;

class Subscriptors extends \Phalcon\Mvc\Collection
{
    public function validation()
    {
        $this->validate(
            new ExclusionInValidator(
                [
                    "field"  => "status",
                    "domain" => ["A", "I"],
                ]
            )
        );

        if ($this->validationHasFailed() === true) {
            return false;
        }
    }
}

```

## Methods

public **validate** ([Phalcon\Mvc\EntityInterface](Phalcon_Mvc_EntityInterface) $record)

Executes the validator

public **__construct** (*array* $options) inherited from [Phalcon\Mvc\Model\Validator](Phalcon_Mvc_Model_Validator)

Phalcon\Mvc\Model\Validator constructor

protected **appendMessage** (*string* $message, [*string* | *array* $field], [*string* $type]) inherited from [Phalcon\Mvc\Model\Validator](Phalcon_Mvc_Model_Validator)

Appends a message to the validator

public **getMessages** () inherited from [Phalcon\Mvc\Model\Validator](Phalcon_Mvc_Model_Validator)

Returns messages generated by the validator

public *array* **getOptions** () inherited from [Phalcon\Mvc\Model\Validator](Phalcon_Mvc_Model_Validator)

Returns all the options from the validator

public **getOption** (*mixed* $option, [*mixed* $defaultValue]) inherited from [Phalcon\Mvc\Model\Validator](Phalcon_Mvc_Model_Validator)

Returns an option

public **isSetOption** (*mixed* $option) inherited from [Phalcon\Mvc\Model\Validator](Phalcon_Mvc_Model_Validator)

Check whether an option has been defined in the validator options