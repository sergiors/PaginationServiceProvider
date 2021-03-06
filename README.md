# PaginationServiceProvider

[![Build Status](https://travis-ci.org/ttskch/PaginationServiceProvider.svg?branch=master)](https://travis-ci.org/ttskch/PaginationServiceProvider)
[![Latest Stable Version](https://poser.pugx.org/ttskch/pagination-service-provider/version.svg)](https://packagist.org/packages/ttskch/pagination-service-provider)
[![Total Downloads](https://poser.pugx.org/ttskch/pagination-service-provider/downloads.svg)](https://packagist.org/packages/ttskch/pagination-service-provider)

This service provider allows you to use [KnpPaginatorBundle](https://github.com/KnpLabs/KnpPaginatorBundle) in your Silex application.

## Requirements

* PHP 5.3+

## Getting Started

First add this dependency into your `composer.json`:

```json
{
    "require": {
        "ttskch/pagination-service-provider": "1.0.*@dev"
    }
}
```

And enable this service provider in your application:

```php
$app->register(new Tch\Silex\Provider\PaginationServiceProvider());
```

If you need, you can configure default query parameter names and templates as below (almost same as [origin](https://github.com/KnpLabs/KnpPaginatorBundle#configuration-example)):

```php
$app['knp_paginator.options'] = array(
    'default_options' => array(
        'sort_field_name' => 'sort',
        'sort_direction_name' => 'direction',
        'filter_field_name' => 'filterField',
        'filter_value_name' => 'filterValue',
        'page_name' => 'page',
        'distinct' => true,
    ),
    'template' => array(
        'pagination' => '@knp_paginator_bundle/sliding.html.twig',
        'filtration' => '@knp_paginator_bundle/filtration.html.twig',
        'sortable' => '@knp_paginator_bundle/sortable_link.html.twig',
    ),
    'page_range' => 5,
);
```

Then you can create pagination instance and render it in view:

```php
// in your controller.

$pagination = $app['knp_paginator']->paginate($someData);

return $app['twig']->render('index.html.twig', array(
    'pagination' => $pagination,
));
```

```twig
{# in your twig template #}

{{ knp_pagination_render(pagination) }}
```

## Usage

KnpPaginatorBundle can paginate [many things](https://github.com/KnpLabs/KnpPaginatorBundle#controller).
But in Silex application we may use for:

 * Array
 * Doctrine\DBALQueryBuilder

However KnpPaginatorBundle doesn't sort or filter these data automatically via request query parameter.
If you want to sort or filter these data you should do by hand.

### Sort or filter array

When you want to sort or filter simple two-dimensional array, you can use [Cake\Utility\Hash](https://github.com/cakephp/utility/blob/master/Hash.php) (autoloaded) like as below:

```php
// in your controller.

$array = /* some two dimensional array */;

$sort = $request->get('sort');
$direction = $request->get('direction', 'asc');
$filterField = $request->get('filterField');
$filterValue = $request->get('filterValue');

$array = Hash::extract($array, "{n}[{$filterField}=/{$filterValue}/]");
$array = Hash::sort($array, "{n}.{$sort}", $direction);

$pagination = $app['knp_paginator']->paginate($array); // You can get filtered and sorted pagination object.
```

See the [official document](http://book.cakephp.org/2.0/en/core-utility-libraries/hash.html) for more information of usage of Hash class.

### Sort or filter Doctrine\DBALQueryBuilder

When you use Doctrine\DBALQueryBuilder you can sort or filter by SQL clauses like as below:

```php
// in your controller.

$sort = $request->get('sort');
$direction = $request->get('direction', 'asc') === 'asc' ? 'asc' : 'desc';
$filterField = $request->get('filterField');
$filterValue = $request->get('filterValue');

$qb = $app['db']->createQueryBuilder()
    ->select('t.*')
    ->from('table', 't')
    ->where("{$app['db']->quoteIdentifier($filterField)} like {$app['db']->quote('%' . $filterValue . '%')}")
    ->orderBy($app['db']->quoteIdentifier($sort), $direction)
;

$pagination = $app['knp_paginator']->paginate($qb);
```

You can see demo code [here](demo/index.php).

## Additional features

This service provider also provides bootstrap3-based beautiful pagination and filtration templates. You can use it as below:

```php
$app['knp_paginator.options'] = array(
    'template' => array(
        'pagination' => '@tch_silex_pagination/pagination-bootstrap3.html.twig',
        'filtration' => '@tch_silex_pagination/filtration-bootstrap3.html.twig',
    ),
);
```

When you use the `pagination-bootstrap3.html.twig` template, you can configure the list of `Items per page` selector.

```php
$app['knp_paginator.limits'] = array(10, 25, 50, 100, 200, 500),
```

You also can define translations for some labels in the `messages` domain.

```php
$app['translator.domains'] = array(
    'messages' => array(
        'ja' => array(
            'Previous' => '前へ',
            'Next' => '次へ',
        ),
    ),
);
```

```php
$app['translator.domains'] = array(
    'messages' => array(
        'ja' => array(
            'Items per page' => '1ページの件数',
            'Filter' => '絞り込み',
        ),
    ),
);
```

## Note

This service provider depends on `TwigServiceProvider`, `UrlGeneratorServiceProvider` and `TranslationServiceProvider`, but you don't have to worry about that because they will be automatically enabled via this provider.
