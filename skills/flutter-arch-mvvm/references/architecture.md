# Flutter 组件化 + MVVM 项目架构规范

**适用平台：** Flutter（Dart）  
**核心思想：** 四层纵向分层 + 业务横向模块化 + 模块内 MVVM + UI 能力隔离  
**定位：** 面向中大型 Flutter 项目的长期迭代规范，重点降低理解成本、稳定模块边界、避免过度抽象。

## 目录

- [1. 架构总览](#1-架构总览)
- [2. 推荐基线目录结构](#2-推荐基线目录结构)
- [3. 四层职责](#3-四层职责)
- [4. UI 隔离规则](#4-ui-隔离规则)
- [5. MVVM 职责](#5-mvvm-职责)
- [6. 目录拆分规则](#6-目录拆分规则)
- [7. 依赖规则](#7-依赖规则)
- [8. 执行判定矩阵](#8-执行判定矩阵)
- [9. 页面布局与间距规则](#9-页面布局与间距规则)
- [10. 命名规范](#10-命名规范)
- [11. 路由规范](#11-路由规范)
- [12. 重构策略](#12-重构策略)
- [13. 代码示例](#13-代码示例)
- [14. Code Review 清单](#14-code-review-清单)

## 1. 架构总览

项目采用四层纵向依赖结构：

| 层级 | 职责 |
| --- | --- |
| `application` | 应用装配层，负责入口、全局配置、初始化、启动任务、路由、应用根容器和跨模块编排 |
| `business` | 业务模块层，负责项目内强业务模块和页面流程 |
| `capability` | 能力组件层，负责公司或产品内可复用、但可能带项目定制的能力 |
| `foundation` | 基础能力层，负责理论上可直接开源、放到任意 Flutter 项目都可用的底层能力 |

依赖方向必须单向：

```text
application -> business -> capability -> foundation
```

业务模块内部采用 MVVM：

```text
View -> ViewModel -> Repository -> Model
```

核心边界：

- 上层可以依赖下层，下层不能依赖上层。
- 同级业务模块不能直接互相 import，跨业务协作通过路由、`application` 编排层或明确的公共能力完成。
- 基础能力保持简单、稳定、可组合，不能为了某个业务场景新增专用接口。
- `capability` 和 `foundation` 都必须将 UI 能力独立隔离，不允许功能性能力模块夹带 UI 组件。

## 2. 推荐基线目录结构

下面结构是职责释义和推荐起点，不是目录上限。可以新增目录，但新增目录必须有清晰职责，并遵守四层依赖、MVVM 边界和 UI 隔离规则。

```text
lib/
├── main.dart
├── application/
│   ├── config/
│   │   ├── app_config.dart
│   │   └── router_config.dart
│   ├── launch_task/
│   │   ├── launch_task.dart
│   │   └── launch_task_runner.dart
│   └── pages/
│       └── root_page.dart
├── business/
│   ├── home/
│   │   ├── page/
│   │   │   └── home_page.dart
│   │   ├── view/
│   │   │   ├── home_item_view.dart
│   │   │   └── home_navigation_bar.dart
│   │   ├── view_model/
│   │   │   └── home_view_model.dart
│   │   ├── model/
│   │   │   └── home_model.dart
│   │   └── repository/
│   │       └── home_repository.dart
│   └── user/
│       ├── profile/
│       │   ├── page/
│       │   ├── view/
│       │   ├── view_model/
│       │   ├── model/
│       │   └── repository/
│       ├── settings/
│       │   ├── page/
│       │   ├── view/
│       │   ├── view_model/
│       │   └── model/
│       └── security/
│           ├── bind_phone/
│           │   ├── page/
│           │   ├── view/
│           │   └── view_model/
│           └── change_pwd/
│               ├── page/
│               ├── view/
│               └── view_model/
├── capability/
│   ├── pay/
│   │   ├── api/
│   │   ├── impl/
│   │   └── model/
│   ├── share/
│   ├── push/
│   ├── update/
│   └── ui/
│       ├── theme/
│       ├── widget/
│       └── dialog/
└── foundation/
    ├── network/
    │   ├── dio_config.dart
    │   └── base_api.dart
    ├── image/
    ├── router/
    ├── db/
    ├── utils/
    ├── base/
    │   ├── base_view.dart
    │   └── base_view_model.dart
    └── ui/
        ├── primitive/
        ├── layout/
        └── feedback/
```

`base/` 是可选目录。是否引入 `BaseView`、`BaseViewModel` 等基类，取决于团队状态管理方案和项目复杂度，不应为了套结构强制添加。

## 3. 四层职责

### 3.1 `foundation`：基础能力层

`foundation` 只承载与公司、团队、项目无关的底层能力。它可以封装第三方库，也可以沉淀通用基础设施，但不能包含业务流程、用户分层、活动规则、品牌规则、项目配置或临时场景。

核心标准：`foundation` 理论上可以直接开源，放到任意 Flutter 项目中都应可用。

典型内容：

- 网络基础封装、请求基类、拦截器基础能力。
- 图片加载、数据库、路由基础工具。
- 通用工具类、扩展方法。
- 通用状态基础类，如可选的 `BaseViewModel`。
- 无项目和公司耦合的 UI 原语、布局工具和反馈组件，统一放入 `foundation/ui/`。

判断标准：

- 换一个公司或项目仍然成立，才适合放入 `foundation`。
- 如果接口名、参数或默认行为里出现业务名、活动名、用户等级、具体页面名、公司品牌或项目配置，通常不属于 `foundation`。
- 如果一个能力需要读取当前项目的业务状态或服务端约定，通常不属于 `foundation`。

### 3.2 `capability`：能力组件层

`capability` 封装公司或产品内可复用能力，通常依赖 `foundation`。它比 `foundation` 更贴近当前项目，允许存在项目定制；迁移到其他公司的项目时，可能需要根据 UI 规范、业务流程或服务端契约做修改。

典型内容：

- 支付、分享、推送、定位、统计、版本更新。
- 能在多个业务模块复用的产品能力。
- 与公司或产品设计规范相关的可复用 UI 能力，统一放入 `capability/ui/`。
- 对外接口放在 `api/`，内部细节放在 `impl/`。

约束：

- 不写具体页面业务流程。
- 不为单个业务页面泄漏内部实现或新增临时接口。
- 功能模块不夹带 UI 组件；例如 `pay/` 不直接放支付按钮、支付弹窗，相关 UI 应进入 `capability/ui/` 或业务页面。
- 对外 API 必须说明调用契约、错误语义、副作用和边界条件。

### 3.3 `business`：业务模块层

`business` 是项目强业务实现。模块按业务功能拆分，例如 `home`、`user`、`login`、`order`。模块内部维护自己的 UI、状态、逻辑和数据入口。

规范：

- 单页面模块直接包含 `page/`、`view/`、`view_model/`、`model/`、`repository/`。
- 多页面模块按子页面或子功能继续拆一级目录。
- 子模块最多嵌套一级，避免路径过深。
- 业务模块之间不直接 import，通过路由或 `application` 层编排协作。
- 业务页面专属 UI 留在本模块 `view/`，不要下沉到 `capability/ui/`。

### 3.4 `application`：应用装配层

`application` 负责把模块组合成完整应用，但不承载具体业务实现。

典型内容：

- 启动配置、主题、环境、日志配置。
- 路由总配置和根页面。
- 启动任务编排，例如恢复登录态、初始化 SDK、拉取远程配置、缓存预热。
- 应用初始化、监听和跨模块编排。
- 主 Tab 容器、应用 Shell。

约束：

- 不写页面业务细节。
- 不直接访问底层实现细节。
- 只承担全局组装和协调职责。

## 4. UI 隔离规则

`capability` 和 `foundation` 都需要把 UI 单独隔离出来，避免功能能力和展示能力互相污染。

### 4.1 `foundation/ui`

`foundation/ui` 放无项目、无品牌、无业务耦合的 UI 基础能力。它应当像 `foundation` 其他能力一样，理论上可以直接开源。

适合放入：

- 基础布局工具，如通用 gap、safe area 辅助、约束布局工具。
- 通用反馈组件，如无品牌语义的 loading、empty、error primitives。
- 无业务语义的交互基础组件。

不适合放入：

- 当前产品设计语言绑定很强的按钮、卡片、弹窗。
- 带业务文案、活动规则、品牌色、接口请求或埋点的组件。
- 需要依赖 `business` 或 `capability` 的 UI。

### 4.2 `capability/ui`

`capability/ui` 放公司或产品内可复用 UI 能力。它可以使用当前项目的设计规范，也可以服务多个业务模块，但不能绑死到某个具体页面流程。

适合放入：

- 产品统一按钮、卡片、弹窗、Toast、空态、加载态。
- 与公司设计系统绑定的主题、颜色、字体、图标封装。
- 多个业务模块都会使用的可复用 UI 表达。

不适合放入：

- 某个页面专用的 section、列表项、导航条。
- 只为一次活动或单个业务流程服务的 UI。
- 支付、分享、推送等功能模块内部私自放置的 UI 组件。

### 4.3 功能模块和 UI 模块的协作

- 功能模块提供能力接口，例如 `pay/api/pay_service.dart`。
- UI 模块提供展示组件，例如 `capability/ui/dialog/payment_result_dialog.dart`。
- 业务页面负责组合功能能力和 UI 表达，避免能力模块主动创建页面 Widget。
- 如果功能流程必须触发 UI 副作用，优先返回结果或事件，由 View 层处理弹窗、Toast、导航。

## 5. MVVM 职责

### 5.1 View

View 包括 `page/` 中的页面级 Widget 和 `view/` 中的页面内子组件。

职责：

- 渲染 UI。
- 响应用户事件并调用 ViewModel。
- 监听 ViewModel 状态并刷新界面。
- 创建 Widget，组织布局和交互。

禁止：

- 写业务流程。
- 直接做网络、数据库或缓存操作。
- 在多个位置维护同一份业务状态。

### 5.2 ViewModel

ViewModel 是页面逻辑和状态变更入口。

职责：

- 管理页面状态。
- 承接用户意图和页面事件。
- 调用 Repository 获取或提交数据。
- 暴露 View 可观察的状态。
- 将 UI 副作用表达为状态、事件或返回结果，由 View 决定具体展示方式。

禁止：

- 持有 `BuildContext`。
- 创建或返回 Widget。
- 直接操作 Navigator、Dialog、Toast 等 UI 细节。

### 5.3 Repository

Repository 是数据入口，不是业务决策层。

职责：

- 封装网络请求、本地缓存、数据库访问。
- 聚合多数据源并转换为模块需要的数据结构。
- 屏蔽数据来源细节。

禁止：

- 写页面业务流程。
- 依赖 View 或 ViewModel。
- 保存可由上层状态稳定推导的展示状态。

### 5.4 Model

Model 负责数据结构定义。

职责：

- 定义实体、DTO、值对象和枚举。
- 承载序列化、反序列化所需的结构代码。

禁止：

- 写页面业务逻辑。
- 持有 UI 状态或流程状态。

## 6. 目录拆分规则

1. 一个页面或子功能对应一个独立文件夹。
2. `page/` 只放路由可打开的完整页面。
3. `view/` 放页面内子组件、列表项、卡片、导航栏、分段控件等。
4. `capability` 和 `foundation` 的功能模块不放 UI；UI 进入同层级 `ui/`。
5. 公共代码放在模块根目录或当前模块内明确命名的目录中。
6. 子模块最多嵌套一级。
7. 所有 Widget 只在 Page/View 层或明确的 `ui/` 层创建。
8. ViewModel 只处理逻辑和状态，不做 UI。

## 7. 依赖规则

允许：

- `application` 依赖 `business`、`capability`、`foundation`。
- `business` 依赖 `capability`、`foundation`。
- `capability` 依赖 `foundation`。
- 同一业务模块内部按 MVVM 方向协作。
- `capability/ui` 依赖 `foundation/ui` 或其他 `foundation` 基础能力。

禁止：

- `foundation` 依赖 `capability`、`business` 或 `application`。
- `capability` 依赖 `business` 或 `application`。
- `business` 依赖 `application`。
- 业务模块之间直接 import。
- ViewModel 依赖 View 或 Widget。
- 功能能力模块反向依赖同层级 `ui/` 来主动创建界面。

## 8. 执行判定矩阵

| 场景 | 放置或处理方式 |
| --- | --- |
| 新增应用启动、路由总表、环境切换 | `application/` |
| 新增有明确顺序的启动任务，如恢复登录态、缓存预热、远程配置、SDK 初始化 | `application/launch_task/` |
| 新增项目内页面或业务流程 | `business/<module>/` |
| 新增页面内列表项、卡片、导航条 | 当前业务模块 `view/` |
| 新增多个业务共用的产品功能，如支付、分享、推送 | `capability/<name>/` |
| 新增多个业务共用的产品 UI，如统一弹窗、品牌按钮 | `capability/ui/` |
| 新增可开源的技术能力，如网络基类、数据库工具、通用扩展 | `foundation/<name>/` |
| 新增可开源的通用 UI 原语或布局辅助 | `foundation/ui/` |
| 新增只服务一次活动或单个页面的能力 | 留在 `business`，不要提升为通用 API |
| 新增 API 只是组合已有基础能力 | 不新增基础 API，在业务或编排层组合 |
| 业务模块之间需要协作 | 路由跳转、`application` 编排，或抽象成 `capability` |
| ViewModel 需要弹窗、Toast、导航 | 暴露状态或事件，由 View 执行 UI 副作用 |

## 9. 页面布局与间距规则

页面间距要遵循「谁渲染区域，谁负责边界」。

强制规则：

1. 页面级视图必须有自己的上下区域，不能只有底部间距而顶部贴边。
2. 一级 Section 必须有自己的上下留白，不能把顶部空间外包给前一个 Section 的底部间距。
3. 滚动容器必须显式处理顶部和底部 padding。
4. 底部为 TabBar 或 SafeArea 预留更多空间时，顶部仍需保留明确区域间距。
5. 提交前检查视觉节奏，避免「上紧下松」或「下紧上松」。

## 10. 命名规范

### 10.1 文件夹命名

使用小写下划线。

| 类型 | 示例 |
| --- | --- |
| 架构分层 | `application`、`business`、`capability`、`foundation` |
| 业务模块 | `home`、`user`、`login`、`order` |
| 子模块 | `profile`、`settings`、`security`、`bind_phone` |
| 固定分层 | `page`、`view`、`view_model`、`model`、`repository` |
| 能力组件 | `pay`、`share`、`push`、`update` |
| 基础能力 | `network`、`db`、`router`、`utils`、`base` |
| 启动任务 | `launch_task` |
| UI 隔离目录 | `ui`、`theme`、`widget`、`dialog`、`primitive`、`layout`、`feedback` |

### 10.2 文件命名

使用小写下划线，见名知意。

| 类型 | 示例 |
| --- | --- |
| 页面 | `home_page.dart` |
| 视图模型 | `home_view_model.dart` |
| 数据模型 | `home_model.dart` |
| 仓库 | `home_repository.dart` |
| 工具 | `date_utils.dart` |
| 基类 | `base_view_model.dart` |

### 10.3 类命名

使用大驼峰。

| 类型 | 示例 |
| --- | --- |
| 页面 | `HomePage` |
| ViewModel | `HomeViewModel` |
| Model | `HomeModel` |
| Repository | `HomeRepository` |
| 基类 | `BaseViewModel` |

### 10.4 变量和方法命名

使用小驼峰。

| 类型 | 示例 |
| --- | --- |
| 变量 | `userInfo`、`isLoading`、`homeRepository` |
| 方法 | `fetchData()`、`loadUserInfo()`、`showToast()` |
| 常量 | `kAppName`、`kRouteHome` |

### 10.5 Page / View 命名

页面：

- 文件固定以 `_page.dart` 结尾。
- 类固定以 `Page` 结尾。
- 示例：`home_page.dart` -> `HomePage`。

视图：

- 常规展示型视图以 `View` 结尾，例如 `home_item_view.dart` -> `HomeItemView`。
- 功能型视图以具体功能结尾，例如 `home_navigation_bar.dart` -> `HomeNavigationBar`。
- 禁止使用 `widget` 作为业务 View 文件或类后缀，例如 `home_item_widget.dart`、`HomeItemWidget`。

## 11. 路由规范

路由路径使用小写路径段：

```text
/home
/user/profile
/user/security/change_pwd
```

示例：

```dart
// application/config/router_config.dart
import 'package:go_router/go_router.dart';
import '../../business/home/page/home_page.dart';
import '../../business/user/profile/page/profile_page.dart';

final router = GoRouter(
  initialLocation: '/',
  routes: [
    GoRoute(path: '/', builder: (_, __) => const HomePage()),
    GoRoute(path: '/profile', builder: (_, __) => const ProfilePage()),
  ],
);
```

## 12. 重构策略

架构重构优先小步迁移，避免一次性大搬家。

推荐顺序：

1. 先确认层级命名、目录目标和迁移范围。
2. 先修依赖方向和业务模块互相 import 的问题。
3. 再拆 `foundation` 与 `capability` 的职责，把不满足开源标准的内容从 `foundation` 移走。
4. 再隔离 `capability/ui` 和 `foundation/ui`，把功能模块里的 UI 搬到同层级 `ui/` 或业务 `view/`。
5. 再整理 MVVM 边界，移除 ViewModel 的 `BuildContext`、Widget、导航和弹窗操作。
6. 再治理状态来源，删除可推导状态和重复缓存。
7. 最后处理命名、目录深度、布局间距和注释。

迁移约束：

- 推荐基线目录结构不是目录上限；新增目录必须说明职责边界和依赖方向。
- 不为了目录漂亮而移动无关文件。
- 每次迁移都要保持可运行状态。
- 对外 API 变更必须同步更新调用方和注释。
- 旧目录到新目录的迁移应优先保持行为不变，再做职责细化。

## 13. 代码示例

以下示例用于说明 MVVM 分层和依赖关系。状态管理可替换为 Provider、Riverpod、GetX、Bloc 或项目现有方案；不要为了示例而强制引入基类。

### 13.1 `foundation/base/base_view_model.dart`

```dart
import 'package:flutter/foundation.dart';

/// 页面 ViewModel 的基础状态能力。
///
/// 只提供通用 loading 状态和通知能力，不包含业务逻辑。
class BaseViewModel extends ChangeNotifier {
  bool _loading = false;

  bool get loading => _loading;

  void setLoading(bool value) {
    if (_loading == value) {
      return;
    }
    _loading = value;
    notifyListeners();
  }
}
```

### 13.2 `business/home/model/home_model.dart`

```dart
/// 首页展示数据。
class HomeModel {
  const HomeModel({
    required this.title,
    required this.content,
  });

  final String title;
  final String content;
}
```

### 13.3 `business/home/repository/home_repository.dart`

```dart
import '../model/home_model.dart';

/// 首页数据入口。
///
/// 负责隐藏数据来源；调用方不需要知道数据来自网络、缓存还是本地 Mock。
class HomeRepository {
  Future<HomeModel> fetchHomeData() async {
    await Future<void>.delayed(const Duration(seconds: 1));
    return const HomeModel(
      title: '首页标题',
      content: '这是首页内容',
    );
  }
}
```

### 13.4 `business/home/view_model/home_view_model.dart`

```dart
import 'package:xxx/foundation/base/base_view_model.dart';
import '../model/home_model.dart';
import '../repository/home_repository.dart';

/// 首页页面逻辑入口。
///
/// 负责加载首页数据并维护页面状态，不创建 Widget，也不持有 BuildContext。
class HomeViewModel extends BaseViewModel {
  HomeViewModel(this._repository);

  final HomeRepository _repository;
  HomeModel? _model;

  HomeModel? get model => _model;

  Future<void> loadData() async {
    setLoading(true);
    try {
      _model = await _repository.fetchHomeData();
    } finally {
      setLoading(false);
    }
    notifyListeners();
  }
}
```

### 13.5 `business/home/page/home_page.dart`

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../repository/home_repository.dart';
import '../view/home_item_view.dart';
import '../view_model/home_view_model.dart';

/// 首页路由页面。
class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (_) => HomeViewModel(HomeRepository())..loadData(),
      child: Scaffold(
        appBar: AppBar(title: const Text('首页')),
        body: Consumer<HomeViewModel>(
          builder: (_, viewModel, __) {
            if (viewModel.loading) {
              return const Center(child: CircularProgressIndicator());
            }

            return SingleChildScrollView(
              padding: const EdgeInsets.fromLTRB(16, 16, 16, 32),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.stretch,
                children: [
                  if (viewModel.model != null)
                    HomeItemView(model: viewModel.model!),
                ],
              ),
            );
          },
        ),
      ),
    );
  }
}
```

## 14. Code Review 清单

### 14.1 审查优先级

按严重程度优先审查：

1. 依赖方向违规，例如 `foundation` 依赖 `business`，或 `business` 依赖 `application`。
2. `foundation` 不满足可开源标准，包含公司、产品、项目或业务耦合。
3. `capability` 或 `foundation` 的功能模块混入 UI。
4. 业务模块之间直接 import。
5. ViewModel 持有 `BuildContext`、创建 Widget、直接导航或弹窗。
6. Repository 写页面业务流程或保存展示状态。
7. 状态来源不唯一、重复保存派生状态、状态变更入口不清晰。
8. 命名、目录深度、布局间距和注释问题。

### 14.2 架构边界

- `application` 是否只负责入口、配置、初始化、启动任务、路由和编排？
- `business` 模块是否没有直接依赖其他业务模块？
- `capability` 是否只暴露稳定能力，内部实现是否隔离？
- `capability` 是否允许项目定制但没有绑死到单个业务页面？
- `foundation` 是否没有业务逻辑、公司规则、产品规则和页面场景？
- `foundation` 是否理论上可以直接开源并放入任意 Flutter 项目？

### 14.3 UI 隔离

- `capability` 中支付、分享、推送等功能模块是否没有夹带 UI 组件？
- `foundation` 中网络、数据库、路由等基础模块是否没有夹带 UI 组件？
- 产品设计系统 UI 是否集中在 `capability/ui/`？
- 通用 UI 原语是否集中在 `foundation/ui/`？
- 业务页面专属 UI 是否留在业务模块 `view/`？

### 14.4 MVVM

- View 是否只负责 UI 渲染和事件转发？
- ViewModel 是否是状态和页面逻辑唯一入口？
- ViewModel 是否没有 `BuildContext` 和 Widget？
- Repository 是否只负责数据访问和聚合？
- Model 是否只表达数据结构？

### 14.5 状态

- 是否存在可推导状态被重复保存？
- 同一状态是否只有一个可信来源？
- 状态变更是否有明确入口？
- 失效状态是否及时清理？

### 14.6 命名和目录

- 顶层目录是否使用 `application`、`business`、`capability`、`foundation`？
- 启动任务是否放在 `application/launch_task/`？
- 页面是否使用 `xxx_page.dart` / `XxxPage`？
- 业务视图是否避免使用 `widget` 后缀？
- 子模块是否最多嵌套一级？
- 文件夹是否使用小写下划线？

### 14.7 布局

- 页面、一级 Section、滚动容器是否各自处理上下间距？
- SafeArea、TabBar、底部操作区是否没有破坏顶部节奏？
- 是否存在只靠上一个兄弟节点底部 margin 撑开下一个区域的情况？
