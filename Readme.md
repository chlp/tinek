Добрый URL: https://www.tinkoff.ru/cardtocard/

Злой URL: https://www.tinek.ru/cardtocard/ - в сноске снизу написано, что страница не имеет прямого отношения к банку Тинькофф и создана для демонстрации уязвимости. Данные карт не перехватываются, это видно по панельке network (данные идут только на родные api tinkoff).

**Общее описание уязвимости**

Обман жертвы с целью совершения перевода средств на "злую" карту с помощью незаметной подмены карты, вписанной пользователем.

Присутствует возможность легкой копии страницы https://www.tinkoff.ru/cardtocard/ с последующим разворачиванием на злом домене с модифицированным js-скриптом, благодаря чему можно подменять введенную карточку для отправки денег.

Сохранение введенных пользователем данных не рассматриваю.

**Как выглядит для клиента**

Клиент попадает на злую страницу. Домен похож, уровень SSL-сертификата такой же, страница выглядит правильно. Клиент вводит карты, все анимации работают корректно, тип карты также определяется корректно, коммссия также расчитывается корректно. Клиент нажимает кнопку "Перевести", адрес карты "На карту" незаметно подменяется и этого он нигде не видит. Сумма остается корректной, так что клиента уже ничего не смутит.

Уже после совершения платежа клиент видит последние 4 цифры номера злой карты и ее банк.

**Почему это возможно + предложения по защите**

1. Достаточно было бы проверять на подходящий домен в Referer из Request Headers. В случае получения некорректного домена не выполнять действия, перенаправлять клиента на нужный адрес (как в самом запросе, так и пытаться это делать из js-скриптов) и получать сигнализацию о "злом" домене.

2. На экране "подтвердите перевод" отображается маскированная карта, взятая из интерфейса, хотя в запросе отправилась уже другая карта.

3. Заказать более дорогой SSL-сертификат с проверкой юр-лица и написанием компании рядом с URL'ом, иначе достаточно бесплатно автоматически сгенерированного сертификата.

4. Слишком легко скопировать страницу, которая выглядит точь-в-точь, как действующая боевая. За счет легкости копирования, легко наладить и автоматическое обновление "злой" страницы.

5. Только некоторые подгружаемые файлы не проходят загрузку из-за cors-политики. Можно было бы разрешить подгрузку бОльшего числа файлов только для указанного списка доменов.

6. Нет строгих cors-политик в api: работаем спокойно со злого домена с https://api.tinkoff.ru/v1, https://cfg.tinkoff.ru, https://www.tinkoff.ru/api/front/log.

7. Можно больше пользоваться наличием пользовательской сессии - не получать в каждом запросе все данные, а запоминать полученные ранее и далее работать по идентификаторам.

8. Можно использовать передаваемые данные для дополнительных проверок (сверяться). Например, передается сумма перевода и комиссия. Можно еще раз считать комиссию на сервере и сравнивать с переданной. Здесь же можно и получать тип карты от сервера, а не подкрашивать не полностью на стороне фронта. Да, так медленнее, зато сейчас клиент видит, как правильно определяется тип карты, хотя реально комиссию считаю уже для подмененной карты.

9. Можно не передавать данные в открытом виде в запросах (даже пусть и через https), а настроить шифрование данных, чтобы немного усилить защиту (например, передавать ключ однажды при авторизации), чтобы злоумышленник не смог получить все данные в открытом виде из запроса, если смог присоединиться только уже после передачи ключа шифрования.
