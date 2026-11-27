# Swagger Petstore CRUD Regression

Навчальна API-регресія для Swagger Petstore v3. Postman Collection v2.1 послідовно створює, читає, оновлює та видаляє одну синтетичну тварину, а потім перевіряє, що вона недоступна. Колекція містить 18 перевірок `pm.test()`.

## Відповідність OpenAPI

Шляхи в таблиці включають префікс сервера `/api/v3`. Контракт звірено з офіційним вихідним OpenAPI Swagger Petstore (OpenAPI 3.0.4, версія API `1.0.30-SNAPSHOT`). [Вихідний OpenAPI](https://github.com/swagger-api/swagger-petstore/blob/master/src/main/resources/openapi.yaml) · [OpenAPI розгортання](https://petstore3.swagger.io/api/v3/openapi.json)

| Запит | Метод і шлях | Операція OpenAPI | Очікування сценарію | Інші описані відповіді |
| --- | --- | --- | --- | --- |
| CREATE | POST `/api/v3/pet` | [addPet](https://petstore3.swagger.io/#/pet/addPet) | `200` і JSON `Pet` із переданими значеннями | `400`, `422`, `default` |
| GET | GET `/api/v3/pet/{petId}` | [getPetById](https://petstore3.swagger.io/#/pet/getPetById) | `200` і JSON `Pet` із початковими значеннями | `400`, `404`, `default` |
| UPDATE | PUT `/api/v3/pet` | [updatePet](https://petstore3.swagger.io/#/pet/updatePet) | `200` і JSON `Pet` з тим самим ID та оновленими полями | `400`, `404`, `422`, `default` |
| GET AFTER UPDATE | GET `/api/v3/pet/{petId}` | [getPetById](https://petstore3.swagger.io/#/pet/getPetById) | `200` і JSON `Pet` зі збереженими змінами | `400`, `404`, `default` |
| DELETE | DELETE `/api/v3/pet/{petId}` | [deletePet](https://petstore3.swagger.io/#/pet/deletePet) | `200` | `400`, `default` |
| VERIFY DELETION | GET `/api/v3/pet/{petId}` | [getPetById](https://petstore3.swagger.io/#/pet/getPetById) | `404` | `200`, `400`, `default` |

- POST і PUT вимагають тіло `Pet`. Колекція надсилає `Content-Type: application/json` і запитує JSON через `Accept: application/json`. Обов'язкові поля `Pet` — рядок `name` і масив рядків `photoUrls`. Специфікація не задає мінімальної кількості елементів масиву, тому сценарій передає `[]`.
- `Pet.id` має тип `integer (int64)`. Для GET і DELETE параметр шляху `petId` обов'язковий і має той самий тип. Колекція використовує додатне ціле число в безпечному діапазоні JavaScript.
- `Pet.status` допускає `available`, `pending` або `sold`. Схема позначає `id` і `status` як необов'язкові поля моделі. Сценарій передає їх і перевіряє їхні значення для кореляції запису та перевірки оновлення.
- OpenAPI не задає схеми тіла успішної відповіді DELETE або відповіді GET зі статусом `404`. Для цих відповідей колекція перевіряє статус.

## Змінні

| Змінна | Область | Значення |
| --- | --- | --- |
| `baseUrl` | Середовище, з типовим значенням у колекції | `https://petstore3.swagger.io` без кінцевого слеша |
| `petId` | Колекція | ID із підтвердженої відповіді CREATE |
| `petName` | Колекція | Початкове синтетичне ім'я |
| `petStatus` | Колекція | Початковий статус `available` |
| `updatedPetName` | Колекція | Синтетичне ім'я після UPDATE |
| `updatedPetStatus` | Колекція | Статус після UPDATE, `pending` |

CREATE ініціалізує дані нового запуску. ID для POST зберігається в локальній змінній `requestedPetId`. Подальші запити використовують `petId`, лише коли CREATE підтвердив ID, ім'я та статус. Якщо CREATE не підтвердив запис, решта запитів пропускається, а його невдалі перевірки залишаються у звіті. Очікувані значення UPDATE формуються до відправлення й використовуються також у GET AFTER UPDATE.

## Запуск у Postman

1. Імпортуйте `postman/Petstore_CRUD.postman_collection.json` та `postman/Petstore.postman_environment.json` через **Import**.
2. Виберіть середовище **Swagger Petstore**.
3. Відкрийте **Run collection** і запустіть усі шість запитів від CREATE в заданому порядку.

## Запуск через Newman

У корені репозиторію виконайте наведені команди. Перевірено з Node.js 24 та npm 11. Команда `npm test` надсилає CRUD-запити до публічного Swagger Petstore і змінює тестовий запис у ньому.

```bash
npm ci
npm test
```

Newman друкує CLI-звіт і записує JSON у `reports/newman.json`, а JUnit XML у `reports/newman.xml`. Невдала перевірка дає ненульовий код завершення. Для власного тестового сервера з таким самим API передайте його адресу через `npm test -- --env-var "baseUrl=http://127.0.0.1:8080"`.

## Обмеження і Needs Clarification

- **Автентифікація.** OpenAPI оголошує OAuth2 для POST, PUT і DELETE та альтернативу API key або OAuth2 для GET. Колекція не містить облікових даних. Застосування цих вимог у публічному демо потребує підтвердження під час дозволеного запуску. Параметр заголовка `api_key` у DELETE описаний як необов'язковий окремо від OAuth2.
- **Тіло DELETE.** Для `200` документується статус і опис «Pet deleted». Наявність або формат тіла потребують уточнення, якщо на них має спиратися клієнт.
- **Спільне середовище.** Випадковий ID зменшує ймовірність конфлікту. Інші користувачі або скидання демо можуть змінити дані під час тесту. Перерваний запуск або невдале видалення можуть залишити запис. Якщо після DELETE GET поверне щось інше за `404`, Newman зафіксує невдалу перевірку.
