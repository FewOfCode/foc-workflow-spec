# Fow Of Code流程编排定义语言 - Version 0.1



## 目录

- [Fow Of Code流程编排定义语言 - Version 0.1](#fow-of-code流程编排定义语言---version-01)
  - [目录](#目录)
  - [基本介绍](#基本介绍)
  - [流程定义](#流程定义)
  - [节点定义](#节点定义)
    - [通用属性](#通用属性)
    - [变量注入节点](#变量注入节点)
      - [示例](#示例)
    - [任务节点](#任务节点)
      - [示例](#示例-1)
    - [分支节点](#分支节点)
      - [condition branch](#condition-branch)
      - [示例](#示例-2)
    - [循环节点](#循环节点)
      - [range](#range)
      - [iterator](#iterator)
      - [示例](#示例-3)
    - [并行节点](#并行节点)
      - [parallel branch](#parallel-branch)
      - [示例](#示例-4)
  - [关联对象属性定义](#关联对象属性定义)
    - [command](#command)
      - [function](#function)
        - [示例一](#示例一)
        - [示例二](#示例二)
      - [subflow](#subflow)
        - [示例](#示例-5)
    - [异常策略](#异常策略)
      - [retry](#retry)
        - [示例](#示例-6)
      - [goto](#goto)
        - [errRefs](#errrefs)
        - [示例](#示例-7)
      - [exit](#exit)
        - [示例](#示例-8)




## 基本介绍

foc-workflow-spec(Fow Of Code Workflow Specification)，流程编排定义语言用于描述和定义业务逻辑。



## 流程定义

| 属性名称          | 类型   | 描述                                                                                         | 是否必填 | 默认值 |
| ----------------- | ------ | -------------------------------------------------------------------------------------------- | -------- | ------ |
| id              | string | 流程定义id                                                                            | 是       ||
| version         | int | 流程定义版本                                                                      | 是       |    |
| name      | string | 流程定义名称                                     | 是       |    |
| description   | string | 流程定义描述                                                                             | 否       ||
| specVersion | string | 使用的规范版本 | 是 |0.1|
| start | string | 开始节点 | 是 ||
| expressionLang | string | 定义表达式语言 | 否       |jq|
| variables | object | 流程变量 | 否 ||
| [nodes](#节点定义) | array | 流程节点 | 是 ||
| metadata | object | 元数据 | 否 ||



## 节点定义

节点是指定义流程执行指令的模块，节点定义了工作流在运行过程中应该执行的一组业务逻辑指令。

### 通用属性

通用属性指所有的节点所共有的属性，跟节点的类型无关。具体参考以下表格:

| 属性名称                   | 类型              | 描述                                                         | 是否必填 | 默认值 |
| -------------------------- | ----------------- | ------------------------------------------------------------ | -------- | ------ |
| id                         | string            | 节点的唯一ID                                                 | 是       |        |
| name                       | string            | 节点名称                                                     | 是       |        |
| next                       | string            | 下一个节点ID。默认为`end`，表示流程结束                      | 否       | end    |
| skip                       | boolean 或 string | 是否忽略该节点。当类型为 string 的时候，表示使用表达式计算值。 | 否       | false  |
| preDelay                   | int 或 string     | 执行前延时，单位：毫秒。当类型为 string 的时候，表示使用表达式计算值。 | 否       | 0      |
| postDelay                  | int 或 string     | 执行后延时，单位：毫秒。当类型为 string 的时候，表示使用表达式计算值。 | 否       | 0      |
| [errorStrategy](#异常策略) | object            | 定义异常策略，当节点执行发生异常的时候执行异常策略。         | 否       |        |
| type                       | enum              | 节点类型。                                                   | 否       | task   |
| metadata                   | object            | 元数据                                                       | 否       |        |



### 变量注入节点

基础属性

| 属性名称  | 类型              | 描述                                                         | 是否必填 | 默认值 |
| --------- | ----------------- | ------------------------------------------------------------ | -------- | ------ |
| id        | string            | 节点的唯一ID                                                 | 是       |        |
| name      | string            | 节点名称                                                     | 是       |        |
| next      | string            | 下一个节点ID。默认为`end`，表示流程结束                      | 否       | end    |
| skip      | boolean 或 string | 是否忽略该节点。当类型为 string 的时候，表示使用表达式计算值。 | 否       | false  |
| preDelay  | int 或 string     | 执行前延时，单位：毫秒。当类型为 string 的时候，表示使用表达式计算值。 | 否       | 0      |
| postDelay | int 或 string     | 执行后延时，单位：毫秒。当类型为 string 的时候，表示使用表达式计算值。 | 否       | 0      |
| type      | string            | 节点类型。固定值：`inject`                                   | 是       |        |
| metadata  | object            | 元数据                                                       | 否       |        |

特有属性

| 属性名称 | 类型   | 描述                                                    | 是否必填 | 默认值 |
| -------- | ------ | ------------------------------------------------------- | -------- | ------ |
| data     | object | 此数据最终会注入流程变量中，如果key存在则修改，否则新增 | 是       |        |

#### 示例

```json
{
  "id": "inject_node",
  "name": "变量赋值节点",
  "type": "inject",
  "data": {
    "person": {
      "name": "Tom",
      "age": 40
    },
    "result": "success",
  }
}
```

运行节点前，流程变量为：

```json
{
  "person": {},
  "num": 1
}
```

运行节点后，流程变量为：

```json
{
  "person": {
    "name": "Tom",
    "age": 40
  },
  "num": 1,
  "result": "success"
}
```


### 任务节点

基础属性

| 属性名称                   | 类型              | 描述                                                         | 是否必填 | 默认值 |
| -------------------------- | ----------------- | ------------------------------------------------------------ | -------- | ------ |
| id                         | string            | 节点的唯一ID                                                 | 是       |        |
| name                       | string            | 节点名称                                                     | 是       |        |
| next                       | string            | 下一个节点ID。默认为`end`，表示流程结束                      | 否       | end    |
| skip                       | boolean 或 string | 是否忽略该节点。当类型为 string 的时候，表示使用表达式计算值。 | 否       | false  |
| preDelay                   | int 或 string     | 执行前延时，单位：毫秒。当类型为 string 的时候，表示使用表达式计算值。 | 否       | 0      |
| postDelay                  | int 或 string     | 执行后延时，单位：毫秒。当类型为 string 的时候，表示使用表达式计算值。 | 否       | 0      |
| [errorStrategy](#异常策略) | object            | 定义异常策略，当节点执行发生异常的时候执行异常策略。         | 否       |        |
| type                       | string            | 节点类型。固定值：`task`                                     | 是       |        |
| metadata                   | object            | 元数据                                                       | 否       |        |

特有属性

| 属性名称             | 类型  | 描述                                                         | 是否必填 | 默认值         |
| -------------------- | ----- | ------------------------------------------------------------ | -------- | -------------- |
| cmdMode              | enum  | 指令执行模式。指令执行模式。`sequentially`：顺序，`parallel`：并行 | 否       | `sequentially` |
| [commands](#command) | array | 指令数组。数组中至少含有一个指令。                           | 是       |                |

#### 示例

```json
{
  "id": "task_node_demo",
  "name": "任务节点示例",
  "type": "task",
  "cmdMode": "parallel",
  "commands": [
    {
      "type": "function",
      "function": {
        "name": "sendEmail",
        "invocation": "path/module/emailOp#send",
        "args": {
          "subject": "hello",
          "body": "${ body }"
        }
      }
    },
    {
      "type": "subflow",
      "subflow": {
        "workflowId": "submitEmailToWebsiteFlow"
      }
    }
  ]
}
```

### 分支节点

基础属性

| 属性名称                   | 类型              | 描述                                                         | 是否必填 | 默认值 |
| -------------------------- | ----------------- | ------------------------------------------------------------ | -------- | ------ |
| id                         | string            | 节点的唯一ID                                                 | 是       |        |
| name                       | string            | 节点名称                                                     | 是       |        |
| skip                       | boolean 或 string | 是否忽略该节点。当类型为 string 的时候，表示使用表达式计算值。 | 否       | false  |
| [errorStrategy](#异常策略) | object            | 定义异常策略，当节点执行发生异常的时候执行异常策略。         | 否       |        |
| type                       | enum              | 节点类型。固定值：`branch`                                   | 是       | branch |
| metadata                   | object            | 元数据                                                       | 否       |        |

特有属性

| 属性名称                            | 类型   | 描述                                         | 是否必填 | 默认值 |
| ----------------------------------- | ------ | -------------------------------------------- | -------- | ------ |
| [branches](#condition-branch)       | array  | 可选分支                                     | 是       |        |
| [defaultBranch](#condition-branch) | object | 默认分支。若不满足以上分支条件，则走默认分支 | 是       |        |

#### condition branch

| 属性名称  | 类型   | 描述                                        | 是否必填 | 默认值 |
| --------- | ------ | ------------------------------------------- | -------- | ------ |
| condition | string | 字符串表达式。执行结果必须为`true`或`false` | 是       |        |
| next      | string | 下一个节点ID。默认为`end`，表示流程结束     | 否       | end    |

#### 示例

```json
{
  "id": "branch_node_demo",
  "name": "分支节点示例",
  "type": "branch",
  "branches": [
    {
      "condition": "${ status == 'ready' }",
      "next": "open_software_node"
    },
    {
      "condition": "${ status == 'completed' }",
      "next": "end"
    }
  ],
  "defaultBranch": {
      "next": "send_email_node"
  }
}
```

### 循环节点

循环容器支持两种方式的循环，一种是`range`，一种是`iterator`.详细属性如下:

基础属性

| 属性名称                   | 类型              | 描述                                                         | 是否必填 | 默认值 |
| -------------------------- | ----------------- | ------------------------------------------------------------ | -------- | ------ |
| id                         | string            | 节点的唯一ID                                                 | 是       |        |
| name                       | string            | 节点名称                                                     | 是       |        |
| next                       | string            | 下一个节点ID。默认为`end`，表示流程结束                      | 否       | end    |
| skip                       | boolean 或 string | 是否忽略该节点。当类型为 string 的时候，表示使用表达式计算值。 | 否       | false  |
| preDelay                   | int 或 string     | 执行前延时，单位：毫秒。当类型为 string 的时候，表示使用表达式计算值。 | 否       | 0      |
| postDelay                  | int 或 string     | 执行后延时，单位：毫秒。当类型为 string 的时候，表示使用表达式计算值。 | 否       | 0      |
| [errorStrategy](#异常策略) | object            | 定义异常策略，当节点执行发生异常的时候执行异常策略。         | 否       |        |
| type                       | enum              | 节点类型。固定值：`loop`                                     | 是       | loop   |
| metadata                   | object            | 元数据                                                       | 否       |        |

特有属性

| 属性名称              | 类型   | 描述                                                  | 是否必填                     | 默认值 |
| --------------------- | ------ | ----------------------------------------------------- | ---------------------------- | ------ |
| [commands](#command)  | array  | 任务指令数组。数组中至少含有一个指令。                | 是                           |        |
| mode                  | enum   | 指令执行模式。`range`：普通循环，`iterator`：迭代循环 | 否                           | range  |
| [range](#range)       | object | 普通循环参数                                          | 如果`mode`为`range`则必填    |        |
| [iterator](#iterator) | object | 迭代循环参数                                          | 如果`mode`为`iterator`则必填 |        |

#### range

| 属性名称 | 类型          | 描述                                                   | 是否必填 | 默认值 |
| -------- | ------------- | ------------------------------------------------------ | -------- | ------ |
| start    | int 或 string | 开始点。当类型为 string 的时候，表示使用表达式计算值。 | 否       | 0      |
| end      | int 或 string | 结束点。当类型为 string 的时候，表示使用表达式计算值。 | 是       |        |
| step     | int 或 string | 步长。当类型为 string 的时候，表示使用表达式计算值。   | 否       | 1      |

#### iterator

| 属性名称     | 类型            | 描述                                                         | 是否必填 | 默认值             |
| ------------ | --------------- | ------------------------------------------------------------ | -------- | ------------------ |
| iteratorMode | enum            | 指定迭代集合的运行模式。`sequentially`：顺序，`parallel`：并行 | 否       | sequentially       |
| collection   | array 或 string | 迭代集合。当类型为 string 的时候，表示使用表达式计算值。     | 是       |                    |
| item         | string          |                                                              |          |                    |
| batchSize    | int             | 指定可以并行执行的迭代数量。当`iteratorMode`为`parallel`时生效。默认为`collection`的长度 | 否       | `collection`的长度 |

#### 示例

range模式循环节点

```json
{
  "id": "range_loop_demo",
  "name": "range类型循环节点示例",
  "type": "loop",
  "mode": "range",
  "range": {
    "start": 0,
    "step": 1,
    "end": 10
  },
  "commands": [
    {
      "type": "function",
      "function": {
        "name": "sendEmail",
        "invocation": "path/module/emailOp#send",
        "args": {
          "subject": "hello",
          "body": "${ emails[#index].body }"
        }
      }
    },
    {
      "type": "subflow",
      "subflow": {
        "workflowId": "submitEmailToWebsiteFlow"
      },
      "continue": "${ isSubmitEmailSuccess }"
    },
    {
      "type": "function",
      "function": {
        "name": "reportErr",
        "invocation": "path/module/emailOp#reportErr",
        "args": {
          "err": "${ err }"
        }
      },
      "break": "${ err == 'SUBMIT_EMAIL_ERROR' }"
    }
  ]
}
```

iterator模式循环节点

```json
{
  "id": "iterator_loop_demo",
  "name": "iterator类型循环节点示例",
  "type": "loop",
  "mode": "iterator",
  "iterator": {
    "iteratorMode": "parallel",
    "collection": [
      "a",
      "b",
      "c",
      "d",
      "e",
      "f"
    ],
    "batchSize": 3,
    "commands": [
      {
      "type": "function",
      "function": {
        "name": "print",
        "invocation": "path/module/MsgPrinter#print",
        "args": {
          "msg": "${ #item }",
        }
      }
    }
    ]
  }
}
```

### 并行节点

基础属性

| 属性名称                   | 类型              | 描述                                                         | 是否必填 | 默认值   |
| -------------------------- | ----------------- | ------------------------------------------------------------ | -------- | -------- |
| id                         | string            | 节点的唯一ID                                                 | 是       |          |
| name                       | string            | 节点名称                                                     | 是       |          |
| next                       | string            | 下一个节点ID。默认为`end`，表示流程结束                      | 否       | end      |
| skip                       | boolean 或 string | 是否忽略该节点。当类型为 string 的时候，表示使用表达式计算值。 | 否       | false    |
| preDelay                   | int 或 string     | 执行前延时，单位：毫秒。当类型为 string 的时候，表示使用表达式计算值。 | 否       | 0        |
| postDelay                  | int 或 string     | 执行后延时，单位：毫秒。当类型为 string 的时候，表示使用表达式计算值。 | 否       | 0        |
| [errorStrategy](#异常策略) | object            | 定义异常策略，当节点执行发生异常的时候执行异常策略。         | 否       |          |
| type                       | enum              | 节点类型。固定值：`parallel`                                 | 是       | parallel |
| metadata                   | object            | 元数据                                                       | 否       |          |

特有属性

| 属性名称                    | 类型   | 描述                                   | 是否必填 | 默认值 |
| --------------------------- | ------ | -------------------------------------- | -------- | ------ |
| [branches](#parallel-branch) | array  | 并行分支数组。数组中至少含有一个分支。 | 是       |        |

#### parallel branch

| 属性名称             | 类型   | 描述                                   | 是否必填 | 默认值 |
| -------------------- | ------ | -------------------------------------- | -------- | ------ |
| id                   | string | 并行分支唯一ID                         | 是       |        |
| [commands](#command) | array  | 任务指令数组。数组中至少含有一个指令。 | 是       |        |

#### 示例

```json
{
  "id": "parallel_node_demo",
  "name": "并行节点示例",
  "type": "parallel",
  "branches": [
    {
      "id": "branch1",
      "commands": [
        {
          "type": "function",
          "function": {
            "name": "print",
            "invocation": "path/module/MsgPrinter#print",
            "args": {
              "msg": "${ msg }",
            }
          }
        }
      ]
    },
    {
      "id": "branch2",
      "commands": [
        {
          "type": "subflow",
          "subflow": {
            "workflowId": "collectMsgSubflow"
          }
        }
          
      ]
    }
  ]
}
```



## 关联对象属性定义

关联到节点的属性结构定义

### command

| 属性名称            | 类型              | 描述                                                         | 是否必填                 | 默认值   |
| ------------------- | ----------------- | ------------------------------------------------------------ | ------------------------ | -------- |
| type                | enum              | 调用类型。可选值：`function`、`subflow`                      | 是                       | function |
| skip                | boolean 或 string | 是否忽略该指令。当类型为 string 的时候，表示使用表达式计算值。 | 否                       | false    |
| continue            | boolean 或 string | 执行完当前指令的时候，是否结束后续指令，并进行下一次循环。当类型为 string 的时候，表示使用表达式计算值。 | 否                       | false    |
| break               | boolean 或 string | 执行完当前指令的时候，是否结束当前节点运行。当类型为 string 的时候，表示使用表达式计算值。iteratorMode": "parallel" | 否                       | false    |
| [function](#函数)   | object            | 要执行的函数                                                 | 当type为`function`时必填 |          |
| [subflow](#subflow) | object            | 要执行的子流程                                               | 当type为`subflow`时必填  |          |

- continue：此字段仅适用于[循环节点](#循环节点)，且不适用于[iteratorMode](#iterator)为`parallel`的情况
- break：此字段仅适用于[循环节点](#循环节点)，且不适用于[iteratorMode](#iterator)为`parallel`的情况

#### function

function即函数，是一种指令（[command](#command)），通常指的是程序中的函数，即代码执行的语句块。

| 属性名称   | 类型    | 描述                                                         | 是否必填 | 默认值 |
| ---------- | ------- | ------------------------------------------------------------ | -------- | ------ |
| name       | string  | 函数名                                                       | 是       |        |
| invocation | string  | 函数调用定义，如`path/module/numberOp#add`                   | 是       |        |
| args       | object  | 函数参数列表。参数名称为`string`类型，参数值为任何类型，也支持表达式。 | 否       |        |
| output     | string  | 返回值的流程变量名，若流程不存在此变量会新增，否则修改。不填表示不接收返回值 | 否       |        |
| async      | boolean | 是否异步执行。                                               | 否       | false  |

##### 示例一

有返回值

```json
{
  "name": "add",
  "invocation": "path/module/numberOp#add",
  "args": {
    "num1": 1,
    "num2": "${ num2 }"
  },
  "output": "result",
  "async": false,
  "skip": false
}
```

##### 示例二

无返回值

```json
{
  "name": "print",
  "invocation": "path/module/Printer#print",
  "args": {
    "str": "${ str }"
  },
  "async": false,
  "skip": false
}
```

#### subflow

subflow即子流程，是一种指令（[command](#command)），用于调用外部流程。

| 属性名称   | 类型    | 描述                                                         | 是否必填 | 默认值 |
| ---------- | ------- | ------------------------------------------------------------ | -------- | ------ |
| workflowId | string  | 子流程定义id                                                 | 是       |        |
| version    | string  | 子流程定义版本                                               | 否       |        |
| async      | boolean | 是否异步执行子流程                                           | 否       | false  |
| cascade    | boolean | 当`async`为`true`的时候，若主流程运行停止，当前子流程是否也级联停止 | 否       | true   |

##### 示例

以下示例定义了一个子流程的执行方式为异步执行，当主流程结束时，该子流程级联结束。

```json
{
  "workflowId": "subflow_id",
  "version": "0.1",
  "async": true,
  "cascade": true
}
```



### 异常策略

异常策略用于当节点执行发生异常的时候，触发的行为。

| 属性名称   | 类型   | 描述                                                         | 是否必填             | 默认值 |
| ---------- | ------ | ------------------------------------------------------------ | -------------------- | ------ |
| type       | enum   | 定义异常策略类型。异常策略目前有 3 种：`retry`，`goto`，`exit` | 是                   | `exit` |
| properties | object | 异常策略参数。可以为[retry](#retry)、[goto](#goto)、[exit](#exit)其中之一 | 除了`exit`类型，其他必填 |        |

#### retry

节点发生异常时，根据参数进行重试。

| 属性名称        | 类型 | 描述     | 是否必填 | 默认值 |
| --------------- | ---- | -------- | -------- | ------ |
| interval | int  | 重试间隔，单位：毫秒 | 否       | 0      |
| retryCount      | int  | 重试次数 | 是       | 1      |

##### 示例

发生异常时，重试三次，每次间隔1000毫秒

```json
{
  "type": "retry",
  "properties": {
    "interval": 1000,
    "retryCount": 3
  }
}
```

#### goto

节点发生异常时，跳转到其他节点。

| 属性名称            | 类型   | 描述                                         | 是否必填 | 默认值 |
| ------------------- | ------ | -------------------------------------------- | -------- | ------ |
| [errRefs](#errRefs) | array  | 错误关联                                     | 否       |        |
| defaultNext         | string | 默认跳转的节点 ID。默认为`end`，表示流程结束 | 否       | end    |

##### errRefs

| 属性名称 | 类型   | 描述                                                         | 是否必填 | 默认值 |
| -------- | ------ | ------------------------------------------------------------ | -------- | ------ |
| errCode  | string | 错误代码                                                     | 是       |        |
| next     | string | 若抛出的错误代码符合，则运行对应节点。默认为`end`，表示流程结束 | 否       | end    |

##### 示例

以下例子定义了异常策略：

- 若`errCode`为`MISSING_ID`时，运行`handle_missing_id_node`节点
- 若`errCode`为`MISSING_PRICE`时，流程结束
- 若没有`errCode`或`errCode`不匹配，流程结束

```json
{
  "type": "goto",
  "properties": {
    "errRefs": [
      {
        "errCode": "MISSING_ID",
        "next": "handle_missing_id_node"
      },
      {
        "errCode": "MISSING_PRICE",
        "next": "end"
      }
    ],
    "defaultNext": "end"
  }
}
```

#### exit

节点发生异常时，直接结束流程，无参数

##### 示例

```json
{
  "type": "exit"
}
```

