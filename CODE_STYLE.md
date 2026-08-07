# Coding Style

## 目标

代码首先服务于阅读，其次才是复用。

不要为了"工程化"而工程化，不要为了"完整"而增加复杂度。

---

# 1. 只解决当前问题

- 不为未来可能存在的需求设计接口。
- 不创建只有一个实现的抽象层。
- 不提前设计插件系统。
- 不创建没有使用方的扩展点。

---

# 2. 少写 Wrapper

只有当 Wrapper 能提供以下能力之一时才允许存在：

- 隐藏复杂逻辑
- 隐藏第三方接口
- 保证统一行为

否则直接调用。

---

# 3. 少写 Helper

禁止出现：

- helper
- utils
- misc
- common

函数应该属于某个明确领域。

例如：

Bad

```
utils::Process()
```

Good

```
DocumentParser::Parse()
```

---

# 4. 一个函数只做一件事

函数应该：

- 输入明确
- 输出明确
- 副作用明确

不要混合：

- 参数检查
- 日志
- Metric
- 真正业务

---

# 5. 不要为了 DRY 而 DRY

允许少量重复。

如果抽象后的理解成本更高，就保持重复。

---

# 6. 优先 Early Return

避免：

```
if (...)
{
    if (...)
    {
        ...
    }
}
```

推荐：

```
if (...)
    return;

if (...)
    return;

...
```

---

# 7. 不要写显而易见的注释

禁止：

```cpp
// Increment count
count++;
```

注释应该解释：

- 为什么
- 前置条件
- 特殊约束

而不是解释代码。

---

# 8. 变量必须体现业务

避免：

```
data
result
temp
item
obj
```

优先：

```
merchant
order
token
request
response
```

---

# 9. 控制抽象层数

尽量：

```
Controller
↓

Service
↓

Repository
```

不要出现五六层调用链。

---

# 10. 日志只记录有价值的信息

不要：

```
Start xxx

End xxx
```

记录：

- 错误
- 性能
- 状态变化
- 关键参数

---

# 11. 不要写防御性模板代码

只有存在真实风险时：

- 判空
- try/catch
- 参数检查

否则省略。

---

# 12. 一个 Context 优于十几个参数

参数超过 5 个时：

优先考虑：

```
RequestContext
```

而不是继续增加参数。

---

# 13. 保持代码有"人味"

允许：

- 少量重复
- 长短不一致的函数
- 不完全对称的实现
- 根据业务做不同处理

不要追求所有函数都长得一样。

---

# 14. 每增加一层，都问自己一句

这一层到底隐藏了什么复杂性？

如果答案是：

"什么都没有"

删除这一层。

---

# 15. 优先可读性

如果两种写法性能一致：

永远选择第一次阅读最容易理解的那一种。

---

# 16. 注释规范

每新增一个函数时，必须在函数定义前添加注释，至少包含以下内容：

- **函数作用**：这个函数做了什么，为什么需要它。
- **调用规约**：
  - 入参：每个参数的含义、类型约束、取值范围。
  - 返回值：返回值的含义，可能的取值情况及对应语义。
  - 副作用：函数对外部状态的影响（修改全局变量、写文件、发网络请求、修改传入的可变参数等）。

在函数体内部，每个分支路径前需要用简短的文本描述该分支的触发条件，帮助读者快速理解逻辑走向，而不需要反推代码。如果是异常路径或错误处理路径，必须额外说明该异常发生的场景及处理策略。

---

# ⚠️ 17. 更新注释规范（特别重要！！！）

每次更新代码之后，**必须**仔细检查修改部分所对应的注释是否已同步更新。注释和实现不一致比没有注释更危险——它会误导后续的阅读者，造成对代码行为的错误理解。

检查范围的最低要求：

1. **必须**检查整个被修改函数的函数头部注释，确保与当前实现一致。
2. **必须**检查该函数内所有被修改分支路径的注释，确保触发条件描述与代码逻辑匹配。
3. 其余受影响的调用方、关联函数、文档等，按照修改的影响范围按需检查。

这是一条硬性要求，**每次提交前都必须执行**，不得跳过。

---

# 18. 用接口替代函数传递

在 Kotlin 等支持高阶函数的语言中，尽量不要通过直接传递一个函数（lambda 或函数引用）来让下层组件访问上层的能力。函数类型签名缺乏语义，无法承载文档注释，也难以在调用处快速理解其职责边界。正确的做法是：定义一个意义明确、职责清晰、注释完备的接口，然后在上层实例化该接口的对象，将对象传递到下层。

