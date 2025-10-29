# MinerU API 使用指南

## 📖 概述

MinerU 提供了多种API调用方式，让你可以轻松地将文档解析功能集成到自己的项目中，而无需使用命令行接口。本指南将详细介绍各种集成方式和使用场景。

## 🚀 快速开始

### 方式一：内置 FastAPI 服务器（推荐）

#### 1. 启动服务器

```bash
# 基本启动
mineru-api

# 自定义配置
mineru-api --host 0.0.0.0 --port 8080 --reload
```

#### 2. 访问API文档

- **Swagger UI**: http://127.0.0.1:8000/docs
- **ReDoc**: http://127.0.0.1:8000/redoc

#### 3. API 调用示例

**Python 请求示例：**

```python
import requests

# 上传文件进行解析
url = "http://127.0.0.1:8000/file_parse"

with open("document.pdf", "rb") as f:
    files = {"files": ("document.pdf", f, "application/pdf")}
    data = {
        "backend": "pipeline",
        "lang_list": ["ch"],
        "formula_enable": True,
        "table_enable": True,
        "return_md": True,
        "return_images": False
    }
    
    response = requests.post(url, files=files, data=data)
    result = response.json()
    
    # 获取解析结果
    markdown_content = result["results"]["document"]["md_content"]
    print(markdown_content)
```

**JavaScript/Node.js 示例：**

```javascript
const FormData = require('form-data');
const fs = require('fs');
const axios = require('axios');

async function parseDocument() {
    const form = new FormData();
    form.append('files', fs.createReadStream('document.pdf'));
    form.append('backend', 'pipeline');
    form.append('lang_list', 'ch');
    form.append('return_md', 'true');
    
    try {
        const response = await axios.post('http://127.0.0.1:8000/file_parse', form, {
            headers: form.getHeaders()
        });
        
        const markdownContent = response.data.results.document.md_content;
        console.log(markdownContent);
    } catch (error) {
        console.error('解析失败:', error.response.data);
    }
}

parseDocument();
```

**cURL 示例：**

```bash
curl -X POST "http://127.0.0.1:8000/file_parse" \
  -F "files=@document.pdf" \
  -F "backend=pipeline" \
  -F "lang_list=ch" \
  -F "return_md=true"
```

### 方式二：直接 Python 函数调用

#### 同步调用

```python
from mineru.cli.common import do_parse, read_fn
from pathlib import Path

def parse_document_sync(file_path: str, output_dir: str = "./output"):
    """同步解析文档"""
    
    # 读取文件
    pdf_bytes = read_fn(file_path)
    file_name = Path(file_path).stem
    
    # 调用解析函数
    result = do_parse(
        output_dir=output_dir,
        pdf_file_names=[file_name],
        pdf_bytes_list=[pdf_bytes],
        p_lang_list=["ch"],
        backend="pipeline",
        parse_method="auto",
        formula_enable=True,
        table_enable=True,
        f_dump_md=True,
        f_dump_middle_json=False,
        f_dump_content_list=False
    )
    
    # 读取生成的 Markdown 文件
    md_file = Path(output_dir) / file_name / "auto" / f"{file_name}.md"
    if md_file.exists():
        return md_file.read_text(encoding='utf-8')
    return None

# 使用示例
markdown_content = parse_document_sync("document.pdf")
print(markdown_content)
```

#### 异步调用

```python
import asyncio
from mineru.cli.common import aio_do_parse, read_fn
from pathlib import Path

async def parse_document_async(file_path: str, output_dir: str = "./output"):
    """异步解析文档"""
    
    # 读取文件
    pdf_bytes = read_fn(file_path)
    file_name = Path(file_path).stem
    
    # 异步调用解析函数
    await aio_do_parse(
        output_dir=output_dir,
        pdf_file_names=[file_name],
        pdf_bytes_list=[pdf_bytes],
        p_lang_list=["ch"],
        backend="pipeline",
        parse_method="auto",
        formula_enable=True,
        table_enable=True,
        f_dump_md=True,
        f_dump_middle_json=False,
        f_dump_content_list=False
    )
    
    # 读取生成的 Markdown 文件
    md_file = Path(output_dir) / file_name / "auto" / f"{file_name}.md"
    if md_file.exists():
        return md_file.read_text(encoding='utf-8')
    return None

# 使用示例
async def main():
    markdown_content = await parse_document_async("document.pdf")
    print(markdown_content)

asyncio.run(main())
```

