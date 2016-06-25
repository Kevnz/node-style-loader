# server-style-loader

`server-style-loader` is a Webpack loader for loading styles on the server side. It behaves almost identically to `style-loader`, which allows you to use it without changing the way you load CSS in your application components.

`server-style-loader` supports critical path style rendering without imposing any import splitting method. This allows you to have a style loading path independant from your rendering path and has performance implications.

## How to install

```
$ npm install server-style-loader --save-dev
```

## How to use

### Webpack configuration 

```js
target: 'node',
module: {
  loaders: [
    {
      test: /\.css$/,
      loader: `server-style!css`
    },
    ...
```

### Server rendering

#### Simple usage

```js
import App from 'components/App'
import {collectInitial} from 'server-style-loader/collect'

// do not call this before your application component has been imported
const initialStyleTag = collectInitial()

function renderPage(props) {

  const reactString = renderToString(createElement(App, props))

  return(
   `<!DOCTYPE html>
    <html>
      <head>
        <meta charset="utf-8">
        ${initialStyleTag}
      </head>
      <body>
        <div id="mount">${reactString}</div>
        <script src="/build/bundle.js"></script>
      </body>
    </html>`
  )
}
```

#### Usage for import splitting

If you conditionally import components during rendering (e.g. in your routes), or import styles in your component render functions, you need to use `collectContext` to collect the critical path CSS.

```js
import {collectInitial, collectContext} from 'server-style-loader/collect'

// do not call this before your routes have been imported
const initialStyleTag = collectInitial()

function renderPage(contextEl, props) {

  // render and capture CSS
  const [contextStyleTag, reactString] = collectContext(
    () => renderToString(createElement(contextEl, props)))

  return(
   `<!DOCTYPE html>
    <html>
      <head>
        <meta charset="utf-8">
        ${initialStyleTag}
        ${contextStyleTag}
      </head>
      <body>
        <div id="mount">${reactString}</div>
        <script src="/build/bundle.js"></script>
      </body>
    </html>`
  )
}

```

### Client-side cleanup

```js
import serverStyleCleanup from 'server-style-loader/clientCleanup'

// initial react render
render(AppElement)

// remove server-generated CSS after your first render
serverStyleCleanup()
```

## Server-side style rendering loaders comparison

|     | style collection | CSS Modules | style import splitting | shadows style-loader rendering |
| --- | ---------------- | ----------- | ---------------------- | ------------------------------ |
| server-style-loader | yes | yes | yes, standard | partial |
| [react-webpack-server-side-example](https://github.com/webpack/react-webpack-server-side-example) | incomplete | no | partial, standard | no |
| [style-collector-loader](https://github.com/thereactivestack/style-collector-loader) | requires globals | no | colisions | no |
| [fake-style-loader](https://github.com/dferber90/fake-style-loader) | partially out of scope | yes | N/A | no |
| [isomorphic-style-loader](https://github.com/kriasoft/isomorphic-style-loader) | yes | yes | partial, non-standard | no |

## Roadmap

- add test coverage
- test contextCollect
- test deduping
- add support for media/sourceMap
- add support for stylesheet url
- test/add hot reload support
- optimize speed/inject stylesInDom into `style-loader` to save first style load on client
