# taro-native-shell
Taro 原生 React Native 壳子

## 启动代码编译及 Metro Bundler Server

运行 `taro build` 命令，Taro 将会开始编译文件：

```sh
➜  taro-demo git:(master) ✗ taro build --type rn --watch
👽 Taro v1.2.20

开始编译项目 taro-demo
编译  JS        /Users/chengshuai/Taro/taro-demo/src/app.js
编译  SCSS      /Users/chengshuai/Taro/taro-demo/src/app.scss
拷贝  HTML      /Users/chengshuai/Taro/taro-demo/src/index.html
生成  生成文件  /Users/chengshuai/Taro/taro-demo/rn_temp/app_styles.js
编译  JS        /Users/chengshuai/Taro/taro-demo/src/pages/index/index.js
编译  SCSS      /Users/chengshuai/Taro/taro-demo/src/pages/index/index.scss
生成  index.js  /Users/chengshuai/Taro/taro-demo/rn_temp/index.js
生成  app.json  /Users/chengshuai/Taro/taro-demo/rn_temp/app.json
生成  package.json  /Users/chengshuai/Taro/taro-demo/rn_temp/package.json
编译  编译完成，花费2504 ms
生成  生成文件  /Users/chengshuai/Taro/taro-demo/rn_temp/pages/index/index_styles.js

初始化完毕，监听文件修改中...

```


