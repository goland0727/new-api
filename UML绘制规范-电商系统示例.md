# 标准大厂 UML 图绘制规范 - 电商系统示例

## 1. 用例图 (Use Case Diagram)

### 用途
描述系统功能和用户交互，展示系统边界和参与者。

```mermaid
graph TB
    subgraph "电商系统"
        UC1((浏览商品))
        UC2((搜索商品))
        UC3((添加购物车))
        UC4((下单))
        UC5((支付))
        UC6((管理订单))
        UC7((管理商品))
        UC8((查看报表))
        UC9((处理退款))
        UC10((评价商品))
    end
    
    Customer[客户 👤]
    Admin[管理员 👨‍💼]
    Payment[支付系统 🏦]
    
    Customer --> UC1
    Customer --> UC2
    Customer --> UC3
    Customer --> UC4
    Customer --> UC5
    Customer --> UC6
    Customer --> UC10
    
    Admin --> UC7
    Admin --> UC8
    Admin --> UC9
    
    UC5 --> Payment
    
    UC3 -.包含.-> UC1
    UC4 -.包含.-> UC3
    UC5 -.扩展.-> UC4
```

### 绘制规范
- 参与者用小人图标表示
- 用例用椭圆表示
- 系统边界用矩形框表示
- 包含关系用虚线箭头 + <<include>>
- 扩展关系用虚线箭头 + <<extend>>

## 2. 类图 (Class Diagram)

### 用途
展示系统的静态结构，包括类、属性、方法和关系。

```mermaid
classDiagram
    class User {
        -Long userId
        -String username
        -String email
        -String password
        -Date createdAt
        +register()
        +login()
        +updateProfile()
    }
    
    class Customer {
        -String shippingAddress
        -String phoneNumber
        +browseProducts()
        +addToCart()
        +placeOrder()
    }
    
    class Admin {
        -String department
        -String role
        +manageProducts()
        +viewReports()
        +handleRefunds()
    }
    
    class Product {
        -Long productId
        -String name
        -String description
        -BigDecimal price
        -Integer stock
        -String category
        +updateStock()
        +updatePrice()
        +isAvailable()
    }
    
    class ShoppingCart {
        -Long cartId
        -List~CartItem~ items
        -BigDecimal totalAmount
        +addItem(Product, int)
        +removeItem(Long)
        +updateQuantity(Long, int)
        +calculateTotal()
        +clear()
    }
    
    class CartItem {
        -Long itemId
        -Product product
        -Integer quantity
        -BigDecimal subtotal
        +updateQuantity(int)
        +calculateSubtotal()
    }
    
    class Order {
        -Long orderId
        -String orderNumber
        -OrderStatus status
        -Date orderDate
        -BigDecimal totalAmount
        -String shippingAddress
        +calculateTotal()
        +updateStatus(OrderStatus)
        +cancel()
        +ship()
    }
    
    class OrderItem {
        -Long orderItemId
        -Product product
        -Integer quantity
        -BigDecimal price
        +calculateSubtotal()
    }
    
    class Payment {
        -Long paymentId
        -PaymentMethod method
        -PaymentStatus status
        -BigDecimal amount
        -Date paymentDate
        +processPayment()
        +refund()
        +verify()
    }
    
    class Review {
        -Long reviewId
        -Integer rating
        -String comment
        -Date reviewDate
        +edit()
        +delete()
    }
    
    User <|-- Customer : 继承
    User <|-- Admin : 继承
    Customer "1" --> "0..1" ShoppingCart : 拥有
    ShoppingCart "1" *-- "0..*" CartItem : 组合
    CartItem "0..*" --> "1" Product : 关联
    Customer "1" --> "0..*" Order : 下单
    Order "1" *-- "1..*" OrderItem : 组合
    OrderItem "0..*" --> "1" Product : 关联
    Order "1" --> "1" Payment : 包含
    Customer "1" --> "0..*" Review : 发表
    Review "0..*" --> "1" Product : 评价
    
    class OrderStatus {
        <<enumeration>>
        PENDING
        PAID
        SHIPPED
        DELIVERED
        CANCELLED
    }
    
    class PaymentMethod {
        <<enumeration>>
        CREDIT_CARD
        DEBIT_CARD
        ALIPAY
        WECHAT_PAY
        BANK_TRANSFER
    }
    
    class PaymentStatus {
        <<enumeration>>
        PENDING
        SUCCESS
        FAILED
        REFUNDED
    }
```

