---
layout: post
title:  "ReactJs for Symfony"
date:   2018-10-30 19:00:00 +0300
categories: symfony tutor reactjs
---

## 1. The World of React + ESLint

Устанавливаем приложение. 
Устанавливаем eslint.

yarn add eslint --dev

Добавляем файл `.eslintrc.js` и включаем его в PhpStorm

`Languages & Frameworks > JavaScript > Code Quality Tools > ESLint`


## 2. React.createElement()

yarn add react react-dom --dev

const el = React.createElement('h2', null, 'Lift History!');

где null - это props. Html атрибуты для элемента.
А дальше идут дети этого элемента.

Через ReactDom элемент добавляется на страницу.

Все просто. Но нужен инструмент, который упростить верстку.


## 3. JSX

Добавили preset для encore.

`yarn add babel-preset-react --dev`

`Encore.enableReactPreset()`


Добавили lint.

`yarn add eslint-plugin-react --dev`

И в файле  `.eslintrc.js` добавил extends `plugin:react/recommended` .


## 4. React Components

`class RepLogApp extends React.Component`

Компонент должен называться с большой буквы и иметь метод render().

Вызывать его можно просто `console.log(<RepLogApp/>);`

В итоге реорганизуем код.


## 5. Props

Когда мы вызываем <RepLog/> по сути мы инстанциируем класс.

Внутри JSX, если нам нужен JS используем его внутри {}.

Свойства передавать как аргументы


## 6. Collection & Rendering a Table

Создаем шаблон таблицы


## 7. The key Prop & Inline Rendering

Объяснение короткого синтаксиса для циклов


## 8. Build the Static App First

Выносим форму в JSX.
Стоит иметь в виду, что атрибут `class` надо заменять на `className`. `novalidate` -> `noValidate`. А `selected="selected""` -> `defaultValue`.

Иногда удобно использовать dev-server. Когда 
```
yarn run encore dev-server --port 8088 --host=127.0.0.1
```


## 9. State: For Magic Updating Good Times

Есть 2 важных вещи в ReactJS. Props & State.

Props data is immutable (no changed).
State - is changeble data.

Рекомендуется поставить `React Developer Tools` для Хрома. И тогда можно вредактировать компоненты React из консоли хрома.


## 10. Handling Events (like onClick)!

Пример обработчика событий onClick.


## 11. Child Component

Не надо дублировать атрибуты а родительского и дочернего элементов. State стоит делать только у одного. А у второго - props.
То есть у нас есть state у родительского компонента и он передает в props это состояние. И дочерний ререндерится с новым props'ом.


## 12. Notifying Parent Components: Callback Props

Дочерний элемент не связан с родительским, потому что он может быть вызван у множества различных родителей.

Поэтому можно передавать callback в дочерний элемент. Но стоит помнить про область видимости this. 
Поэтому используют `this.handleRowClick = this.handleRowClick.bind(this);`

Типы компонентов
- statefull - smart components
- stateless - dumb components


## 13. Smart vs Dumb Components

При грубом округлении имеется один умный компонент, который имеет состояния. А все дочерние компоненты - компоненты представления.
Они просто получают данные и отображают их.

По аналогии с PHP. Smart - Controller. Dumb - Template.

Dumb компоненты по сути имеют только один метод render. Поэтому можно `export default function RepLogList(props)` вместо class.

На самом деле они могут иметь состояния. Но об этом позже.


У умного копонента наоборот не должно быть шаблона. А только логика.

Поэтому для самого верхнего компонента логики нужно создать его компонент представления.


## 14. Prop Validation: PropTypes

`yarn add prop-types --dev`

Добавляем такой код в конце компонента с описанием типов.
```js
RepLogList.propTypes = {
    highlightedRowId: PropTypes.any,
    onRowClick: PropTypes.func.isRequired
};
```

По-умолчанию все props is optional.


## 15. Removing propTypes on Production

yarn add babel-plugin-transform-react-remove-prop-types --dev


## 16. Moving the Rep Logs to State

Родитель отправляет данные детям. Но он не спрашивает у детей данные. Это возможно, но это плохой flow. Это называется "поток вниз".
Это нужно, чтобы состояниие знал только верхний компонент. Информация не может идти вверх.