### Bad（传递裸函数）

```kotlin
// SkillExecutor 依赖一个"能列出所有 skill 名称"的能力，
// 但通过函数类型来表达，既没有名称约束，也没有注释附着点。
class SkillExecutor(
    private val listAllSkillNames: () -> List<String>
) {
    fun reload() {
        val names = listAllSkillNames()
        for (name in names) {
            // ...
        }
    }
}

// 调用方
val executor = SkillExecutor(
    listAllSkillNames = { skillService.getNames() }
)
```

### Good（定义接口并实例化传入）

```kotlin
/**
 * 提供 Skill 元信息的接口。由上层 SkillService 实现，
 * 供 SkillExecutor 在需要查询 Skill 列表时调用。
 */
interface ISkillMetaInfoProvider {
    /**
     * 获取当前系统中所有已注册 Skill 的名称列表。
     * @return 按注册顺序排列的 Skill 名称，不会返回 null，无 Skill 时返回空列表。
     */
    fun listAllSkillNames(): List<String>
}

class SkillExecutor(
    private val skillMetaProvider: ISkillMetaInfoProvider
) {
    fun reload() {
        val names = skillMetaProvider.listAllSkillNames()
        for (name in names) {
            // ...
        }
    }
}

// 调用方：上层实例化接口对象并传入
val executor = SkillExecutor(
    skillMetaProvider = object : ISkillMetaInfoProvider {
        override fun listAllSkillNames(): List<String> = skillService.getNames()
    }
)
```

接口版本的优势：接口名称 `ISkillMetaInfoProvider` 直接表达了"提供 Skill 元信息"这一职责；方法上的注释和 `@return` 规约依附在接口定义上，可以被 IDE 和文档工具识别；后续需要新增查询能力时，接口可以自然地扩展方法，而函数类型方案只能不断增加新的函数参数。

---

# 19. 接口命名以 I 开头

在 Kotlin/Java 代码中，接口的类名必须以大写字母 `I` 开头，紧随其后的字母也必须大写（如 `ISkillMetaInfoProvider`、`IUserRepository`、`IConfigLoader`）。这条规则的作用是让接口在调用处能被一眼识别——读者不需要跳转到定义就能知道这是一个接口，从而理解其背后可能存在多态行为或 Mock 实现。

---

# 20. 禁止魔法字符串/魔法数字表示状态

状态、类型、模式等可枚举的量在代码中必须以**枚举（enum）**的形式出现，禁止使用裸字符串或裸整数直接表示。枚举提供了编译期类型检查、IDE 自动补全，以及集中的定义点，而不像散落各处的魔法字符串那样难以追踪和重构。

如果状态值需要传输（例如序列化为 JSON、写入数据库、通过 HTTP 传递），可以在传输边界处进行枚举与字符串/整数的双向转换。转换时：

1. 每个字符串/整数到枚举值的映射关系必须以**命名常量 + 注释**的形式在代码中明确定义，常量放在枚举定义附近或专门的转换工具类中，不得在调用处内联魔法字符串。
2. 反序列化（字符串/整数 → 枚举）时必须**妥善处理识别不了的值**：要么返回 `null` 由调用方处理，要么抛出带有明确错误信息的异常，绝不能静默地映射到一个默认状态。

### Bad（魔法字符串）

```kotlin
fun process(order: Order) {
    when (order.status) {
        "pending"    -> handlePending(order)
        "processing" -> handleProcessing(order)
        "done"       -> archive(order)
        // 其他分支散落各处，靠字符串匹配，容易写错
    }
}
```

### Good（枚举 + 传输转换）

```kotlin
enum class OrderStatus {
    PENDING,
    PROCESSING,
    DONE;

    companion object {
        // 传输层常量：对外 JSON 字段值 → 枚举映射
        private const val JSON_PENDING    = "pending"
        private const val JSON_PROCESSING = "processing"
        private const val JSON_DONE       = "done"

        /** 从传输层字符串解析订单状态。返回 null 表示遇到未识别的值。 */
        fun fromJson(value: String): OrderStatus? = when (value) {
            JSON_PENDING    -> PENDING
            JSON_PROCESSING -> PROCESSING
            JSON_DONE       -> DONE
            else            -> null  // 不认识的输入，明确返回 null 让调用方决策
        }
    }

    /** 序列化为传输层字符串。 */
    fun toJson(): String = when (this) {
        PENDING    -> JSON_PENDING
        PROCESSING -> JSON_PROCESSING
        DONE       -> JSON_DONE
    }
}

fun process(order: Order) {
    when (order.status) {
        OrderStatus.PENDING    -> handlePending(order)
        OrderStatus.PROCESSING -> handleProcessing(order)
        OrderStatus.DONE       -> archive(order)
    } // 编译器会检查 when 是否覆盖了所有枚举值
}
```

