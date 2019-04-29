# zabbix-react



## Обзор
Проект по созданию React-компонент для работы с Zabbix-API.
Компоненты общаяются с API в соответствии с [OpenAPI-Spec](https://github.com/OAI/OpenAPI-Specification) спецификацией, на базе которой можно собрать как API-сервер, так и SDK-клиента.

![Image](https://github.com/ars-anosov/zabbix-react/blob/master/images/drawio_main.png)

### Компоненты
[<img src="https://github.com/npm/logos/blob/7fb0bc425e0dac1bab065217c4ed595594448db4/npm-transparent.png" height="20" alt="npm">](https://www.npmjs.com)

[zabbix-react-component](https://www.npmjs.com/package/zabbix-react-component)

### Примеры
![Image](https://github.com/ars-anosov/zabbix-react/blob/master/images/demo_screen.png)
![Image](https://github.com/ars-anosov/zabbix-react/blob/master/images/demo_graph.png)

- [Demo](http://109.173.22.60/www-zabbix-component/)
- [zabbix-server demo](http://109.173.22.60/zabbix-server/hostinventories.php) (пользователь guest)
- [API demo](http://109.173.22.60:8002/spec-ui/) (token test)



## Цель
1. Предоставить FrontEnd в виде [React-компоненты](https://github.com/ars-anosov/zabbix-react/tree/master/web-front)
2. Взаимодействие Front-Back свести до простейших REST-запросов
3. Сосредоточить всю логику взаимодействия с Zabbix-сервером на BackEnd - [zabbix-reactor](https://github.com/ars-anosov/zabbix-react/tree/master/node-back)

## Настройка Zabbix-сервера
Мы хотим открыть доступ только к определенным Host group на Zabbix. Для этого необходимо создать отдельного пользователя «react_user».

- ***Administration/User groups***: добавляем группу пользователей ***react_user_group***
- ***Administration/Users***: добавляем пользователя ***react_user***, включаем в группу пользователей react_user_group
- ***Administration/User groups/Permissions***: добавляем ***Host groups***, с которыми будет позволено работать «react_user». Выставляем ***Read-write***. Именно здесь ограничиваем возможности React компонент по воздействию на Zabbix.



## Установка / Использование
Я собирал на своей машине **IP=192.168.13.97**, все ссылки относительно этого IP.



### 1. Back: zabbix-reactor
OpenAPI-сервер обрабатывает REST-запросы от React-компонент. Общается с Zabbix-API.  

В директории [node-back](https://github.com/ars-anosov/zabbix-react/tree/master/node-back) описана установка.

Результат:
- WEB-интерфейс для тестовых запросов через Swagger-UI - [192.168.13.97:8002/spec-ui/](http://192.168.13.97:8002/spec-ui/)
- OpenAPI-Spec файл доступен - [192.168.13.97:8002/spec/swagger.json](http://192.168.13.97:8002/spec/swagger.json)
- API принимает REST-запросы - [192.168.13.97:8002/v2api](http://192.168.13.97:8002/v2api/)

В поле "token" вписываем "test".



### 2. Front: zabbix-react-component



#### 2.1. Через gulp в Docker-контейнере NodeJS
```
docker build -t 'zabbix-react-front:latest' github.com/ars-anosov/zabbix-react#:web-front

docker run \
  --name zabbix-react-front \
  --publish=8003:8003 \
  -it \
  zabbix-react-front:latest
```
Выскочить из контейнера : Ctrl+P+Q

Живые компоненты - [192.168.13.97:8003](http://192.168.13.97:8003/)



#### 2.2. В своем React bundler
Например через [create-react-app](https://reactjs.org/tutorial/tutorial.html)
```bash
mkdir ~/react-app && cd ~/react-app

docker run \
  --name react-app \
  --publish=3000:3000 \
  -v $PWD:/my-app \
  -w / \
  -it \
  node:8 bash

# Дальше все действия в контейнере:
npm install -g create-react-app
create-react-app my-app
cd my-app
rm -f src/*

yarn add zabbix-react-component

# Редактируем файлы. См. ниже.
touch src/index.js
touch src/index.css

npm start
```

- Содержимое **index.js**

```js
import React from 'react';
import ReactDOM from 'react-dom'

import { OpenApiSwagger, HostConfig, HostGraph } from 'zabbix-react-component'
import './index.css';

window.localStorage.setItem('token', 'test')

const specUrl = 'http://192.168.13.97:8002/spec/swagger.json'
const swg = new OpenApiSwagger(specUrl)

swg.connect((client, err) => {
  if (err) {
    // Error actions
  }
  else {
    ReactDOM.render(
      <div>
        <HostConfig swgClient={client} headerTxt='HostConfig component' />
        <HostGraph swgClient={client} headerTxt='HostGraph component' />
      </div>,
      document.getElementById('root')
    )
  }
})
```

- Содержимое **index.css** копируем отсюда [examples/css](https://github.com/ars-anosov/zabbix-react/tree/master/examples/css)

Тыкаем компоненты на локальном web-сервере - [192.168.13.97:3000](http://192.168.13.97:3000/)









## Пилим проект

### zabbix-reactor
- [node-back](https://github.com/ars-anosov/zabbix-react/tree/master/node-back): Swagger-сервер "zabbix-reactor-node": 

### zabbix-react-component
- [web-front](https://github.com/ars-anosov/zabbix-react/tree/master/web-front): livereload-сервер с компонентами
- [web-front/src/js/components](https://github.com/ars-anosov/zabbix-react/tree/master/web-front/src/js/components): содержимое npm пакета "zabbix-react-component"

### OpenAPI spec - файл

#### [swagger-editor](https://github.com/swagger-api/swagger-editor)
```
docker run \
  --publish=8001:8080 \
  --name=swagger-editor \
  swaggerapi/swagger-editor
```
Пишем файл спецификации - [192.168.13.97:8001](http://192.168.13.97:8001/), там же генерим Server API / Client SDK

#### [swagger-codegen](https://github.com/swagger-api/swagger-codegen#development-in-docker)
Любителям хардкора.

Компилируем 
```
git clone https://github.com/swagger-api/swagger-codegen
cd swagger-codegen
./run-in-docker.sh mvn package
```

Генерим Server API
```
cp node-back/api/zabbix-api.yaml swagger-codegen/ && \
swagger-codegen/run-in-docker.sh generate \
  -i zabbix-api.yaml \
  -l nodejs-server \
  -o /gen/out/node-back
```

Генерим Client SDK
```
cp node-back/api/zabbix-api.yaml swagger-codegen/ && \
swagger-codegen/run-in-docker.sh generate \
  -i zabbix-api.yaml \
  -l javascript 
  -o /gen/out/web_front
```

Results
```
ls -la swagger-codegen/out/
cp -r swagger-codegen/out/web-front/src/ web-front/src/js/partails/swagger_client/
```
