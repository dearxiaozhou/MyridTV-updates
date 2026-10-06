# MyridTV 增量更新通道

本仓库托管 MyridTV 客户端的**增量热更新资源**。客户端启动后按配置的更新地址拉取
`update-manifest.json`，仅下载内容发生变化的资源。

## 内容

- `update-manifest.json` — 更新清单，含版本号与全部可更新文件的 `sha256` 和下载地址。
  清单以 **Ed25519 私钥签名**，公钥内置于客户端（`update-public-key.pub`），客户端只接受签名有效的清单。
- `blob-<sha256>` — 内容寻址的资源文件。未发生变化的资源不会重复上传，新版本沿用上一版的下载地址。

## 重要约定

- **本仓库只发布前端静态资源（`_internal/static` 下的 `.html` / `.css` / `.js`），
  不包含任何 Python 源码。** 后端代码随安装包分发，不经此通道下发。
- **不要手工编辑或手工上传本仓库的任何文件。** 全部内容由
  `installer/make-update-release.py` 生成，签名与清单是配套的，手工改动会导致客户端验签失败。
- 发布旧版本资源时**必须保留对应的历史 Release**：跳过多版更新的客户端会沿用旧清单中的
  下载地址，删掉历史 Release 会让这些客户端拉取失败。

## 生成方式

```powershell
$env:MYRIDTV_UPDATE_SIGNING_KEY = '<base64 私钥>'
python installer/make-update-release.py `
  --stage installer/build/MyridTV `
  --out installer/build/update-release `
  --repo dearxiaozhou/MyridTV-updates `
  --previous installer/build/previous-update-manifest.json
```

未变化的文件沿用 `--previous` 清单中的地址，因此只有变化的资源被写进 `--out`。
`--previous` 可省略，此时会生成包含全部可更新文件的完整基线。
