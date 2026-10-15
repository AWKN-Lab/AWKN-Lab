# AWKN Lab 技术选型规则与现有项目清单

> 文件定位：`D:\awkn-lab\TECH-STACK-POLICY.md`（本机预定路径；Git 备份存放于 `AWKN-Lab/AWKN-Lab` 根目录）  
> 制定日期：2026-10-16  
> 状态：`GIT_SAVED / REMOTE_EVIDENCE_ONLY / LOCAL_WRITE_PENDING`  
> 本轮依据：GitHub 已连接仓库的 README、`package.json`、`pyproject.toml`、工程文档、状态文档及此前的 `ai-stack-audit-20261016.md`。  
> 证据限制：本机 ENO 与 LAY 文件接口本轮返回内部错误；Desktop Commander 设备 `eno` 为 Offline。尚未复验 `D:\awkn-lab` 根目录及各本地工作树，未执行本地构建、写入或部署。本文件保存至 Git 仓库根目录；本机尚未写入。完成本机根目录规范核对与同步后，方可将本地应用状态改为 `ACTIVE`。

## 一、总裁决

1. **现有项目优先保留已经运行的语言、数据库、框架与测试资产。** 选型变化须解决已测量的缺口；禁止仅因“AI 写 Go 更稳”而跨语言重写。
2. **统一运行合同、输入输出、错误语义、权限边界、证据回执和质量门禁。** 项目可以多语言，业务事实、数据写入权和组件责任必须唯一。
3. **默认先复用，再封装，最后新增。** 优先级：`COPY → ADAPTER → BUILD`。复制或吸收前核对上游 Exact SHA、许可证、来源路径、目标组件、差异与回退方案。
4. **文档遵循金字塔：系统 PRD → 大组件工程文档 → 模块章节 → 源码/测试 → Runtime Gate → Receipt → Exact SHA。** 技术选型改动记入受影响组件，不另造并行技术真源。
5. **性能、可靠性、AI 生成效率的结论须有对照实验。** 语言排行榜、编译成功率、测试总数、容器启动成功，都不能单独代表业务验收通过。

### 规则适用顺序

- 业务权限、隐私和安全硬约束最高。
- 当前产品/组件的 Owner、系统契约和已验证生产真值优先于此通用指南。
- 此文件管控**新增选型与技术迁移的裁决方法**，并汇总现有项目清单；不替换各项目 PRD、组件工程文档和运行时状态总账。
- 若实际代码、锁文件、现行活跃分支与本清单冲突，先核实 exact SHA，再更新此文件；禁止靠 README 自述推导生产现状。

## 二、选型默认值及触发条件

| 层/场景 | 默认选择 | 必须满足的门禁 | 允许改变的触发条件 |
|---|---|---|---|
| Agent OS / H5 / 用户工作台 / JS 生态后端 | 保留 TypeScript + 当前 Node/Bun/Nest/Vite/Next 组合 | TypeScript 严格检查覆盖实际入口；运行时 Zod/JSON Schema；消费者/提供者合同测试 | 同等功能复测证实新实现可明显降低故障、成本或运行时复杂度 |
| 数据采集、量化、LLM 数据处理、文件/专利生成 | 保留 Python + Pydantic + pytest；逐模块增加 pyright/mypy | 输入校验、版本范围及锁定、异常路径、真实文件金标、结果溯源 | 发现明确计算瓶颈，并排除算法、I/O、缓存优化 |
| 独立高吞吐网络服务/无状态 Worker | Go 作为候选 | 原型与现有 TS/Python 实现并跑，测试、故障注入、基准及部署回滚齐全 | 对网络负载、内存、冷启、成本有已量化收益 |
| 原生桌面、低层计算、安全敏感核心 | 当前 Tauri/Rust、Go/Wails 等保持所属项目既有组合 | FFI/进程边界、安装包、主机兼容、升级回退 | 平台 SDK 约束或真实性能/内存安全需求 |
| 单机应用/本地 Runtime DB | SQLite/WAL 可作为长期方案 | 事务、外键、唯一约束、迁移、备份恢复、写入争用监测 | 多宿主并发写、频繁 `SQLITE_BUSY`、复杂共享事务或托管 HA 有实测需求 |
| 多实例共享事务/关系查询 | PostgreSQL 优先评估 | schema 迁移、双端核数、隔离/锁语义、权限、备份回滚 | 现有 SQLite 已无法满足可验证的业务 SLO |
| 缓存/异步队列 | 先用现有 outbox、数据库任务表、Workflow；已有 BullMQ 保留 | 幂等、去重、重试、DLQ、fencing/lease、TTL、容量和清理 | 明确跨实例队列/延时任务/高频缓存需要 Redis |
| 前后端契约 | HTTP 用 OpenAPI；事件、MCP、CLI、WebSocket 各用适合协议的机读 Schema | 代码生成可选；双端合同、错误码、鉴权、版本兼容真实验收 | 不允许仅上传规范就宣称前后端一致 |
| 交付 | Linux 服务用 OCI/Docker；Windows 宿主、Electron、Tauri、微信小程序按真实平台验收 | 锁版本、镜像 digest、配置、数据卷、架构、健康门、回滚和真实设备验证 | 不把 Docker 当作环境 100% 相同的凭据 |
| 检索/向量 | 复用 Memory OS 既有检索；pgvector 为按需启用的 PostgreSQL 扩展 | 召回率、延迟、索引运维、证据引用、权限隔离测试 | 有明确的检索指标改善，禁止先装后找场景 |

