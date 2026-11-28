# Spotify Backstage Scaffolder 工作区与容器镜像边界 (`NEW-BS-05`)

[English](../52-SPOTIFY-BACKSTAGE-SCAFFOLDER-WORKSPACE-AND-CONTAINER-BOUNDARIES.md) | [Türkçe](../tr/52-SPOTIFY-BACKSTAGE-SCAFFOLDER-WORKSPACE-AND-CONTAINER-BOUNDARIES.md) | [日本語](../ja/52-SPOTIFY-BACKSTAGE-SCAFFOLDER-WORKSPACE-AND-CONTAINER-BOUNDARIES.md) | **简体中文** | [한국어](../ko/52-SPOTIFY-BACKSTAGE-SCAFFOLDER-WORKSPACE-AND-CONTAINER-BOUNDARIES.md)

**VulKill 发现：**在 Spotify/Backstage 审计中发现，并于 2026-11-01 使用真实上游 Scaffolder 模块在隔离的本地运行时测试中观察到。本记录使用审计编号 `NEW-BS-05`，不指定上游安全公告编号。

## 1. 代码缺陷如何形成安全边界问题

`fetch:rails` 对输出路径使用字面 `startsWith` 比较，因此具有相同前缀的相邻路径或工作区内符号链接可解析到任务工作区之外。Cookiecutter action 还会把调用者选择的 `imageName` 传给容器运行器，而 action 层没有镜像允许列表。VulKill 使用临时工作区和禁用网络的本地测试容器运行了真实 action handler。

该范围要求具备执行受影响 Scaffolder action 的权限并能控制相关输入。测试未访问 Spotify 在线服务或外部容器 registry。

## 2. 现实场景

- **场景 A — 写入相邻工作区：**生成项目向名称与当前工作区共享前缀的相邻任务目录写入文件。
- **场景 B — 符号链接逃逸：**字面位于工作区内的路径经符号链接写到任务目录外。
- **场景 C — 未批准生成器镜像：**模板选择会在生成项目中产生意外 workflow 或 dependency 文件的镜像。

## 3. 运行时观察

```text
control: 普通子路径保留在临时工作区内
RED:     相邻前缀和符号链接路径在工作区外写入合成标记
RED:     调用者选择的测试镜像到达本地容器运行器
GREEN:   safe-child 解析和镜像允许列表拒绝同样输入
BENIGN:  已允许镜像和普通子路径继续工作
```

- Backstage source commit: `ac2de0fcbea3edcb38da36116268aae3cce189c1`
- Harness SHA-256: `fa4dab592b61c8f8fcde81743694189b5d988cd6a9c7cbf29accc50c4bc097ae`
- Runtime chain SHA-256: `0e2de393ee65136719495c2824cfe5133512ce570636174ee942fc208bbdc47a`

## 4. 防护

使用平台 safe-child helper 和 realpath 检查限制每个目标，拒绝符号链接逃逸，在 action 边界实施容器镜像允许列表，并限制敏感模板的创建和执行权限。
