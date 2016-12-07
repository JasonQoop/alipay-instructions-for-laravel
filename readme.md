#laravel如何利用omnipay-alipay完成支付宝支付

### 安装ignited/laravel-omnipay
```php
composer require ignited/laravel-omnipay
```
这时如果有报event-dispatcher的错误,可尝试安装一下
```php
composer require symfony/event-dispatcher:^2.8
```
再执行上面的命令

### 安装lokielse/omnipay-alipay
```php
composer require lokielse/omnipay-alipay
```

### Laravel的使用

安装完成后在config/app.php中注册服务提供者:
```php
Ignited\LaravelOmnipay\LaravelOmnipayServiceProvider::class
```

添加门面:
```php
'Omnipay' => Ignited\LaravelOmnipay\Facades\OmnipayFacade::class
```

发布配置文件: 
```php
php artisan vendor:publish
```

### Lumen的使用

在bootstrap/app.php中注册服务提供者:
```php
$app->register(Ignited\LaravelOmnipay\LumenOmnipayServiceProvider::class);
```


找到lumen-framework中的Application.php添加门面
```php
class_alias('Ignited\LaravelOmnipay\Facades\OmnipayFacade','Omnipay');
```

拷贝vendor下面的ignited/laravel-omnipay/src/config中的config.php到config目录下,重命名为laravel-omnipay.php

在bootstrap/app.php中添加
```php
$app->configure('laravel-omnipay');
```

### 在laravel-omnipay.php配置文件中添加alipay

```php
<?php

return [

	// The default gateway to use
	'default' => 'alipay',

	// Add in each gateway here
	'gateways' => [
		'paypal' => [
			'driver'  => 'PayPal_Express',
			'options' => [
				'solutionType'   => '',
				'landingPage'    => '',
				'headerImageUrl' => ''
			]
		],
		'alipay' => [
			'driver' => 'Alipay_AopApp',
			'options' => [
				'appId' => 'your appId',
				'alipayPublicKey' => 'your public key',
				'notifyUrl' => 'your notify url'
			]
		]
	]

];
```

### 创建订单
```php
    $gateway = Omnipay::gateway();
    $gateway->setPrivateKey('your private key'); //这里注意区分是安卓还是ios,两者的key不一样
    $request = $gateway->purchase();
    $request->setBizContent([
        'subject'      => 'test',
        'out_trade_no' => date('YmdHis') . mt_rand(1000, 9999),
        'total_amount' => '0.01',
        'product_code' => 'QUICK_MSECURITY_PAY',
    ]);
    $response = $request->send();
    return $response->getOrderString();
```

### 异步通知
```php
public function result(Request $request){
    $gateway = Omnipay::gateway();

    $req = $gateway->completePurchase();
    $req->setParams($request->all());

    try{
        $response = $request->send();
        if($response->isPaid()) {
            Log::info('支付成功');
        } else{
            Log::info('支付失败!!');
        }
    }catch (\Exception $e){
        Log::info('支付失败!!!');
    }
}
```