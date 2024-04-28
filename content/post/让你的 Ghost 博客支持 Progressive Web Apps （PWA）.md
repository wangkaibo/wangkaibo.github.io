---
title: "让你的Ghost博客支持Progressive Web Apps (PWA)"
date: 2017-09-30T01:13:02+08:00
tags: ["PWA"]
draft: false
url: '/node-express-ghost-progressive-web-apps-pwa/'
---

### Ghost 博客支持 Progressive Web Apps （PWA）

一直对 Google 的 Progressive Web Apps （以下简称 PWA） 有点兴趣，所以拿我的博客做实验，让 Ghost 博客支持 PWA。

PS：如果你的博客还不支持 https 的话，那么也就不支持 Service Worker，也就是说实现不了 Progressive Web Apps。

<!--more-->

如果对 PWA 不是很了解，建议先阅读：

* 你的首个Progressive Web App [[English]](https://codelabs.developers.google.com/codelabs/your-first-pwapp/#0)
  [[中文]](https://developers.google.cn/web/fundamentals/getting-started/codelabs/your-first-pwapp/?hl=zh-cn)
* [下一代 Web 应用模型 —— Progressive Web App](https://huangxuan.me/2017/02/09/nextgen-web-pwa/)
* [Service worker 概念和用法](https://developer.mozilla.org/zh-CN/docs/Web/API/Service_Worker_API)
* [Service Worker Tools](https://github.com/GoogleChrome/sw-toolbox)

### 添加 Service Worker Toolbox

在主题目录下（默认为 ghost/content/themes/casper）的 `assets` 新建 `dist` 目录。

在该目录添加 `sw-toolbox.js` 文件，文件内容在[这里](https://raw.githubusercontent.com/GoogleChrome/sw-toolbox/master/sw-toolbox.js)

###添加缓存脚本

在您的主题根目录下创建名为serviceworker-v1.js的文件，这里需要注意 `serviceworker-v1.js` 这个文件一定要放到网站根目录，因为它的位置也指定了 Service Worker 可执行的目录权限。

在该文件中包含以下代码：

```javascript
'use strict';

(function () {
  'use strict';



    /**
    * Service Worker Toolbox caching
    */

    var cacheVersion = '-toolbox-v1';
    var dynamicVendorCacheName = 'dynamic-vendor' + cacheVersion;
    var staticVendorCacheName = 'static-vendor' + cacheVersion;
    var staticAssetsCacheName = 'static-assets' + cacheVersion;
    var contentCacheName = 'content' + cacheVersion;
    var maxEntries = 50;

    self.importScripts('assets/dist/sw-toolbox.js');

    self.toolbox.options.debug = false;

    // Cache own static assets
    self.toolbox.router.get('/assets/(.*)', self.toolbox.cacheFirst, {
        cache: {
          name: staticAssetsCacheName,
          maxEntries: maxEntries
        }
    });

    // cache dynamic vendor assets, things which have no other update mechanism like filename change/version hash
    self.toolbox.router.get('/css', self.toolbox.fastest, {
        origin: /fonts\.googleapis\.com/,
            cache: {
              name: dynamicVendorCacheName,
              maxEntries: maxEntries
            }
    });

    // Do not cache disqus
    self.toolbox.router.get('/(.*)', self.toolbox.networkOnly, {
        origin: /disqus\.com/
    });
    self.toolbox.router.get('/(.*)', self.toolbox.networkOnly, {
        origin: /disquscdn\.com/
    });


    // Cache all static vendor assets, e.g. fonts whose version is bind to the according url
    self.toolbox.router.get('/(.*)', self.toolbox.cacheFirst, {
        origin: /(fonts\.gstatic\.com|www\.google-analytics\.com)/,
        cache: {
          name: staticVendorCacheName,
          maxEntries: maxEntries
        }
    });

    self.toolbox.router.get('/content/(.*)', self.toolbox.fastest, {
        cache: {
          name: contentCacheName,
          maxEntries: maxEntries
        }
    });

    self.toolbox.router.get('/*', function (request, values, options) {
        if (!request.url.match(/(\/ghost\/|\/page\/)/) && request.headers.get('accept').includes('text/html')) {
            return self.toolbox.fastest(request, values, options);
        } else {
            return self.toolbox.networkOnly(request, values, options);
        }
        }, {
        cache: {
            name: contentCacheName,
            maxEntries: maxEntries
        }
    });

    // immediately activate this serviceworker
    self.addEventListener('install', function (event) {
        return event.waitUntil(self.skipWaiting());
    });

    self.addEventListener('activate', function (event) {
        return event.waitUntil(self.clients.claim());
    }); 

})();
//# sourceMappingURL=serviceworker-v1.js.map
```
如果在部署阶段，可以将上段代码中的 `self.toolbox.options.debug` 改为 `true`。

从下面这段代码可以看出 `service worker` 使用 Express 路由的代码风格来定位指定的资源：

```javascript
    // 缓存静态资源
    self.toolbox.router.get('/assets/(.*)', self.toolbox.cacheFirst, {
        cache: {
          name: staticAssetsCacheName,
          maxEntries: maxEntries
        }
    });
```

有以下几种获取资源的方式：

* cacheFirst（缓存优先）： 如果请求与缓存条目匹配，则回应。 否则尝试从网络中获取资源。 如果网络请求成功，请更新缓存。 此选项适用于不更改的资源，或具有其他更新机制。
* cacheFirst（最快）：从缓存和网络并行请求资源。以首先回报的方式作出回应。通常这将是缓存的版本，如果有的话。一方面，即使资源被缓存，这个策略总是会产生一个网络请求。另一方面，如果/当网络请求完成时，缓存被更新，以便将来的高速缓存读取将是更新的。
* networkFirst（网络优先）：尝试通过从网络中提取来处理请求。如果成功，将响应存储在缓存中。 否则，尝试从缓存中完成请求。这是用于基本的直读缓存的策略。它也适用于API请求，当您始终希望最新数据可用时，而不是没有数据的陈旧数据。
* cacheOnly：只使用缓存资源
* networkOnly：只使用网络资源

如果我们不想缓存评论内容，我这里使用的是网易云跟帖，可以指定来自 163.com 和 netease.com 的文件的请求始终为网络：
```javascript
   // Do not cache 163.com and netease.com
    self.toolbox.router.get('/(.*)', self.toolbox.networkOnly, {
        origin: /163\.com/
    });
    self.toolbox.router.get('/(.*)', self.toolbox.networkOnly, {
        origin: /netease\.com/
    });
```

缓存 Google fonts 和 Google Analytics:

```
    // Cache all static vendor assets, e.g. fonts whose version is bind to the according url
    self.toolbox.router.get('/(.*)', self.toolbox.cacheFirst, {
        origin: /(fonts\.gstatic\.com|www\.google-analytics\.com)/,
        cache: {
            name: staticVendorCacheName,
            maxEntries: maxEntries
        }
    });
```

### 激活 Service Worker

只需通过 script 标签将下面的 JavaScript 代码引入到 `defalut.hbs` 即可：

```
var serviceWorkerUri = '/serviceworker-v1.js';

if ('serviceWorker' in navigator) {
    navigator.serviceWorker.register(serviceWorkerUri).then(function() {

      // Registration was successful. Now, check to see whether the service worker is controlling the page.
      if (navigator.serviceWorker.controller) {

        console.log('Assets cached by the controlling service worker.');

      } else {

        console.log('Please reload this page to allow the service worker to handle network operations.');

      }
    }).catch(function(error) {

      console.log('ERROR: ' + error);

    });

} else {

    // The current browser doesn't support service workers.
    console.log('Service workers are not supported in the current browser.');

}
```

可以打开开发者工具查看 Service Worker 的运行状况，在 Application 一栏中有 Service Worker 的选项卡。如果出现错误可以通过它进行调试。

### 添加到主屏幕
Progressive Web Apps 的官方文档指出，开发者需要提供一个 `manifest.json` 文件，这里我放在了网站根目录，它包含以下内容：

```json
{
  "short_name": "Pavel's Blog",
  "name": "Pavel's Blog",
  "icons": [
    {
      "src": "assets/images/android-icon-144x144.png",
      "sizes": "144x144",
      "type": "image/png",
      "density": "3.0"
    }
  ],
  "start_url": "/",
  "display": "standalone",
  "orientation": "portrait",
  "background_color": "#6595b8"
}
```

在 `default.hbs` 中添加以下代码：
```
<link rel="manifest" href="/manifest.json">
```

下面在 Android Chrome 中打开你的博客，点右上角按钮——>添加到主屏幕，就可以看到类似原生 App 的效果了。

有任何问题可以在下方评论讨论。

### 参考资料

[Node, Ghost, and Progressive Web Apps (PWA)](https://codeseo.io/node-ghost-and-progressive-web-apps-pwa/)