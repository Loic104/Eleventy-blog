---
layout: post-layout.njk 
title: How to use javascript web workers with Create React App and Typescript
date: 2019-10-31
tags: ['post']
---

<!-- Excerpt Start -->
I don't know if you ever used javascript web workers? It is basically the way to have multi thread in javascript. Unfortunately they are a bit cumbersome to use hence why they are not very popular... I invite you to read the [doc]() to see by yourself the heavy syntax.
<!-- Excerpt End -->
I first came across it 6 years ago when I needed to load STL files in the browser and run lengthy calculations without the website hanging. It worked but it wasn't elegant and difficult to maintain...

Nowadays there are libraries to make the all process a bit easier ([Comlink](https://github.com/GoogleChromeLabs/comlink), [worker-loader](https://github.com/webpack-contrib/worker-loader), [Workerize](https://github.com/developit/workerize), [workerize-loader](https://github.com/developit/workerize-loader)) but even large library like Create React App [don't support it yet](https://github.com/facebook/create-react-app/pull/5886). 

If you are using React and Redux and can customise your build pipeline I will invite you to read this [awesome post](https://dassur.ma/things/react-redux-comlink/)

## 6 years later, same problem, modern solution

Right now I'm having the same problem as 6 years ago, I need to load a STL file and run heavy calculations on it, and I don't want the UI to freeze. I also want to play around OffscreenCanvas to make some [offscreen rendering](https://evilmartians.com/chronicles/faster-webgl-three-js-3d-graphics-with-offscreencanvas-and-web-workers).

As you can imagine my toolset has changed quite a bit in those years it is now 2019, no more Jquery, no more spaghetti code (hopefully) it is all React in Typescript!

## React, Typescript, webworkers here we go!

First a bit about my setup, I use the Typescript version of Create React App.

```
npx create-react-app awesome-typescript-app --typescript
```

Then I use the awesome [react-app-rewired](https://github.com/timarney/react-app-rewired) to add modules to my [webpack config without ejecting](/react-app-rewired-is-awesome/). I decided to use [workerize-loader](https://github.com/developit/workerize-loader) because I like the async...await pattern.

So I create my `config-overide.js`
```Javascript
/* config-overrides.js */

module.exports = function override(config, env) {
    config.module.rules.push({
        test: /\.worker\.js$/,
        use: { loader: 'workerize-loader' }
    })
    config.output.globalObject = 'self'
    return config
}
```

I create a simple webworker `example.worker.js`
```Javascript
export default function worker() {
    async function expensive(time) {
        let start = Date.now(),
            count = 0
        while (Date.now() - start < time) count++
        return count
    }
    return { expensive }
}
```

The trick to play well with Typescript is to create a generic type wrapper function `createWorker`
```Typescript
/* create-worker.ts */
type WorkerType<T> = T & Pick<Worker, 'terminate'>
export function createWorker<T>(workerPath: T): WorkerType<T> {
    return (workerPath as any)()
}
export type createWorker<T> = WorkerType<T>
```


Which I use in a simple React component
```Typescript
import * as React from 'react'
import { createWorker } from 'utils/create-worker'
import * as worker from 'utils/example.worker'

export interface IProps {}

export interface IState {}

export default class DashboardPage extends React.Component<IProps, IState> {
    public componentDidMount() {
      const instance = createWorker(worker)
        instance.expensive(1000)
        .then(count => {
            console.log(`Ran ${count} loops`)
        })
        .finally(() => instance.terminate())
    }

    public render() {
        return (
            <div>
                <h2>This is my component</h2>
            </div>
        )
    }
}


```

___

### Resources:

[https://github.com/developit/workerize-loader/issues/3](https://github.com/developit/workerize-loader/issues/3)
[]