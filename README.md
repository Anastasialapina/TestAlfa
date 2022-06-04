# Задача №1.
Требуется создать систему которая будет показывать генеалогическое древо, как бы выглядела структура базы данных для хранения данных, из каких таблиц и с какими атрибутами она состояла.

## Решение
Создадим таблицу с полями: 
* fullName - имя человека, 
* motherFullName - имя матери,
* fatherFullName - имя отца,
* dateOfBirth - дата рождения человека

В качестве Primary key будем использовать fullName, но при этом необходима связь для родителей для этого использованы forein key.

```
create table People
(
  fullName varchar(30) primary key,
  motherFullName varchar(30),
  fatherFullName varchar(30),
  dateOfBirth date,
  
  FOREIGN KEY (motherFullName) REFERENCES People(fullName),
  FOREIGN KEY (fatherFullName) REFERENCES People(fullName)
);
```
Создадим небольшое генеалогическое дерево

![alt text](https://i.ibb.co/1G1Ft0m/alpha1.png)

с помощью 
```
INSERT INTO People (fullName, motherFullName, fatherFullName, dateOfBirth) VALUES ('Ann Korablina', null , null, '1948-05-29');
INSERT INTO People (fullName, motherFullName, fatherFullName, dateOfBirth) VALUES ('Andrey Prikota', null , null, '1947-03-01');
INSERT INTO People (fullName, motherFullName, fatherFullName, dateOfBirth) VALUES ('Lena Prikota', 'Ann Korablina' , 'Andrey Prikota', '1972-01-08');

INSERT INTO People (fullName, motherFullName, fatherFullName, dateOfBirth) VALUES ('Sveta Orlova', null , null, '1950-10-02');
INSERT INTO People (fullName, motherFullName, fatherFullName, dateOfBirth) VALUES ('Nikita Ivanov', null , null, '1949-08-21');
INSERT INTO People (fullName, motherFullName, fatherFullName, dateOfBirth) VALUES ('Sergay Ivanov', 'Sveta Orlova' , 'Nikita Ivanov', '1971-04-10');

INSERT INTO People (fullName, motherFullName, fatherFullName, dateOfBirth) VALUES ('Vika Ivanova', 'Lena Prikota' , 'Sergay Ivanov', '2001-03-15');
```

Выведем созданную таблицу 
```
SELECT * FROM People;
```
![alt text](https://i.ibb.co/rmYLPnN/2022-06-04-00-51-28.png)

Подробнее можно посмотреть здесь [https://www.db-fiddle.com/f/tDeE57pUtskXkygjU8ksej/2](https://www.db-fiddle.com/f/tDeE57pUtskXkygjU8ksej/2)

# Задача №2.
Спроектируйте структуру БД для хранения лимитов на суммы платежных операций (минимальный разовый платеж, максимальный разовый платеж и максимальная сумма платежей в месяц), совершаемых клиентами через удаленные каналы доступа (банкомат, интернет-банк, мобильный банк и т.д.) Под платежными операциями понимаются платежи (за мобильный, за интернет и др.), переводы (в сторонний банк, между своими счетами и т.д.). пополнение депозита и другие.

### Требования: 
Суммы лимитов должны настраиваться как для отдельной операции (платеж за мобильный), так и для группы операций (все переводы со счета клиента)

Суммы лимитов могут отличаться в разных каналах самообслуживания

### Пример.
Через интернет-банк клиент может оплатить мобильный на сумму  от 100 до 2000 руб., при этом в сутки таких платежей может быть выполнено на сумму не более 2000 руб.
Через  банкомат клиент может оплатить мобильный на сумму от 100 до 5000 руб., при этом в день таких платежей может быть выполнено на сумму не более 5000 руб.
## Решение
Реализуем 4 сущности:
* OperationName - название операции, например, оплата мобильного.
* AccessChannel - канал доступа (банкомат, интернет-банк, мобильный банк и тд).
* TypeOperation - тип операции (платеж, перевод, пополнение, снятие), где также есть дополнительное поле суммы лимитов для группы операций LimitTypeOfMonth.
* SeparateOperation - отдельная операция, где для каждого сочетания других 3 сущностей формируются критерии минимального разового платежа (min), максимального разового платежа (max) и лимит операций LimitOperarionOfDay.

![alt text](https://i.ibb.co/ysc3SrG/z2.png)


# Задача №3.
Напишите на псевдокоде (не SQL) алгоритм функции определения возможности совершения платежа по БД, спроектированной в предыдущей задаче.

__*Входные параметры:*__ 
    
cумма платежа,      

тип операции,            

канал самообслуживания

__*Выходные параметры:*__

результат проверки ("Y" - платеж разрешен, "N" -- платеж запрещен)

```bash
 function canPay(float sumPay, String operationName, String channel) {
     TypeOperation typeOperation = TypeOperation.getByOperationName(operationName); //получим объект TypeOperation через бд
     //проверяем условие, что не превышен лимит операций такого типа в месяце 
     double sumTypeOperationOfMonth; //считаем сумму выполненных операций такого типа в текущем месяце
     if (sumTypeOperationOfMonth > typeOperation.LimitTypeOfMonth) {
     	return "N";
     }
     SeparateOperation separateOperation = SeparateOperation.get(operationName, channel, typeOperation); //получили объект SeparateOperation через бд
     //проверяем условие, что сумма платежа входит в диапазон
     if (sumPay < separateOperation.Min OR sumPay > separateOperation.Max) {
         return "N";
     }
     
     //проверяем условие, что не превышен лимит операций за день
     double sumThisOperationOfDay; //считаем сумму выполненных операций nameOperation за день
     if (sumThisOperationOfDay > separateOperation.LimitOperationOfDay) {
         return "N";
     }
     //если все условия выполнены, то возвращаем "Y"
     return "Y";
 }
```
 
# Задача №4.
Банковская система содержит основную таблицу счетов, в которой хранятся счета всех клиентов банка:
```sql
CREATE TABLE account (

id CHAR NOT NULL, -- идентификатор счета

client_id CHAR NOT NULL, -- идентификатор клиента, владельца счета

created DATE, -- дата открытия счета

closed DATE, -- дата закрытия счета

amount BIGINT, -- баланс по счету

PRIMARY KEY(id)

)
```

Необходимо написать SQL выражения для запросов, возвращающих:

* список открытых счетов клиента client_id=<номер клиента> на сегодняшний день
```sql
SELECT COUNT(id) FROM account
WHERE client_id = <номер клиента> AND closed > CURDATE();
```

* список клиентов, которые закрыли все свои счета
```sql
SELECT DISTINCT client_id FROM account
WHERE client_id not in (SELECT DISTINCT client_id FROM account WHERE closed > CURDATE());
```
* список самых "ценных" для банка клиентов, пояснив, как вы определяете "ценность" клиента

__*Вариант 1*__

Определить ценного клиента как клиента с наличием открытых счетов, количество которых больше 5

```sql
SELECT DISTINCT client_id FROM account
WHERE client_id in (SELECT DISTINCT client_id FROM account WHERE closed > CURDATE())
GROUP BY client_id
HAVING COUNT(id)>5;
```
__*Вариант 2*__

Определить ценного клиента как клиента с наличием открытых счетов, сумма на счетах у которого больше 100000

```sql
SELECT DISTINCT client_id FROM account
WHERE client_id in (SELECT DISTINCT client_id FROM account WHERE closed > CURDATE())
GROUP BY client_id
HAVING SUM(amount)>100000;
```

# Задача №5.
Даны две упорядоченные очереди, в которых могут быть одинаковые элементы. Объедините (Java) эти очереди в одну упорядоченную очередь, исключив повторяющиеся элементы.
```java

public static Deque<Integer> mergingDeque(Deque<Integer> deque1, Deque<Integer> deque2) {
        if (deque1.peek() == null)
            return deque2;
        if (deque2.peek() == null)
            return deque1;
        Integer a1 = deque1.pop();
        Integer a2 = deque2.pop();
        Deque<Integer> newDeque = new ArrayDeque<Integer>();
        while(deque1.peek() != null && deque2 != null) {
            if (a1 >= a2) {
                newDeque.add(a1);
                
                if (a1 == a2){
                    if (deque2.peek() != null)
                        a2 = deque2.pop();
                    else {
                        while (deque1.peek() != null) {
                            a2 = deque1.pop();
                            if (a1 != a2){
                                newDeque.add(a2);
                                a1 = a2;
                            }
                        }
                    return newDeque;
                    }
                }
                
                if (deque1.peek() != null)
                    a1 = deque1.pop();
                else {
                    while (deque2.peek() != null) {
                            a1 = deque2.pop();
                            if (a1 != a2){
                                newDeque.add(a1);
                                a2 = a1;
                            }
                    }
                    return newDeque;
                }
            }
            else {
                newDeque.add(a2);
                if (deque2.peek() != null)
                    a2 = deque2.pop();
                else {
                    while (deque1.peek() != null){
                        a2 = deque1.pop();
                        if (a1 != a2){
                            newDeque.add(a2);
                            a1 = a2;
                        }                        
                    }
                    return newDeque;
                }
            }
        }
        return newDeque;
    }
```



# Задача №6.
Реализуйте (Java) симметрическую разность двух коллекций используя методы Collection (addAll(), removeAll(), retainAll()).

## Решение
__*Симметрическая разность*__ коллекций - это множество элементов, одновременно не принадлежащих обоим исходным коллекциям.

```java
public static <T> Collection<T> symmetricDifference(Collection<T> collection1, Collection<T> collection2) {
    Collection<T> result = new ArrayList<>(collection1);
    // объеденим коллекции
    result.addAll(collection2);
    // найдем пересечение коллекций
    Collection<T> crossing = new ArrayList<>(collection1);
    crossing.retainAll(collection2);
    // удалим элементы, которые находятся в 1 и во 2 коллекциях
    result.removeAll(crossing);
    return result;
}
```
