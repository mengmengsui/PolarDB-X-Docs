# 如何贡献

我们欢迎贡献者提交代码和思路。长期来看，我们希望PolarDB-X开源项目能够由阿里云内外的开发者共同管理和维护。

### 贡献前

* 阅读并遵循我们的[行为守则](Code-of-Conduct.md)。
* 签署PolarDB-X贡献者许可协议（CLA）：请下载[PolarDB-X CLA](https://gist.github.com/alibaba-oss/151a13b0a72e44ba471119c7eb737d74)。按照说明进行签署。



如下是准备和提交Pull Request（PR）所需的流程清单。

* 通过克隆（Fork）PolarDB-X 相关仓库创建自己的GitHub分支。
* 关于如何通过源码部署PolarDB-X，请阅读[快速入门](../quickstart/topics/Quick-Start.md)。
* 将修改推送到您的个人Fork，并确保它们遵循阿里云的[编码风格](Style.md)。
* 如果Commit消息无法明确表达其含义，请创建带有详细描述的PR。
* 将PR提交审核，并解决所有的反馈。
* 等待合并（由Committer完成）。

下面通过一个示例来演示该流程。



## 提交PolarDB-X代码修改的示例

### Fork自己的分支

PolarDB-X有多个相关仓库。这里以ApsaraDB GalaxySQL和ApsaraDB GalaxyGlue为例。在GitHub的[GalaxySQL](https://github.com/apsaradb/galaxysql)和[GalaxyGlue](https://github.com/apsaradb/galaxyglue)，点击**Fork**按钮，创建自己的galaxysql和galaxyglue仓库。

### 创建本地仓库

```bash
git clone --recursive https://github.com/your_github/galaxysql.git
```

### 创建一个dev分支（命名为your_github_id_feature_name）

```bash
git branch your_github_id_feature_name
```

### 在本地进行修改和提交

```bash
git status
git add files-to-change
git commit -m "messages for your modifications"
```

### 变基（Rebase）并提交到远程仓库

```bash
git checkout develop
git pull
git checkout your_github_id_feature_name
git rebase develop
-- resolve conflict, compile and test --
git push --recurse-submodules=on-demand origin your_github_id_feature_name
```

### 创建PR

点击**New pull request**或者**Compare & pull request**按钮，选择并对比apsaradb/galaxysql和your_github/your_github_id_feature_name分支，并填写PR描述。

### 解决评审意见

解决评委提出的所有问题并更新PR。

### 合并操作（Merge）

此操作由PolarDB-X的Committer完成。
