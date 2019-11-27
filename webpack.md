## webpack安装

1. 通过npm安装webpack

    ```
    npm init -y
    npm i -D webpack webpack-cli
    ```

1. 创建并修改文件

    src/index.js
    ```javascript
    function renderText(dom, text) {
        dom.innerText = text;
        return dom;
    }
    renderText(document.body, 'hello');
    ```
    /index.html
    ```html
    <!DOCTYPE html>
    <html lang="en">

    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <meta http-equiv="X-UA-Compatible" content="ie=edge">
        <title>webpack</title>
    </head>

    <body>

        <!-- <script src="./src/index.js"></script> -->
        <script src="./dist/main.js"></script>
    </body>

    </html>
    ```
    /package.json
    ```json
    {
        "description": "",
        "private": true, // 确保我们的安装包是私有的
        // "main":"index.js", // 移除main入口
        "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1"
    },
    }
    ```

1. 执行`npx webpack`会脚本输出为main.js，然后在浏览器中打开index.html，会看到文本`Hello`
    
    ```
    npx webpack
    Hash: 538ffea8e1ab86809feb
    Version: webpack 4.41.2
    Time: 275ms
    Built at: 2019-11-27 18:19:09
    Asset       Size  Chunks             Chunk Names
    main.js  977 bytes       0  [emitted]  main
    Entrypoint main = main.js
    [0] ./src/index.js 116 bytes {0} [built]
    ```

1. 新建配置文件webpack.config.js

    webpack.config.js
    ```js
    const path = require('path');

    module.exports = {
        entry: './src/index.js',
        output: {
            filename: 'bundle.js',
            path: path.resolve(__dirname, 'dist')
        }
    };
    ```

1. 添加npm脚本命令

    package.json
    ```json
    {
        "build": "webpack"
    }
    ```
    执行`npm run build`，然后就over了

## 