---

# 21. SpringBoot 测试用例命名使用英文

在 SpringBoot 项目中编写测试用例时，函数名和测试用例名（`@Test` 方法名、`@DisplayName` 注解值、`@ParameterizedTest` 的参数名等）**必须使用英文命名**，禁止使用中文。

英文命名可以让测试函数名与生产代码保持一致的命名风格，避免中英文混杂导致的编码问题（如部分工具链对非 ASCII 标识符支持不完善），同时便于国际化团队协作。

### Bad（中文命名）

```java
@Test
@DisplayName("测试创建订单接口在库存不足时应该返回错误")
void 测试创建订单_库存不足_返回错误() {
    // ...
}
```

### Good（英文命名）

```java
@Test
@DisplayName("should return error when stock is insufficient for order creation")
void shouldReturnError_whenStockInsufficient_forOrderCreation() {
    // ...
}
```

---

# 22. 代码注释使用中文

所有代码注释（包括函数头部注释、行内注释、TODO 注释等）**必须使用中文**编写。中文注释可以降低团队内部的理解门槛，让不熟悉英文的开发者也能快速理解代码意图。

这条规则与第 21 条（测试命名用英文）并不矛盾：标识符（类名、函数名、变量名、测试用例名）是代码的一部分，必须能被编译器处理，保持英文命名以保证兼容性；而注释是给人读的，使用中文可以提高沟通效率。

### Bad（英文注释）

```java
/**
 * Processes the order and updates inventory.
 * Returns the updated order status.
 */
public OrderStatus processOrder(Order order) {
    // Check if inventory is sufficient
    if (inventory.getStock(order.getProductId()) < order.getQuantity()) {
        throw new InsufficientStockException();
    }
}
```

### Good（中文注释）

```java
/**
 * 处理订单并更新库存。
 * 返回更新后的订单状态。
 */
public OrderStatus processOrder(Order order) {
    // 检查库存是否充足
    if (inventory.getStock(order.getProductId()) < order.getQuantity()) {
        throw new InsufficientStockException();
    }
}
```

---

# 23. 测试钩子必须加 @VisibleForTesting 注解

在 Java/Kotlin 业务代码中，如果出于测试目的需要提升方法或字段的可见性（例如将 `private` 改为 `package-private` 或 `protected`），或者新增仅用于测试的访问方法，**必须**在声明处添加 `@VisibleForTesting` 注解。该注解明确标记了"这个方法/字段不是业务接口的一部分，仅为了测试而暴露"，避免后续维护者误将其当作正常的业务 API 来调用或扩展。

此注解来源于 Google Guava 库（`com.google.common.annotations.VisibleForTesting`），Android 项目中也可使用 `androidx.annotation.VisibleForTesting`。

### Bad（无注解标记）

```kotlin
// 原为 private，为了测试改为 internal，但没有任何标记说明原因
internal fun calculateDiscount(price: BigDecimal): BigDecimal {
    // ...
}
```

```java
// 新增一个仅用于测试的访问方法，但没有注解
boolean isRetryable() {
    return this.retryCount < MAX_RETRY;
}
```

### Good（添加 @VisibleForTesting 注解）

```kotlin
import com.google.common.annotations.VisibleForTesting

@VisibleForTesting
internal fun calculateDiscount(price: BigDecimal): BigDecimal {
    // ...
}
```

```java
import com.google.common.annotations.VisibleForTesting;

@VisibleForTesting
boolean isRetryable() {
    return this.retryCount < MAX_RETRY;
}
```

该注解的习惯用法还包括：如果方法仅因为测试而改为非 private，可以在注解中说明本来的意图：

```kotlin
@VisibleForTesting(otherwise = VisibleForTesting.PRIVATE)
internal fun calculateDiscount(price: BigDecimal): BigDecimal {
    // ...
}
```

# 21. 注释禁止描述变更历史

注释只能诚实描述当前代码在做什么（为什么、前置条件、特殊约束），禁止描述变更历史——不能写"已迁移至 X"、"原 Y 迁移至此"、"历史上…现已拆分…"、"以前干了什么"、"现在哪些事情不干了"等。