如果编译没有报错，会自动打开一个终端，并在 8081 端口启动 [Metro](https://github.com/facebook/metro) Bundler 负责打包 jsbundle：

![image](https://user-images.githubusercontent.com/9441951/59322399-85780180-8d08-11e9-9ea7-b3e4b23c077c.png)

这时，在浏览器输入 http://127.0.0.1:8081，可以看到如下页面：

![image](https://user-images.githubusercontent.com/9441951/55865494-13245d00-5bb1-11e9-9a97-8a785a83b584.png)

输入 http://127.0.0.1:8081/rn_temp/index.bundle?platform=ios&dev=true 会触发对应终端平台的 js bundle 构建。

![image](https://user-images.githubusercontent.com/9441951/55865039-37336e80-5bb0-11e9-8aca-c121be4542f6.png)

构建完成后，浏览器会显示构建后的 js 代码。

> Note：进入下一步之前请确保 Metro Bundler Server 正常启动，即浏览器能正常访问访问 jsbundle。


### 启动应用
如果上一步的编译和 Metro Bundler Server 启动没问题，接下来就可以启动应用了。

开发者可以自行整合 React Native (0.55.4) 到原生应用，同时为了方便大家开发和整合，Taro 将 React Native 工程中原生的部分剥离出来，单独放在一个工程里面 [NervJS/taro-native-shell](https://github.com/NervJS/taro-native-shell)，你可以把它看成是 React Native iOS/Android 空应用的壳子。

首先将应用代码 clone 下来：

```
git clone git@github.com:NervJS/taro-native-shell.git
```
然后 `cd taro-native-shell`，使用 yarn 或者 npm install 安装依赖。

工程目录如下：

```sh
➜  taro-native-shell git:(master) ✗ tree -L 1
.
├── LICENSE
├── README.md
├── android // Android 工程目录
├── ios // iOS 工程目录
├── node_modules
├── package.json
└── yarn.lock
```

### iOS
### 安装依赖
在 iOS 目录运行
```sh
$ pod install 
```
如果没有安装 CocoaPods，可以参考官方文档：[Getting Started
](https://guides.cocoapods.org/using/getting-started.html)


#### 使用 React Native 命令启动

```sh
$ react-native run-ios
```

iOS 模拟器会自行启动，并访问 8081 端口获取 js bundle，这时 Metro Bundler 终端会打印以下内容：

```sh
 BUNDLE  [ios, dev] ./index.js ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓ 100.0% (1/1), done.
```

#### 使用 Xcode 启动
iOS 的启动比较简单，使用 Xcode 打开 ios 目录，然后点击 Run 按钮就行。

![image](https://developer.apple.com/library/archive/documentation/ToolsLanguages/Conceptual/Xcode_Overview/Art/XC_O_SchemeMenuWithCallouts_2x.png)

这里需要注意的是 jsBundle 的 moduleName，默认的 moduleName 为 "taroDemo"，需要和 `rn_temp/app.json` 里面的 name 字段保持一致。该配置在 `AppDelegate.m` 文件中。

```objc
@implementation AppDelegate

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
  NSURL *jsCodeLocation;

  jsCodeLocation = [[RCTBundleURLProvider sharedSettings] jsBundleURLForBundleRoot:@"rn_temp/index" fallbackResource:nil];

  RCTRootView *rootView = [[RCTRootView alloc] initWithBundleURL:jsCodeLocation
                                                      moduleName:@"taroDemo"
                                               initialProperties:nil
                                                   launchOptions:launchOptions];
  rootView.backgroundColor = [[UIColor alloc] initWithRed:1.0f green:1.0f blue:1.0f alpha:1];

  self.window = [[UIWindow alloc] initWithFrame:[UIScreen mainScreen].bounds];
  UIViewController *rootViewController = [UIViewController new];
  rootViewController.view = rootView;
  self.window.rootViewController = rootViewController;
  [self.window makeKeyAndVisible];
  return YES;
}

@end
```

app.json 字段的配置默认取自于 package.json 的 name 字段，除非你在 rn -> appJson 里面有配置。

更多资料，可以查看 Xcode 文档：[Building Your App](https://developer.apple.com/library/archive/documentation/ToolsLanguages/Conceptual/Xcode_Overview/BuildingYourApp.html)

### Android 
在 taro-native-shell/android 目录下，你就可以看到 React Native 的工程代码。

#### 使用 React Native 命令启动

```sh
$ react-native run-android
```

Android 模拟器会自行启动，并访问 8081 端口获取 js bundle，这时 Metro Bundler 终端会打印一下内容：

```sh
 BUNDLE  [android, dev] ./index.js ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓ 100.0% (1/1), done.
```


#### 在真实设备上运行

按照以下步骤设置您的设备：

1. 使用一根 USB 电缆将您的设备连接到您的开发机器。如果您是在 Windows 上开发，可能需要为您的设备 [安装相应的 USB 驱动程序](https://developer.android.com/studio/run/oem-usb.html?hl=zh-cn)。
2. 按照以下步骤操作，在 Developer options 中启用 USB debugging。
首先，您必须启用开发者选项：

	1. 打开 Settings 应用。
	2. （仅在 Android 8.0 或更高版本上）选择 System。
	3. 滚动到底部，然后选择 About phone。
	4. 滚动到底部，点按 Build number 7 次。
	5. 返回上一屏幕，在底部附近可找到 Developer options。
打开 Developer options，然后向下滚动以找到并启用 USB debugging。

按照以下步骤操作，在您的设备上运行应用：

1. 在 Android Studio 中，点击 Project 窗口中的 app 模块，然后选择 Run > Run（或点击工具栏中的 Run  ）。

![image](https://sdtimes.com/wp-content/uploads/2016/04/0408.sdt-androidstudio.png)

2. 在 Select Deployment Target 窗口中，选择您的设备，然后点击 OK。

![image](https://developer.android.com/training/basics/firstapp/images/run-device_2x.png?hl=zh-cn)

Android Studio 会在您连接的设备上安装并启动应用。

### 在模拟器上运行
按照以下步骤操作，在模拟器上运行应用：

1. 在 Android Studio 中，点击 Project 窗口中的 app 模块，然后选择 Run > Run（或点击工具栏中的 Run  ）。
2. 在 Select Deployment Target 窗口中，点击 Create New Virtual Device。

![image](https://developer.android.com/training/basics/firstapp/images/run-avd_2x.png?hl=zh-cn)

3. 在 Select Hardware 屏幕中，选择电话设备（如 Pixel），然后点击 Next。
4. 在 System Image 屏幕中，选择具有最高 API 级别的版本。如果您未安装该版本，将显示一个 Download 链接，因此，请点击该链接并完成下载。
5. 点击 Next。
6. 在 Android Virtual Device (AVD) 屏幕上，保留所有设置不变，然后点击 Finish。
7. 返回到 Select Deployment Target 对话框中，选择您刚刚创建的设备，然后点击 OK。

Android Studio 会在模拟器上安装并启动应用。

#### Module Name

同样，Android 这边默认的 jsBundle moduleName 也是 “taroDemo”，位于 `MainActivity.java` 的文件里面：

```java
package com.tarodemo;

import com.facebook.react.ReactActivity;

public class MainActivity extends ReactActivity {

    /**
     * Returns the name of the main component registered from JavaScript.
     * This is used to schedule rendering of the component.
     */
    @Override
    protected String getMainComponentName() {
        return "taroDemo";
    }
}

```

你可以根据实际情况自行修改。

