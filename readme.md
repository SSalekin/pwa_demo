# PWA
Progressive web applications (PWAs) are web applications that load like regular web pages or websites but can offer the user functionality such as working offline, push notifications, and device hardware access traditionally available only to native applications. PWAs combine the flexibility of the web with the experience of a native application. - [wikipedia](https://en.wikipedia.org/wiki/Progressive_web_applications). 

PWAs are light weight (only few kb installation size), supports all app platforms (google, apple, microsoft, linux), uses a fraction of the bandwidth of the native apps and are easier to build.
One great thing is you can use any modern web frameworks (react, angular, vue, polymer, preact, mithril or just pure vanilla js) to create a pwa.

Here is HackerNews implementation and benchmarks of PWAs using all modern web frameworks. 
#### [HNPWA](https://hnpwa.com/)

# Techs in use

### ServiceWorker
Service workers are proxies that handle the communication between our App and the Server. It's nothing but a simple js file that the browser runs in the background, separate from the main thread. More about ServiceWorkers [here](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API).

Let's create a demo project

```bash
mkdir pwa_demo
cd pwa_demo
touch index.html app.js sw.js
```
We create a very basic html page just to show when we run the server.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>PWA Demo</title>
  </head>
  <body>    
    <script src="app.js"></script>
  </body>
</html>
```

now, we modify our app.js file

First we make sure ServiceWorker is supported in our browser
```JavaScript
(() => {

  if('serviceWorker' in navigator){
    console.log("Service Worker is supported.")
  }

})();
```

![](https://images.viblo.asia/a7a29550-bde9-4549-b7be-40c98bd01222.png)

Now we modify the `sw.js` file and see if it runs
```javascript
(() => {
  self.addEventListener('fetch', e => {
    console.log("Fetching Data through sw")
  });
})();
```
![](https://images.viblo.asia/9346f494-5ffa-4e9b-8fc4-82d091cf3971.png)


### Manifest
Manifest files are used to turn your webpage to similar to a native app
to add them in you need to create a json file and link using
`<link rel="manifest" href="manifest.json">`

You can create the json by hand, but there are many helping websites like [App manifest generator](https://app-manifest.firebaseapp.com/)
Let'f fill up the form and generate a manifest
![](https://images.viblo.asia/5765ee5a-525a-4a7b-a13f-bd4c2b699985.png)

Now, after downloading the zip, this is what you'll get as your manifest
![](https://images.viblo.asia/663a266e-f19b-4e8a-abd9-2b0305b6a8c4.png)

Here's the generated manifest file
```json
{
  "name": "PWA_Demo",
  "short_name": "pwa",
  "theme_color": "#2196f3",
  "background_color": "#2196f3",
  "display": "standalone",
  "Scope": "/",
  "start_url": "/",
  "icons": [
    {
      "src": "images/icons/icon-72x72.png",
      "sizes": "72x72",
      "type": "image/png"
    },
    {
      "src": "images/icons/icon-96x96.png",
      "sizes": "96x96",
      "type": "image/png"
    },
    {
      "src": "images/icons/icon-128x128.png",
      "sizes": "128x128",
      "type": "image/png"
    },
    {
      "src": "images/icons/icon-144x144.png",
      "sizes": "144x144",
      "type": "image/png"
    },
    {
      "src": "images/icons/icon-152x152.png",
      "sizes": "152x152",
      "type": "image/png"
    },
    {
      "src": "images/icons/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "images/icons/icon-384x384.png",
      "sizes": "384x384",
      "type": "image/png"
    },
    {
      "src": "images/icons/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ],
  "splash_pages": null
}
```

and modify our `index.html`
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>PWA Demo</title>
    <link rel="manifest" href="manifest.json">
  </head>
  <body>    
    <script src="app.js"></script>
  </body>
</html>
```

# Demo
We will use a News reading app with vanilla js using the free [Newsapi.org]() api

#### index.html
```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <title>News</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">

  <link rel="stylesheet" href="styles.css">
  <link rel="manifest" href="manifest.json">
  <meta name="theme-color" content="#ffffff" />
</head>

<body>
  <header>
    <h1>News</h1>
    <select id="sources"></select>
  </header>
  <main></main>
  <script src="app.js"></script>
</body>

</html>
```

#### app.js
```javascript
(
  () => {
    const apiKey = 'sign up and use your own token here';
    const defaultSource = 'the-washington-post';
    const sourceSelector = document.querySelector('#sources');
    const newsArticles = document.querySelector('main');

    if ('serviceWorker' in navigator) {
      window.addEventListener('load', () =>
        navigator.serviceWorker.register('sw.js')
          .then(registration => console.log('Service Worker registered'))
          .catch(err => 'SW registration failed'));
    }

    window.addEventListener('load', e => {
      sourceSelector.addEventListener('change', evt => updateNews(evt.target.value));
      updateNewsSources().then(() => {
        sourceSelector.value = defaultSource;
        updateNews();
      });
    });

    window.addEventListener('online', () => updateNews(sourceSelector.value));

    async function updateNewsSources() {
      const response = await fetch(`https://newsapi.org/v2/sources?apiKey=${apiKey}`);
      const json = await response.json();
      sourceSelector.innerHTML =
        json.sources
          .map(source => `<option value="${source.id}">${source.name}</option>`)
          .join('\n');
    }

    async function updateNews(source = defaultSource) {
      newsArticles.innerHTML = '';
      const response = await fetch(`https://newsapi.org/v2/top-headlines?sources=${source}&sortBy=top&apiKey=${apiKey}`);
      const json = await response.json();
      newsArticles.innerHTML =
        json.articles.map(createArticle).join('\n');
    }

    function createArticle(article) {
      return `
        <div class="article">
          <a href="${article.url}">
            <h2>${article.title}</h2>
            <img src="${article.urlToImage}" alt="${article.title}">
            <p>${article.description}</p>
          </a>
        </div>
      `;
    }
  }
)();
```

#### sw.js

```javascript
(
  () => {
    const cacheName = 'news-v1';
    const staticAssets = [
      './',
      './app.js',
      './styles.css',
      './fallback.json',
      './images/fallback_image.jpg'
    ];

    self.addEventListener('install', async function () {
      const cache = await caches.open(cacheName);
      cache.addAll(staticAssets);
    });

    self.addEventListener('activate', event => {
      event.waitUntil(self.clients.claim());
    });

    self.addEventListener('fetch', event => {
      const request = event.request;
      const url = new URL(request.url);
      if (url.origin === location.origin) {
        event.respondWith(cacheFirst(request));
      } else {
        event.respondWith(networkFirst(request));
      }
    });

    async function cacheFirst(request) {
      const cachedResponse = await caches.match(request);
      return cachedResponse || fetch(request);
    }

    async function networkFirst(request) {
      const dynamicCache = await caches.open('news-dynamic');
      try {
        const networkResponse = await fetch(request);
        dynamicCache.put(request, networkResponse.clone());
        return networkResponse;
      } catch (err) {
        const cachedResponse = await dynamicCache.match(request);
        return cachedResponse || await caches.match('./fallback.json');
      }
    }
  }
)();
```

#### fallback.json
```json
{
  "articles": [
    {
      "title": "You are out of dragon balls (or either mobile data) ¯\\_(ツ)_/¯",
      "url": "",
      "urlToImage": "images/fallback_image.jpg",
      "description": "Refresh page after a while"
    }
  ]
}
```


# Tools and resouces

[Resource from Google](https://developers.google.com/web/progressive-web-apps/)

[PWA Builder](https://www.pwabuilder.com/)

[App manifest generator](https://app-manifest.firebaseapp.com/)

[WorkBox](https://developers.google.com/web/tools/workbox/)

[LightHouse](https://developers.google.com/web/tools/lighthouse/)

[Your first PWA](https://codelabs.developers.google.com/codelabs/your-first-pwapp/#0)

[PWA Checklist](https://developers.google.com/web/progressive-web-apps/checklist)

Source code was taken from:
{@embed: https://www.youtube.com/embed/gcx-3qi7t7c}