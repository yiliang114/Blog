# 移除 mobx vscode 装饰器报错

不知各位有没有在使用 vscode 写 mobx+react 的时候，遇到过 `experimentalDecorators` warning？ 我花了一点时间来解决这个问题，希望你看了下文之后能够不会浪费自己宝贵的时间。

## 问题截图

![1](https://upload-images.jianshu.io/upload_images/4907895-dddf5cfa9ed3800a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![2](https://upload-images.jianshu.io/upload_images/4907895-82cfc9a9ef5dc5ed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当我在 vscode 新创建一个 react+mobx 项目的时候，遇到了下面这个警告。

**Experimental support for decorators is a feature that is subject to change in a future release. Set the ‘experimentalDecorators’ option to remove this warning.**

每当我新引入`MobX`的 `@observable` 装饰器时，vscode 并不识别，并将相关的 react class 以及 声明的 observable 属性都下划红线。

不过 webpack 编译项目的时候并没有错误，只是 vscode 一直下划线警告我，很难受。

下面说一个解决办法：

## 解决办法

![3](https://upload-images.jianshu.io/upload_images/4907895-aa89ff2332781fcf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在项目的根目录创建一个`tsconfig.json`，并在文件里输入下面的配置：

```
{
    "compilerOptions": {
        "experimentalDecorators": true,
        "allowJs": true
    }
}
```

重启一下 vscode 或者 关闭文件 tab 重新打开之后，你就应该看不到`experimentalDecorators` 警告了。

![4](https://upload-images.jianshu.io/upload_images/4907895-ad9e1c75d5469abd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

希望对你有用，感谢阅读。
