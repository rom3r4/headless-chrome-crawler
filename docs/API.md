# API reference

## Table of Contents

* [class: HCCrawler](#class-hccrawler)
  * [HCCrawler.connect([options])](#hccrawlerconnectoptions)
  * [HCCrawler.launch([options])](#hccrawlerlaunchoptions)
  * [HCCrawler.executablePath()](#hccrawlerexecutablepath)
  * [HCCrawler.defaultArgs()](#hccrawlerdefaultargs)
  * [crawler.queue([options])](#crawlerqueueoptions)
  * [crawler.setMaxRequest(maxRequest)](#crawlersetmaxrequestmaxrequest)
  * [crawler.pause()](#crawlerpause)
  * [crawler.resume()](#crawlerresume)
  * [crawler.clearCache()](#crawlerclearcache)
  * [crawler.close()](#crawlerclose)
  * [crawler.disconnect()](#crawlerdisconnect)
  * [crawler.version()](#crawlerversion)
  * [crawler.userAgent()](#crawleruseragent)
  * [crawler.wsEndpoint()](#crawlerwsendpoint)
  * [crawler.onIdle()](#crawleronidle)
  * [crawler.isPaused()](#crawlerispaused)
  * [crawler.queueSize()](#crawlerqueuesize)
  * [crawler.pendingQueueSize()](#crawlerpendingqueuesize)
  * [crawler.requestedCount()](#crawlerrequestedcount)
  * [event: 'requestdisallowed'](#event-requestdisallowed)
  * [event: 'requeststarted'](#event-requeststarted)
  * [event: 'requestskipped'](#event-requestskipped)
  * [event: 'requestfinished'](#event-requestfinished)
  * [event: 'requestretried'](#event-requestretried)
  * [event: 'requestfailed'](#event-requestfailed)
  * [event: 'robotstxtrequestfailed'](#event-robotstxtrequestfailed)
  * [event: 'sitemapxmlrequestfailed'](#event-sitemapxmlrequestfailed)
  * [event: 'maxdepthreached'](#event-maxdepthreached)
  * [event: 'maxrequestreached'](#event-maxrequestreached)
  * [event: 'disconnected'](#event-disconnected)
* [class: SessionCache](#class-sessioncache)
* [class: RedisCache](#class-rediscache)
* [class: BaseCache](#class-basecache)
* [class: CSVExporter](#class-csvexporter)
* [class: JSONLineExporter](#class-jsonlineexporter)
* [class: BaseExporter](#class-baseexporter)

## class: HCCrawler

HCCrawler provides methods to launch or connect to a Chromium instance.

```js
const HCCrawler = require('headless-chrome-crawler');

(async () => {
  const crawler = await HCCrawler.launch({
    evaluatePage: (() => ({
      title: $('title').text(),
    })),
    onSuccess: (result => {
      console.log(result);
    }),
  });
  crawler.queue('https://example.com/');
  await crawler.onIdle();
  await crawler.close();
})();
```

### HCCrawler.connect([options])

* `options` <[Object]>
  * `maxConcurrency` <[number]> Maximum number of pages to open concurrently, defaults to `10`.
  * `maxRequest` <[number]> Maximum number of requests, defaults to `0`. Pass `0` to disable the limit.
  * `exporter` <[Exporter]> An exporter object which extends [BaseExporter](#class-baseexporter)'s interfaces to export results, default to `null`.
  * `cache` <[Cache]> A cache object which extends [BaseCache](#class-basecache)'s interfaces to remember and skip duplicate requests, defaults to a [SessionCache](#class-sessioncache) object.
  * `persistCache` <[boolean]> Whether to clear cache on closing or disconnecting from the Chromium instance, defaults to `false`.
  * `preRequest(options)` <[Function]> Function to do anything like modifying `options` before each request. You can also return `false` if you want to skip the request.
    * `options` <[Object]> [crawler.queue()](#crawlerqueueoptions)'s options with default values.
  * `customCrawl(page, crawl)` <[Function]> Function to customize crawled result, allowing access to [Puppeteer](https://github.com/GoogleChrome/puppeteer)'s raw API.
    * `page` <[Page]> [Puppeteer](https://github.com/GoogleChrome/puppeteer)'s raw API.
    * `crawl` <[Function]> Function to run crawling, which resolves to the result passed to `onSuccess` function.
  * `onSuccess(result)` <[Function]> Function to be called when `evaluatePage()` successes.
    * `result` <[Object]>
      * `redirectChain` <[Array]<[Object]>> Redirect chain of requests.
        * `url` <[string]> Requested url.
        * `headers` <[Object]> Request headers.
      * `cookies` <[Array]<[Object]>> List of cookies.
        * `name` <[string]>
        * `value` <[string]>
        * `domain` <[string]>
        * `path` <[string]>
        * `expires` <[number]> Unix time in seconds.
        * `httpOnly` <[boolean]>
        * `secure` <[boolean]>
        * `session` <[boolean]>
        * `sameSite` <[string]> `"Strict"` or `"Lax"`.
      * `response` <[Object]>
        * `ok` <[boolean]> whether the status code in the range 200-299 or not.
        * `status` <[string]> status code of the request.
        * `url` <[string]> Last requested url.
        * `headers` <[Object]> Response headers.
      * `options` <[Object]> [crawler.queue()](#crawlerqueueoptions)'s options with default values.
      * `result` <[Serializable]> The result resolved from `evaluatePage()` option.
      * `screenshot` <[Buffer]> Buffer with the screenshot image, which is `null` when `screenshot` option not passed.
      * `links` <[Array]<[string]>> List of links found in the requested page.
      * `depth` <[number]> Depth of the followed links.
      * `previousUrl` <[string]> The previous request's url. The value is `null` for the initial request.
  * `onError(error)` <[Function]> Function to be called when request fails.
    * `error` <[Error]> Error object.
      * `options` <[Object]> [crawler.queue()](#crawlerqueueoptions)'s options with default values.
      * `depth` <[number]> Depth of the followed links.
      * `previousUrl` <[string]> The previous request's url. The value is `null` for the initial request.
* returns: <[Promise]<[HCCrawler]>> Promise which resolves to HCCrawler instance.

This method connects to an existing Chromium instance. The following options are passed to [puppeteer.connect()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#puppeteerconnectoptions).

```
browserWSEndpoint, ignoreHTTPSErrors, slowMo
```

Also, the following options can be set as default values when [crawler.queue()](#crawlerqueueoptions) are executed.

```
url, allowedDomains, deniedDomains, timeout, priority, depthPriority, delay, retryCount, retryDelay, jQuery, browserCache, device, username, password, evaluatePage, cookies, extraHeaders
```

> **Note**: In practice, setting the options every time you queue equests is redundant. Therefore, it's recommended to set the default values and override them depending on the necessity.

### HCCrawler.launch([options])

* `options` <[Object]>
  * `maxConcurrency` <[number]> Maximum number of pages to open concurrently, defaults to `10`.
  * `maxRequest` <[number]> Maximum number of requests, defaults to `0`. Pass `0` to disable the limit.
  * `exporter` <[Exporter]> An exporter object which extends [BaseExporter](#class-baseexporter)'s interfaces to export results, default to `null`.
  * `cache` <[Cache]> A cache object which extends [BaseCache](#class-basecache)'s interfaces to remember and skip duplicate requests, defaults to a [SessionCache](#class-sessioncache) object.
  * `persistCache` <[boolean]> Whether to clear cache on closing or disconnecting from the Chromium instance, defaults to `false`.
  * `preRequest(options)` <[Function]> Function to do anything like modifying `options` before each request. You can also return `false` if you want to skip the request.
    * `options` <[Object]> [crawler.queue()](#crawlerqueueoptions)'s options with default values.
  * `customCrawl(page, crawl)` <[Function]> Function to customize crawled result, allowing access to [Puppeteer](https://github.com/GoogleChrome/puppeteer)'s raw API.
    * `page` <[Page]> [Puppeteer](https://github.com/GoogleChrome/puppeteer)'s raw API.
    * `crawl` <[Function]> Function to run crawling, which resolves to the result passed to `onSuccess` function.
  * `onSuccess(result)` <[Function]> Function to be called when `evaluatePage()` successes.
    * `result` <[Object]>
      * `redirectChain` <[Array]<[Object]>> Redirect chain of requests.
        * `url` <[string]> Requested url.
        * `headers` <[Object]> Request headers.
      * `cookies` <[Array]<[Object]>> List of cookies.
        * `name` <[string]>
        * `value` <[string]>
        * `domain` <[string]>
        * `path` <[string]>
        * `expires` <[number]> Unix time in seconds.
        * `httpOnly` <[boolean]>
        * `secure` <[boolean]>
        * `session` <[boolean]>
        * `sameSite` <[string]> `"Strict"` or `"Lax"`.
      * `response` <[Object]>
        * `ok` <[boolean]> whether the status code in the range 200-299 or not.
        * `status` <[string]> status code of the request.
        * `url` <[string]> Last requested url.
        * `headers` <[Object]> Response headers.
      * `options` <[Object]> [crawler.queue()](#crawlerqueueoptions)'s options with default values.
      * `result` <[Serializable]> The result resolved from `evaluatePage()` option.
      * `screenshot` <[Buffer]> Buffer with the screenshot image, which is `null` when `screenshot` option not passed.
      * `links` <[Array]<[string]>> List of links found in the requested page.
      * `depth` <[number]> Depth of the followed links.
      * `previousUrl` <[string]> The previous request's url. The value is `null` for the initial request.
  * `onError(error)` <[Function]> Function to be called when request fails.
    * `error` <[Error]> Error object.
      * `options` <[Object]> [crawler.queue()](#crawlerqueueoptions)'s options with default values.
      * `depth` <[number]> Depth of the followed links.
      * `previousUrl` <[string]> The previous request's url. The value is `null` for the initial request.
* returns: <[Promise]<[HCCrawler]>> Promise which resolves to HCCrawler instance.

The method launches a Chromium instance. The following options are passed to [puppeteer.launch()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#puppeteerlaunchoptions).

```
ignoreHTTPSErrors, headless, executablePath, slowMo, args, ignoreDefaultArgs, handleSIGINT, handleSIGTERM, handleSIGHUP, dumpio, userDataDir, env, devtools
```

Also, the following options can be set as default values when [crawler.queue()](#crawlerqueueoptions) are executed.

```
url, allowedDomains, deniedDomains, timeout, priority, depthPriority, delay, retryCount, retryDelay, jQuery, browserCache, device, username, password, evaluatePage, cookies, extraHeaders
```

> **Note**: In practice, setting the options every time you queue the requests is redundant. Therefore, it's recommended to set the default values and override them depending on the necessity.

### HCCrawler.executablePath()

* returns: <[string]> An expected path to find bundled Chromium.

### HCCrawler.defaultArgs()

* returns: <[Array]<[string]>> The default flags that Chromium will be launched with.

### crawler.queue([options])

* `options` <[Object]>
  * `url` <[string]> Url to navigate to. The url should include scheme, e.g. `https://`.
  * `maxDepth` <[number]> Maximum depth for the crawler to follow links automatically, default to 1. Leave default to disable following links.
  * `priority` <[number]> Basic priority of queues, defaults to `1`. Priority with larger number is preferred.
  * `depthPriority` <[boolean]> Whether to adjust priority based on its depth, defaults to `true`. Leave default to increase priority for higher depth, which is [depth-first search](https://en.wikipedia.org/wiki/Depth-first_search).
  * `skipDuplicates` <[boolean]> Whether to skip duplicate requests, default to `true`. The request is considered to be the same if `url`, `userAgent`, `device` and `extraHeaders` are strictly the same.
  * `skipRequestedRedirect` <[boolean]> Whether to skip requests already appeared in redirect chains of requests, default to `false`. This option is ignored when `skipDuplicates` is set `false`.
  * `obeyRobotsTxt` <[boolean]> Whether to obey [robots.txt](https://developers.google.com/search/reference/robots_txt), default to `true`.
  * `followSitemapXml` <[boolean]> Whether to use [sitemap.xml](https://www.sitemaps.org/) to find locations, default to `false`.
  * `allowedDomains` <[Array]<[string]|[RegExp]>> List of domains allowed to request. Pass `null` or leave default to skip checking allowed domain.
  * `deniedDomains` <[Array]<[string]|[RegExp]>> List of domains not allowed to request. Pass `null` or leave default to skip checking denied domain.
  * `allowedPaths` <[Array]<[string]|[RegExp]>> List of paths in the domain allowed to request. Pass `null` or leave default to skip checking allowed domain paths.
  * `deniedPaths` <[Array]<[string]|[RegExp]>> List of domain paths not allowed to request. Pass `null` or leave default to skip checking denied domain paths.
  * `delay` <[number]> Number of milliseconds after each request, defaults to `0`. When delay is set, `maxConcurrency` option must be `1`.
  * `timeout` <[number]> Navigation timeout in milliseconds, defaults to `30` seconds, pass `0` to disable timeout.
  * `waitUntil` <[string]|[Array]<[string]>> When to consider navigation succeeded, defaults to `load`. See the [Puppeteer's page.goto()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagegotourl-options)'s `waitUntil` options for further details.
  * `waitFor` <[Object]> See [Puppeteer's page.waitFor()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagewaitforselectororfunctionortimeout-options-args) for further details.
    * `selectorOrFunctionOrTimeout` <[string]|[number]|[function]> A selecctor, predicate or timeout to wait for.
    * `options` <[Object]> Optional waiting parameters.
    * `args` <[Array]<[Serializable]>> List of arguments to pass to the predicate function.
  * `retryCount` <[number]> Number of limit when retry fails, defaults to `3`.
  * `retryDelay` <[number]> Number of milliseconds after each retry fails, defaults to `10000`.
  * `jQuery` <[boolean]> Whether to automatically add [jQuery](https://jquery.com) tag to page, defaults to `true`.
  * `browserCache` <[boolean]> Whether to enable browser cache for each request, defaults to `true`.
  * `device` <[string]> Device to emulate. Available devices are listed [here](https://github.com/GoogleChrome/puppeteer/blob/master/DeviceDescriptors.js).
  * `username` <[string]> Username for basic authentication. pass `null` if it's not necessary.
  * `screenshot` <[Object]> Screenshot option, defaults to `null`. This option is passed to [Puppeteer's page.screenshot()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagescreenshotoptions). Pass `null` or leave default to disable screenshot.
  * `viewport` <[Object]> See [Puppeteer's page.setViewport()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagesetviewportviewport) for further details.
    * `width` <[number]> Page width in pixels.
    * `height` <[number]> Page height in pixels.
  * `password` <[string]> Password for basic authentication. pass `null` if it's not necessary.
  * `userAgent` <[string]> User agent string to override in this page.
  * `extraHeaders` <[Object]> An object containing additional headers to be sent with every request. All header values must be strings.
  * `cookies` <[Array]<[Object]>> List of cookies to be sent with every request. Either url or domain must be specified for each cookie.
    * `name` <[string]> **required**
    * `value` <[string]> **required**
    * `url` <[string]>
    * `domain` <[string]>
    * `path` <[string]>
    * `expires` <[number]> Unix time in seconds.
    * `httpOnly` <[boolean]>
    * `secure` <[boolean]>
    * `sameSite` <[string]> `"Strict"` or `"Lax"`.
  * `evaluatePage()` <[Function]> Function to be evaluated in browsers. Return serializable object. If it's not serializable, the result will be `undefined`.
* returns: <[Promise]> Promise resolved when queue is pushed.

> **Note**: `response.url` may be different from `options.url` especially when the requested url is redirected.

The options can be either an object, an array, or a string. When it's an array, each item in the array will be executed. When it's a string, the options are transformed to an object with only url defined.

### crawler.setMaxRequest(maxRequest)

* `maxRequest` <[number]> Modify `maxRequest` option you passed to [HCCrawler.connect()](#hccrawlerconnectoptions) or [HCCrawler.launch()](#hccrawlerlaunchoptions).

### crawler.pause()

This method pauses processing queues. You can resume the queue by calling [crawler.resume()](#crawlerresume).

### crawler.resume()

This method resumes processing queues. This method may be used after the crawler is intentionally closed by calling [crawler.pause()](#crawlerpause) or request count reached `maxRequest` option.

### crawler.clearCache()

* returns: <[Promise]> Promise resolved when the cache is cleared.

This method clears the cache when it's used.

### crawler.close()

* returns: <[Promise]> Promise resolved when ther browser is closed.

### crawler.disconnect()

* returns: <[Promise]> Promise resolved when ther browser is disconnected.

### crawler.version()

* returns: <[Promise]<[string]>> Promise resolved with the Chromium version.

### crawler.userAgent()

* returns: <[Promise]<[string]>> Promise resolved with the default user agent.

### crawler.wsEndpoint()

* returns: <[string]> Websocket url to connect to the browser.

### crawler.onIdle()

* returns: <[Promise]> Promise resolved when queues become empty or paused.

### crawler.isPaused()

* returns: <[boolean]> Whether the queue is paused.

### crawler.queueSize()

* returns: <[Promise]<[number]>> Promise resolves to the size of queues.

### crawler.pendingQueueSize()

* returns: <[number]> The size of pending queues.

### crawler.requestedCount()

* returns: <[number]> The count of total requests.

### event: 'requestdisallowed'

* `options` <[Object]>

Emitted when a request is disallowed by robots.txt.

### event: 'requeststarted'

* `options` <[Object]>

Emitted when a request started.

### event: 'requestskipped'

* `options` <[Object]>

Emitted when a request is skipped.

### event: 'requestfinished'

* `options` <[Object]>

Emitted when a request finished successfully.

### event: 'requestretried'

* `options` <[Object]>

Emitted when a request is retried.

### event: 'requestfailed'

* `error` <[Error]>
  * `options` <[Object]> [crawler.queue()](#crawlerqueueoptions)'s options with default values.
  * `depth` <[number]> Depth of the followed links.
  * `previousUrl` <[string]> The previous request's url. The value is `null` for the initial request.

Emitted when a request failed.

### event: 'robotstxtrequestfailed'

* `error` <[Error]>
  * `options` <[Object]> [crawler.queue()](#crawlerqueueoptions)'s options with default values.
  * `depth` <[number]> Depth of the followed links.
  * `previousUrl` <[string]> The previous request's url. The value is `null` for the initial request.

Emitted when a request to [robots.txt](https://developers.google.com/search/reference/robots_txt) failed

### event: 'sitemapxmlrequestfailed'

* `error` <[Error]>
  * `options` <[Object]> [crawler.queue()](#crawlerqueueoptions)'s options with default values.
  * `depth` <[number]> Depth of the followed links.
  * `previousUrl` <[string]> The previous request's url. The value is `null` for the initial request.


Emitted when a request to [sitemap.xml](https://www.sitemaps.org/) failed

### event: 'maxdepthreached'

* `options` <[Object]>

Emitted when a queue reached the [crawler.queue()](#crawlerqueueoptions)'s `maxDepth` option.

### event: 'maxrequestreached'

Emitted when a queue reached the [HCCrawler.connect()](#hccrawlerconnectoptions) or [HCCrawler.launch()](#hccrawlerlaunchoptions)'s `maxRequest` option.

### event: 'disconnected'

Emitted when the browser instance is disconnected.

### class: SessionCache

`SessionCache` is the [HCCrawler.connect()](#hccrawlerconnectoptions)'s default `cache` option. By default, the crawler remembers already requested urls on its memory.

```js
const HCCrawler = require('headless-chrome-crawler');

(async () => {
  const crawler = await HCCrawler.launch({ cache: null }); // Pass null to the cache option to disable it.
  // ...
})();
```

### class: RedisCache

* `options` <[Object]>
  * `expire` <[number]> Seconds to expires cache after setting each value, default to `null`.

Passing a `RedisCache` object to the [HCCrawler.connect()](#hccrawlerconnectoptions)'s `cache` option allows you to persist requested urls and [robots.txt](https://developers.google.com/search/reference/robots_txt) in [Redis](https://redis.io) so that it prevent from requesting same urls in a distributed servers' environment. It also works well with its `persistCache` option to be true.

Other constructing options are passed to [NodeRedis's redis.createClient()](https://github.com/NodeRedis/node_redis#rediscreateclient)'s options.

```js
const HCCrawler = require('headless-chrome-crawler');
const RedisCache = require('headless-chrome-crawler/cache/redis');

const cache = new RedisCache({ host: '127.0.0.1', port: 6379 });

(async () => {
  const crawler = await HCCrawler.launch({
    persistCache: true, // Set true so that cache won't be cleared when closing the crawler
    cache,
  });
  // ...
})();
```

### class: BaseCache

You can create your own cache by extending the [BaseCache's interfaces](https://github.com/yujiosaka/headless-chrome-crawler/blob/master/cache/base.js).

See [here](https://github.com/yujiosaka/headless-chrome-crawler/blob/master/examples/custom-cache.js) for example.

### class: CSVExporter

* `options` <[Object]>
  * `file` <[string]> File path to export output.
  * `fields` <[Array]<[string]>> List of fields to be used for columns. This option is also used for the headers.
  * `separator` <[string]> Character to separate columns.

```js
const HCCrawler = require('headless-chrome-crawler');
const CSVExporter = require('headless-chrome-crawler/exporter/csv');

const FILE = './tmp/result.csv';

const exporter = new CSVExporter({
  file: FILE,
  fields: ['response.url', 'response.status', 'links.length'],
  separator: '\t',
});

(async () => {
  const crawler = await HCCrawler.launch({ exporter });
  // ...
})();
```

### class: JSONLineExporter

* `options` <[Object]>
  * `file` <[string]> File path to export output.
  * `fields` <[Array]<[string]>> List of fields to be filtered in json, defaults to `null`. Leave default not to filter fields.
  * `jsonReplacer` <[Function]> Function that alters the behavior of the stringification process, defaults to `null`. This is useful to sorts keys always in the same order.

```js
const HCCrawler = require('headless-chrome-crawler');
const JSONLineExporter = require('headless-chrome-crawler/exporter/json-line');

const FILE = './tmp/result.json';

const exporter = new JSONLineExporter({
  file: FILE,
  fields: ['options', 'response'],
});

(async () => {
  const crawler = await HCCrawler.launch({ exporter });
  // ...
})();
```

### class: BaseExporter

You can create your own exporter by extending the [BaseExporter's interfaces](https://github.com/yujiosaka/headless-chrome-crawler/blob/master/exporter/base.js).

See [here](https://github.com/yujiosaka/headless-chrome-crawler/blob/master/examples/custom-exporter.js) for example.

[Array]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array "Array"
[boolean]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Boolean_type "Boolean"
[Buffer]: https://nodejs.org/api/buffer.html#buffer_class_buffer "Buffer"
[function]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function "Function"
[number]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Number_type "Number"
[Object]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object "Object"
[Promise]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise "Promise"
[string]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#String_type "String"
[RegExp]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp "RegExp"
[Serializable]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify#Description "Serializable"
[Error]: https://nodejs.org/api/errors.html#errors_class_error "Error"
[HCCrawler]: #class-hccrawler "HCCrawler"
[Exporter]: #baseexporter "Exporter"
[Cache]: #basecache "Cache"
[Page]: https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#class-page "Page"
