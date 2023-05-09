# Fow Of Code流程编排定义语言 - Version 0.1



## 目录

- [基本介绍](#基本介绍)
- [流程定义结构](#流程定义结构)



## 基本介绍

foc-workflow-spec(Fow Of Code Workflow Specification)

流程编排定义语言用于描述和定义业务逻辑



## 流程定义结构

| 属性名称          | 类型   | 描述                                                                                         | 是否必填 | 默认值 |
| ----------------- | ------ | -------------------------------------------------------------------------------------------- | -------- | ------ |
| id              | string | 流程定义id                                                                            | 是       ||
| version         | string | 流程定义版本                                                                      | 是       |    |
| name      | string | 流程定义名称                                     | 是       |    |
| description   | string | 流程定义描述                                                                             | 否       ||
| specVersion | string | 使用的规范版本 | 是 ||
| start | string | 开始节点 | 是 ||
| expressionLang | string | 定义表达式语言 | 否       |jq|
| nodes | array | 流程节点 | 是 ||
| metadata | object | 元数据 | 否 ||
