# Задание 3  ·  ШРИ-2017

Мобилизация.Гифки – сервис для поиска гифок в перерывах между занятиями.

Сервис написан с использованием [bem-components](https://ru.bem.info/platform/libs/bem-components/5.0.0/).

Работа избранного в оффлайне реализована с помощью технологии [Service Worker](https://developer.mozilla.org/ru/docs/Web/API/Service_Worker_API/Using_Service_Workers).

Для поиска изображений используется [API сервиса Giphy](https://github.com/Giphy/GiphyAPI).

В браузерах, не поддерживающих сервис-воркеры, приложение так же должно корректно работать, 
за исключением возможности работы в оффлайне.

## Структура проекта

  * `gifs.html` – точка входа
  * `assets` – статические файлы проекта
  * `vendor` –  статические файлы внешних библиотек
  * `service-worker.js` – скрипт сервис-воркера

Открывать `gifs.html` нужно с помощью локального веб-сервера – не как файл. 
Это можно сделать с помощью встроенного в WebStorm/Idea веб-сервера, с помощью простого сервера
из состава PHP или Python. Можно воспользоваться и любым другим способом.

## Решение
Перенёс воркера в корень, чтобы он увидел остальные файлы приложения. Потом нашёл, где он регистрируется и поправил путь → `blocks.js:469`
  
Добавил `gifs.html` в функцию `needStoreForOffline`, чтобы тот кешировался → `service-worker.js:146`
  
Нашёл ошибку, из-за которой воркер ничего не фетчил, даже если была сеть — достаточно было поменять местами функции в операторе *ИЛИ*:
```javascript
// service-worker.js:41
if (needStoreForOffline(cacheKey)) {
    response = caches.match(cacheKey)
        .then(cacheResponse => fetchAndPutToCache(cacheKey, event.request) || cacheResponse);
…
```

Потом увидел событие `favorite:add`, но не увидел `favorite:remove`. Добавил, повесил обработчик — теперь гифки больше не остаются в кеше после того, как их удаляли из избранного. 

Ещё увеличил значение постоянной `CACHE_VERSION`, чтобы затриггерить функцию `deleteObsoleteCaches()`.

---

Выполнил дополнительное задание. Чтобы приложение могло переходить в офлайн-режим после первого запроса, просим его кешировать все файлы по самом первому событию, `install` → `service-worker.js:14`  
И, собственно, кешируем, самостоятельно реализовав такой метод:
```javascript
const assets = [ '/gifs.html', '//yastatic.net/jquery/3.1.0/jquery.min.js',
                 '/assets/blocks.js', '/assets/templates.js', '/assets/star.svg',
                 '/assets/style.css' ]

function preCacheAllAssets() {
    return caches.open(CACHE_VERSION)
        .then(cache => {
            return cache.addAll(assets)
    })
}
```
  
  
## Ответы на вопросы
```javascript
// Вопрос №1: зачем нужен этот вызов?
self.skipWaiting()
```
— Для активации нового воркера *сразу* после установки.
<br><br>
  
```javascript
// Вопрос №2: зачем нужен этот вызов?
self.clients.claim()
```
— Для того, чтобы воркер смог управлять всеми страницами — которые он может увидеть — сразу, без перезагрузки.
<br><br>

```javascript
// Вопрос №3: для всех ли случаев подойдёт такое построение ключа?
const cacheKey = url.origin + url.pathname
```
— Да, только если в урле не окажется какого-нибудь параметра.
<br><br>

```javascript
// Вопрос №4: зачем нужна эта цепочка вызовов?
return Promise.all(
    names.filter(name => name !== CACHE_VERSION)
        .map(name => {
            console.log('[ServiceWorker] Deleting obsolete cache:', name)
            return caches.delete(name)
        })
);
```
— Чтобы очищать весь кэш каждый раз, когда меняется постоянная CACHE_VERSION.
<br><br>

```javascript
// Вопрос №5: для чего нужно клонирование?
cache.put(cacheKey, response.clone())
```
— Для того, чтобы не потерять значение ответа. Яваскрипт считает `response` потоком, [не буферизирует его содержимое](https://jakearchibald.com/2014/reading-responses/) и, как следствие, не даёт обращаться к нему больше одного раза.
<br><br>
