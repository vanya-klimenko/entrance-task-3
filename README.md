# Задание 3

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

# 
```javascript
		// Вопрос №1: зачем нужен этот вызов?
			.then(() => self.skipWaiting())
		// Вопрос №2: зачем нужен этот вызов?
			self.clients.claim();
		// Вопрос №3: для всех ли случаев подойдёт такое построение ключа?
			const cacheKey = url.origin + url.pathname;
		// Вопрос №4: зачем нужна эта цепочка вызовов?
			return Promise.all(
			    names.filter(name => name !== CACHE_VERSION)
			        .map(name => {
			            console.log('[ServiceWorker] Deleting obsolete cache:', name);
			            return caches.delete(name);
			        })
			);
		// Вопрос №5: для чего нужно клонирование?
			cache.put(cacheKey, response.clone());
		
```