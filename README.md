# Zu Verschenken 地图平台

这是一个基于Vue和OpenLayers的地图应用，用于分享和查找"zu verschenken"（免费赠送）物品的位置信息。

## 项目背景

在德国，"zu verschenken"是一种常见的做法，人们会将不再需要的物品放在街道上，并标上"zu Verschenken"（免费赠送）的牌子。虽然有些人会在Facebook或eBay Kleinanzeige等社交媒体上发布信息，但传统的"zu Verschenken"很少在社交媒体上公布。

通过这个网站，人们可以：
- 发布免费物品的位置信息，无需与人见面
- 查找附近的免费物品
- 如果看到街边的"zu Verschenken"箱子，即使不是物主，也可以在网站上发布信息

## 功能

- 显示OpenStreetMap地图
- （即将推出）添加免费物品的位置标记
- （即将推出）查看物品详情
- （即将推出）发布新的免费物品信息

## 如何使用

### 安装依赖
```bash
npm install
```

### 开发环境启动
```bash
npm run dev
```

### 生产环境构建
```bash
npm run build
```

## 技术栈

- Vue.js - 前端框架
- OpenLayers - 地图库
- OpenStreetMap - 底图数据源

## Recommended IDE Setup

[VSCode](https://code.visualstudio.com/) + [Volar](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (and disable Vetur).

## Customize configuration

See [Vite Configuration Reference](https://vite.dev/config/).