Перенесли все данные в родительский компонент


## 17. Smart Components & Spread Attributes

Нам приходится прокидывать в дочерний шаблонный компонент все props и state. Есть короткий синтаксис. (spread ...)

```js
return <RepLog
    {...this.props}
    {...this.state}
    onRowClick={this.handleRowClick}
/>;
```

Для подсчета общего веса в файле компонента-шаблона (который по сути одна функция render) добавили вспомогательную функцию. И привели пример короткой стролочной функции.


## 18. Handling a Form Submit

На onSubmitForm React обеспечивет, что event.target - это компонент React.

Так же этот пример о том, что родительскому компоненту вообще не важно, кто и как будет использовать `handleCreateNewItem`.
Будь то форма или рандомная генерация. Или еще что-то.

Так что родительский компонент не должен знать про обработчик форм. А обработчик формы должен быть на уровне представления. 
А внутри него уже вызовется родетельский метод о создании сущности.


## 19.1. New Component to Hold our Form

Создали просто DumbComponent. Без состояния и логики.

Но наш этот компонент имеет ссылку на refs - что есть Dom элементом.

## 19.2. New Component to Hold our Form. Refs

Для формы все таки создаем компонент с обработчиком. 

Заметка. Не забываем bind(this) в методах, когда используем this. Надо `this.handleFormSubmit = this.handleFormSubmit.bind(this);` в конструкторе

## 20. Refs

Refs - система ReactJS для доступа к любому DOM-элементу.

В конструкторе создали.
```js
this.quantityInput = React.createRef();
this.itemSelect = React.createRef();
```
В шаблоне завязали
```js
<select id="rep_log_item" ref={this.itemSelect} >
<input type="number" id="rep_log_reps" ref={this.quantityInput}>
```
И дальше в обработчике их можно использовать
```js
const quantityInput = this.quantityInput.current;
const itemSelect = this.itemSelect.current;
```

Для генереции uuid `yarn add uuid --dev`


## 21. Immutability / Don't Mutate my State!

Непонятный момент. Вроде бы когда мы сооздаем `const repLogs = this.state.repLogs;` мы просто создаем ссылку на объект. И его изменение меняет исходный код.
Но почему-то мы все равно вызываем setState().

Это связано с особенностью ReactJS. Который ререндерить DOM только на этих моментах. И надо делать установку нового состояния рядом в setState().

А еще лучше вообще передавать в setState колбек.

Небольшой рефакторинг


## 22. Dumb Components with State

Что с валидацией?

1. Валидация на сервера
2. JS валидация
3. Валидация HTML5

Для показа сообщения об ошибки, нужно изменять состояние компонента. То есть нужно менять state.
Сообщение валидации - это не настоящая бизнес логика. Это лишь логика отображения формы. Так как наш RepLog вообще не знает, что у него есть форма.


## 23. Form Validation State

Добавим состояние `quantityInputError` в компонент `Creator`

Так как в атрибут className я хочу передать js, приходится и постоянный класс `form-group` также указывать внутри js.


Используем спец синтаксис && от JSX.


## 24. Controlled Form Input

Есть 2 подхода к взаимодействию с формами:
- Брать из DOM
- Брать из state

В первом случае можно брать event.target в  onChange handler.

А во втором просто прокидываем свойство


## 25. Controlled Component Form

Чет опять тупанул и не понял, для чего нам нужны эти Controlled Components.

Я так понял, это второй вариант управления формами.

Тут объясняется Controlled vs Uncontrolled Components. 

ControlledComponents официально рекомендует React. Но Райн рекомендует исползьвать Uncontrolled Components потому что контролируемые сложные и имеют состояния.
В большинстве случаев. Если что не сложно перейти от неконтролируемых к контролируемым.

Контролируемы стоит использовать ... хз я не понял


## 26. Deleting Items

Удаление должно происходить по кнопке. Но состояние хранит RefLogApp. Значить надо в нем создать обработчик. И прокинуть его внутрь дочерних элементов.

Так же стоит помнить, что надо опасно переопределять `state`. См как сделано с `prevState`.
```js
this.setState((prevState) => {
    return {
        repLogs: prevState.repLogs.filter(repLog => repLog.id !== id)
    };
});
``` 


