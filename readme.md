# next-offline

Use [sw-precache](https://github.com/GoogleChrome/sw-precache) with [Next.js](https://github.com/zeit/next.js)

## Installation

```
npm install --save next-offline
```

or

```
yarn add next-offline
```

## Usage

Create a `next.config.js` in your project

```js
// next.config.js
const withOffline = require('next-offline')
module.exports = withOffline()
```

Then create a `server.js`

```js
// server.js
const { createServer } = require('http')
const next = require('next')
const { join } = require('path')

const app = next({ dev: process.env.NODE_ENV !== 'production' })
const handle = app.getRequestHandler()

app.prepare()
  .then(() => {
    createServer((req, res) => {
      const parsedUrl = parse(req.url, true)
      const { pathname } = parsedUrl

      if (pathname === '/service-worker.js') {
        const filePath = join(__dirname, '.next', pathname)
        app.serveStatic(req, res, filePath)
      } else {
        handle(req, res, parsedUrl)
      }
    })
    .listen(port, () => {
      console.log(`> Ready on http://localhost:${port}`)
    })
  })
```

Optionally you can add your custom Next.js configuration as parameter

```js
// next.config.js
const withOffline = require('next-offline')

module.exports = withOffline({
  webpack(config, options) {
    return config
  }
})
```

## Options

The default object passed to [sw-precache-webpack-plugin](https://github.com/goldhand/sw-precache-webpack-plugin) is here:
```js
{
  verbose: false,
  minify: true,
  staticFileGlobsIgnorePatterns: [/\.next\//],
  runtimeCaching: [
    {
      handler: 'networkFirst',
      urlPattern: /^https?.*/
    }
  ]
}
```

It can easily be modified by passing a `swPreCacheOptions` object to `withOffline` in your `next.config.js`

```js
// next.config.js
const withOffline = require('next-offline')
module.exports = withOffline({
  swPreCacheOptions: {
    verbose: true,
    minify: false
  }
})
```

## Custom Service worker register script
By default `next-offline` will register a service worker with the script below, this is automatically be add to your client side bundle once `nextOffline` is invoked.

```js
if ('serviceWorker' in navigator) {
  navigator.serviceWorker
    .register('/service-worker.js')
    .then(registration => console.info('service worker registration successful'))
    .catch(err => console.warn('service worker registration failed', err.message))
}
```

You can pass in your own custom service worker register script by using the `registerPath` option like this:

```js
// next.config.js
const withOffline = require('next-offline')
const { resolve } = require('path')

module.exports = withOffline({
  swPreCacheOptions: {
    registerPath: resolve(__dirname, 'my-service-worker.js')
  }
})
```

Questions? Feedback? [Please let me know](https://github.com/hanford/next-offline/issues/new)

## License (MIT)

```
WWWWWW||WWWWWW
 W W W||W W W
      ||
    ( OO )__________
     /  |           \
    /o o|    MIT     \
    \___/||_||__||_|| *
         || ||  || ||
        _||_|| _||_||
       (__|__|(__|__|
```
Copyright © 2017-present [Jack Hanford](http://jackhanford.com), jackhanford@gmail.com

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the 'Software'), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
