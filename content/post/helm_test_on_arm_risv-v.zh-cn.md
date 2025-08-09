---
title: "Helm 和 Helm chart Test 在 ARM 和 RISV-V 上的自动化测试"
date: 2024-06-29T10:44:41+08:00
draft: false
tags: [linux,kubernetes,helm,risv-v]
categories: [tech]
---
本文是我参加[《第一期傲来操作系统（EulixOS）训练营》](https://opencamp.cn/EulixOS/camp/202401)的项目实习报告。

[傲来操作系统（EulixOS）](https://eulixos.com/)是由中科院软件所 / 中科南京软件技术研究院团队基于 openEuler 打造的操作系统发行版。

## 任务目标 

[Helm](https://github.com/helm/helm) 是一个 `Kubernetes` 的包管理工具，它可以帮助用户定义、安装和升级运行在 `Kubernetes` 上的应用程序。

[Helm chart test](https://github.com/helm/chart-testing) 是一个用于测试 `Helm` 图表的 CLI 工具，用于测试 `Helm chart` 的拉取请求，能自动检测与目标分支相比已经更改的 chart。

本任务计划在 ARM 和 RISC-V 架构上运行 `Helm` 和 `Helm chart Test` 的测试，以此来对比这两种平台上云原生软件的成熟度。

## Helm 的单元测试

分析 Helm 的 [Makefile](https://github.com/helm/helm/blob/main/Makefile) 文件，发现 `test-unit` 目标是用来运行单元测试的：

```makefile
.PHONY: test-unit
test-unit:
	@echo
	@echo "==> Running unit tests <=="
	GO111MODULE=on go test $(GOFLAGS) -run $(TESTS) $(PKG) $(TESTFLAGS)
```
可以看出，helm 的单元测试可以直接通过 `go test` 命令来执行。

查看 [go.mod](https://github.com/helm/helm/blob/main/go.mod) 文件，确定该项目使用的 Go 版本是 1.22.0：

```
module helm.sh/helm/v3

go 1.22.0

require (
	github.com/BurntSushi/toml v1.3.2
    ...
}
```



## Helm chart test 的单元测试

`Helm chart test` 项目使用 [build.sh](https://github.com/helm/chart-testing/blob/main/build.sh) 脚本进行构建发布。分析 `build.sh` 发现，在每次构建前，会使用 `go test -race ./...` 运行单元测试：

```bash
...
go test -race ./...
goreleaser "${goreleaser_args[@]}"
...
```

查看[go.mod](https://github.com/helm/chart-testing/blob/main/go.mod)文件，确定该项目使用的 Go 版本是 1.22.0：

```
module github.com/helm/chart-testing/v3

go 1.22.0

toolchain go1.22.4
...
```

## 自动化执行测试

helm 和 helm chart test 的单元测试都可以直接通过 `go test` 命令来执行。我们可以使用 bash 编写脚本使测试过程自动化。

这个脚本的主要流程为：

1. 自动识别和配置：脚本首先检测硬件平台，然后自动下载并在测试目录下安装 Go，不干扰系统中的其他设置或版本。
2. 环境设置：配置必要的环境变量，确保测试在适当的环境下执行。
3. 代码仓库管理：自动从配置的 Git 仓库地址克隆代码到本地指定目录。
4. 测试执行：运行单元测试，并将结果输出到报告文件中。
5. 性能数据收集：通过调用 `performance_counter_920.sh` 收集和记录测试期间的性能指标。

为了避免网络环境对测试影响，脚本可以自定义配置：

* 通过 `GO_BASE_URL` 定义 go 安装包下载网址 
* 通过 `REPO_URL` 定义项目源码仓库的地址。可以提前将项目从 GitHub 同步到 Gitee。
* 通过 `GOPROXY` 定义 Go 镜像地址。缺省设置为 `GOPROXY=https://goproxy.cn`，从国内镜像下载 moudle。

另外，helm 在测试插件功能时，会访问 [https://github.com/adamreese/helm-env](https://github.com/adamreese/helm-env) 。由于网络环境问题，涉及的测试经常会失败。所以我将 helm-env 项目同步到了 gitee，并修改了测试案例 [vcs_installer_test.go](https://gitee.com/mahaoliang/helm/blob/02417b31ab70be386bb0a18045cbbca8dd9b8a8b/pkg/plugin/installer/vcs_installer_test.go)，让它从 gitee 下载插件，保证了测试运行的稳定。

两个项目的自动化测试脚本都已经提交到了 gitee，分别为：

* https://gitee.com/mahaoliang/helm-test
* https://gitee.com/mahaoliang/helm-chart-test

## 测试结果

下面是在 ARM 上运行的 helm 的测试结果。

```bash
Avg 10 times duration time: 24558698523
Avg 10 times task clock: 130738.640
Avg 10 times cpu-cycles:   290310169669
Avg 10 times instructions: 269526073395
Avg 10 times cache references: 95029267023
Avg 10 times cache misses: 764800123
Avg 10 times branches: <not
Avg 10 times branch misses: 647318994
Avg 10 times L1 dcache loads: 95029267023
Avg 10 times L1 dcache load misses: 764800123
Avg 10 times LLC load misses: 480961351
Avg 10 times LLC load: 1156309818
Avg 10 times IPC: 0.928
helmrequiresearchfil.txt has been deleted
Number of packages: 50
Analyzing test results in /home/cloud2/helm-test/reports/20240628135442/test_result
Total tests: 1327
Passed tests: 1327
Failed tests: 0
```
一共运行了 1327 个测试用例，全部通过。

在 ARM 上运行的 helm chart test 的测试结果：

```bash
Avg 10 times duration time: 704309016
Avg 10 times task clock: 5337.880
Avg 10 times cpu-cycles:   10071410187
Avg 10 times instructions: 8342502186
Avg 10 times cache references: 3300195194
Avg 10 times cache misses: 34476953
Avg 10 times branches: <not
Avg 10 times branch misses: 39065962
Avg 10 times L1 dcache loads: 3300195194
Avg 10 times L1 dcache load misses: 34476953
Avg 10 times LLC load misses: 15971288
Avg 10 times LLC load: 50427439
Avg 10 times IPC: 0.828
Number of packages: 8
Analyzing test results in /home/cloud2/helm-chart-test/reports/20240628140012/test_result
Total tests: 92
Passed tests: 92
Failed tests: 0
```

一共运行了 92 个测试用例，全部通过。

测试过程详细输出的原始文件如下：

* [helm 的 go test 输出](https://cdn.mahaoliang.tech/2024/202507261057451)
* [helm 测试过程的 perf 统计指标](https://cdn.mahaoliang.tech/2024/202507261058094)
* [helm chart test 的 go test 输出](https://cdn.mahaoliang.tech/2024/202507261058288)
* [helm chart test 测试过程的 perf 统计指标](https://cdn.mahaoliang.tech/2024/202507261059355)

到测试截止时间，RISC-V 机器还未准备好，因此没有运行 RISC-V 架构的测试。不过我们已经提供了自动化测试脚本，可以直接在 RISC-V 架构的机器上运行，为后面在 RISC-V 上的测试做好了准备。