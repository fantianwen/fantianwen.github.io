title: ReactNative原生移植
date: 2016-07-01 10:02:00
tags: [ReactNative,android]
---

[ReactNative原生移植](http://www.lcode.org/react-native%E7%A7%BB%E6%A4%8D%E5%8E%9F%E7%94%9Fandroid%E9%A1%B9%E7%9B%AE-%E5%B7%B2%E6%9B%B4%E6%96%B0%E7%89%88%E6%9C%AC/)


## 几点注意：

### 1、如果使用`npm install`生成`package.json`文件，本身是缺少

```
"dependencies": {
    "react": "xx.x.x",
    "react-native": "x.x.x"
  }
```
的，需要自己添加，并且版本号参考init的那个Awesome的项目的配置，保证不会出错，这个配错了会很麻烦。

> 如果在已经使用`npm install`生成module文件后，修改上面的react的依赖的版本，那么需要重新使用`npm install`重新生成以更新依赖模块。

<!-- more -->

### 2、提示`Overlay Manager。。。。`这样的错误，需要授予app悬浮框的权限

### 3、错误：

```
Application reacthelloone has not been registered. This is either due to a require() error during initialization or failure to call AppRegistry.registerComponent.
```

首先保证`package.json`中的`name`字段和`index.android.js`中
```
AppRegistry.registerComponent('reacthelloone', () => MyAwesomeApp);
```
注册的模块名称是一样的。


然后还是不行的话，[这里](http://blog.xjspace.net/2016/03/react-native-application-has-not-been-registered/),保证`npm start`开启的是本app的开发服务器。







