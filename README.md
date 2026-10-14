# Diana 本地补丁 + 自动编译

这个分支只放补丁和编译流程，不含 Diana 源码。源码每次从上游 `SuInk/diana` 的指定版本拉取。

## 补丁

| 文件 | 作用 |
| --- | --- |
| `patches/0001-bilibili-wbi-view.patch` | B 站元数据接口 `view` → `wbi/view`（旧接口对美国 IP 返回 412） |
| `patches/0002-rss-twitter-images.patch` | X 订阅推送附带命中推文的配图（仅 `pbs.twimg.com`，每条 ≤2 张，每次 ≤4 张），含测试 |

上游合并了哪个，就删掉对应的补丁文件。

## 编译

Actions → **Build patched Diana** → Run workflow，填上游版本号（如 `v0.8.150`）。

产出：

- 镜像 `ghcr.io/c837835323/diana:<版本>-patched`（以官方同版本镜像为底，只替换 `/app/diana-webui`）
- 二进制 `diana-webui`（在运行页面的 Artifacts 里，保留 7 天）

补丁套不上或测试失败时，运行会直接失败，不会产出镜像。

## 本地编译（不用 Actions 时）

```sh
git clone --depth 1 --branch <版本> https://github.com/SuInk/diana && cd diana
for p in /path/to/patches/*.patch; do git apply "$p"; done
go test ./model/assistant/ -run 'RSSWatch|Bilibili' -count=1
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -trimpath -ldflags="-s -w -X main.buildVersion=<版本>" -o diana-webui ./cmd/webui
```
