# 访问控制 / 作用域权限

> 引用自 `SKILL.md`。Swift 5 修饰符从严到宽：`private` < `fileprivate` < `internal`（默认） < `public` < `open`。

## 一、默认最严格原则（核心）

**每个变量、方法、类型，默认按"被最少的人看到"来标修饰符**，向上放宽到刚好够用为止。

判定流程：

| 谁会用到？ | 修饰符 |
|---|---|
| 只在当前类型内部 | `private` |
| 当前文件内（含 extension） | `private`（Swift 4+ 已扩展到同文件 extension） |
| 跨文件，仅同一辅助类型用 | `fileprivate` |
| 整个模块内（含主工程） | 不写（默认 internal） |
| Pod 等其他模块要调用 | `public` |
| Pod 调用方还要继承/override | `open` |

```swift
class JacoRoomListViewController: ViewController {

    private let disposeBag = DisposeBag()              // 只本类用 → private
    private lazy var titleLabel = UILabel()            // 同上
    private var pageIndex: Int = 1                     // 内部分页状态 → private

    func enterRoom(lid: String) { }                    // 跨页面跳转才调用 → internal（不写）

    private func configUI() { }                        // 只 viewDidLoad 调 → private
    private func bindViewModel() { }                   // 同上
}
```

**反例**：
```swift
// ❌ 全部不写访问控制 = 全是 internal，过度暴露
class JacoRoomListViewController: ViewController {
    let disposeBag = DisposeBag()
    var pageIndex: Int = 1
    func configUI() { }
}
```

## 二、`private(set)`：外部只读、内部可写

ViewModel / Manager 类的状态属性高频场景：外部需要读，但只能内部改。

```swift
class JacoRoomListViewModel {
    // ✅ 外部能读 dataList，但不能 viewModel.dataList = []
    private(set) var dataList: [JacoRoomModel] = []
    private(set) var isLoading: Bool = false

    func loadData() {
        isLoading = true                    // 内部随便改
        // ...
        dataList = newList
    }
}

// 调用方
viewModel.dataList.count          // ✅ 能读
viewModel.dataList = []           // ❌ 编译报错
```

**别这样写**（冗余、易错）：
```swift
// ❌ 计算属性 + 私有存储 = 写了一堆样板
class JacoRoomListViewModel {
    private var _dataList: [JacoRoomModel] = []
    var dataList: [JacoRoomModel] { _dataList }
}
```

`private(set)` 同样可以是 `fileprivate(set)`、`internal(set)`、`public(set)`，按需选最严的。

## 三、不被继承的类必须加 `final`

`final` 表示"这个类/方法不能被继承/重写"：

- **防误用**：明确表达设计意图，子类化时编译器会拦
- **性能**：编译器可以去虚化（devirtualize），方法调用从动态派发变静态派发
- **重构友好**：类不被继承，重命名、改方法签名都更安全

**规则**：项目里 99% 的 class 不需要被继承，**默认加 `final`**，需要继承时再去掉。

```swift
final class JacoRoomListViewController: ViewController { }    // ✅
final class JacoUserModel: BaseObjcModel { }                  // ✅
final class JacoLoginManager { }                              // ✅
```

**例外**（不加 final）：
- 明确设计为基类的（`BaseViewController`、`BaseObjcModel`）
- 必须被 OC 子类化或 KVO（`@objc dynamic` 一般也要求非 final）

**只放开个别方法**：
```swift
class JacoBaseRoomViewController: ViewController {
    final func enterRoom(lid: String) { }    // 这个方法子类不能改
    func didEnterRoom() { }                  // 这个允许子类 override
}
```

## 四、Pod 公开 API 细则

Pod / Framework 是独立 module，主工程能看到的**只有 `public` / `open` 的符号**。常踩的坑：

### 4.1 类要可见 → 类标 public，public 类的成员**默认仍是 internal**

```swift
// JacoRoomKit Pod 里
public class JacoRoomPlayer {

    public var currentLid: String = ""        // ✅ 外部可见
    var bufferSize: Int = 0                   // ❌ internal，外部看不到

    public func play() { }                    // ✅ 外部可调
    func reset() { }                          // ❌ 外部看不到
}
```

**规则**：public 类的成员必须再标 `public` 一遍才对外可见，不要假设"父类 public 就行"。

### 4.2 public 类的 `init` 必须显式 `public`

Swift 不会推断 init 的访问级别，**不写就是 internal**，结果主工程拿不到这个类的实例。

```swift
public class JacoRoomPlayer {
    public var lid: String

    public init(lid: String) {        // ✅ 必须显式 public
        self.lid = lid
    }
}

// 反例
public class JacoRoomPlayer {
    init(lid: String) { ... }         // ❌ internal，主工程报错 "cannot find init"
}
```

### 4.3 public 函数的参数类型必须 ≥ public

签名里出现的所有类型都得**至少和函数本身一样可见**，否则会触发一连串编译错误：

```swift
// ❌ JacoRoomConfig 是 internal，但函数是 public，编译报错
struct JacoRoomConfig { }
public func setup(config: JacoRoomConfig) { }

// ✅ 两个一起 public
public struct JacoRoomConfig { }
public func setup(config: JacoRoomConfig) { }

// ✅ 如果 config 是内部细节，别让它出现在 public 签名上
public func setup(lid: String, maxCount: Int) { }
```

返回类型、闭包参数类型、属性类型同理。

### 4.4 `public` vs `open`：默认 public，禁止外部继承

- `public class`：外部能用，**不能继承**、**不能 override 方法**
- `open class`：外部能用，**能继承**、**能 override**

**默认用 `public`**（更严格），只有真的需要业务方子类化的才用 `open`。

```swift
public final class JacoRoomPlayer { }    // ✅ 推荐：public + final，对外可见且禁止扩展
public class JacoRoomViewController { }  // ✅ 对外可见，但子类化要在同模块内
open class JacoBasePlayerView { }        // ⚠ 仅当外部业务方需要继承时
```

### 4.5 不要泄漏内部类型

Pod 对外接口应该是稳定的契约，**实现细节不要从 public API 漏出去**：

```swift
// ❌ 把内部状态 enum 暴露给外部
public enum JacoInternalPlayState { case prepare, buffering, ... }
public class JacoRoomPlayer {
    public var state: JacoInternalPlayState   // 外部依赖了内部细节，以后改不动
}

// ✅ 用 public 的语义状态，内部状态自己消化
public enum JacoPlayState { case idle, playing, paused, ended }
public class JacoRoomPlayer {
    public private(set) var state: JacoPlayState = .idle
    private var internalState: JacoInternalPlayState = .prepare   // 内部细节藏起来
}
```

## 五、规则汇总（速查）

- 默认从 `private` 起步，按需放宽
- ViewModel / Manager 的状态属性 → `private(set)`
- 所有不需要被继承的 class → `final class`
- Pod 内对外类型：`public class` + 成员显式 `public` + `public init` + 参数类型 ≥ public
- Pod 不需要外部子类化的 class → `public final class`，需要的才用 `open`
- 不要把内部状态/枚举/类型从 public API 暴露给主工程