**限制与反例**：Go 编译通过仍可能有授权、并发、业务逻辑缺陷；Rust 所有权检查也不能证明业务正确；Python 动态类型可用 Pydantic、类型检查与合同测试获得可靠闭环；SQLite → PostgreSQL 迁移包含方言、时间、UPSERT、锁、迁移脚本与驱动差异，必须演练。

## 三、AI 开发质量规则

### 3.1 单模块最小上下文

- AI 接任务先读取当前组件工程文档、模块章节、目标代码、现有测试和所涉契约。
- 只传必要文件与源码段，避免整仓投喂；记录输入 Token、跨组件读取数与首次验收结果。
- 采用本仓既有唯一实现路径；发现两个持久化 Owner、并行 Agent Runtime、两个状态机时先做归属审查。
- 同一个业务事实只能由一个 Owner 写入。不同语言和进程通过稳定 ID、版本化 Schema、错误枚举及事件信封交接。

### 3.2 静态检查与真实运行分门

`G0 当前 Source Truth → G1 类型/格式/安全/Schema → G2 单元/金标/负例 → G3 契约/集成/迁移 → G4 Runtime/宿主/E2E → G5 Review/Receipt/Exact SHA`

- `tsc` 需要覆盖全部正式入口；若某个前端被根 `tsconfig` 排除，必须具有独立类型门且故障注入能使门变红。
- Python 项目为关键 Pydantic 模型、数据库映射、CLI/API 加类型检查与测试；不能只以 `compileall` 作为通过。
- 不将旧 SHA 的 PASS 自动继承给新 HEAD；不把 `HTTP 200`、容器启动、连接器安装、单一 Green Badge 当作闭环。
- 真实宿主必须验证：ENO/桌面使用 Windows/隧道/本机路径；Linux/云端服务检验其真实部署环境。
- 失败分为 `CODE_FAIL / CONTRACT_FAIL / ENV_FAIL / TOOL_UNAVAILABLE / EVIDENCE_INCOMPLETE`，禁止将环境故障冒充业务失败或忽略。

### 3.3 语言/数据库迁移 Gate

1. **测缺口**：事故、延迟 P95/P99、内存、CPU、锁等待、恢复时间、Token 与维护成本。
2. **定义候选**：同一组件同一业务输入，保留现有实现作为基线；新栈用独立 Adapter/Shadow 实施。
3. **可比实验**：至少 3 个代表性任务，固定模型、上下文、输入、运行环境、功能覆盖与负例。
4. **决策**：收益可重复、缺陷不增加、数据核数一致且回滚演练通过，才可进入逐模块切换。可先以单一关键指标改善 20% 作为试验门槛，阈值须由组件文档冻结；不得把 20% 写成未经测量的事实。
5. **退出旧实现**：只有新链 Runtime Gate + 完整 Receipt + 远端 Exact SHA 全部通过，才允许停止旧链；不得先删后补。

## 四、现有项目技术选型清单（2026-10-16 可得证据）

**证据级别**：`A`=直接读到依赖/构建配置；`B`=仓库 README/工程文档自述；`U`=本轮缺必要来源，待本机核验。当前建议均为选型治理建议，无生产运行验收。

