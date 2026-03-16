# AI Agent 开发知识笔记

> 本文档整理了基于 LangChain 的 AI Agent 开发核心概念、技术要点及常见面试问题，适用于学习笔记或技术复盘。

---

## 1. RAG（检索增强生成）

### 1.1 什么是 RAG？
RAG（Retrieval-Augmented Generation）是一种将**信息检索**与**大语言模型生成**相结合的架构。它通过从外部知识库检索相关文档片段，并将其作为上下文输入给 LLM，从而生成更准确、更新、更可靠的答案。

### 1.2 为什么需要 RAG？
- **克服知识截止日期**：LLM 的训练数据可能过时，RAG 可接入最新信息。
- **减少幻觉**：提供事实依据，降低模型编造的概率。
- **私有知识接入**：企业可将内部文档作为知识库，无需微调模型。

### 1.3 RAG 工作流程
```
用户问题 → 检索器（从向量数据库检索相似文档） → 增强提示（问题 + 检索文档） → LLM 生成答案
```
- **索引阶段**：将文档分块、向量化，存入向量数据库。
- **查询阶段**：将用户问题向量化，在向量数据库中执行相似性搜索，返回 top-k 文档块。
- **生成阶段**：将检索到的文档块与原始问题组合成提示，交给 LLM 生成最终答案。

### 1.4 关键点
- **分块策略**：块大小、重叠率影响检索质量。
- **元数据过滤**：可结合时间、来源等过滤，提升精准度。
- **多路召回**：可结合关键词搜索（BM25）与向量搜索，提高召回率。

---

## 2. 向量数据库与 Embedding

### 2.1 Embedding（嵌入）
- **定义**：将文本、图像等数据映射到高维向量空间的技术，使得语义相似的实体在向量空间中距离更近。
- **常用模型**：OpenAI `text-embedding-3-small`、阿里 `text-embedding-v2`、BAAI/bge-large-zh、`@xenova/transformers` 托管的本地模型等。
- **LangChain 使用示例**：
  ```javascript
  import { OpenAIEmbeddings } from "@langchain/openai";
  const embeddings = new OpenAIEmbeddings({ model: "text-embedding-3-small" });
  const vector = await embeddings.embedQuery("你好");
  ```

### 2.2 向量数据库
- **作用**：高效存储和检索高维向量，支持近似最近邻搜索（ANN）。
- **常见产品**：Chroma、Pinecone、Weaviate、Qdrant、Milvus。
- **Chroma 特点**：轻量级、开源、内存/持久化、支持 JS/Python，适合本地开发。

### 2.3 Chroma 核心概念
- **Collection**：一组向量的集合，可附带元数据。
- **Document**：包含 `pageContent` 和 `metadata` 的对象。
- **Embedding Function**：将文本转为向量的函数，可自定义。


### 2.4 Embedding 与向量数据库的关系
- **Embedding** 将文本转化为向量。
- **向量数据库** 存储这些向量，并提供检索能力。
- 二者结合实现了**语义搜索**，是 RAG 和长期记忆的基石。

---

## 3. ReAct Agent

### 3.1 什么是 ReAct？
ReAct（Reason + Act）是一种 Agent 设计模式，让 LLM 在循环中交替进行**推理**（思考下一步该做什么）和**行动**（调用工具），并根据观察结果调整计划。它由 Shunyu Yao 等人提出，LangChain 的 `create_react_agent` 实现了该模式。

### 3.2 ReAct 工作流程
1. **思考（Thought）**：分析当前状态，决定下一步行动。
2. **行动（Action）**：调用一个工具，传入参数。
3. **观察（Observation）**：获取工具返回的结果。
4. 重复上述步骤，直到得出最终答案。


### 3.3 核心设计要点
- **工具描述**：工具的名称和描述要清晰，便于 LLM 理解何时调用。
- **循环终止**：可设置最大迭代次数，防止死循环。
- **记忆管理**：Agent 需要访问历史消息，LangGraph 中可通过 `StateGraph` 管理状态。

### 3.4 ReAct vs. Plan-and-Execute
- **ReAct**：边思考边行动，灵活但可能效率较低。
- **Plan-and-Execute**：先规划步骤，再依次执行，适合确定性任务。

---

## 4. 什么是幻觉？为什么会发生？

### 4.1 幻觉的定义
**幻觉**指 LLM 生成的文本中包含与事实不符、虚构或无中生有的内容。例如，模型声称“爱因斯坦发现了引力波”，但实际是错的。

### 4.2 幻觉产生的原因
- **训练数据问题**：数据中存在错误、偏见或相互矛盾的信息。
- **模型架构缺陷**：LLM 本质是“下一个词预测器”，没有内在的事实校验机制。
- **解码策略**：随机采样、top-k 等可能引入不合理词汇组合。
- **知识截止日期**：模型不知道最新信息，却强行作答。
- **缺乏上下文**：当问题超出上下文窗口或未提供足够信息时，模型倾向于“编造”。

