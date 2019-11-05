## Webpack

本文为学习 [阮一峰webpack demos](https://github.com/xu1685/webpack-demos) 的一个学习笔记（基本翻译） 
webpack需要以一个配置文件进行配置，webpack.config.js

### demo1-demo2 Entry file
webpack读取输入文件（例如 main.js），将其打包成打包文件bundle.js
```
// webpack.config.js
module.exports = {
  entry: './main.js',
  output: {
    filename: 'bundle.js'
  }
};
```
也可以同时配置多个入口文件,用于有多个不同入口文件的app
```
// main1.js
document.write('<h1>this is page1</h1>');

// main2.js
document.write('<h2>this is page2</h2>');

// index.html
<html>
  <body>
    <script src="bundle1.js"></script>
    <script src="bundle2.js"></script>
  </body>
</html>

// webpack.config.js
module.exports = {
    entry: {
        bundle1: './main1.js',
        bundle2: './main2.js'
    },
    output: {
        filename: '[name].js'
    }
}
```

### demo3-demo5 Loader
loaders是webpack打包文件之前，将文件进行转化的预处理器  
比如babel-loader可以将jsx/es6文件转化为普通js文件，之后再由webpack进行打包  

关于loader  
* loader是模块的预处理器，作用优先级为从右至左，从下至上
* loader使用之前安装,不同的文件不同的处理需求会使用不同的loaders   
eg: npm install --save-dev css-loader ts-loader

#### Example1 (babel-loader)
```
// mian.jsx
const React = require('react)
const ReactDom = require('react-dom')

ReactDom.render(
    <h1>HELLO WORLD!</h1>
    document.querySelector('#wrapper')
)

// index.html
<html>
  <body>
    <div id="wrapper"></div>
    <script src="bundle.js"></script>
  </body>
</html>

//webpack.config.js
module.exports = {
    entry: 'mian.jsx',
    output: {
        filename: 'bundle.js'
    },
    module: {
        rules: [
            {
                test: /\.jsx?$/,
                exclude: /node_modules/,
                use: {
                    loader: 'babel-loader',
                    options: {
                        presets: ['es2015','react']
                    }
                    //这里是需要用babel-preset-es2015和babel-preset-react来转换es6和react
                }
            }
        ]
    }
}
```

#### Example2 (css-loader)
在js文件中引入css文件，可以使用css-loader来对css文件进行预处理  
```
// main.js
require('./app.css');

// app.css
body {
  background-color: blue;
}

// index.html
<html>
  <head>
    <script type="text/javascript" src="bundle.js"></script>
  </head>
  <body>
    <h1>Hello World</h1>
  </body>
</html>

// webpack.config.js
module.exports = {
    entry: './mian.js',
    output: {
        filename: 'bundle.js'
    },
    module: {
        rules: [
            {
                test: /\.css$/,
                use: ['style-loader','css-loader']
                // css-loader用来读取css文件，style-loader用来将<style>tag插入html中
            }
        ]
    }
}
```
webpack处理之后的 `index.html`
```
<head>
  <script type="text/javascript" src="bundle.js"></script>
  <style type="text/css">
    body {
      background-color: blue;
    }
  </style>
</head>
```

#### Example3 (image loader (url-loader))
```
...
{
    test: /\.(png|jpg)$/,
    use: [
        {
            loader: 'url-loader',
            options: {
                limit: 8192
            }
        }
    ]
}
```
url-loader可以将js文件中引入的图片文件转为`<img src="xxx">`的形式，图片大小可以做出限制`limit`，当小于limit被转化为 Data URL，否则被转为普通url
```
<img src="data:image/png;base64,iVBOR...uQmCC">
<img src="4853ca667a2b8b8844eb2693ac1b2578.png">
```

### demo6 CSS Module
在js文件中require引入css文件，仅作用在该js文件内，但是可以通过使用`:global(selector)`
来将其本地范围的作用关掉，变成全局样式   
```
// index.html
<html>
<body>
  <h1 class="h1">Hello World</h1>
  <h2 class="h2">Hello Webpack</h2>
  <div id="example"></div>
  <script src="./bundle.js"></script>
</body>
</html>
```
```
// app.css
/* local scope */
.h1 {
  color:red;
}

/* global scope */
:global(.h2) {
  color: blue;
}
```
```
// main.jsx
var React = require('react');
var ReactDOM = require('react-dom');
var style = require('./app.css');

ReactDOM.render(
  <div>
    <h1 className={style.h1}>Hello World</h1>
    <h2 className="h2">Hello Webpack</h2>
  </div>,
  document.getElementById('example')
);
```
```
// webpack.config.js
module.exports = {
    entry: '.main.jsx',
    output: {
        filename: 'bundle.js'
    },
    module: {
        rules: [
            {
                test: /\.js[x]?$/,
                exclude: /node_modules/,
                use: {
                    loader: 'babel-loader',
                    options: {
                        presets: ['es2015','react']
                    }
                }
            },
            {
                test: /\.css$/,
                use: [
                    {loader: 'style-loader'},
                    {
                        loader: 'css-loader', 
                        options: {
                            modules: true
                        }
                    }
                ]
            }
        ]
    }
}
```

页面展示依次为h1 h2 h1 h2
但是只有第二个h1为红色，因为这个样式为本地范围内的；两个h2都是蓝色，该css为全局范围的

### demo7-demo8 Plugins 插件
Webpack通过plugin（插件）来扩展功能
Example1 (demo7)
例如可以通过UglifyJs Plugin来缩小bundle.js文件
```
var webpack = require('webpack)
var UglifyJsPlugin = require('uglifyjs-webpack-plugin')

module.exports = {
    entry: './main.js',
    output: {
        filename: 'bundle.js'
    },
    plugins: [
        new UglifyJsPlugin()
    ]
}
```
Example2 (demo8)
html-webpack-plugin 可以自动创建html文件，无需手写html
open-browser-webpack-plugin 可以在webpack运行时自动打开浏览器tab
```
// main.js

document.write('<h1>Hello World</h1>');
```
```
// webpack.config.js

var HtmlwebpackPlugin = require('html-webpack-plugin');
var OpenBrowserPlugin = require('open-browser-webpack-plugin');

module.exports = {
  entry: './main.js',
  output: {
    filename: 'bundle.js'
  },
  plugins: [
    new HtmlwebpackPlugin({
      title: 'Webpack-demos',
      filename: 'index.html'
    }),
    new OpenBrowserPlugin({
      url: 'http://localhost:8080'
    })
  ]
};
```


### demo9 Environment flags 环境标识
通过环境标识来使得某些代码仅在开发模式下运行  
```
// main.js

document.write('<h1>Hello World</h1>');
// 仅在本地开发模式下 运行的代码
if (__DEV__) {
  document.write(new Date());
}
```
在webpack中设置
```
var webpack = require('webpack');

var devFlagPlugin = new webpack.DefinePlugin({
  __DEV__: JSON.stringify(JSON.parse(process.env.DEBUG || 'false'))
});

module.exports = {
  entry: './main.js',
  output: {
    filename: 'bundle.js'
  },
  plugins: [devFlagPlugin]
};
```

设置package.json 将环境变量传递给webpack
```
{
  // ...
  "scripts": {
    "dev": "cross-env DEBUG=true webpack-dev-server --open",
  },
  // ...
}
```

### demo10-demo11 Code Splitting 代码分割

在构建大型app时，通常会将一整个大的js文件分割为多个小的js块，可以按需引入
#### Example1 (require.ensure)
webpack使用require.ensure来定义一个(打包)分离断点
```
// mian.js 
require.ensure(['./a'],function(require){
  var content = require('./a');
  document.open();
  document.write('<h1>' + content + '</h1>');
  document.close();
})
```
其他配置完全一样，但是`require.ensure`会告诉webpack将其引入的块单独分离出来打包  
最后的打包结果是webpack将main.js 和 a.js 分别打包在bundle.js 和 0.bundle.js中，并且按需加载0.bundle.js

#### Example2 (bundle-loader)
第二种方法就是使用bundle-loader

```
// main.js
var load = require('bundle-loader!./a.js')

load(function(file){
    ...
})
```

### demo12-demo13 Common chunk 共用模块
使用CommomsChunkPlugin将多个文件中共用重复的部分提取到一个特殊文件中，利于浏览器缓存及节省带宽
#### Example1 
```
// main1.jsx
var React = require('react')
var ReactDom = require('react-dom')

ReactDom.render(
    <h1>hello world</h1>,
    document.getElementById('a')
)

// main2.jsx
var React = require('react')
var ReactDom = require('react-dom')

ReactDom.render(
    <h2>hello world</h2>,
    document.getElementById('b')
)
```

```
// index.html
<html>
  <body>
    <div id="a"></div>
    <div id="b"></div>
    <script src="commons.js"></script>
    <script src="bundle1.js"></script>
    <script src="bundle2.js"></script>
  </body>
</html>
```
`common.js`就是提取的 main1.jsx 和 main2.jsx 的公共部分，（包含了react和reactdom）
```
// webpack.config.js
var webpack = require('webpack');

module.exports = {
  entry: {
    bundle1: './main1.jsx',
    bundle2: './main2.jsx'
  },
  output: {
    filename: '[name].js'
  },
  module: {
    rules:[
      {
        test: /\.js[x]?$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['es2015', 'react']
          }
        }
      },
    ]
  },
  plugins: [
    new webpack.optimize.CommonsChunkPlugin({
      name: "commons",
      // (the commons chunk name)

      filename: "commons.js",
      // (the filename of the commons chunk)
    })
  ]
}
```

#### Example2 （Vender chunk）

还可以利用CommonsChunkPlugin将类似与jquery这种公共库抽取出来  
```
// webpack.config.js

var webpack = require('webpack');
module.exports = {
  entry: {
    app: './main.js',
    vendor: ['jquery'],
  },
  output: {
    filename: 'bundle.js'
  },
  plugins: [
    new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor',
      filename: 'vendor.js'
    })
  ]
};
```

`vendor: ['jquery']`告知webpack将jquery抽出放在共用模块vender.js中

#### Others: 
ProvidePlugin插件可以自动加载模块，不需要每次都手动import或require

### demo14 Exposing global variables
想使用某个变量为全局变量，但是不想将其打包时，可以配置webpack.config.js 中的externals
```
// data.js
var data = 'Hello World';

// index.html
<html>
  <body>
    <script src="data.js"></script>
    <script src="bundle.js"></script>
  </body>
</html>
```
Webpack 仅构建 bundle.js，不会处理 data.js.  
将data暴露为全局变量：
```
// webpack.config.js
module.exports = {
  entry: './main.jsx',
  output: {
    filename: 'bundle.js'
  },
  module: {
    rules:[
      {
        test: /\.js[x]?$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['es2015', 'react']
          }
        }
      },
    ]
  },
  externals: {
    // require('data') is external and available
    //  on the global var data
    'data': 'data'
  }
};
```
```
// main.jsx
var data = require('data');
var React = require('react');
var ReactDOM = require('react-dom');

ReactDOM.render(
  <h1>{data}</h1>,
  document.body
);
```
在main.jsx中引入data.js模块，此时data作为全局变量使用了   
 
*可以将react、reactdom放在externals中，会大大降低build的时长和减小打包文件大小*