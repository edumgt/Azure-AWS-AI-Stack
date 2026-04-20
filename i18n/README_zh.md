🌐 **语言 / Language:** [한국어](../README.md) | [English](./README_en.md) | [日本語](./README_ja.md) | [中文](./README_zh.md)

---

# AWS AI IaaS 综合 AI 解决方案提案书（宣传册）
**基于 Lex · Rekognition · Textract · Bedrock · Kendra · Lambda**  
**OCR / 文档自动化 · PDF 输出 · 二维码 · 客户专属 AI 智能体开发**

---

## 1. 概述（执行摘要）

本文档是一份综合提案宣传册，概述了将 AWS 托管 AI 服务与无服务器 / 云原生技术栈相结合的整合交付模型，用于针对每位客户的需求（业务流程、数据治理、安全性和 SLA）设计、构建和运营 AI 解决方案。

核心服务领域如下：

1. 对话式 AI（呼叫中心 / 聊天机器人）  
2. 视觉 AI（人脸 / 对象 / 文字识别）  
3. 文档 AI（OCR · 数据提取）  
4. 生成式 AI / RAG（基于内部文档的答复）  
5. 自动文档输出（PDF）  
6. 基于二维码的业务集成  
7. 客户专属 AI 智能体开发  

---

## 2. 解决方案组合

### 2.1 对话式 AI — 基于 Amazon Lex 的业务聊天机器人 / 语音机器人

- 多轮对话（意图 / 槽位）收集业务参数 → Lambda 执行（内部 API / 数据库集成）
- 呼叫中心 / IVR / 客户支持：常见问题、订单 / 物流查询、账户 / 权限管理、受理 / 投诉处理、预约 / 变更
- 语音输入输出扩展：Transcribe（语音 → 文字）、Polly（文字 → 语音）集成

### 2.2 视觉 AI — 基于 Amazon Rekognition 的图像 / 视频分析

- 人脸检测 / 分析：DetectFaces 返回人脸位置、置信度、关键点、姿势、遮挡等属性
- 人脸比对 / 身份验证：注册照片与提交照片比对、重复注册检测、门禁认证
- 标签（对象 / 场景）检测及文字检测，用于内容自动标注 / 审核流水线

### 2.3 文档 AI — Amazon Textract OCR / 文档数字化

- 从扫描 PDF / 图像中自动提取文字、手写、表格和表单数据
- 提供票据 / 收据 / 合同 / 身份证 / 检查表 / 验收单等文档数字化及验证工作流
- 基于规则的精炼 + LLM 后处理（字段标准化 / 摘要 / 异常检测）提升质量

### 2.4 生成式 AI / RAG — 基于 Bedrock · Kendra 的内部知识问答

- 内部文档（S3 / 门户 / Wiki / 政策 / 手册）检索（Kendra）+ 生成式答复（Bedrock）
- 提供答案依据（参考链接 / 文档片段）；应用政策护栏（违禁词 / 个人信息等）
- 客户级租户 / 知识库隔离（含权限、加密和日志记录）

### 2.5 PDF 输出 — 自动生成报告 / 证书 / 验收单 / 合同

- 基于模板的 PDF 渲染（含表格 / 表单 / 印章 / 签名图像），支持大批量输出
- 生成历史 / 版本管理、水印、密码 / 权限控制、审计日志
- 成果物存储（S3）+ 下载 API + 过期策略（Lifecycle）

### 2.6 二维码 — 线下到线上业务集成

- 在文档 / 资产 / 现场设备 / 门禁证 / 工单中嵌入二维码 → 通过移动设备即时查询 / 登记 / 审批
- 二维码支持基于令牌 / 签名的短链接（含过期时间 / 一次性使用 / 权限控制）
- 自动收集扫码事件日志 / 业务历史（现场数字化转型）

### 2.7 客户专属 AI 智能体开发 — 业务编排智能体

- 安全连接客户系统（API / 数据库 / 协同办公系统）工具，实现"通过对话执行业务"
- 示例："帮我总结合同并提交审批"、"生成现场检查 PDF 并分发给负责人"
- 基于策略的工具使用（权限 / 审批 / 审计），租户隔离，提示词 / 知识 / 工具集自定义