## 27. API Setup & AJAX with fetch()

Рассказ про то, как устроент RepLogController. Это API контроллер. Он имеет пару методов для помощи в построении вывода. А так же спец ApiRepLogModel.

Файл `rep_log_api.js` потому что он экспортирует функцию и это соглашение, когда `export function`.

У ReactJS есть функция `fetch('/reps')` для ajax запросов, которая возвращает Promise. 

Возникла проблема с аутентификацией.


## 28. API Auth & State via AJAX

API auth - большая тема. Ее пропустим. Просто скажем fetch() чтобы он также отправлял COOKIE.

```js
fetch('/reps', {
    credentials: 'same-origin'
})
```

В ReactJs есть метод, который вызывается автоматом. Это отличное место, куда можно добавлять ajax , вместо конструктора
```js
componentDidMount() {
    ...
}
```


## 29. Loading Messages

Добавим Loader, который ожидает подгрузки ajax'a. Просто добавили еще одно состояние.


## 30. Hitting the DELETE Endpoint

Добавим еще одну функцию в `rep_log_api.js`. `export function deleteRepLog(id)`

Проблема, мы хардкодим url'ы. Надо чем-то генерить эти url'ы. Можно использовать FOSJsRoutingBundle. Или создать переменную и в нее записать роуты.

Не если мы работаем с API в одном месте - все нужные роуты будут там. И это норм.

Импортим эту функцию и используем ее в нужном месте.
```js
handleDeleteRepLog(id) {
    deleteRepLog(id);
    this.setState((prevState) => {
        return {
            repLogs: prevState.repLogs.filter(repLog => repLog.id !== id)
        };
    });
}
```

Сейчас мы оптимистично удаляем в интерфейс строку сразу, не дожидаясь ответа аякса. Но могу быть ситуации, когда надо ждать.

В rep_log_api.js создадим приватную функцию общую для обоих методов. Обратить вниминие, что она имеет один `.then()`, а getList имеет еще один.
```js
function fetchJson(url, options) {
    return fetch(url, Object.assign({
        credentials: 'same-origin',
    }, options))
        .then(response => {
            return response.json();
        });
}
export function getRepLogs() {
    return fetchJson('/reps')
        .then(data => data.items);
}
export function deleteRepLog(id) {
    return fetchJson(`/reps/${id}`, {
        method: 'DELETE'
    });
}
```


## 31. The POST Create API

Создание RepLog происходит в контроллере `\App\Controller\RepLogController::newRepLogAction()`

Создадим новый метод `createRepLog()` в `rep_log_api.js`.

Этот метод заюзаем в месте, где добавляем строку `RepLogApp.handleAddRepLog()`.

Получаем ошибку `This form should not contain extra fields.`. По условиям формы, надо получать только нужные поля. 
И это reps, item. Будем передавать только их.

Тут тот случай, когда нельзя обновлять UI без получения подтверждения от сервера. 

Так же можно сделать следующее. Использовать вместо AUTO_INCREMENT uuid. А uuid можно генерировать на фронте. 
Это позволит не ждать сервак, если мы уверены в валидности данных.


## 32. Polyfills: fetch & Promise

Полифилы нужны, для поддержки старых браузеров и неподдерживаемого функционала в разных браузерах.

`yarn add whatwg-fetch --dev `

Чтобы полифилы использовались, надо их просто заимпортить в layout.js. Это как это - share entry point.

Тоже надо сделать и для Promise. `yarn add promise-polyfill --dev`

Небольшое объяснение, как он работает (whatwg-fetch). Это самовызывающаяся функция, куда как self передается объект window. 
Там проверяется, есть ли self.fetch(). Если есть - выходим. Если нет - добавляем.


## 33. Success Messages + The Style Attribute

Добавим лодинг при создании записи. Это будет доп строка в конце таблицы.

Тут так же как обычно в компоненте состояния создаем поле. И прокидываем его до компонента отображения.
Появился новый интересный синтаксис для атрибута `style`:
```js
{isSavingNewRepLog && (
    <tr>
        <td
            colSpan="4"
            className="text-center"
            style={{
                opacity: .5
            }}
        >Lifting to the database ...
        </td>
    </tr>
)}
```

##### Success Message

