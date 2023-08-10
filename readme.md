## #1
### Задание
В БД mysql есть одна единственная таблица «abc» с полями: id (int), name (varchar), cnt (int). В таблице содержится порядка 10 млн записей. Что нужно сделать, что быстро работали следующие запросы (3 разных случая)? Все остальные факторы кроме скорости чтения не критичны.
```sql
SELECT * FROM abc WHERE name = 'xxx' AND cnt = yyy
SELECT * FROM abc WHERE cnt = xxx AND name LIKE 'yyy%'
SELECT * FROM abc ORDER BY cnt ASC
```
### Решение
Можно создать индексы на столбцах, по которым будут производиться поисковые операции. В данном случае, для первого и второго запроса это столбцы name и cnt. Для третьего запроса – столбец cnt.
```sql
CREATE INDEX idx_name_cnt ON abc (name, cnt);
CREATE INDEX idx_cnt ON abc (cnt);
```

## #2
### Задание
Напишите функцию на php, которая на входе принимает номер банковской карты, а на выходе возвращает булевый результат проверки корректности ее номера, используя алгоритм «Луна».
### Решение
```php
function isLuhnValid($cardNumber) {
// Удаляем пробелы и другие символы из номера карты
$cardNumber = preg_replace('/\D/', '', $cardNumber);

    // Проверяем, что номер карты состоит хотя бы из одной цифры
    if (empty($cardNumber) || !is_numeric($cardNumber)) {
        return false;
    }
    
    $sum = 0;
    $numDigits = strlen($cardNumber);
    $parity = $numDigits % 2;
    
    for ($i = 0; $i < $numDigits; $i++) {
        $digit = (int)$cardNumber[$i];
        
        // Удваиваем цифры на нечетных позициях
        if ($i % 2 == $parity) {
            $digit *= 2;
            
            // Если результат удвоения больше 9, вычитаем 9
            if ($digit > 9) {
                $digit -= 9;
            }
        }
        
        $sum += $digit;
    }
    
    // Корректный номер карты должен иметь сумму, кратную 10
    return $sum % 10 == 0;
}

// Пример использования
$cardNumber = "1234 5678 9012 3456";
if (isLuhnValid($cardNumber)) {
    echo "Номер карты корректен.";
} else {
    echo "Номер карты некорректен.";
}
```
Это только реализация алгоритма Луна. Здесь нет проверки подлинности (соответствие номера банку).

## #3
### Задание
Есть код и запрос к БД, чтобы вы в нем изменили? Почему?
```php
$DB->query("SELECT * FROM abc WHERE id=" . $_GET['id']);
```
### Решение
Этот код может пострадать от SQL-инъекций.
Можно использовать параметризованные запросы или отображать что получили.
```php
$id = $_GET['id'];
$query = "SELECT * FROM abc WHERE id = ?";
$statement = $DB->prepare($query);
$statement->bind_param("i", $id);
$statement->execute();
$result = $statement->get_result();

while ($row = $result->fetch_assoc()) {
// обработка данных
}

$statement->close();
```

## #4
### Задание
В БД есть таблица заказов (orders) с полями:
- date - дата оформления заказа
- customer_name - имя клиента
- order_price - сумма заказа

Напишите sql запросы для выборки:
1. Запрос, который покажет сколько денег принес каждый отдельно взятый покупатель с группировкой по месяцам.
2. Запрос, который выведет имена клиентов, у которых суммарные покупки за весь период превысили 10 тыс. руб. и одновременно никогда не было заказов менее 500 руб.
### Решение
```sql
SELECT
    DATE_FORMAT(date, '%Y-%m') AS month,
    customer_name,
    SUM(order_price) AS total_amount
FROM
    orders
GROUP BY
    month, customer_name
ORDER BY
    month, total_amount DESC;
```
```sql
SELECT
    customer_name
FROM
    orders
GROUP BY
    customer_name
HAVING
    SUM(order_price) > 10000
    AND MIN(order_price) >= 500;
```

## #5
### Задание
Есть две javascript-функции:
```php
function f(a,b) { return a+b }
```
и
```php
var f = function(a,b) { return a+b }
```
Есть ли между ними разница? Если есть какая-то?
### Решение
Обе функции складывают два числа a и b и возвращают результат.
Отличается синтаксис и область видимости.
1. Объявление функции с поднятием (hoisting) и возможностью переопределения в той же области видимости.
2. Объявление анонимной функции и присвоение её переменной. Не поднимается, переопределение переменной может повлиять на функцию.

## #6
### Задание
Чем принципиально отличаются между собой условия **WHERE** и **HAVING** в mysql?
### Решение
- **WHERE** применяется к отдельным строкам перед агрегацией или выборкой.
- **HAVING** применяется к агрегатным значениям после группировки.

## #7
### Задание
Вам нужно реализовать консольный php-скрипт на сервере под Unix, который бы выводил каждые 15 секунд фразу «Hello». 
После вывода «Hello» скрипт всегда завершает работу. 
Напишите этот скрипт и пошагово расскажите, что нужно сделать, чтобы выполнялись исходные условия.
### Решение
1. Создадим `hello_script.php` (в любом текстовом редакторе)
```php
<?php
while (true) {
    echo "Hello\n";
    sleep(15); // Приостановка выполнения на 15 секунд
}
?>
```
2. Откроем терминал, перейдем в директорию и выполним команду
```bash
php hello_script.php
```
(чтобы остановить Ctrl + C)

## #8
### Задание
Напишите на php функцию для распределения рублевой скидки по купону на все товары в корзине пропорционально стоимости товара. 
На входе в функцию передаются два параметра: размер скидки в рублях (!) и массив из цен товаров, на выходе тот же массив цен, но уже с учетом скидки: `distribute_discount(int $discount, array $prices) → return array $prices;`
### Решение
```php
function distribute_discount(int $discount, array $prices) {
// Вычисляем общую стоимость товаров в корзине
$totalPrice = array_sum($prices);

    // Вычисляем коэффициент пропорции
    $proportionFactor = ($totalPrice - $discount) / $totalPrice;
    
    // Распределяем скидку пропорционально стоимости каждого товара
    $discountedPrices = [];
    foreach ($prices as $price) {
        $discountedPrice = $price * $proportionFactor;
        $discountedPrices[] = $discountedPrice;
    }
    
    return $discountedPrices;
}

// Пример использования
$discount = 100; // размер скидки в рублях
$prices = [500, 800, 1200]; // массив цен товаров
$discountedPrices = distribute_discount($discount, $prices);

print_r($discountedPrices);
```