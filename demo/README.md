# MinerU API 使用示例和文档

本目录包含了 MinerU 的各种 API 调用方式的完整示例和使用指南。

## 📁 文件结构

```
demo/
├── README.md                           # 本说明文档
├── MinerU_API使用指南.md               # 完整的 API 使用指南
├── demo.py                            # 原始演示脚本
├── fastapi_integration_demo.py        # FastAPI 集成示例
├── direct_function_demo.py            # 直接函数调用示例
└── tianshu_enterprise_demo.py         # 天枢企业级服务示例
```

## 📖 文档和示例说明

### 1. MinerU_API使用指南.md
**完整的 API 使用指南文档**
- 📋 详细的 API 调用方法说明
- 🔧 参数配置和选项说明
- 💡 最佳实践和使用建议
- 🚀 快速开始指南

### 2. fastapi_integration_demo.py
**FastAPI 集成示例**
- 🌐 完整的 FastAPI 应用示例
- 🔄 同步和异步解析端点
- 📦 批量处理功能
- 📊 任务状态管理
- 🏥 健康检查和监控

**主要功能:**
- `/parse` - 同步文档解析
- `/parse/async` - 异步文档解析
- `/parse/batch` - 批量文档处理
- `/task/{task_id}` - 任务状态查询
- `/health` - 服务健康检查
- `/backends` - 可用后端列表

### 3. direct_function_demo.py
**直接函数调用示例**
- 🔧 直接调用 MinerU 核心函数
- ⚡ 同步和异步解析方法
- 📚 批量处理功能
- 🛡️ 错误处理和异常管理
- 📊 结果处理和文件管理

**主要类:**
- `MinerUDirectClient` - 直接调用客户端
- 支持同步/异步/批量解析
- 完整的错误处理机制

### 4. tianshu_enterprise_demo.py
**天枢企业级服务示例**
- 🏢 企业级多 GPU 服务集成
- 🔄 任务提交和管理
- 📈 批量处理和负载均衡
- 📊 任务状态监控
- 📥 结果下载和管理

**主要类:**
- `TianshuClient` - 天枢服务客户端
- `TianshuTaskManager` - 任务管理器
- 支持大规模并发处理

## 🚀 快速开始

### 1. 使用内置 FastAPI 服务器
```bash
# 启动 MinerU API 服务器
mineru-api

# 访问 API 文档
# http://127.0.0.1:8000/docs
```

### 2. 运行 FastAPI 集成示例
```bash
cd demo
python fastapi_integration_demo.py

# 访问示例应用
# http://localhost:8001
```

### 3. 运行直接函数调用示例
```bash
cd demo
python direct_function_demo.py
```

### 4. 运行天枢企业服务示例
```bash
# 确保天枢服务器运行在 http://localhost:8080
cd demo
python tianshu_enterprise_demo.py
```

## 📋 使用前准备

### 环境要求
- Python 3.8+
- 已安装 MinerU
- 相关依赖包 (aiohttp, fastapi, uvicorn 等)

### 演示文件
在运行示例前，请确保有测试 PDF 文件:
```
demo/
└── pdfs/
    ├── demo1.pdf
    ├── demo2.pdf
    └── demo3.pdf
```

### 输出目录
示例会在以下目录创建输出文件:
- `./demo_output/` - 直接函数调用输出
- `./downloads/` - 天枢服务下载文件
- `./temp_uploads/` - FastAPI 临时上传文件

## 🔧 配置说明

### 后端选择
- **pipeline**: 默认后端，速度快，适合大多数场景
- **vlm-transformers**: 高精度视觉语言模型
- **vlm-vllm**: 优化的视觉语言模型，性能更好

### 输出格式
- **md**: Markdown 格式
- **json**: 中间 JSON 格式
- **content_list**: 内容列表格式

### 语言支持
- **ch**: 中文
- **en**: 英文

## 🛡️ 错误处理

所有示例都包含完整的错误处理机制:
- 文件不存在检查
- 文件格式验证
- 网络请求异常处理
- 解析错误处理
- 超时处理

## 📊 性能优化建议

1. **批量处理**: 使用批量 API 提高吞吐量
2. **异步调用**: 使用异步方法避免阻塞
3. **并发控制**: 合理设置并发数量
4. **资源管理**: 及时清理临时文件
5. **错误重试**: 实现重试机制提高稳定性

## 🔗 相关链接

- [MinerU 官方文档](https://github.com/opendatalab/MinerU)
- [FastAPI 文档](https://fastapi.tiangolo.com/)
- [异步编程指南](https://docs.python.org/3/library/asyncio.html)

## 💡 提示

- 所有示例都可以独立运行
- 可以根据需要修改配置参数
- 建议先阅读 `MinerU_API使用指南.md` 了解详细用法
- 遇到问题请检查日志输出和错误信息

## 🤝 贡献

欢迎提交 Issue 和 Pull Request 来改进这些示例!