# 宠物记录微信小程序开发方案（MVP）

## 1. 产品定位
面向养宠用户，提供「宠物档案 + 日常记录 + 关键事件提醒」的一体化管理工具，核心目标是：
- 快速登记宠物信息（昵称、品种、生日、头像等）
- 便捷记录日常（饮食、体重、洗澡、美容、驱虫、就医）
- 准时提醒关键事项（生日、疫苗、理发/美容、驱虫、复查）

## 2. MVP 功能范围

### 2.1 宠物管理
- 新增宠物：
  - 基础字段：昵称、品种、性别、生日、领养/到家日期、头像
  - 可选字段：体重、绝育状态、过敏史、备注
- 宠物列表：卡片形式展示头像、昵称、年龄、下一个提醒
- 宠物详情：展示档案信息、最近记录、提醒概览

### 2.2 日常记录
- 支持记录类型：
  - 喂食
  - 体重
  - 洗澡
  - 理发/美容
  - 疫苗
  - 驱虫
  - 就医
  - 自定义
- 每条记录字段：记录时间、类型、内容、图片（可选）、备注（可选）
- 记录页能力：
  - 按宠物筛选
  - 按类型筛选
  - 时间倒序展示

### 2.3 提醒功能
- 提醒类型：
  - 生日提醒（每年重复）
  - 疫苗提醒（按针次/下一针时间）
  - 理发/美容提醒（按周期）
  - 驱虫提醒（按周期）
  - 自定义提醒
- 提醒规则：
  - 支持提前 N 天提醒（默认 3 天）
  - 支持重复策略（不重复 / 每周 / 每月 / 每年）
- 通知形式：
  - 小程序订阅消息（主）
  - 站内提醒列表（兜底）

## 3. 页面结构建议
- `pages/home/index`：首页（今日待办、最近记录、快捷入口）
- `pages/pet/list`：宠物列表
- `pages/pet/edit`：新增/编辑宠物
- `pages/pet/detail`：宠物详情
- `pages/record/list`：记录列表
- `pages/record/edit`：新增记录
- `pages/reminder/list`：提醒列表
- `pages/reminder/edit`：新增/编辑提醒
- `pages/profile/index`：个人中心（设置、订阅消息授权）

## 4. 数据模型（建议）

### 4.1 `pets`
- `id`：string
- `ownerOpenId`：string
- `name`：string
- `species`：string（猫/狗/其他）
- `breed`：string
- `gender`：string
- `birthday`：date
- `avatar`：string
- `weight`：number
- `adoptDate`：date
- `notes`：string
- `createdAt` / `updatedAt`

### 4.2 `records`
- `id`：string
- `ownerOpenId`：string
- `petId`：string
- `type`：string
- `recordTime`：datetime
- `content`：string
- `images`：string[]
- `nextDueDate`：date（如疫苗/驱虫可选）
- `createdAt` / `updatedAt`

### 4.3 `reminders`
- `id`：string
- `ownerOpenId`：string
- `petId`：string
- `title`：string
- `type`：string
- `dueDate`：date
- `advanceDays`：number
- `repeatRule`：string
- `enabled`：boolean
- `lastNotifiedAt`：datetime
- `createdAt` / `updatedAt`

## 5. 技术实现建议

### 5.1 前端（微信小程序）
- 原生小程序 + TypeScript（推荐）
- 状态管理：轻量 store（如 MobX/miniprogram-store）
- UI：统一设计组件（卡片、时间线、提醒标签）

### 5.2 后端与存储
优先使用微信云开发（CloudBase）快速上线：
- 云数据库：`pets / records / reminders`
- 云函数：
  - `createReminderFromRecord`（如疫苗记录自动生成下次提醒）
  - `dailyReminderScheduler`（每天扫描待提醒任务）
  - `sendSubscribeMessage`（调用订阅消息）
- 云存储：记录图片

## 6. 提醒触发流程（简化）
1. 用户创建或更新提醒
2. 云函数按 `dueDate - advanceDays` 计算触发时间
3. 定时任务每日 08:00 扫描当日应提醒数据
4. 检查 `enabled=true` 且未发送
5. 推送订阅消息并记录 `lastNotifiedAt`
6. 如有重复规则，自动计算下一次 `dueDate`

## 7. 里程碑计划（4 周）
- 第 1 周：需求细化 + 页面原型 + 数据库结构
- 第 2 周：宠物管理 + 记录管理（核心 CRUD）
- 第 3 周：提醒系统 + 订阅消息 + 定时任务
- 第 4 周：体验优化 + 测试 + 上线准备

## 8. 后续可扩展方向
- 健康趋势图（体重、就医频次）
- 多宠家庭共享（家庭成员协同）
- 宠物医院/门店服务对接
- AI 文案辅助（自动生成日常记录摘要）

## 9. 最小可交付版本（MVP 验收标准）
- 能新增至少 1 只宠物并编辑资料
- 能添加至少 3 种日常记录
- 能创建生日与疫苗提醒，并在到期前收到提醒
- 能在提醒列表中查看历史提醒状态