Все тоже самое. Создали новое свойство состояния. Прокинули во View. Но надо ее скрывать через некоторое время. Эт следующим шагом


## 34. Temporary Messages & componentWillUnmount

Сделаем небольшой рефакторинг. Создадим метод `RepLogApp.setSuccessMessage()`. А в `RepLogApp.handleAddRepLog()` будем его вызывать.
Это дает дополнительный оверхед. Так как сперва изменяется по таблице. А потом заново будет изменено состояние для successMessage.

При работе с таймаутами надо сбрасывать предыдущий счетчик.

Но тут надо создать еще один реактовский метод `componentWillUnmount`, который вызывается перед удалением компонента со страницы. 
У нас уже есть `componentDidMount` который вызывается после рендеринга компонента на странице.
```js
componentWillUnmount() {
    clearTimeout(this.successMessageTimeoutHandle);
}    
```


## 35. Updating Deep State Data

Если ответ по Ajax приходит пустой, у нас может быть ошибка. Так как мы получаем ответ от ajax в след коде:
```js
function fetchJson(url, options) {
    return fetch(url, Object.assign({
        credentials: 'same-origin',
    }, options))
        .then(response => {
            return response.json();
        });
}
```
И json может не распарситься.
Поэтому заменяем ответ на следующее:
```js
return response.text()
   .then(text => text ? JSON.parse(text) : '')
```
То есть парсим json самостоятельно, а не с использованием встроенного метода `response.json()`.

Добавим сообщение об удалении.

Ага. Нам надо указывать какое именно сообщение мы удалили.

Добавим у repLog поле isDeleted.

Сделаем перерендеринг перед отправкой ajax'a, где в нажатой строки добавиться поле isDeleted.
А после ajax'a - отрисуем новый список.

**!! замечание** для того чтобы создать клон объекта можно использовать конструкцию `Object.assign({}, repLog)`. То есть мы мержим в новый пустой объект все поля из repLog.

**!! замечание** Обратить вниманиие на `prevState.repLogs.map()`

```js
handleDeleteRepLog(id) {
    this.setState((prevState) => {
        return {
            repLogs: prevState.repLogs.map(repLog => {
                if (repLog.id !== id) {
                    return repLog;
                }

                return Object.assign({}, repLog, {isDeleting: true});
            })
        };
    });

    deleteRepLog(id).then(() => {
        // remove the rep log without mutating state
        // filter returns a new array
        this.setState((prevState) => {
            return {
                repLogs: prevState.repLogs.filter(repLog => repLog.id !== id)
            };
        });

        this.setSuccessMessage('Item was Un-lifted!');
    });
}
```

Само отобразение записи таблицы стилизуем:
```js
<tr
    key={repLog.id}
    className={highlightedRowId === repLog.id ? 'info' : ''}
    onClick={(event) => onRowClick(repLog.id)}
    style={{
        opacity: repLog.isDeleting ? .3 : 1
    }}
>
```


## 36. Server Validation & fetch Failing

Ранее говорилось про 3 типа валидации:
1. HTML5
2. Js - что значение должно быть больше 0
3. Server.

Серверная валидация нужна от хакеров. И если сервер вернул ошибку валидации - ReactJS должен ее показать.

У нас есть серверная валидация. `\App\Controller\RepLogController::newRepLogAction()`, `\App\Controller\BaseController::getErrorsFromForm()`.
Данные уходят в переменную 'errors'. И возвращается статус 400.

Для того, чтобы проверить валидацию сервера, в селекте отправим не существующее значение.

За ajax у нас отвечает один метод. `fetchJson()`. В нем добавим в цепочку промисов еще один `.then(checkStatus)`

Теперь будет кидаться исключение если статус ответа >= 400.


## 37. Displaying Server Validation Errors

По сути ошибка валидации - это не ошибка формы. И за это будет отвечать сам RepLogsApp.

По сути все тоже самое.


## 38. ...Object Rest Spread

Сейчас у нас при ошибке все равно остается loader и сбрасываются введенные данные. Надо фиксить.

Для отображения этого Лоадера используется `RepLogApp.state.isSavingNewRepLog: false`.

Мы можем это устанавиливать в каждом месте, где это надо. Но получиться много дублирования.
Вместо этого используем новую фичу языка.

