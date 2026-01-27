# Ethereum Trading MCP Server（以太坊交易 MCP 服务器）

<div align="center">

[![Rust](https://img.shields.io/badge/Rust-000000?style=flat&logo=rust&logoColor=white)](https://www.rust-lang.org/)
[![Tokio](https://img.shields.io/badge/Tokio-000000?style=flat&logo=rust&logoColor=white)](https://tokio.rs/)
[![Ethereum](https://img.shields.io/badge/Ethereum-3C3C3D?style=flat&logo=ethereum&logoColor=white)](https://ethereum.org/)
[![Ethers.rs](https://img.shields.io/badge/Ethers.rs-627EEA?style=flat&logo=ethereum&logoColor=white)](https://github.com/gakonst/ethers-rs)
[![MCP](https://img.shields.io/badge/MCP-Model%20Context%20Protocol-000000?style=flat)](https://modelcontextprotocol.io/)
[![Uniswap](https://img.shields.io/badge/Uniswap-FF007A?style=flat&logo=uniswap&logoColor=white)](https://uniswap.org/)

</div>

> [English](../README.md) | [简体中文](README.zh-CN.md)

这是一个使用 Rust 编写的 Model Context Protocol（MCP）服务器，使 AI 代理能够查询以太坊账户余额并在 Uniswap 上模拟代币交换。

**[快速开始指南](quick-start.md)** —— 帮助你迅速运行项目。

## 功能特性

服务器提供三种 MCP 工具：

1. **`get_balance`** —— 查询 ETH 与 ERC20 代币余额

   - 输入：钱包地址，可选的代币合约地址
   - 输出：带有正确小数位的余额信息

2. **`get_token_price`** —— 获取代币的 USD 与 ETH 现价

   - 输入：代币地址（0x...）或代号（例如 “USDC”“WETH”）
   - 输出：来自 CoinGecko API 的价格数据

3. **`swap_tokens`** —— 在 Uniswap V2/V3 上模拟代币交换
   - 输入：from_token、to_token、amount、slippage tolerance
   - 输出：估算的兑换结果与 Gas 成本
   - **说明**：构造真实的 Uniswap 交易并通过 `eth_call` 进行模拟，不会在链上执行

## 项目结构

```
src/
├── main.rs          # 服务器入口与传输配置
├── server.rs        # MCP 服务器与工具处理逻辑
├── swap.rs          # Uniswap V2/V3 兑换模拟实现
└── bin/
    ├── client.rs               # 用于测试的 MCP 客户端
    ├── test_eth_balance.rs     # ETH 余额查询测试
    ├── test_erc20_balance.rs   # ERC20 余额查询测试
    ├── test_token_price.rs     # 代币价格查询测试
    └── test_swap.rs            # 兑换模拟测试
```

## 依赖组件

- **Rust**（最新稳定版）
- **tokio** —— 异步运行时
- **ethers-rs** —— 以太坊 RPC 客户端库
- **rmcp** —— Rust 版 MCP SDK
- **rust_decimal** —— 金融级高精度处理
- **tracing** —— 结构化日志
- **reqwest** —— 价格 API 的 HTTP 客户端
- **ethabi** —— ABI 编码与解码

## 安装与运行

### 前置条件

1. 安装 Rust（如尚未安装）：

   ```bash
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   ```

2. 克隆仓库：

   ```bash
   git clone https://github.com/chenjjiaa/eth-mcp;
   cd eth-mcp
   ```

### 环境变量

在项目根目录创建 `.env`（或直接设置环境变量）：

```bash
# 以太坊 RPC 端点（必填）
ETH_RPC_URL=https://eth.llamarpc.com
# 也可使用 Infura/Alchemy：
# ETH_RPC_URL=https://mainnet.infura.io/v3/YOUR_API_KEY

# 服务器配置（可选）
SERVER_HOST=127.0.0.1        # 默认值：127.0.0.1
SERVER_PORT=0                # 默认值：0（stdio 模式），>0 则启用 TCP 模式

# 日志（可选）
RUST_LOG=info                 # 默认 info，可选 trace、debug、info、warn、error

# 钱包地址
WALLET_ADDRESS=0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045
# USDC 合约地址
TOKEN_ADDRESS=0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48
```

**提示**：当 `SERVER_PORT=0` 时，服务器默认使用符合 MCP 规范的 stdio 模式。若需 TCP 模式，请设置为有效端口。

### 构建与运行

1. **构建项目**：

   ```bash
   cargo b
   # 或发布模式：
   cargo b --release
   ```

2. **启动服务器**：

   ```bash
   # stdio 模式（默认，供 MCP 客户端使用）
   cargo r --bin server

   # 或直接运行二进制
   ./target/debug/server
   ```

3. **运行测试**：

   ```bash
   # 运行全部测试
   cargo test

   # 运行特定测试二进制
   cargo r --bin test_eth_balance
   cargo r --bin test_erc20_balance
   cargo r --bin test_token_price
   cargo r --bin test_swap
   ```

## MCP 工具调用示例

### 1. 查询 ETH 余额

**请求**：

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "get_balance",
    "arguments": {
      "wallet_address": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb"
    }
  }
}
```

**响应**：

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "{\n  \"wallet_address\": \"0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb\",\n  \"token_address\": null,\n  \"balance\": \"0.000000000000000000\",\n  \"decimals\": 18,\n  \"raw_balance\": \"0\"\n}"
      }
    ]
  }
}
```

### 2. 查询 ERC20 余额

**请求**：

```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "tools/call",
  "params": {
    "name": "get_balance",
    "arguments": {
      "wallet_address": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
      "token_address": "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48"
    }
  }
}
```

**响应**：

```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "{\n  \"wallet_address\": \"0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb\",\n  \"token_address\": \"0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48\",\n  \"balance\": \"1000.000000\",\n  \"decimals\": 6,\n  \"raw_balance\": \"1000000000\"\n}"
      }
    ]
  }
}
```

### 3. 查询代币价格

**请求**：

```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "method": "tools/call",
  "params": {
    "name": "get_token_price",
    "arguments": {
      "token": "USDC"
    }
  }
}
```

**响应**：

```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "{\n  \"token\": \"USDC\",\n  \"token_address\": null,\n  \"price_usd\": \"1.000000\",\n  \"price_eth\": \"0.000300\",\n  \"last_updated\": null\n}"
      }
    ]
  }
}
```

### 4. 模拟代币兑换

**请求**：

```json
{
  "jsonrpc": "2.0",
  "id": 4,
  "method": "tools/call",
  "params": {
    "name": "swap_tokens",
    "arguments": {
      "from_token": "ETH",
      "to_token": "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48",
      "amount": "1.0",
      "slippage_tolerance": "0.5",
      "version": "v2"
    }
  }
}
```

**响应**：

```json
{
  "jsonrpc": "2.0",
  "id": 4,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "{\n  \"from_token\": \"ETH\",\n  \"to_token\": \"0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48\",\n  \"input_amount\": \"1.0\",\n  \"estimated_output\": \"3200.123456\",\n  \"minimum_output\": \"3184.122842\",\n  \"slippage_tolerance\": \"0.5\",\n  \"estimated_gas\": \"150000\",\n  \"estimated_gas_eth\": \"0.003000\",\n  \"price_impact\": null,\n  \"involves_eth\": true,\n  \"version\": \"v2\"\n}"
      }
    ]
  }
}
```

## 设计考量

1. **MCP 协议实现**：使用 `rmcp` SDK 处理 MCP 通信，提供类型安全的工具注册与请求响应。服务器同时支持 stdio（标准 MCP 模式）与 TCP 传输。
2. **以太坊交互**：依赖 `ethers-rs` 进行以太坊 RPC 调用，使用 `eth_call` 读取 ERC20 余额和模拟兑换，确保不需要私钥即可获取链上真实数据。
3. **金融精度**：全局使用 `rust_decimal` 处理金额，避免浮点误差，并根据代币 `decimals` 输出正确小数位。
4. **兑换模拟**：构造真实的 Uniswap V2/V3 交易并通过 `eth_call` 在本地执行，提供包含 Gas 成本的精确估算，而无需上链。
5. **价格数据**：集成 CoinGecko API，既支持合约地址也支持常见代币符号，并提供回退机制。

## 已知限制与假设

1. **仅支持以太坊主网**：当前默认连接主网，如需 Goerli 或 Sepolia 需额外配置并替换 Uniswap 路由地址。
2. **价格 API 依赖**：CoinGecko 免费 API 存在频率限制，且并非支持所有代币。
3. **Uniswap 路由写死**：代码中硬编码了主网的 V2/V3 路由地址，若网络或路由升级需手动修改。
4. **Gas 估算不精确**：Gas 费用基于模拟，真实交易可能受网络状态影响。
5. **无钱包管理与签名**：仅做模拟，不会执行链上交易，也不处理私钥与签名。
6. **滑点计算简化**：使用固定百分比公式，尚未实现基于池子流动性的动态滑点。
7. **价格影响计算有限**：对于大额交易的价格冲击尚未全面覆盖。

## TODO / 后续改进

[Future Improvements](future-improvements.md)

## 更新日志

[Changelog](CHANGELOG.md)

## 许可证

本项目遵循 [Apache License 2.0](../LICENSE) 授权条款。
