## Определение контракта с интерфейсами (52 ст)
### (Defining a Contract with Interfaces)  
  
В предыдущей главе мы обсуждали принцип инверсии управления, и описывали, как внедрение зависимостей (dependency injection)  может упростить рефакторинг и тестирование кода.  Мы начали с обсуждения связей между объектами, и представленный dependency injection  как метод уменьшения связи между ними. Однако, когда мы дойдем до конца, мы не полностью избавимся от зависимости, мы только  сделаем связь слабее переместив ее из методов класса в конструктор.  
_However, when we reached the end, we realized that we hadn’t entirely removed the coupling, we only   
made it weaker by moving it to the constructor from the methods of the class._
  
```php 
class CustomerController {  
    protected $repository;  
  
    public function __construct(CustomerRepository $repo) {  
        $this->repository = $repo;  
    } 
    
    public function viewAction() { 
        $customer = $this->repository->getById(1001);  
        return $customer; 
    }  
}
```  
Этот класс все еще зависит от CustomerRepository. Так как CustomerRepository класс отвечает за, 
по крайней мере через его зависимость, подключение и извлечение данных из БД, это означает, 
что CustomerController также связан с этой базой данных и зависит от нее.

Это может быть проблематично, если мы когда-нибудь решили переключить источники данных, например, 
перейти на NoSQL вариант, или переключение на какой-либо API-сервис для предоставления наших данных. 
Если мы это сделаем, тогда мы должны будем измените весь код где использует CustomerRepository.
_If we do that, we’ll have to then go modify all the code across our entire code base that uses this 
CustomerRepository to make it use something else._

Кроме того, если мы хотим протестировать CustomerController, мы все равно должны дать ему экземпляр 
CustomerRepository. Любой создаваемый нами фиктивный объект необходимо расширить из CustomerRepository 
пройти проверку типа подсказки. _type-hint check_
Это означает, что наш простой репозиторий все еще должен иметь вся с база данных, даже если мы 
переопределяем все, что он делает. Это довольно неряшливо (небрежно). 


### Интерфейсы в PHP
#### (Interfaces in PHP)

Напомним, что в PHP интерфейс - это определение класса, без подробностей реализации. 
Вы можете думать об этом как о скелете класса. В интрефейсе методы не могут быть реализованные, только описание метода (signatures).

```php
interface Automobile {
    public function drive();
    public function idle();
    public function park();
}
```

Любой класс, реализующий интерфейс, должен реализовывать все методы этого интерфейса.

```php
class Car implements Automobile {
    public function drive() {
        echo "Driving!";
    }
    public function idle() {
        echo "Idling!";
    }
    public function park() {
        echo "Parking!";
    }
}
```    

Для интерфейса Automobile может существовать любое количество реализаций:

```php
class DumpTruck implements Automobile {
    public function drive() {
        echo "Driving a Dump Truck!";
    }
    public function idle() {
        echo "Idling in my Dump Truck!";
    }
    public function park() {
        echo "Parking my Dump Truck!";
    }
}
```

Два класса Car и DumpTruck считаются совместимыми, поскольку они оба определяют Automobile
интерфейс, и любой из них может быть использован в любом случае, когда необходим Automobile.

Это называется полиморфизмом (polymorphism), где объекты различных типов могут использоваться 
взаимозаменяемо, до тех пор, пока они все наследуются от общего подтипа.

### Использование интерфейсов в качестве типа подсказки (ст 54)
#### (Using Interfaces as Type Hints)

Использование интерфейсов удобно при попытке уменьшить связь внутри класса. Мы можем определить интерфейс 
некоторой зависимости, а затем ссылаться только на этот интерфейс. Мы можем определить интерфейс некоторой 
зависимости, а затем ссылаться только на этот интерфейс. До сих пор, мы применяли конкретный экземпляр CustomerRepository.
(_So far, we’ve been passing around concrete instances of CustomerRepository._) Сейчас, мы будем создавать интерфейс 
который определяет функциональность этого хранилища:

```php
interface CustomerRepositoryInterface {
    public function getById($id);
}
```

У нас есть простой интерфейс с одним методом, getById(), который возвращает объект Customer для данных клиента, 
идентифицированных $id. (_We have a simple interface with one method, getById(), which returns a Customer object for 
the customer data identified by $id._) Поскольку этот интерфейс не предоставляет реализацию, поэтому этот класс не 
предоставляет информации о том, откуда поступают данные, или как их извлекают.

Сейчас в нашем контроллере, мы используем подсказку типа (PHP’s type-hints) для методов и функций объявленных как 
аргументы, аргумент передаваемый в конструктор __construct()должен быть экземпларом нашего нового интерфейса CustomerRepositoryInterface.

```php
class CustomerController {
    protected $repository;

    public function __construct(CustomerRepositoryInterface $repo) {
        $this->repository = $repo;
    }

    public function viewAction() {
        $customer = $this->repository->getById(1001);
        return $customer;
    }
}
```
Теперь класс CustomerController соединен только с CustomerRepositoryInterface, и это круто: этот интерфейс не является 
конкретной реализацией, это просто определение реализация. Мы можем привязаться к этому и, фактически, мы должны, 
поскольку это определяет, как наше приложение взаимодействует, не ссылаясь на конкретные реализации.

Какой бы механизм ни отвечал за создание экземпляра CustomerController, он все же может предоставить это с конкретным 
CustomerRepository, пока этот класс реализует ‘CustomerRepositoryInterface.

```php
class CustomerRepository implements CustomerRepositoryInterface {
        public function getById($id) {
        // get and return the customer...
    }
}
```
CustomerRepository предоставляет реализацию метода getById() выполняя требования интерфейса.
Если мы сейчас захотим протестировать данный контроллер, вместо этого мы можем использовать фиктивный экземпляр 
CustomerRepositoryInterface:

```php
class MockCustomerRepository implements CustomerRepositoryInterface {
    public function getById($id) {
        if ($id == 1001) {
            return (new Customer())
                ->setId(1)
                ->setName('Customer #1001');
        }
    }
}
```

Хотя это может быть не самый лучший код (лучше использовать библиотеки такие как [mockety](https://github.com/mockery/mockery) 
или [phpunit](https://phpunit.de/manual/4.1/en/test-doubles.html)), тем не менее, мы можем передать CustomersController 
для выполнения подсказки типа CustomerRepositoryInterface. Теперь контроллер тестируется изолированно, без конкретной 
конфигурации и зависимостей реального CustomerRepository.

CustomersController неважно какой объект вы будете передавать в конструктор. До тех пор пока объект будет реализовывать нужный интерфейс
(в противном случаии будет выброшено исключение и вы получите ошибку), контроллер должен работать точно также.
Это означает что внедренная зависимость фактически не влияет на работу контроллера и возвращает нужные результаты.

> #### Принцип подстановки Лисков  
> (If this sounds familiar, it should.) Мы уже говорили об этом принципе когда рассматривали Принцип подстановки Лисков
> (Liskov Substitution Principle). Наши хранилища взаимозаменяемы, поскольку реализуют один и тотже интерфейс.
>
> Принцип подстановки Лискова также применяется здесь, так как мы модифицируем код высшего уровня (контроллер в нашем примере),
> чтобы не зависеть от кода низшего уровня (хранилища), и вместо этого зависеть от обстракции.  