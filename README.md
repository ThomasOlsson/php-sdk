# OnPay.io PHP SDK

[![Latest Stable Version](https://poser.pugx.org/onpayio/php-sdk/v/stable)](https://packagist.org/packages/onpayio/php-sdk)
[![Total Downloads](https://poser.pugx.org/onpayio/php-sdk/downloads)](https://packagist.org/packages/onpayio/php-sdk)
[![License](https://poser.pugx.org/onpayio/php-sdk/license)](https://packagist.org/packages/onpayio/php-sdk)

*Notice: Early unstable development version!*

A PHP-SDK for developing against the OnPay.io platform.
API documentation at: https://manage.onpay.io/docs/api_v1.html 

## Requirements

PHP 7.1 and later.

## Composer

You can install the SDK via [Composer](https://getcomposer.org/). Run the following command:
```bash
composer require onpayio/php-sdk
```

## Getting started
A simple usage example looks like:

```php
<?php

require_once __DIR__ . '/vendor/autoload.php';

class TokenStorage implements \OnPay\TokenStorageInterface {
    protected $token;
    protected $fileName;

    public function __construct($filename) {
        $this->fileName = $filename;
        if (file_exists($filename)) {
            $this->token = file_get_contents($this->fileName);
        }
    }

    public function getToken(): ?string {
        return $this->token;
    }

    public function saveToken(string $token) {
        $this->token = $token;
        file_put_contents($this->fileName, $token);
    }
}

// TODO: It is extremely important that the token.bin file is not accessible from the internet.
// It gives complete API access to anyone that gets a hold of it, treat it like a database password!
$tokenStorage = new TokenStorage(__DIR__ . '/token.bin');

$onPayAPI = new \OnPay\OnPayAPI($tokenStorage, [
    'client_id' => 'example.com', // It is recommended to set it to the domain name the integration resides on
    'redirect_uri' => 'http://localhost/onpay-php-sdk/example2.php?auth',
    'gateway_id' => '1234', // Should be set to the gateway id you are integrating with
]);

// Special handling if we are about to auth against the API
if (isset($_GET['auth'])) {
    if (!isset($_GET['code'])) {
        $authUrl = $onPayAPI->authorize();
        header('Location: ' . $authUrl);
    } else {
        $onPayAPI->finishAuthorize($_GET['code']);
        echo 'Authorized :tada:' . PHP_EOL;
    }
    exit;
}

// Check if we need to authenticate
if (!$onPayAPI->isAuthorized()) {
    echo 'Not authorized for the API! ' . PHP_EOL;
    echo '<a href=?auth>Click here to initiate authorization</a>' . PHP_EOL;
    exit;
}

// Execute API method
var_dump($onPayAPI->ping());

```

## Creating a payment window

Creating a payment window . Read more about the fields here: https://manage.onpay.io/docs/paymentwindow_v3.html

```php 
<?php 
declare(strict_types=1);
require_once 'vendor/autoload.php';

$paymentWindow = new \OnPay\API\PaymentWindow();
$paymentWindow->setGatewayId("YourGatewayId");
$paymentWindow->setSecret("YourSecret");
$paymentWindow->setCurrency("DKK");
$paymentWindow->setAmount("123400");
// Reference must be unique (eg. invoice number)
$paymentWindow->setReference("UniqieReferenceId");
$paymentWindow->setAcceptUrl("https://example.com/payment?success=1");
$paymentWindow->setDeclineUrl("https://example.com/payment?success=0");
$paymentWindow->setType("payment");
$paymentWindow->setDesign("DesignName");
// Force 3D secure
$paymentWindow->setSecureEnabled(true);
// Set payment method to be card
$paymentWindow->setMethod(\OnPay\API\PaymentWindow::METHOD_CARD);
// Enable testmode
$paymentWindow->setTestMode(true);
$paymentWindow->setLanguage("en");

?>

<?php if($paymentWindow->isValid()) { ?>
<form method="post" action="<?php echo $paymentWindow->getActionUrl(); ?>">
    <?php
        foreach ($paymentWindow->getFormFields() as $key => $value) { ?>
            <input type="hidden" name="<?php echo $key;?>" value="<?php echo $value;?>">
    <?php } ?>
    <input type="submit" value="Make payment">
</form>

<?php } else { ?>
    <h1>Payment window is not configured correct</h1>
<?php } ?>

```