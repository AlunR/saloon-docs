# 🔥 Handling Failures

Saloon has a powerful exception handler with a strong set of default exception classes. Still, Saloon's logic can also be extended for your custom use, like for APIs that don't respond with traditional error responses, like a 2xx response with errors in the body or if you would like to use your exceptions.

### Default Exceptions

Saloon has a set of exception classes that will cover most of the scenarios.

```
SaloonException
└── TransferException
    ├── FatalRequestException (Connection Errors)
    └── RequestException (Request Errors)
        ├── ServerException (5xx)
        │   ├── InternalServerErrorException (500)
        │   ├── ServiceUnavailableException (503)
        │   └── GatewayTimeoutException (504)
        └── ClientException (4xx)
            ├── UnauthorizedException (401)
            ├── ForbiddenException (403)
            ├── NotFoundException (404)
            ├── MethodNotAllowedException (405)
            ├── RequestTimeOutException (408)
            ├── UnprocessableEntityException (422)
            └── TooManyRequestsException (429)
```

### Default Behaviour

When you send a request, Saloon will not do anything if the request fails, but by default, it will use the status code to determine if a request is successful or not.

```php
<?php

$connector = new ForgeConnector;
$response = $connector->send(new ErrorRequest);

$response->status(); // 500
$response->body(); // {"message": "Server Error"}
```

### Using the throw method

On a per-response basis, you may use the `throw` method after sending your response. This method will throw an exception if the response has a "failed" HTTP status code like 4xx or 5xx.

```php
<?php

$connector = new ForgeConnector;
$response = $connector->send(new ErrorRequest);

$response->throw();
```

### Always throwing on failed requests

Alternatively, you may wish to always throw an exception if a request fails. You may add the `AlwaysThrowsOnError` trait on your connector, and every request that fails will throw an exception, just like if you were to use the `throw` method.

```php
<?php

use Saloon\Traits\Plugins\AlwaysThrowsOnErrors;

class ForgeConnector extends Connector
{
    use AlwaysThrowsOnError;
    
    // {...}
}
```

{% hint style="info" %}
You may also add this trait to a request.
{% endhint %}

### Using the onError method

You may wish to write some custom logic in your application if a request fails, but you don't want to throw an exception. You may use the `onError` the method from the response and provide a callable to be executed if an error happens.

```php
<?php

$response = $connector->send(new ErrorRequest);

$response->onError(function (RequestException $exception) {
    // Handle any logic when an error happens.
    $this->reportException($exception);
});

// Application logic is continued
```

### Handling failures with promises

When sending requests using `sendAsync` or using request pooling, you will receive a `PromiseInterface` instance. Since this class catches exceptions, Saloon will automatically pass the request exception in the `otherwise` block, and you do not have to use the `throw` method.

```php
<?php

use Saloon\Contracts\Response;

$connector = new ForgeConnector('api-token');
$promise = $connector->sendAsync(new GetServersRequest);

$promise
    ->then(function (Response $response) {
        // Handle successful response
    })
    ->otherwise(function (RequestException $exception) {
        // Handle failed request
    });
```

### Other response methods

Saloon offers some other methods to handle failed responses.

| Method             | Description                                                                                            |
| ------------------ | ------------------------------------------------------------------------------------------------------ |
| failed             | Determines if a request has failed. By default, it will return true if the response status is not 2xx. |
| serverError        | Returns true if the response status is >= 500.                                                         |
| clientError        | Returns true if the response status is between 400 and 500.                                            |
| toException        | Creates an exception if the response is considered "failed".                                           |
| throw              | Will throw an exception if the response is considered "failed".                                        |
| getSenderException | Get the sender exception if a request failed.                                                          |
| onError            | Allows you to define a callback if the response is considered "failed".                                |







* Using handler methods, use tabs to show connector/request&#x20;
* Use the shouldThrow methods if APIs provide crappy 2xx responses with "Error" messages
