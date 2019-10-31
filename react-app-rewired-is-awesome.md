---
layout: layout.njk 
title: How to easily modify your webpack config in Create React app without ejecting?
date: 2019-10-31
tags: ['post']
---

#  How to easily modify your webpack config in Create React app without ejecting?

If you are using Create React App, I'm sure you had the same problem.

I have been using React for years, using different tools such as gulp, rollup and webpack, so I am well aware on how to configure a build pipeline and customise it to my particular needs. But nowadays I prefer to just use [Create React App](https://github.com/facebook/create-react-app) which works perfectly fine for 95% of the project (assuming you don't want server side rendering or precompiled html).

Just run ``npx create-react-app my-app`` and you are ready to go!

The main issue that I see people having is that while CRA is suitable for a lot of applications, all the configuration is hidden under the hood and can't be modified. This leads to people [ejecting](https://create-react-app.dev/docs/available-scripts/#npm-run-eject) or creating a [fork](https://auth0.com/blog/how-to-configure-create-react-app/) for just adding another module to webpack.

### There must be a better way!!!

I recently came across an awesome tool that make this a all lot easier, [React App Rewired](https://github.com/timarney/react-app-rewired) to the rescue! This tool allows you to easily modify the default CRA webpack config just by adding a single file to your project.

1) Obviously you start by adding react-app-rewired to your project

```
npm i -D react-app-rewired
```

2) next you create a `config-overrides.js` file in the project root which will specify what changes to your webpack config needs to be done.

```javascript
module.exports = function override(config, env) {
    /* 
    You can modify the config object which ever way you want. You even have access to the env to have different config in dev and prod
    */    

    config.module.rules.push({
        test: /\.worker\.js$/,
        use: { loader: 'workerize-loader' }
    })


    config.output.globalObject = 'self'

    /* You just need to return a webpack config object */
    return config
}
```

3) Change your npm script to use react-app-rewired

```json
    "scripts": {
        "start": "react-app-rewired start",
        "build": "react-app-rewired build",
        "test": "react-app-rewired test",
        "eject": "react-scripts eject"
    }
```

## Et voila!

The doc explain the differences if you are using an older version of CRA and some other configuration examples. 

***
P.S. it can also work with custom forks of react-script that way you can have the best of both ;)