| 项目 / 仓库 | 已核技术栈与关键技术 | 本轮裁决 | 优先核查项 | 证据 |
|---|---|---|---|---|
| **Agent OS** / `AWKN-Lab/agent-os` | TypeScript、Node、pnpm、Vitest；DSH Runtime、Grant、Receipt | **KEEP** | 活跃组件合同、门禁 SHA、源码边界及本机 Runtime | A/B |
| **Forge 枢密院** / `AWKN-Lab/Forge` | TypeScript (`strict`)、Node、Vitest；Fusion/Owner Twin/Council/Workbench | **KEEP** | 根 `tsconfig` 排除 `frontend`；需前台独立 typecheck/E2E | A |
| **PRI 关系经营** / `AWKN-Lab/Pri` | TypeScript、Bun/Node、Vite、Electron、Vitest、Biome、Workbench migrations | **KEEP** | PRI 业务域与 Forge 前台的 Authority 漂移；旧工程命名 | A/B |
| **MONT / Mr.Mont** / `AWKN-Lab/Mr.Mont` | React/Vite/TS、NestJS/Prisma、BullMQ；Redis 实际运行态待核 | **KEEP** | 算法金标、队列恢复；当前后端脚本允许 1636 条 ESLint 警告 | A |
| **Memory OS** / `AWKN-Lab/AWKN-Memory-OS` | Python、FastAPI、SQLite/WAL、Migration、outbox/fencing、检索 | **KEEP** | SQLite 并发实测与迁移回滚；不先行改 PostgreSQL | B |
| **AWKN MCP / ENO** / `AWKN-Lab/awkn-mcp` | Node 控制面/Gateway；Python Memory MCP；Registry/Health/Lease | **KEEP** | 实际连接可用性、合约失败、Windows 宿主和单一 Authority | A/B |
| **GOPAI / 全球派** / `GOPAI-team/GOPAI` | Python/FastAPI/SQLite pipeline；Node agent；Next/React；JSON Bridge | **KEEP** | 双语言契约、持久化单写、幂等重放、跨进程失败恢复 | A/B |
| **Patent Agent** / `AWKN-Lab/patent-agent` | Python ≥3.11、Pydantic、DOCX/PDF/PPTX/XML 处理、pytest | **KEEP** | Source Lock、人工确认、文档金标、Runtime Receipt | A |
| **VALUE 投资猎手** / `AWKN-Lab/Value` | Python/FastAPI/pandas/SQLite；React/Vite；Go/Wails 桌面子项目（README） | **KEEP** | 数据源真值、行情时序、结果账本；Go 子项目当前源码核验 | A/B |
| **WIN / 程掌柜** / `AWKN-Lab/win` | Node/Express/SQLite、Vitest；Python/Tesseract 文档解析 | **KEEP** | 多租户数据、文档来源、OCR 失败回执、任务恢复 | A/B |
| **Annie 健身** / `AWKN-Lab/annie` | 微信小程序、后端 JS 业务模块、Presence Kernel（README） | **KEEP / VERIFY** | 当前有效分支、微信授权/支付/设备合同 | B |
| **AWKN UI** / `AWKN-Lab/UI` | pnpm monorepo；TypeScript/React，Design Tokens/Adapters | **KEEP** | 单一组件库、主题、各产品适配、UI E2E | A/B |
| **Marketing** / `AWKN-Lab/marketing` | Next.js 16、React 19、TypeScript | **KEEP** | Typecheck、权限链、业务结果与回执 | A |
| **Moli / 莫式心法** / `AWKN-Lab/moli` | Node workspace、TypeScript 工程包、Skill/评测 | **KEEP** | 工程模块验收与技能增量评估 | A/B |
| **Learning Agent** / `AWKN-Lab/Learning-Agent` | H5 + FastAPI Boundary（README） | **KEEP / VERIFY** | 当前实际运行模块、前后端合同、学习金标与状态 | B |
| **GUNDAM 桌宠** / `AWKN-Lab/GUNDAM` | Tauri 桌面壳、Node 工具/测试、sidecar（README/包配置） | **KEEP / VERIFY** | 原生构建、桌面运行时、多端状态同步、授权与回滚 | A/B |
| **天枢 / Agent Engine** / `AWKN-Lab/tianshu` | Node Runtime（README） | **KEEP / OWNER_REVIEW** | 与 Agent OS 根 Runtime 归属；现行版本代码对标 | B |
| **酒店公开 Demo** / `AWKN-Lab/hotel-ai` | 浏览器赛事演示（README）；具体构建依赖未核 | **KEEP DEMO** | 禁接生产数据；不重复 WIN 正式产品技术真源 | B |
| **天火 PCEC** / `AWKN-Lab/tianhuo` | 演化规则/能力树/日志仓，运行语言未核 | **DOC ASSET** | 不把治理资产当独立生产服务 | B |
| **life-choice** / `AWKN-Lab/life-choice` | Vite、TypeScript、UI 组件依赖 | **KEEP / VERIFY** | 是否正式产品、与 UI 母仓的复用边界 | A |
| **GPT-MCP** / `AWKN-Lab/GPT-MCP` | README 信息不足；AWKN MCP 标记历史迁入来源 | **HOLD** | 是否历史供体，防止双 Gateway 维护 | B/U |

### 附：尚未证实当前有效技术栈的仓库

