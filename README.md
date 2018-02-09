# Шаг 0

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


Клиентский код мог бы выглядеть так:
```javascript
import { callApi } from 'core/redux-call-api';

export const getUser = (userId) => callApi({
    types: [...],
    inject: 'UserDataService',
    exec: (dataService) => dataService.getUser(userId)
});
```
