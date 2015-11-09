
# Nette integration

Example integration

## Step 1: Define parameters, service

```neon
parameters:
    gopay.config:
        goid: my_goid
        clientId: my_id
        clientSecret: my_secret
        isProductionMode: false
        # optional config
        scope: payment-all
        language: CS
        timeout: 30

services:
    gopay.payments:
        factory: GoPay\Api
        setup:
            - payments(%gopay.config%);
        autowired: no
```

## Step 2: Call API in controller

Pass configured payments to GoPayPresenter (in services section)

```neon
    # services section
    - App\Presenters\GoPayPresenter(@gopay.payments)
```

```php

namespace App\Presenters;

use GoPay\Payments;
use Nette\Application\BadRequestException;

class GoPayPresenter
{
    private $payments;

    public function __construct(Payments $p)
    {
        $this->payments = $p;
    }

    public function renderPay()
    {
        $response = $this->payments->createPayment([/* define your payment  */]);
        if ($response->hasSucceed()) {
            $this->template->setParameters(
                'gatewayUrl' => $response->json['gw_url'],
                'embedJs' => $this->payments->urlToEmbedJs()
            ]);
        } else {
            throw new BadRequestException((string) $response);
        } 
    }

    public function renderStatus($id = 0)
    {
        $response = $this->payments->getStatus($id);
        if ($response->hasSucceed()) {
            return ['payment' => $response->json];
        } else {
            throw new BadRequestException((string) $response);
        } 
    }
}
```

```latte
// app/presenter/templates/GoPay/pay.latte
<!DOCTYPE html>
<html>
    <head>
        <title>Pay</title>
    </head>
    <body>
        <form action="{$gatewayUrl}" method="post" id="gopay-payment-button">
          <button name="pay" type="submit">Pay</button>
          <script type="text/javascript" src="{$embedJs}"></script>
        </form>
    </body>
</html>
```

## Optional: Register custom cache and logger

```neon
services:
    gopay.payments:
        setup:
            - payments(%gopay.config%, @gopay.cache, @gopay.logger);
        autowired: no

    gopay.cache:
        class: GoPay\Token\InMemoryTokenCache

    gopay.logger:
        class: GoPay\Http\Log\PrintHttpRequest
```