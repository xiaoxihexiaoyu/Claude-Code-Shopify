# Claude Code + Shopify 开发指南

用 Claude Code 作为 AI 结对编程工具，高效开发 Shopify 项目。

## 快速开始

\`\`\`bash
# 安装 Claude Code
npm install -g @anthropic-ai/claude-code

# 启动
claude
\`\`\`

## 核心内容

| 章节 | 说明 |
|-------|-------|
| [Claude Code 速览](#一claude-code-速览) | 安装、特性、与 ChatGPT 区别 |
| [Shopify 开发基础](#二shopify-开发基础完整版) | 三大 API、数据模型、GraphQL、Metafields、Policies、Webhooks |
| [Hydrogen 完全指南](#三hydrogen-完全指南) | 项目结构、核心组件、Hooks、Oxygen 部署 |
| [工作流](#四claude-code--shopify-工作流) | 典型会话、Agent 系统、工具能力 |
| [第三方服务接入](#五第三方服务接入) | Klaviyo、GA4、Chatwoot 集成详解 |
| [实战场景](#六实战场景案例) | 组件开发、批量操作、API 集成、代码审查 |
| [最佳实践](#七最佳实践与技巧) | 提示词技巧、CLAUDE.md 配置、工作流建议 |

## 常用命令

\`\`\`bash
# Shopify Admin API 管理
npm run admin -- blog:list
npm run admin -- article:create --blog-id <BLOG_GID> --file <JSON>
npm run admin -- policy:update --type SHIPPING_POLICY --body @file.html

# 多语言翻译
npm run translate:list
npm run translate:import

# SEO 批量更新
npx tsx scripts/seo-admin.ts shop:seo:i18n
npx tsx scripts/seo-admin.ts article:seo
\`\`\`

## 学习资源

- [Claude Code 文档](https://docs.anthropic.com/claude-code)
- [Shopify 开发文档](https://shopify.dev/docs)
- [Hydrogen 指南](https://shopify.dev/docs/custom-storefronts)
- [Admin GraphQL API](https://shopify.dev/docs/api/admin-graphql)

---

*文档版本：2026-02-15*
