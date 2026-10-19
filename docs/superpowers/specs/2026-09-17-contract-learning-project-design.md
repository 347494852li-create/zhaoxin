# 合同管理学习项目（zxcm-learn）设计文档

> 日期：2026-09-17
> 状态：设计已确认，待写入实施计划
> 答案册：`E:\Develop\project\zhaoxin\zhaoxin-contract`（开源项目，不修改）
> 目标项目：`E:\Develop\project\zxcm-learn`（新建）

---

## 一、背景与目标

### 用户现状

- 已具备基础 Java 技术栈：Spring Boot、Redis、MySQL、MyBatis-Plus、阿里云技术栈
- 无企业级项目实战经验，需要把一个真实商业项目（肇新合同管理系统）扎实掌握到「能从 0 手敲」的程度
- 中间件（MySQL 8 / Redis）部署在 Linux 虚拟机中，不需要从 0 搭建，也不在本项目的实施范围内

### 目标

产出一个**用户亲手写的合同全生命周期管理系统**，使其能够：

1. 从零独立敲出 M1-M4 全部业务代码，而不是照抄
2. 讲清每条技术链路的底层原理，包括 Sai-Token 鉴权、动态菜单、数据权限
3. 在求职答辩中，对架构决策、技术难点、设计取舍给出有说服力的解释

### 成功标准

用户能对该项目的任意一个功能，回答清楚三个问题：

- 这个功能的数据从哪张表来、经过哪些类、最后怎么渲染
- 为什么这样设计，替代方案是什么，为什么不选
- 如果需求变了（加字段、加状态、加审批节点），要改哪几处

---

## 二、已确认的前置决策

以下六条在需求澄清阶段逐一确认，后续设计与实施均以此为准。

| # | 决策项 | 结论 |
|---|---|---|
| 1 | 完成线 | **M1-M4 完整业务闭环**（合同 CRUD + 审批流 + 模板拼装 + 用印/归档/借阅/变更） |
| 2 | 底座策略 | **拿现有项目做底座，清空业务代码**。保留 `ruoyi-admin` + `ruoyi-base`，清空 `zxcm-*` 业务模块的 `src` |
| 3 | 前端范围 | **全部自己写**（含登录页、布局、路由守卫、动态菜单、权限指令、axios 封装、业务页面） |
| 4 | 技术栈版本 | **与原项目完全一致**（Spring Boot 2.6.4 / Java 11 / MyBatis-Plus 3.5.2 / Sa-Token / Activiti 6 / Vue 2 + Element UI） |
| 5 | 协作方式 | **混合模式**：骨架与配置由 Claude 搭建，业务代码由用户自己敲，难点先给可运行最小示例再让用户重敲。业务代码**允许用户按文件自行决定手敲或从答案册复制** |
| 6 | 推进方式 | **A 垂直切片**：每个里程碑都是一条从建表到浏览器演示的端到端链路 |

### 补充需求

用户额外提出对**后端底座底层原理**（Sa-Token 鉴权、动态菜单、数据权限）有深入兴趣。这部分不通过重写底座实现（会多耗 2-3 个月且质量不如成熟框架），而是以**逆向学习**方式处理：打断点走通链路、理解设计、改一处做验证。详见第 4.4 节 D1-D3。

---

## 三、项目定位与产出物

### 定位

一个用户亲手编写的、覆盖合同全生命周期的企业级管理系统。技术上是**开源框架二次开发**——这与原团队的真实做法一致，也是企业中的普遍形态，在答辩中无需回避。

### 产出物

| 产出物 | 内容 | 存放位置 |
|---|---|---|
| **项目本体** | `zxcm-learn`，可构建、可启动、可演示 | `E:\Develop\project\zxcm-learn` |
| **施工图** | 每个垂直切片的「要敲哪些文件 + 每个文件干什么 + 关键代码讲解」 | `E:\Develop\project\zhaoxin\learning\` |
| **答辩手册** | 架构决策、技术难点、设计取舍、常见提问与应对 | 同上，M4 完成后产出 |

### 三者的关系

```
施工图   →  用户照着敲  →  zxcm-learn（项目本体）
   ↑                              ↓
   └── 卡住时查答案册 zhaoxin-contract ┘
   ↓