### 方式三：天枢企业级服务

#### 1. 启动服务

```bash
cd projects/mineru_tianshu
python start_all.py --accelerator gpu --devices 0,1 --workers 4
```

#### 2. 客户端调用

```python
import asyncio
import aiohttp
from pathlib import Path

class TianshuClient:
    def __init__(self, api_url='http://localhost:8000'):
        self.api_url = api_url
        self.base_url = f"{api_url}/api/v1"
    
    async def submit_task(self, session, file_path: str, **kwargs):
        """提交解析任务"""
        with open(file_path, 'rb') as f:
            data = aiohttp.FormData()
            data.add_field('file', f, filename=Path(file_path).name)
            data.add_field('backend', kwargs.get('backend', 'pipeline'))
            data.add_field('lang', kwargs.get('lang', 'ch'))
            data.add_field('method', kwargs.get('method', 'auto'))
            data.add_field('formula_enable', str(kwargs.get('formula_enable', True)).lower())
            data.add_field('table_enable', str(kwargs.get('table_enable', True)).lower())
            
            async with session.post(f'{self.base_url}/tasks/submit', data=data) as resp:
                return await resp.json()
    
    async def get_task_status(self, session, task_id: str):
        """查询任务状态"""
        async with session.get(f'{self.base_url}/tasks/{task_id}') as resp:
            return await resp.json()
    
    async def wait_for_completion(self, session, task_id: str, timeout: int = 600):
        """等待任务完成"""
        import time
        start_time = time.time()
        
        while time.time() - start_time < timeout:
            status = await self.get_task_status(session, task_id)
            if status.get('status') == 'completed':
                return status
            elif status.get('status') == 'failed':
                raise Exception(f"任务失败: {status.get('error')}")
            
            await asyncio.sleep(2)
        
        raise TimeoutError("任务超时")

# 使用示例
async def parse_with_tianshu():
    client = TianshuClient()
    
    async with aiohttp.ClientSession() as session:
        # 提交任务
        result = await client.submit_task(session, "document.pdf")
        task_id = result['task_id']
        print(f"任务已提交，ID: {task_id}")
        
        # 等待完成
        final_result = await client.wait_for_completion(session, task_id)
        print("解析完成:", final_result)

asyncio.run(parse_with_tianshu())
```

## 🔧 集成到 FastAPI 项目

### 完整示例

```python
from fastapi import FastAPI, UploadFile, File, HTTPException
from fastapi.responses import JSONResponse
from mineru.cli.common import aio_do_parse, read_fn
import tempfile
import os
from pathlib import Path
import uuid

app = FastAPI(title="文档解析服务", description="基于 MinerU 的文档解析 API")

@app.post("/parse", summary="解析文档", description="上传 PDF 或图片文件进行解析")
async def parse_document(
    file: UploadFile = File(..., description="要解析的文件"),
    backend: str = "pipeline",
    language: str = "ch",
    formula_enable: bool = True,
    table_enable: bool = True
):
    """
    解析上传的文档文件
    
    - **file**: PDF 或图片文件
    - **backend**: 解析后端 (pipeline/vlm-transformers/vlm-vllm)
    - **language**: 文档语言 (ch/en)
    - **formula_enable**: 是否启用公式识别
    - **table_enable**: 是否启用表格识别
    """
    
    # 验证文件类型
    if not file.filename.lower().endswith(('.pdf', '.png', '.jpg', '.jpeg')):
        raise HTTPException(status_code=400, detail="不支持的文件类型")
    
    # 创建临时文件
    temp_dir = tempfile.mkdtemp()
    temp_file = os.path.join(temp_dir, file.filename)
    
    try:
        # 保存上传文件
        content = await file.read()
        with open(temp_file, "wb") as f:
            f.write(content)
        
        # 读取文件
        pdf_bytes = read_fn(temp_file)
        file_name = Path(file.filename).stem
        
        # 创建输出目录
        output_dir = os.path.join(temp_dir, "output")
        
        # 调用 MinerU 解析
        await aio_do_parse(
            output_dir=output_dir,
            pdf_file_names=[file_name],
            pdf_bytes_list=[pdf_bytes],
            p_lang_list=[language],
            backend=backend,
            parse_method="auto",
            formula_enable=formula_enable,
            table_enable=table_enable,
            f_dump_md=True,
            f_dump_middle_json=True,
            f_dump_content_list=True
        )
        
        # 读取结果
        result_dir = os.path.join(output_dir, file_name, "auto")
        results = {}
        
        # Markdown 内容
        md_file = os.path.join(result_dir, f"{file_name}.md")
        if os.path.exists(md_file):
            with open(md_file, 'r', encoding='utf-8') as f:
                results['markdown'] = f.read()
        
        # 中间 JSON
        json_file = os.path.join(result_dir, f"{file_name}_middle.json")
        if os.path.exists(json_file):
            with open(json_file, 'r', encoding='utf-8') as f:
                import json
                results['middle_json'] = json.load(f)
        
        return JSONResponse(content={
            "status": "success",
            "filename": file.filename,
            "backend": backend,
            "language": language,
            "results": results
        })
        
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"解析失败: {str(e)}")
    
    finally:
        # 清理临时文件
        import shutil
        shutil.rmtree(temp_dir, ignore_errors=True)

@app.get("/health")
async def health_check():
    """健康检查"""
    return {"status": "healthy", "service": "MinerU Document Parser"}

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8080)
```

