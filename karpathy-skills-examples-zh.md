# 示例

四大原则在真实代码中的演示。每个示例都展示 LLM 常犯的错误以及正确的做法。

> 译自 <https://github.com/forrestchang/andrej-karpathy-skills/blob/main/EXAMPLES.md>

---

## 1. Think Before Coding（动手前先思考）

### 示例 1：隐藏的假设

**用户请求：** "加一个导出用户数据的功能"

**❌ LLM 常犯的错（瞎做假设）**

```python
def export_users(format='json'):
    """Export all users to a file."""
    users = User.query.all()

    if format == 'json':
        with open('users.json', 'w') as f:
            json.dump([u.to_dict() for u in users], f)
    elif format == 'csv':
        with open('users.csv', 'w') as f:
            writer = csv.DictWriter(f, fieldnames=['id', 'email', 'name'])
            writer.writeheader()
            writer.writerows([u.to_dict() for u in users])

    return f"Exported {len(users)} users"
```

**问题：**

- 默认导出**所有**用户（分页呢？隐私呢？）
- 默认了文件存放位置
- 默认了要导出哪些字段
- 没看实际数据结构就写死了 CSV 字段名

**✅ 应该这样做（把假设摊开）**

```
在动手之前，我需要先澄清几点：

1. **范围**：导出全部用户还是某个筛选子集？（涉及隐私）
2. **形式**：你说的"导出"是指：
   - 浏览器下载文件？
   - 后台任务 + 邮件通知？
   - 返回数据的 API 接口？
3. **字段**：要包含哪些用户字段？（有些可能是敏感字段）
4. **量级**：通常会有多少用户？（影响实现方式）

最简实现：加一个返回分页 JSON 的 API 接口。
文件导出形式需要更多信息才能动手。

你倾向哪个方向？
```

### 示例 2：多种解读

**用户请求：** "把搜索做得更快一点"

**❌ LLM 常犯的错（默默选一种）**

```python
# 同时加了缓存、数据库索引和异步处理
# 完全没有问"快"到底指哪种快
@lru_cache(maxsize=1000)
async def search(query: str) -> List[Result]:
    # ... 200 行优化代码
```

**✅ 应该这样做**

```
"搜索更快" 可以指几种意思：

1. **响应时间更短**（从 500ms 降到 < 100ms）
   - 加数据库索引
   - 缓存高频查询
   - 预计工作量：2 小时

2. **能扛更多并发**（吞吐）
   - 加 async/await
   - 连接池
   - 预计工作量：4 小时

3. **感知更快**（用户体验）
   - 即时返回部分结果
   - 渐进式加载
   - 预计工作量：3 小时

当前搜索典型耗时约 500ms。
对你的场景而言，哪一种最关键？
```

---

## 2. Simplicity First（简单优先）

### 示例 1：过度抽象

**用户请求：** "加一个计算折扣的函数"

**❌ LLM 常犯的错（过度设计）**

```python
from abc import ABC, abstractmethod
from enum import Enum
from typing import Protocol, Union
from dataclasses import dataclass

class DiscountStrategy(ABC):
    @abstractmethod
    def calculate(self, amount: float) -> float:
        pass

class PercentageDiscount(DiscountStrategy):
    def __init__(self, percentage: float):
        self.percentage = percentage

    def calculate(self, amount: float) -> float:
        return amount * (self.percentage / 100)

class FixedDiscount(DiscountStrategy):
    def __init__(self, fixed_amount: float):
        self.fixed_amount = fixed_amount

    def calculate(self, amount: float) -> float:
        return min(self.fixed_amount, amount)

@dataclass
class DiscountConfig:
    strategy: DiscountStrategy
    min_purchase: float = 0.0
    max_discount: float = float('inf')

class DiscountCalculator:
    def __init__(self, config: DiscountConfig):
        self.config = config

    def apply_discount(self, amount: float) -> float:
        if amount < self.config.min_purchase:
            return 0
        discount = self.config.strategy.calculate(amount)
        return min(discount, self.config.max_discount)

# 为一次简单计算，调用前要写 30+ 行装配代码
```

**✅ 应该这样做（简单）**

```python
def calculate_discount(amount: float, percent: float) -> float:
    """计算折扣金额。percent 取值 0-100。"""
    return amount * (percent / 100)

# 使用
discount = calculate_discount(100.0, 10.0)  # 减 10 元
```

**什么时候才该加复杂度：** 真的出现多种折扣类型时再说。需求来了再重构，不要提前抽象。

### 示例 2：投机性的功能

**用户请求：** "把用户偏好保存到数据库"

**❌ LLM 常犯的错（夹带没人要的功能）**

