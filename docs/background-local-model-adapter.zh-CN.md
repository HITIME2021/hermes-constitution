# 本地后台模型适配器

本地后台模型适配器定义 Hermes 如何使用 Ollama 本地模型，例如
`qwen2.5:14b`，完成有边界、低风险的文字处理。

这个适配器的权限低于正式 Provider Adapter。本地后台模型可以转换文本，但不得产生带权威效力的执行结果。

## 核心规则

<!-- snapshot:block id="background-local-model-adapter.zh-CN" section="Tools Layer" priority="64" -->
Ollama background model 是文字处理工具，不是 planning、review、execution、
approval 或 memory authority。

允许用途：

- prompt 压缩
- evidence 摘要
- Kanban comment 草稿
- final report 初稿
- 日志归类
- task 文本结构化
- 中英互译
- 重复信息去重
- token 降噪
- artifact 文本预处理
- stop-condition checklist 的候选提取

禁止用途：

- 最终 planning source of record
- 替代 Codex review
- 替代 CodeBuddy execution
- 修改 code、config、memory、database、auth 或 credential
- 执行 shell
- 决定 provider routing
- 做 approval decision
- 做最终 stop-condition decision
- 写入长期 memory
- git commit、push 或创建 PR
- 作为 constitution authority
- 在上下文过大时决定哪些 evidence 可以丢弃

Hermes 必须先验证和路由 Ollama 输出，才能让它影响任务。Ollama 输出只能是草稿文本或输入证据。

## Context Budget Gate

本地模型上下文有限。对于 `qwen2.5:14b`，当前 Ollama 模型元数据为：

```yaml
model: qwen2.5:14b
parameter_size: 14.8B
quantization_level: Q4_K_M
context_length: 32768
```

Hermes 应使用更保守的工作预算：

```yaml
ollama_background:
  model: qwen2.5:14b
  model_context_length: 32768
  operational_context_budget_tokens: 12000
  hard_max_input_tokens: 16000
```

在派发文本给 Ollama 前，Hermes 必须估算文本包大小。如果文本包可能超过本地模型配置的 context budget，Hermes 不得：

- 盲目切块
- 无监督递归摘要
- 静默丢弃内容
- 让 Ollama 决定哪些 evidence 可以丢弃

Hermes 必须把任务交还给 Hermes Primary，由 Hermes Primary 决定裁剪、分段、升级到更强 Provider，或请求 operator 判断。

Context budget estimation 必须由本地确定性计算完成。Hermes 不得为了判断文本是否适合本地模型上下文窗口而调用 LLM 消耗 token。
如果没有精确 tokenizer，Hermes 应使用保守的本地估算器，并在 evidence 中记录 estimate method。

路由规则：

```text
estimated_input_tokens <= 12000:
  Ollama 可以处理该有边界文本包。

12000 < estimated_input_tokens <= 16000:
  仅当低风险文字处理且 Hermes Primary 批准时，才可有限使用。

estimated_input_tokens > 16000:
  不调用 Ollama，交还 Hermes Primary。
```

Ollama 永远不得作为决定哪些上下文可以从 evidence、planning packet 或 review packet 中省略的权威。
<!-- /snapshot:block -->

## 适配器边界

适配器可以调用本地 Ollama API，例如：

```text
http://127.0.0.1:11434/api/generate
```

该调用仍属于 Backend Tool invocation，因为它消耗本地运行时资源并依赖本地服务状态，但允许的效果仅限文本转换。除非确有必要，不得把 credential 或完整私有材料传给 Ollama；能用脱敏摘要时应使用脱敏摘要。

## 本地代理绕过

很多 operator 的工作机都会开启 HTTP 代理访问外部网络。本地 Ollama 流量不得被送到这些代理。

常见症状：

```text
curl http://127.0.0.1:11434/api/tags
HTTP/1.1 503 Service Unavailable
Proxy-Connection: close
```

如果加上 `--noproxy` 后同一命令成功，说明 Ollama 服务本身正常，失败原因是代理路由：

```bash
curl --noproxy 127.0.0.1,localhost -i http://127.0.0.1:11434/api/tags
```

推荐 shell 配置：

```bash
export NO_PROXY=127.0.0.1,localhost,::1,$NO_PROXY
export no_proxy=127.0.0.1,localhost,::1,$no_proxy
```

这些行应放在 `HTTP_PROXY`、`HTTPS_PROXY`、`http_proxy` 或 `https_proxy` 设置之后，例如 `~/.bashrc` 末尾。

验证：

```bash
curl -i http://127.0.0.1:11434/api/tags
```

预期结果：

```text
HTTP/1.1 200 OK
```

在检查 proxy bypass、service status 和 `/api/tags` 之前，不应把本地 Ollama `503` 直接当成模型失败。

## 初始角色建议

Hermes 初始使用方式应限制为：

```text
User / logs / evidence / artifacts
  -> Ollama text preprocessing
  -> Hermes Primary validates and routes
  -> Codex planning or review when needed
  -> CodeBuddy execution when needed
```

不要用 Ollama 替代 Codex 或 CodeBuddy。它只用于在带权威效力的 workflow 阶段之前降低格式整理、摘要和清理成本。

## 证据要求

每个 Ollama 辅助阶段应记录：

- model name
- observed context length，如可用
- estimated input size
- context budget gate 是否通过
- prompt purpose
- output role，例如 `draft_summary` 或 `candidate_checklist`
- Hermes Primary 是否接受、编辑、拒绝或升级该输出

如果无法获取 context size，应记录 `estimated_input_tokens: unavailable`，并对任何超过短文本包的任务优先交还 Hermes Primary。
