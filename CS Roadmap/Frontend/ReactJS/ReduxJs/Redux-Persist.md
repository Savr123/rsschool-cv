#### Основы.
Redux - in-memory state , поэтому каждый раз при перезагрузке страницы, состояние будет сброшено. Для сохранения состояния можно хранить его в localStorage хранилище. Для его хранения можно использовать библиотеку redux-persist.

Команда для установки пакета Redux-Persist через **npm** пакетный менеджер:

```
npm i redux-persist
```

Для использования Redux-Persist, нужно провести модификации над сущностью Store, в которой хранится всё состояние приложения:
Пример 1.1:
```
// src/redux/store.js
import { configureStore } from "@reduxjs/toolkit";
import { persistStore, persistReducer } from "redux-persist";
import storage from "redux-persist/lib/storage";
import userReducer from "./slices/userSlice";

const persistConfig = {
  key: "root",
  storage,
};

const persistedReducer = persistReducer(persistConfig, userReducer);
export const store = configureStore({
  reducer: persistedReducer,
  devTools: process.env.NODE_ENV !== "production",
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoredActions: ["persist/PERSIST", "persist/REHYDRATE"],
      },
    }),
});

export const persistor = persistStore(store);
```

Представленный код выше будет сохранять состояние в `localStorage`, выбор хранилища находится в конфигурации `persistConfig`. Кроме `localStorage` можно также сохранять состояние в `sessionStorage` и `Redux Persist Cookie Adapter Storage`.
Для выбора другого хранилища нам нужно изменить значение `storage` в конфигурации на нужное нам. Пример конфигурации 1.2:
```
import storageSession from 'redux-persist/lib/storage/session'

const persistConfig = {
  key: 'root',
  storageSession,
}
```
Ранее в примере 1.1 в конфигурации `store` мы добавили `getDefaultMiddleware`
```
middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoredActions: ["persist/PERSIST", "persist/REHYDRATE"],
      },
    }),
```
Это было сделано поскольку Redux-Persist автоматически устанавливает несколько middleware, которые могут включать non-serializable сущности в период восстановления состояния в консоли.

В большинстве случаев нам может потребоваться задержка перед рендером пользовательского интерфейса до тех пор пока состояние не восстановится и не будет доступна в `Redux store`. Для таких случаев в Redux-Persist есть компонента PersistGate. Для его использования нам нужно перейти в index.js файл и добавить следующий `import`;
```
// src/index.js
import { persistor, store } from './redux/store';
import { PersistGate } from 'redux-persist/integration/react';

root.render(
  <React.StrictMode>
    <Provider store={store}>
      <PersistGate loading={null} persistor={persistor}>
        <App />
      </PersistGate>
    </Provider>
  </React.StrictMode>
);
```

#### Продвинутые возможности использования Redux-Persist:
##### Nested Persists.
Бывают случаи когда нам требуется несколько ==reducers==, для подобных ситуаций обычно используются ==combineReducers==. Но что если нам нужно использовать разные конфигурации для разных reducers? Например мы хотим изменить хранилище для одного из ==reducer== на ==sessionStorage==. Для решения такой задачи нам потребуется использовать ==nested persists==. Фича, которая позволяет строить вложенные ==persistReducer==, предоставляя нам возможность настраивать ==reducer== так как нам нужно.
Например:
```
const rootPersistConfig = {
  key: 'root',
  storage,
}

const userPersistConfig = {
  key: 'user',
  storage: storageSession,
}

const rootReducer = combineReducers({
  user: persistReducer(userPersistConfig, userReducer),
  notes: notesReducer
})

const persistedReducer = persistReducer(rootPersistConfig, rootReducer)

const store = configureStore({
  reducer: persistedReducer
})
```
##### Трансформация хранимой информации через Redux-Persist
Могут возникнуть ситуации, где вы захотите изменить как определенные участки вашего состояния хранятся. Изменение хранимой информации становится важной в таких случаях, потому что это дает вам контроль над хранением, способом хранения и восстановлением состояния. 



> [!NOTE]
> Ссылки:
> 1. https://blog.logrocket.com/persist-state-redux-persist-redux-toolkit-react/
> 
> 