# Проектная работа "Веб-ларек"

Стек: HTML, SCSS, TS, Webpack

Структура проекта:
- src/ — исходные файлы проекта
- src/components/ — папка с JS компонентами
- src/components/base/ — папка с базовым кодом

Важные файлы:
- src/pages/index.html — HTML-файл главной страницы
- src/types/index.ts — файл с типами
- src/index.ts — точка входа приложения
- src/scss/styles.scss — корневой файл стилей
- src/utils/constants.ts — файл с константами
- src/utils/utils.ts — файл с утилитами

## Установка и запуск
Для установки и запуска проекта необходимо выполнить команды

```
npm install
npm run start
```

или

```
yarn
yarn start
```
## Сборка

```
npm run build
```

или

```
yarn build
```
## Архитектура приложения

Приложение реализовано на основе MVP архитектуры. Интерфейс и бизнес-логика взаимодействуют через брокер событий (событийно - ориентированный подход). Приложение состоит из следующих компонентов:

## Детальное описание работы приложения

Главная страница состоит из списка доступных товаров, корзины с указанием товаров в ней и название магазина вместе с логотипом.
Пользователь кликом может открыть карточку товара для просмотра. Карточка товара содержит следующие данные:
- идентификатор товара;
- название товара;
- изображение товара;
- цена товара;
- описание товара;
- категория товара;
Аналогично кликом открывается корзина. В ней содержатся данные товаров, которые были туда добавлены:
- название товара;
- цена товара;
В корзину нельзя добавить несколько одинаковых товаров. Цена товаров в корзине суммируется. Также в корзине имеется возможность удалить любой товар. 
Далее в корзине есть возможность оформить заказ. При оформлении заказа необходимо указать следующие данные: 
- способ оплаты;
- адрес доставки;
- телефон;  
- Email;
Также при оформлении заказа понадобятся данные по общей сумме на оплату. Во время оформления заказа будет производиться проверка полей. Далее происходит завершение оформления заказа. В окне успеха отображается общая сумма заказа. Все формы после их использования должны быть очищены.

## Базовый код

### 1. Класс `Api`

Базовый класс доступа к веб-серверу. Реализует четыре основных типа запроса: GET -  запрос на получение данных из сервера, POST - запрос на создание нового ресурса на сервере, DELETE - запрос на удаление существующего ресурса на сервере, PUT - запрос на обновление существующего ресурса на сервере

### 2. Класс `Model`

Базовый класс для компонентов модели данных. Позволяет связать переданные данные со свойствами объекта.

### 3. Класс `EventEmitter`

Базовый класс, который позволяет подписываться на события и уведомлять подписчиков о наступлении события(паттерн "наблюдатель"). У класса имеются методы: on - подписка на событие, off - отписка от события, emit - уведомление о наступлении события.

### 4. Класс `View`

Базовый класс для работы с элементами отображения.

### Данные с сервера

API используется:
- при получении с сервера данных для карточки товара (создается список карточек);
- при отправки на сервер данных по заказу;
Интерфейс:
```js 
interface IWebLarekAPI {
	getCardItem: (id: string) => Promise<ICard>; // метод получения информации по конкретной карточки
	getCardList: () => Promise<ICard[]>; // метод получения списка карточек
	postOrderCards: (order: IOrderAPI) => Promise<IOrderResult>; //метод отправки на сервер запроса по заказу
}
```
IOrderResult - интерфейс ответа на post запрос на оформление заказа.

### MODEL (бизнес - логика)