### 绘制规范
- 类用三段矩形表示（类名、属性、方法）
- 属性和方法的可见性：- 私有，+ 公共，# 保护，~ 包级
- 关系类型：
  - 继承：实线空心三角箭头
  - 实现：虚线空心三角箭头
  - 关联：实线箭头
  - 聚合：空心菱形
  - 组合：实心菱形
  - 依赖：虚线箭头

## 3. 序列图 (Sequence Diagram)

### 用途
描述对象之间的交互顺序，展示业务流程的时序关系。

```mermaid
sequenceDiagram
    participant C as 客户
    participant UI as 前端界面
    participant API as API网关
    participant PS as 商品服务
    participant CS as 购物车服务
    participant OS as 订单服务
    participant PAY as 支付服务
    participant INV as 库存服务
    
    C->>UI: 浏览商品
    UI->>API: GET /products
    API->>PS: 查询商品列表
    PS-->>API: 返回商品数据
    API-->>UI: 商品列表响应
    UI-->>C: 展示商品
    
    C->>UI: 添加商品到购物车
    UI->>API: POST /cart/items
    API->>CS: 添加购物车项
    CS->>PS: 验证商品信息
    PS-->>CS: 商品信息
    CS->>INV: 检查库存
    INV-->>CS: 库存充足
    CS-->>API: 添加成功
    API-->>UI: 成功响应
    UI-->>C: 更新购物车图标
    
    C->>UI: 提交订单
    UI->>API: POST /orders
    API->>OS: 创建订单
    OS->>CS: 获取购物车数据
    CS-->>OS: 购物车项目
    OS->>INV: 锁定库存
    INV-->>OS: 库存锁定成功
    OS->>PAY: 创建支付单
    PAY-->>OS: 支付单创建成功
    OS-->>API: 订单创建成功
    API-->>UI: 返回订单号
    UI-->>C: 跳转支付页面
    
    C->>UI: 确认支付
    UI->>API: POST /payments
    API->>PAY: 处理支付
    PAY->>PAY: 调用第三方支付
    alt 支付成功
        PAY-->>API: 支付成功
        API->>OS: 更新订单状态
        OS->>INV: 扣减库存
        INV-->>OS: 库存更新成功
        OS-->>API: 订单更新成功
        API-->>UI: 支付成功
        UI-->>C: 显示订单详情
    else 支付失败
        PAY-->>API: 支付失败
        API->>OS: 取消订单
        OS->>INV: 释放库存
        INV-->>OS: 库存释放成功
        OS-->>API: 订单取消
        API-->>UI: 支付失败
        UI-->>C: 提示重新支付
    end
```

### 绘制规范
- 参与者用矩形框表示
- 生命线用垂直虚线表示
- 同步消息用实线箭头
- 异步消息用开放箭头
- 返回消息用虚线箭头
- 激活框表示对象活跃期

## 4. 活动图 (Activity Diagram)

### 用途
描述业务流程或算法的工作流程，类似流程图但更规范。

```mermaid
graph TB
    Start([开始]) --> Browse[浏览商品]
    Browse --> Search{需要搜索?}
    Search -->|是| SearchProduct[搜索商品]
    Search -->|否| ViewDetail[查看商品详情]
    SearchProduct --> ViewDetail
    
    ViewDetail --> CheckStock{库存充足?}
    CheckStock -->|否| Notify[通知缺货]
    CheckStock -->|是| AddCart[添加到购物车]
    
    Notify --> End1([结束])
    
    AddCart --> ViewCart[查看购物车]
    ViewCart --> ModifyCart{修改购物车?}
    ModifyCart -->|是| UpdateItems[更新商品数量]
    UpdateItems --> ViewCart
    ModifyCart -->|否| Checkout[结算]
    
    Checkout --> Login{已登录?}
    Login -->|否| LoginPage[跳转登录]
    LoginPage --> FillAddress[填写收货地址]
    Login -->|是| FillAddress
    
    FillAddress --> SelectPayment[选择支付方式]
    SelectPayment --> ConfirmOrder[确认订单]
    ConfirmOrder --> CreateOrder[创建订单]
    
    CreateOrder --> Pay[支付]
    Pay --> PayResult{支付成功?}
    
    PayResult -->|是| UpdateOrder[更新订单状态]
    PayResult -->|否| CancelOrder[取消订单]
    
    UpdateOrder --> SendNotification[发送通知]
    CancelOrder --> ReleaseStock[释放库存]
    
    SendNotification --> End2([结束])
    ReleaseStock --> End3([结束])
```

### 绘制规范
- 开始/结束用圆角矩形
- 活动用矩形
- 判断用菱形
- 并发用粗横线（分叉和汇合）
- 泳道表示不同参与者的职责

## 5. 状态图 (State Diagram)

