# Шаг 0
Наивный подход
```javascript
export const getUser = (userId) => {
    callApi: {
        types: [USER_FETCH_START, USER_FETCH_SUCCESS, USER_FETCH_ERROR],
        method: 'get',
        url: '/user',
        options: {
            params: { userId }
        }
    }
};
```

# Шаг 1
Сервис
```javascript
export const getUser = (userId) => {
    callApi: {
        types: [USER_FETCH_START, USER_FETCH_SUCCESS, USER_FETCH_ERROR],
        serviceName: 'UserService',
        method: 'getUser',
        params: [userId]
    }
};
```

# Шаг 2
Вызов функции
```javascript
export const getUser = (userId) => {
    callApi: {
        types: [USER_FETCH_START, USER_FETCH_SUCCESS, USER_FETCH_ERROR],
        inject: 'UserService',
        exec: (UserService) => UserService.getUser(userId)
    }
};
```

# Шаг 3
Множественные инжекты
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
