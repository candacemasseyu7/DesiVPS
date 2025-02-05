# OpenAI API 与 Azure OpenAI API 对比分析

> 注意：由于定价和限制会随时间变化，本文仅供撰写当前时间参考。

## 1. 关键术语解析

- **RPM (requests per minute)**：每分钟请求次数
- **RPD (requests per day)**：每天请求次数
- **TPM (tokens per minute)**：每分钟 Token 数
- **TPD (tokens per day)**：每天 Token 数

在 OpenAI 的 [Tokenizer](https://platform.openai.com/tokenizer) 工具中，可以根据文本查询对应的 Token 数。此外，通过 [Tiktoken 项目](https://github.com/openai/tiktoken/blob/main/tiktoken/model.py)，可以发现 `text-embedding-ada-002` 与 `gpt-3.5`、`gpt-4` 的词表均为 `cl100k_base`。如果仅需进行向量化操作，性价比高的 `text-embedding-ada-002` 是一个不错的选择。

👉 [WildCard | 一分钟注册，轻松订阅海外线上服务](https://bbtdd.com/WildCard)

## 2. OpenAI API 详解

OpenAI 会根据使用情况动态调整配额，当付费超过 5 美元后，配额会有显著提升。

**支付范围：`> $5 paid, < $50 paid`**

| Models                  | Input  | Output | RPM  | RPD   | TPM  |
|-------------------------|--------|--------|------|-------|------|
| gpt-3.5-turbo-1106      | $0.0010 | $0.0020 | 3500 | 10000 | 60K  |
| gpt-3.5-turbo-instruct  | $0.0015 | $0.0020 | 3500 | 10000 | 60K  |
| gpt-4                    | $0.03   | $0.06   | 500  | 10000 | 10K  |
| gpt-4-32k                | $0.06   | $0.12   | 500  | 10000 | 10K  |

[模型定价参考](https://openai.com/pricing) | [限流指南](https://platform.openai.com/docs/guides/rate-limits/usage-tiers)

## 3. Azure OpenAI API 详解

不同区域的价格和限制有所不同，以下以 East US 2 为例：

| Models                | Context | Prompt (Per 1k tokens) | Completion (Per 1k tokens) | TPM   |
|-----------------------|---------|------------------------|----------------------------|-------|
| GPT-3.5-Turbo         | 4K      | $0.0015                | $0.002                     | 300K  |
| GPT-3.5-Turbo         | 16K     | $0.003                 | $0.004                     | 300K  |
| GPT-4                  | 8K      | $0.03                  | $0.06                      | 40K   |
| GPT-4                  | 32K     | $0.06                  | $0.12                      | 80K   |

[限流参考](https://learn.microsoft.com/en-us/azure/ai-services/openai/quotas-limits)

## 4. Azure OpenAI API 常见问题与解决方案

### 4.1 404 Resource Not Found

若请求的 Api Version 设置错误，会返回 404 Resource Not Found。参考：

[API 版本列表](https://learn.microsoft.com/en-us/azure/ai-services/openai/reference)

当前可用版本如下：

- 2022-12-01
- 2023-03-15-preview
- 2023-05-15
- 2023-06-01-preview
- 2023-07-01-preview
- 2023-08-01-preview
- 2023-09-01-preview

### 4.2 Unsupported Data Type

Azure OpenAI API 的标准格式如下：

text
https://{your-resource-name}.openai.azure.com/openai/deployments/{deployment-id}/completions?api-version={api-version}


若使用以下格式（URL 中多了一个 `chat` 字段），则会导致 `Unsupported data type` 错误：

text
https://{your-resource-name}.openai.azure.com/openai/deployments/{deployment-id}/chat/completions?api-version={api-version}


## 5. 限流优化策略

### 5.1 应用侧优化

- 在应用程序中增加重试逻辑
- 分散请求时间，避免集中请求

### 5.2 部署侧优化

- Azure 上部署多个相同模型
- OpenAI 多注册几个账号
- 使用代理池，将请求分散到多个 API Key 上

## 6. 总结与建议

### 账号获取

- **OpenAI**：国内有大量卖账号、代注册服务，提供免费额度，超出部分需使用国外信用卡支付。
- **Azure OpenAI API**：按量计费，使用国外信用卡后付费，开通 GPT-4 需单独申请。

### 价格对比

OpenAI API 的价格相对更便宜。批量购买的 OpenAI 账号单价低于 2 元，且提供 5 美元三个月的使用额度。相比之下，Azure 按量计费的模式成本较高。

### 请求限制

免费版的 OpenAI API 限制较多，需构建 API 代理池以发挥其优势。Azure OpenAI API 的配额较为宽松，且可通过部署多个模型提升配额。

### 网络访问

**OpenAI** 需要国际网络出口，需依赖国外代理，可能涉及法律风险。**Azure OpenAI** 在国内可直接访问，无需代理。

### 使用场景建议

- **生产环境**：推荐使用 Azure OpenAI API。
- **开发测试**：OpenAI API 更为适合。
- **对成本敏感的场景**：可考虑构建 OpenAI API 代理池，适用于实时性要求不高、允许反复尝试的场景。