### 用途
描述对象的生命周期，展示状态转换。

```mermaid
stateDiagram-v2
    [*] --> 待支付: 创建订单
    
    待支付 --> 已取消: 超时未支付/用户取消
    待支付 --> 已支付: 支付成功
    
    已支付 --> 待发货: 确认收款
    已支付 --> 退款中: 申请退款
    
    待发货 --> 已发货: 商家发货
    待发货 --> 退款中: 申请退款
    
    已发货 --> 已签收: 用户签收
    已发货 --> 退货中: 申请退货
    
    已签收 --> 已完成: 确认收货/自动确认
    已签收 --> 售后中: 申请售后
    
    退款中 --> 已退款: 退款成功
    退款中 --> 已取消: 退款失败
    
    退货中 --> 已退款: 退货成功
    退货中 --> 已完成: 退货失败
    
    售后中 --> 已完成: 售后完成
    售后中 --> 已退款: 售后退款
    
    已取消 --> [*]
    已完成 --> [*]
    已退款 --> [*]
    
    note right of 待支付
        超时时间：30分钟
        可以修改订单
    end note
    
    note right of 已发货
        物流信息可查询
        不可修改订单
    end note
```

### 绘制规范
- 状态用圆角矩形表示
- 初始状态用实心圆
- 结束状态用双圆圈
- 转换用箭头表示，标注触发事件
- 可以包含进入/退出动作

## 6. 组件图 (Component Diagram)

### 用途
展示系统的物理架构，组件之间的依赖关系。

```mermaid
graph TB
    subgraph "前端层"
        Web[Web应用]
        Mobile[移动应用]
        Admin[管理后台]
    end
    
    subgraph "网关层"
        Gateway[API网关]
        Auth[认证服务]
        RateLimit[限流服务]
    end
    
    subgraph "业务服务层"
        UserService[用户服务]
        ProductService[商品服务]
        CartService[购物车服务]
        OrderService[订单服务]
        PaymentService[支付服务]
        InventoryService[库存服务]
        NotificationService[通知服务]
    end
    
    subgraph "数据层"
        UserDB[(用户数据库)]
        ProductDB[(商品数据库)]
        OrderDB[(订单数据库)]
        Redis[(Redis缓存)]
        MQ[消息队列]
    end
    
    subgraph "外部服务"
        ThirdPayment[第三方支付]
        SMS[短信服务]
        Email[邮件服务]
    end
    
    Web --> Gateway
    Mobile --> Gateway
    Admin --> Gateway
    
    Gateway --> Auth
    Gateway --> RateLimit
    Gateway --> UserService
    Gateway --> ProductService
    Gateway --> CartService
    Gateway --> OrderService
    
    UserService --> UserDB
    UserService --> Redis
    
    ProductService --> ProductDB
    ProductService --> Redis
    
    CartService --> Redis
    
    OrderService --> OrderDB
    OrderService --> MQ
    
    PaymentService --> ThirdPayment
    PaymentService --> MQ
    
    InventoryService --> ProductDB
    InventoryService --> Redis
    InventoryService --> MQ
    
    NotificationService --> SMS
    NotificationService --> Email
    NotificationService --> MQ
```

### 绘制规范
- 组件用矩形表示，可带组件图标
- 接口用圆圈或棒棒糖表示
- 依赖关系用虚线箭头
- 组件可以嵌套表示包含关系

## 7. 部署图 (Deployment Diagram)

### 用途
展示系统的物理部署架构。

```mermaid
graph TB
    subgraph "用户端"
        Browser[浏览器]
        MobileApp[手机APP]
    end
    
    subgraph "CDN"
        CDN1[CDN节点]
        CDN2[静态资源]
    end
    
    subgraph "负载均衡层"
        LB[Nginx负载均衡器]
    end
    
    subgraph "应用服务器集群"
        AppServer1[应用服务器1<br/>Tomcat]
        AppServer2[应用服务器2<br/>Tomcat]
        AppServer3[应用服务器3<br/>Tomcat]
    end
    
    subgraph "微服务集群"
        Service1[用户服务<br/>Spring Boot]
        Service2[商品服务<br/>Spring Boot]
        Service3[订单服务<br/>Spring Boot]
    end
    
    subgraph "缓存层"
        RedisCluster[Redis集群]
    end
    
    subgraph "消息中间件"
        Kafka[Kafka集群]
    end
    
    subgraph "数据库集群"
        Master[(MySQL主库)]
        Slave1[(MySQL从库1)]
        Slave2[(MySQL从库2)]
    end
    
    subgraph "监控系统"
        Prometheus[Prometheus]
        Grafana[Grafana]
        ELK[ELK Stack]
    end
    
    Browser --> CDN1
    MobileApp --> CDN1
    CDN1 --> LB
    
    LB --> AppServer1
    LB --> AppServer2
    LB --> AppServer3
    
    AppServer1 --> Service1
    AppServer1 --> Service2
    AppServer1 --> Service3
    
    Service1 --> RedisCluster
    Service2 --> RedisCluster
    Service3 --> Kafka
    
    Service1 --> Master
    Service2 --> Slave1
    Service3 --> Slave2
    
    Master --> Slave1
    Master --> Slave2
    
    Service1 --> Prometheus
    Service2 --> Prometheus
    Service3 --> Prometheus
    
    Prometheus --> Grafana
    AppServer1 --> ELK
```

