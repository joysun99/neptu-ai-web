# Global / Site 安装和版本选择

本仓库是公开安装分发入口，下载脚本及查询版本无需 GitHub token。Global/Site 源码仍由各自项目维护，镜像构建和发行由 neptu-ai-images 的工具完成。当前还未发布首个安装发行，以下版本号是占位示例。

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
  --root /srv/docker/global-modular --config-file /srv/config/global-modules.json
bash site-install.sh install --from-catalog --version <SITE_VERSION> \
  --root /srv/docker/site-modular --config-file /srv/config/site-install.json
```

将版本和配置替换为实际值。--from-catalog 将同一批次的 runtime/installer 解析为固定 digest，安装角色默认分别为 control 和 agent；其他服务镜像按 release-manifest.json 填入配置。安装配置默认不启动服务，随后按 help 配置 Leader/Follower 或 Standalone/Join，再明确启动选定模块。

离线安装或恢复使用发行参数文本里的固定版本和镜像 digest，必要时追加 --offline，不使用 --from-catalog。不要 source 参数文本文件。

更新前下载目标版本脚本并保留当前配置和回滚镜像。Site 使用 prepare-business-upgrade、preview-business-upgrade、execute-business-upgrade；Global 使用对应模块的停止、升级及显式启动流程。版本查询不会改写业务配置、绕过原始哈希/恢复检查或自动重启整套服务。

历史 Release 不证明 GHCR 镜像仍存在；安装时 Docker pull 失败必须停止。发布回拉验证和版本配对也不等于生产准入或跨项目兼容验收。