### 4.3 缓解幻觉的方法
- **RAG**：引入外部知识库，提供事实依据。
- **提示工程**：明确要求“如果不知道，请说不知道”。
- **检索增强微调**：在微调时加入检索任务。
- **自洽性检查**：多次生成并投票选取一致答案。
- **工具调用**：让模型调用计算器、数据库等工具获取准确信息。

### 4.4 在 Agent 中处理幻觉
- 强制 Agent 在不确定时调用搜索工具。
- 对工具返回的结果进行验证（例如检查 HTTP 状态码）。
- 使用 `output parsers` 确保输出格式符合预期，减少自由生成。

---

## 5. SSE（Server-Sent Events）推送实现

### 5.1 什么是 SSE？
SSE 是一种服务器向客户端推送数据的技术，基于 HTTP 协议，客户端通过 `EventSource` API 接收服务器推送的流式数据。与 WebSocket 相比，SSE 更轻量，适合单向文本数据流，如 LLM 的 token 流式输出。

### 5.2 SSE 与 Agent 流式输出
在 Agent 应用中，我们通常希望实时输出 LLM 生成的 token，而不是等待完整响应。SSE 是实现这一目标的常用方案。

### 5.3 Node.js 后端实现 SSE
```javascript
// 设置 SSE 响应头
  res.writeHead(200, {
    "Content-Type": "text/event-stream; charset=utf-8",
    "Cache-Control": "no-cache",
    Connection: "keep-alive",
  });
  // createAgent创建的是一个ReAct Agent
  // To stream tokens as they are produced by the LLM, use streamMode: "messages":配置流式返回
  const stream = await agent.stream({ messages }, { streamMode: "messages" });
```

### 5.4 前端接收 SSE
```javascript
const reader = response.body.getReader(); //获取响应流的读取器
const decoder = new TextDecoder(); // 解码二进制数据为文
while (true) {
  // read()返回 { done: 是否读完, value: 二进制数据块 }
  const { done, value } = await reader.read(); //异步阻塞：如果后端还没发送下一个数据块，代码会停在这一行等待，直到新数据到达或流结束。
  if (done) break;
  // 解码二进制数据并逐字追加到页面
  buffer += decoder.decode(value, { stream: true });
  // 按行分割 SSE 数据
  const lines = buffer.split("\n");
  buffer = lines.pop() || ""; // 最后不完整的行留在缓冲区

  for (const line of lines) {
    if (line.startsWith("data: ")) {
      const data = line.slice(6); // 去掉前缀 "data: "，获取真正的JSON字符串
      try {
        const parsed = JSON.parse(data);
        if (parsed.content) {
          onChunk(parsed.content);
        } else if (parsed.done) {
          onDone(parsed.fullContent);
        } else if (parsed.error) {
          onError(parsed.error);
        }
      } catch (e) {
        console.warn("Failed to parse chunk:", data);
      }
    }
  }
}
```

### 5.5 SSE 注意事项
- 每条消息必须以 `data: ` 开头，以 `\n\n` 结束。
- 连接超时问题：可定期发送心跳（如 `: keepalive\n\n`）。
- 浏览器限制：同一域名下并发 SSE 连接数有限（通常 6 个）。

---

## 6. 补充：AI Agent 开发重要概念

### 6.1 LangChain 核心组件
- **Models**：LLM 和 Embedding 模型封装。
- **Prompts**：模板化提示词管理。
- **Chains**：组合 LLM 和工具的调用序列。
- **Agents**：动态决策调用哪些工具。
- **Memory**：跨轮对话的状态管理。
- **Document Loaders**：从各种源加载文档。
- **Text Splitters**：文档分块。
- **Vector Stores**：向量数据库集成。
- **Output Parsers**：解析模型输出为结构化数据。

### 6.2 工具（Tool）设计原则
- **命名清晰**：如 `web_search`、`calculator`。
- **描述详尽**：说明何时使用、参数含义、返回格式。
- **输入输出规范**：使用 Zod 定义 schema，便于验证。
- **错误处理**：工具内部应捕获异常并返回友好信息，避免 Agent 崩溃。

### 6.3 Agent 的评估指标
- **成功率**：是否完成了用户任务。
- **工具调用正确率**：是否在正确时机调用了正确的工具。
- **效率**：步数、耗时、token 消耗。
- **鲁棒性**：对模糊指令、错误信息的处理能力。

### 6.4 常见面试题
1. **解释 RAG 的工作原理及其优点。**
2. **什么是 Embedding？为什么需要向量数据库？**
3. **ReAct Agent 和 Plan-and-Execute Agent 有什么区别？**
4. **如何减少 LLM 的幻觉？**
5. **SSE 和 WebSocket 的适用场景对比。**
6. **LangChain 中的 Chain 和 Agent 有何不同？**
7. **如何设计一个能够长期记忆用户偏好的 Agent？**
8. **在 LangChain 中如何实现工具调用？**
9. **什么是 Prompt 工程？如何优化提示词提高 Agent 表现？**
10. **如何监控和调试 Agent 的行为？**

---
