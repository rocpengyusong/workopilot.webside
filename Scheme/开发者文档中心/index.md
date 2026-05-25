---
# 使用 VitePress 默认的主页布局
layout: home

# Hero 区域配置
hero:
  name: "喔壳智能体赋能平台"
  text: "企业级多模态AI应用构建平台"
  tagline: |
    助力软件开发商、硬件制造商和应用集成商
    快速搭建起基于业务场景的<span class="highlight">复杂</span>智能体应用
  actions:
    - theme: brand
      text: 了解平台架构
      link: /guide/intro
    - theme: alt
      text: 查看核心功能
      link: /guide/features
  image:
    src: /workopilot_logo_blue.png
    alt: Workopilot Logo

# 特性网格区域 (提取了我们之前设计的四大卖点)
features:
  - title: 🧩 伴随式插件审核
    details: 告别传统后台审核模式。审核员在 OA/ERP 页面作业时，AI 助手以悬浮窗形式实时提供合规建议与风险预警。
    icon: 🖥️

  - title: 🧠 混合制度引擎
    details: 结合 RAG 语义检索与结构化参数计算。既能回答“能不能报销”，又能精确计算“是否超标”。
    icon: ⚖️

  - title: ⚡ 动态白名单技术
    details: 独创 Vision 预分类技术，动态加载附件白名单，大幅降低 Token 消耗，提升审核精准度与速度。
    icon: 🚀

  - title: 🛠️ 策略低代码编排
    details: 类 IDE 的规则配置中心。支持 Prompt 编排、MCP 工具挂载及全链路仿真沙箱测试。
    icon: ⚙️
---
