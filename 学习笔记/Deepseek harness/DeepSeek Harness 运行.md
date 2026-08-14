github：
https://github.com/deepseek-ai/deepseek-harness/blob/master/README.zh.md

## 运行

[](https://github.com/deepseek-ai/deepseek-harness/blob/master/README.zh.md#%E8%BF%90%E8%A1%8C)

### 通过 `npm` 运行

[](https://github.com/deepseek-ai/deepseek-harness/blob/master/README.zh.md#%E9%80%9A%E8%BF%87-npm-%E8%BF%90%E8%A1%8C)

安装 `Node.js`，然后运行：

```shell
npx @deepseek-ai/dsh web
```

该命令会启动 Web UI，默认地址为 `http://127.0.0.1:3080`。详见 [Web UI 指南](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/user/guide/index.md)。

### 从源码运行

[](https://github.com/deepseek-ai/deepseek-harness/blob/master/README.zh.md#%E4%BB%8E%E6%BA%90%E7%A0%81%E8%BF%90%E8%A1%8C)

如需从仓库源码运行：

```shell
git clone https://github.com/deepseek-ai/deepseek-harness.git
cd deepseek-harness
pnpm install
pnpm run build
pnpm dsh web
```