Создадим newState для метода `create`. И его будем мержить с новыми необходимыми состояниями.

```js
return Object.assign({
    repLogs: newRepLogs,
    newRepLogValidationErrorMessage: '',
}, newState);
```

Проблема в том, что этот `Object.assign()`. Заюзаем spread. Но webpack его не может схавать. Добавим babel plugin.

```bash
yarn add babel-plugin-transform-object-rest-spread --dev
```

```js webpack.config.js
.configureBabel((babelConfig) => {
    babelConfig.plugins.push('transform-object-rest-spread');
})
```

Но Storm пока все равно ругается на этот спред. Добавим в линтер:
```js .eslintrc.js
module.exports = {
    parserOptions: {
        ecmaFeatures: {
            experimentalObjectRestSpread: true
        }
    },
};
```

Заменим везде `Object.assign()` на spread.


## 39. Passing Data from your Server to React

Посмотрим на RepLogCreator. Сейчас данные для селекта захардкожены. В реальности - это спец набор валидных данных
из backend'a.

Перенесем наши данные в умный компонент.

Есть 2 способа получить этот список от сервера.
Первый, как мы делали с RepLogs. В методе `componentDidMount()`. Для этого нам нужен ajax end point.
Второй, мы можем вывести в шаблоне twig'ом. А в js прочитать этот атрибут. Тогда не нужен ajax запрос. Этот метод подходит, так как у нас не меняются опции селекта.

Уберем `RepLogApp.state.itemOptions` и добавим `RepLogApp.props.itemOptions`. Но так как мы в компонент отображения 
прокидываем и пропсы и state, все будет работать.

Если у нас опции могут меняться часто. Можно сделать `RepLogApp.props.initialItemOptions`, а в state указать `state.itemProps: this.props.initialItemOptions`.

*Notice:*  Так как у нас в 

```js
// RepLogApp.js

render() {
    return <RepLog
        {...this.props}
        {...this.state}
        onRowClick={this.handleRowClick}
        onAddRepLog={this.handleAddRepLog}
        onHeartChange={this.handleHeartChange}
        onDeleteRepLog={this.handleDeleteRepLog}
    />;
```
state стоит после props, то state переопределит props, в случае если ключи будут совпадать.

Вообще, всегда стоит указывать props как обязательные. Но иногда этого не получается. 
А в случаем массива, если полу не обязательное, но в него передали null, может возникнуть ошибка.
Исправить это можно:

```js 
// RepLogApp.js

RepLogApp.defaultProps = {
    itemOptions: [],
}; 
```

Сейчас мы захардкодили массив в js. Далее нам надо вывести это в twig'e.


## 40. Passing Server Data to React Props

Данные для компонента отправляем в переменную `window.REP_LOG_APP_PROPS`

Удалим старый код.

Работает.

Время создать reusable component.


## 41. Reusable Components

Создадим переиспользуемые компоненты. Например компонент Button.

`assets/js/Components/Button.js`

Это будет Dump Presentation Component.

Чтобы прокидывать все атрибуты можно использовать spread `{...props}`.

Так же используем `{props.children}` чтобы отправить все что было внутри presentation.

Стоит обратить внимание на трюк с className. Так как мы хотим, чтобы переданный className добавлялся к дефолтному, но 
не перезаписывал его. Надо деструктировать props.

`const { className, ...otherProps } = props;` И передавать уже именно `otherProps`.


## 42. CSRF Protection Part 1

Для защиты от кросс доменных атак надо придерживаться 2х правил
1. Не позволять делать ajax запросы с других доменов.
2. Всегда используйте json, так как формы не умеют слать json.

CSRF атаки работают только
1 Session based authentication
2 HTTP basic authentication

Если использует API токен - все ОК.

Надо добавить `Content Type: application/json`


## 43. CSRF Protection Part 2

Переопределили `Annotation\Route` и используем его где наши actions - это api. 
Обратите внимание, что достаточно заюзать только в аннотации к самому классу контрллера.

В нашем Subscriber'e проверяем, что если у нас ApiRoute, а ContentType !== application/json - отдаем 415 код.

В нашем api.js надо, чтобы все запросы к серверу были с заголовком application/json.

КОнец!