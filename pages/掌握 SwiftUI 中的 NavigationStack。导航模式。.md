- 15 Jun 2022
  SwiftUI 是声明式的数据驱动框架，允许我们通过定义屏幕上的数据渲染来构建复杂的用户界面。导航一直是该框架从第一天起的主要痛点。幸运的是，自 WWDC 22 以来，情况已经改变，SwiftUI 提供了新的数据驱动导航 API。本周我们将学习如何使用新的导航 API 来构建复杂的用户流程。
- 使用 Xcode 构建，使用 Helm 打包。
  一站式 macOS 应用程序，增强 App Store Connect 功能，通过 AI 动力工具加速你的应用更新、本地化和应用优化 (ASO)。现在试用享受 25% 折扣！
  基础
  首先，我必须提到旧的 NavigationView 已经弃用，我们应该使用新的 NavigationStack。让我们来看一个快速的例子。
- ```
  struct MasterView: View {
    let products: [Product]
    
    var body: some View {
        NavigationStack {
            List(products) { product in
                NavigationLink(product.title) {
                    ProductDetailView(product: product)
                }
            }
            .navigationTitle("Products")
        }
    }
  }
  struct ProductDetailView: View {
    let product: Product
    
    var body: some View {
        Text(product.title)
            .font(.title)
            .navigationTitle(product.title)
    }
  }
  ```
-
- 在上面的示例中，我们定义了一个简单的主从流程。我们在视图层次结构的根部放置了 NavigationStack。接下来，我们定义了一条消息列表，其中每条消息都提供了一个链接到该消息详细屏幕的链接。如你所见，我们仍然在这里使用旧的 NavigationLink 类型，它在这个用例中表现得非常好。
- 要了解更多 SwiftUI 的新功能，请参阅我发布的“WWDC22 之后的 SwiftUI 新特性”文章。
- The NavigationLink 类型增加了新的数据驱动能力。一个新的初始化器使我们能够创建一个绑定到某些值的链接。这里是我们之前示例使用新的数据驱动导航 API 重构后的版本。
- struct MasterView1: View {
    let products: [Product]
    
    var body: some View {
        NavigationStack {
            List(products) { product in
                NavigationLink(product.title, value: product)
            }
            .navigationTitle("Products")
            .navigationDestination(for: Product.self) { product in
                ProductDetailView(product: product)
            }
        }
    }
  }
  我们使用基于值的导航链接来引导用户通过应用。看看我们是如何将列表中的每个项目与特定值关联的。请注意，该值必须符合 Hashable 协议。接下来，我们使用 navigationDestination 视图修饰符为特定值定义一个目标视图。在当前示例中，我们只有一个类型的目标，但你可以通过应用多个 navigationDestination 视图修饰符来拥有你需要的任何数量的目标。
- struct MasterView2: View {
    let categories: [Category]
    let recentProducts: [Product]
    
    var body: some View {
        NavigationStack {
            List {
                Section("Categories") {
                    ForEach(categories) { category in
                        NavigationLink(value: category) {
                            Text(category.query)
                                .font(.headline)
                        }
                    }
                }
                
                Section("Recent") {
                    ForEach(recentProducts) { product in
                        NavigationLink(product.title, value: product)
                    }
                }
            }
            .navigationTitle("Home")
            .navigationDestination(for: Category.self) { category in
                CategoryView(category: category)
            }
            .navigationDestination(for: Product.self) { product in
                ProductDetailView(product: product)
            }
        }
    }
  }
  记得我们还有一个基于值的 NavigationLink 初始化器的另一个版本，允许我们为链接提供自定义标签。我们使用这个初始化器版本为类别链接提供标题字体。
- 放置规则
  您在视图层次结构中放置 navigationDestination 视图修饰符时应小心。放置 navigationDestination 视图修饰符有三条规则：
- The navigationDestination view modifier 应该在 NavigationStack 内部。
  不要在 List、ScrollView、LazyVStack 等懒加载容器的子项上放置 navigationDestination 视图修饰符。
  顶级导航 Destination 视图修饰符将始终覆盖同类型中最底层的一个。
  导航模式
  我喜欢将功能的导航流程保持在一个地方。这就是为什么我通常会实现 Navigator 模式，这样我可以以类型安全的方式处理导航。使用 SwiftUI 的新数据驱动导航 API 实现 Navigator 模式非常简单。首先，我们应该创建一个枚举类型来定义所有我们的应用/功能/模块路由。
- enum Route: Hashable {
    case product(Product)
    case category(Category)
  }
  接下来，我们应该使用 Route 枚举和新的值基导航链接。
- struct MasterView3: View {
    let categories: [Category]
    let recentProducts: [Product]
    
    var body: some View {
        List {
            Section("Categories") {
                ForEach(categories) { category in
                    NavigationLink(value: Route.category(category)) {
                        Text(category.query)
                    }
                }
            }
            
            Section("Recent") {
                ForEach(recentProducts) { product in
                    NavigationLink(value: Route.product(product)) {
                        Text(product.title)
                    }
                }
            }
        }
        .navigationTitle("Home")
    }
  }
  最后，我们可以放置单个 navigationDestination 视图修饰符并处理我们 Route 类型的所有情况。
- struct AppContainerView: View {
    @StateObject private var store = Store()
    
    var body: some View {
        NavigationStack {
            MasterView3(
                categories: store.categories,
                recentProducts: store.products
            )
            .navigationDestination(for: Route.self) { route in
                switch route {
                case let .category(category):
                    CategoryView(category: category)
                case let .product(product):
                    ProductDetailView(product: product)
                }
            }
        }
    }
  }
  结论
  我非常兴奋能在 SwiftUI 中使用新的数据驱动导航 API。如果你关注我的博客，你可能知道自从第一天起我就在等待这个功能。我将继续详细讲解新的数据驱动导航 API，下周我们将讨论深层链接。希望你喜欢这篇文章。欢迎在 Twitter 上关注我并就这篇文章提出你的问题。谢谢阅读，下周见！
- 最近的帖子
  Introducing UIGestureRecognizerRepresentable 协议在 SwiftUI 17 年 12 月 2024 日
  Text 字段增强在 SwiftUI 03 Dec 2024
  Xcode 中预览的功能 26 Nov 2024
  mecid
  Hi there! My name is Majid.
  I'm Swift developer 👨🏻‍💻SwiftUI addicted 🚀
  Creator of CardioBot, NapBot, FastBot and SugarBot.
  Feel free to follow me on Mastodon, Twitter or Github.
  Majid Jabrayilov © 2024. All rights reserved.