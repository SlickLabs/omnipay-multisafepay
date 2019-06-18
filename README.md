# Omnipay: MultiSafepay

**MultiSafepay driver for the Omnipay PHP payment processing library**

[![Build Status](https://travis-ci.org/thephpleague/omnipay-multisafepay.png?branch=master)](https://travis-ci.org/thephpleague/omnipay-multisafepay)
[![Latest Stable Version](https://poser.pugx.org/omnipay/multisafepay/version.png)](https://packagist.org/packages/omnipay/multisafepay)
[![Total Downloads](https://poser.pugx.org/omnipay/multisafepay/d/total.png)](https://packagist.org/packages/omnipay/multisafepay)

[Omnipay](https://github.com/thephpleague/omnipay) is a framework agnostic, multi-gateway payment
processing library for PHP 5.3+. This package implements MultiSafepay support for Omnipay.

## Installation

Omnipay is installed via [Composer](http://getcomposer.org/). To install, simply add it
to your `composer.json` file:

```json
{
    "require": {
        "omnipay/multisafepay": "~2.0"
    }
}
```

And run composer to update your dependencies:

    $ curl -s http://getcomposer.org/installer | php
    $ php composer.phar update

## Basic Usage

The following gateways are provided by this package:

* MultiSafepay_Rest
* MultiSafepay_Xml (Deprecated)

For general usage instructions, please see the main [Omnipay](https://github.com/thephpleague/omnipay)
repository.

## Support

If you are having general issues with Omnipay, we suggest posting on
[Stack Overflow](http://stackoverflow.com/). Be sure to add the
[omnipay tag](http://stackoverflow.com/questions/tagged/omnipay) so it can be easily found.

If you want to keep up to date with release anouncements, discuss ideas for the project,
or ask more detailed questions, there is also a [mailing list](https://groups.google.com/forum/#!forum/omnipay) which
you can subscribe to.

If you believe you have found a bug, please report it using the [GitHub issue tracker](https://github.com/thephpleague/omnipay-multisafepay/issues),
or better yet, fork the library and submit a pull request.

## Alterations

The file `src/Message/RestPurchaseRequest.php` are altered to make it possible to send all files the payment method "Afterpay" requires.

New methods:

`->setBirthday();` //Set the birthday of the user that is ordering
`->getBirthday();` //Get the birthday of the user that is ordering

`->setGender();` //Set the gender of the user that is ordering
`->getGender();` //Get the gender of the user that is ordering

`->setDelivery();` //Set the delivery address of the user that is ordering
`->getDelivery();` //Get the delivery address of the user that is ordering

`->setShoppingCart();` //Set all items that are in the cart
`->getShoppingCart();` //Get all items that are in the cart

`->setCheckoutOptions();` //Set all the checkout options
`->getCheckoutOptions();` //Get all the checkout options

Example usage:
```
$omnipay = Omnipay::create('MultiSafepay_Rest');
$omnipay->setApiKey('APIKEY');

        $card = array( //Billing Address
            'clientIP' => \Request::ip(),
            "locale"=> \App::getLocale(),

            "first_name"=> $billingAddress->firstname,
            "last_name"=> $billingAddress->lastname,
            "address1"=> $billingAddress->street,
            "house_number"=> $billingAddress->housenumber,
            "zip_code"=> $billingAddress->postcode,
            "city"=> $billingAddress->city,
            "country"=> $billingAddress->country_id,
            "email"=> $billingAddress->email,
            "phone" => $billingAddress->telephone,
        );
        $delivery = array( //Shipping Address
            "first_name"=> $shippingAddress->firstname,
            "last_name"=> $shippingAddress->lastname,
            "address1"=> $shippingAddress->street,
            "house_number"=> $shippingAddress->housenumber,
            "zip_code"=> $shippingAddress->postcode,
            "city"=> $shippingAddress->city,
            "country"=> $shippingAddress->country_id,
            "email"=> $shippingAddress->email,
            "phone" => $shippingAddress->telephone,
        );

        //Prepare the general information about the order
        $request = $omnipay->purchase(array(
            'gateway' => 'AFTERPAY',
            'amount' => $dashboardOrderData->total,
            'locale' => \App::getLocale(),
            'currency' => 'EUR',
            'description' => sprintf(config('omnipay.description'), $dashboardOrderData->number),
            'orderId' => $dashboardOrderData->number,
            'transactionId' => $transactionId,
            'type' => 'redirect',
            'returnUrl' => route(config('omnipay.returnUrl')),
            'cancelUrl' => route(config('omnipay.cancelUrl')),
            'notificationUrl' => route(config('omnipay.notificationUrl')),
            'card' => $card,
        ));
        $request->setDelivery($delivery);

        $request->setGender('mrs');
        $request->setBirthday('2001-02-23');

        //Every product that is ordered (dont forget to include shipping and transaction fees as a product to!)
        $request->setShoppingCart([
        	0 => [
        		'name' => 'ITEMNAME',
        		'description' => 'ITEMDESCRIPTION',
        		'unit_price' => '1.00',
        		'quantity' => '4',
        		'merchant_item_id' => 'sku38273',
        		'tax_table_selector' => 'VAT6', //This needs to link to the corresponding VAT from the checkoutoptions array
        	]
    	]); 

    	//Set all different taxes that are applied to products
    	$request->setCheckoutOptions([
            "tax_tables"=> [
                "default"=> [
                    "shipping_taxed"=> "true",
                    "rate"=> "0.21"
                ],
                "alternate"=> [
                    [
                        "name"=> "VAT6",
                        "standalone"=> true,
                        "rules"=> [
                            [
                                "rate"=> "0.06"
                            ]
                        ]
                    ],
                ]
            ]
        ]);   
```