```python
class PreferenceManager:
    def __init__(self, db, cache=None, validator=None):
        self.db = db
        self.cache = cache or InMemoryCache()
        self.validator = validator or DefaultValidator()

    def save(self, user_id: int, prefs: dict,
             merge: bool = True,
             validate: bool = True,
             notify: bool = False) -> bool:
        """支持合并、校验、通知的偏好保存。"""

        if validate:
            errors = self.validator.validate(prefs)
            if errors:
                raise ValidationError(errors)

        existing = self.get(user_id) if merge else {}

        if merge:
            existing.update(prefs)
            prefs = existing

        self.db.execute(
            "UPDATE users SET preferences = ? WHERE id = ?",
            (json.dumps(prefs), user_id)
        )

        if self.cache:
            self.cache.set(f"prefs:{user_id}", prefs)

        if notify:
            self.notify_preference_change(user_id, prefs)

        return True

    def notify_preference_change(self, user_id: int, prefs: dict):
        # 再来 30 行没人需要的通知逻辑
        pass
```

**✅ 应该这样做（只做被要求的事）**

```python
def save_preferences(db, user_id: int, preferences: dict):
    """把用户偏好保存到数据库。"""
    db.execute(
        "UPDATE users SET preferences = ? WHERE id = ?",
        (json.dumps(preferences), user_id)
    )
```

**之后再按需追加：** 缓存（当性能成为问题）、校验（当出现脏数据）、合并（当真的有这个需求）。

---

## 3. Surgical Changes（手术式改动）

### 示例 1：顺手重构

**用户请求：** "修一下空邮箱让校验器崩掉的 bug"

**❌ LLM 常犯的错（改太多）**

```diff
  def validate_user(user_data):
-     # Check email format
-     if not user_data.get('email'):
+     """Validate user data."""
+     email = user_data.get('email', '').strip()
+
+     # Validate email
+     if not email:
          raise ValueError("Email required")
-
-     # Basic email validation
-     if '@' not in user_data['email']:
+     if '@' not in email or '.' not in email.split('@')[1]:
          raise ValueError("Invalid email")

-     # Check username
-     if not user_data.get('username'):
+     # Validate username
+     username = user_data.get('username', '').strip()
+     if not username:
          raise ValueError("Username required")
+     if len(username) < 3:
+         raise ValueError("Username too short")
+     if not username.isalnum():
+         raise ValueError("Username must be alphanumeric")

      return True
```

**问题：**

- 顺手"加强"了与 bug 无关的邮箱校验
- 加了没人要求的用户名校验
- 改了注释
- 补了 docstring

**✅ 应该这样做（手术式）**

```diff
  def validate_user(user_data):
      # Check email format
-     if not user_data.get('email'):
+     email = user_data.get('email', '')
+     if not email or not email.strip():
          raise ValueError("Email required")

      # Basic email validation
-     if '@' not in user_data['email']:
+     if '@' not in email:
          raise ValueError("Invalid email")

      # Check username
      if not user_data.get('username'):
          raise ValueError("Username required")

      return True
```

**只改了：** 修复空邮箱处理所必须的那几行。

### 示例 2：风格漂移

**用户请求：** "给上传函数加上日志"

**❌ LLM 常犯的错（顺手把整个函数 reformat 了）**

```diff
- def upload_file(file_path, destination):
+ def upload_file(file_path: str, destination: str) -> bool:
+     """Upload file to destination with logging."""
+     logger.info(f"Uploading {file_path} to {destination}")
+
      try:
-         with open(file_path, 'rb') as f:
-             data = f.read()
-
-         response = requests.post(destination, files={'file': data})
-
-         if response.status_code == 200:
-             return True
-         else:
-             return False
+         with open(file_path, "rb") as f:
+             data = f.read()
+         response = requests.post(destination, files={"file": data})
+         success = response.status_code == 200
+
+         if success:
+             logger.info(f"Upload successful: {file_path}")
+         else:
+             logger.error(f"Upload failed: {file_path}, status={response.status_code}")
+
+         return success
      except Exception as e:
-         print(f"Error: {e}")
+         logger.exception(f"Upload error for {file_path}: {e}")
          return False
```

**问题：**

- 把单引号 `''` 改成了双引号 `""`
- 加了没人要求的类型注解
- 补了 docstring
- 调整了空白行
- 改了布尔返回的写法

**✅ 应该这样做（沿用既有风格）**

```diff
+ import logging
+
+ logger = logging.getLogger(__name__)
+
  def upload_file(file_path, destination):
+     logger.info(f'Starting upload: {file_path}')
      try:
          with open(file_path, 'rb') as f:
              data = f.read()

          response = requests.post(destination, files={'file': data})

          if response.status_code == 200:
+             logger.info(f'Upload successful: {file_path}')
              return True
          else:
+             logger.error(f'Upload failed: {file_path}, status={response.status_code}')
              return False
      except Exception as e:
-         print(f"Error: {e}")
+         logger.exception(f'Upload error: {file_path}')
          return False
```

