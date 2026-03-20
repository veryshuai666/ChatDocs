# 欢迎使用 ChatDocs 📖

<div align="center">
  <span style="display: inline-block; background: #2196f3; color: white; padding: 4px 12px; border-radius: 16px; font-size: 14px;">C++ 后端开发</span>
  <span style="display: inline-block; margin-left: 8px; background: #4caf50; color: white; padding: 4px 12px; border-radius: 16px; font-size: 14px;">技术实践笔记</span>
</div>

---

## 📚 文档概览
本站点聚焦 C++ 后端核心技术落地实践，涵盖认证授权、网络通信、消息系统等核心场景，所有文档均为实战导向，可直接参考落地。

### 核心文档模块
| 模块方向 | 核心内容 | 文档链接 |
|----------|----------|----------|
| 认证授权 | Refresh Token 模式设计与 JWT 实现 | [RefreshToken 模式-规划](./开发文档/基于jwt -cpp 实现 Refresh token模式.md)<br>[RefreshToken 模式-实现](./开发文档/Refresh token模式的实现.md) |
| 网络通信 | Websocket 连接生命周期管理 | [Websocket 断连机制](./开发文档/Websocket 连接过期和回收机制.md) |
| 消息系统 | 消息转发、投递策略设计 | [消息处理机制](./开发文档/消息转发分类和投递策略.md) |

## ⚡ 快速开始
1. 左侧侧边栏可浏览完整文档目录
2. 点击表格内链接直达对应技术模块
3. 文档内代码块可直接复制，适配 Windows/Linux 双平台

## 💡 使用提示
- 所有代码均基于 C++ 标准库实现，无第三方强依赖（JWT 依赖 `jwt-cpp`，Websocket 依赖对应服务端库）
- 文档内的设计思路可根据业务场景灵活调整，核心逻辑保持通用