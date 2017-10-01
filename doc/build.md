---
---
# 构建一个 Tabris.js 应用

Tabris.js 利用[Apache Cordova](http://cordova.apache.org)来构建和打包应用。在tabrisjs.com [使用免费在线的构建服务](#build-service)，可以帮你不进行任何本地安装的构建应用。想要[在你自己的本机上构建应用](#local-build)，你需要安装像Xcode，Visual Studio或者Android SDK此类的开发者工具。这两种不同的构建方式支持如下不同的特性。

|                           | 构建服务 | 本机构建 |
| :------------------------ |:---------------:| :---------------: |
| 构建 iOS 应用         |       ✓         |       ✓      |
| 构建 Android 应用     |       ✓         |       ✓      |
| 构建 Windows 应用     |       ✓         |       ✓      |
| [集成 Cordova 插件](cordova.md)     |       ✓      |       ✓      |
| [Cordova 构建钩子（Hooks）](http://cordova.apache.org/docs/en/edge/guide_appdev_hooks_index.md.html#Hooks%20Guide)       |       ✓      |       ✓      |
| 自定义项目结构  |       ✓      |       ✓      |
| 自定义构建脚本         |              |       ✓      |
| 使用自己的构建硬件  |              |       ✓      |
| 除了Git的管理系统       |              |       ✓      |

> :point_right: 线上构建服务对无限制的公共 Github 库和一个私有库是免费的。为了从无限制的私有库构建应用，你需要一个[Pro 通行证](https://tabrisjs.com/pricing/)。[Local builds](#local-build)对每个人都是免费的。

> :point_right: 构建 Windows 应用时，请阅读相关[Windows 支持文档](windows-support.md)。

## 项目结构

构建的准备工作，你需要创建一个叫`cordova`的子目录，其包含有构建配置。一个Tabris.js项目的结构看起来可能像这样：
```
/
|- cordova/
    |- config.xml
|- src/
    |- <.js files>
|- test/
    |- <.spec.js files>
|- package.json
|- .tabrisignore
```

### package.json 文件

查阅 [package.json | npm documentation](https://docs.npmjs.com/files/package.json) 获取关于`package.json`文件格式的更多信息。

`package.json` 包含除了一些其他配置在内的应用的主脚本和npm包依赖。

```json
{
  // ...
  "main": "src/app.js",
  "dependencies": {
    "tabris": "^2.0.0",
    "left-pad": "^1.1.3",
    // ...
  }
  // ...
}
```

构建过程中依赖会被自动安装。

#### 构建脚本

Tabris.js应用构建时，`package.json`中的`build`脚本会在JavaScript代码被打包进应用前被执行。他们可以被用来转义（transpile）JavaScript应用代码。

```json
{
  ...
  "scripts": {
    ...
    "build": "..."
  }
  ...
}
```
支持的构建钩子有：

  - `"build"`: 运行所有平台构建
  - `"build:android"`: 运行Android构建
  - `"build:ios"`: 运行iOS构建
  - `"build:windows"`: 运行Windows构建

在执行`tabris serve`前（本机运行时）请确保已经运行了`build`脚本。在使用转译器（transpiler）的监测模式（watch mode）时这一步会被自动执行，监测模式会自动编译变动的文件（您的转译器的文件）。像[npm-run-all](https://www.npmjs.com/package/npm-run-all)这样的模块能帮你运行监测模式的同时运行`tabris serve`。

对于更大型的项目，你可能需要使用像[rollup.js](https://rollupjs.org)这样的模块打包工具。它会消除多模块加载的系统开箱并且使用静态分析来移除打包产出中的无用代码。

#### 示例: 转译 ES6 代码

安装Babel转译器和必要的插件。`--save-dev`配置会将依赖项添加到你的`package.json`中

```
npm install --save-dev babel-cli babel-plugin-transform-es2015-modules-commonjs
```

在你项目的跟路径下创建一个`.babelrc`文件

```json
{
  "plugins": ["transform-es2015-modules-commonjs"]
}
```
在你的`package.json`中的`scripts`部分添加如下构建脚本：

```json
{
  "scripts": {
    "build": "babel --compact false --out-dir dist/ src/"
  }
  ...
}
```
让`main`字段指向`dist/`下转译好的`app.js`文件

```json
{
  "main": "dist/app.js",
  ...
}
```

考虑iOS9的支持，你需要安装额外的Babel插件来支持缺失的ES6特性。
查阅[EcmaScript 6](lang.md#ecmascript-6)获取有关iOS9中支持ES6特性的更多信息。

#### Example: 转译 TypeScript 代码

安装 TypeScript 编译器:

```
npm install --save-dev typescript
```

在你的`package.json`中的`scripts`部分添加如下构建脚本：

```json
{
  "scripts": {
    "build": "tsc -p ."
  }
  ...
}
```
让`main`字段指向`dist/`下转译好的`app.js`文件

```json
{
  "main": "dist/app.js",
  ...
}
```

### config.xml 文件

你所需的最简单的构建配置是一个描述应用的`cordova/config.xml`文件。包含像应用id、版本、图标、启动画面此类信息。`config.xml`的格式和标准[Cordova config.xml](https://cordova.apache.org/docs/en/4.0.0/config_ref_index.md.html#The%20config.xml%20File)文件相同。最简单的示例配置看起来像这样：

```xml
<?xml version='1.0' encoding='utf-8'?>
<widget id="my.first.app" version="1.0.0">
  <name>HelloWorld</name>
  <description>
    A sample Tabris.js application.
  </description>
  <author email="dev@tabrisjs.com" href="https://tabrisjs.com">
    Tabris.js Team
  </author>
</widget>
```

#### 集成 Cordova 插件

你只需要向`config.xml`中插入`<plugin />`标签就能添加一套`Apache Cordova`插件。允许你使用ID、HTTP URL或git URL的方式添加插件。

举个栗子，为了添加[Cordova Camera 插件](http://plugins.cordova.io/#/package/org.apache.cordova.camera)，你需要添加这一行：

```xml
<plugin name="cordova-plugin-camera" spec="^2.3.0" />
```

使用将其添加进`config.xml`的方式你可以集成所有可用的[Cordova 插件](http://plugins.cordova.io/#/)。

**Important:** 你可以安装所有可用的Cordova插件。大部分插件开箱即用。然而，由于Tabris.js使用了**native UI** 和 **no HTML5**，基于HTML5 UI (即DOM)的插件都会无效（Tabris.js是无dom的）。


#### content 元素

可选的`<content>`元素在传统的Cordova应用中定义了应用的起始页。在Tabris.js中你可以用`package.json`文件的路径位置来替代。示例：

```xml
<content src="mySubFolder/package.json" />
```

#### 首选项

除了[Cordova config.xml Guide](http://cordova.apache.org/docs/en/dev/config_ref/)中提到的设置，Tabris.js还接受以下自定义首选项：

| 名称                   | 允许值 | 默认值 | 描述 |
|------------------------|----------------|---------------|-------------|
| EnableDeveloperConsole | true/false     | false         | 启用/禁用[Tabris.js Developer Console](getting-started.md#the-developer-console)。将值设置为`$IS_DEBUG`会使其与[debug mode](#settings)的值同步|
| UseStrictSSL           | true/false     | true          | 激活/停用[XHR](w3c-api.md#xmlhttprequest)的SSL证书验证。禁用时，自签名SSL证书被允许。生产环境是应启用。 |

示例:
```xml
<preference name="EnableDeveloperConsole" value="true" />
```

#### Android 特定首选项

| 名称                    | 值 |
|-------------------------|-------|
| Theme                   | <ul><li>`@style/Theme.Tabris`</li><li>`@style/Theme.Tabris.Light`</li><li>`@style/Theme.Tabris.Light.DarkAppBar` (Default)</ul>除了捆绑的Tabris主题，还可以制定自定义的Android主题的引用。自定义主题需要继承自Tabris基础主题。<br/><br/>示例: `<preference name="Theme" value="@style/Theme.MyAppTheme" />` |
| ThemeSplash             | <ul><li>`@style/Theme.Tabris.SplashScreen`</li><li>`@style/Theme.Tabris.Light.SplashScreen` (默认)</ul>启动画面会在应用启动时展示给用户。磨人的画面是一个白屏。`ThemeSplash`首选项允许设置一个捆绑的主题或提供一个自定义主题。<br/><br/>示例: `<preference name="ThemeSplash" value="@style/Theme.Tabris.SplashScreen" />`<br/><br/>注意，`config.xml` 元素 `<splash .. />` 可以用来给启动画面设置图像。查阅[launch screens](https://material.google.com/patterns/launch-screens.html)的`material design`指南获取样式设计指导。 |

#### Windows 特定首选项

Windows 应用通常都一个启动画面。如果不配置，会使用默认的Tabris.js启动画面。自定义启动画面时，你需要提供三种不同分辨率和背景色下的logo。文件名需个如下名称对应：

```xml
<platform name="windows">
    <splash src="resources/windows/splash/SplashScreen.scale-100.png" width="620" height="300"/>
    <splash src="resources/windows/splash/SplashScreen.scale-150.png" width="930" height="450"/>
    <splash src="resources/windows/splash/SplashScreen.scale-200.png" width="1240" height="600"/>
    <preference name="SplashScreenBackgroundColor" value="#009688"/>
</platform>
```

替换启动器标题、Windows商城和任务图标中的tabris标志，你需要提供如下所有文件。同样的，名称也需要对应。

```xml
<platform name="windows">
    <icon src="res/windows/storelogo.png" target="StoreLogo" />
    <icon src="res/windows/smalllogo.png" target="Square30x30Logo" />
    <icon src="res/Windows/Square44x44Logo.png" target="Square44x44Logo" />
    <icon src="res/Windows/Small71x71Logo.png" target="Square71x71Logo" />
    <icon src="res/Windows/Square150x150Logo.png" target="Square150x150Logo" />
    <icon src="res/Windows/Large310x310Logo.png" target="Square310x310Logo" />
    <icon src="res/Windows/Wide310x150Logo.png" target="Wide310x150Logo" />
</platform>
```

### .tabrisignore 文件

tabris.js将项目的内容打包进应用中。你可以排除打包应用程序中不需要的某些文件或目录，例如测试或开发人员文档。要被构建忽略的文件和目录可以列在名为`.tabrisignore`的文件中。这个忽略文件的格式与[`.gitignore`](http://git-scm.com/docs/gitignore)文件遵从相同的规则。

如下的文件不列入`.tabrisignore`中也会被默认排除：

* `.git/`
* `node_modules/`
* `build/`
* The file `.tabrisignore` itself

## 构建服务

[Tabrisjs.com](https://tabrisjs.com) 为Tabris.js应用提供免费的在线构建服务。在登录后，你可以在“My Apps”部分点击“Create App”来创建一个应用。现在你可以从库列表（如果不可见，你需要按“synchronize/同步”按钮）中选取你的GitHub库。[Pro 计划](https://tabrisjs.com/pricing/)中的用户也可以使用自己的Git库。
![Create an App](img/build-create-app.png)
选择库之后，将会启动一个验证，此验证会监测选中的库是否含有一个合法的Tabris.js[项目结构](build.md#project-layout)。如果你有一个合法的项目结构和`config.xml`，你的应用很快就会通过验证。如果不合法，此网站会提示你哪里出错。这种情况下，请跟随显示说明。
![Valid App](img/build-valid-app.png)
在你的应用合法后，你已经可以进行第一次构建。只需选中新创建的应用然后点击"Start Android Build"按钮。几分钟后，你会得到一个可以在你设备上安装的Andriod .apk文件。但是iOS、生产构建和签名呢？所有的这些都可以使用“Settings”进行设置。

> :point_right: 构建服务会从npm安装你的`package.json`中的依赖项（）。The build service installs the dependencies specified in your package.json from npm (devDependencies除外)。所以，你不需要将`node_modules`文件放入版本控制中。

### Settings

![App Settings](img/build-app-settings.png)

* **Repository URL:** This is the URL of your git repository. If you're using the free build, it should point to a GitHub repository. Users who are on the [Pro plan](https://tabrisjs.com/pricing/) can also use custom repository locations.
* **SSH Private Key:** A SSH private key to access your repository. Only relevant for git repositories which are not hosted on GitHub.
* **Branch:** The git branch to build from. The default value is `master`. If you want to build from a feature branch, you may specify the branch here.
* **App Directory:** The directory within your repository that contains your Tabris.js app. The value must be relative to the repository root.
* **iOS Signing Key:** iOS apps can not be deployed to a mobile device without being signed. If you want to build an iOS app you need an Apple Developer account and provide the certificate together with the provisioning profile. A very good tutorial on how to get these files can be found in the [Phonegap Build documentation](http://docs.build.phonegap.com/en_US/signing_signing-ios.md.html#iOS%20Signing).
* **Android Signing Key:** Android apps need to be signed with a certificate only if you want to deploy them to Play Store. You can find a very good tutorial in the [Phonegap Build documentation](http://docs.phonegap.com/phonegap-build/signing/android/) as well.
* **Windows Architecure** Choose which CPU architecture you want to build your package for.
* **Environment Variables:** Key/Value pairs that will be stored and transferred encrypted to the build machines. They can be used within the config.xml or custom hooks. Use cases are adding plug-ins from private git repositories or handling access keys.
* **Builds to keep:** Specifies the number of builds that should be kept before deleting them automatically.
* **Tabris.js Version:** The Tabris.js *client* version to use in the app. In contrast to the "tabris" dependency to your `package.json` which defines the version of the JavaScript module, this setting defines the version of the native client that will interpret your JavaScript code. In most cases, the value `latest` is good enough here. But if you want to stick to a fixed Tabris.js version you can configure it here.
* **Debug:** Enables the *debug mode*. If set to `ON`, your app will be built including debug symbols and it will be packaged into the Tabris.js Developer App to make development easier. This allows you to use all the benefits like the developer console or the reload also with your own app. Please be aware that debug versions can not be submitted to the app stores. Debug `OFF` means your app will be built to be ready for release: no Developer App, no console, no reload. Only your JavaScript code is executed.

## Local Build

You can build Tabris.js apps on your local machine using the [Tabris CLI](https://www.npmjs.com/package/tabris-cli).

### Prerequisites

To build apps on your machine, the development environment for the target platform must be installed.
If you're targeting iOS, you need macOS with a recent version of Xcode.
For Windows, you need a Windows PC with Visual Studio 2017.
Android apps can be build on any OS with the latest Android SDK installed.

The Tabris CLI must be installed globally on your system:

```
npm install -g tabris-cli
```

You also need an account on [tabrisjs.com](https://tabrisjs.com). A free account is sufficient.

### Building an App

On the first start, the Tabris CLI will ask for a build key.
You can find this key on your [profile page](https://tabrisjs.com/settings/account).

To build your app, run

```
tabris build [android|ios|windows]
```

For more command-line options, please refer to the [CLI documentation](https://www.npmjs.com/package/tabris-cli).
