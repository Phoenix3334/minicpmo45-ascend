# port310p/release — 310P 移植包 · GitHub 传输分支

本分支承载 `PORT310P` 移植运行包的可经 git 传输部分（走 SSH-443 纯 git 通道，
单文件 ≤100MiB 分块；**非 LFS**——本机到 GitHub LFS/CDN 实测仅 ~4KB/s，不可用）。

## 内容

| 路径 | 说明 |
|---|---|
| `parts/PORT310P-code-20260901.tar.gz.part00..02` | code 包 3 分块（95+95+71MiB，重组后 262MB） |
| `assets/token2wav-rts-nfe2.prompt_cache.gguf` | **NFE2 prompt cache（独有资产，RTS 必需）** 90MB |
| `SHA256SUMS` | 全部校验和（含重组后整包哈希） |

## 目标机重组（310P 机器上）

```bash
git clone -b port310p/release --depth 1 git@ssh.github.com:443:Phoenix3334/minicpmo45-ascend-private.git port310p-dl
cd port310p-dl
sha256sum -c SHA256SUMS                       # 先校验分块
cat parts/PORT310P-code-20260901.tar.gz.part* > /tmp/PORT310P-code-20260901.tar.gz
sha256sum /tmp/PORT310P-code-20260901.tar.gz  # 应 = 19d4deef…6483c
tar xzf /tmp/PORT310P-code-20260901.tar.gz    # 得到 PORT310P-kit/
# NFE2 cache 放进 kit 布局：
mkdir -p PORT310P-kit/models/token2wav-rts-nfe2
mv assets/token2wav-rts-nfe2.prompt_cache.gguf PORT310P-kit/models/token2wav-rts-nfe2/prompt_cache.gguf
```

## 模型权重（20.2GB）不在这个分支——两条获取路径

**路径 A（推荐）：目标机直接下公共权重。** 权重 95% 是公开的
`openbmb/MiniCPM-o-4_5-gguf`（HF / ModelScope 镜像），按 kit 需要的布局放置：

```
PORT310P-kit/models/OpenBMB/MiniCPM-o-4_5-gguf/
├── MiniCPM-o-4_5-F16.gguf            (16.4GB)
├── audio/MiniCPM-o-4_5-audio-F16.gguf  (660MB)
├── tts/{MiniCPM-o-4_5-tts-F16.gguf, MiniCPM-o-4_5-projector-F16.gguf}
├── vision/MiniCPM-o-4_5-vision-F16.gguf (1.1GB)
└── token2wav-gguf/{encoder,flow_extra,flow_matching,hifigan2,prompt_cache}.gguf
```
（完整清单与哈希见 code 包内 `MANIFEST-PORT.txt`；本分支的 NFE2 cache 是唯一的
非公开权重资产，已在 assets/ 里。）

**路径 B：拷贝 21GB models tarball**（`PORT310P-models-20260901.tar`，sha256
`5710dcdc…8f738`，内含 7 份指导文件）。本机到 GitHub 的可用通道
（SSH git ~10min/350MB 级）传 21GB 需十余小时且要拆 200+ 个 <100MiB 块，未上传；
如需走 GitHub Release（单资产 ≤2GiB，需 `gh auth login`）或 scp/U盘，另行操作。

## 为什么要分块、为什么不是 LFS

- GitHub 普通 blob 上限 100MiB → `split -b 95M`；
- git-lfs 上传/下载都走 `objects.githubusercontent.com` CDN，本机实测被限到
  ~4KB/s（10 分钟 2.4MB），LFS 路线在本机不可行；
- SSH-443 git 通道（`ssh.github.com:443`）是本机唯一快速上行，走纯 git push。

---
完整使用文档在 code 包内 `README-PORT.md`（310P 差异矩阵/四步运行/排障）。
