# Ferron 文档

此目录包含 Ferron Web 服务器的文档。如果您正在寻找服务器文档，可以访问 <https://ferron.sh/docs>。

## `links.json` 文件

`links.json` 文件包含指向文档页面的链接列表。列表采用以下格式：

```json
[
  {
    "href": "/docs", // 目标路径
    "target": "_self", // 目标（例如 "_self" 或 "_blank"）
    "sub": false, // 该链接是否为子页面
    "label": "欢迎来到文档！", // 链接文本
  },
  // ...
]
```
