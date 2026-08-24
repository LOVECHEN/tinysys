# tinysys

极简 glibc 容器底座 —— **Alpine / scratch / distroless** 上的 glibc + OpenSSL3 运行时，让「glibc + OpenSSL3 动态链接」的程序跑在 **几 MB 到几十 MB** 的底座上。多架构（amd64 / arm64 / arm-v7 / ppc64le / riscv64 / s390x），单层，`LABEL 作者=LOVE`。

镜像发布在 Docker Hub：**[`lovechen/tinysys`](https://hub.docker.com/r/lovechen/tinysys)**（一个 repo，所有成品作 tag）。

```bash
docker pull lovechen/tinysys:alpine-cpp-openssl-s6       # 最常用：glibc+OpenSSL3+s6+C++，能进容器调试
docker pull lovechen/tinysys:scratch-glibc-openssl-s6    # 最小：纯空底、单层无 OS
docker pull lovechen/tinysys:distroless-glibc-openssl-s6 # 要 Google 维护的基底
```

## Tag 命名

滚动 tag = `<底>-<口味>-<级别>`（glibc 级即纯运行时，tag 就是 `<底>-<口味>`，如 `alpine-cpp`）：

- **底**：`alpine`（带 apk/busybox，可调试补包）· `scratch`（纯空底，非 s6 级真无 shell，最小）· `distroless`（Google 维护基底，含 busybox）
- **口味**：`glibc`（精简）· `cpp`（+libstdc++，**推荐**）· `full`（+全字符集 gconv）
- **级别**：glibc（纯运行时）· `-openssl`（+OpenSSL3/zstd）· `-s6`（+进程监管）· `-openssl-s6`（全家桶）

每个成品还带一个**钉死版本+日期的参考 tag**（可复现/审计），如 `alpine3.24.1-glibc2.39-openssl3.0.13-s6-3.2.3.2-20260822`。

## 完整对比表

全部 25 个成品，逐行拉齐。列含义：**C++**=带 libstdc++；**gconv**=保留冷门字符集（`iconv` 转 GBK/BIG5/SHIFT-JIS…）；**SSL**=OpenSSL3+zstd/lzma/bz2/ffi；**s6**=s6-overlay v3 进程监管；**shell**=能否进容器（`无`=最小攻击面）；**体积**=`docker export` 未压缩真实 rootfs（arm64）。

| 镜像（滚动 tag） | C++ | gconv | SSL | s6 | shell | 体积 arm64 (MB) |
|---|:--:|:--:|:--:|:--:|:--:|--:|
| `alpine-glibc` | ✗ | ✗ | ✗ | ✗ | apk+busybox | 14.6 |
| `alpine-glibc-openssl` | ✗ | ✗ | ✓ | ✗ | apk+busybox | 21.1 |
| `alpine-glibc-s6` | ✗ | ✗ | ✗ | ✓ | apk+busybox | 25.3 |
| `alpine-glibc-openssl-s6` | ✗ | ✗ | ✓ | ✓ | apk+busybox | 31.8 |
| `alpine-cpp` | ✓ | ✗ | ✗ | ✗ | apk+busybox | 17.1 |
| `alpine-cpp-openssl` | ✓ | ✗ | ✓ | ✗ | apk+busybox | 23.6 |
| `alpine-cpp-s6` | ✓ | ✗ | ✗ | ✓ | apk+busybox | 27.8 |
| `alpine-cpp-openssl-s6` ⭐ | ✓ | ✗ | ✓ | ✓ | apk+busybox | 34.3 |
| `alpine-full` | ✓ | ✓ | ✗ | ✗ | apk+busybox | 36.0 |
| `alpine-full-openssl` | ✓ | ✓ | ✓ | ✗ | apk+busybox | 42.5 |
| `alpine-full-s6` | ✓ | ✓ | ✗ | ✓ | apk+busybox | 46.7 |
| `alpine-full-openssl-s6` | ✓ | ✓ | ✓ | ✓ | apk+busybox | 53.2 |
| `scratch-glibc` | ✗ | ✗ | ✗ | ✗ | **无** | **4.0** |
| `scratch-glibc-openssl` | ✗ | ✗ | ✓ | ✗ | **无** | **10.5** |
| `scratch-glibc-s6` | ✗ | ✗ | ✗ | ✓ | busybox | 16.2 |
| `scratch-glibc-openssl-s6` | ✗ | ✗ | ✓ | ✓ | busybox | 22.7 |
| `scratch-cpp` | ✓ | ✗ | ✗ | ✗ | **无** | 6.6 |
| `scratch-cpp-openssl` | ✓ | ✗ | ✓ | ✗ | **无** | 13.1 |
| `scratch-cpp-s6` | ✓ | ✗ | ✗ | ✓ | busybox | 18.8 |
| `scratch-cpp-openssl-s6` | ✓ | ✗ | ✓ | ✓ | busybox | 25.3 |
| `scratch-full` | ✓ | ✓ | ✗ | ✗ | **无** | 25.4 |
| `scratch-full-openssl` | ✓ | ✓ | ✓ | ✗ | **无** | 31.9 |
| `scratch-full-s6` | ✓ | ✓ | ✗ | ✓ | busybox | 37.6 |
| `scratch-full-openssl-s6` | ✓ | ✓ | ✓ | ✓ | busybox | 44.1 |
| `distroless-glibc-openssl-s6` | ✓ | ✓ | ✓ | ✓ | busybox | 46.6 |

**架构**：alpine / scratch 全 6 种（amd64 · arm64 · arm/v7 · ppc64le · riscv64 · s390x）；distroless 5 种（无 riscv64，上游 distroless 没有）。`docker` 自动按平台拉。

> 注：`distroless-glibc-openssl-s6` 名字里的 `-glibc-` 只是命名约定；Google distroless/cc 基底本就带 libstdc++ + 完整 gconv，所以它实际是 **full 级内容**（表中 C++/gconv 都 ✓，并非精简口味）。

**读表规律**：

- 体积 **scratch < alpine < distroless**（精简/cpp 口味每级都成立）；`scratch-glibc` 最小 **4.0MB**。
- `cpp`（带 C++、剔 gconv）比 `full` 省一大截，覆盖绝大多数场景 → **`alpine-cpp-openssl-s6` ⭐ 默认首选**。
- 真正无 shell（最小攻击面）只有 **scratch 的非 s6 级**（`scratch-glibc/cpp/full` 及 `-openssl`）。
- 装上一个真实服务（~90MB 二进制）整机：scratch 65.5 / alpine 74.6 / distroless 89.4 MB —— 约同类官方镜像（真实 rootfs 309.5MB）的 **1/4**。

## 运行时内存（glibc 2.4x 的坑，务必知道）

同一个多线程程序（64 线程 × malloc），只换底层 glibc，cgroup 内存实测：

| glibc 版本 | 内存 | |
|---|--:|---|
| 2.35 | 7.2 MiB | ✅ |
| 2.36（distroless 底） | 5.2 MiB | ✅ |
| **2.39（本项目 alpine/scratch 底）** | **5.1 MiB** | ✅ |
| 2.43 | **134 MiB** | 💥 爆 ~25× |
| 2.43 + `MALLOC_ARENA_MAX=2` | 8.1 MiB | 压回 |

glibc **2.43** 多线程下 malloc 疯狂开 arena，RSS 暴涨 ~25×。tinysys 底座用 **glibc 2.39**、distroless 用 **2.36**，都躲开了这个坑——所以不仅磁盘小、运行时内存也低。任何 glibc 2.4x 多线程容器都建议加 `ENV MALLOC_ARENA_MAX=2`（成本为零）。

## 用法

`FROM` 对应成品，直接放 glibc 二进制即可（无需设 `LD_LIBRARY_PATH`）：

```dockerfile
FROM lovechen/tinysys:alpine-cpp-openssl-s6
ENV MALLOC_ARENA_MAX=2                    # 防 glibc 多线程内存膨胀（成本为零）
COPY --from=build /etc/s6-overlay/s6-rc.d /etc/s6-overlay/s6-rc.d   # 你的 s6 服务
COPY your-glibc-binary /usr/bin/
WORKDIR /data
# ENTRYPOINT ["/init"]   # s6 底已设好
```

## 选型速查

- **默认首选 → `alpine-cpp-openssl-s6`** ⭐：小、能调试、s6 适配、带 C++，覆盖 ~90% 场景。
- **极致最小 / 最小攻击面（真无 shell）→ scratch 非 s6 级**：`scratch-glibc` / `scratch-cpp`（可带 `-openssl`），连 busybox 都没有。
- **要 Google 维护/定期打补丁的基底 → `distroless-glibc-openssl-s6`**（是 s6 全家桶、含 busybox，并非无 shell）。
- **要冷门字符集 iconv → `*-full-*`**。

> 完整分析（口径说明、vs distroless、优劣排序、构建/发布流程图）见项目技术报告。