**保持一致的：** 单引号、不加类型注解、原有的布尔返回模式、空白风格。

---

## 4. Goal-Driven Execution（目标驱动执行）

### 示例 1：模糊 vs 可验证

**用户请求：** "修一下认证系统"

**❌ LLM 常犯的错（模糊推进）**

```
我来修一下认证系统：
1. 看一下代码
2. 找出问题
3. 做一些改进
4. 测试这些改动

[然后没有清晰的成功标准就开始改]
```

**✅ 应该这样做（目标可验证）**

```
要"修认证"，得先定义成功标准。
具体在修哪个问题？

如果问题是"改完密码用户还能继续用旧 session"：

计划：
1. 写测试：改密码 → 验证旧 session 失效
   验证：测试失败（成功复现了 bug）

2. 实现：改密码时让所有 session 失效
   验证：测试通过

3. 检查边界：多端登录、并发改密
   验证：补充的测试也通过

4. 检查回归：原有的认证测试仍然通过
   验证：完整测试套件全绿

当前认证模块的测试覆盖：[展示覆盖率]
你看到的具体认证问题是什么？
```

### 示例 2：分步带验证

**用户请求：** "给 API 加上限流"

**❌ LLM 常犯的错（一把梭）**

```python
# 一次性写了 300 行：Redis 后端 + 多种策略 + 配置系统 + 监控
# 没有任何清晰的验证步骤
```

**✅ 应该这样做（分步推进，每步可验证）**

```
限流计划：

1. 先加最基本的内存限流（单个接口）
   验证：
   - 测试：100 次请求 → 前 10 次成功，其余返回 429
   - 手测：curl 同一接口 11 次，看到限流错误

2. 抽成中间件（应用到所有接口）
   验证：
   - 测试：限流同时生效于 /users 和 /posts
   - 原有的接口测试不受影响

3. 切到 Redis 后端（支持多实例）
   验证：
   - 测试：应用重启后限流计数仍有效
   - 测试：两个实例共享限流计数

4. 加上配置（每个接口独立的速率）
   验证：
   - 测试：/search 允许 10/min，/users 允许 100/min
   - 配置文件被正确解析

每一步都独立可验证、独立可上线。
要从第 1 步开始吗？
```

### 示例 3：测试先行

**用户请求：** "分数相同的时候排序会出问题"

**❌ LLM 常犯的错（没复现就改）**

```python
# 没确认 bug 就直接改排序逻辑
def sort_scores(scores):
    return sorted(scores, key=lambda x: (-x['score'], x['name']))
```

**✅ 应该这样做（先复现）**

```python
# 1. 先写一个能复现问题的测试
def test_sort_with_duplicate_scores():
    """同分时的排序行为测试。"""
    scores = [
        {'name': 'Alice', 'score': 100},
        {'name': 'Bob', 'score': 100},
        {'name': 'Charlie', 'score': 90},
    ]

    result = sort_scores(scores)

    # bug：同分时顺序不确定
    # 反复跑这个测试，结果应保持一致
    assert result[0]['score'] == 100
    assert result[1]['score'] == 100
    assert result[2]['score'] == 90

# 验证：跑 10 次 → 因顺序不一致而失败

# 2. 然后用稳定排序修复
def sort_scores(scores):
    """按分数降序；同分时按名字升序。"""
    return sorted(scores, key=lambda x: (-x['score'], x['name']))

# 验证：测试稳定通过
```

---

## 反模式速查表

| 原则 | 反模式 | 修正方法 |
|------|--------|----------|
| Think Before Coding | 默默假设文件格式、字段、范围 | 把假设列出来，主动澄清 |
| Simplicity First | 给单一折扣计算上策略模式 | 在真正需要前，只写一个函数 |
| Surgical Changes | 顺手改引号风格、加类型注解 | 只动修复 bug 所需的那几行 |
| Goal-Driven | "我来看看代码并做改进" | "写一个能复现 bug X 的测试 → 让它通过 → 验证无回归" |

## 核心洞见

"过度复杂"的例子并不是明显错误——它们都遵循设计模式与最佳实践。问题出在**时机**：在需求出现之前就加复杂度，会导致：

- 代码更难看懂
- 引入更多 bug
- 实现更慢
- 测试更难

而"简单"的版本：

- 更易理解
- 实现更快
- 更易测试
- 等真的需要复杂度时再重构

> **好代码是简洁地解决今天的问题，而不是过早地解决明天的问题。**
