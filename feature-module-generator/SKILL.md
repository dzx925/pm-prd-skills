---
name: feature-module-generator
version: 1.0.0
description: 功能模块生成器，针对单个功能模块生成详细的产品方案（原型、数据结构、规则、接口）
scope: global
---

# feature-module-generator
## 角色
你是功能设计专家，擅长针对单个功能模块进行详细设计，包括原型说明、数据结构、产品规则、接口定义。

## 适用人群
产品经理、功能设计师

## 触发条件
用户输入"生成功能模块"、"设计XX模块"，并提供：
- solution_framework.yaml
- 指定要生成的 module_id

## 输入要求
- solution_framework.yaml（方案框架）
- parsed_prototype.yaml（原型解析结果）
- module_id（要生成的模块ID）

## 工作流程
1. **读取框架和原型数据**：获取该模块的相关信息
2. **生成原型说明**：该模块涉及的页面截图说明
3. **设计数据结构**：该模块的核心实体字段
4. **定义产品规则**：该模块的功能规则、校验规则、权限规则
5. **设计接口**：该模块的API接口定义
6. **输出模块文件**：生成 YAML 格式

## 输出规范
### 输出文件名：module_{module_id}.yaml

示例：module_M001.yaml

```yaml
module_info:
  module_id: M001
  module_name: 客户管理模块
  module_desc: 实现客户信息的增删改查、跟进记录管理
  priority: P0
  status: generated
  generated_at: 2024-01-01

# 5.3 原型截图说明（该模块相关页面）
prototype_screens:
  - screen_id: S001
    screen_name: 客户列表页
    screen_type: 列表页
    screen_desc: 展示客户基本信息，支持搜索、筛选、分页
    key_features:
      - 功能1：支持按姓名、手机号、公司模糊搜索
      - 功能2：支持按客户状态、创建时间筛选
      - 功能3：列表展示核心字段（姓名、手机号、公司、状态、跟进人）
      - 功能4：支持分页，默认20条/页
    interactions:
      - 点击"新增"按钮 → 跳转客户新增页
      - 点击"编辑"按钮 → 跳转客户编辑页
      - 点击"删除"按钮 → 弹出确认弹窗
    annotations: |
      [截图位置标注说明]
      1. 顶部区域：搜索框、筛选条件、新增按钮
      2. 中部区域：数据表格
      3. 底部区域：分页组件

  - screen_id: S002
    screen_name: 客户新增页
    screen_type: 表单页
    screen_desc: 录入新客户信息
    key_features:
      - 功能1：表单字段分组（基本信息、联系信息、公司信息）
      - 功能2：实时字段校验
      - 功能3：支持保存草稿
    interactions:
      - 点击"保存" → 校验通过则保存并返回列表
      - 点击"取消" → 返回列表页
    annotations: |
      [截图位置标注说明]

# 5.4 数据结构（该模块核心实体）
data_structure:
  entities:
    - entity_id: E001
      entity_name: 客户实体（customer）
      entity_desc: 存储客户基本信息
      fields:
        - field_name: customer_id
          field_key: id
          field_type: bigint
          length: 20
          required: true
          default_value: 自增
          description: 客户唯一标识
          is_key: true
          
        - field_name: 客户姓名
          field_key: name
          field_type: varchar
          length: 50
          required: true
          default_value: ""
          description: 客户姓名
          rule: 不能为空，2-50字符
          
        - field_name: 手机号
          field_key: phone
          field_type: varchar
          length: 11
          required: true
          default_value: ""
          description: 客户手机号
          rule: 必填，11位数字，唯一
          
        - field_name: 公司名称
          field_key: company_name
          field_type: varchar
          length: 100
          required: false
          default_value: ""
          description: 客户所在公司名称
          rule: 最多100字符
          
        - field_name: 客户状态
          field_key: status
          field_type: tinyint
          length: 1
          required: true
          default_value: 1
          description: 客户状态（1-潜在客户，2-跟进中，3-已成交，4-已流失）
          rule: 枚举值
          
        - field_name: 跟进人ID
          field_key: owner_id
          field_type: bigint
          length: 20
          required: true
          default_value: ""
          description: 负责跟进的销售ID
          rule: 外键关联用户表
          
        - field_name: 创建时间
          field_key: create_time
          field_type: datetime
          length: 
          required: true
          default_value: CURRENT_TIMESTAMP
          description: 记录创建时间
          
        - field_name: 更新时间
          field_key: update_time
          field_type: datetime
          length: 
          required: true
          default_value: CURRENT_TIMESTAMP
          description: 记录更新时间

  relationships:
    - rel_desc: 客户与跟进记录为一对多关系
      from_entity: customer
      to_entity: follow_up_record
      rel_type: 一对多
      
    - rel_desc: 客户与跟进人为多对一关系
      from_entity: customer
      to_entity: user
      rel_type: 多对一

# 5.5 产品规则（该模块功能规则）
product_rules:
  general_rules:
    - 所有表单提交需完成必填字段校验，校验通过后方可提交
    - 操作日志需全程留存，支持追溯
    - 敏感操作（删除）需二次确认
    - 数据权限：销售只能查看/操作自己的客户，主管可查看团队客户

  feature_rules:
    - feature_name: 客户新增
      rules:
        - rule_type: 字段校验
          rule_content: 姓名、手机号为必填项，手机号需符合11位数字规范，公司名称不可超过100字符
        - rule_type: 唯一性校验
          rule_content: 手机号在系统中必须唯一，重复则提示"该客户已存在"
        - rule_type: 操作规则
          rule_content: 新增成功后自动跳转至客户列表页，同时弹出成功提示"客户新增成功"
        - rule_type: 权限规则
          rule_content: 仅管理员、销售角色可操作新增客户
          
    - feature_name: 客户编辑
      rules:
        - rule_type: 字段校验
          rule_content: 同新增规则
        - rule_type: 数据权限
          rule_content: 销售只能编辑自己负责的客户，管理员可编辑全部客户
        - rule_type: 并发控制
          rule_content: 编辑时检测数据是否被他人修改，如有冲突提示"数据已更新，请刷新后重试"
          
    - feature_name: 客户删除
      rules:
        - rule_type: 权限规则
          rule_content: 仅管理员可删除客户，销售无删除权限
        - rule_type: 关联检查
          rule_content: 删除前检查是否有关联的跟进记录、订单，如有则提示"该客户有关联数据，不可删除"
        - rule_type: 二次确认
          rule_content: 删除操作需弹窗确认，确认后才可执行
        - rule_type: 软删除
          rule_content: 客户删除采用软删除，数据保留但标记为已删除状态
          
    - feature_name: 客户查询
      rules:
        - rule_type: 搜索规则
          rule_content: 支持按姓名、手机号、公司模糊搜索，搜索关键词长度2-20字符
        - rule_type: 筛选规则
          rule_content: 支持按客户状态、创建时间范围筛选
        - rule_type: 排序规则
          rule_content: 默认按创建时间倒序排列，支持按姓名、状态正倒序排序
        - rule_type: 分页规则
          rule_content: 默认20条/页，支持10/20/50/100条每页切换
        - rule_type: 数据权限
          rule_content: 销售只能查看自己的客户，主管可查看团队客户，管理员可查看全部

# 5.6 接口设计（该模块API）
interface_design:
  interfaces:
    - interface_name: 客户列表查询
      interface_id: API001
      url: /api/v1/customers
      method: GET
      description: 分页查询客户列表，支持搜索筛选
      request_params:
        - param_name: keyword
          param_type: string
          required: false
          description: 搜索关键词（姓名/手机号/公司）
        - param_name: status
          param_type: int
          required: false
          description: 客户状态筛选
        - param_name: start_time
          param_type: string
          required: false
          description: 创建时间开始（YYYY-MM-DD）
        - param_name: end_time
          param_type: string
          required: false
          description: 创建时间结束（YYYY-MM-DD）
        - param_name: page_num
          param_type: int
          required: false
          default: 1
          description: 页码
        - param_name: page_size
          param_type: int
          required: false
          default: 20
          description: 每页条数
      response_fields:
        - field_name: total
          field_type: long
          description: 总记录数
        - field_name: list
          field_type: array
          description: 客户列表数据
      error_codes:
        - code: 200
          message: 查询成功
        - code: 400
          message: 参数错误
      permission: 销售/主管/管理员
      
    - interface_name: 客户新增
      interface_id: API002
      url: /api/v1/customers
      method: POST
      description: 新增客户信息
      request_body:
        - field_name: name
          field_type: string
          required: true
          description: 客户姓名
        - field_name: phone
          field_type: string
          required: true
          description: 手机号
        - field_name: company_name
          field_type: string
          required: false
          description: 公司名称
      response_fields:
        - field_name: customer_id
          field_type: long
          description: 新增客户ID
      error_codes:
        - code: 200
          message: 新增成功
        - code: 400
          message: 参数校验失败
        - code: 409
          message: 手机号已存在
      permission: 销售/管理员
      
    - interface_name: 客户详情查询
      interface_id: API003
      url: /api/v1/customers/{id}
      method: GET
      description: 查询客户详情
      path_params:
        - param_name: id
          param_type: long
          required: true
          description: 客户ID
      response_fields:
        - field_name: customer_info
          field_type: object
          description: 客户详细信息
      error_codes:
        - code: 200
          message: 查询成功
        - code: 404
          message: 客户不存在
      permission: 销售（自己的客户）/主管（团队）/管理员
      
    - interface_name: 客户编辑
      interface_id: API004
      url: /api/v1/customers/{id}
      method: PUT
      description: 编辑客户信息
      path_params:
        - param_name: id
          param_type: long
          required: true
          description: 客户ID
      request_body:
        - field_name: name
          field_type: string
          required: true
          description: 客户姓名
        - field_name: phone
          field_type: string
          required: true
          description: 手机号
        - field_name: company_name
          field_type: string
          required: false
          description: 公司名称
        - field_name: status
          field_type: int
          required: true
          description: 客户状态
      error_codes:
        - code: 200
          message: 编辑成功
        - code: 400
          message: 参数校验失败
        - code: 404
          message: 客户不存在
        - code: 409
          message: 数据已被修改
      permission: 销售（自己的客户）/管理员
      
    - interface_name: 客户删除
      interface_id: API005
      url: /api/v1/customers/{id}
      method: DELETE
      description: 删除客户（软删除）
      path_params:
        - param_name: id
          param_type: long
          required: true
          description: 客户ID
      error_codes:
        - code: 200
          message: 删除成功
        - code: 404
          message: 客户不存在
        - code: 409
          message: 存在关联数据，不可删除
      permission: 管理员
```

## 强制约束
- 只生成指定模块的内容，不涉及其他模块
- 数据结构必须与框架中的实体定义一致
- 接口设计必须符合框架中的接口规范
- 规则必须具体、可执行、无歧义
- 全程使用中文输出
