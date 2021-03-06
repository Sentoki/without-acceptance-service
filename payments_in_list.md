# Метод API для получения списка входящих платежей

В случае если необходимо получить список входящих платежей для пользователя можно воспользоваться
данным методом API. Метод возвращает limit платежей начиная от самого свежего.

Необходимо отправить POST - запрос на адрес
https://lk.payin-payout.net/payment-in/get-last-payments-for-user указав следующие параметры:

|Параметр|Значение|Описание|
|---|---|---|
|user_id   | Целое число, например 1111   |Идентификатор пользователя который осуществляет работу с API   |
|timestamp   | Целое число, время Unix в секундах | [Текущее время Unix](calculate-hash.md#Метка-текущего-времени-в-параметрах) в секундах в UTC |
|email   | Строка  |email пользователя   |
|phone   | Строка  |телефон пользователя   |
|limit   | Целое число  |количество платежей, не может быть больше 1000   |
|hash   | Строка  |[Хэш безопасности](calculate-hash.md)   |

### Пример запроса реализованный на PHP

```php
<?php

$paramPost = [
    'user_id' => 7026,
    'email' => 'email@gmail.com',
    'phone' => '9131112233',
    'limit' => 100,
    'timestamp' => time(),
];

uksort($paramPost, 'strcasecmp');
$data_string = http_build_query($paramPost, '', '&');
$hash = hash_hmac('md5', $data_string, 'secret_key');
$paramPost['hash'] = $hash;
$curl = curl_init();
curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);
curl_setopt($curl, CURLOPT_POST, true);
curl_setopt($curl, CURLOPT_HTTPHEADER, [
    'Accept: application/json' // чтобы получить ошибку в случае её возникновения в json
]);
curl_setopt($curl, CURLOPT_POSTFIELDS, $paramPost);
curl_setopt($curl, CURLOPT_URL, 'https://lk.payin-payout.net/payment-in/get-last-payments-for-user');
$out = curl_exec($curl);
echo "\nout:\n$out";
$error = curl_error($curl);
echo "\nerror:\n$error";
```

В случае корректного запроса возвращается ответ:

```json
{
  "status": true,
  "result": [
    {
      "id": 1743599,
      "status": 3,
      "created": "2015-12-30 14:25:37.353059+06",
      "last_update": "2015-12-30 14:36:16+06"
    },
    {
      "id": 1769023,
      "status": 1,
      "created": "2016-01-12 15:40:03.918944+06",
      "last_update": "2016-01-12 15:41:28+06"
    }
  ],
  "tracker": " gid_5d5b9950e239b8.22633142"
}
```

### Список статусов платежей

|Статус|Описание|
|---|---|
|0   | Новый платёж, только что создан
|1   |Платёж успешно оплачен, средства зачислены получателю
|2   |Платёж отменён
|3   |Платёж упал в ошибку
|4   |Платёж был успешно оплачен, средства заморожены, ожидают зачисления
|5   |Произошло некорректное событие при обработке, необходимо обратиться в поддержку
|6   |Платёж истёк
|8   |Был произведён возврат по платежу
|9   |Был произведён чарджбек по платежу
|11   |Платёж в обработке
|13   |Платёж частично оплачен, средства заморожены, ожидают зачисления
|14   |Платёж частично оплачен, средства зачислены получателю
