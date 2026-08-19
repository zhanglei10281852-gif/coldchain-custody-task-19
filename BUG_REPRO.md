# Bug Reproduction

## 包的性质

当前 test_model_fix 保存的是被测模型修复后的结果源码，不是初始含 Bug 源码。要复现原始缺陷，必须检出下面固定的 parent SHA；不要在当前修复结果源码上期待重新出现修复前失败。生成系统使用的可信验证补丁和完整验证日志仅在本地留存，不提交到结果分支。

## 问题现象

系统已有启用研究、就绪样本和可用保温箱，但运营汇总接口稳定返回全零；明细查询仍能看到这些记录。请修复汇总结果在仓储层到服务层之间的传递。 请只修改必要的生产代码，不得新增、删除或修改测试文件，不得跳过测试或放宽断言。

## 含 Bug 版本

- 仓库：zhanglei10281852-gif/coldchain-custody-task-19
- 仓库地址：https://github.com/zhanglei10281852-gif/coldchain-custody-task-19.git
- parent SHA：400b90c8d99c15fc3df684dc466e7f71a0f89b7e

## 复现步骤

```bash
git clone -- https://github.com/zhanglei10281852-gif/coldchain-custody-task-19.git bug-repro
cd bug-repro
git checkout --detach 400b90c8d99c15fc3df684dc466e7f71a0f89b7e
go test ./internal/service -run "^TestOperationalSummaryRequiresReadPermissionAndCountsRows$" -count=1
```

## 双架构完整错误信息

### linux/amd64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/service -run "^TestOperationalSummaryRequiresReadPermissionAndCountsRows$" -count=1
--- FAIL: TestOperationalSummaryRequiresReadPermissionAndCountsRows (0.50s)
    service_test.go:366: summary = {StudiesActive:0 SamplesReady:0 SamplesInTransit:0 SamplesQuarantined:0 ContainersAvailable:0 ShipmentsActive:0 OpenExcursions:0 PendingHandoffs:0 FailedJobs:0}
FAIL
FAIL	github.com/zhanglei10281852-gif/coldchain-custody-base/internal/service	0.501s
FAIL

```

stderr：

```text
warning: internal/service/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644
warning: internal/service/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644

```

### linux/arm64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/service -run "^TestOperationalSummaryRequiresReadPermissionAndCountsRows$" -count=1
--- FAIL: TestOperationalSummaryRequiresReadPermissionAndCountsRows (1.24s)
    service_test.go:366: summary = {StudiesActive:0 SamplesReady:0 SamplesInTransit:0 SamplesQuarantined:0 ContainersAvailable:0 ShipmentsActive:0 OpenExcursions:0 PendingHandoffs:0 FailedJobs:0}
FAIL
FAIL	github.com/zhanglei10281852-gif/coldchain-custody-base/internal/service	1.468s
FAIL

```

stderr：

```text
warning: internal/service/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644
warning: internal/service/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644

```

## 通过条件

定向公开行为验证通过，相关包和全量测试通过，go vet 及 linux/amd64 构建通过。 定向命令 go test ./internal/service -run ^TestOperationalSummaryRequiresReadPermissionAndCountsRows$ -count=1 必须由修复前失败变为修复后通过；相关包与 go test ./... -count=1 全量回归通过，回退 gold 关键修改后定向命令重新失败。
