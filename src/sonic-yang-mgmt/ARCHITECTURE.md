# Sonic YANG Management 模块架构详解

## 目录
1. [模块概述](#模块概述)
2. [核心类详解](#核心类详解)
3. [主要功能](#主要功能)
4. [数据结构](#数据结构)
5. [工作流程](#工作流程)

---

## 模块概述

**sonic-yang-mgmt** 是 SONiC 中用于 YANG 模型管理和配置验证的核心 Python 模块。

### 核心职责
- **YANG 模型管理**: 加载、解析和管理 YANG schema
- **数据验证**: 基于 YANG 模型验证配置数据
- **格式转换**: ConfigDB JSON ? YANG JSON 双向转换
- **路径转换**: ConfigDB Path ? XPath 相互转换
- **依赖分析**: 查找配置项之间的依赖关系
- **数据树操作**: 提供数据节点的增删改查

### 模块文件
```
sonic-yang-mgmt/
├── sonic_yang.py          # 核心类 SonicYang
├── sonic_yang_ext.py      # 扩展功能 SonicYangExtMixin
├── sonic_yang_path.py     # 路径处理 SonicYangPathMixin
└── sonic-cfg-help         # 命令行工具
```

### 依赖关系
```python
# 外部依赖
libyang (yang)      # YANG 模型解析和验证
xmltodict           # XML 到字典转换
jsonpointer         # JSON 路径操作
ijson               # 流式 JSON 解析
jsondiff            # JSON 差异比较
tabulate            # 表格格式化

# 继承关系
SonicYang(SonicYangExtMixin, SonicYangPathMixin)
SonicYangExtMixin(SonicYangPathMixin)
```

---

## 核心类详解

### 1. SonicYang 类

**位置**: `sonic_yang.py`

**继承**: `SonicYang(SonicYangExtMixin, SonicYangPathMixin)`

#### 核心属性

```python
class SonicYang:
    # libyang 组件
    ctx: ly.Context              # libyang 上下文
    module: ly.Module            # 当前 YANG 模块
    root: ly.Data_Node           # 数据树根节点

    # 配置和映射
    yang_dir: str                # YANG 模型目录
    yangFiles: list              # YANG 文件列表
    confDbYangMap: dict          # ConfigDB表到YANG容器映射

    # JSON 数据
    yJson: list                  # YANG 模型的 JSON 表示
    jIn: dict                    # 输入的 ConfigDB JSON
    xlateJson: dict              # 转换后的 YANG JSON
    revXlateJson: dict           # 反向转换的 ConfigDB JSON
    tablesWithOutYang: dict      # 没有 YANG 模型的表

    # 预处理和缓存
    preProcessedYang: dict       # 预处理的 YANG 模型
    backlinkCache: dict          # 反向引用缓存
    mustCache: dict              # must 语句计数缓存
    configPathCache: dict        # 路径转换缓存

    # 其他
    elementPath: list            # 当前元素路径
    DEBUG: bool                  # 调试模式
    print_log_enabled: bool      # 日志打印开关
```

#### 核心方法

##### 1.1 初始化和模型加载

```python
def __init__(yang_dir, debug=False, print_log_enabled=True,
             sonic_yang_options=0):
    """
    初始化 SonicYang 实例

    参数:
        yang_dir: YANG 模型目录
        debug: 调试模式
        print_log_enabled: 是否打印日志
        sonic_yang_options: libyang 选项标志
    """
```

**内部方法**:
- `_load_schema_module(yang_file)`: 加载单个 YANG 模型文件
- `_load_schema_module_list(yang_files)`: 加载多个 YANG 模型
- `_load_schema_modules(yang_dir)`: 从目录加载所有 YANG 模型
- `_load_schema_modules_ctx(yang_dir)`: 创建 libyang 上下文并加载模型

##### 1.2 数据加载和验证

```python
def _load_data_file(data_file):
    """加载数据文件到数据树"""

def validate_data_tree():
    """
    验证数据树是否符合 YANG 模型

    返回: (True/False, error_message)
    """

def _validate_data(node, ctx):
    """内部验证方法"""
```

##### 1.3 数据节点操作

```python
def _find_data_node(data_xpath):
    """根据 XPath 查找数据节点"""

def _find_schema_node(schema_xpath):
    """根据 XPath 查找 schema 节点"""

def _new_data_node(xpath, value):
    """创建新的数据节点"""

def _add_data_node(data_xpath, value):
    """添加数据节点"""

def _deleteNode(xpath, node):
    """删除数据节点"""

def _find_data_node_value(data_xpath):
    """获取数据节点的值"""

def _set_data_node_value(data_xpath, value):
    """设置数据节点的值"""
```

##### 1.4 依赖分析

```python
def find_data_dependencies(data_xpath):
    """
    查找引用指定数据路径的所有路径

    参数:
        data_xpath: 数据 XPath

    返回: 依赖路径列表

    抛出: SonicYangException
    """

def find_schema_dependencies(schema_xpath, match_ancestors=True):
    """
    查找引用指定 schema 路径的所有 schema 路径

    参数:
        schema_xpath: Schema XPath
        match_ancestors: 是否匹配祖先节点

    返回: 依赖路径列表

    抛出: SonicYangException
    """

def find_schema_must_count(schema_xpath, match_ancestors=True):
    """
    统计 schema 路径上的 must 语句数量

    参数:
        schema_xpath: Schema XPath
        match_ancestors: 是否匹配祖先节点

    返回: must 语句数量

    抛出: SonicYangException
    """
```

##### 1.5 辅助方法

```python
def _get_module(module_name):
    """获取 YANG 模块"""

def _get_module_prefix(module_name):
    """获取模块前缀"""

def _get_module_name(schema_xpath):
    """从 XPath 提取模块名"""

def _get_data_type(schema_xpath):
    """获取数据类型"""

def _get_leafref_path(schema_xpath):
    """获取 leafref 引用的路径"""
```

---

### 2. SonicYangExtMixin 类

**位置**: `sonic_yang_ext.py`

**继承**: `SonicYangExtMixin(SonicYangPathMixin)`

**职责**: ConfigDB 与 YANG 格式之间的转换

#### 公开方法

```python
def loadYangModel():
    """
    加载所有 YANG 模型并创建映射

    抛出: SonicYangException
    """

def loadData(configdbJson, debug=False):
    """
    加载 ConfigDB JSON 并转换为 YANG 格式

    参数:
        configdbJson: ConfigDB 格式的 JSON
        debug: 调试模式

    抛出: SonicYangException
    """

def getData(debug=False):
    """
    获取 ConfigDB 格式的数据

    返回: ConfigDB JSON

    抛出: SonicYangException
    """

def deleteNode(xpath):
    """
    删除指定 XPath 的节点

    参数:
        xpath: 要删除的节点路径

    抛出: SonicYangException
    """

def findXpathPort(portName):
    """查找端口的 XPath"""

def findXpathPortLeaf(portName):
    """查找端口叶子节点的 XPath"""

def XlateYangToConfigDB(yang_data):
    """
    将 YANG 格式转换为 ConfigDB 格式

    参数:
        yang_data: YANG JSON 数据

    返回: ConfigDB JSON
    """
```

#### 内部转换方法

##### 2.1 YANG 模型预处理

```python
def _loadJsonYangModel():
    """加载 YANG 模型的 JSON 表示"""

def _preProcessYangGrouping(moduleName, module):
    """预处理 YANG grouping"""

def _preProcessYang(moduleName, module):
    """预处理 YANG 模型"""

def _compileUsesClause():
    """编译 uses 子句"""
```

##### 2.2 ConfigDB → YANG 转换

```python
def _xlateConfigDBtoYang(jIn, yangJ):
    """
    ConfigDB JSON 转 YANG JSON

    处理流程:
    1. 遍历 ConfigDB 表
    2. 查找对应的 YANG 容器
    3. 根据 YANG 模型类型转换数据
    """

def _xlateList(model, yang, config, table, exceptionList):
    """转换列表类型"""

def _xlateContainer(model, yang, config, table):
    """转换容器类型"""

def _xlateType1MapList(model, yang, config, table, exceptionList):
    """转换 Type 1 映射列表（特殊嵌套列表）"""
```

##### 2.3 YANG → ConfigDB 转换

```python
def _revXlateYangtoConfigDB(yangJ, cDbJson):
    """
    YANG JSON 转 ConfigDB JSON

    处理流程:
    1. 遍历 YANG 容器
    2. 查找对应的 ConfigDB 表
    3. 根据模型类型反向转换数据
    """

def _revXlateList(model, yang, config, table):
    """反向转换列表"""

def _revXlateContainer(model, yang, config, table):
    """反向转换容器"""

def _revXlateType1MapList(model, yang, config, table):
    """反向转换 Type 1 映射列表"""
```

##### 2.4 辅助方法

```python
def _createDBTableToModuleMap():
    """创建 ConfigDB 表到 YANG 模块的映射"""

def _extractKey(tableKey, keys):
    """从表键中提取键值"""

def _createKey(entry, keys):
    """创建表键"""

def _createLeafDict(model, table):
    """创建叶子节点字典"""

def _findYangTypedValue(key, value, leafDict):
    """查找 YANG 类型的值（ConfigDB → YANG）"""

def _revFindYangTypedValue(key, value, leafDict):
    """反向查找值（YANG → ConfigDB）"""
```

---

### 3. SonicYangPathMixin 类

**位置**: `sonic_yang_path.py`

**职责**: ConfigDB 路径与 YANG XPath 之间的转换

#### 公开方法

```python
def configdb_path_to_xpath(configdb_path, schema_xpath="", configdb=None):
    """
    ConfigDB 路径转 XPath

    参数:
        configdb_path: ConfigDB 路径 (JSON Pointer 格式)
        schema_xpath: 起始 schema XPath (可选)
        configdb: ConfigDB 数据 (用于查找键值)

    返回: XPath 字符串

    示例:
        /PORT/Ethernet0/admin_status
        → /sonic-port:sonic-port/PORT/PORT_LIST[name='Ethernet0']/admin_status
    """

def xpath_to_configdb_path(xpath, configdb=None):
    """
    XPath 转 ConfigDB 路径

    参数:
        xpath: YANG XPath
        configdb: ConfigDB 数据

    返回: ConfigDB 路径 (JSON Pointer 格式)

    示例:
        /sonic-port:sonic-port/PORT/PORT_LIST[name='Ethernet0']/admin_status
        → /PORT/Ethernet0/admin_status
    """
```

#### 静态方法

```python
@staticmethod
def configdb_path_split(configdb_path):
    """分割 ConfigDB 路径为 token 列表"""

@staticmethod
def configdb_path_join(configdb_tokens):
    """连接 token 列表为 ConfigDB 路径"""

@staticmethod
def xpath_split(xpath):
    """分割 XPath 为 token 列表"""

@staticmethod
def xpath_join(xpath_tokens, schema_xpath=""):
    """连接 token 列表为 XPath"""
```

---

## 主要功能

### 1. YANG 模型加载

```python
sy = SonicYang("/path/to/yang/models")
sy.loadYangModel()
```

**流程**:
1. 创建 libyang 上下文
2. 加载所有 .yang 文件
3. 预处理 YANG 模型（处理 grouping 和 uses）
4. 创建 ConfigDB 表到 YANG 容器的映射

### 2. 配置验证

```python
# 加载配置
sy.loadData(configdb_json)

# 验证
is_valid, error = sy.validate_data_tree()
if not is_valid:
    print(f"验证失败: {error}")
```

### 3. 格式转换

```python
# ConfigDB → YANG
sy.loadData(configdb_json)
yang_json = sy.xlateJson

# YANG → ConfigDB
configdb_json = sy.XlateYangToConfigDB(yang_json)
```

### 4. 路径转换

```python
# ConfigDB Path → XPath
xpath = sy.configdb_path_to_xpath("/PORT/Ethernet0/admin_status")
# 结果: /sonic-port:sonic-port/PORT/PORT_LIST[name='Ethernet0']/admin_status

# XPath → ConfigDB Path
path = sy.xpath_to_configdb_path(xpath)
# 结果: /PORT/Ethernet0/admin_status
```

### 5. 依赖分析

```python
# 查找引用 Ethernet0 的所有配置
deps = sy.find_data_dependencies("/sonic-port:sonic-port/PORT/PORT_LIST[name='Ethernet0']")
# 可能返回: ["/VLAN_MEMBER/...", "/ACL_TABLE/...", ...]
```

---

## 数据结构

### confDbYangMap 结构

```python
{
    "TABLE_NAME": {
        "module": "module_name",
        "topLevelContainer": "container_name",
        "container": {...},  # YANG 容器定义
        "yangModule": {...}  # YANG 模块 JSON
    }
}
```

### Type 1 List Maps

特殊的嵌套列表类型，需要特殊处理:

```python
Type_1_list_maps_model = [
    "DSCP_TO_TC_MAP_LIST",
    "DOT1P_TO_TC_MAP_LIST",
    "TC_TO_PRIORITY_GROUP_MAP_LIST",
    "TC_TO_QUEUE_MAP_LIST",
    ...
]
```

### Leaf List 字符串值字典

某些字段在 YANG 中定义为 leaf-list，但在 ConfigDB 中存储为字符串:

```python
LEAF_LIST_WITH_STRING_VALUE_DICT = {
    ("MIRROR_SESSION", "src_ip"): ",",
    ("NTP", "src_intf"): ";",
    ("PORT", "adv_speeds"): ",",
    ...
}
```

---

## 工作流程

### 完整的配置验证流程

```
1. 初始化
   SonicYang(yang_dir) → 创建 libyang 上下文

2. 加载模型
   loadYangModel() → 加载所有 YANG 文件
                  → 创建 confDbYangMap

3. 加载数据
   loadData(configdb_json) → _xlateConfigDBtoYang()
                           → 转换为 YANG JSON
                           → 加载到数据树

4. 验证
   validate_data_tree() → libyang 验证
                        → 返回结果

5. 获取数据
   getData() → _revXlateYangtoConfigDB()
            → 转换回 ConfigDB JSON
```

完整文档已创建！