答辩手册（把「敲过的东西」翻译成「能讲出来的话」）
```

---

## 四、详细设计

### 4.1 目录结构与底座裁剪

复制 `zhaoxin-contract` 到 `E:\Develop\project\zxcm-learn`，然后按模块裁剪。

| 模块 | 处理方式 | 说明 |
|---|---|---|
| `ruoyi-admin` | **保留** | 启动入口、系统管理 Controller（21 个）、全部配置文件 |
| `ruoyi-base/ruoyi-common` | **保留** | 通用工具、注解、异常、基础类 |
| `ruoyi-base/ruoyi-framework` | **保留** | Sa-Token、Redis、MyBatis-Plus、AOP 等框架配置 |
| `ruoyi-base/ruoyi-system` | **保留** | 用户/角色/菜单/字典/部门/岗位/租户 |
| `ruoyi-base/ruoyi-generator` | **保留** | 代码生成器，M1 起手要用 |
| `ruoyi-base/ruoyi-tenant` | **保留** | 多租户。已知问题：`pom.xml` 声明 `maven.compiler.source=17`，与根 pom 的 Java 11 冲突，需改为 11 |
| `zxcm-contract` | **清空 `src`，保留 `pom.xml`** | 用户的主战场 |
| `zxcm-workflow` | **清空 `src`，保留 `pom.xml`** | 原计划只删设计器、保留引擎配置，但实测发现其 `pom.xml` 依赖 `zxcm-contract`（已清空），且 9 个源文件引用 `contract`/`conf`/`template` 包，**无法只删一部分**。Activiti 的 Maven 依赖仍在 `pom.xml` 中，M2 时重写引擎配置 |
| `zxcm-notify` | **清空 `src`，保留 `pom.xml`** | 消息通知不在 M1-M4 范围内，先留空壳 |
| `zxcm-form` | **清空 `src`，保留 `pom.xml`** | 动态表单，不在 M1-M4 范围内 |
| `zxcm-third` | **清空 `src`，保留 `pom.xml`** | Excel 导入导出与钉钉同步，不在 M1-M4 范围内 |
| `zxcm-ui` | **只留工程配置，`src` 全部自己写** | 保留 `package.json`、`vue.config.js`、`.env.development`、babel 等配置；删除 `src/` 下全部内容 |

**为什么保留 `pom.xml` 而清空 `src`**：模块结构、依赖版本、构建配置全部现成且正确，用户只需往里填代码。这是最省力且不易出错的起步方式。

### 4.2 数据库策略

`script/sql/zxcm_init.sql` 共 92 张表，按归属分四类，只有一类需要用户设计。

| 表分类 | 数量 | 处理方式 |
|---|---|---|
| `sys_*` 框架表 | 24 | **直接使用，不改动**。含 RuoYi-Vue-Plus 扩展表：`sys_business_module`、`sys_tenant`、`sys_oss`、`sys_user_dept` 等 |
| `gen_*` 代码生成器表 | 2 | **直接使用** |
| `act_*` Activiti 引擎表 | 25 | **一行不写**。引擎启动时自动建表 |
| `xxl_job_*` 调度中心表 | 8 | **跳过**。属于 XXL-JOB Admin 自己的库 |
| **业务表** | **32** | **用户设计**。M1-M4 实际需要约 20 张 |

**业务表前缀分布**（用于规划切片）：`contract_*` 11、`template_*` 7、`conf_*` 6、`notify_*` 4、`sync_*` 2、`work_*` 1、`form` 1。

**用户设计每张表的固定节奏**：

1. Claude 讲清这张表要解决什么业务问题、有哪些关键字段、为什么这么切分
2. 用户自己写出 DDL
3. 对照答案册 `zxcm_init.sql` 中同表的定义，找出差异
4. Claude 解释差异为什么存在（哪个更好、原项目为什么选它）

**底座 SQL 的准备方式**：从 `zxcm_init.sql` 中裁出 `sys_*`(24) + `gen_*`(2) 的建表语句与基础数据（用户、角色、菜单、字典、配置），去掉全部业务表、业务菜单与 `act_*`、`xxl_job_*`，形成底座初始化脚本。

### 4.3 迭代计划

采用垂直切片：每个里程碑完整走通「设计表 → 敲后端 → 敲前端 → 配菜单权限 → 浏览器端到端演示」。

#### M0 · 骨架（Claude 主导搭建）

**后端**
- 裁剪底座模块，清理 `ruoyi-admin` 中的 zxcm 业务配置项
- 生成并导入底座 SQL（`sys_*` + `gen_*`）
- 修改 `application-dev.yml`：数据库与 Redis 指向虚拟机 IP
- 处理三个已知坑：JDK 切到 11、`ruoyi-tenant` 的 Java 17 声明、`application.yml` 的 Maven `@xxx@` 占位符
- 验收：后端启动成功，`/actuator/health` 返回 UP

**前端（用户自己写）**
- Claude 完成 vue-cli 工程初始化与配置（`package.json`、`vue.config.js`、`.env.development`、代理配置）
- 用户依次敲：axios 封装（`utils/request.js`） → 登录页 → 主布局 → 路由守卫 → 动态菜单加载 → 权限指令
- 验收：浏览器打开前端，用 `zxcmAdmin / 123456` 登录成功，左侧菜单正确渲染

#### M1 · 合同 CRUD（第一个垂直切片）

- **表**：`contract_draft`、`contract_draft_files`、`conf_bill_rule`（编号规则）
- **后端**：Controller / Service / ServiceImpl / Mapper / domain-bo-vo 全套
- **前端**：合同列表页、合同表单页、API 层 js、菜单与权限码配置
- **业务逻辑**：合同编号按规则自动生成、负责人自动取当前登录用户、列表默认只查草稿状态、逻辑删除、时间范围查询、编号唯一性校验
- **验收**：能新增/查询/修改/删除合同；数据库中 `create_by`/`create_time` 自动填充；删除后数据仍在表中且 `del_flag` 变更

#### M2 · 审批流与事件状态机

- **表**：`contract_audit_form`、`contract_point_log`
- **工作流**：手写最简单的 BPMN XML（开始 → 审批任务 → 结束）放入 `resources/processes/`，让 Activiti 自动部署。**不碰自研流程设计器**
- **事件状态机**：`ContractEvent` + `ContractEventEnum` + 多个 `@EventListener` 监听器
- **前端**：审批发起页、待办任务列表
- **验收**：提交审批后 `act_ru_task` 出现待办；审批通过后合同状态变更；`contract_point_log` 记录操作日志

#### M3 · 模板体系与文档生成

- **表**：`template_contract`、`template_clause`、`template_clause_type`、`template_element`、`template_scope`
- **文档生成**（渐进三步）：① POI 替换 `${占位符}` → ② 多条款按序拼装 → ③ 要素变量从合同数据自动填充
- **前端**：模板管理、条款库维护、起草时选择模板与条款
- **验收**：选一个模板 + 勾选 3 个条款，能生成一份 `.docx`，文档内容正确

#### M4 · 用印/归档/借阅/变更

- **表**：`contract_seal`、`contract_seal_notes`、`conf_archives`、`contract_his`、`contract_his_files`、`contract_borrow`、`contract_review`
- **核心难点**：`@Compare` 注解 + 反射，自动比对新旧合同对象找出变更字段，用于变更单差异定位
- **前端**：用印申请、归档、借阅、变更单页面
- **验收**：完整生命周期闭环可用

#### 4.4 并行穿插线 D1-D3（底座原理深挖）

不占独立里程碑，融入各切片的间隙执行。每条线的固定做法：**画链路图 → 打断点走一遍 → 改一处做验证**。

| 线 | 主题 | 关键代码位置 | 验证动作 |
|---|---|---|---|
| **D1** | Sa-Token 鉴权链路 | `LoginHelper`、`SaInterfaceImpl`、`PlusSaTokenDao`、`UserActionListener`、`SaTokenConfig`、`application.yml` 的 `sa-token` 配置 | 在 `SaTokenConfig` 拦截器打断点，观察一次请求如何被校验；观察 token 如何存入 Redis |
| **D2** | 动态菜单加载 | `sys_menu` / `sys_role_menu` 查询链路、菜单树构建、前端路由生成 | 在 `sys_menu` 新增一个菜单，观察它如何流到前端并生成路由 |
| **D3** | 数据权限 | `@DataPermission` 注解、`PlusDataPermissionHandler`、`DataPermissionHelper`、`DataScopeType` | 新增一个 `DataScopeType`，观察 SQL 如何被动态改写 |

### 4.5 协作规则

每个垂直切片固定执行十个动作，顺序不跳步。

**Claude 负责**

1. 出「施工图」：本切片要敲的文件清单、每个文件的职责、关键代码的逐行讲解，并**逐文件标注答案册的对应路径**，供用户自行决定是手敲还是复制
2. 搭建骨架：pom、配置、必要的空类结构
3. 难点的最小可运行示例（如 Activiti 最小流程、POI 替换示例、`@Compare` 示例）
4. 出验收清单

**用户负责**

5. 自己敲业务代码。**每个文件自行决定手敲还是从答案册复制**——复制动作由用户自己做，Claude 不代替用户往项目里拷业务代码
6. 跑通并在卡住时反馈（附报错信息）
7. 难点示例读懂后，**自己重敲一遍**

**Claude 收尾**

8. Debug：根据报错定位问题，讲清根因而非直接改代码
9. Code review：对比用户实现与答案册，指出差异与改进点
10. 讲解「原项目为什么这么写」以及替代方案

### 4.6 验收方式

每个里程碑三条硬标准，**三条全过才算完成**：

| # | 标准 | 验证方式 |
|---|---|---|
| 1 | 接口通 | 打开 `http://localhost:8080/doc.html`（Knife4j），手动调通本切片的所有接口 |
| 2 | 功能通 | 浏览器里真实操作一遍完整业务流程 |
| 3 | 数据对 | 直接用 SQL 查表，确认字段值正确（含 `create_by`、`del_flag` 等自动填充字段） |