后续读者看不到变更前的代码，这类变更描述对他们起不到任何辅助作用，反而让人困惑（代码已移除，注释却在描述被移除的逻辑），还会因后续再次变更而过时失真。变更历史交给 git 承担，注释不重复。

### Bad（迁移后残留的变更描述）

```cpp
  AdjustFold(ui);

  // XX 配置的控制已迁移至 ApplyConfig（Render 第三步），
  // 作用于缓存，见 XxxAssembler::ApplyConfig。
  return final_ret;
```

```cpp
  // 填充 XX 配置（原 BuildContent 末尾逻辑迁移至此）
  FillConfig(lang, data.scene(), ui);
```

### Good

```cpp
  AdjustFold(ui);

  return final_ret;
```

```cpp
  FillConfig(lang, data.scene(), ui);
```

### Bad（函数作用变更后，注释描述变更而非当前职责）

```cpp
/**
 * @brief 组装结果
 * 历史上本函数同时做校验与组装，现已拆分，校验移至 ResultValidator。
 */
void AssembleResult(...);
```

### Good

```cpp
/**
 * @brief 组装结果
 */
void AssembleResult(...);
```

---

# 22. 函数名必须与其实际契约一致——动作型名字不得内含"是否执行"的判断

函数名是无声的契约。以动作动词命名的函数（`HideX` / `RemoveX` / `SendX`）承诺"调用它就执行该动作"。不要把"是否要做"的判断塞进这类函数内部——一旦它变成条件 no-op，名字就在撒谎：调用处看到 `HideX(...)` 会以为隐藏一定发生，实际可能什么都没做。

两种修法：

1. **（推荐）判断上移到调用方，函数只保留纯执行**。名字不变即诚实，且判断与执行分离，职责更清晰。
2. **（次选）若判断必须留在函数内，把条件性写进名字**：`…IfNeeded` / `Maybe…`。可行，但"是否做"和"怎么做"仍揉在一起，仅在无法上移判断时使用。

### Bad（名字承诺动作，内部却判断、可能 no-op）

```cpp
// 名字承诺"执行隐藏"，实际却先读配置判断、可能什么都不做
void HideBanner(const std::string& scene, Ui& ui) {
  const auto& cfg = GetConfig(scene);
  const bool show = cfg.has_show_banner() ? cfg.show_banner() : true;
  if (show) {
    return;  // 名字叫 Hide，这里却不动手
  }
  // ... 真正的移除逻辑
}

// 调用方
HideBanner(data.scene(), ui);
```

### Good（推荐：判断上移，函数只执行）

```cpp
void ApplyConfig(...) {
  // ...
  const bool show = cfg.has_show_banner() ? cfg.show_banner() : true;
  if (!show) {
    HideBanner(ui);  // 调用它就一定执行隐藏
  }
}

// 只执行移除，不做判断——名字与行为一致
void HideBanner(Ui& ui) {
  // ... 真正的移除逻辑
}
```

### Good（次选：名字显式表达条件性）

```cpp
// 若判断必须留在函数内，名字要带上条件语义
void HideBannerIfNeeded(const std::string& scene, Ui& ui);
void MaybeHideBanner(const std::string& scene, Ui& ui);
```

---

# 23. guard 之后只跟单个高内聚动作时，不要用 early return 截断控制流

规则 #6 推荐优先 early return，但有适用边界：early return 的价值在于 guard 之后还有**一长串代码**时压平嵌套。当 guard 之后只跟**一个高内聚动作**时，early return 反而有害——应该把动作收进 `if (!cond) { … }` 分支。

为什么此时 early return 不好：

- 动作孤零零落在 `return` 之后，读者要回头确认它属于"不 return"的分支，控制流隐晦。
- 更危险的是：后续若在动作之后追加**必须执行**的步骤，会被 guard 的 `return` 意外跳过，且不易察觉。

判断标准：guard 后是一长串流程 → early return；guard 后是单个内聚动作 → 收进 if 分支。

### Bad（单个内聚动作用 early return 截断）

```cpp
void ApplyConfig(...) {
  FillHeader(lang, data.scene(), ui);
  // ... 读配置得 show_banner
  if (show_banner) {
    return;            // 截断
  }
  HideBanner(ui);   // 孤零零落在 return 之后
}
```

### Good（内聚动作收进 if 分支）

```cpp
void ApplyConfig(...) {
  FillHeader(lang, data.scene(), ui);
  // ... 读配置得 show_banner
  if (!show_banner) {
    HideBanner(ui);   // 内聚动作收进分支，意图局部且显式
  }
}
```

