## babel7升级指南
- 安装
``` console
tnpm i -D @babel/core @babel/plugin-proposal-class-properties @babel/plugin-proposal-decorators @babel/plugin-proposal-object-rest-spread @babel/plugin-syntax-dynamic-import @babel/plugin-transform-runtime @babel/preset-env @babel/preset-react @babel/runtime
```
- babel-config.js
``` javascript
module.exports = {
    "plugins": [
        "@babel/plugin-syntax-dynamic-import",
        "@babel/plugin-proposal-object-rest-spread",
        "@babel/plugin-transform-runtime",
        ["@babel/plugin-proposal-decorators", {
            legacy: true
        }],
        "@babel/plugin-proposal-class-properties"
    ],
    "ignore": [
        "**.min.js"
    ],
    "presets": [
        ["@babel/env", {
            "modules": "umd",
            "targets": {
                "browsers": [
                    "chrome > 55",
                    "last 5 versions",
                    "ie >= 8"
                ]
            },
            // "useBuiltIns": "usage"
        }],
        "@babel/preset-react"
    ],
    "env": {
        "development": {
            "plugins": [
                "react-hot-loader/babel"
            ]
        }
    }
}

```
- 使用babel.config.js可以避免babel-loader使用其他目录的.babelrc配置