### 绘制规范
- 节点用立方体表示
- 部署的组件用矩形表示
- 通信路径用实线连接
- 可以标注协议和端口

## 8. 时序图最佳实践示例

### 用户登录流程
```mermaid
sequenceDiagram
    participant User as 用户
    participant Client as 客户端
    participant Gateway as API网关
    participant Auth as 认证服务
    participant User_Service as 用户服务
    participant Cache as Redis缓存
    participant DB as 数据库
    
    User->>Client: 输入用户名密码
    Client->>Client: 本地校验
    Client->>Gateway: POST /api/login
    Gateway->>Auth: 转发登录请求
    
    Auth->>User_Service: 验证用户信息
    User_Service->>DB: 查询用户
    DB-->>User_Service: 用户数据
    
    alt 用户存在且密码正确
        User_Service-->>Auth: 验证成功
        Auth->>Auth: 生成JWT Token
        Auth->>Cache: 缓存Session
        Cache-->>Auth: 缓存成功
        Auth-->>Gateway: 返回Token
        Gateway-->>Client: 登录成功(Token)
        Client->>Client: 存储Token
        Client-->>User: 跳转首页
    else 用户不存在或密码错误
        User_Service-->>Auth: 验证失败
        Auth-->>Gateway: 认证失败
        Gateway-->>Client: 401 Unauthorized
        Client-->>User: 显示错误信息
    end
```

## UML 绘制总体规范

### 1. 命名规范
- **类名**：使用大驼峰命名法（PascalCase）
- **属性和方法**：使用小驼峰命名法（camelCase）
- **包名**：使用小写字母
- **常量**：使用全大写字母，下划线分隔

### 2. 图形规范
- 保持图形简洁，避免交叉线
- 合理使用颜色区分不同类型的元素
- 添加必要的注释说明
- 控制图形大小，确保可读性

### 3. 关系表示
- **依赖**（Dependency）：虚线箭头 -->
- **关联**（Association）：实线箭头 →
- **聚合**（Aggregation）：空心菱形 ◇→
- **组合**（Composition）：实心菱形 ◆→
- **继承**（Inheritance）：空心三角箭头 ▷→
- **实现**（Implementation）：虚线空心三角箭头 ▷-->

### 4. 文档规范
- 每个图都应有标题和说明
- 复杂的图需要添加图例
- 版本控制和更新记录
- 与代码保持同步

### 5. 工具推荐
- **在线工具**：Draw.io、Lucidchart、PlantUML
- **IDE插件**：IntelliJ IDEA UML插件、Visual Studio Code PlantUML
- **专业工具**：Enterprise Architect、Visual Paradigm、StarUML

### 6. 实践建议
1. **按需绘制**：不是所有项目都需要所有类型的UML图
2. **保持更新**：UML图应该随着代码的变化而更新
3. **团队规范**：制定团队统一的UML绘制规范
4. **适度细节**：避免过度设计，保持适当的抽象层次
5. **版本管理**：将UML图纳入版本控制系统

## 常见错误示例

### ❌ 错误示例
1. 类图中包含过多实现细节
2. 序列图消息顺序混乱
3. 使用非标准的符号表示
4. 图形过于复杂，难以理解
5. 命名不规范，含义不清

### ✅ 正确做法
1. 保持适当的抽象层次
2. 清晰的时序关系
3. 使用标准UML符号
4. 图形简洁，重点突出
5. 命名规范，含义明确

## 总结

UML是软件设计的重要工具，正确使用UML可以：
- 提高团队沟通效率
- 减少设计歧义
- 便于系统维护和扩展
- 作为技术文档的重要组成部分

在电商系统这个例子中，我们展示了各种UML图的实际应用，这些图形可以帮助团队更好地理解系统架构、业务流程和技术实现。记住，UML是为了更好地交流和设计，而不是为了画图而画图。