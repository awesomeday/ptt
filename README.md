# Шаг 0
Наивный подход
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

# Шаг 1
Сервис
```javascript
export const getUser = (userId) => ({
    callApi: {
        types: [
            ActionTypes.USER_FETCH_START, 
            ActionTypes.USER_FETCH_SUCCESS, 
            ActionTypes.USER_FETCH_ERROR
        ],
        serviceName: 'UserService',
        method: 'getUser',
        params: [userId]
    }
});
```

# Шаг 2
Вызов функции
```javascript
export const getUser = (userId) => ({
    callApi: {
        types: [...],
        inject: 'UserService',
        exec: (UserService) => UserService.getUser(userId)
    }
});
```

# Шаг 3
Напрашивается очевидное улучшение - разрешить передавать в `inject` не только строку, но и массив. Это позволит инжектить в функцию любые зависимости. Например так:
```javascript
export const getUser = (userId) => ({
    callApi: {
        types: [...],
        inject: ['UserService', 'UserFormatter'],
        exec: (UserService, UserFormatter) =>
            UserService.getUser(userId).then(UserFormatter.format) // но лучше это делать в самом UserService
    }
});
```
# Шаг 4
Обертка callApi
Этот способ создания объекта несколько многословен и в какой-то мере является нарушением инкапсуляции: если бы наш middleware поставлялся как сторонняя библиотека, получалось бы, что мы используем особенности внутренней реализации. Было бы лучше, если бы сама библиотека предоставляла функцию-фабрику, которая конструировала бы для нас объекты с нужной структурой.

Клиентский код мог бы выглядеть так:
```javascript
import { callApi } from 'core/redux-call-api';

export const getUser = (userId) => callApi({
    types: [...],
    inject: 'UserDataService',
    exec: (dataService) => dataService.getUser(userId)
});
```

# Шаг 5
Inject middleware

```javascript
import { inject } from 'core/redux-inject';

const userFetchStart = () => ({ type: USER_FETCH_START });
const userFetchSuccess = (user) => ({ type: USER_FETCH_SUCCESS, payload: user });
const userFetchError = (error) => ({ type: USER_FETCH_ERROR, payload: error });

export const getUser = (userId) => inject(
    'UserDataService',
    
    (dataService) => (dispatch) => {
        dispatch(userFetchStart());
        
        dataService.getUser(userId)
            .then((user) => dispatch(userFetchSuccess(user)))
            .catch((error) => dispatch(userFetchError(error)));
    }
);
```
