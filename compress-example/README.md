# Webpack 性能优化 （二） 压缩 JS

大家新年好 ^^，这是 [Webpack 性能优化](https://github.com/wyvernnot/webpack_performance) 系列文章的第二篇。
在这篇里我们来通过一个实际的例子来看看如何在 Webpack 打包的时候启用压缩。

这个例子是由 Github 网友 `@zjy01` 提供的: [react_and_redux_and_router_example](https://github.com/zjy01/react_and_redux_and_router_example)

**在不做任何更改的情况下**

```sh
webpack --config webpack.static.config.js
```

结果：

```txt
Time: 1726ms
    Asset     Size  Chunks             Chunk Names
bundle.js   949 kB    0, 1  [emitted]  bundle
common.js  3.55 kB       1  [emitted]  common.js
    + 257 hidden modules
```

输出文件加起来将近 1 MB。

打开 `bundle.js`，发现所有的换行和注释都保留着，所有的变量名都原汁原味，这些问题很容易通过 `uglify` 来解决。
但是由于 `uglify` 以后代码可读性就会大大降低，这样就不容易发现隐藏的其它问题，所以我们先不急着 `uglify` 而是先看看生成的代码。

不难发现，为了调试的需要，React 发布在 NPM 上的源码里有很多地方检测了当前环境。

```js
if (process.env.NODE_ENV !== 'production') {
  // 如果不是产品环境就怎样怎样
}
```

## 使用 DefinePlugin 消除环境监测的代码

通过编辑 webpack 的配置文件，加上 DefinePlugin，把 process.env.NODE_ENV 替换掉

```js
{
 plugins:[
    ...
    new webpack.DefinePlugin({
      'process.env': {
          'NODE_ENV': JSON.stringify(process.env.NODE_ENV)
      }
    })
 ]
}
```

再次执行

```sh
NODE_ENV=production webpack --config webpack.static.config.js
```

结果：

```txt
Time: 1859ms
    Asset     Size  Chunks             Chunk Names
bundle.js   932 kB    0, 1  [emitted]  bundle
common.js  3.55 kB       1  [emitted]  common.js
    + 256 hidden modules
```

仅仅减小了 16 kB左右，这是因为 DefinePlugin 只会做简单的替换，之前有关环境检测的代码替换后如下：

```js
if (false) {
  // 如果不是产品环境就怎样怎样
}
```

实际上 if 块里的代码已经变成了 `dead code` ，即死区，永远都不会被执行到。而清除代码死区正是 `uglifyjs` 非常擅长的地方。

## 使用 uglifyjs 压缩代码

在前端打包的流程中，`uglify` 操作是如此的常见以至于被内置进了 `webpack` 里，使用的时候加上 `--optimize-minimize` 参数即可。

```sh
NODE_ENV=production webpack --config webpack.static.config.js --optimize-minimize
```

结果：

```txt
Time: 4059ms
    Asset       Size  Chunks             Chunk Names
bundle.js     223 kB    0, 1  [emitted]  bundle
common.js  735 bytes       1  [emitted]  common.js
    + 252 hidden modules
```

## 关于

webpack 版本 1.12.14

## 总结

通过这篇文章，简单介绍了如何在 Webpack 打包的时候启用压缩，重点在于消除环境相关的代码。