---

## 3. 参考架构（代表性配置）

### 3.1 事件驱动型视觉 / 文档处理流水线

- S3 上传 → （S3 事件）Lambda → Rekognition / Textract → 结果存储（DynamoDB / RDS / OpenSearch）→ 通知（SNS）
- 流量突增 / 需要重新处理时：S3 → SQS → Lambda（缓冲、**DLQ**、重试）模式

### 3.2 Lex 业务聊天机器人 + 内部系统集成

- Lex（意图 / 槽位）→ Fulfillment Lambda → 内部 API（ERP / CRM / 协同办公）→ 结果响应
- 长时间任务：进度更新消息 + 异步作业（队列 / Step Functions）改善用户体验

### 3.3 RAG 知识机器人（基于内部文档）

- 文档采集 / 精炼 → 索引（Kendra / 向量）→ 查询 → 检索结果 → Bedrock 生成答复 + 政策（护栏）

---

## 4. 构建 / 运营方法论

- **第一阶段（2〜4 周）**：需求 / 安全 / 数据分类，PoC（准确率 / 延迟 / 成本）
- **第二阶段（4〜8 周）**：MVP 构建（核心场景 / 权限 / 审计 / 部署自动化）
- **第三阶段（持续）**：高级化（质量、用户反馈、知识 / 工具扩展、基于运营指标的优化）

---

## 5. 必要的 AWS 技术栈（清单）

### 5.1 计算 / 集成
- AWS Lambda，（可选）Step Functions、EventBridge
- API Gateway / ALB、SQS、SNS

### 5.2 数据 / 存储
- S3（原始 / 成果物），DynamoDB（元数据 / 历史），RDS（业务数据），（可选）OpenSearch（搜索）
- KMS（加密），S3 Lifecycle / 版本管理

### 5.3 AI 服务
- Lex（对话），Rekognition（视觉），Textract（OCR），Bedrock（生成式），Kendra（搜索 / RAG）
- （可选）Transcribe / Polly（语音）

### 5.4 安全 / 网络
- IAM（最小权限），VPC / 安全组，（如可能）VPC Endpoints / PrivateLink，WAF
- Secrets Manager / Parameter Store，CloudTrail（审计）

### 5.5 可观测性 / 运营
- CloudWatch Logs / Metrics / Alarms，X-Ray（分布式追踪），故障告警集成

### 5.6 DevOps / IaC
- CloudFormation / CDK / Terraform，CodeBuild / CodePipeline 或 GitHub Actions

---

## 6. 代表性应用场景

- **现场检查**：扫描二维码 → 填写检查清单 → 上传照片 → Rekognition / Textract 自动分析 → 自动生成 / 分发检查 PDF
- **门禁 / 身份验证**：通过应用拍照 → Rekognition 人脸比对 → 通过后发放门禁证 PDF / 二维码 + 审计日志存储
- **文档自动化**：上传合同 / 票据 PDF → Textract 提取 → 字段验证 → 通过 RAG 自动提示相关规定 / 指南
- **客户支持**：Lex 语音机器人受理咨询 → 内部系统创建工单 → 通过聊天提示进度状态
- **内部知识机器人**：基于政策 / 手册的检索 + 摘要答复（Bedrock / Kendra）+ 提供依据链接

---

## 7. 参考资料（官方文档 / 服务说明）

- [打开提案 PDF](./AWS_AI_Solution_Company_Brochure_ko.pdf)

- Amazon Rekognition DetectFaces API: https://docs.aws.amazon.com/rekognition/latest/APIReference/API_DetectFaces.html  
- AWS Lambda — 使用 Lambda 处理 Amazon S3 事件通知: https://docs.aws.amazon.com/lambda/latest/dg/with-s3.html  
- Amazon Lex V2 — 集成 Lambda: https://docs.aws.amazon.com/lexv2/latest/dg/lambda.html  
- Amazon Textract OCR: https://aws.amazon.com/textract/ocr/  
- Amazon Textract 概述: https://aws.amazon.com/textract/  