**不接受「看懂了」作为完成标志。** 只有敲出来并跑通才算。

---

## 五、范围之外（明确不做）

以下内容**不在本项目范围内**，避免范围蔓延：

- 重写 `ruoyi-base` 底座代码（改为逆向学习 D1-D3）
- 自研 Activiti 流程设计器（`zxcm-workflow/engine/designer`、`engine/modeler`）
- OnlyOffice 在线编辑对接
- 钉钉组织架构与用户同步
- 动态表单（`zxcm-form`）
- XXL-JOB 定时任务（`archiveReturnHandler`、`errorMsgScheduleHandler`）
- Excel 导入导出、系统消息通知（属 M5 内容，不在本轮范围内）
- 多租户改造
- 中间件的安装部署（由用户在虚拟机上完成，不属于本项目实施内容）

---

## 六、已知风险与坑

| # | 风险 | 影响 | 应对 |
|---|---|---|---|
| 1 | **前端全部自己写是最大变量**。用户无 Vue 实战经验，而 `zxcm-ui` 原版有 528 个文件，且不是若依原版（登录态、字典、权限、主题均为自研） | 可能显著拖长 M0 和每个切片的前端部分 | 前端拆得更细，每敲一个页面给一次验收；通用组件（文件上传、富文本）优先复用 Element UI 原生能力，不自己造 |
| 2 | 本机 JDK 默认为 15，项目要求 11 | Lombok 版本过旧导致编译直接失败 | M0 第一步即切 JDK 11（`D:\Develop\jdk-11.0.2`） |
| 3 | `ruoyi-tenant/pom.xml` 声明 `maven.compiler.source=17` | 用 JDK 11 编译该模块报错 | M0 中将其改为 11 |
| 4 | `application.yml` 含 Maven 占位符 `@profiles.active@` | 若 IDE 未执行资源过滤，占位符残留导致启动失败 | 启动配置加 VM 参数 `-Dspring.profiles.active=dev,workflow`，或先 `mvn package` |
| 5 | `application-dev.yml` 默认库名为 `opensource`，文档建议 `zxcm` | 不一致会导致「表不存在」 | M0 中统一为 `zxcm` |
| 6 | 原项目存在已知缺陷，如 `ContractEvent` 四参数构造方法漏赋值 `businessJson` | 用户照抄会继承缺陷 | 涉及该处时明确指出，让用户自己判断是否修复 |
| 7 | 时长约 8-12 周，中途放弃风险高 | 无成果产出 | 垂直切片保证每个里程碑都有可演示成果；M1 完成即具备演示价值 |

---

## 七、下一步

1. 用户审阅本设计文档
2. 通过后，调用 writing-plans 技能产出实施计划，将 M0 拆解为可执行的原子任务
3. 从 M0 开始执行
