## Использование интерфейсов в качестве соглашения (56 ст)
### (Using Interfaces as a Contract)

Еще один способ использования интерфейсоов вместе с dependency injection это то, что он исполняет контракт.
Наш интерфейс - это контракт между поставщиком, наш код создает экземпляр зависимости и вводить его, 
клиент - класс с зависимостью которая нам нужна. Контракт выполняется, когда правильный объект вводится в объект.

Эта концепция была описана как программирование по контракту. Это интересный путь использования интерфейсов и внедрения зависимостей.

## Приведение кодекса поведения третьей стороны в соответствие с контрактами
### (Making Third Party Code Conform to Contracts)

Использование интерфейсов для определения контракта (договоренности) легко когда это наш код, но как это использовать 
когда это сторенняя библиотека и привести ее в соответствие для внедрения зависимости? Прежде всего мы не должны изменять 
стороннюю библиотеку для нашего интерфейса. 

Ответ в том. чтобы использовать **паттерн Адаптер**.