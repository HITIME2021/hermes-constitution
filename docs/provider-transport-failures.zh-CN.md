# Provider Transport Failures 中文说明

Hermes 必须区分“Provider 通道失败”和“任务语义失败”。

这是一个硬规则。否则 CLI 连接 AI 超时、网络错误或限流，会被错误地解释成架构失败、流程失败或任务计划失败，进而触发不必要的 replanning。

## 两类失败的区别

### Provider Transport Failure

Provider 通道失败表示 Hermes 没有稳定地拿到 Provider 的结果。

例子：

```text
CLI timeout
network_error
rate_limit
auth_error
provider_unavailable
cli_crash
malformed_response
```

这些问题说明“电话没打通”，不说明任务计划错了。

### Task Reasoning / Execution Failure

任务语义失败表示 Provider 已经执行了任务，但结果本身有问题。

例子：

```text
implementation_failed
test_failure
review_failed
scope_conflict
missing_context
invalid_assumption
```

这些问题才可能触发 revision、review failure handling 或 replanning。

## 状态流转规则

```text
executing -> executing:
  transport_error 且 retry budget 未耗尽。

executing -> blocked:
  auth_error、quota 用尽、provider 多次不可用。

executing -> replanning:
  semantic execution failure、review failure、invalid assumptions、repeated implementation failure。

executing -> reviewing:
  execution completed。
```

## ExecutionAttempt

Transport retry 应记录在 `ExecutionAttempt` 中，而不是算作 Task retry。

```yaml
execution_attempt:
  id: attempt-001
  execution_id: exec-001
  provider: codex
  status: transport_failed
  error:
    category: transport_error
    type: timeout
    retryable: true
    counts_as_task_failure: false
    should_trigger_replanning: false
```

## 硬规则

```text
1. Transport failure 默认不能触发 task replanning。
2. Timeout 不是架构、workflow 或 task plan 错误的证据。
3. Provider Adapter 必须先分类错误，再返回给 Workflow Engine。
4. Auth error 和 quota/rate-limit 应进入 blocked 或 delayed，不是 failed。
5. Transport retry 必须有上限，并记录为 ExecutionAttempt。
6. 只有语义失败、review 失败、反复实现失败或新发现的约束，才可能触发 replanning。
```

一句话：

```text
CLI 超时是 Provider 通道故障，不是任务失败，更不是架构失败。
```

