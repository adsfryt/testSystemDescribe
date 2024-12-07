# Описание каждого модуля
## Модуль Авторизации

Модуль должен отвечать на все эти запросы:

### Функции
-------------
> GET /user/link_reg_yandex
#### Принимает параметры:
| Имя    | Обязательно | Тип |
| -------- | ------- | ------- |
| none  |     |       |

#### Возвращает параметры:
| Имя    | Обязательно | Тип |
| -------- | ------- | ------- |
| reg_user  |   да  |   сторка   |
| state |   да  |   сторка   |

#### Описание
Должен вернуть ссылку типа:
```
https://oauth.yandex.ru/authorize?response_type=code&client_id=44c728c0ab8f455784e845387e2c9bd8&state=<state>
```
По которой должен перейти пользователь и подтвердить передачу данных<br>

state - набор случайных символов, по которому система затем идентефецирует пользователя, так что его нужно запомнить.
подробнее здесь: https://yandex.ru/dev/id/doc/ru/codes/code-url

-------------
> GET /user/reg_yandex
#### Принимает параметры:
| Имя    | Обязательно | Тип |
| -------- | ------- | ------- |
| state  |   да  |  сторка   |
| code  |   да  |  сторка   |

#### Возвращает параметры:
| Имя    | Обязательно | Тип |
| -------- | ------- | ------- |
| refresh_token  |   да  |   сторка   |
| access_token |   да  |   сторка   |
| activate |   нет  |   булевый   |

#### Описание

Модуль принимает <code>state</code> пользователя, проверяет его наличие (когда он был создан про создании ссылки). Далее обмениваем токен на два ключа доступа [подробнее](https://yandex.ru/dev/id/doc/ru/codes/code-url#token) 
(access_token, refresh_token) и с их помощью модуль получает информацию о пользователех [подробнее](https://yandex.ru/dev/id/doc/ru/user-information) типа:
```json
{
   "login": "ivan",
   "old_social_login": "uid-mmzxrnry",
   "default_email": "test@yandex.ru",
   "id": "1000034426",
   "client_id": "4760187d81bc4b7799476b42b5103713",
   "emails": [
      "test@yandex.ru",
      "other-test@yandex.ru"
   ],
   "psuid": "1.AAceCw.tbHgw5DtJ9_zeqPrk-Ba2w.qPWSRC5v2t2IaksPJgnge"
}
```
Нас интересуют поля <code>login</code>, <code>id</code> и <code>default_email</code>.<br>
Генерируем два ключа <code>access_token</code>, <code>refresh_token</code>.<br>
Ищем в базе данных пользователя с полем userId равным: "yandex"+<code>id</code><br>
* Если нашли, то проверяем поле "activate" и если оно равно 1, то просто возвращем сайту <code>access_token</code> и <code>refresh_token</code> [подробнее](https://www.youtube.com/watch?v=IB9gsnlRt-4) (дополнительно зависит от языка программирования и библиотеки) . А если равен 0, то добавляеи в ответ поле <code>activate</code> со значение 1.
* Есои не нашли то, создаем запись с данными пользователя в базе данных mongodb, где <code>"/login"</code> = <code>login</code>, <code>"/id"</code> = <code>id</code>, <code>"/default_email"</code> = <code>default_email</code>,
<code>"/refresh_token"</code> = <code>refresh_token</code>, <code>"/state"</code> = <code>state</code>:
```json
{
  "login": "yandex/login",
  "userId": "yandex/id",
  "email": "/default_email",
  "state": "/state",
  "role": -1,
  "activate": 0,
  "disciplinesId": [],
  "method": "yandex",
  "refresh_token": "refresh_token",
  "access_token": "access_token",
  "refresh_token": [ "/refresh_token" ] 
}
```
Полученные ранее <code>access_token</code>, <code>refresh_token</code> и <code>activate</code> со значение 1 возвращаем пользователю.

-------------
> GET /user/activate_reg_yandex
#### Принимает параметры:
| Имя    | Обязательно | Тип |
| -------- | ------- | ------- |
| role  |   да  |  целое число (1 или 2)   |
| access_token  |   да  |  сторка   |

#### Возвращает параметры:
| Имя    | Обязательно | Тип |
| -------- | ------- | ------- |
| ok  |   да  |   булевый   |

#### Описание

Модуль проверяет <code>access_token</code> и<code>role</code> на корректность. Получив из данных <code>access_token</code>а id пользователя находим его в базе данных mongodb. Устанавливаем ему соответствующую роль и меняем поле
"activate" на 1. Если база данных смогда изменить поля и вернула значение, то возвращаем поле <code>ok</code> со значение true, иначе false.



-------------
> GET /user/link_reg_github
#### Принимает параметры:
| Имя    | Обязательно | Тип |
| -------- | ------- | ------- |
| none  |       |       |

#### Возвращает параметры:
| Имя    | Обязательно | Тип |
| -------- | ------- | ------- |
| reg_user  |   да  |   сторка   |
| state |   да  |   сторка   |

#### Описание
* Процесс аналогичный с регистрации через яндекс

Должен вернуть ссылку типа:
```
https://github.com/login/oauth/authorize?client_id=Ov23li5Ud4OIJbz3CExS&state=<state>
```
