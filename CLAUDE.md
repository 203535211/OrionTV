# CLAUDE.md

本文件用于指导 Claude Code（claude.ai/code）在处理此代码库时的工作。

## 项目概述

云朵TV（YunDuoTV）是一款基于 React Native TVOS 的视频播放应用，使用 Expo 构建，专为电视平台（Apple TV 和 Android TV）设计。这是一个纯前端应用，通过对接外部 API 获取数据，并内置遥控服务端，支持外部设备控制。

## 常用命令

### 开发命令

#### 电视端开发（Apple TV & Android TV）
- `yarn start` – 在电视模式下启动 Metro 打包工具（EXPO_TV=1）
- `yarn android` – 编译并运行在 Android TV 上
- `yarn ios` – 编译并运行在 Apple TV 上
- `yarn prebuild` – 为电视平台生成原生项目文件（修改依赖后需执行）
- `yarn build` – 编译电视端发布版 Android APK

#### 测试命令
- `yarn test` – 以监听模式运行 Jest 测试
- `yarn test-ci` – 为 CI 环境运行 Jest 测试并生成覆盖率
- `yarn test utils` – 对指定目录/文件运行测试
- `yarn lint` – 执行 ESLint 代码检查
- `yarn typecheck` – 执行 TypeScript 类型检查

#### 编译与部署
- `yarn copy-config` – 复制电视专属的 Android 配置
- `yarn build-debug` – 编译调试版 Android APK
- `yarn clean` – 清理缓存与编译产物
- `yarn clean-modules` – 重新安装所有 node 模块

## 架构概述

### 多平台响应式设计

云朵TV 实现了完善的多端适配架构：
- **设备识别**：基于宽度断点（手机 <768px，平板 768-1023px，电视 ≥1024px）
- **组件变体**：通过 `.tv.tsx`、`.mobile.tsx`、`.tablet.tsx` 后缀区分平台
- **响应式工具**：`DeviceUtils` 和 `ResponsiveStyles` 用于自适应布局与缩放
- **自适应导航**：不同设备使用不同交互模式（触屏 vs 遥控器）

### 状态管理架构（Zustand）

按业务领域拆分 Store，模式统一：
- **homeStore.ts** – 首页内容、分类、豆瓣 API 数据、播放记录
- **playerStore.ts** – 视频播放器状态、控制、剧集管理
- **settingsStore.ts** – 应用设置、API 配置、用户偏好
- **remoteControlStore.ts** – 遥控服务功能与 HTTP 桥接
- **authStore.ts** – 用户登录状态
- **updateStore.ts** – 自动更新检查与版本管理
- **favoritesStore.ts** – 用户收藏管理

### 服务层模式

各服务模块职责清晰分离：
- **api.ts** – 外部 API 集成，带错误处理与缓存
- **storage.ts** – AsyncStorage 封装，带类型接口
- **remoteControlService.ts** – 基于 TCP 的 HTTP 服务，用于外部设备控制
- **updateService.ts** – 版本自动检查与 APK 下载管理
- **tcpHttpServer.ts** – 底层 TCP 服务实现

### 电视遥控系统

完善的电视端交互处理：
- **useTVRemoteHandler** – 统一处理电视遥控器事件的 Hook
- **硬件事件**：处理电视专属控制（播放/暂停、进度跳转、菜单）
- **焦点管理**：电视端专属焦点状态与导航流程
- **手势支持**：长按、方向键快进快退、控件自动隐藏

## 核心技术栈
- **React Native TVOS (0.74.x)** – 电视端优化版 React Native，支持电视专属事件
- **Expo SDK 51** – 开发平台，提供原生能力与编译工具
- **TypeScript** – 完整类型安全，配置 `@/*` 路径别名
- **Zustand** – 轻量级全局状态管理
- **Expo Router** – 文件式路由系统，支持类型路由
- **Expo AV** – 视频播放，带电视端优化控制条

## 开发流程

### 电视优先开发模式

项目采用「电视优先」策略，再适配其他端：
- **主要目标**：Apple TV、Android TV（遥控器交互）
- **次要目标**：手机、平板（触屏优化响应式）
- **编译环境**：`EXPO_TV=1` 环境变量开启电视专属功能
- **组件策略**：公共组件 + 平台变体（文件后缀区分）

### 测试策略
- **单元测试**：工具类完整测试（`utils/__tests__/`）
- **Jest 配置**：使用 Expo 预设 + Babel 转译
- **测试模式**：对 React Native 模块与外部依赖做 Mock 测试
- **覆盖率报告**：兼容 CI 的详细覆盖率报告

### 重要开发注意事项
- 添加新依赖后需执行 `yarn prebuild` 生成原生文件
- 使用 `yarn copy-config` 应用电视专属 Android 配置
- 电视组件必须处理焦点与遥控器支持
- 需在电视设备（Apple TV/Android TV）与移动端/平板端分别测试布局
- 所有 API 请求统一放在 `/services` 目录，带错误处理
- 存储操作使用 `storage.ts` 中的 AsyncStorage 封装，带类型接口

### 组件开发规范
- **平台变体**：使用 `.tv.tsx`、`.mobile.tsx`、`.tablet.tsx` 实现平台专属逻辑
- **响应式工具**：使用 `DeviceUtils.getDeviceType()` 做响应式判断
- **电视遥控**：使用 `useTVRemoteHandler` Hook 处理电视交互
- **焦点管理**：电视组件必须实现焦点状态，支持遥控器导航
- **公共逻辑**：通用逻辑放在 `/hooks` 目录复用

## 常见开发任务

### 添加新组件
1. 在 `/components` 创建基础组件
2. 如需则添加平台变体（`.tv.tsx`）
3. 从 `@/utils/DeviceUtils` 导入并使用响应式工具
4. 在多设备上测试适配效果

### 状态管理开发
1. 在 `/stores` 中选择对应 Zustand Store
2. 遵循现有 Action 与状态结构规范
3. 使用 TypeScript 接口保证类型安全
4. 注意跨 Store 依赖与数据流

### API 对接
1. 在 `/services/api.ts` 添加新接口
2. 实现错误处理与加载状态
3. 高频数据使用缓存策略
4. 在对应 Store 中更新 API 返回数据

## 目录结构说明
- `/app` – Expo Router 页面与路由
- `/components` – 可复用 UI 组件（含 `.tv.tsx` 变体）
- `/stores` – Zustand 状态管理 Store
- `/services` – API、存储、遥控、更新服务
- `/hooks` – 自定义 React Hook，含 `useTVRemoteHandler`
- `/constants` – 常量、主题、更新配置
- `/assets` – 静态资源，含电视专属图标与横幅

---

# 重要指令提醒
只完成要求的内容，不多做也不少做。
除非绝对必要，否则**不要新建文件**。
优先修改现有文件，而不是新建。
不要主动新建文档文件（*.md）或 README 文件，仅在用户明确要求时才创建。
当模式从计划切换为编辑时，需将 plan 和 todo 内容输出为文档。
