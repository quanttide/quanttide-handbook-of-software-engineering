# Flutter

Flutter 框架的默认选型约定。项目未特殊声明时，遵循以下默认。

## 状态管理

默认使用 Bloc。

## 路由

路由使用 Go router。

## 界面

默认使用 Material Design 库。

## Web 部署

Flutter Web 生产环境默认使用 CanvasKit 渲染器，页面初始化时会从 `gstatic.com`（Google 域）下载渲染引擎的 wasm 文件，约 6MB。国内网络对 `gstatic.com` 基本不可达，由此产生的白屏表现为：页面 HTML 秒开，`main.dart.js` 加载完成后卡在 CanvasKit 下载，画面一直空白。是否走代理决定了该域是否可达，因此问题表现为时好时坏。

处理方法是构建时将 CanvasKit 指向自托管地址，例如：

```bash
flutter build web --dart-define=FLUTTER_WEB_CANVASKIT_URL=/canvaskit/
```

并将对应版本的 CanvasKit 资源部署到站点该路径。资源可从 Flutter SDK 缓存目录（`bin/cache/artifacts/engine/*/canvaskit/`）或 npm 包 `canvaskit-wasm` 获取，注意版本需与构建使用的 Flutter 引擎版本一致。

发布 Flutter Web 应用前，应在无代理的网络环境下验证页面可以完整渲染。

其余约定后续补充。
