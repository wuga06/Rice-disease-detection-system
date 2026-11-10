# 基于卷积神经网络的水稻病虫害图像识别系统

本项目基于 `MobileNetV2` 迁移学习，实现水稻病害分类训练、多模式推理、Flask REST API、Mistral AI 农业问答与响应式前端界面。

## 1. 项目结构

```text
RiceProject/
├── src/
│   ├── train.py                    # 模型训练与评估
│   ├── run_inference.py            # 命令行推理入口
│   ├── inference/engine.py         # 多模式推理引擎
│   └── api/
│       ├── app.py                  # Flask 服务端
│       └── services/
│           ├── mistral_client.py   # Mistral API 封装
│           └── knowledge_base.py   # 知识库加载
├── knowledge/solutions.json        # 病害防治方案知识库
├── web/                            # 前端页面
├── models/                         # 训练后模型与类别文件
├── outputs/                        # 指标与混淆矩阵输出
└── requirements.txt
```

## 2. 环境准备（首次）

在 PowerShell 执行：

```powershell
cd D:\riceprogram\RiceProject
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

## 3. 配置 Mistral API Key（问答功能）

项目支持读取 `.env` 中的 `MISTRAL_API_KEY`：

```env
MISTRAL_API_KEY=your_mistral_api_key
PORT=5000
```

## 4. 训练模型（推荐顺序）

### 4.1 数据集路径

数据集默认路径：`D:\riceprogram\Rice`  
目录结构应包含 `train/` 和 `test/` 子目录。

### 4.2 执行训练

```powershell
python -m src.train --data-dir D:\riceprogram\Rice --head-epochs 10 --finetune-epochs 30 --batch-size 16
```

参数说明：

- `--head-epochs`：冻结主干网络后，训练分类头轮数
- `--finetune-epochs`：解冻后微调整体轮数
- `--batch-size`：批大小
- `--head-lr` / `--finetune-lr`：两个阶段学习率

### 4.3 训练输出

训练完成后会生成：

- `models/rice_mobilenetv2.keras`：训练好的模型
- `models/class_names.json`：类别映射
- `outputs/metrics.json`：准确率/精确率/召回率/F1 等指标
- `outputs/confusion_matrix.png`：混淆矩阵图
- `outputs/train_runtime.json`：训练耗时

## 5. 启动项目（推理 + Web）

**推荐方式（最稳定）：**

```powershell
cd D:\riceprogram\RiceProject
.\scripts\start.ps1
```

或手动启动：

```powershell
cd D:\riceprogram\RiceProject
.\.venv\Scripts\Activate.ps1
$env:USE_HTTPS="0"
python -m src.api.app
```

浏览器**必须**打开（注意是 `http` 且是 `127.0.0.1`）：

- [http://127.0.0.1:5000](http://127.0.0.1:5000)

`127.0.0.1` 属于浏览器安全环境，**摄像头可用**。不要用 `192.168.x.x`。

终端里应看到 `Running on http://127.0.0.1:5000`，否则说明服务未启动成功。

可选 HTTPS（局域网访问时需要，可能遇到证书提示）：

```powershell
pip install cryptography pyopenssl
$env:USE_HTTPS="1"
python -m src.api.app
```

然后访问 [https://127.0.0.1:5000](https://127.0.0.1:5000)

## 6. 命令行推理（可选）

```powershell
python -m src.run_inference --mode image --input "D:\test.jpg"
python -m src.run_inference --mode video --input "D:\test.mp4" --output "D:\out.mp4"
python -m src.run_inference --mode batch --input "D:\a.jpg,D:\b.jpg"
python -m src.run_inference --mode camera
```

## 7. API 接口列表

- `GET /api/health`：服务健康检查
- `GET /api/model-info`：模型文件状态
- `POST /api/predict/image`：单图识别
- `POST /api/predict/video`：视频识别
- `POST /api/predict/batch`：批量图片识别
- `POST /api/chat`：普通问答
- `POST /api/chat/stream`：SSE 流式问答
- `GET /api/solution/<disease_class>`：防治方案查询

## 8. 常见问题

- 模型不存在：请先完成训练，确认 `models/` 下存在模型文件
- 摄像头打不开：检查系统权限，或关闭占用摄像头的软件
- 问答无响应：确认 `MISTRAL_API_KEY` 是否有效
cd D:\riceprogram\RiceProject
.\scripts\start.ps1