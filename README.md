# MengZe Flags Generator

Minecraft 服务器 JVM 启动参数生成工具

## 快速使用

1. 访问在线版本：https://flags.mengze.vip/
2. 设置内存大小（支持GB/MB切换）
3. 选择垃圾回收器类型
4. 点击【复制参数】使用生成的配置

## 本地运行

```bash
git clone https://github.com/YShenZe/MengZe-Flags.git
cd MengZe-Flags
npm install
npm run dev
```

## 主要功能

- 智能内存分配建议
- 支持 G1GC/ZGC/Shenandoah 三种GC算法
- 集成 Aikar's 优化参数
- 一键复制生成配置