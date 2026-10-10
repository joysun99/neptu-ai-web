# Global / Site 安装和版本选择

本仓库是公开安装分发入口，下载脚本及查询版本无需 GitHub token。Global/Site 源码仍由各自项目维护，镜像构建和发行由 neptu-ai-images 的工具完成。可用批次以 GitHub Releases 和安装器查询结果为准；文中的占位版本需替换为实际已发布版本。

## 前置准备

宿主机准备 Bash、Docker Engine/CLI、镜像仓库登录、域名/DNS，以及服务所需网络和存储。私有 GHCR 镜像仍需先 docker login ghcr.io。安装管理不要求宿主机安装 Node、pnpm、Git 或 gh；必要逻辑在固定工具镜像中运行。

## 下载和校验

从 GitHub Releases 选择明确版本，下载该发行的安装脚本、release-manifest.json、SHA256SUMS、INSTALL.md、参数文本和 installers.tar.gz。Global-only 或 Site-only 发行只包含对应项目附件。

固定版本地址示例：

```text
https://github.com/joysun99/neptu-ai-web/releases/download/<VERSION>/global-install.sh
https://github.com/joysun99/neptu-ai-web/releases/download/<VERSION>/site-install.sh
https://github.com/joysun99/neptu-ai-web/releases/download/<VERSION>/release-manifest.json
https://github.com/joysun99/neptu-ai-web/releases/download/<VERSION>/SHA256SUMS
https://github.com/joysun99/neptu-ai-web/releases/download/<VERSION>/installers.tar.gz
```

下载方式不限于命令行，可通过浏览器完成。校验 SHA256SUMS 后再解压模板和执行脚本；不要使用 curl | bash。校验文件用于检查下载完整性，仍需从可信 GitHub HTTPS 入口获取。

```bash
sha256sum -c SHA256SUMS
tar -xzf installers.tar.gz
bash global-install.sh --help
bash site-install.sh --help
```

sha256sum/tar 是常见校验解压示例，可替换为等效工具，不是安装脚本运行依赖。所有模板中的 example.com、凭据占位和路径需先修改。

## 查询版本

```bash
bash global-install.sh versions --available
bash global-install.sh versions --latest --json
bash global-install.sh resolve-version --version latest
bash site-install.sh versions --available
bash site-install.sh resolve-version --version latest --json
```

最多列出最近五个不同的已发布安装批次，查询最多浏览最近 200 个 Release。最新批次可能是 prerelease；输出单独标记生产资格。缺少发行、网络失败或镜像组合不匹配都会报错，不会静默使用其他版本。

## 安装和更新

```bash
bash global-install.sh install-modules --from-catalog --version <GLOBAL_VERSION> \
  --root /srv/docker/global-modular --config-file /srv/docker/global-modular/config/global-modules.json
bash site-install.sh install --from-catalog --version <SITE_VERSION> \
  --root /srv/docker/site-modular --config-file /srv/docker/site-modular/config/site-install.json
```

将版本和配置替换为实际值。--from-catalog 将同一批次的 runtime/installer 解析为固定 digest，安装角色默认分别为 control 和 agent；其他服务镜像按 release-manifest.json 填入配置。安装配置默认不启动服务，随后按 help 配置 Leader/Follower 或 Standalone/Join，再明确启动选定模块。

离线安装或恢复使用发行参数文本里的固定版本和镜像 digest，必要时追加 --offline，不使用 --from-catalog。不要 source 参数文本文件。

更新前下载目标版本脚本并保留当前配置和回滚镜像。Site 使用 prepare-business-upgrade、preview-business-upgrade、execute-business-upgrade；Global 使用对应模块的停止、升级及显式启动流程。版本查询不会改写业务配置、绕过原始哈希/恢复检查或自动重启整套服务。

历史 Release 不证明 GHCR 镜像仍存在；安装时 Docker pull 失败必须停止。发布回拉验证和版本配对也不等于生产准入或跨项目兼容验收。


## 统一工作流与模块重启

以下命令用于 family 清单含 `deploymentWorkflow: neptu-deployment-workflow-v1` 的新发行。历史脚本和镜像不会因文档更新自动获得这些功能；先核对下载批次。

```bash
bash global-install.sh
bash site-install.sh
bash global-install.sh update --from-catalog --version <GLOBAL_VERSION> \
  --root /srv/neptu-global --config-file /srv/neptu-global/update.json \
  --modules installed --start --plan --allow-prerelease
bash site-install.sh restart --from-catalog --version <CURRENT_SITE_VERSION> \
  --root /srv/neptu-site --config-file /srv/neptu-site/restart.json \
  --modules site-ai --plan --allow-prerelease
bash site-install.sh rollout --from-catalog --version <CURRENT_SITE_VERSION> \
  --root /srv/neptu-site --config-file /srv/neptu-site/rollout.json \
  --modules installed --plan --allow-prerelease
```

无参数打开终端向导；命令行和向导使用同一受保护配置。先阅读发行包内 `<family>-templates/deployment-workflow.md`，复制并修改配套模板。配置、manifest 和秘密文件放在安装根目录内：根目录 0700、文件 0600，归操作者所有，不使用链接。

`--plan` 列出模块、实际 stack 影响范围、步骤和固定镜像，不执行停服或替换。核对后去掉该参数；终端只确认一次 `EXECUTE`，自动化显式增加 `--yes`。安装默认完成启动前准备并保持停止；`--start` 才执行私有基础设施 bootstrap、资源初始化及业务启动。`--allow-prerelease` 只表示接受未获生产资格的批次，不改变该资格。

restart 只重启选定服务并等待健康，使用当前 Compose 镜像；不会重新载入环境变量或挂载定义。配置需重建容器或镜像更新时使用 update。rollout 按受审阅依赖顺序处理整组服务；单机单实例会中断连接，不承诺零停机，设备 WS/语音需要重连。更新已通过健康验证后，无需默认再次 rollout 全部服务。

Site 的普通业务重启通过 `restart-business` 重新验证当前数据库/存储凭据与运行模式，再限制选定服务重启；不会通过通用重启旁路资源资格检查。Agent 重启须与 Device 成组。

首期通过受审阅配方组合既有安装/升级动作。Global 普通设备网络模块具备停止后镜像替换入口，Health 的签名/BOM 后继仍须单独资格流程，不能直接替换旧绑定。Site Device-only 支持 Joined/Standalone 的单个或多个业务镜像后继；私人就绪回执不等于公开准入。APP/payment 不会随默认 Device 配方加入。

`workflow-status` 列出本地日志；`resume --operation <ID>` 使用相同配置继续已确认步骤。结果不确定的初始化/Join 不会自动重复。`rollback` 仅执行预先审阅的反向配方，数据或密钥恢复继续使用原流程。日志、备份和凭据不得上传到公开仓库。
