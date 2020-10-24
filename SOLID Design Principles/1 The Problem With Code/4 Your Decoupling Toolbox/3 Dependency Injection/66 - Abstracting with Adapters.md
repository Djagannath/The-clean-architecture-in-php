## Абстрагирование через адаптер
### Abstracting with Adapters

Интерфейсы предоставили возможность полностью отделить наш код от конкретных реализаций их зависимостей.
До сих пор мы показывали, как это сделать только с помощью нашего собственного низкоуровневого кода.
Что, если вместо этого мы захотим использовать какую-нибудь стороннюю библиотеку?

Допустим, мы находим отличную стороннюю библиотеку через Packagist под названием Bill's Geocoder, 
которая проверяет адреса с помощью Google Maps, USPS или другого сервиса.

```php
class AddressController extends AbstractController 
{
    protected $geocoder;

    public function __construct(BillsGeocoder $geocoder) 
    {
        $this->geocoder = $geocoder;
    }

    public function validateAddressAction() 
    {
        $address = $this->vars()->fromPost('address');
        $isValid = $this->geocoder->geocode($address) !== false;
    }
}
```

Мы внедрили в контроллер некую зависимость. Это шаг в правильном направлении и решает некоторые проблемы, 
но он все еще сильно связывает наш контроллер с пакетом BillsGeocoder. Аесли данный пакет будет удален или перестанет поддерживаться.
Что если ты решишь, что DavesGeocoder намного лучше потому что он поддерживает Zip+4, а BillsGeocoder нет?
А что если ты используешь этот geocoder в разных местах своего кода, а со временем нужно обновить все эти ссылки?
Что делать, если DavesGeocoder не имеет метода geocode(), а вместо этого имеет validateAddress(). Ты столкнешся с кошмаром рефакторинга.

### Настройка адаптера

Вспомните наше обсуждение шаблонов дизайна _design patterns_, в частности, шаблона адаптера _Adapter Pattern_. 
Адаптеры идеально подходят для решения этой проблемы, так как они позволяют нам "обернуть" функциональность кода сторонней библиотеки, 
и сделав это в соответствии с интерфейсом, который мы определяем, так что мы можем внедрить адаптер для использования интерфейса.

Именно это мы и сделали, когда обсуждали модель адаптера. Мы начали с определения нашего интерфейса:

```php
interface GeocoderInterface 
{
    public function geocode($address);
}
```

Тогда мы сделаем так, чтобы наш контроллер зависел только от этого интерфейса:

```php
class AddressController extends AbstractController 
{
    protected $geocoder;

    public function __construct(GeocoderInterface $geocoder) 
    {
        $this->geocoder = $geocoder;
    }

    public function validateAddressAction() 
    {
        $address = $this->vars()->fromPost('address');
        $isValid = $this->geocoder->geocode($address) !== false;
    }
}
```

Наконец, мы создадим адаптер для обёртывания BillsGeocoder и приведём его в соответствие с нашим GeocoderInterface, 
который требуется нашим классом AddressController:

```php
class BillsGeocoderAdapter implements GeocoderInterface 
{
    protected $geocoder;

    public function __construct(BillsGeocoder $geocoder) 
    {
        $this->geocoder = $geocoder;
    }

    public function geocode($address) 
    {
        return $this->geocoder->geocode($address);
    }
}
```

В нашем методе geocode() мы просто передаём обработку нашему экземпляру BillsGeocoder, который мы берём через конструктор. 
Мы можем использовать инъекцию зависимостей для внедрения экземпляра BillsGeocoderAdapter в наш AddressController, 
что позволяет нам использовать стороннюю библиотеку, но при этом убедиться, что она соответствует нужному нам интерфейсу.

### Как это поможет?

Этот метод использования адаптеров со сторонними библиотеками позволяет нам оставаться отделенными и свободными от 
зависимости со сторонними библиотеками. Он позволяет нам свободно менять этими зависимостями без необходимости переписывать код, 
который их использует, и позволяет нам легко протестировать наше приложение и его использование этих зависимостей, 
не тестирую на самом деле эти зависимости самим. Мы только должны проверить, что мы правильно их используем. 
Позже мы обсудим важность _External Agency Independenc_, когда будем обсуждать "The Clean Architecture".
