## МЫ ВСЕ ЕЩЕ СВЯЗАНЫ? (СТ 50)

Причина использования dependency injection (внедрения зависимостей) заключается в том,  чтобы уменьшить связьв в нашем коде.
Напомним, что наш первоначальный пример, предварительная инверсия контроллера выгледела так:
```php
class CustomerController {
    public function viewAction() {
        $repository = new CustomerRepository();
        $customer = $repository->getById(1001);
        return $customer;
    }
}
```
В этом коде мы тесно связаны с CustomerRepository поскольку мы объявляем конкретную зависимость прямо в середине нашего кода.
Переключаясь на использование dependency injection, мы закончили этим:
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
(Now we’re being provided some kind of class that is an instance of CustomerRepository.)
Теперь нам предоставляется некоторый класс, который является экземпляром CustomerRepository.
Это намного более слабая связь, но это все еще связь.