- `AWKN-Lab/CosmoSpark`（创意宇宙）：本轮常用入口文件未读到；应在本机读取真实源码与 LAY 项目注册表。
- `AWKN-Lab/skills`：技能资产仓，以 SKILL 合同、触发器、评测/回放为选型对象；不强制指定后端语言。
- `AWKN-Lab/awkn-Feel`：内容处理项目，README 可读，未核当前构建配置。
- `AWKN-Lab/AWKN-Lab`：品牌/组织介绍仓，缺产品技术选型必要性。
- 其余数据、展示或历史仓，如 `CosmoSpark-data`、`TUT-data`、`awkn-life-data`、`SOLO-Skill-`、`solo-skill-booster`、`subtitle`、`game`、`GUNDAM` 关联资产等：**先判定是否独立运行项目，再纳入清单；不以仓库数量代替活跃产品数量**。

## 五、跨项目 Authority / 禁止重建项

| 真值/能力 | 建议唯一 Owner | 禁止动作 |
|---|---|---|
| 智能体执行 / Skill / Grant / Action / Receipt | Agent OS | 产品另起第二套 Agent Runtime / Memory Runtime |
| 人脉、商机、BusinessCase、业务 Action/Outcome | PRI | Forge 或 MCP 另写同一业务表 |
| 枢密院前台与决策融合/Workbench | Forge | 用历史 PRI “枢密院”文档覆盖现行产品裁决 |
| 命理/术数专业计算及证据 | MONT | 由 LLM 输出覆盖确定性算法真值 |
| 长期记忆、Evidence、Claim、Lineage 与记忆演化 | Memory OS | 另一服务保存第二份长期记忆 Authority |
| 连接器、工具、数据来源、健康与传输 | AWKN MCP | `HTTP 200` 自动晋升业务能力 PASS |
| 投资行情/结果、专利文件、酒店数据、营销任务等 | 各业务域仓自身 | 因技术栈合并而迁走业务 Owner |

## 六、维护方法与执行清单

### 新项目立项（最少记录 8 项）

`repo / component_owner / runtime_language / framework / storage / queue_cache / deployment_target / gate_and_rollback`。

每项必须附：`source_ref`、配置文件路径、验证日期、Source SHA/工作树指纹；缺项标 `UNKNOWN`。对已选技术的批量升级记录 ADR 与回退入口。

### 存量项目迁移及整改优先级

| 顺序 | 现有组件优先事项 | 通过标志 |
|---|---|---|
| **P0** | Forge/PRI 身份及组件 Owner 核查；Forge 前台类型检查；AWKN MCP/ENO 可用性；GOPAI Python/Node 数据桥 | 统一 Authority、前台错误注入红门、真实连接器读回、双语言契约及重放通过 |
| **P1** | Memory OS SQLite 压测；MONT lint/队列失败注入；Patent/VALUE 金标和依赖锁 | 本机 exact-SHA 证据、原始负例、清晰回滚方案 |
| **P2** | 一个孤立 Go Worker 与现有 TS/Python 同功能对照实验 | 可重复的 Token、可靠性、部署与性能实验；无收益即放弃 |
| **P3** | 将被实测证明有效的编码规则固化到工程技能；删除不改变模型行为的规则 | 弱模型回放成功率提升，原始证据可追溯 |

### 维护动作

1. 优先校验 `D:\awkn-lab` 中是否已有同职责技术选型母文档；若有，按锚点合并内容，避免第二权威来源。
2. 读取各项目**当前活跃分支/本机工作树**的依赖与状态；逐行核实本表，记录 `verified_ref` 与 `verified_at`，分清远端与本地。
3. 确认本地根目录写入并重新读取，校验 SHA-256。此后文档状态可从 `PROPOSED` 改为 `ACTIVE`。
4. 任何组件完成技术调整，更新相应工程文档章节、门禁与回执，然后更新本索引。未收到真实 Gate 时保留 `PENDING_RUNTIME`。
5. 不因本索引存在而触发安装、生产部署、数据库迁移或代码重写。生产写入和发布仍按各 Owner 授权规则执行。

---

**参考证据入口**：
[Agent OS](https://github.com/AWKN-Lab/agent-os) · [Forge](https://github.com/AWKN-Lab/Forge) · [PRI](https://github.com/AWKN-Lab/Pri) · [Memory OS](https://github.com/AWKN-Lab/AWKN-Memory-OS) · [AWKN MCP](https://github.com/AWKN-Lab/awkn-mcp) · [GOPAI](https://github.com/GOPAI-team/GOPAI) · [Patent Agent](https://github.com/AWKN-Lab/patent-agent) · [MONT](https://github.com/AWKN-Lab/Mr.Mont) · [VALUE](https://github.com/AWKN-Lab/Value) · [WIN](https://github.com/AWKN-Lab/win) · [AWKN UI](https://github.com/AWKN-Lab/UI)。