Данные по карточке товара, которые мы получаем с сервера: 
```js 
interface IСardApi {  
    id: string; //идентификатор товара  
    description: string; //описание товара  
    image: string; //изображение товара  
    title: string; //название товара  
    category: ProductCategory; //категория товара  
    price: number | null; //цена товара  
}  
```
Категории товара описаны в отдельном типе. Данные типа взяты из макета.
```js 
type ProductCategory = "софт-скил" | "кнопка" | "другое" | "хард-скил" | "дополнительное";  
```
Данные, связанные с корзиной:  
```js 
interface ICardBasket {  
    selected: boolean; //значение выбора  
    addToBasket(): void; //метод добавления в корзину  
    removeFromBasket(): void; //метод удаления из корзины  
}  
```
Карточка товара должна использовать все данные, что описаны выше. Для этого используем extends (наследование):
```js 
interface ICard extends IСardApi, ICardBasket {}  
```
При оформлении заказа последовательно заполняются две формы:
Форма для указания способа оплаты и адреса доставки:
```js 
interface IOrderDeliveryPaymentForm {  
	payment: PaymentType; //способ оплаты  
	address: string; // адрес доставки  
}
```
для параметра payment используется тип PaymentType.
```js 
type PaymentType = "Онлайн" | "При получении";  
```
Форма для указания контактов пользователя:
```js 
interface IOrderContactsForm {  
    email: string; //email пользователя  
    phone: string; //телефон пользователя  
}  
```
Интрефейс общей формы заказа будет наследовать интерфейс форм, что описаны выше. 
```js 
interface IOrderCommonForm extends IOrderDeliveryPaymentForm, IOrderContactsForm {}  
```
Необходимо учесть, что при заполнении будет проводиться их валидация, а после отправки данных на сервер формы нужны очистить. Итоговый интерфейс оформления заказа будет выглядеть так:
```js 
interface IOderForm extends IOrderCommonForm {  
    items: ICard[]; //перечень карточек в корзине (беру из postman название значения)  
    validateOderForm(): boolean; //функция проверки корректности введенных пользователем данных  
    validateEmail(): boolean; //функция проверки корректности введенных пользователем данных - email  
    validatePhone(): boolean; //функция проверки корректности введенных пользователем данных - номера телефона  
    validatePayment(): boolean; //функция проверки корректности выбора способа оплаты - онлайн или при получении  
    validateAdress(): boolean;//функция проверки корректности введенных пользователем данных - адрес  
    clearOderForm(): void; //очистка формы  
    submitOder(): void; //завершение оформления  
}  
```
После успешного оформления заказа на сервер будут отправлены данные какие товары куплены и какова общая сумма заказа. Интерфейс отправки, у которого родителем будет IOderForm:
```js 
interface IOrderAPI extends IOderForm{
    items: ICard[]; // покупаемые товары
	total: number; // общая сумма заказа
}
```
Интерфейс самого приложения:
```js 
interface IAppState {  
    cardList: ICard[]; //перечень карточек  
    selectCard: ICard; //карточка при открытии  
    order: IOderForm; //оформление заказа  
    basket: ICard[]; //перечень карточек в корзине  
    isCardInBasket(): boolean;//метод для проверки наличия товара в корзине   
    getCardInBasket(): number;//метод получить количество карточек в корзине  
    getCardIdInBasket(): number;//метод получить id карточек в корзине  
    getTotalPrice(): number;//метод отобразить сумму заказа по всем карточкам в корзине, меняется в зависимости от доб./удаления карточек  
    makeOrder(): void;//метод сдеалть заказ  
    clearBasket(): void;//метод очистить данные корзины после подтверждения оформления заказа  
}  
```
### PRESENTER
Брокер событий будет связывать бизнес-логику и интерфейс приложения. Данные будем брать из interface IWebLarekAPI, а EventEmitter будет описывать события.

### VIEW
Интерфейс (изображение) приложения.

Главная страница. Отображается список карточек товаров и корзину:
```js 
interface IMainPage {
	galery: HTMLElement[]; // список карточек
    counter: number; // счётчик товаров в корзине
}
```
Карточка товара:
```js
interface IСard {  
    id: string; //идентификатор товара  
    description: string; //описание товара  
    image: string; //изображение товара  
    title: string; //название товара  
    category: ProductCategory; //категория товара  
    price: number | null; //цена товара  
    selected: boolean; //значение выбора  
    addToBasket(): void; //метод добавления в корзину  
    removeFromBasket(): void; //метод удаления из корзины
    button?: string; // кнопка добавления в заказ
}

interface ICardClick {
	onClick(event: MouseEvent): void; // метод открытия по клику
}
```

- Модальное окно (контент, кнопка).  
```js 
interface IModalContent {
	content: HTMLElement;
}
```

Корзина состоит из двух сущностей: отображение самой корзины с общей суммой товаров и нумерованный список самих товаров, включающий порядковый номер в списке, название товара, цена товара, кнопка удалить (реализована методом удаления).
```js 
interface IBasket {
	items: HTMLElement[];
	total: number;
}
interface IBasketCard {
	index: number; // номер по порядку в корзине
	title: string; // название товара
	price: number; // цена товара
	delete(): void; // метод удаления товара из корзины
}
```
Оформление товара состоит из 2 форм. В первой указан способ оплаты и адрес доставки. Во второй email пользователя и телефон. Поля для заполнения проходят валидацию.
```js 
interface IOrderDeliveryPaymentForm {  
	payment: PaymentType; //способ оплаты  
	address: string; // адрес доставки  
}  
  
interface IOrderContactsForm {  
    email: string; //email пользователя  
    phone: string; //телефон пользователя  
} 

interface IFormState {
	valid: boolean;
	errors: string[];
}
```
Окно успешного завершения заказа содержит общую сумму заказа.
```js 
interface ISuccessOder {
    total: number;  // общая сумма заказа
}
```
Каждой сущности будет созданн класс, который наследует базовый класс и описывает данную сущность.

Скорее всего базовый класс с описанием конструктора для представления будет вынесен в базовые сущности (как абстрактный класс, например) и будет использоваться (наследоваться) в классах сущностей.
Конструктор в базовом классе будет примерно такой:
```js 
constructor(container: HTMLElement, events: IEvents) {}
```
Для каждой сущности создается класс, который базово наследует базовый класс для представления и описывает конкретную сущность (главную страницу, карточку, корзину и т.д. ...).