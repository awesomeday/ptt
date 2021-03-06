## Шаг 0
```javascript
export const getUser = (userId) => ({
    callApi: {
        types: [
            ActionTypes.USER_FETCH_START, 
            ActionTypes.USER_FETCH_SUCCESS, 
            ActionTypes.USER_FETCH_ERROR
        ],
        method: 'get',
        url: '/user',
        options: {
            params: { userId }
        }
    }
});
```

## Шаг 1. Использование сервисов
```diff
export const getUser = (userId) => ({
    callApi: {
        types: [
            ActionTypes.USER_FETCH_START, 
            ActionTypes.USER_FETCH_SUCCESS, 
            ActionTypes.USER_FETCH_ERROR
        ],
+       serviceName: 'UserService',
+       method: 'getUser',
+       params: [userId]
    }
});
```

## Шаг 2
Вызов функции
```diff
export const getUser = (userId) => ({
    callApi: {
        types: [...],
        serviceName: 'UserService',
+       exec: (UserService) => UserService.getUser(userId)
    }
});
```

Теперь вместо раздельной передачи строки с именем метода и параметров для него мы имеем полноценный вызов функции. А если в проекте используется TypeScript, мы получаем полноценную поддержку от IDE: подсказки имен методов и параметров, подсветка ошибок, навигация по коду, рефакторинг.


## Шаг 3
Теперь то, что мы пишем в поле `serviceName` инжектится в нашу функцию. Логично переименовать это поле в `inject` (в лучших традициях Angular). Теперь напрашивается очевидное улучшение - добавить возможность передавать в `inject` не только строку, но и массив. Это позволит инжектить в функцию любые зависимости. Например так:
```diff
export const getUser = (userId) => ({
    callApi: {
        types: [...],
+       inject: ['UserService', 'UserFormatter'],
        
+       exec: (UserService, UserFormatter) =>
+           UserService.getUser(userId).then(UserFormatter.format)
    }
});
```


## Шаг 4. Сокрытие реализации
Такой способ создания объекта несколько многословен и в какой-то мере является нарушением инкапсуляции: если бы наш middleware поставлялся как сторонняя библиотека, получалось бы, что мы используем особенности внутренней реализации. Было бы лучше, если бы сама библиотека предоставляла функцию-фабрику, которая конструировала бы для нас объекты с нужной структурой.

Клиентский код мог бы выглядеть так:
```javascript
import { callApi } from 'core/redux-call-api';

export const getUser = (userId) => callApi({
    types: [...],
    inject: 'UserDataService',
    exec: (dataService) => dataService.getUser(userId)
});
```
Код остался читаемым, и стал чуть более компактным. При этом если будет необходимо поменять внутреннюю структуру объекта, это можно будет сделать в одном месте.


## Шаг 5. thunk-middleware на стероидах

```javascript
import { inject } from 'core/redux-inject';

// где-то тут реализация userFetchStart(), userFetchSuccess(), userFetchError()

export const getUser = (userId) => callApi({
    inject: 'UserDataService',
    exec: (dataService) => (dispatch) => {
        dispatch(userFetchStart());
        
        dataService.getUser(userId)
            .then((user) => dispatch(userFetchSuccess(user)))
            .catch((error) => dispatch(userFetchError(error)));
    }
});
```

```javascript
import { inject } from 'core/redux-inject';

// где-то тут реализация userFetchStart(), userFetchSuccess(), userFetchError()

export const getUser = (userId) => inject('UserDataService',
    (dataService) => (dispatch) => {
        dispatch(userFetchStart());
        
        dataService.getUser(userId)
            .then((user) => dispatch(userFetchSuccess(user)))
            .catch((error) => dispatch(userFetchError(error)));
    }
);
```
Этот подход является компромиссным - с одной стороны кода получается больше, с другой - мы получаем все преимущества thunk-middleware в виде полного контроля над обработкой данных.
