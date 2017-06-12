Api.ai PHP sdk
==============

[![version][packagist-version]][packagist-url]
[![Downloads][packagist-downloads]][packagist-url]

[packagist-url]: https://packagist.org/packages/iboldurev/api-ai-php
[packagist-version]: https://img.shields.io/packagist/v/iboldurev/api-ai-php.svg?style=flat
[packagist-downloads]: https://img.shields.io/packagist/dm/iboldurev/api-ai-php.svg?style=flat

This is an unofficial php sdk for [Api.ai][1] and it's still in progress...

```
Api.ai: Build brand-unique, natural language interactions for bots, applications and devices.
```

## Install:

Via composer:

```
$ composer require iboldurev/api-ai-php
```

## Usage:

Using the low level `Client`:

```php
require_once __DIR__.'/vendor/autoload.php';

use ApiAi\Client;

try {
    $client = new Client('access_token');

    $query = $client->get('query', [
        'query' => 'Hello',
    ]);

    $response = json_decode((string) $query->getBody(), true);
} catch (\Exception $error) {
    echo $error->getMessage();
}
```

## Usage:   

Using the low level `Query`:

```php
require_once __DIR__.'/vendor/autoload.php';

use ApiAi\Client;
use ApiAi\Model\Query;
use ApiAi\Method\QueryApi;

try {
    $client = new Client('access_token');
    $queryApi = new QueryApi($client);

    $meaning = $queryApi->extractMeaning('Hello', [
        'sessionId' => '1234567890',
        'lang' => 'en',
    ]);
    $response = new Query($meaning);
} catch (\Exception $error) {
    echo $error->getMessage();
}
```

## Usage

Using the low level asynchronous api:

```php
require_once __DIR__.'/vendor/autoload.php';

use ApiAi\Client;
use ApiAi\Model\Query;
use ApiAi\Method\QueryApi;
use GuzzleHttp\HandlerStack;
use React\EventLoop\Factory;
use WyriHaximus\React\GuzzlePsr7\HttpClientAdapter;

$loop = Factory::create();
$reactGuzzle = new \GuzzleHttp\Client([
    'base_uri' => Client::API_BASE_URI . Client::DEFAULT_API_ENDPOINT,
    'timeout' => Client::DEFAULT_TIMEOUT,
    'connect_timeout' => Client::DEFAULT_TIMEOUT,
    'handler' => HandlerStack::create(new HttpClientAdapter($loop))
]);

$client = new Client('bc0a6d712bba4b3c8063a9c7ff0fa4ea', new ApiAi\HttpClient\GuzzleHttpClient($reactGuzzle));
$queryApi = new QueryApi($client);

$queryApi->extractMeaningAsync('Hello', [
    'sessionId' => '123456789',
    'lang' => 'en'
])->then(
    function ($meaning) {
        $response = new Query($meaning);
    },
    function ($error) {
        echo $error;
    }
);

$loop->run();
```

## Dialog

The `Dialog` class provides an easy way to use the `query` api and execute automatically the chaining steps :

First, you need to create an `ActionMapping` class to customize the actions behavior.

```php
namespace Custom;

class MyActionMapping extends ActionMapping
{
    /**
     * @inheritdoc
     */
    public function action($sessionId, $action, $parameters, $contexts)
    {
        return call_user_func_array(array($this, $action), array($sessionId, $parameters, $contexts));
    }

    /**
     * @inheritdoc
     */
    public function speech($sessionId, $speech, $contexts)
    {
        echo $speech;
    }
    
    /**
     * @inheritdoc
     */
    public function error($sessionId, $error)
    {
        echo $error;
    }
}

```

And using it in the `Dialog` class. 

```php
require_once __DIR__.'/vendor/autoload.php';

use ApiAi\Client;
use ApiAi\Method\QueryApi;
use ApiAi\Dialog;
use Custom\MyActionMapping;

try {
    $client = new Client('access_token');
    $queryApi = new QueryApi($client);
    $actionMapping = new MyActionMapping();
    $dialog = new Dialog($queryApi, $actionMapping);
    
    // Start dialog ..
    $dialog->create('1234567890', 'Привет', 'ru');
    
} catch (\Exception $error) {
    echo $error->getMessage();
}

```

Some examples are describe in the [iboldurev/api-ai-php-example][2] repository.

[1]: https://api.ai
[2]: https://github.com/iboldurev/api-ai-php-example
