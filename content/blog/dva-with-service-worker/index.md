---
title: 在dva中添加Service Worker
date: "2019-05-08"
description: support service-worker with workbox in dva.
---

在使用[dva@2.x](https://github.com/dvajs/dva)处理数据流，以及[roadhog@2.x](https://github.com/sorrycc/roadhog)简化webpack配置的开发体系中，实现ServiceWorker离线缓存。  

> [workbox](https://developers.google.com/web/tools/workbox/guides/get-started)在Webpack中的使用

## 安装workbox-webpack-plugin
使用npm
```
npm install workbox-webpack-plugin --save-dev
```
使用yarn
```
yarn add workbox-webpack-plugin --dev
```

## 自定义service worker
自定义资源缓存策略、执行逻辑，更加灵活方便。新建文件`src/service-worker.js`，添加如下代码：
```javascript
// v4

workbox.core.setCacheNameDetails({
  prefix: 'my-app',
  suffix: 'v1',
});

workbox.core.skipWaiting();
workbox.core.clientsClaim();

workbox.precaching.precacheAndRoute(self.__precacheManifest || []);

// Register a navigation route
workbox.routing.registerNavigationRoute('/index.html', {
  blacklist: [new RegExp('/api/')],
});


// Handle API requests
workbox.routing.registerRoute(/\/api\//, new workbox.strategies.NetworkFirst());

// Handle third party requests
workbox.routing.registerRoute(
  url, // third party url
  new workbox.strategies.NetworkFirst()
);
```

如果你是v3版本，需要修改一下
```javascript{8,9,20,25}
// v3

workbox.core.setCacheNameDetails({
  prefix: 'my-app',
  suffix: 'v1',
});

workbox.skipWaiting();
workbox.clientsClaim();

workbox.precaching.precacheAndRoute(self.__precacheManifest || []);

// Register a navigation route
workbox.routing.registerNavigationRoute('/index.html', {
  blacklist: [new RegExp('/api/')],
});


// Handle API requests
workbox.routing.registerRoute(/\/api\//, workbox.strategies.networkFirst());

// Handle third party requests
workbox.routing.registerRoute(
  url, // third party url
  workbox.strategies.networkFirst()
);
```

workbox提供了`workbox.routing.registerRoute`API针对不同类型路由文件实现不同的缓存策略。在单页应用程序中，每当用户在浏览器进入你的网站时，该页面的请求将执行导航请求，我们使用`workbox.routing.registerNavigationRoute`API以缓存的HTML`index.html`响应所有导航请求。如果在项目中有通过下载地址下载文件或者其它类似的导航请求，需要考虑把地址匹配规则加入黑名单`blacklist`，避免点击下载按钮是从缓存的HTML返回响应。具体可查看[如何注册导航路由](https://developers.google.com/web/tools/workbox/modules/workbox-routing#how_to_register_a_navigation_route)、[什么是导航请求](https://developers.google.com/web/fundamentals/primers/service-workers/high-performance-loading#first_what_are_navigation_requests)

## 配置webpack
要在构建过程中生成service worker，你需要将插件添加到`webpack.config.js`文件中，如下所示：

```javascript
const { InjectManifest } = require('workbox-webpack-plugin');

module.exports = webpackConfig => {
  if (process.env.NODE_ENV === 'production') {
    webpackConfig.plugins.push(
      new InjectManifest({
        swSrc: './src/service-worker.js',
        importWorkboxFrom: 'local',
      })
    );
  }
  return webpackConfig;
};
```

## 注册service worker
新建文件`src/registerServiceWorker.js`，注册方法实现如下：

```javascript
const isLocalhost = Boolean(
  window.location.hostname === 'localhost' ||
    // [::1] is the IPv6 localhost address.
    window.location.hostname === '[::1]' ||
    // 127.0.0.1/8 is considered localhost for IPv4.
    window.location.hostname.match(/^127(?:\.(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)){3}$/)
);

export default function register() {
  if (process.env.NODE_ENV === 'production' && 'serviceWorker' in navigator) {
    window.addEventListener('load', () => {
      const swUrl = '/service-worker.js';

      if (!isLocalhost) {
        // Is not local host. Just register service worker
        registerValidSW(swUrl);
      } else {
        // This is running on localhost. Lets check if a service worker still exists or not.
        checkValidServiceWorker(swUrl);
      }
    });
  }
}

function registerValidSW(swUrl) {
  navigator.serviceWorker
    .register(swUrl)
    .then(registration => {
      registration.onupdatefound = () => {
        const installingWorker = registration.installing;
        installingWorker.onstatechange = () => {
          if (installingWorker.state === 'installed') {
            if (navigator.serviceWorker.controller) {
              // At this point, the old content will have been purged and
              // the fresh content will have been added to the cache.
              // It's the perfect time to display a "New content is
              // available; please refresh." message in your web app.
              console.log('New content is available; please refresh.');

              // Refresh current page to use the updated HTML and other assets
              window.location.reload();
            } else {
              // At this point, everything has been precached.
              // It's the perfect time to display a
              // "Content is cached for offline use." message.
              console.log('Content is cached for offline use.');
            }
          }
        };
      };
    })
    .catch(error => {
      console.error('Error during service worker registration:', error);
    });
}

function checkValidServiceWorker(swUrl) {
  // Check if the service worker can be found. If it can't reload the page.
  fetch(swUrl)
    .then(response => {
      // Ensure service worker exists, and that we really are getting a JS file.
      if (
        response.status === 404 ||
        response.headers.get('content-type').indexOf('javascript') === -1
      ) {
        // No service worker found. Probably a different app. Reload the page.
        navigator.serviceWorker.ready.then(registration => {
          registration.unregister().then(() => {
            window.location.reload();
          });
        });
      } else {
        // Service worker found. Proceed as normal.
        registerValidSW(swUrl);
      }
    })
    .catch(() => {
      console.log('No internet connection found. App is running in offline mode.');
    });
}

export function unregister() {
  if ('serviceWorker' in navigator) {
    navigator.serviceWorker.ready.then(registration => {
      registration.unregister();
    });
  }
}
```

然后在入口文件`src/index.js`里引入并注册
```javascript{1,5}
import registerServiceWorker from './registerServiceWorker';

// other code...

registerServiceWorker();
```
运行构建命令，查看dist目录

胡说八道~