# Eagle-Plugin-Lottie-JSON-Converter
> Eagle Lottie JSON 转 dotLottie 插件

## 开发
### 安装依赖
```shell
pnpm i # npm i
```
### 本地运行
```shell
pnpm dev # npm run dev
```
### 打包编译
```shell
pnpm build # npm run build
```
### 升级依赖
```shell
pnpm run up # npm run up
```
## 本地开发调试
1. 在 [安装依赖](#安装依赖) 完成后，运行 [打包编译](#打包编译)，编译文件位于项目根目录的 `/build` 目录下，打开 Eagle 插件面板，点击 `开发者选项` → `导入本地项目`，选择编译的 `/build` 目录，插件会显示在 Eagle 插件面板的 `开发` 分类下。
2. 运行 [本地运行](#本地运行)，项目会默认运行在 `5173` 端口。
3. 此时在 Eagle 插件面板点击打开插件，出现插件窗口后，按 `F12` 呼出 `DevTools`，在 `DevTools` 中切换至 `控制台(Console)` 界面，输入 `location.href = "http://localhost:5173"` 后回车运行。
4. 稍等一会等插件界面完整显示，在 `DevTools` 中按 `F5` 刷新页面，此时插件窗口就具备实时热更新的本地开发环境了。
## 说明
* 关于插件开发或`Eagle`相关问题，可前往 [Eagle 插件开发群](https://discord.gg/eGFYpRx7x4) 进行交流或反馈。