## 📋 API 参数说明

### 通用参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `backend` | str | "pipeline" | 解析后端：pipeline/vlm-transformers/vlm-vllm |
| `lang_list` | List[str] | ["ch"] | 语言列表：ch(中文)/en(英文) |
| `parse_method` | str | "auto" | 解析方法：auto/ocr |
| `formula_enable` | bool | True | 是否启用公式识别 |
| `table_enable` | bool | True | 是否启用表格识别 |

### 输出控制参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `return_md` | bool | True | 返回 Markdown 格式 |
| `return_middle_json` | bool | False | 返回中间 JSON 数据 |
| `return_model_output` | bool | False | 返回模型原始输出 |
| `return_content_list` | bool | False | 返回内容列表 |
| `return_images` | bool | False | 返回提取的图片 |
| `response_format_zip` | bool | False | 以 ZIP 格式返回结果 |

### 页面范围参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `start_page_id` | int | 0 | 起始页码 |
| `end_page_id` | int | 99999 | 结束页码 |

## 🔍 后端选择指南

### Pipeline 后端（推荐）

- ✅ **支持 CPU 推理**
- ✅ **内存需求较低**（16GB+）
- ✅ **功能完整**（布局分析、OCR、公式、表格）
- ✅ **跨平台支持**
- ⚠️ **速度相对较慢**

**适用场景**：
- 服务器无 GPU 环境
- 内存资源有限
- 需要稳定可靠的解析

### VLM-Transformers 后端

- ✅ **解析质量高**
- ✅ **支持复杂文档**
- ❌ **需要 GPU**
- ❌ **内存需求大**（32GB+）

**适用场景**：
- 有 GPU 资源
- 对解析质量要求高
- 处理复杂学术文档

### VLM-VLLM 后端

- ✅ **推理速度快**
- ✅ **并发性能好**
- ❌ **需要 GPU**
- ❌ **部署复杂**

**适用场景**：
- 大规模生产环境
- 高并发需求
- 有专业运维团队

## 🚨 注意事项

### CPU 模式限制

1. **后端限制**：只有 `pipeline` 后端支持 CPU 推理
2. **内存需求**：最少 16GB RAM，推荐 32GB+
3. **存储需求**：20GB+ SSD 空间
4. **性能影响**：CPU 模式速度较慢，适合小批量处理

### 错误处理

```python
try:
    result = await aio_do_parse(...)
except Exception as e:
    if "CUDA" in str(e):
        print("GPU 不可用，请使用 pipeline 后端或安装 CUDA")
    elif "memory" in str(e).lower():
        print("内存不足，请减少批处理大小或增加内存")
    else:
        print(f"解析失败: {e}")
```

### 性能优化

1. **批处理**：一次处理多个文件可提高效率
2. **内存管理**：及时清理临时文件和缓存
3. **并发控制**：根据硬件资源调整并发数量

## 📚 更多资源

- [MinerU 官方文档](https://github.com/opendatalab/MinerU)
- [FastAPI 官方文档](https://fastapi.tiangolo.com/)
- [问题反馈](https://github.com/opendatalab/MinerU/issues)

---

*最后更新：2024年12月*