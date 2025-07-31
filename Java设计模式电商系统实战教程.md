# Java设计模式电商系统实战教程

## 目录
- [创建型模式](#创建型模式)
- [结构型模式](#结构型模式)
- [行为型模式](#行为型模式)

---

## 创建型模式

### 1. 单例模式 (Singleton Pattern)

**场景**: 订单号生成器、数据库连接池管理器

```java
/**
 * 订单号生成器单例 - 双重检查锁定
 * 参考：阿里巴巴订单系统
 */
public class OrderIdGenerator {
    private static volatile OrderIdGenerator instance;
    private final AtomicLong counter = new AtomicLong(0);
    
    private OrderIdGenerator() {}
    
    public static OrderIdGenerator getInstance() {
        if (instance == null) {
            synchronized (OrderIdGenerator.class) {
                if (instance == null) {
                    instance = new OrderIdGenerator();
                }
            }
        }
        return instance;
    }
    
    public String generateOrderId() {
        long timestamp = System.currentTimeMillis();
        long sequence = counter.incrementAndGet();
        return String.format("ORD%d%06d", timestamp, sequence % 1000000);
    }
}

/**
 * Redis连接池管理器 - 枚举单例
 * 参考：京东缓存架构
 */
public enum RedisConnectionManager {
    INSTANCE;
    
    private JedisPool jedisPool;
    
    RedisConnectionManager() {
        JedisPoolConfig config = new JedisPoolConfig();
        config.setMaxTotal(200);
        config.setMaxIdle(50);
        jedisPool = new JedisPool(config, "localhost", 6379);
    }
    
    public Jedis getConnection() {
        return jedisPool.getResource();
    }
}
```

### 2. 工厂方法模式 (Factory Method Pattern)

**场景**: 支付方式创建、物流服务创建

```java
/**
 * 支付服务抽象工厂
 * 参考：支付宝支付架构
 */
public abstract class PaymentServiceFactory {
    public abstract PaymentService createPaymentService();
    
    public PaymentResult processPayment(PaymentRequest request) {
        PaymentService service = createPaymentService();
        return service.pay(request);
    }
}

public interface PaymentService {
    PaymentResult pay(PaymentRequest request);
    boolean refund(String orderId, BigDecimal amount);
}

/**
 * 支付宝支付工厂
 */
public class AlipayServiceFactory extends PaymentServiceFactory {
    @Override
    public PaymentService createPaymentService() {
        return new AlipayService();
    }
}

/**
 * 微信支付工厂
 */
public class WechatPayServiceFactory extends PaymentServiceFactory {
    @Override
    public PaymentService createPaymentService() {
        return new WechatPayService();
    }
}

/**
 * 支付宝实现
 */
public class AlipayService implements PaymentService {
    @Override
    public PaymentResult pay(PaymentRequest request) {
        // 调用支付宝SDK
        System.out.println("Processing Alipay payment: " + request.getAmount());
        return new PaymentResult(true, "ALI" + System.currentTimeMillis());
    }
    
    @Override
    public boolean refund(String orderId, BigDecimal amount) {
        System.out.println("Alipay refund for order: " + orderId);
        return true;
    }
}

/**
 * 微信支付实现
 */
public class WechatPayService implements PaymentService {
    @Override
    public PaymentResult pay(PaymentRequest request) {
        // 调用微信支付SDK
        System.out.println("Processing WeChat payment: " + request.getAmount());
        return new PaymentResult(true, "WX" + System.currentTimeMillis());
    }
    
    @Override
    public boolean refund(String orderId, BigDecimal amount) {
        System.out.println("WeChat refund for order: " + orderId);
        return true;
    }
}
```

### 3. 抽象工厂模式 (Abstract Factory Pattern)

**场景**: 多渠道电商平台适配

```java
/**
 * 电商平台抽象工厂
 * 参考：腾讯多平台电商解决方案
 */
public interface ECommerceFactory {
    ProductService createProductService();
    OrderService createOrderService();
    UserService createUserService();
}

/**
 * 淘宝平台工厂
 */
public class TaobaoFactory implements ECommerceFactory {
    @Override
    public ProductService createProductService() {
        return new TaobaoProductService();
    }
    
    @Override
    public OrderService createOrderService() {
        return new TaobaoOrderService();
    }
    
    @Override
    public UserService createUserService() {
        return new TaobaoUserService();
    }
}

/**
 * 京东平台工厂
 */
public class JDFactory implements ECommerceFactory {
    @Override
    public ProductService createProductService() {
        return new JDProductService();
    }
    
    @Override
    public OrderService createOrderService() {
        return new JDOrderService();
    }
    
    @Override
    public UserService createUserService() {
        return new JDUserService();
    }
}

/**
 * 电商平台上下文
 */
public class ECommerceContext {
    private final ProductService productService;
    private final OrderService orderService;
    private final UserService userService;
    
    public ECommerceContext(ECommerceFactory factory) {
        this.productService = factory.createProductService();
        this.orderService = factory.createOrderService();
        this.userService = factory.createUserService();
    }
    
    public void processOrder(String userId, String productId) {
        User user = userService.getUser(userId);
        Product product = productService.getProduct(productId);
        Order order = orderService.createOrder(user, product);
        System.out.println("Order processed: " + order.getOrderId());
    }
}
```

### 4. 建造者模式 (Builder Pattern)

**场景**: 复杂订单构建、商品信息构建

```java
/**
 * 订单建造者
 * 参考：美团订单系统
 */
public class Order {
    private String orderId;
    private String userId;
    private List<OrderItem> items;
    private ShippingAddress shippingAddress;
    private PaymentMethod paymentMethod;
    private BigDecimal totalAmount;
    private String couponCode;
    private DeliveryType deliveryType;
    private String remark;
    private LocalDateTime createTime;
    
    // 私有构造函数
    private Order(Builder builder) {
        this.orderId = builder.orderId;
        this.userId = builder.userId;
        this.items = builder.items;
        this.shippingAddress = builder.shippingAddress;
        this.paymentMethod = builder.paymentMethod;
        this.totalAmount = builder.totalAmount;
        this.couponCode = builder.couponCode;
        this.deliveryType = builder.deliveryType;
        this.remark = builder.remark;
        this.createTime = builder.createTime;
    }
    
    public static class Builder {
        // 必填字段
        private final String orderId;
        private final String userId;
        private final List<OrderItem> items;
        
        // 可选字段
        private ShippingAddress shippingAddress;
        private PaymentMethod paymentMethod = PaymentMethod.ALIPAY;
        private BigDecimal totalAmount = BigDecimal.ZERO;
        private String couponCode;
        private DeliveryType deliveryType = DeliveryType.STANDARD;
        private String remark;
        private LocalDateTime createTime = LocalDateTime.now();
        
        public Builder(String orderId, String userId, List<OrderItem> items) {
            this.orderId = orderId;
            this.userId = userId;
            this.items = new ArrayList<>(items);
        }
        
        public Builder shippingAddress(ShippingAddress address) {
            this.shippingAddress = address;
            return this;
        }
        
        public Builder paymentMethod(PaymentMethod method) {
            this.paymentMethod = method;
            return this;
        }
        
        public Builder totalAmount(BigDecimal amount) {
            this.totalAmount = amount;
            return this;
        }
        
        public Builder couponCode(String code) {
            this.couponCode = code;
            return this;
        }
        
        public Builder deliveryType(DeliveryType type) {
            this.deliveryType = type;
            return this;
        }
        
        public Builder remark(String remark) {
            this.remark = remark;
            return this;
        }
        
        public Order build() {
            validateOrder();
            calculateTotalAmount();
            return new Order(this);
        }
        
        private void validateOrder() {
            if (items.isEmpty()) {
                throw new IllegalStateException("Order must have at least one item");
            }
            if (shippingAddress == null) {
                throw new IllegalStateException("Shipping address is required");
            }
        }
        
        private void calculateTotalAmount() {
            this.totalAmount = items.stream()
                .map(item -> item.getPrice().multiply(BigDecimal.valueOf(item.getQuantity())))
                .reduce(BigDecimal.ZERO, BigDecimal::add);
        }
    }
    
    // Getters...
    public String getOrderId() { return orderId; }
    public String getUserId() { return userId; }
    public BigDecimal getTotalAmount() { return totalAmount; }
    // ... 其他getter方法
}

/**
 * 使用示例
 */
public class OrderBuilderExample {
    public void createOrder() {
        List<OrderItem> items = Arrays.asList(
            new OrderItem("P001", "iPhone 14", new BigDecimal("5999"), 1),
            new OrderItem("P002", "AirPods Pro", new BigDecimal("1899"), 1)
        );
        
        ShippingAddress address = new ShippingAddress(
            "张三", "13800138000", "北京市朝阳区xxx路xxx号"
        );
        
        Order order = new Order.Builder("ORD001", "U001", items)
            .shippingAddress(address)
            .paymentMethod(PaymentMethod.WECHAT_PAY)
            .couponCode("SAVE100")
            .deliveryType(DeliveryType.EXPRESS)
            .remark("请快递员电话联系")
            .build();
            
        System.out.println("订单创建成功: " + order.getOrderId());
    }
}
```

### 5. 原型模式 (Prototype Pattern)

**场景**: 商品模板复制、营销活动配置复制

```java
/**
 * 商品原型
 * 参考：阿里巴巴商品中心
 */
public class Product implements Cloneable {
    private String productId;
    private String name;
    private String description;
    private BigDecimal price;
    private String category;
    private List<String> images;
    private Map<String, String> attributes;
    private ProductConfig config;
    
    public Product(String productId, String name) {
        this.productId = productId;
        this.name = name;
        this.images = new ArrayList<>();
        this.attributes = new HashMap<>();
        this.config = new ProductConfig();
    }
    
    @Override
    public Product clone() {
        try {
            Product cloned = (Product) super.clone();
            // 深度复制可变对象
            cloned.images = new ArrayList<>(this.images);
            cloned.attributes = new HashMap<>(this.attributes);
            cloned.config = this.config.clone();
            return cloned;
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException("Clone not supported", e);
        }
    }
    
    // Getters and Setters
    public void setProductId(String productId) { this.productId = productId; }
    public void setName(String name) { this.name = name; }
    public void setPrice(BigDecimal price) { this.price = price; }
    public void addImage(String imageUrl) { this.images.add(imageUrl); }
    public void setAttribute(String key, String value) { this.attributes.put(key, value); }
    
    @Override
    public String toString() {
        return String.format("Product{id='%s', name='%s', price=%s}", 
            productId, name, price);
    }
}

/**
 * 商品配置
 */
public class ProductConfig implements Cloneable {
    private boolean isVirtual;
    private boolean requiresShipping;
    private int stockThreshold;
    private List<String> tags;
    
    public ProductConfig() {
        this.tags = new ArrayList<>();
        this.stockThreshold = 10;
        this.requiresShipping = true;
    }
    
    @Override
    public ProductConfig clone() {
        try {
            ProductConfig cloned = (ProductConfig) super.clone();
            cloned.tags = new ArrayList<>(this.tags);
            return cloned;
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException("Clone not supported", e);
        }
    }
    
    // Getters and Setters
    public void addTag(String tag) { this.tags.add(tag); }
    public void setVirtual(boolean virtual) { this.isVirtual = virtual; }
    public void setRequiresShipping(boolean requiresShipping) { this.requiresShipping = requiresShipping; }
}

/**
 * 商品原型管理器
 */
public class ProductPrototypeManager {
    private Map<String, Product> prototypes = new HashMap<>();
    
    public void addPrototype(String key, Product prototype) {
        prototypes.put(key, prototype);
    }
    
    public Product getPrototype(String key) {
        Product prototype = prototypes.get(key);
        return prototype != null ? prototype.clone() : null;
    }
    
    // 预定义一些常用商品模板
    public ProductPrototypeManager() {
        // 电子产品模板
        Product electronics = new Product("TPL_ELECTRONICS", "电子产品模板");
        electronics.setPrice(new BigDecimal("0.00"));
        electronics.setAttribute("warranty", "1年");
        electronics.setAttribute("brand", "");
        electronics.addImage("default_electronics.jpg");
        addPrototype("ELECTRONICS", electronics);
        
        // 服装模板
        Product clothing = new Product("TPL_CLOTHING", "服装模板");
        clothing.setPrice(new BigDecimal("0.00"));
        clothing.setAttribute("material", "");
        clothing.setAttribute("size", "");
        clothing.addImage("default_clothing.jpg");
        addPrototype("CLOTHING", clothing);
    }
}

/**
 * 使用示例
 */
public class PrototypeExample {
    public void createProducts() {
        ProductPrototypeManager manager = new ProductPrototypeManager();
        
        // 基于电子产品模板创建iPhone
        Product iphone = manager.getPrototype("ELECTRONICS");
        iphone.setProductId("P001");
        iphone.setName("iPhone 14 Pro");
        iphone.setPrice(new BigDecimal("8999"));
        iphone.setAttribute("brand", "Apple");
        iphone.addImage("iphone14_1.jpg");
        
        // 基于电子产品模板创建MacBook
        Product macbook = manager.getPrototype("ELECTRONICS");
        macbook.setProductId("P002");
        macbook.setName("MacBook Pro");
        macbook.setPrice(new BigDecimal("15999"));
        macbook.setAttribute("brand", "Apple");
        macbook.addImage("macbook_1.jpg");
        
        System.out.println("创建商品: " + iphone);
        System.out.println("创建商品: " + macbook);
    }
}
```

---

## 结构型模式

### 6. 适配器模式 (Adapter Pattern)

**场景**: 第三方支付接口适配、物流接口适配

```java
/**
 * 统一支付接口
 * 参考：拼多多支付聚合
 */
public interface UnifiedPaymentService {
    PaymentResult pay(String orderId, BigDecimal amount, String userId);
    boolean queryPaymentStatus(String paymentId);
}

/**
 * 第三方支付宝SDK（不可修改）
 */
public class AlipaySDK {
    public AlipayResponse submitPayment(AlipayRequest request) {
        System.out.println("Alipay processing: " + request.getTotalAmount());
        return new AlipayResponse("SUCCESS", "2023" + System.currentTimeMillis());
    }
    
    public String checkTradeStatus(String tradeNo) {
        return "TRADE_SUCCESS";
    }
}

/**
 * 第三方微信支付SDK（不可修改）
 */
public class WechatPaySDK {
    public WxPayResult unifiedOrder(WxPayRequest request) {
        System.out.println("WeChat processing: " + request.getTotal_fee());
        return new WxPayResult(0, "wx" + System.currentTimeMillis());
    }
    
    public WxPayResult orderQuery(String transactionId) {
        return new WxPayResult(0, "SUCCESS");
    }
}

/**
 * 支付宝适配器
 */
public class AlipayAdapter implements UnifiedPaymentService {
    private AlipaySDK alipaySDK;
    
    public AlipayAdapter(AlipaySDK alipaySDK) {
        this.alipaySDK = alipaySDK;
    }
    
    @Override
    public PaymentResult pay(String orderId, BigDecimal amount, String userId) {
        // 适配参数格式
        AlipayRequest request = new AlipayRequest();
        request.setOutTradeNo(orderId);
        request.setTotalAmount(amount.toString());
        request.setSubject("订单支付");
        request.setBuyerId(userId);
        
        AlipayResponse response = alipaySDK.submitPayment(request);
        
        // 适配返回结果
        boolean success = "SUCCESS".equals(response.getCode());
        return new PaymentResult(success, response.getTradeNo(), "Alipay");
    }
    
    @Override
    public boolean queryPaymentStatus(String paymentId) {
        String status = alipaySDK.checkTradeStatus(paymentId);
        return "TRADE_SUCCESS".equals(status);
    }
}

/**
 * 微信支付适配器
 */
public class WechatPayAdapter implements UnifiedPaymentService {
    private WechatPaySDK wechatSDK;
    
    public WechatPayAdapter(WechatPaySDK wechatSDK) {
        this.wechatSDK = wechatSDK;
    }
    
    @Override
    public PaymentResult pay(String orderId, BigDecimal amount, String userId) {
        // 适配参数格式
        WxPayRequest request = new WxPayRequest();
        request.setOut_trade_no(orderId);
        request.setTotal_fee(amount.multiply(new BigDecimal("100")).intValue()); // 转换为分
        request.setBody("订单支付");
        request.setOpenid(userId);
        
        WxPayResult result = wechatSDK.unifiedOrder(request);
        
        // 适配返回结果
        boolean success = result.getReturn_code() == 0;
        return new PaymentResult(success, result.getPrepay_id(), "WeChat");
    }
    
    @Override
    public boolean queryPaymentStatus(String paymentId) {
        WxPayResult result = wechatSDK.orderQuery(paymentId);
        return result.getReturn_code() == 0 && "SUCCESS".equals(result.getTrade_state());
    }
}

/**
 * 支付服务使用示例
 */
public class PaymentAdapterExample {
    public void processPayments() {
        // 支付宝支付
        UnifiedPaymentService alipayService = new AlipayAdapter(new AlipaySDK());
        PaymentResult alipayResult = alipayService.pay("ORD001", new BigDecimal("100"), "user123");
        System.out.println("支付宝支付结果: " + alipayResult.isSuccess());
        
        // 微信支付
        UnifiedPaymentService wechatService = new WechatPayAdapter(new WechatPaySDK());
        PaymentResult wechatResult = wechatService.pay("ORD002", new BigDecimal("200"), "user456");
        System.out.println("微信支付结果: " + wechatResult.isSuccess());
    }
}
```

### 7. 桥接模式 (Bridge Pattern)

**场景**: 跨平台消息推送、多渠道营销

```java
/**
 * 消息发送抽象
 * 参考：字节跳动消息推送系统
 */
public abstract class MessageSender {
    protected MessageChannel channel;
    
    public MessageSender(MessageChannel channel) {
        this.channel = channel;
    }
    
    public abstract void sendMessage(String recipient, String content);
    public abstract void sendBatchMessage(List<String> recipients, String content);
}

/**
 * 消息渠道接口
 */
public interface MessageChannel {
    boolean send(String recipient, String content, MessageType type);
    boolean sendBatch(List<String> recipients, String content, MessageType type);
    String getChannelName();
}

/**
 * 普通消息发送器
 */
public class SimpleMessageSender extends MessageSender {
    public SimpleMessageSender(MessageChannel channel) {
        super(channel);
    }
    
    @Override
    public void sendMessage(String recipient, String content) {
        System.out.println("发送普通消息通过 " + channel.getChannelName());
        channel.send(recipient, content, MessageType.TEXT);
    }
    
    @Override
    public void sendBatchMessage(List<String> recipients, String content) {
        System.out.println("批量发送普通消息通过 " + channel.getChannelName());
        channel.sendBatch(recipients, content, MessageType.TEXT);
    }
}

/**
 * 营销消息发送器
 */
public class MarketingMessageSender extends MessageSender {
    public MarketingMessageSender(MessageChannel channel) {
        super(channel);
    }
    
    @Override
    public void sendMessage(String recipient, String content) {
        // 添加营销消息的特殊处理
        String marketingContent = addMarketingTemplate(content);
        System.out.println("发送营销消息通过 " + channel.getChannelName());
        channel.send(recipient, marketingContent, MessageType.MARKETING);
    }
    
    @Override
    public void sendBatchMessage(List<String> recipients, String content) {
        String marketingContent = addMarketingTemplate(content);
        System.out.println("批量发送营销消息通过 " + channel.getChannelName());
        // 分批发送，避免被限流
        for (int i = 0; i < recipients.size(); i += 100) {
            List<String> batch = recipients.subList(i, 
                Math.min(i + 100, recipients.size()));
            channel.sendBatch(batch, marketingContent, MessageType.MARKETING);
            // 添加延迟避免频率限制
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
    
    private String addMarketingTemplate(String content) {
        return "【特惠活动】" + content + " 【退订回T】";
    }
}

/**
 * 短信渠道实现
 */
public class SmsChannel implements MessageChannel {
    private String apiKey;
    private String signature;
    
    public SmsChannel(String apiKey, String signature) {
        this.apiKey = apiKey;
        this.signature = signature;
    }
    
    @Override
    public boolean send(String recipient, String content, MessageType type) {
        // 调用短信服务API
        System.out.println("短信发送: " + recipient + " -> " + content);
        return true;
    }
    
    @Override
    public boolean sendBatch(List<String> recipients, String content, MessageType type) {
        System.out.println("批量短信发送: " + recipients.size() + "个用户");
        return true;
    }
    
    @Override
    public String getChannelName() {
        return "短信";
    }
}

/**
 * 邮件渠道实现
 */
public class EmailChannel implements MessageChannel {
    private String smtpServer;
    private String username;
    
    public EmailChannel(String smtpServer, String username) {
        this.smtpServer = smtpServer;
        this.username = username;
    }
    
    @Override
    public boolean send(String recipient, String content, MessageType type) {
        System.out.println("邮件发送: " + recipient + " -> " + content);
        return true;
    }
    
    @Override
    public boolean sendBatch(List<String> recipients, String content, MessageType type) {
        System.out.println("批量邮件发送: " + recipients.size() + "个用户");
        return true;
    }
    
    @Override
    public String getChannelName() {
        return "邮件";
    }
}

/**
 * 微信渠道实现
 */
public class WechatChannel implements MessageChannel {
    private String appId;
    private String appSecret;
    
    public WechatChannel(String appId, String appSecret) {
        this.appId = appId;
        this.appSecret = appSecret;
    }
    
    @Override
    public boolean send(String recipient, String content, MessageType type) {
        System.out.println("微信消息发送: " + recipient + " -> " + content);
        return true;
    }
    
    @Override
    public boolean sendBatch(List<String> recipients, String content, MessageType type) {
        System.out.println("批量微信消息发送: " + recipients.size() + "个用户");
        return true;
    }
    
    @Override
    public String getChannelName() {
        return "微信";
    }
}

/**
 * 使用示例
 */
public class BridgePatternExample {
    public void sendNotifications() {
        // 短信营销
        MessageSender smsMarketing = new MarketingMessageSender(
            new SmsChannel("api-key", "【商城】"));
        smsMarketing.sendMessage("13800138000", "限时特惠，iPhone14仅售5999元！");
        
        // 邮件通知
        MessageSender emailNotification = new SimpleMessageSender(
            new EmailChannel("smtp.qq.com", "service@mall.com"));
        emailNotification.sendMessage("user@example.com", "您的订单已发货");
        
        // 微信营销
        MessageSender wechatMarketing = new MarketingMessageSender(
            new WechatChannel("wx123", "secret456"));
        List<String> users = Arrays.asList("openid1", "openid2", "openid3");
        wechatMarketing.sendBatchMessage(users, "新品上市，点击查看详情");
    }
}

enum MessageType {
    TEXT, MARKETING, NOTIFICATION
}
```

### 8. 组合模式 (Composite Pattern)

**场景**: 商品分类树、权限菜单树

```java
/**
 * 商品分类组件抽象
 * 参考：天猫商品分类体系
 */
public abstract class CategoryComponent {
    protected String id;
    protected String name;
    protected CategoryComponent parent;
    
    public CategoryComponent(String id, String name) {
        this.id = id;
        this.name = name;
    }
    
    // 基本操作
    public abstract void display(int depth);
    public abstract int getProductCount();
    public abstract List<Product> getProducts();
    
    // 容器操作 - 默认抛出异常，只有容器类重写
    public void add(CategoryComponent component) {
        throw new UnsupportedOperationException("不支持添加操作");
    }
    
    public void remove(CategoryComponent component) {
        throw new UnsupportedOperationException("不支持删除操作");
    }
    
    public CategoryComponent getChild(int index) {
        throw new UnsupportedOperationException("不支持获取子节点操作");
    }
    
    public List<CategoryComponent> getChildren() {
        throw new UnsupportedOperationException("不支持获取子节点列表操作");
    }
    
    // Getters
    public String getId() { return id; }
    public String getName() { return name; }
    public CategoryComponent getParent() { return parent; }
    public void setParent(CategoryComponent parent) { this.parent = parent; }
}

/**
 * 商品分类（容器）
 */
public class Category extends CategoryComponent {
    private List<CategoryComponent> children;
    private String description;
    private boolean isActive;
    
    public Category(String id, String name, String description) {
        super(id, name);
        this.description = description;
        this.children = new ArrayList<>();
        this.isActive = true;
    }
    
    @Override
    public void add(CategoryComponent component) {
        component.setParent(this);
        children.add(component);
    }
    
    @Override
    public void remove(CategoryComponent component) {
        component.setParent(null);
        children.remove(component);
    }
    
    @Override
    public CategoryComponent getChild(int index) {
        return children.get(index);
    }
    
    @Override
    public List<CategoryComponent> getChildren() {
        return new ArrayList<>(children);
    }
    
    @Override
    public void display(int depth) {
        StringBuilder indent = new StringBuilder();
        for (int i = 0; i < depth; i++) {
            indent.append("  ");
        }
        System.out.println(indent + "📁 " + name + " (" + getProductCount() + "个商品)");
        
        for (CategoryComponent child : children) {
            child.display(depth + 1);
        }
    }
    
    @Override
    public int getProductCount() {
        return children.stream()
            .mapToInt(CategoryComponent::getProductCount)
            .sum();
    }
    
    @Override
    public List<Product> getProducts() {
        List<Product> allProducts = new ArrayList<>();
        for (CategoryComponent child : children) {
            allProducts.addAll(child.getProducts());
        }
        return allProducts;
    }
    
    public String getDescription() { return description; }
    public boolean isActive() { return isActive; }
    public void setActive(boolean active) { this.isActive = active; }
}

/**
 * 叶子商品分类
 */
public class ProductCategory extends CategoryComponent {
    private List<Product> products;
    private String categoryImage;
    private Map<String, String> attributes;
    
    public ProductCategory(String id, String name) {
        super(id, name);
        this.products = new ArrayList<>();
        this.attributes = new HashMap<>();
    }
    
    @Override
    public void display(int depth) {
        StringBuilder indent = new StringBuilder();
        for (int i = 0; i < depth; i++) {
            indent.append("  ");
        }
        System.out.println(indent + "📦 " + name + " (" + products.size() + "个商品)");
        
        // 显示商品列表
        for (Product product : products) {
            System.out.println(indent + "  - " + product.getName() + " ¥" + product.getPrice());
        }
    }
    
    @Override
    public int getProductCount() {
        return products.size();
    }
    
    @Override
    public List<Product> getProducts() {
        return new ArrayList<>(products);
    }
    
    public void addProduct(Product product) {
        products.add(product);
    }
    
    public void removeProduct(Product product) {
        products.remove(product);
    }
    
    public void setAttribute(String key, String value) {
        attributes.put(key, value);
    }
    
    public String getAttribute(String key) {
        return attributes.get(key);
    }
}

/**
 * 商品分类管理器
 */
public class CategoryManager {
    private Category rootCategory;
    
    public CategoryManager() {
        // 构建商品分类树
        buildCategoryTree();
    }
    
    private void buildCategoryTree() {
        // 根分类
        rootCategory = new Category("ROOT", "全部商品", "商城根分类");
        
        // 一级分类
        Category electronics = new Category("C1", "数码电器", "数码电器分类");
        Category clothing = new Category("C2", "服装鞋帽", "服装鞋帽分类");
        Category books = new Category("C3", "图书文娱", "图书文娱分类");
        
        rootCategory.add(electronics);
        rootCategory.add(clothing);
        rootCategory.add(books);
        
        // 二级分类 - 数码电器
        Category smartphones = new Category("C11", "智能手机", "智能手机分类");
        Category laptops = new Category("C12", "笔记本电脑", "笔记本电脑分类");
        
        electronics.add(smartphones);
        electronics.add(laptops);
        
        // 三级分类 - 智能手机品牌
        ProductCategory iphones = new ProductCategory("C111", "苹果手机");
        ProductCategory samsungs = new ProductCategory("C112", "三星手机");
        ProductCategory huaweis = new ProductCategory("C113", "华为手机");
        
        smartphones.add(iphones);
        smartphones.add(samsungs);
        smartphones.add(huaweis);
        
        // 添加具体商品
        iphones.addProduct(new Product("P001", "iPhone 14 Pro", new BigDecimal("8999")));
        iphones.addProduct(new Product("P002", "iPhone 14", new BigDecimal("5999")));
        
        samsungs.addProduct(new Product("P003", "Galaxy S23", new BigDecimal("4999")));
        
        huaweis.addProduct(new Product("P004", "Mate50 Pro", new BigDecimal("6699")));
        huaweis.addProduct(new Product("P005", "P60", new BigDecimal("4988")));
        
        // 笔记本电脑
        ProductCategory macbooks = new ProductCategory("C121", "MacBook");
        ProductCategory thinkpads = new ProductCategory("C122", "ThinkPad");
        
        laptops.add(macbooks);
        laptops.add(thinkpads);
        
        macbooks.addProduct(new Product("P006", "MacBook Pro 16", new BigDecimal("18999")));
        thinkpads.addProduct(new Product("P007", "ThinkPad X1", new BigDecimal("12999")));
    }
    
    public void displayAllCategories() {
        System.out.println("=== 商品分类树 ===");
        rootCategory.display(0);
    }
    
    public CategoryComponent findCategory(String categoryId) {
        return findCategoryRecursive(rootCategory, categoryId);
    }
    
    private CategoryComponent findCategoryRecursive(CategoryComponent component, String categoryId) {
        if (component.getId().equals(categoryId)) {
            return component;
        }
        
        try {
            for (CategoryComponent child : component.getChildren()) {
                CategoryComponent found = findCategoryRecursive(child, categoryId);
                if (found != null) {
                    return found;
                }
            }
        } catch (UnsupportedOperationException e) {
            // 叶子节点没有子节点，忽略异常
        }
        
        return null;
    }
    
    public List<Product> getProductsByCategory(String categoryId) {
        CategoryComponent category = findCategory(categoryId);
        return category != null ? category.getProducts() : new ArrayList<>();
    }
}

/**
 * 使用示例
 */
public class CompositePatternExample {
    public void demonstrateCategories() {
        CategoryManager manager = new CategoryManager();
        
        // 显示整个分类树
        manager.displayAllCategories();
        
        System.out.println("\n=== 查找特定分类 ===");
        CategoryComponent smartphones = manager.findCategory("C11");
        if (smartphones != null) {
            System.out.println("找到分类: " + smartphones.getName());
            smartphones.display(0);
        }
        
        System.out.println("\n=== 获取iPhone分类下的所有商品 ===");
        List<Product> iphones = manager.getProductsByCategory("C111");
        iphones.forEach(product -> 
            System.out.println("- " + product.getName() + ": ¥" + product.getPrice()));
    }
}
```

### 9. 装饰器模式 (Decorator Pattern)

**场景**: 商品价格计算（优惠券、会员折扣等）、订单服务增强

```java
/**
 * 价格计算接口
 * 参考：美团优惠计算系统
 */
public interface PriceCalculator {
    BigDecimal calculatePrice(Order order);
    String getDescription();
}

/**
 * 基础价格计算器
 */
public class BasePriceCalculator implements PriceCalculator {
    @Override
    public BigDecimal calculatePrice(Order order) {
        return order.getOriginalAmount();
    }
    
    @Override
    public String getDescription() {
        return "原价";
    }
}

/**
 * 价格计算装饰器抽象类
 */
public abstract class PriceCalculatorDecorator implements PriceCalculator {
    protected PriceCalculator calculator;
    
    public PriceCalculatorDecorator(PriceCalculator calculator) {
        this.calculator = calculator;
    }
    
    @Override
    public BigDecimal calculatePrice(Order order) {
        return calculator.calculatePrice(order);
    }
    
    @Override
    public String getDescription() {
        return calculator.getDescription();
    }
}

/**
 * 会员折扣装饰器
 */
public class MemberDiscountDecorator extends PriceCalculatorDecorator {
    private MemberLevel memberLevel;
    private BigDecimal discountRate;
    
    public MemberDiscountDecorator(PriceCalculator calculator, MemberLevel memberLevel) {
        super(calculator);
        this.memberLevel = memberLevel;
        this.discountRate = getDiscountRate(memberLevel);
    }
    
    @Override
    public BigDecimal calculatePrice(Order order) {
        BigDecimal basePrice = super.calculatePrice(order);
        BigDecimal discount = basePrice.multiply(discountRate);
        System.out.println(memberLevel.getName() + "折扣: -¥" + discount);
        return basePrice.subtract(discount);
    }
    
    @Override
    public String getDescription() {
        return super.getDescription() + " + " + memberLevel.getName() + "折扣";
    }
    
    private BigDecimal getDiscountRate(MemberLevel level) {
        switch (level) {
            case VIP: return new BigDecimal("0.05");      // 5%折扣
            case GOLD: return new BigDecimal("0.03");     // 3%折扣
            case SILVER: return new BigDecimal("0.02");   // 2%折扣
            default: return BigDecimal.ZERO;
        }
    }
}

/**
 * 优惠券装饰器
 */
public class CouponDecorator extends PriceCalculatorDecorator {
    private Coupon coupon;
    
    public CouponDecorator(PriceCalculator calculator, Coupon coupon) {
        super(calculator);
        this.coupon = coupon;
    }
    
    @Override
    public BigDecimal calculatePrice(Order order) {
        BigDecimal basePrice = super.calculatePrice(order);
        
        if (!isCouponApplicable(order, basePrice)) {
            return basePrice;
        }
        
        BigDecimal discount = calculateCouponDiscount(basePrice);
        System.out.println("优惠券减免: -¥" + discount);
        return basePrice.subtract(discount);
    }
    
    @Override
    public String getDescription() {
        return super.getDescription() + " + " + coupon.getName();
    }
    
    private boolean isCouponApplicable(Order order, BigDecimal currentPrice) {
        // 检查优惠券是否可用
        if (coupon.isExpired()) {
            System.out.println("优惠券已过期");
            return false;
        }
        
        if (currentPrice.compareTo(coupon.getMinAmount()) < 0) {
            System.out.println("未达到优惠券使用门槛: ¥" + coupon.getMinAmount());
            return false;
        }
        
        return true;
    }
    
    private BigDecimal calculateCouponDiscount(BigDecimal price) {
        switch (coupon.getType()) {
            case AMOUNT:
                return coupon.getDiscountValue();
            case PERCENTAGE:
                BigDecimal percentDiscount = price.multiply(coupon.getDiscountValue())
                    .divide(new BigDecimal("100"), 2, RoundingMode.HALF_UP);
                // 限制最大优惠金额
                return coupon.getMaxDiscount() != null && 
                       percentDiscount.compareTo(coupon.getMaxDiscount()) > 0
                    ? coupon.getMaxDiscount() 
                    : percentDiscount;
            default:
                return BigDecimal.ZERO;
        }
    }
}

/**
 * 满减活动装饰器
 */
public class FullReductionDecorator extends PriceCalculatorDecorator {
    private List<FullReductionRule> rules;
    
    public FullReductionDecorator(PriceCalculator calculator, List<FullReductionRule> rules) {
        super(calculator);
        this.rules = rules.stream()
            .sorted((r1, r2) -> r2.getThreshold().compareTo(r1.getThreshold())) // 按门槛倒序
            .collect(Collectors.toList());
    }
    
    @Override
    public BigDecimal calculatePrice(Order order) {
        BigDecimal basePrice = super.calculatePrice(order);
        
        // 找到适用的满减规则
        FullReductionRule applicableRule = rules.stream()
            .filter(rule -> basePrice.compareTo(rule.getThreshold()) >= 0)
            .findFirst()
            .orElse(null);
            
        if (applicableRule != null) {
            System.out.println("满减活动: 满" + applicableRule.getThreshold() + 
                "减" + applicableRule.getReduction());
            return basePrice.subtract(applicableRule.getReduction());
        }
        
        return basePrice;
    }
    
    @Override
    public String getDescription() {
        return super.getDescription() + " + 满减活动";
    }
}

/**
 * 新用户首单装饰器
 */
public class FirstOrderDecorator extends PriceCalculatorDecorator {
    private BigDecimal firstOrderDiscount;
    private User user;
    
    public FirstOrderDecorator(PriceCalculator calculator, User user, BigDecimal discount) {
        super(calculator);
        this.user = user;
        this.firstOrderDiscount = discount;
    }
    
    @Override
    public BigDecimal calculatePrice(Order order) {
        BigDecimal basePrice = super.calculatePrice(order);
        
        if (user.isFirstOrder()) {
            System.out.println("新用户首单优惠: -¥" + firstOrderDiscount);
            return basePrice.subtract(firstOrderDiscount);
        }
        
        return basePrice;
    }
    
    @Override
    public String getDescription() {
        return super.getDescription() + (user.isFirstOrder() ? " + 新用户首单优惠" : "");
    }
}

/**
 * 支持类定义
 */
public enum MemberLevel {
    ORDINARY("普通会员"),
    SILVER("银牌会员"), 
    GOLD("金牌会员"), 
    VIP("VIP会员");
    
    private String name;
    
    MemberLevel(String name) {
        this.name = name;
    }
    
    public String getName() { return name; }
}

public class Coupon {
    private String id;
    private String name;
    private CouponType type;
    private BigDecimal discountValue;
    private BigDecimal minAmount;
    private BigDecimal maxDiscount;
    private LocalDateTime expireTime;
    
    // 构造函数、getters、setters...
    public boolean isExpired() {
        return LocalDateTime.now().isAfter(expireTime);
    }
    
    // getters...
    public String getName() { return name; }
    public CouponType getType() { return type; }
    public BigDecimal getDiscountValue() { return discountValue; }
    public BigDecimal getMinAmount() { return minAmount; }
    public BigDecimal getMaxDiscount() { return maxDiscount; }
}

public enum CouponType {
    AMOUNT,      // 固定金额
    PERCENTAGE   // 百分比
}

public class FullReductionRule {
    private BigDecimal threshold;
    private BigDecimal reduction;
    
    public FullReductionRule(BigDecimal threshold, BigDecimal reduction) {
        this.threshold = threshold;
        this.reduction = reduction;
    }
    
    public BigDecimal getThreshold() { return threshold; }
    public BigDecimal getReduction() { return reduction; }
}

/**
 * 价格计算示例
 */
public class PriceCalculatorExample {
    public void calculateOrderPrice() {
        // 创建订单
        Order order = createSampleOrder();
        User user = new User("U001", "张三", MemberLevel.GOLD, false);
        
        System.out.println("=== 订单价格计算 ===");
        System.out.println("原价: ¥" + order.getOriginalAmount());
        
        // 基础价格计算
        PriceCalculator calculator = new BasePriceCalculator();
        
        // 添加会员折扣
        calculator = new MemberDiscountDecorator(calculator, user.getMemberLevel());
        
        // 添加优惠券
        Coupon coupon = createCoupon();
        calculator = new CouponDecorator(calculator, coupon);
        
        // 添加满减活动
        List<FullReductionRule> rules = Arrays.asList(
            new FullReductionRule(new BigDecimal("500"), new BigDecimal("50")),
            new FullReductionRule(new BigDecimal("200"), new BigDecimal("20")),
            new FullReductionRule(new BigDecimal("100"), new BigDecimal("10"))
        );
        calculator = new FullReductionDecorator(calculator, rules);
        
        // 添加新用户首单优惠
        calculator = new FirstOrderDecorator(calculator, user, new BigDecimal("30"));
        
        // 计算最终价格
        BigDecimal finalPrice = calculator.calculatePrice(order);
        
        System.out.println("\n优惠详情: " + calculator.getDescription());
        System.out.println("最终价格: ¥" + finalPrice);
        System.out.println("总共优惠: ¥" + order.getOriginalAmount().subtract(finalPrice));
    }
    
    private Order createSampleOrder() {
        Order order = new Order();
        order.setOriginalAmount(new BigDecimal("599"));
        return order;
    }
    
    private Coupon createCoupon() {
        Coupon coupon = new Coupon();
        coupon.setName("满200减30优惠券");
        coupon.setType(CouponType.AMOUNT);
        coupon.setDiscountValue(new BigDecimal("30"));
        coupon.setMinAmount(new BigDecimal("200"));
        coupon.setExpireTime(LocalDateTime.now().plusDays(30));
        return coupon;
    }
}
```

### 10. 外观模式 (Facade Pattern)

**场景**: 订单处理流程、支付流程统一接口

```java
/**
 * 订单处理外观
 * 参考：京东订单处理系统
 */
public class OrderProcessingFacade {
    private InventoryService inventoryService;
    private PaymentService paymentService;
    private LogisticsService logisticsService;
    private NotificationService notificationService;
    private CouponService couponService;
    private MemberService memberService;
    private AuditService auditService;
    
    public OrderProcessingFacade() {
        this.inventoryService = new InventoryService();
        this.paymentService = new PaymentService();
        this.logisticsService = new LogisticsService();
        this.notificationService = new NotificationService();
        this.couponService = new CouponService();
        this.memberService = new MemberService();
        this.auditService = new AuditService();
    }
    
    /**
     * 处理订单 - 统一入口
     */
    public OrderResult processOrder(OrderRequest request) {
        System.out.println("开始处理订单: " + request.getOrderId());
        
        try {
            // 1. 订单预处理
            preprocessOrder(request);
            
            // 2. 库存检查和锁定
            if (!checkAndLockInventory(request)) {
                return OrderResult.failure("库存不足");
            }
            
            // 3. 价格计算（包含优惠）
            BigDecimal finalPrice = calculateFinalPrice(request);
            
            // 4. 支付处理
            PaymentResult paymentResult = processPayment(request, finalPrice);
            if (!paymentResult.isSuccess()) {
                // 支付失败，释放库存
                releaseInventory(request);
                return OrderResult.failure("支付失败: " + paymentResult.getMessage());
            }
            
            // 5. 生成物流订单
            String trackingNumber = createLogisticsOrder(request);
            
            // 6. 发送通知
            sendNotifications(request, trackingNumber);
            
            // 7. 记录审计日志
            auditService.logOrderCreated(request, finalPrice, paymentResult.getTransactionId());
            
            // 8. 更新会员积分
            updateMemberPoints(request, finalPrice);
            
            System.out.println("订单处理完成: " + request.getOrderId());
            return OrderResult.success(request.getOrderId(), trackingNumber);
            
        } catch (Exception e) {
            // 异常处理，回滚操作
            handleOrderException(request, e);
            return OrderResult.failure("订单处理异常: " + e.getMessage());
        }
    }
    
    /**
     * 取消订单 - 统一入口
     */
    public boolean cancelOrder(String orderId, String reason) {
        System.out.println("开始取消订单: " + orderId);
        
        try {
            // 1. 获取订单信息
            Order order = getOrderById(orderId);
            if (order == null) {
                return false;
            }
            
            // 2. 检查订单状态
            if (!order.isCancellable()) {
                System.out.println("订单状态不允许取消");
                return false;
            }
            
            // 3. 退款处理
            if (order.isPaid()) {
                boolean refundSuccess = paymentService.refund(
                    order.getPaymentId(), order.getFinalAmount());
                if (!refundSuccess) {
                    System.out.println("退款失败，无法取消订单");
                    return false;
                }
            }
            
            // 4. 释放库存
            inventoryService.releaseInventory(order.getItems());
            
            // 5. 取消物流
            if (order.getTrackingNumber() != null) {
                logisticsService.cancelShipment(order.getTrackingNumber());
            }
            
            // 6. 返还优惠券
            if (order.getCouponId() != null) {
                couponService.returnCoupon(order.getUserId(), order.getCouponId());
            }
            
            // 7. 发送取消通知
            notificationService.sendOrderCancelNotification(
                order.getUserId(), orderId, reason);
            
            // 8. 记录审计日志
            auditService.logOrderCancelled(orderId, reason);
            
            System.out.println("订单取消完成: " + orderId);
            return true;
            
        } catch (Exception e) {
            System.out.println("订单取消异常: " + e.getMessage());
            return false;
        }
    }
    
    /**
     * 查询订单状态 - 统一入口
     */
    public OrderStatusInfo getOrderStatus(String orderId) {
        try {
            Order order = getOrderById(orderId);
            if (order == null) {
                return null;
            }
            
            // 获取支付状态
            PaymentStatus paymentStatus = paymentService.getPaymentStatus(
                order.getPaymentId());
            
            // 获取物流状态
            LogisticsStatus logisticsStatus = null;
            if (order.getTrackingNumber() != null) {
                logisticsStatus = logisticsService.getShipmentStatus(
                    order.getTrackingNumber());
            }
            
            return new OrderStatusInfo(order, paymentStatus, logisticsStatus);
            
        } catch (Exception e) {
            System.out.println("查询订单状态异常: " + e.getMessage());
            return null;
        }
    }
    
    // 私有辅助方法
    private void preprocessOrder(OrderRequest request) {
        // 订单参数验证
        validateOrderRequest(request);
        
        // 用户权限检查
        memberService.validateUser(request.getUserId());
        
        // 商品信息验证
        validateProducts(request.getItems());
    }
    
    private boolean checkAndLockInventory(OrderRequest request) {
        for (OrderItem item : request.getItems()) {
            if (!inventoryService.checkStock(item.getProductId(), item.getQuantity())) {
                return false;
            }
        }
        
        // 锁定库存
        return inventoryService.lockInventory(request.getItems());
    }
    
    private BigDecimal calculateFinalPrice(OrderRequest request) {
        // 计算原价
        BigDecimal originalPrice = request.getItems().stream()
            .map(item -> item.getPrice().multiply(BigDecimal.valueOf(item.getQuantity())))
            .reduce(BigDecimal.ZERO, BigDecimal::add);
        
        // 应用优惠券
        BigDecimal discount = BigDecimal.ZERO;
        if (request.getCouponId() != null) {
            discount = couponService.calculateDiscount(
                request.getCouponId(), originalPrice);
        }
        
        // 会员折扣
        MemberLevel memberLevel = memberService.getMemberLevel(request.getUserId());
        BigDecimal memberDiscount = memberService.calculateMemberDiscount(
            memberLevel, originalPrice);
        
        return originalPrice.subtract(discount).subtract(memberDiscount);
    }
    
    private PaymentResult processPayment(OrderRequest request, BigDecimal amount) {
        return paymentService.processPayment(new PaymentRequest(
            request.getOrderId(),
            request.getUserId(),
            amount,
            request.getPaymentMethod()
        ));
    }
    
    private String createLogisticsOrder(OrderRequest request) {
        LogisticsRequest logisticsRequest = new LogisticsRequest(
            request.getOrderId(),
            request.getShippingAddress(),
            request.getItems()
        );
        return logisticsService.createShipment(logisticsRequest);
    }
    
    private void sendNotifications(OrderRequest request, String trackingNumber) {
        // 发送订单确认短信
        notificationService.sendOrderConfirmSMS(
            request.getUserId(), request.getOrderId());
        
        // 发送订单确认邮件
        notificationService.sendOrderConfirmEmail(
            request.getUserId(), request.getOrderId(), trackingNumber);
        
        // 推送消息
        notificationService.sendPushNotification(
            request.getUserId(), "您的订单已成功提交，正在准备发货");
    }
    
    private void updateMemberPoints(OrderRequest request, BigDecimal finalPrice) {
        int points = finalPrice.intValue(); // 1元 = 1积分
        memberService.addPoints(request.getUserId(), points);
    }
    
    private void handleOrderException(OrderRequest request, Exception e) {
        // 记录错误日志
        auditService.logOrderError(request.getOrderId(), e.getMessage());
        
        // 释放已锁定的资源
        try {
            releaseInventory(request);
            if (request.getCouponId() != null) {
                couponService.returnCoupon(request.getUserId(), request.getCouponId());
            }
        } catch (Exception rollbackException) {
            auditService.logOrderError(request.getOrderId(), 
                "回滚操作失败: " + rollbackException.getMessage());
        }
    }
    
    private void releaseInventory(OrderRequest request) {
        inventoryService.releaseInventory(request.getItems());
    }
    
    private void validateOrderRequest(OrderRequest request) {
        if (request.getItems() == null || request.getItems().isEmpty()) {
            throw new IllegalArgumentException("订单商品不能为空");
        }
        if (request.getShippingAddress() == null) {
            throw new IllegalArgumentException("收货地址不能为空");
        }
    }
    
    private void validateProducts(List<OrderItem> items) {
        for (OrderItem item : items) {
            if (!inventoryService.isProductValid(item.getProductId())) {
                throw new IllegalArgumentException("商品不存在: " + item.getProductId());
            }
        }
    }
    
    private Order getOrderById(String orderId) {
        // 从数据库获取订单信息
        return new Order(); // 模拟返回
    }
}

/**
 * 支持服务类（简化版）
 */
class InventoryService {
    public boolean checkStock(String productId, int quantity) {
        System.out.println("检查库存: " + productId + ", 数量: " + quantity);
        return true; // 模拟库存充足
    }
    
    public boolean lockInventory(List<OrderItem> items) {
        System.out.println("锁定库存: " + items.size() + "个商品");
        return true;
    }
    
    public void releaseInventory(List<OrderItem> items) {
        System.out.println("释放库存: " + items.size() + "个商品");
    }
    
    public boolean isProductValid(String productId) {
        return true; // 模拟商品有效
    }
}

class NotificationService {
    public void sendOrderConfirmSMS(String userId, String orderId) {
        System.out.println("发送订单确认短信: " + userId + ", 订单: " + orderId);
    }
    
    public void sendOrderConfirmEmail(String userId, String orderId, String trackingNumber) {
        System.out.println("发送订单确认邮件: " + userId + ", 订单: " + orderId + 
            ", 物流单号: " + trackingNumber);
    }
    
    public void sendPushNotification(String userId, String message) {
        System.out.println("推送消息: " + userId + " -> " + message);
    }
    
    public void sendOrderCancelNotification(String userId, String orderId, String reason) {
        System.out.println("发送订单取消通知: " + userId + ", 订单: " + orderId + 
            ", 原因: " + reason);
    }
}

/**
 * 使用示例
 */
public class FacadePatternExample {
    public void processOrderExample() {
        OrderProcessingFacade facade = new OrderProcessingFacade();
        
        // 创建订单请求
        OrderRequest request = new OrderRequest();
        request.setOrderId("ORD20231201001");
        request.setUserId("U001");
        request.setItems(Arrays.asList(
            new OrderItem("P001", "iPhone 14", new BigDecimal("5999"), 1),
            new OrderItem("P002", "AirPods", new BigDecimal("1299"), 1)
        ));
        request.setShippingAddress(new ShippingAddress("张三", "13800138000", "北京市朝阳区"));
        request.setPaymentMethod(PaymentMethod.ALIPAY);
        request.setCouponId("COUPON001");
        
        // 处理订单
        OrderResult result = facade.processOrder(request);
        
        if (result.isSuccess()) {
            System.out.println("订单处理成功!");
            System.out.println("订单号: " + result.getOrderId());
            System.out.println("物流单号: " + result.getTrackingNumber());
            
            // 查询订单状态
            OrderStatusInfo status = facade.getOrderStatus(result.getOrderId());
            System.out.println("订单状态: " + status);
            
        } else {
            System.out.println("订单处理失败: " + result.getMessage());
        }
    }
}
```

### 11. 享元模式 (Flyweight Pattern)

**场景**: 商品属性缓存、用户会话管理

```java
/**
 * 商品规格享元接口
 * 参考：淘宝商品规格系统
 */
public interface ProductSpecification {
    void display(ProductContext context);
    String getSpecKey();
}

/**
 * 具体商品规格享元
 */
public class ConcreteProductSpecification implements ProductSpecification {
    private final String category;      // 内在状态 - 商品类别
    private final String brand;         // 内在状态 - 品牌
    private final Map<String, String> commonAttributes; // 内在状态 - 通用属性
    
    public ConcreteProductSpecification(String category, String brand, 
                                      Map<String, String> commonAttributes) {
        this.category = category;
        this.brand = brand;
        this.commonAttributes = new HashMap<>(commonAttributes);
        System.out.println("创建商品规格享元: " + category + "-" + brand);
    }
    
    @Override
    public void display(ProductContext context) {
        System.out.println("=== 商品信息 ===");
        System.out.println("商品ID: " + context.getProductId());
        System.out.println("商品名称: " + context.getProductName());
        System.out.println("类别: " + category);
        System.out.println("品牌: " + brand);
        System.out.println("价格: ¥" + context.getPrice());
        System.out.println("库存: " + context.getStock());
        
        System.out.println("通用属性:");
        commonAttributes.forEach((key, value) -> 
            System.out.println("  " + key + ": " + value));
            
        System.out.println("特有属性:");
        context.getUniqueAttributes().forEach((key, value) -> 
            System.out.println("  " + key + ": " + value));
    }
    
    @Override
    public String getSpecKey() {
        return category + "-" + brand + "-" + 
               commonAttributes.hashCode();
    }
    
    // 内在状态访问器
    public String getCategory() { return category; }
    public String getBrand() { return brand; }
    public Map<String, String> getCommonAttributes() { 
        return new HashMap<>(commonAttributes); 
    }
}

/**
 * 商品上下文（外在状态）
 */
public class ProductContext {
    private String productId;
    private String productName;
    private BigDecimal price;
    private int stock;
    private Map<String, String> uniqueAttributes;
    private String sellerId;
    private LocalDateTime createTime;
    
    public ProductContext(String productId, String productName, BigDecimal price) {
        this.productId = productId;
        this.productName = productName;
        this.price = price;
        this.uniqueAttributes = new HashMap<>();
        this.createTime = LocalDateTime.now();
    }
    
    // Getters and Setters
    public String getProductId() { return productId; }
    public String getProductName() { return productName; }
    public BigDecimal getPrice() { return price; }
    public void setPrice(BigDecimal price) { this.price = price; }
    public int getStock() { return stock; }
    public void setStock(int stock) { this.stock = stock; }
    public Map<String, String> getUniqueAttributes() { return uniqueAttributes; }
    public void addUniqueAttribute(String key, String value) { 
        uniqueAttributes.put(key, value); 
    }
    public String getSellerId() { return sellerId; }
    public void setSellerId(String sellerId) { this.sellerId = sellerId; }
}

/**
 * 商品规格享元工厂
 */
public class ProductSpecificationFactory {
    private static final Map<String, ProductSpecification> flyweights = new HashMap<>();
    private static int createdCount = 0;
    
    public static ProductSpecification getSpecification(String category, String brand, 
                                                       Map<String, String> commonAttributes) {
        String key = generateKey(category, brand, commonAttributes);
        
        ProductSpecification spec = flyweights.get(key);
        if (spec == null) {
            spec = new ConcreteProductSpecification(category, brand, commonAttributes);
            flyweights.put(key, spec);
            createdCount++;
        }
        
        return spec;
    }
    
    private static String generateKey(String category, String brand, 
                                    Map<String, String> commonAttributes) {
        return category + "-" + brand + "-" + commonAttributes.hashCode();
    }
    
    public static int getCreatedCount() {
        return createdCount;
    }
    
    public static int getFlyweightCount() {
        return flyweights.size();
    }
    
    public static void printFlyweightInfo() {
        System.out.println("=== 享元统计信息 ===");
        System.out.println("创建的享元对象数量: " + createdCount);
        System.out.println("缓存的享元对象数量: " + flyweights.size());
        System.out.println("享元对象详情:");
        flyweights.forEach((key, spec) -> 
            System.out.println("  " + key + " -> " + spec.getClass().getSimpleName()));
    }
}

/**
 * 商品管理器
 */
public class ProductManager {
    private List<ProductItem> products;
    
    public ProductManager() {
        this.products = new ArrayList<>();
    }
    
    public void addProduct(String category, String brand, Map<String, String> commonAttrs,
                          String productId, String productName, BigDecimal price,
                          Map<String, String> uniqueAttrs) {
        
        // 获取享元对象
        ProductSpecification spec = ProductSpecificationFactory.getSpecification(
            category, brand, commonAttrs);
        
        // 创建上下文对象
        ProductContext context = new ProductContext(productId, productName, price);
        uniqueAttrs.forEach(context::addUniqueAttribute);
        
        products.add(new ProductItem(spec, context));
    }
    
    public void displayProduct(String productId) {
        ProductItem item = products.stream()
            .filter(p -> p.getContext().getProductId().equals(productId))
            .findFirst()
            .orElse(null);
            
        if (item != null) {
            item.getSpecification().display(item.getContext());
        } else {
            System.out.println("商品不存在: " + productId);
        }
    }
    
    public void displayAllProducts() {
        System.out.println("=== 所有商品列表 ===");
        for (ProductItem item : products) {
            ProductContext context = item.getContext();
            System.out.println(context.getProductId() + ": " + 
                context.getProductName() + " - ¥" + context.getPrice());
        }
    }
    
    public List<ProductItem> getProductsByCategory(String category) {
        return products.stream()
            .filter(item -> {
                if (item.getSpecification() instanceof ConcreteProductSpecification) {
                    ConcreteProductSpecification spec = 
                        (ConcreteProductSpecification) item.getSpecification();
                    return spec.getCategory().equals(category);
                }
                return false;
            })
            .collect(Collectors.toList());
    }
}

/**
 * 商品项（包含享元和上下文）
 */
public class ProductItem {
    private ProductSpecification specification;
    private ProductContext context;
    
    public ProductItem(ProductSpecification specification, ProductContext context) {
        this.specification = specification;
        this.context = context;
    }
    
    public ProductSpecification getSpecification() { return specification; }
    public ProductContext getContext() { return context; }
}

/**
 * 使用示例
 */
public class FlyweightPatternExample {
    public void demonstrateFlyweight() {
        ProductManager manager = new ProductManager();
        
        // 准备通用属性
        Map<String, String> phoneCommonAttrs = new HashMap<>();
        phoneCommonAttrs.put("warranty", "1年全国联保");
        phoneCommonAttrs.put("category_desc", "智能手机");
        
        Map<String, String> laptopCommonAttrs = new HashMap<>();
        laptopCommonAttrs.put("warranty", "2年全球联保");
        laptopCommonAttrs.put("category_desc", "笔记本电脑");
        
        // 添加多个iPhone产品（共享相同的规格享元）
        Map<String, String> iphone14Attrs = new HashMap<>();
        iphone14Attrs.put("storage", "128GB");
        iphone14Attrs.put("color", "深空黑");
        manager.addProduct("手机", "Apple", phoneCommonAttrs, 
            "P001", "iPhone 14 128GB 深空黑", new BigDecimal("5999"), iphone14Attrs);
        
        Map<String, String> iphone14ProAttrs = new HashMap<>();
        iphone14ProAttrs.put("storage", "256GB");
        iphone14ProAttrs.put("color", "远峰蓝");
        manager.addProduct("手机", "Apple", phoneCommonAttrs,
            "P002", "iPhone 14 Pro 256GB 远峰蓝", new BigDecimal("8999"), iphone14ProAttrs);
        
        Map<String, String> iphone14PlusAttrs = new HashMap<>();
        iphone14PlusAttrs.put("storage", "512GB");
        iphone14PlusAttrs.put("color", "紫色");
        manager.addProduct("手机", "Apple", phoneCommonAttrs,
            "P003", "iPhone 14 Plus 512GB 紫色", new BigDecimal("7999"), iphone14PlusAttrs);
        
        // 添加MacBook产品（不同的规格享元）
        Map<String, String> macbookAttrs = new HashMap<>();
        macbookAttrs.put("processor", "M2芯片");
        macbookAttrs.put("memory", "16GB");
        macbookAttrs.put("storage", "512GB SSD");
        manager.addProduct("笔记本", "Apple", laptopCommonAttrs,
            "P004", "MacBook Pro 16英寸", new BigDecimal("18999"), macbookAttrs);
        
        // 添加更多相同品牌类别的产品（继续共享享元）
        Map<String, String> macbookAirAttrs = new HashMap<>();
        macbookAirAttrs.put("processor", "M2芯片");
        macbookAirAttrs.put("memory", "8GB");
        macbookAirAttrs.put("storage", "256GB SSD");
        manager.addProduct("笔记本", "Apple", laptopCommonAttrs,
            "P005", "MacBook Air 13英寸", new BigDecimal("9999"), macbookAirAttrs);
        
        // 添加不同品牌的产品（新的享元）
        Map<String, String> samsungPhoneAttrs = new HashMap<>();
        samsungPhoneAttrs.put("storage", "256GB");
        samsungPhoneAttrs.put("color", "幻影白");
        manager.addProduct("手机", "Samsung", phoneCommonAttrs,
            "P006", "Galaxy S23 256GB 幻影白", new BigDecimal("4999"), samsungPhoneAttrs);
        
        // 显示所有商品
        manager.displayAllProducts();
        
        System.out.println("\n=== 商品详情展示 ===");
        // 展示特定商品详情
        manager.displayProduct("P001");
        System.out.println();
        manager.displayProduct("P004");
        
        // 显示享元统计信息
        System.out.println();
        ProductSpecificationFactory.printFlyweightInfo();
        
        // 展示内存使用优化效果
        System.out.println("\n=== 内存优化效果 ===");
        System.out.println("总商品数量: " + 6);
        System.out.println("享元对象数量: " + ProductSpecificationFactory.getFlyweightCount());
        System.out.println("内存节省率: " + 
            String.format("%.1f%%", (1.0 - (double)ProductSpecificationFactory.getFlyweightCount() / 6) * 100));
    }
}
```

### 12. 代理模式 (Proxy Pattern)

**场景**: 商品图片懒加载、用户权限控制、缓存代理

```java
/**
 * 商品服务接口
 * 参考：阿里云商品服务
 */
public interface ProductService {
    Product getProduct(String productId);
    List<Product> searchProducts(String keyword);
    boolean updateProduct(String productId, Product product);
    void deleteProduct(String productId);
}

/**
 * 真实商品服务实现
 */
public class RealProductService implements ProductService {
    private Map<String, Product> productDatabase;
    
    public RealProductService() {
        this.productDatabase = new HashMap<>();
        initializeProducts();
    }
    
    @Override
    public Product getProduct(String productId) {
        System.out.println("从数据库查询商品: " + productId);
        simulateLatency(); // 模拟数据库查询延迟
        return productDatabase.get(productId);
    }
    
    @Override
    public List<Product> searchProducts(String keyword) {
        System.out.println("在数据库中搜索商品: " + keyword);
        simulateLatency();
        
        return productDatabase.values().stream()
            .filter(product -> product.getName().toLowerCase()
                .contains(keyword.toLowerCase()))
            .collect(Collectors.toList());
    }
    
    @Override
    public boolean updateProduct(String productId, Product product) {
        System.out.println("更新数据库中的商品: " + productId);
        simulateLatency();
        
        productDatabase.put(productId, product);
        return true;
    }
    
    @Override
    public void deleteProduct(String productId) {
        System.out.println("从数据库删除商品: " + productId);
        simulateLatency();
        productDatabase.remove(productId);
    }
    
    private void simulateLatency() {
        try {
            Thread.sleep(1000); // 模拟1秒延迟
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    private void initializeProducts() {
        productDatabase.put("P001", new Product("P001", "iPhone 14", new BigDecimal("5999")));
        productDatabase.put("P002", new Product("P002", "MacBook Pro", new BigDecimal("15999")));
        productDatabase.put("P003", new Product("P003", "AirPods Pro", new BigDecimal("1899")));
    }
}

/**
 * 缓存代理 - 虚拟代理
 */
public class CacheProductServiceProxy implements ProductService {
    private RealProductService realService;
    private Map<String, Product> cache;
    private Map<String, List<Product>> searchCache;
    private Map<String, Long> cacheTimestamps;
    private static final long CACHE_DURATION = 5 * 60 * 1000; // 5分钟缓存
    
    public CacheProductServiceProxy() {
        this.cache = new HashMap<>();
        this.searchCache = new HashMap<>();
        this.cacheTimestamps = new HashMap<>();
    }
    
    @Override
    public Product getProduct(String productId) {
        // 检查缓存
        if (isCacheValid(productId)) {
            System.out.println("从缓存返回商品: " + productId);
            return cache.get(productId);
        }
        
        // 懒加载真实服务
        if (realService == null) {
            realService = new RealProductService();
        }
        
        // 从真实服务获取数据
        Product product = realService.getProduct(productId);
        
        // 更新缓存
        if (product != null) {
            cache.put(productId, product);
            cacheTimestamps.put(productId, System.currentTimeMillis());
        }
        
        return product;
    }
    
    @Override
    public List<Product> searchProducts(String keyword) {
        String searchKey = "search:" + keyword.toLowerCase();
        
        // 检查搜索缓存
        if (isCacheValid(searchKey)) {
            System.out.println("从缓存返回搜索结果: " + keyword);
            return searchCache.get(searchKey);
        }
        
        // 懒加载真实服务
        if (realService == null) {
            realService = new RealProductService();
        }
        
        // 从真实服务搜索
        List<Product> results = realService.searchProducts(keyword);
        
        // 更新搜索缓存
        searchCache.put(searchKey, results);
        cacheTimestamps.put(searchKey, System.currentTimeMillis());
        
        return results;
    }
    
    @Override
    public boolean updateProduct(String productId, Product product) {
        // 懒加载真实服务
        if (realService == null) {
            realService = new RealProductService();
        }
        
        boolean success = realService.updateProduct(productId, product);
        
        if (success) {
            // 更新缓存
            cache.put(productId, product);
            cacheTimestamps.put(productId, System.currentTimeMillis());
            
            // 清除相关搜索缓存
            clearSearchCache();
        }
        
        return success;
    }
    
    @Override
    public void deleteProduct(String productId) {
        // 懒加载真实服务
        if (realService == null) {
            realService = new RealProductService();
        }
        
        realService.deleteProduct(productId);
        
        // 从缓存中移除
        cache.remove(productId);
        cacheTimestamps.remove(productId);
        
        // 清除相关搜索缓存
        clearSearchCache();
    }
    
    private boolean isCacheValid(String key) {
        Long timestamp = cacheTimestamps.get(key);
        if (timestamp == null) {
            return false;
        }
        
        return (System.currentTimeMillis() - timestamp) < CACHE_DURATION;
    }
    
    private void clearSearchCache() {
        searchCache.clear();
        // 移除搜索相关的时间戳
        cacheTimestamps.entrySet().removeIf(entry -> 
            entry.getKey().startsWith("search:"));
    }
    
    public void clearCache() {
        cache.clear();
        searchCache.clear();
        cacheTimestamps.clear();
        System.out.println("缓存已清空");
    }
    
    public void printCacheStats() {
        System.out.println("=== 缓存统计 ===");
        System.out.println("商品缓存数量: " + cache.size());
        System.out.println("搜索缓存数量: " + searchCache.size());
    }
}

/**
 * 权限控制代理 - 保护代理
 */
public class SecurityProductServiceProxy implements ProductService {
    private ProductService target;
    private UserContext currentUser;
    
    public SecurityProductServiceProxy(ProductService target, UserContext currentUser) {
        this.target = target;
        this.currentUser = currentUser;
    }
    
    @Override
    public Product getProduct(String productId) {
        // 读取操作无需特殊权限
        return target.getProduct(productId);
    }
    
    @Override
    public List<Product> searchProducts(String keyword) {
        // 搜索操作无需特殊权限
        return target.searchProducts(keyword);
    }
    
    @Override
    public boolean updateProduct(String productId, Product product) {
        if (!hasWritePermission()) {
            throw new SecurityException("用户无商品更新权限: " + currentUser.getUsername());
        }
        
        // 记录操作日志
        logOperation("UPDATE", productId, currentUser.getUsername());
        
        return target.updateProduct(productId, product);
    }
    
    @Override
    public void deleteProduct(String productId) {
        if (!hasDeletePermission()) {
            throw new SecurityException("用户无商品删除权限: " + currentUser.getUsername());
        }
        
        // 记录操作日志
        logOperation("DELETE", productId, currentUser.getUsername());
        
        target.deleteProduct(productId);
    }
    
    private boolean hasWritePermission() {
        return currentUser.hasRole("ADMIN") || currentUser.hasRole("PRODUCT_MANAGER");
    }
    
    private boolean hasDeletePermission() {
        return currentUser.hasRole("ADMIN");
    }
    
    private void logOperation(String operation, String ProductId, String username) {
        System.out.println(String.format("[AUDIT] %s - User: %s, Operation: %s, ProductId: %s, Time: %s",
            "PRODUCT_SERVICE", username, operation, ProductId, LocalDateTime.now()));
    }
}

/**
 * 日志记录代理 - 智能代理
 */
public class LoggingProductServiceProxy implements ProductService {
    private ProductService target;
    private String serviceName;
    
    public LoggingProductServiceProxy(ProductService target, String serviceName) {
        this.target = target;
        this.serviceName = serviceName;
    }
    
    @Override
    public Product getProduct(String productId) {
        long startTime = System.currentTimeMillis();
        System.out.println("[LOG] 开始获取商品: " + productId);
        
        try {
            Product result = target.getProduct(productId);
            long duration = System.currentTimeMillis() - startTime;
            System.out.println(String.format("[LOG] 获取商品成功: %s, 耗时: %dms", 
                productId, duration));
            return result;
        } catch (Exception e) {
            long duration = System.currentTimeMillis() - startTime;
            System.out.println(String.format("[LOG] 获取商品失败: %s, 耗时: %dms, 错误: %s", 
                productId, duration, e.getMessage()));
            throw e;
        }
    }
    
    @Override
    public List<Product> searchProducts(String keyword) {
        long startTime = System.currentTimeMillis();
        System.out.println("[LOG] 开始搜索商品: " + keyword);
        
        try {
            List<Product> results = target.searchProducts(keyword);
            long duration = System.currentTimeMillis() - startTime;
            System.out.println(String.format("[LOG] 搜索商品成功: %s, 找到 %d 个结果, 耗时: %dms", 
                keyword, results.size(), duration));
            return results;
        } catch (Exception e) {
            long duration = System.currentTimeMillis() - startTime;
            System.out.println(String.format("[LOG] 搜索商品失败: %s, 耗时: %dms, 错误: %s", 
                keyword, duration, e.getMessage()));
            throw e;
        }
    }
    
    @Override
    public boolean updateProduct(String productId, Product product) {
        long startTime = System.currentTimeMillis();
        System.out.println("[LOG] 开始更新商品: " + productId);
        
        try {
            boolean result = target.updateProduct(productId, product);
            long duration = System.currentTimeMillis() - startTime;
            System.out.println(String.format("[LOG] 更新商品%s: %s, 耗时: %dms", 
                result ? "成功" : "失败", productId, duration));
            return result;
        } catch (Exception e) {
            long duration = System.currentTimeMillis() - startTime;
            System.out.println(String.format("[LOG] 更新商品异常: %s, 耗时: %dms, 错误: %s", 
                productId, duration, e.getMessage()));
            throw e;
        }
    }
    
    @Override
    public void deleteProduct(String productId) {
        long startTime = System.currentTimeMillis();
        System.out.println("[LOG] 开始删除商品: " + productId);
        
        try {
            target.deleteProduct(productId);
            long duration = System.currentTimeMillis() - startTime;
            System.out.println(String.format("[LOG] 删除商品成功: %s, 耗时: %dms", 
                productId, duration));
        } catch (Exception e) {
            long duration = System.currentTimeMillis() - startTime;
            System.out.println(String.format("[LOG] 删除商品异常: %s, 耗时: %dms, 错误: %s", 
                productId, duration, e.getMessage()));
            throw e;
        }
    }
}

/**
 * 用户上下文
 */
public class UserContext {
    private String username;
    private Set<String> roles;
    
    public UserContext(String username, Set<String> roles) {
        this.username = username;
        this.roles = new HashSet<>(roles);
    }
    
    public String getUsername() { return username; }
    public boolean hasRole(String role) { return roles.contains(role); }
    public Set<String> getRoles() { return new HashSet<>(roles); }
}

/**
 * 代理模式使用示例
 */
public class ProxyPatternExample {
    
    public void demonstrateProxyPattern() {
        System.out.println("=== 代理模式演示 ===\n");
        
        // 1. 缓存代理演示
        demonstrateCacheProxy();
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // 2. 权限控制代理演示
        demonstrateSecurityProxy();
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // 3. 多重代理演示
        demonstrateMultipleProxies();
    }
    
    private void demonstrateCacheProxy() {
        System.out.println("=== 缓存代理演示 ===");
        
        CacheProductServiceProxy cacheProxy = new CacheProductServiceProxy();
        
        // 第一次查询 - 从数据库
        System.out.println("第一次查询:");
        Product product1 = cacheProxy.getProduct("P001");
        System.out.println("查询结果: " + product1.getName());
        
        System.out.println("\n第二次查询 (应该从缓存):");
        Product product2 = cacheProxy.getProduct("P001");
        System.out.println("查询结果: " + product2.getName());
        
        // 搜索演示
        System.out.println("\n搜索演示:");
        List<Product> results1 = cacheProxy.searchProducts("iPhone");
        System.out.println("搜索结果数量: " + results1.size());
        
        System.out.println("\n第二次相同搜索 (应该从缓存):");
        List<Product> results2 = cacheProxy.searchProducts("iPhone");
        System.out.println("搜索结果数量: " + results2.size());
        
        cacheProxy.printCacheStats();
    }
    
    private void demonstrateSecurityProxy() {
        System.out.println("=== 权限控制代理演示 ===");
        
        // 创建真实服务和缓存代理
        ProductService cacheProxy = new CacheProductServiceProxy();
        
        // 普通用户
        UserContext normalUser = new UserContext("john", Set.of("USER"));
        ProductService normalUserProxy = new SecurityProductServiceProxy(cacheProxy, normalUser);
        
        // 管理员用户
        UserContext adminUser = new UserContext("admin", Set.of("ADMIN", "PRODUCT_MANAGER"));
        ProductService adminProxy = new SecurityProductServiceProxy(cacheProxy, adminUser);
        
        // 普通用户操作
        System.out.println("普通用户操作:");
        Product product = normalUserProxy.getProduct("P001");
        System.out.println("查询成功: " + product.getName());
        
        try {
            normalUserProxy.updateProduct("P001", new Product("P001", "Updated iPhone", new BigDecimal("6999")));
        } catch (SecurityException e) {
            System.out.println("权限被拒绝: " + e.getMessage());
        }
        
        // 管理员操作
        System.out.println("\n管理员操作:");
        boolean updated = adminProxy.updateProduct("P001", 
            new Product("P001", "iPhone 14 Pro", new BigDecimal("6999")));
        System.out.println("更新成功: " + updated);
        
        try {
            adminProxy.deleteProduct("P002");
            System.out.println("删除成功");
        } catch (Exception e) {
            System.out.println("删除失败: " + e.getMessage());
        }
    }
    
    private void demonstrateMultipleProxies() {
        System.out.println("=== 多重代理演示 ===");
        
        // 构建代理链: 日志 -> 安全 -> 缓存 -> 真实服务
        ProductService realService = new RealProductService();
        ProductService cacheProxy = new CacheProductServiceProxy();
        
        UserContext adminUser = new UserContext("admin", Set.of("ADMIN"));
        ProductService securityProxy = new SecurityProductServiceProxy(cacheProxy, adminUser);
        ProductService loggingProxy = new LoggingProductServiceProxy(securityProxy, "ProductService");
        
        System.out.println("通过代理链查询商品:");
        Product product = loggingProxy.getProduct("P001");
        System.out.println("最终结果: " + product.getName());
        
        System.out.println("\n通过代理链更新商品:");
        boolean updated = loggingProxy.updateProduct("P001", 
            new Product("P001", "New iPhone", new BigDecimal("7999")));
        System.out.println("更新结果: " + updated);
    }
}
```

---

## 行为型模式

### 13. 责任链模式 (Chain of Responsibility Pattern)

**场景**: 订单审核流程、价格计算链、异常处理链

```java
/**
 * 订单处理抽象处理器
 * 参考：阿里巴巴订单审核系统
 */
public abstract class OrderProcessor {
    protected OrderProcessor nextProcessor;
    protected String processorName;
    
    public OrderProcessor(String processorName) {
        this.processorName = processorName;
    }
    
    public void setNext(OrderProcessor next) {
        this.nextProcessor = next;
    }
    
    public final ProcessResult process(OrderRequest request) {
        System.out.println("[" + processorName + "] 开始处理订单: " + request.getOrderId());
        
        // 执行当前处理器的逻辑
        ProcessResult result = doProcess(request);
        
        if (result.isSuccess()) {
            System.out.println("[" + processorName + "] 处理成功");
            
            // 如果有下一个处理器且当前处理成功，继续处理
            if (nextProcessor != null) {
                return nextProcessor.process(request);
            }
            return result;
        } else {
            System.out.println("[" + processorName + "] 处理失败: " + result.getMessage());
            return result; // 处理失败，终止链
        }
    }
    
    protected abstract ProcessResult doProcess(OrderRequest request);
}

/**
 * 订单基础信息验证器
 */
public class BasicValidationProcessor extends OrderProcessor {
    
    public BasicValidationProcessor() {
        super("基础信息验证");
    }
    
    @Override
    protected ProcessResult doProcess(OrderRequest request) {
        // 验证订单基础信息
        if (request.getOrderId() == null || request.getOrderId().trim().isEmpty()) {
            return ProcessResult.failure("订单ID不能为空");
        }
        
        if (request.getUserId() == null || request.getUserId().trim().isEmpty()) {
            return ProcessResult.failure("用户ID不能为空");
        }
        
        if (request.getItems() == null || request.getItems().isEmpty()) {
            return ProcessResult.failure("订单商品不能为空");
        }
        
        if (request.getShippingAddress() == null) {
            return ProcessResult.failure("收货地址不能为空");
        }
        
        return ProcessResult.success("基础信息验证通过");
    }
}

/**
 * 库存检查处理器
 */
public class InventoryCheckProcessor extends OrderProcessor {
    private InventoryService inventoryService;
    
    public InventoryCheckProcessor() {
        super("库存检查");
        this.inventoryService = new InventoryService();
    }
    
    @Override
    protected ProcessResult doProcess(OrderRequest request) {
        for (OrderItem item : request.getItems()) {
            int availableStock = inventoryService.getAvailableStock(item.getProductId());
            
            if (availableStock < item.getQuantity()) {
                return ProcessResult.failure(
                    String.format("商品 %s 库存不足，需要 %d 件，可用 %d 件",
                        item.getProductId(), item.getQuantity(), availableStock));
            }
        }
        
        // 预锁定库存
        boolean lockSuccess = inventoryService.lockInventory(request.getItems());
        if (!lockSuccess) {
            return ProcessResult.failure("库存锁定失败");
        }
        
        return ProcessResult.success("库存检查通过");
    }
}

/**
 * 风控检查处理器
 */
public class RiskControlProcessor extends OrderProcessor {
    private RiskControlService riskService;
    
    public RiskControlProcessor() {
        super("风控检查");
        this.riskService = new RiskControlService();
    }
    
    @Override
    protected ProcessResult doProcess(OrderRequest request) {
        // 用户风险评估
        RiskLevel userRisk = riskService.evaluateUserRisk(request.getUserId());
        if (userRisk == RiskLevel.HIGH) {
            return ProcessResult.failure("用户风险等级过高，订单被拒绝");
        }
        
        // 订单金额风险检查
        BigDecimal totalAmount = calculateTotalAmount(request);
        if (totalAmount.compareTo(new BigDecimal("10000")) > 0 && userRisk == RiskLevel.MEDIUM) {
            return ProcessResult.failure("高金额订单需要人工审核");
        }
        
        // 异常行为检测
        if (riskService.detectAbnormalBehavior(request)) {
            return ProcessResult.failure("检测到异常购买行为");
        }
        
        return ProcessResult.success("风控检查通过");
    }
    
    private BigDecimal calculateTotalAmount(OrderRequest request) {
        return request.getItems().stream()
            .map(item -> item.getPrice().multiply(BigDecimal.valueOf(item.getQuantity())))
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
}

/**
 * 价格计算处理器
 */
public class PriceCalculationProcessor extends OrderProcessor {
    private PriceCalculatorService priceService;
    
    public PriceCalculationProcessor() {
        super("价格计算");
        this.priceService = new PriceCalculatorService();
    }
    
    @Override
    protected ProcessResult doProcess(OrderRequest request) {
        try {
            // 计算原价
            BigDecimal originalPrice = calculateOriginalPrice(request);
            
            // 应用优惠券
            BigDecimal couponDiscount = BigDecimal.ZERO;
            if (request.getCouponId() != null) {
                couponDiscount = priceService.calculateCouponDiscount(
                    request.getCouponId(), originalPrice);
            }
            
            // 计算会员折扣
            BigDecimal memberDiscount = priceService.calculateMemberDiscount(
                request.getUserId(), originalPrice);
            
            // 计算最终价格
            BigDecimal finalPrice = originalPrice.subtract(couponDiscount).subtract(memberDiscount);
            
            // 更新订单价格信息
            request.setOriginalPrice(originalPrice);
            request.setCouponDiscount(couponDiscount);
            request.setMemberDiscount(memberDiscount);
            request.setFinalPrice(finalPrice);
            
            System.out.println("  原价: ¥" + originalPrice);
            System.out.println("  优惠券减免: ¥" + couponDiscount);
            System.out.println("  会员折扣: ¥" + memberDiscount);
            System.out.println("  最终价格: ¥" + finalPrice);
            
            return ProcessResult.success("价格计算完成");
            
        } catch (Exception e) {
            return ProcessResult.failure("价格计算失败: " + e.getMessage());
        }
    }
    
    private BigDecimal calculateOriginalPrice(OrderRequest request) {
        return request.getItems().stream()
            .map(item -> item.getPrice().multiply(BigDecimal.valueOf(item.getQuantity())))
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
}

/**
 * 支付处理器
 */
public class PaymentProcessor extends OrderProcessor {
    private PaymentService paymentService;
    
    public PaymentProcessor() {
        super("支付处理");
        this.paymentService = new PaymentService();
    }
    
    @Override
    protected ProcessResult doProcess(OrderRequest request) {
        try {
            PaymentRequest paymentRequest = new PaymentRequest(
                request.getOrderId(),
                request.getUserId(),
                request.getFinalPrice(),
                request.getPaymentMethod()
            );
            
            PaymentResult paymentResult = paymentService.processPayment(paymentRequest);
            
            if (paymentResult.isSuccess()) {
                request.setPaymentId(paymentResult.getPaymentId());
                return ProcessResult.success("支付处理成功，支付ID: " + paymentResult.getPaymentId());
            } else {
                return ProcessResult.failure("支付失败: " + paymentResult.getMessage());
            }
            
        } catch (Exception e) {
            return ProcessResult.failure("支付处理异常: " + e.getMessage());
        }
    }
}

/**
 * 订单创建处理器
 */
public class OrderCreationProcessor extends OrderProcessor {
    private OrderService orderService;
    
    public OrderCreationProcessor() {
        super("订单创建");
        this.orderService = new OrderService();
    }
    
    @Override
    protected ProcessResult doProcess(OrderRequest request) {
        try {
            // 创建订单记录
            Order order = new Order();
            order.setOrderId(request.getOrderId());
            order.setUserId(request.getUserId());
            order.setItems(request.getItems());
            order.setShippingAddress(request.getShippingAddress());
            order.setOriginalAmount(request.getOriginalPrice());
            order.setFinalAmount(request.getFinalPrice());
            order.setPaymentId(request.getPaymentId());
            order.setStatus(OrderStatus.PAID);
            order.setCreateTime(LocalDateTime.now());
            
            boolean saved = orderService.saveOrder(order);
            
            if (saved) {
                return ProcessResult.success("订单创建成功");
            } else {
                return ProcessResult.failure("订单保存失败");
            }
            
        } catch (Exception e) {
            return ProcessResult.failure("订单创建异常: " + e.getMessage());
        }
    }
}

/**
 * 通知处理器
 */
public class NotificationProcessor extends OrderProcessor {
    private NotificationService notificationService;
    
    public NotificationProcessor() {
        super("通知发送");
        this.notificationService = new NotificationService();
    }
    
    @Override
    protected ProcessResult doProcess(OrderRequest request) {
        try {
            // 发送订单确认短信
            notificationService.sendOrderConfirmSMS(request.getUserId(), request.getOrderId());
            
            // 发送订单确认邮件
            notificationService.sendOrderConfirmEmail(request.getUserId(), request.getOrderId());
            
            // 推送APP通知
            notificationService.sendPushNotification(request.getUserId(), 
                "您的订单 " + request.getOrderId() + " 已确认，正在准备发货");
            
            return ProcessResult.success("通知发送完成");
            
        } catch (Exception e) {
            // 通知失败不应该影响订单处理
            System.out.println("通知发送失败，但不影响订单: " + e.getMessage());
            return ProcessResult.success("通知发送完成（部分失败）");
        }
    }
}

/**
 * 订单处理链构建器
 */
public class OrderProcessorChainBuilder {
    
    public static OrderProcessor buildStandardChain() {
        // 构建标准订单处理链
        OrderProcessor basicValidation = new BasicValidationProcessor();
        OrderProcessor inventoryCheck = new InventoryCheckProcessor();
        OrderProcessor riskControl = new RiskControlProcessor();
        OrderProcessor priceCalculation = new PriceCalculationProcessor();
        OrderProcessor payment = new PaymentProcessor();
        OrderProcessor orderCreation = new OrderCreationProcessor();
        OrderProcessor notification = new NotificationProcessor();
        
        // 连接处理链
        basicValidation.setNext(inventoryCheck);
        inventoryCheck.setNext(riskControl);
        riskControl.setNext(priceCalculation);
        priceCalculation.setNext(payment);
        payment.setNext(orderCreation);
        orderCreation.setNext(notification);
        
        return basicValidation;
    }
    
    public static OrderProcessor buildQuickChain() {
        // 构建快速订单处理链（跳过风控）
        OrderProcessor basicValidation = new BasicValidationProcessor();
        OrderProcessor inventoryCheck = new InventoryCheckProcessor();
        OrderProcessor priceCalculation = new PriceCalculationProcessor();
        OrderProcessor payment = new PaymentProcessor();
        OrderProcessor orderCreation = new OrderCreationProcessor();
        
        basicValidation.setNext(inventoryCheck);
        inventoryCheck.setNext(priceCalculation);
        priceCalculation.setNext(payment);
        payment.setNext(orderCreation);
        
        return basicValidation;
    }
    
    public static OrderProcessor buildVipChain() {
        // 构建VIP订单处理链（添加特殊处理）
        OrderProcessor basicValidation = new BasicValidationProcessor();
        OrderProcessor inventoryCheck = new InventoryCheckProcessor();
        OrderProcessor vipPriceCalculation = new VipPriceCalculationProcessor();
        OrderProcessor payment = new PaymentProcessor();
        OrderProcessor orderCreation = new OrderCreationProcessor();
        OrderProcessor vipNotification = new VipNotificationProcessor();
        
        basicValidation.setNext(inventoryCheck);
        inventoryCheck.setNext(vipPriceCalculation);
        vipPriceCalculation.setNext(payment);
        payment.setNext(orderCreation);
        orderCreation.setNext(vipNotification);
        
        return basicValidation;
    }
}

/**
 * VIP价格计算处理器
 */
public class VipPriceCalculationProcessor extends PriceCalculationProcessor {
    
    public VipPriceCalculationProcessor() {
        super();
        this.processorName = "VIP价格计算";
    }
    
    @Override
    protected ProcessResult doProcess(OrderRequest request) {
        // 先执行基础价格计算
        ProcessResult baseResult = super.doProcess(request);
        
        if (baseResult.isSuccess()) {
            // 应用VIP额外折扣
            BigDecimal currentPrice = request.getFinalPrice();
            BigDecimal vipDiscount = currentPrice.multiply(new BigDecimal("0.05")); // VIP额外5%折扣
            BigDecimal vipPrice = currentPrice.subtract(vipDiscount);
            
            request.setFinalPrice(vipPrice);
            System.out.println("  VIP额外折扣: ¥" + vipDiscount);
            System.out.println("  VIP最终价格: ¥" + vipPrice);
            
            return ProcessResult.success("VIP价格计算完成");
        }
        
        return baseResult;
    }
}

/**
 * VIP通知处理器
 */
public class VipNotificationProcessor extends NotificationProcessor {
    
    public VipNotificationProcessor() {
        super();
        this.processorName = "VIP通知发送";
    }
    
    @Override
    protected ProcessResult doProcess(OrderRequest request) {
        // 先执行基础通知
        ProcessResult baseResult = super.doProcess(request);
        
        // 发送VIP专属通知
        try {
            System.out.println("  发送VIP专属服务通知");
            System.out.println("  安排VIP客服跟进");
            System.out.println("  优先发货处理");
            
            return ProcessResult.success("VIP通知发送完成");
        } catch (Exception e) {
            return ProcessResult.success("VIP通知发送完成（部分失败）");
        }
    }
}

/**
 * 处理结果
 */
public class ProcessResult {
    private boolean success;
    private String message;
    
    private ProcessResult(boolean success, String message) {
        this.success = success;
        this.message = message;
    }
    
    public static ProcessResult success(String message) {
        return new ProcessResult(true, message);
    }
    
    public static ProcessResult failure(String message) {
        return new ProcessResult(false, message);
    }
    
    public boolean isSuccess() { return success; }
    public String getMessage() { return message; }
}

/**
 * 支持类定义
 */
enum RiskLevel {
    LOW, MEDIUM, HIGH
}

enum OrderStatus {
    PENDING, PAID, SHIPPED, DELIVERED, CANCELLED
}

class RiskControlService {
    public RiskLevel evaluateUserRisk(String userId) {
        // 模拟风险评估
        return RiskLevel.LOW;
    }
    
    public boolean detectAbnormalBehavior(OrderRequest request) {
        // 模拟异常行为检测
        return false;
    }
}

class PriceCalculatorService {
    public BigDecimal calculateCouponDiscount(String couponId, BigDecimal originalPrice) {
        // 模拟优惠券折扣计算
        return new BigDecimal("50");
    }
    
    public BigDecimal calculateMemberDiscount(String userId, BigDecimal originalPrice) {
        // 模拟会员折扣计算
        return originalPrice.multiply(new BigDecimal("0.02")); // 2%会员折扣
    }
}

/**
 * 责任链模式使用示例
 */
public class ChainOfResponsibilityExample {
    
    public void demonstrateOrderProcessing() {
        System.out.println("=== 责任链模式 - 订单处理演示 ===\n");
        
        // 创建订单请求
        OrderRequest request = createSampleOrder();
        
        // 标准订单处理
        System.out.println("1. 标准订单处理链:");
        System.out.println("-".repeat(40));
        OrderProcessor standardChain = OrderProcessorChainBuilder.buildStandardChain();
        ProcessResult result1 = standardChain.process(request);
        System.out.println("最终结果: " + (result1.isSuccess() ? "成功" : "失败"));
        if (!result1.isSuccess()) {
            System.out.println("失败原因: " + result1.getMessage());
        }
        
        System.out.println("\n" + "=".repeat(60) + "\n");
        
        // VIP订单处理
        System.out.println("2. VIP订单处理链:");
        System.out.println("-".repeat(40));
        OrderRequest vipRequest = createVipOrder();
        OrderProcessor vipChain = OrderProcessorChainBuilder.buildVipChain();
        ProcessResult result2 = vipChain.process(vipRequest);
        System.out.println("最终结果: " + (result2.isSuccess() ? "成功" : "失败"));
        
        System.out.println("\n" + "=".repeat(60) + "\n");
        
        // 模拟处理失败的情况
        System.out.println("3. 异常订单处理（库存不足）:");
        System.out.println("-".repeat(40));
        OrderRequest invalidRequest = createInvalidOrder();
        OrderProcessor chain = OrderProcessorChainBuilder.buildStandardChain();
        ProcessResult result3 = chain.process(invalidRequest);
        System.out.println("最终结果: " + (result3.isSuccess() ? "成功" : "失败"));
        if (!result3.isSuccess()) {
            System.out.println("失败原因: " + result3.getMessage());
        }
    }
    
    private OrderRequest createSampleOrder() {
        OrderRequest request = new OrderRequest();
        request.setOrderId("ORD20231201001");
        request.setUserId("U001");
        
        List<OrderItem> items = Arrays.asList(
            new OrderItem("P001", "iPhone 14", new BigDecimal("5999"), 1),
            new OrderItem("P002", "AirPods Pro", new BigDecimal("1899"), 1)
        );
        request.setItems(items);
        
        request.setShippingAddress(new ShippingAddress("张三", "13800138000", "北京市朝阳区"));
        request.setPaymentMethod(PaymentMethod.ALIPAY);
        request.setCouponId("COUPON001");
        
        return request;
    }
    
    private OrderRequest createVipOrder() {
        OrderRequest request = createSampleOrder();
        request.setOrderId("VIP20231201001");
        request.setUserId("VIP001");
        return request;
    }
    
    private OrderRequest createInvalidOrder() {
        OrderRequest request = new OrderRequest();
        request.setOrderId("ORD20231201002");
        request.setUserId("U002");
        
        // 添加库存不足的商品
        List<OrderItem> items = Arrays.asList(
            new OrderItem("P999", "Out of Stock Product", new BigDecimal("999"), 999)
        );
        request.setItems(items);
        
        request.setShippingAddress(new ShippingAddress("李四", "13900139000", "上海市浦东新区"));
        request.setPaymentMethod(PaymentMethod.WECHAT_PAY);
        
        return request;
    }
}

### 14. 命令模式 (Command Pattern)

**场景**: 订单操作撤销/重做、批量操作、宏命令

```java
/**
 * 命令接口
 * 参考：美团外卖订单操作系统
 */
public interface Command {
    void execute();
    void undo();
    String getDescription();
}

/**
 * 订单接收器 - 真正执行操作的对象
 */
public class OrderReceiver {
    private Map<String, Order> orders = new HashMap<>();
    private InventoryService inventoryService = new InventoryService();
    private PaymentService paymentService = new PaymentService();
    
    public boolean createOrder(Order order) {
        System.out.println("创建订单: " + order.getOrderId());
        orders.put(order.getOrderId(), order);
        
        // 锁定库存
        boolean inventoryLocked = inventoryService.lockInventory(order.getItems());
        if (inventoryLocked) {
            order.setStatus(OrderStatus.CREATED);
            return true;
        }
        return false;
    }
    
    public boolean cancelOrder(String orderId) {
        Order order = orders.get(orderId);
        if (order != null && order.getStatus() != OrderStatus.CANCELLED) {
            System.out.println("取消订单: " + orderId);
            
            // 释放库存
            inventoryService.releaseInventory(order.getItems());
            
            // 如果已支付，处理退款
            if (order.getStatus() == OrderStatus.PAID) {
                paymentService.refund(order.getPaymentId(), order.getFinalAmount());
            }
            
            order.setPreviousStatus(order.getStatus());
            order.setStatus(OrderStatus.CANCELLED);
            return true;
        }
        return false;
    }
    
    public boolean restoreOrder(String orderId) {
        Order order = orders.get(orderId);
        if (order != null && order.getStatus() == OrderStatus.CANCELLED) {
            System.out.println("恢复订单: " + orderId);
            
            // 重新锁定库存
            boolean inventoryLocked = inventoryService.lockInventory(order.getItems());
            if (inventoryLocked) {
                order.setStatus(order.getPreviousStatus());
                return true;
            }
        }
        return false;
    }
    
    public boolean updateOrderStatus(String orderId, OrderStatus newStatus) {
        Order order = orders.get(orderId);
        if (order != null) {
            System.out.println("更新订单状态: " + orderId + " -> " + newStatus);
            order.setPreviousStatus(order.getStatus());
            order.setStatus(newStatus);
            return true;
        }
        return false;
    }
    
    public boolean processPayment(String orderId, BigDecimal amount) {
        Order order = orders.get(orderId);
        if (order != null && order.getStatus() == OrderStatus.CREATED) {
            System.out.println("处理订单支付: " + orderId + ", 金额: ¥" + amount);
            
            PaymentRequest paymentRequest = new PaymentRequest(orderId, order.getUserId(), amount, PaymentMethod.ALIPAY);
            PaymentResult result = paymentService.processPayment(paymentRequest);
            
            if (result.isSuccess()) {
                order.setPaymentId(result.getPaymentId());
                order.setPreviousStatus(order.getStatus());
                order.setStatus(OrderStatus.PAID);
                return true;
            }
        }
        return false;
    }
    
    public Order getOrder(String orderId) {
        return orders.get(orderId);
    }
}

/**
 * 创建订单命令
 */
public class CreateOrderCommand implements Command {
    private OrderReceiver receiver;
    private Order order;
    private boolean executed = false;
    
    public CreateOrderCommand(OrderReceiver receiver, Order order) {
        this.receiver = receiver;
        this.order = order;
    }
    
    @Override
    public void execute() {
        if (!executed) {
            executed = receiver.createOrder(order);
        }
    }
    
    @Override
    public void undo() {
        if (executed) {
            receiver.cancelOrder(order.getOrderId());
            executed = false;
        }
    }
    
    @Override
    public String getDescription() {
        return "创建订单: " + order.getOrderId();
    }
}

/**
 * 取消订单命令
 */
public class CancelOrderCommand implements Command {
    private OrderReceiver receiver;
    private String orderId;
    private boolean executed = false;
    
    public CancelOrderCommand(OrderReceiver receiver, String orderId) {
        this.receiver = receiver;
        this.orderId = orderId;
    }
    
    @Override
    public void execute() {
        if (!executed) {
            executed = receiver.cancelOrder(orderId);
        }
    }
    
    @Override
    public void undo() {
        if (executed) {
            receiver.restoreOrder(orderId);
            executed = false;
        }
    }
    
    @Override
    public String getDescription() {
        return "取消订单: " + orderId;
    }
}

/**
 * 支付命令
 */
public class PaymentCommand implements Command {
    private OrderReceiver receiver;
    private String orderId;
    private BigDecimal amount;
    private boolean executed = false;
    
    public PaymentCommand(OrderReceiver receiver, String orderId, BigDecimal amount) {
        this.receiver = receiver;
        this.orderId = orderId;
        this.amount = amount;
    }
    
    @Override
    public void execute() {
        if (!executed) {
            executed = receiver.processPayment(orderId, amount);
        }
    }
    
    @Override
    public void undo() {
        if (executed) {
            // 支付的撤销比较复杂，这里简化处理
            Order order = receiver.getOrder(orderId);
            if (order != null) {
                receiver.updateOrderStatus(orderId, order.getPreviousStatus());
            }
            executed = false;
        }
    }
    
    @Override
    public String getDescription() {
        return "订单支付: " + orderId + ", 金额: ¥" + amount;
    }
}

/**
 * 宏命令 - 组合多个命令
 */
public class MacroCommand implements Command {
    private List<Command> commands;
    private String description;
    
    public MacroCommand(String description) {
        this.commands = new ArrayList<>();
        this.description = description;
    }
    
    public void addCommand(Command command) {
        commands.add(command);
    }
    
    @Override
    public void execute() {
        System.out.println("执行宏命令: " + description);
        for (Command command : commands) {
            command.execute();
        }
    }
    
    @Override
    public void undo() {
        System.out.println("撤销宏命令: " + description);
        // 反向撤销所有命令
        for (int i = commands.size() - 1; i >= 0; i--) {
            commands.get(i).undo();
        }
    }
    
    @Override
    public String getDescription() {
        return description + " (包含 " + commands.size() + " 个操作)";
    }
}

/**
 * 命令调用器 - 负责调用命令和管理历史
 */
public class OrderInvoker {
    private Stack<Command> commandHistory = new Stack<>();
    private Stack<Command> undoHistory = new Stack<>();
    private static final int MAX_HISTORY_SIZE = 100;
    
    public void executeCommand(Command command) {
        System.out.println("执行命令: " + command.getDescription());
        command.execute();
        
        // 添加到历史记录
        commandHistory.push(command);
        
        // 清空重做历史
        undoHistory.clear();
        
        // 限制历史记录大小
        if (commandHistory.size() > MAX_HISTORY_SIZE) {
            commandHistory.remove(0);
        }
    }
    
    public boolean undo() {
        if (!commandHistory.isEmpty()) {
            Command command = commandHistory.pop();
            System.out.println("撤销命令: " + command.getDescription());
            command.undo();
            undoHistory.push(command);
            return true;
        }
        System.out.println("没有可撤销的命令");
        return false;
    }
    
    public boolean redo() {
        if (!undoHistory.isEmpty()) {
            Command command = undoHistory.pop();
            System.out.println("重做命令: " + command.getDescription());
            command.execute();
            commandHistory.push(command);
            return true;
        }
        System.out.println("没有可重做的命令");
        return false;
    }
    
    public void showHistory() {
        System.out.println("=== 命令历史 ===");
        if (commandHistory.isEmpty()) {
            System.out.println("暂无历史记录");
            return;
        }
        
        for (int i = 0; i < commandHistory.size(); i++) {
            System.out.println((i + 1) + ". " + commandHistory.get(i).getDescription());
        }
    }
    
    public void clearHistory() {
        commandHistory.clear();
        undoHistory.clear();
        System.out.println("历史记录已清空");
    }
}

/**
 * 批量订单操作命令
 */
public class BatchOrderCommand implements Command {
    private List<Command> orderCommands;
    private String batchId;
    
    public BatchOrderCommand(String batchId) {
        this.batchId = batchId;
        this.orderCommands = new ArrayList<>();
    }
    
    public void addOrderCommand(Command command) {
        orderCommands.add(command);
    }
    
    @Override
    public void execute() {
        System.out.println("执行批量订单操作: " + batchId);
        for (Command command : orderCommands) {
            try {
                command.execute();
            } catch (Exception e) {
                System.out.println("批量操作中发生错误: " + e.getMessage());
                // 可以选择继续或回滚
            }
        }
    }
    
    @Override
    public void undo() {
        System.out.println("撤销批量订单操作: " + batchId);
        // 反向撤销
        for (int i = orderCommands.size() - 1; i >= 0; i--) {
            try {
                orderCommands.get(i).undo();
            } catch (Exception e) {
                System.out.println("批量撤销中发生错误: " + e.getMessage());
            }
        }
    }
    
    @Override
    public String getDescription() {
        return "批量订单操作: " + batchId + " (包含 " + orderCommands.size() + " 个订单)";
    }
}

/**
 * 命令模式使用示例
 */
public class CommandPatternExample {
    
    public void demonstrateCommands() {
        System.out.println("=== 命令模式演示 ===\n");
        
        // 创建接收器和调用器
        OrderReceiver receiver = new OrderReceiver();
        OrderInvoker invoker = new OrderInvoker();
        
        // 1. 基础命令操作
        demonstrateBasicCommands(receiver, invoker);
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // 2. 撤销和重做
        demonstrateUndoRedo(invoker);
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // 3. 宏命令
        demonstrateMacroCommands(receiver, invoker);
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // 4. 批量命令
        demonstrateBatchCommands(receiver, invoker);
    }
    
    private void demonstrateBasicCommands(OrderReceiver receiver, OrderInvoker invoker) {
        System.out.println("1. 基础命令操作:");
        
        // 创建订单
        Order order1 = createSampleOrder("ORD001", "U001");
        Command createCmd1 = new CreateOrderCommand(receiver, order1);
        invoker.executeCommand(createCmd1);
        
        // 处理支付
        Command payCmd1 = new PaymentCommand(receiver, "ORD001", new BigDecimal("999"));
        invoker.executeCommand(payCmd1);
        
        // 创建另一个订单
        Order order2 = createSampleOrder("ORD002", "U002");
        Command createCmd2 = new CreateOrderCommand(receiver, order2);
        invoker.executeCommand(createCmd2);
    }
    
    private void demonstrateUndoRedo(OrderInvoker invoker) {
        System.out.println("2. 撤销和重做操作:");
        
        // 显示历史
        invoker.showHistory();
        
        // 撤销操作
        System.out.println("\n执行撤销:");
        invoker.undo();
        invoker.undo();
        
        // 重做操作
        System.out.println("\n执行重做:");
        invoker.redo();
    }
    
    private void demonstrateMacroCommands(OrderReceiver receiver, OrderInvoker invoker) {
        System.out.println("3. 宏命令操作:");
        
        // 创建宏命令：创建订单 + 支付
        MacroCommand createAndPayMacro = new MacroCommand("创建订单并支付");
        
        Order order3 = createSampleOrder("ORD003", "U003");
        createAndPayMacro.addCommand(new CreateOrderCommand(receiver, order3));
        createAndPayMacro.addCommand(new PaymentCommand(receiver, "ORD003", new BigDecimal("1599")));
        
        // 执行宏命令
        invoker.executeCommand(createAndPayMacro);
        
        System.out.println("\n撤销宏命令:");
        invoker.undo();
    }
    
    private void demonstrateBatchCommands(OrderReceiver receiver, OrderInvoker invoker) {
        System.out.println("4. 批量命令操作:");
        
        // 创建批量订单命令
        BatchOrderCommand batchCmd = new BatchOrderCommand("BATCH001");
        
        // 添加多个订单创建命令
        for (int i = 4; i <= 6; i++) {
            Order order = createSampleOrder("ORD00" + i, "U00" + i);
            batchCmd.addOrderCommand(new CreateOrderCommand(receiver, order));
        }
        
        // 执行批量命令
        invoker.executeCommand(batchCmd);
        
        // 显示最终历史
        System.out.println("\n最终命令历史:");
        invoker.showHistory();
    }
    
    private Order createSampleOrder(String orderId, String userId) {
        Order order = new Order();
        order.setOrderId(orderId);
        order.setUserId(userId);
        order.setItems(Arrays.asList(
            new OrderItem("P001", "测试商品", new BigDecimal("999"), 1)
        ));
        order.setFinalAmount(new BigDecimal("999"));
        order.setCreateTime(LocalDateTime.now());
        return order;
    }
}

### 15. 解释器模式 (Interpreter Pattern)

**场景**: 优惠规则表达式解析、搜索查询解析

```java
/**
 * 表达式接口
 * 参考：淘宝优惠规则引擎
 */
public interface Expression {
    boolean interpret(OrderContext context);
    String toString();
}

/**
 * 订单上下文
 */
public class OrderContext {
    private BigDecimal totalAmount;
    private int itemCount;
    private String userLevel;
    private String productCategory;
    private boolean isFirstOrder;
    private LocalDateTime orderTime;
    private Map<String, Object> attributes;
    
    public OrderContext(BigDecimal totalAmount, int itemCount, String userLevel, 
                       String productCategory, boolean isFirstOrder) {
        this.totalAmount = totalAmount;
        this.itemCount = itemCount;
        this.userLevel = userLevel;
        this.productCategory = productCategory;
        this.isFirstOrder = isFirstOrder;
        this.orderTime = LocalDateTime.now();
        this.attributes = new HashMap<>();
    }
    
    // Getters and Setters
    public BigDecimal getTotalAmount() { return totalAmount; }
    public int getItemCount() { return itemCount; }
    public String getUserLevel() { return userLevel; }
    public String getProductCategory() { return productCategory; }
    public boolean isFirstOrder() { return isFirstOrder; }
    public LocalDateTime getOrderTime() { return orderTime; }
    
    public void setAttribute(String key, Object value) {
        attributes.put(key, value);
    }
    
    public Object getAttribute(String key) {
        return attributes.get(key);
    }
}

/**
 * 终结符表达式 - 金额条件
 */
public class AmountExpression implements Expression {
    private BigDecimal targetAmount;
    private String operator; // ">=", ">", "<=", "<", "=="
    
    public AmountExpression(String operator, BigDecimal targetAmount) {
        this.operator = operator;
        this.targetAmount = targetAmount;
    }
    
    @Override
    public boolean interpret(OrderContext context) {
        BigDecimal orderAmount = context.getTotalAmount();
        
        switch (operator) {
            case ">=":
                return orderAmount.compareTo(targetAmount) >= 0;
            case ">":
                return orderAmount.compareTo(targetAmount) > 0;
            case "<=":
                return orderAmount.compareTo(targetAmount) <= 0;
            case "<":
                return orderAmount.compareTo(targetAmount) < 0;
            case "==":
                return orderAmount.compareTo(targetAmount) == 0;
            default:
                return false;
        }
    }
    
    @Override
    public String toString() {
        return "订单金额 " + operator + " " + targetAmount;
    }
}

/**
 * 终结符表达式 - 商品数量条件
 */
public class ItemCountExpression implements Expression {
    private int targetCount;
    private String operator;
    
    public ItemCountExpression(String operator, int targetCount) {
        this.operator = operator;
        this.targetCount = targetCount;
    }
    
    @Override
    public boolean interpret(OrderContext context) {
        int orderItemCount = context.getItemCount();
        
        switch (operator) {
            case ">=":
                return orderItemCount >= targetCount;
            case ">":
                return orderItemCount > targetCount;
            case "<=":
                return orderItemCount <= targetCount;
            case "<":
                return orderItemCount < targetCount;
            case "==":
                return orderItemCount == targetCount;
            default:
                return false;
        }
    }
    
    @Override
    public String toString() {
        return "商品数量 " + operator + " " + targetCount;
    }
}

/**
 * 终结符表达式 - 用户等级条件
 */
public class UserLevelExpression implements Expression {
    private String requiredLevel;
    
    public UserLevelExpression(String requiredLevel) {
        this.requiredLevel = requiredLevel;
    }
    
    @Override
    public boolean interpret(OrderContext context) {
        return requiredLevel.equals(context.getUserLevel());
    }
    
    @Override
    public String toString() {
        return "用户等级 == " + requiredLevel;
    }
}

/**
 * 终结符表达式 - 商品类别条件
 */
public class CategoryExpression implements Expression {
    private String requiredCategory;
    
    public CategoryExpression(String requiredCategory) {
        this.requiredCategory = requiredCategory;
    }
    
    @Override
    public boolean interpret(OrderContext context) {
        return requiredCategory.equals(context.getProductCategory());
    }
    
    @Override
    public String toString() {
        return "商品类别 == " + requiredCategory;
    }
}

/**
 * 终结符表达式 - 首单条件
 */
public class FirstOrderExpression implements Expression {
    private boolean expectedValue;
    
    public FirstOrderExpression(boolean expectedValue) {
        this.expectedValue = expectedValue;
    }
    
    @Override
    public boolean interpret(OrderContext context) {
        return context.isFirstOrder() == expectedValue;
    }
    
    @Override
    public String toString() {
        return "是否首单 == " + expectedValue;
    }
}

/**
 * 终结符表达式 - 时间条件
 */
public class TimeRangeExpression implements Expression {
    private LocalTime startTime;
    private LocalTime endTime;
    
    public TimeRangeExpression(LocalTime startTime, LocalTime endTime) {
        this.startTime = startTime;
        this.endTime = endTime;
    }
    
    @Override
    public boolean interpret(OrderContext context) {
        LocalTime orderTime = context.getOrderTime().toLocalTime();
        return !orderTime.isBefore(startTime) && !orderTime.isAfter(endTime);
    }
    
    @Override
    public String toString() {
        return "下单时间在 " + startTime + " 到 " + endTime + " 之间";
    }
}

/**
 * 非终结符表达式 - AND操作
 */
public class AndExpression implements Expression {
    private Expression leftExpression;
    private Expression rightExpression;
    
    public AndExpression(Expression left, Expression right) {
        this.leftExpression = left;
        this.rightExpression = right;
    }
    
    @Override
    public boolean interpret(OrderContext context) {
        return leftExpression.interpret(context) && rightExpression.interpret(context);
    }
    
    @Override
    public String toString() {
        return "(" + leftExpression.toString() + " AND " + rightExpression.toString() + ")";
    }
}

/**
 * 非终结符表达式 - OR操作
 */
public class OrExpression implements Expression {
    private Expression leftExpression;
    private Expression rightExpression;
    
    public OrExpression(Expression left, Expression right) {
        this.leftExpression = left;
        this.rightExpression = right;
    }
    
    @Override
    public boolean interpret(OrderContext context) {
        return leftExpression.interpret(context) || rightExpression.interpret(context);
    }
    
    @Override
    public String toString() {
        return "(" + leftExpression.toString() + " OR " + rightExpression.toString() + ")";
    }
}

/**
 * 非终结符表达式 - NOT操作
 */
public class NotExpression implements Expression {
    private Expression expression;
    
    public NotExpression(Expression expression) {
        this.expression = expression;
    }
    
    @Override
    public boolean interpret(OrderContext context) {
        return !expression.interpret(context);
    }
    
    @Override
    public String toString() {
        return "NOT(" + expression.toString() + ")";
    }
}

/**
 * 优惠规则
 */
public class DiscountRule {
    private String ruleId;
    private String ruleName;
    private Expression condition;
    private BigDecimal discountAmount;
    private BigDecimal discountPercentage;
    private String discountType; // "AMOUNT" 或 "PERCENTAGE"
    private boolean isActive;
    
    public DiscountRule(String ruleId, String ruleName, Expression condition, 
                       String discountType, BigDecimal discountValue) {
        this.ruleId = ruleId;
        this.ruleName = ruleName;
        this.condition = condition;
        this.discountType = discountType;
        this.isActive = true;
        
        if ("AMOUNT".equals(discountType)) {
            this.discountAmount = discountValue;
        } else {
            this.discountPercentage = discountValue;
        }
    }
    
    public boolean isApplicable(OrderContext context) {
        return isActive && condition.interpret(context);
    }
    
    public BigDecimal calculateDiscount(BigDecimal orderAmount) {
        if ("AMOUNT".equals(discountType)) {
            return discountAmount;
        } else {
            return orderAmount.multiply(discountPercentage).divide(new BigDecimal("100"));
        }
    }
    
    // Getters
    public String getRuleId() { return ruleId; }
    public String getRuleName() { return ruleName; }
    public Expression getCondition() { return condition; }
    public boolean isActive() { return isActive; }
    public void setActive(boolean active) { this.isActive = active; }
    
    @Override
    public String toString() {
        return ruleName + ": " + condition.toString();
    }
}

/**
 * 优惠规则引擎
 */
public class DiscountRuleEngine {
    private List<DiscountRule> rules;
    
    public DiscountRuleEngine() {
        this.rules = new ArrayList<>();
        initializeDefaultRules();
    }
    
    public void addRule(DiscountRule rule) {
        rules.add(rule);
    }
    
    public List<DiscountRule> getApplicableRules(OrderContext context) {
        return rules.stream()
            .filter(rule -> rule.isApplicable(context))
            .collect(Collectors.toList());
    }
    
    public BigDecimal calculateTotalDiscount(OrderContext context) {
        BigDecimal totalDiscount = BigDecimal.ZERO;
        List<DiscountRule> applicableRules = getApplicableRules(context);
        
        System.out.println("适用的优惠规则:");
        for (DiscountRule rule : applicableRules) {
            BigDecimal discount = rule.calculateDiscount(context.getTotalAmount());
            totalDiscount = totalDiscount.add(discount);
            System.out.println("- " + rule.getRuleName() + ": -¥" + discount);
        }
        
        return totalDiscount;
    }
    
    private void initializeDefaultRules() {
        // 规则1: 满200减20
        Expression rule1Condition = new AmountExpression(">=", new BigDecimal("200"));
        DiscountRule rule1 = new DiscountRule("R001", "满200减20", rule1Condition, 
            "AMOUNT", new BigDecimal("20"));
        addRule(rule1);
        
        // 规则2: VIP用户9折
        Expression rule2Condition = new UserLevelExpression("VIP");
        DiscountRule rule2 = new DiscountRule("R002", "VIP用户9折", rule2Condition,
            "PERCENTAGE", new BigDecimal("10"));
        addRule(rule2);
        
        // 规则3: 首单用户且金额大于100减30
        Expression rule3Condition = new AndExpression(
            new FirstOrderExpression(true),
            new AmountExpression(">", new BigDecimal("100"))
        );
        DiscountRule rule3 = new DiscountRule("R003", "首单优惠", rule3Condition,
            "AMOUNT", new BigDecimal("30"));
        addRule(rule3);
        
        // 规则4: 电子产品类别或者金额大于500
        Expression rule4Condition = new OrExpression(
            new CategoryExpression("电子产品"),
            new AmountExpression(">=", new BigDecimal("500"))
        );
        DiscountRule rule4 = new DiscountRule("R004", "电子产品特惠", rule4Condition,
            "PERCENTAGE", new BigDecimal("5"));
        addRule(rule4);
        
        // 规则5: 深夜购物优惠 (23:00-06:00)
        Expression rule5Condition = new OrExpression(
            new TimeRangeExpression(LocalTime.of(23, 0), LocalTime.of(23, 59)),
            new TimeRangeExpression(LocalTime.of(0, 0), LocalTime.of(6, 0))
        );
        DiscountRule rule5 = new DiscountRule("R005", "深夜购物优惠", rule5Condition,
            "AMOUNT", new BigDecimal("15"));
        addRule(rule5);
        
        // 规则6: 复杂组合规则 - (VIP用户 或 首单) 且 (金额>=300 且商品数量>=3)
        Expression rule6Condition = new AndExpression(
            new OrExpression(
                new UserLevelExpression("VIP"),
                new FirstOrderExpression(true)
            ),
            new AndExpression(
                new AmountExpression(">=", new BigDecimal("300")),
                new ItemCountExpression(">=", 3)
            )
        );
        DiscountRule rule6 = new DiscountRule("R006", "超级组合优惠", rule6Condition,
            "AMOUNT", new BigDecimal("50"));
        addRule(rule6);
    }
    
    public void printAllRules() {
        System.out.println("=== 所有优惠规则 ===");
        for (DiscountRule rule : rules) {
            System.out.println(rule.getRuleId() + ": " + rule);
        }
    }
}

/**
 * 规则表达式解析器 - 可以解析字符串表达式
 */
public class ExpressionParser {
    
    public static Expression parse(String expressionString) {
        // 简化的解析器实现，实际项目中可能需要更复杂的词法和语法解析
        expressionString = expressionString.trim();
        
        // 处理AND操作
        if (expressionString.contains(" AND ")) {
            String[] parts = expressionString.split(" AND ", 2);
            return new AndExpression(parse(parts[0]), parse(parts[1]));
        }
        
        // 处理OR操作
        if (expressionString.contains(" OR ")) {
            String[] parts = expressionString.split(" OR ", 2);
            return new OrExpression(parse(parts[0]), parse(parts[1]));
        }
        
        // 处理NOT操作
        if (expressionString.startsWith("NOT(") && expressionString.endsWith(")")) {
            String inner = expressionString.substring(4, expressionString.length() - 1);
            return new NotExpression(parse(inner));
        }
        
        // 处理括号
        if (expressionString.startsWith("(") && expressionString.endsWith(")")) {
            return parse(expressionString.substring(1, expressionString.length() - 1));
        }
        
        // 解析基础表达式
        return parseBasicExpression(expressionString);
    }
    
    private static Expression parseBasicExpression(String expr) {
        expr = expr.trim();
        
        if (expr.startsWith("amount")) {
            // amount >= 200
            String[] parts = expr.split(" ");
            String operator = parts[1];
            BigDecimal amount = new BigDecimal(parts[2]);
            return new AmountExpression(operator, amount);
        }
        
        if (expr.startsWith("itemCount")) {
            // itemCount >= 3
            String[] parts = expr.split(" ");
            String operator = parts[1];
            int count = Integer.parseInt(parts[2]);
            return new ItemCountExpression(operator, count);
        }
        
        if (expr.startsWith("userLevel")) {
            // userLevel == VIP
            String[] parts = expr.split(" == ");
            return new UserLevelExpression(parts[1]);
        }
        
        if (expr.startsWith("category")) {
            // category == 电子产品
            String[] parts = expr.split(" == ");
            return new CategoryExpression(parts[1]);
        }
        
        if (expr.startsWith("firstOrder")) {
            // firstOrder == true
            String[] parts = expr.split(" == ");
            return new FirstOrderExpression(Boolean.parseBoolean(parts[1]));
        }
        
        throw new IllegalArgumentException("无法解析表达式: " + expr);
    }
}

/**
 * 解释器模式使用示例
 */
public class InterpreterPatternExample {
    
    public void demonstrateInterpreter() {
        System.out.println("=== 解释器模式演示 ===\n");
        
        DiscountRuleEngine engine = new DiscountRuleEngine();
        
        // 显示所有规则
        engine.printAllRules();
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // 测试不同的订单场景
        testOrderScenarios(engine);
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // 动态解析表达式
        demonstrateExpressionParsing();
    }
    
    private void testOrderScenarios(DiscountRuleEngine engine) {
        // 场景1: 普通用户，首单，金额250
        System.out.println("场景1: 普通用户首单购买，金额250元");
        OrderContext context1 = new OrderContext(
            new BigDecimal("250"), 2, "NORMAL", "电子产品", true);
        
        BigDecimal discount1 = engine.calculateTotalDiscount(context1);
        System.out.println("总优惠金额: ¥" + discount1);
        System.out.println("最终支付: ¥" + context1.getTotalAmount().subtract(discount1));
        
        System.out.println("\n" + "-".repeat(30) + "\n");
        
        // 场景2: VIP用户，非首单，金额500
        System.out.println("场景2: VIP用户购买，金额500元");
        OrderContext context2 = new OrderContext(
            new BigDecimal("500"), 1, "VIP", "服装", false);
        
        BigDecimal discount2 = engine.calculateTotalDiscount(context2);
        System.out.println("总优惠金额: ¥" + discount2);
        System.out.println("最终支付: ¥" + context2.getTotalAmount().subtract(discount2));
        
        System.out.println("\n" + "-".repeat(30) + "\n");
        
        // 场景3: VIP用户，首单，金额400，3件商品
        System.out.println("场景3: VIP用户首单，金额400元，3件商品");
        OrderContext context3 = new OrderContext(
            new BigDecimal("400"), 3, "VIP", "电子产品", true);
        
        BigDecimal discount3 = engine.calculateTotalDiscount(context3);
        System.out.println("总优惠金额: ¥" + discount3);
        System.out.println("最终支付: ¥" + context3.getTotalAmount().subtract(discount3));
    }
    
    private void demonstrateExpressionParsing() {
        System.out.println("动态表达式解析演示:");
        
        // 解析复杂表达式
        String expressionStr = "(userLevel == VIP OR firstOrder == true) AND amount >= 200";
        Expression dynamicExpr = ExpressionParser.parse(expressionStr);
        
        System.out.println("解析的表达式: " + dynamicExpr.toString());
        
        // 测试表达式
        OrderContext testContext = new OrderContext(
            new BigDecimal("300"), 2, "NORMAL", "图书", true);
        
        boolean result = dynamicExpr.interpret(testContext);
        System.out.println("表达式结果: " + result);
    }
}

### 16. 迭代器模式 (Iterator Pattern)

**场景**: 订单列表遍历、商品目录遍历、购物车遍历

```java
/**
 * 迭代器接口
 * 参考：Java Collections框架
 */
public interface Iterator<T> {
    boolean hasNext();
    T next();
    void remove();
}

/**
 * 聚合接口
 */
public interface Aggregate<T> {
    Iterator<T> createIterator();
    int size();
    boolean isEmpty();
}

/**
 * 订单项
 */
public class OrderItem {
    private String productId;
    private String productName;
    private BigDecimal price;
    private int quantity;
    private String category;
    private Map<String, String> attributes;
    
    public OrderItem(String productId, String productName, BigDecimal price, int quantity) {
        this.productId = productId;
        this.productName = productName;
        this.price = price;
        this.quantity = quantity;
        this.attributes = new HashMap<>();
    }
    
    // Getters and Setters
    public String getProductId() { return productId; }
    public String getProductName() { return productName; }
    public BigDecimal getPrice() { return price; }
    public int getQuantity() { return quantity; }
    public String getCategory() { return category; }
    public void setCategory(String category) { this.category = category; }
    
    public void setAttribute(String key, String value) {
        attributes.put(key, value);
    }
    
    public String getAttribute(String key) {
        return attributes.get(key);
    }
    
    public BigDecimal getTotalPrice() {
        return price.multiply(BigDecimal.valueOf(quantity));
    }
    
    @Override
    public String toString() {
        return String.format("%s - %s x%d = ¥%s", 
            productId, productName, quantity, getTotalPrice());
    }
}

/**
 * 购物车 - 实现聚合接口
 */
public class ShoppingCart implements Aggregate<OrderItem> {
    private List<OrderItem> items;
    private String userId;
    private LocalDateTime createTime;
    
    public ShoppingCart(String userId) {
        this.userId = userId;
        this.items = new ArrayList<>();
        this.createTime = LocalDateTime.now();
    }
    
    public void addItem(OrderItem item) {
        // 检查是否已存在相同商品，如果存在则更新数量
        OrderItem existingItem = findItemByProductId(item.getProductId());
        if (existingItem != null) {
            int newQuantity = existingItem.getQuantity() + item.getQuantity();
            existingItem.setQuantity(newQuantity);
        } else {
            items.add(item);
        }
    }
    
    public void removeItem(String productId) {
        items.removeIf(item -> item.getProductId().equals(productId));
    }
    
    public void updateItemQuantity(String productId, int newQuantity) {
        OrderItem item = findItemByProductId(productId);
        if (item != null) {
            if (newQuantity <= 0) {
                removeItem(productId);
            } else {
                item.setQuantity(newQuantity);
            }
        }
    }
    
    private OrderItem findItemByProductId(String productId) {
        return items.stream()
            .filter(item -> item.getProductId().equals(productId))
            .findFirst()
            .orElse(null);
    }
    
    @Override
    public Iterator<OrderItem> createIterator() {
        return new ShoppingCartIterator();
    }
    
    @Override
    public int size() {
        return items.size();
    }
    
    @Override
    public boolean isEmpty() {
        return items.isEmpty();
    }
    
    public BigDecimal getTotalAmount() {
        BigDecimal total = BigDecimal.ZERO;
        Iterator<OrderItem> iterator = createIterator();
        while (iterator.hasNext()) {
            OrderItem item = iterator.next();
            total = total.add(item.getTotalPrice());
        }
        return total;
    }
    
    public String getUserId() { return userId; }
    
    /**
     * 购物车迭代器内部类
     */
    private class ShoppingCartIterator implements Iterator<OrderItem> {
        private int currentIndex = 0;
        private boolean canRemove = false;
        
        @Override
        public boolean hasNext() {
            return currentIndex < items.size();
        }
        
        @Override
        public OrderItem next() {
            if (!hasNext()) {
                throw new NoSuchElementException("没有更多商品");
            }
            
            OrderItem item = items.get(currentIndex);
            currentIndex++;
            canRemove = true;
            return item;
        }
        
        @Override
        public void remove() {
            if (!canRemove) {
                throw new IllegalStateException("无法删除，请先调用next()");
            }
            
            items.remove(currentIndex - 1);
            currentIndex--;
            canRemove = false;
        }
    }
}

/**
 * 订单列表 - 支持多种遍历方式
 */
public class OrderList implements Aggregate<Order> {
    private List<Order> orders;
    
    public OrderList() {
        this.orders = new ArrayList<>();
    }
    
    public void addOrder(Order order) {
        orders.add(order);
    }
    
    public void removeOrder(String orderId) {
        orders.removeIf(order -> order.getOrderId().equals(orderId));
    }
    
    @Override
    public Iterator<Order> createIterator() {
        return new OrderIterator();
    }
    
    public Iterator<Order> createReverseIterator() {
        return new ReverseOrderIterator();
    }
    
    public Iterator<Order> createFilteredIterator(OrderStatus status) {
        return new FilteredOrderIterator(status);
    }
    
    public Iterator<Order> createSortedIterator(Comparator<Order> comparator) {
        return new SortedOrderIterator(comparator);
    }
    
    @Override
    public int size() {
        return orders.size();
    }
    
    @Override
    public boolean isEmpty() {
        return orders.isEmpty();
    }
    
    /**
     * 正向迭代器
     */
    private class OrderIterator implements Iterator<Order> {
        private int currentIndex = 0;
        private boolean canRemove = false;
        
        @Override
        public boolean hasNext() {
            return currentIndex < orders.size();
        }
        
        @Override
        public Order next() {
            if (!hasNext()) {
                throw new NoSuchElementException();
            }
            
            Order order = orders.get(currentIndex);
            currentIndex++;
            canRemove = true;
            return order;
        }
        
        @Override
        public void remove() {
            if (!canRemove) {
                throw new IllegalStateException();
            }
            
            orders.remove(currentIndex - 1);
            currentIndex--;
            canRemove = false;
        }
    }
    
    /**
     * 反向迭代器
     */
    private class ReverseOrderIterator implements Iterator<Order> {
        private int currentIndex;
        private boolean canRemove = false;
        
        public ReverseOrderIterator() {
            this.currentIndex = orders.size() - 1;
        }
        
        @Override
        public boolean hasNext() {
            return currentIndex >= 0;
        }
        
        @Override
        public Order next() {
            if (!hasNext()) {
                throw new NoSuchElementException();
            }
            
            Order order = orders.get(currentIndex);
            currentIndex--;
            canRemove = true;
            return order;
        }
        
        @Override
        public void remove() {
            if (!canRemove) {
                throw new IllegalStateException();
            }
            
            orders.remove(currentIndex + 1);
            canRemove = false;
        }
    }
    
    /**
     * 过滤迭代器
     */
    private class FilteredOrderIterator implements Iterator<Order> {
        private OrderStatus filterStatus;
        private int currentIndex = 0;
        private Order nextOrder = null;
        private boolean canRemove = false;
        private int lastReturnedIndex = -1;
        
        public FilteredOrderIterator(OrderStatus filterStatus) {
            this.filterStatus = filterStatus;
            findNext();
        }
        
        private void findNext() {
            nextOrder = null;
            while (currentIndex < orders.size()) {
                Order order = orders.get(currentIndex);
                if (order.getStatus() == filterStatus) {
                    nextOrder = order;
                    break;
                }
                currentIndex++;
            }
        }
        
        @Override
        public boolean hasNext() {
            return nextOrder != null;
        }
        
        @Override
        public Order next() {
            if (!hasNext()) {
                throw new NoSuchElementException();
            }
            
            Order order = nextOrder;
            lastReturnedIndex = currentIndex;
            currentIndex++;
            findNext();
            canRemove = true;
            return order;
        }
        
        @Override
        public void remove() {
            if (!canRemove) {
                throw new IllegalStateException();
            }
            
            orders.remove(lastReturnedIndex);
            currentIndex = lastReturnedIndex;
            findNext();
            canRemove = false;
        }
    }
    
    /**
     * 排序迭代器
     */
    private class SortedOrderIterator implements Iterator<Order> {
        private List<Order> sortedOrders;
        private int currentIndex = 0;
        private boolean canRemove = false;
        
        public SortedOrderIterator(Comparator<Order> comparator) {
            this.sortedOrders = new ArrayList<>(orders);
            this.sortedOrders.sort(comparator);
        }
        
        @Override
        public boolean hasNext() {
            return currentIndex < sortedOrders.size();
        }
        
        @Override
        public Order next() {
            if (!hasNext()) {
                throw new NoSuchElementException();
            }
            
            Order order = sortedOrders.get(currentIndex);
            currentIndex++;
            canRemove = true;
            return order;
        }
        
        @Override
        public void remove() {
            if (!canRemove) {
                throw new IllegalStateException();
            }
            
            Order orderToRemove = sortedOrders.get(currentIndex - 1);
            orders.remove(orderToRemove);
            sortedOrders.remove(currentIndex - 1);
            currentIndex--;
            canRemove = false;
        }
    }
}

/**
 * 商品目录 - 支持分页迭代
 */
public class ProductCatalog implements Aggregate<Product> {
    private List<Product> products;
    
    public ProductCatalog() {
        this.products = new ArrayList<>();
        initializeProducts();
    }
    
    public void addProduct(Product product) {
        products.add(product);
    }
    
    @Override
    public Iterator<Product> createIterator() {
        return new ProductIterator();
    }
    
    public Iterator<Product> createPagedIterator(int pageSize) {
        return new PagedProductIterator(pageSize);
    }
    
    public Iterator<Product> createCategoryIterator(String category) {
        return new CategoryProductIterator(category);
    }
    
    @Override
    public int size() {
        return products.size();
    }
    
    @Override
    public boolean isEmpty() {
        return products.isEmpty();
    }
    
    private void initializeProducts() {
        // 初始化一些示例商品
        addProduct(new Product("P001", "iPhone 14", new BigDecimal("5999")));
        addProduct(new Product("P002", "MacBook Pro", new BigDecimal("15999")));
        addProduct(new Product("P003", "AirPods Pro", new BigDecimal("1899")));
        addProduct(new Product("P004", "iPad Air", new BigDecimal("4599")));
        addProduct(new Product("P005", "Apple Watch", new BigDecimal("2999")));
        
        // 设置商品类别
        products.get(0).setCategory("手机");
        products.get(1).setCategory("电脑");
        products.get(2).setCategory("耳机");
        products.get(3).setCategory("平板");
        products.get(4).setCategory("手表");
    }
    
    /**
     * 基础商品迭代器
     */
    private class ProductIterator implements Iterator<Product> {
        private int currentIndex = 0;
        
        @Override
        public boolean hasNext() {
            return currentIndex < products.size();
        }
        
        @Override
        public Product next() {
            if (!hasNext()) {
                throw new NoSuchElementException();
            }
            return products.get(currentIndex++);
        }
        
        @Override
        public void remove() {
            throw new UnsupportedOperationException("商品目录不支持删除操作");
        }
    }
    
    /**
     * 分页迭代器
     */
    private class PagedProductIterator implements Iterator<Product> {
        private int pageSize;
        private int currentPage = 0;
        private int currentIndexInPage = 0;
        
        public PagedProductIterator(int pageSize) {
            this.pageSize = pageSize;
        }
        
        @Override
        public boolean hasNext() {
            int globalIndex = currentPage * pageSize + currentIndexInPage;
            return globalIndex < products.size();
        }
        
        @Override
        public Product next() {
            if (!hasNext()) {
                throw new NoSuchElementException();
            }
            
            int globalIndex = currentPage * pageSize + currentIndexInPage;
            Product product = products.get(globalIndex);
            
            currentIndexInPage++;
            if (currentIndexInPage >= pageSize) {
                currentPage++;
                currentIndexInPage = 0;
                System.out.println("--- 第 " + currentPage + " 页 ---");
            }
            
            return product;
        }
        
        @Override
        public void remove() {
            throw new UnsupportedOperationException();
        }
        
        public boolean hasNextPage() {
            return (currentPage + 1) * pageSize < products.size();
        }
        
        public void nextPage() {
            if (hasNextPage()) {
                currentPage++;
                currentIndexInPage = 0;
            }
        }
    }
    
    /**
     * 类别过滤迭代器
     */
    private class CategoryProductIterator implements Iterator<Product> {
        private String category;
        private int currentIndex = 0;
        private Product nextProduct = null;
        
        public CategoryProductIterator(String category) {
            this.category = category;
            findNext();
        }
        
        private void findNext() {
            nextProduct = null;
            while (currentIndex < products.size()) {
                Product product = products.get(currentIndex);
                if (category.equals(product.getCategory())) {
                    nextProduct = product;
                    break;
                }
                currentIndex++;
            }
        }
        
        @Override
        public boolean hasNext() {
            return nextProduct != null;
        }
        
        @Override
        public Product next() {
            if (!hasNext()) {
                throw new NoSuchElementException();
            }
            
            Product product = nextProduct;
            currentIndex++;
            findNext();
            return product;
        }
        
        @Override
        public void remove() {
            throw new UnsupportedOperationException();
        }
    }
}

/**
 * 迭代器模式使用示例
 */
public class IteratorPatternExample {
    
    public void demonstrateIterators() {
        System.out.println("=== 迭代器模式演示 ===\n");
        
        // 1. 购物车迭代
        demonstrateShoppingCart();
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // 2. 订单列表迭代
        demonstrateOrderList();
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // 3. 商品目录迭代
        demonstrateProductCatalog();
    }
    
    private void demonstrateShoppingCart() {
        System.out.println("1. 购物车迭代演示:");
        
        ShoppingCart cart = new ShoppingCart("U001");
        cart.addItem(new OrderItem("P001", "iPhone 14", new BigDecimal("5999"), 1));
        cart.addItem(new OrderItem("P002", "AirPods Pro", new BigDecimal("1899"), 2));
        cart.addItem(new OrderItem("P003", "MacBook Pro", new BigDecimal("15999"), 1));
        
        System.out.println("购物车商品列表:");
        Iterator<OrderItem> cartIterator = cart.createIterator();
        while (cartIterator.hasNext()) {
            OrderItem item = cartIterator.next();
            System.out.println("- " + item);
        }
        
        System.out.println("购物车总金额: ¥" + cart.getTotalAmount());
        
        // 删除第二个商品
        System.out.println("\n删除AirPods Pro后的购物车:");
        cartIterator = cart.createIterator();
        while (cartIterator.hasNext()) {
            OrderItem item = cartIterator.next();
            if ("P002".equals(item.getProductId())) {
                cartIterator.remove();
                break;
            }
        }
        
        cartIterator = cart.createIterator();
        while (cartIterator.hasNext()) {
            System.out.println("- " + cartIterator.next());
        }
    }
    
    private void demonstrateOrderList() {
        System.out.println("2. 订单列表迭代演示:");
        
        OrderList orderList = createSampleOrders();
        
        // 正向遍历
        System.out.println("正向遍历所有订单:");
        Iterator<Order> iterator = orderList.createIterator();
        while (iterator.hasNext()) {
            Order order = iterator.next();
            System.out.println("- " + order.getOrderId() + " (" + order.getStatus() + ")");
        }
        
        // 反向遍历
        System.out.println("\n反向遍历所有订单:");
        Iterator<Order> reverseIterator = orderList.createReverseIterator();
        while (reverseIterator.hasNext()) {
            Order order = reverseIterator.next();
            System.out.println("- " + order.getOrderId() + " (" + order.getStatus() + ")");
        }
        
        // 过滤遍历
        System.out.println("\n只遍历已支付的订单:");
        Iterator<Order> paidIterator = orderList.createFilteredIterator(OrderStatus.PAID);
        while (paidIterator.hasNext()) {
            Order order = paidIterator.next();
            System.out.println("- " + order.getOrderId() + " (" + order.getStatus() + ")");
        }
        
        // 排序遍历
        System.out.println("\n按创建时间排序遍历:");
        Iterator<Order> sortedIterator = orderList.createSortedIterator(
            Comparator.comparing(Order::getCreateTime));
        while (sortedIterator.hasNext()) {
            Order order = sortedIterator.next();
            System.out.println("- " + order.getOrderId() + " (" + 
                order.getCreateTime().format(DateTimeFormatter.ofPattern("MM-dd HH:mm")) + ")");
        }
    }
    
    private void demonstrateProductCatalog() {
        System.out.println("3. 商品目录迭代演示:");
        
        ProductCatalog catalog = new ProductCatalog();
        
        // 基础遍历
        System.out.println("所有商品:");
        Iterator<Product> productIterator = catalog.createIterator();
        while (productIterator.hasNext()) {
            Product product = productIterator.next();
            System.out.println("- " + product.getName() + " (" + product.getCategory() + ") - ¥" + product.getPrice());
        }
        
        // 分页遍历
        System.out.println("\n分页遍历 (每页2个商品):");
        Iterator<Product> pagedIterator = catalog.createPagedIterator(2);
        int count = 0;
        while (pagedIterator.hasNext()) {
            if (count % 2 == 0) {
                System.out.println("--- 第 " + (count / 2 + 1) + " 页 ---");
            }
            Product product = pagedIterator.next();
            System.out.println("- " + product.getName() + " - ¥" + product.getPrice());
            count++;
        }
        
        // 类别过滤遍历
        System.out.println("\n只显示手机类商品:");
        Iterator<Product> phoneIterator = catalog.createCategoryIterator("手机");
        while (phoneIterator.hasNext()) {
            Product product = phoneIterator.next();
            System.out.println("- " + product.getName() + " - ¥" + product.getPrice());
        }
    }
    
    private OrderList createSampleOrders() {
        OrderList orderList = new OrderList();
        
        Order order1 = new Order();
        order1.setOrderId("ORD001");
        order1.setUserId("U001");
        order1.setStatus(OrderStatus.PAID);
        order1.setCreateTime(LocalDateTime.now().minusHours(2));
        
        Order order2 = new Order();
        order2.setOrderId("ORD002");
        order2.setUserId("U002");
        order2.setStatus(OrderStatus.PENDING);
        order2.setCreateTime(LocalDateTime.now().minusHours(1));
        
        Order order3 = new Order();
        order3.setOrderId("ORD003");
        order3.setUserId("U003");
        order3.setStatus(OrderStatus.PAID);
        order3.setCreateTime(LocalDateTime.now().minusMinutes(30));
        
        Order order4 = new Order();
        order4.setOrderId("ORD004");
        order4.setUserId("U004");
        order4.setStatus(OrderStatus.SHIPPED);
        order4.setCreateTime(LocalDateTime.now().minusMinutes(10));
        
        orderList.addOrder(order1);
        orderList.addOrder(order2);
        orderList.addOrder(order3);
        orderList.addOrder(order4);
        
        return orderList;
    }
}

### 17. 中介者模式 (Mediator Pattern)

**场景**: 订单流程协调、多服务协调、UI组件交互

```java
/**
 * 中介者接口
 * 参考：Spring Integration消息中介
 */
public interface OrderMediator {
    void notify(Component sender, String event, Object data);
    void registerComponent(Component component);
}

/**
 * 组件抽象类
 */
public abstract class Component {
    protected OrderMediator mediator;
    protected String componentId;
    
    public Component(String componentId, OrderMediator mediator) {
        this.componentId = componentId;
        this.mediator = mediator;
        if (mediator != null) {
            mediator.registerComponent(this);
        }
    }
    
    public abstract void handleEvent(String event, Object data);
    
    public String getComponentId() {
        return componentId;
    }
}

/**
 * 具体中介者 - 订单处理协调器
 */
public class OrderProcessingMediator implements OrderMediator {
    private Map<String, Component> components;
    private Map<String, List<String>> eventSubscriptions;
    
    public OrderProcessingMediator() {
        this.components = new HashMap<>();
        this.eventSubscriptions = new HashMap<>();
        initializeEventSubscriptions();
    }
    
    @Override
    public void registerComponent(Component component) {
        components.put(component.getComponentId(), component);
        System.out.println("注册组件: " + component.getComponentId());
    }
    
    @Override
    public void notify(Component sender, String event, Object data) {
        System.out.println(String.format("[中介者] 收到事件: %s 来自: %s", 
            event, sender.getComponentId()));
        
        // 获取订阅该事件的组件
        List<String> subscribers = eventSubscriptions.get(event);
        if (subscribers != null) {
            for (String componentId : subscribers) {
                Component component = components.get(componentId);
                if (component != null && !component.equals(sender)) {
                    System.out.println(String.format("[中介者] 转发事件 %s 到: %s", 
                        event, componentId));
                    component.handleEvent(event, data);
                }
            }
        }
        
        // 处理特殊的业务逻辑协调
        handleBusinessLogic(sender, event, data);
    }
    
    private void initializeEventSubscriptions() {
        // 库存服务订阅的事件
        eventSubscriptions.put("ORDER_CREATED", Arrays.asList("INVENTORY_SERVICE", "NOTIFICATION_SERVICE"));
        eventSubscriptions.put("PAYMENT_SUCCESS", Arrays.asList("INVENTORY_SERVICE", "LOGISTICS_SERVICE", "NOTIFICATION_SERVICE"));
        eventSubscriptions.put("PAYMENT_FAILED", Arrays.asList("INVENTORY_SERVICE", "NOTIFICATION_SERVICE"));
        eventSubscriptions.put("INVENTORY_LOCKED", Arrays.asList("PAYMENT_SERVICE"));
        eventSubscriptions.put("INVENTORY_INSUFFICIENT", Arrays.asList("NOTIFICATION_SERVICE"));
        eventSubscriptions.put("ORDER_SHIPPED", Arrays.asList("NOTIFICATION_SERVICE"));
        eventSubscriptions.put("ORDER_DELIVERED", Arrays.asList("NOTIFICATION_SERVICE"));
    }
    
    private void handleBusinessLogic(Component sender, String event, Object data) {
        // 处理复杂的业务协调逻辑
        switch (event) {
            case "ORDER_CREATED":
                handleOrderCreated(sender, data);
                break;
            case "PAYMENT_SUCCESS":
                handlePaymentSuccess(sender, data);
                break;
            case "INVENTORY_INSUFFICIENT":
                handleInventoryInsufficient(sender, data);
                break;
        }
    }
    
    private void handleOrderCreated(Component sender, Object data) {
        System.out.println("[协调] 开始订单创建流程协调");
        // 可以在这里添加额外的协调逻辑
    }
    
    private void handlePaymentSuccess(Component sender, Object data) {
        System.out.println("[协调] 支付成功，开始后续流程协调");
        // 启动物流和通知流程
    }
    
    private void handleInventoryInsufficient(Component sender, Object data) {
        System.out.println("[协调] 库存不足，终止订单流程");
        // 通知相关服务取消订单流程
        notify(sender, "ORDER_CANCELLED", data);
    }
}

/**
 * 库存服务组件
 */
public class InventoryServiceComponent extends Component {
    private Map<String, Integer> inventory;
    
    public InventoryServiceComponent(OrderMediator mediator) {
        super("INVENTORY_SERVICE", mediator);
        this.inventory = new HashMap<>();
        initializeInventory();
    }
    
    @Override
    public void handleEvent(String event, Object data) {
        switch (event) {
            case "ORDER_CREATED":
                handleOrderCreated((OrderData) data);
                break;
            case "PAYMENT_SUCCESS":
                handlePaymentSuccess((OrderData) data);
                break;
            case "PAYMENT_FAILED":
                handlePaymentFailed((OrderData) data);
                break;
        }
    }
    
    private void handleOrderCreated(OrderData orderData) {
        System.out.println("[库存服务] 处理订单创建，检查库存");
        
        boolean sufficient = true;
        for (OrderItem item : orderData.getItems()) {
            int available = inventory.getOrDefault(item.getProductId(), 0);
            if (available < item.getQuantity()) {
                sufficient = false;
                break;
            }
        }
        
        if (sufficient) {
            // 锁定库存
            lockInventory(orderData.getItems());
            mediator.notify(this, "INVENTORY_LOCKED", orderData);
        } else {
            mediator.notify(this, "INVENTORY_INSUFFICIENT", orderData);
        }
    }
    
    private void handlePaymentSuccess(OrderData orderData) {
        System.out.println("[库存服务] 支付成功，扣减库存");
        deductInventory(orderData.getItems());
    }
    
    private void handlePaymentFailed(OrderData orderData) {
        System.out.println("[库存服务] 支付失败，释放库存");
        releaseInventory(orderData.getItems());
    }
    
    private void lockInventory(List<OrderItem> items) {
        System.out.println("  锁定库存:");
        for (OrderItem item : items) {
            System.out.println("    " + item.getProductId() + ": " + item.getQuantity() + "件");
        }
    }
    
    private void deductInventory(List<OrderItem> items) {
        System.out.println("  扣减库存:");
        for (OrderItem item : items) {
            int current = inventory.get(item.getProductId());
            inventory.put(item.getProductId(), current - item.getQuantity());
            System.out.println("    " + item.getProductId() + ": " + 
                item.getQuantity() + "件，剩余: " + inventory.get(item.getProductId()));
        }
    }
    
    private void releaseInventory(List<OrderItem> items) {
        System.out.println("  释放库存:");
        for (OrderItem item : items) {
            System.out.println("    " + item.getProductId() + ": " + item.getQuantity() + "件");
        }
    }
    
    private void initializeInventory() {
        inventory.put("P001", 100);
        inventory.put("P002", 50);
        inventory.put("P003", 200);
    }
}

/**
 * 支付服务组件
 */
public class PaymentServiceComponent extends Component {
    
    public PaymentServiceComponent(OrderMediator mediator) {
        super("PAYMENT_SERVICE", mediator);
    }
    
    @Override
    public void handleEvent(String event, Object data) {
        switch (event) {
            case "INVENTORY_LOCKED":
                handleInventoryLocked((OrderData) data);
                break;
        }
    }
    
    private void handleInventoryLocked(OrderData orderData) {
        System.out.println("[支付服务] 库存已锁定，开始处理支付");
        
        // 模拟支付处理
        boolean paymentSuccess = processPayment(orderData);
        
        if (paymentSuccess) {
            mediator.notify(this, "PAYMENT_SUCCESS", orderData);
        } else {
            mediator.notify(this, "PAYMENT_FAILED", orderData);
        }
    }
    
    private boolean processPayment(OrderData orderData) {
        System.out.println("  处理支付，金额: ¥" + orderData.getTotalAmount());
        
        // 模拟支付成功率90%
        boolean success = Math.random() > 0.1;
        System.out.println("  支付结果: " + (success ? "成功" : "失败"));
        
        return success;
    }
}

/**
 * 物流服务组件
 */
public class LogisticsServiceComponent extends Component {
    
    public LogisticsServiceComponent(OrderMediator mediator) {
        super("LOGISTICS_SERVICE", mediator);
    }
    
    @Override
    public void handleEvent(String event, Object data) {
        switch (event) {
            case "PAYMENT_SUCCESS":
                handlePaymentSuccess((OrderData) data);
                break;
        }
    }
    
    private void handlePaymentSuccess(OrderData orderData) {
        System.out.println("[物流服务] 支付成功，创建物流订单");
        
        String trackingNumber = createShipment(orderData);
        orderData.setTrackingNumber(trackingNumber);
        
        // 模拟发货过程
        simulateShipping(orderData);
    }
    
    private String createShipment(OrderData orderData) {
        String trackingNumber = "TN" + System.currentTimeMillis();
        System.out.println("  创建物流订单，物流单号: " + trackingNumber);
        return trackingNumber;
    }
    
    private void simulateShipping(OrderData orderData) {
        // 模拟异步发货过程
        new Thread(() -> {
            try {
                Thread.sleep(2000); // 模拟处理时间
                
                System.out.println("[物流服务] 商品已发货");
                mediator.notify(this, "ORDER_SHIPPED", orderData);
                
                Thread.sleep(3000); // 模拟运输时间
                
                System.out.println("[物流服务] 商品已送达");
                mediator.notify(this, "ORDER_DELIVERED", orderData);
                
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }).start();
    }
}

/**
 * 通知服务组件
 */
public class NotificationServiceComponent extends Component {
    
    public NotificationServiceComponent(OrderMediator mediator) {
        super("NOTIFICATION_SERVICE", mediator);
    }
    
    @Override
    public void handleEvent(String event, Object data) {
        OrderData orderData = (OrderData) data;
        
        switch (event) {
            case "ORDER_CREATED":
                sendOrderCreatedNotification(orderData);
                break;
            case "PAYMENT_SUCCESS":
                sendPaymentSuccessNotification(orderData);
                break;
            case "PAYMENT_FAILED":
                sendPaymentFailedNotification(orderData);
                break;
            case "INVENTORY_INSUFFICIENT":
                sendInventoryInsufficientNotification(orderData);
                break;
            case "ORDER_SHIPPED":
                sendOrderShippedNotification(orderData);
                break;
            case "ORDER_DELIVERED":
                sendOrderDeliveredNotification(orderData);
                break;
            case "ORDER_CANCELLED":
                sendOrderCancelledNotification(orderData);
                break;
        }
    }
    
    private void sendOrderCreatedNotification(OrderData orderData) {
        System.out.println("[通知服务] 发送订单创建通知");
        System.out.println("  短信: 您的订单 " + orderData.getOrderId() + " 已创建");
    }
    
    private void sendPaymentSuccessNotification(OrderData orderData) {
        System.out.println("[通知服务] 发送支付成功通知");
        System.out.println("  邮件: 订单 " + orderData.getOrderId() + " 支付成功，金额 ¥" + orderData.getTotalAmount());
    }
    
    private void sendPaymentFailedNotification(OrderData orderData) {
        System.out.println("[通知服务] 发送支付失败通知");
        System.out.println("  短信: 订单 " + orderData.getOrderId() + " 支付失败，请重新支付");
    }
    
    private void sendInventoryInsufficientNotification(OrderData orderData) {
        System.out.println("[通知服务] 发送库存不足通知");
        System.out.println("  APP推送: 抱歉，商品库存不足，订单已取消");
    }
    
    private void sendOrderShippedNotification(OrderData orderData) {
        System.out.println("[通知服务] 发送发货通知");
        System.out.println("  短信: 您的订单 " + orderData.getOrderId() + 
            " 已发货，物流单号: " + orderData.getTrackingNumber());
    }
    
    private void sendOrderDeliveredNotification(OrderData orderData) {
        System.out.println("[通知服务] 发送送达通知");
        System.out.println("  APP推送: 您的订单 " + orderData.getOrderId() + " 已送达，请确认收货");
    }
    
    private void sendOrderCancelledNotification(OrderData orderData) {
        System.out.println("[通知服务] 发送订单取消通知");
        System.out.println("  邮件: 订单 " + orderData.getOrderId() + " 已取消");
    }
}

/**
 * 订单数据传输对象
 */
public class OrderData {
    private String orderId;
    private String userId;
    private List<OrderItem> items;
    private BigDecimal totalAmount;
    private String trackingNumber;
    
    public OrderData(String orderId, String userId, List<OrderItem> items) {
        this.orderId = orderId;
        this.userId = userId;
        this.items = items;
        this.totalAmount = calculateTotalAmount();
    }
    
    private BigDecimal calculateTotalAmount() {
        return items.stream()
            .map(item -> item.getPrice().multiply(BigDecimal.valueOf(item.getQuantity())))
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
    
    // Getters and Setters
    public String getOrderId() { return orderId; }
    public String getUserId() { return userId; }
    public List<OrderItem> getItems() { return items; }
    public BigDecimal getTotalAmount() { return totalAmount; }
    public String getTrackingNumber() { return trackingNumber; }
    public void setTrackingNumber(String trackingNumber) { this.trackingNumber = trackingNumber; }
}

/**
 * 订单系统 - 客户端
 */
public class OrderSystem {
    private OrderMediator mediator;
    private InventoryServiceComponent inventoryService;
    private PaymentServiceComponent paymentService;
    private LogisticsServiceComponent logisticsService;
    private NotificationServiceComponent notificationService;
    
    public OrderSystem() {
        // 创建中介者
        this.mediator = new OrderProcessingMediator();
        
        // 创建各个服务组件
        this.inventoryService = new InventoryServiceComponent(mediator);
        this.paymentService = new PaymentServiceComponent(mediator);
        this.logisticsService = new LogisticsServiceComponent(mediator);
        this.notificationService = new NotificationServiceComponent(mediator);
    }
    
    public void createOrder(OrderData orderData) {
        System.out.println("=== 开始创建订单: " + orderData.getOrderId() + " ===");
        
        // 通过中介者通知所有相关组件
        mediator.notify(new Component("ORDER_SYSTEM", mediator) {
            @Override
            public void handleEvent(String event, Object data) {
                // 订单系统本身不需要处理事件
            }
        }, "ORDER_CREATED", orderData);
    }
}

/**
 * 中介者模式使用示例
 */
public class MediatorPatternExample {
    
    public void demonstrateMediator() {
        System.out.println("=== 中介者模式演示 ===\n");
        
        OrderSystem orderSystem = new OrderSystem();
        
        // 场景1: 正常订单处理流程
        System.out.println("场景1: 正常订单处理");
        System.out.println("-".repeat(40));
        
        List<OrderItem> items1 = Arrays.asList(
            new OrderItem("P001", "iPhone 14", new BigDecimal("5999"), 1),
            new OrderItem("P002", "AirPods Pro", new BigDecimal("1899"), 1)
        );
        
        OrderData order1 = new OrderData("ORD001", "U001", items1);
        orderSystem.createOrder(order1);
        
        // 等待异步物流处理完成
        try {
            Thread.sleep(6000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // 场景2: 库存不足的订单
        System.out.println("场景2: 库存不足订单处理");
        System.out.println("-".repeat(40));
        
        List<OrderItem> items2 = Arrays.asList(
            new OrderItem("P001", "iPhone 14", new BigDecimal("5999"), 150) // 库存不足
        );
        
        OrderData order2 = new OrderData("ORD002", "U002", items2);
        orderSystem.createOrder(order2);
        
        System.out.println("\n订单处理演示完成");
    }
}

### 18. 备忘录模式 (Memento Pattern)

**场景**: 订单状态回滚、购物车历史、用户操作撤销

```java
/**
 * 备忘录接口
 * 参考：Git版本控制系统
 */
public interface Memento {
    String getDescription();
    LocalDateTime getTimestamp();
}

/**
 * 订单备忘录
 */
public class OrderMemento implements Memento {
    private String orderId;
    private OrderStatus status;
    private BigDecimal amount;
    private String paymentId;
    private String trackingNumber;
    private List<OrderItem> items;
    private Map<String, Object> properties;
    private LocalDateTime timestamp;
    private String description;
    
    public OrderMemento(Order order, String description) {
        this.orderId = order.getOrderId();
        this.status = order.getStatus();
        this.amount = order.getFinalAmount();
        this.paymentId = order.getPaymentId();
        this.trackingNumber = order.getTrackingNumber();
        this.items = new ArrayList<>(order.getItems());
        this.properties = new HashMap<>(order.getProperties());
        this.timestamp = LocalDateTime.now();
        this.description = description;
    }
    
    // Package-private getter methods for Order class
    String getOrderId() { return orderId; }
    OrderStatus getStatus() { return status; }
    BigDecimal getAmount() { return amount; }
    String getPaymentId() { return paymentId; }
    String getTrackingNumber() { return trackingNumber; }
    List<OrderItem> getItems() { return new ArrayList<>(items); }
    Map<String, Object> getProperties() { return new HashMap<>(properties); }
    
    @Override
    public String getDescription() { return description; }
    
    @Override
    public LocalDateTime getTimestamp() { return timestamp; }
    
    @Override
    public String toString() {
        return String.format("%s - %s (%s)", 
            timestamp.format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")),
            description, status);
    }
}

/**
 * 发起人 - 订单类（增强版）
 */
public class Order {
    private String orderId;
    private String userId;
    private OrderStatus status;
    private OrderStatus previousStatus;
    private List<OrderItem> items;
    private BigDecimal originalAmount;
    private BigDecimal finalAmount;
    private String paymentId;
    private String trackingNumber;
    private LocalDateTime createTime;
    private LocalDateTime updateTime;
    private Map<String, Object> properties;
    
    public Order(String orderId, String userId) {
        this.orderId = orderId;
        this.userId = userId;
        this.status = OrderStatus.CREATED;
        this.items = new ArrayList<>();
        this.properties = new HashMap<>();
        this.createTime = LocalDateTime.now();
        this.updateTime = LocalDateTime.now();
    }
    
    /**
     * 创建备忘录
     */
    public OrderMemento createMemento(String description) {
        return new OrderMemento(this, description);
    }
    
    /**
     * 从备忘录恢复状态
     */
    public void restoreFromMemento(OrderMemento memento) {
        if (!this.orderId.equals(memento.getOrderId())) {
            throw new IllegalArgumentException("备忘录不属于当前订单");
        }
        
        this.previousStatus = this.status;
        this.status = memento.getStatus();
        this.finalAmount = memento.getAmount();
        this.paymentId = memento.getPaymentId();
        this.trackingNumber = memento.getTrackingNumber();
        this.items = memento.getItems();
        this.properties = memento.getProperties();
        this.updateTime = LocalDateTime.now();
        
        System.out.println("订单状态已恢复到: " + memento.getDescription());
    }
    
    /**
     * 更新订单状态
     */
    public void updateStatus(OrderStatus newStatus, String reason) {
        this.previousStatus = this.status;
        this.status = newStatus;
        this.updateTime = LocalDateTime.now();
        System.out.println(String.format("订单 %s 状态更新: %s -> %s (%s)", 
            orderId, previousStatus, status, reason));
    }
    
    /**
     * 添加商品
     */
    public void addItem(OrderItem item) {
        this.items.add(item);
        recalculateAmount();
    }
    
    /**
     * 移除商品
     */
    public void removeItem(String productId) {
        items.removeIf(item -> item.getProductId().equals(productId));
        recalculateAmount();
    }
    
    /**
     * 设置属性
     */
    public void setProperty(String key, Object value) {
        properties.put(key, value);
        this.updateTime = LocalDateTime.now();
    }
    
    private void recalculateAmount() {
        this.originalAmount = items.stream()
            .map(item -> item.getPrice().multiply(BigDecimal.valueOf(item.getQuantity())))
            .reduce(BigDecimal.ZERO, BigDecimal::add);
        
        if (this.finalAmount == null) {
            this.finalAmount = this.originalAmount;
        }
    }
    
    // Getters and Setters
    public String getOrderId() { return orderId; }
    public String getUserId() { return userId; }
    public OrderStatus getStatus() { return status; }
    public OrderStatus getPreviousStatus() { return previousStatus; }
    public List<OrderItem> getItems() { return new ArrayList<>(items); }
    public BigDecimal getOriginalAmount() { return originalAmount; }
    public BigDecimal getFinalAmount() { return finalAmount; }
    public void setFinalAmount(BigDecimal finalAmount) { this.finalAmount = finalAmount; }
    public String getPaymentId() { return paymentId; }
    public void setPaymentId(String paymentId) { this.paymentId = paymentId; }
    public String getTrackingNumber() { return trackingNumber; }
    public void setTrackingNumber(String trackingNumber) { this.trackingNumber = trackingNumber; }
    public LocalDateTime getCreateTime() { return createTime; }
    public LocalDateTime getUpdateTime() { return updateTime; }
    public Map<String, Object> getProperties() { return new HashMap<>(properties); }
    
    @Override
    public String toString() {
        return String.format("Order{id='%s', status=%s, amount=¥%s, items=%d}", 
            orderId, status, finalAmount, items.size());
    }
}

/**
 * 管理者 - 订单历史管理器
 */
public class OrderHistoryManager {
    private Map<String, Stack<OrderMemento>> orderHistories;
    private static final int MAX_HISTORY_SIZE = 50;
    
    public OrderHistoryManager() {
        this.orderHistories = new HashMap<>();
    }
    
    /**
     * 保存订单状态
     */
    public void saveState(Order order, String description) {
        String orderId = order.getOrderId();
        
        orderHistories.computeIfAbsent(orderId, k -> new Stack<>());
        Stack<OrderMemento> history = orderHistories.get(orderId);
        
        // 创建备忘录
        OrderMemento memento = order.createMemento(description);
        history.push(memento);
        
        // 限制历史记录大小
        if (history.size() > MAX_HISTORY_SIZE) {
            // 移除最旧的记录
            Stack<OrderMemento> newHistory = new Stack<>();
            List<OrderMemento> mementos = new ArrayList<>(history);
            mementos.subList(1, mementos.size()).forEach(newHistory::push);
            orderHistories.put(orderId, newHistory);
        }
        
        System.out.println("保存订单状态: " + description);
    }
    
    /**
     * 回滚订单到上一个状态
     */
    public boolean rollback(Order order) {
        String orderId = order.getOrderId();
        Stack<OrderMemento> history = orderHistories.get(orderId);
        
        if (history == null || history.size() <= 1) {
            System.out.println("没有可回滚的历史状态");
            return false;
        }
        
        // 移除当前状态
        history.pop();
        
        // 恢复到上一个状态
        OrderMemento previousState = history.peek();
        order.restoreFromMemento(previousState);
        
        System.out.println("订单已回滚到: " + previousState.getDescription());
        return true;
    }
    
    /**
     * 回滚到指定的历史状态
     */
    public boolean rollbackToVersion(Order order, int versionFromTop) {
        String orderId = order.getOrderId();
        Stack<OrderMemento> history = orderHistories.get(orderId);
        
        if (history == null || history.size() <= versionFromTop) {
            System.out.println("指定的历史版本不存在");
            return false;
        }
        
        // 获取指定版本
        List<OrderMemento> mementos = new ArrayList<>(history);
        int targetIndex = mementos.size() - 1 - versionFromTop;
        OrderMemento targetMemento = mementos.get(targetIndex);
        
        // 恢复状态
        order.restoreFromMemento(targetMemento);
        
        // 移除指定版本之后的所有历史
        while (history.size() > targetIndex + 1) {
            history.pop();
        }
        
        System.out.println("订单已回滚到版本: " + targetMemento.getDescription());
        return true;
    }
    
    /**
     * 显示订单历史
     */
    public void showHistory(String orderId) {
        Stack<OrderMemento> history = orderHistories.get(orderId);
        
        if (history == null || history.isEmpty()) {
            System.out.println("订单 " + orderId + " 没有历史记录");
            return;
        }
        
        System.out.println("=== 订单 " + orderId + " 历史记录 ===");
        List<OrderMemento> mementos = new ArrayList<>(history);
        
        for (int i = mementos.size() - 1; i >= 0; i--) {
            OrderMemento memento = mementos.get(i);
            String indicator = (i == mementos.size() - 1) ? " <- 当前" : "";
            System.out.println(String.format("%d. %s%s", 
                mementos.size() - i, memento.toString(), indicator));
        }
    }
    
    /**
     * 清理订单历史
     */
    public void clearHistory(String orderId) {
        orderHistories.remove(orderId);
        System.out.println("订单 " + orderId + " 历史记录已清理");
    }
    
    /**
     * 获取历史统计信息
     */
    public void showStatistics() {
        System.out.println("=== 历史记录统计 ===");
        System.out.println("管理的订单数量: " + orderHistories.size());
        
        int totalMementos = orderHistories.values().stream()
            .mapToInt(Stack::size)
            .sum();
        System.out.println("总历史记录数量: " + totalMementos);
        
        if (!orderHistories.isEmpty()) {
            double avgHistorySize = (double) totalMementos / orderHistories.size();
            System.out.println(String.format("平均每订单历史记录: %.2f", avgHistorySize));
        }
    }
}

/**
 * 购物车备忘录
 */
public class ShoppingCartMemento implements Memento {
    private List<OrderItem> items;
    private Map<String, Object> properties;
    private LocalDateTime timestamp;
    private String description;
    
    public ShoppingCartMemento(ShoppingCart cart, String description) {
        this.items = new ArrayList<>(cart.getItems());
        this.properties = new HashMap<>(cart.getProperties());
        this.timestamp = LocalDateTime.now();
        this.description = description;
    }
    
    List<OrderItem> getItems() { return new ArrayList<>(items); }
    Map<String, Object> getProperties() { return new HashMap<>(properties); }
    
    @Override
    public String getDescription() { return description; }
    
    @Override
    public LocalDateTime getTimestamp() { return timestamp; }
    
    @Override
    public String toString() {
        return String.format("%s - %s (%d items)", 
            timestamp.format(DateTimeFormatter.ofPattern("HH:mm:ss")),
            description, items.size());
    }
}

/**
 * 增强的购物车类
 */
public class ShoppingCart {
    private String userId;
    private List<OrderItem> items;
    private Map<String, Object> properties;
    private LocalDateTime createTime;
    private LocalDateTime updateTime;
    
    public ShoppingCart(String userId) {
        this.userId = userId;
        this.items = new ArrayList<>();
        this.properties = new HashMap<>();
        this.createTime = LocalDateTime.now();
        this.updateTime = LocalDateTime.now();
    }
    
    public ShoppingCartMemento createMemento(String description) {
        return new ShoppingCartMemento(this, description);
    }
    
    public void restoreFromMemento(ShoppingCartMemento memento) {
        this.items = memento.getItems();
        this.properties = memento.getProperties();
        this.updateTime = LocalDateTime.now();
        System.out.println("购物车已恢复到: " + memento.getDescription());
    }
    
    public void addItem(OrderItem item) {
        items.add(item);
        this.updateTime = LocalDateTime.now();
    }
    
    public void removeItem(String productId) {
        items.removeIf(i -> i.getProductId().equals(productId));
        this.updateTime = LocalDateTime.now();
    }
    
    public void clear() {
        items.clear();
        this.updateTime = LocalDateTime.now();
    }
    
    // Getters
    public String getUserId() { return userId; }
    public List<OrderItem> getItems() { return new ArrayList<>(items); }
    public Map<String, Object> getProperties() { return new HashMap<>(properties); }
    public void setProperty(String key, Object value) { 
        properties.put(key, value);
        this.updateTime = LocalDateTime.now();
    }
    
    public BigDecimal getTotalAmount() {
        return items.stream()
            .map(item -> item.getPrice().multiply(BigDecimal.valueOf(item.getQuantity())))
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
}

/**
 * 购物车历史管理器
 */
public class ShoppingCartHistoryManager {
    private Map<String, Stack<ShoppingCartMemento>> cartHistories;
    private static final int MAX_HISTORY_SIZE = 20;
    
    public ShoppingCartHistoryManager() {
        this.cartHistories = new HashMap<>();
    }
    
    public void saveState(ShoppingCart cart, String description) {
        String userId = cart.getUserId();
        
        cartHistories.computeIfAbsent(userId, k -> new Stack<>());
        Stack<ShoppingCartMemento> history = cartHistories.get(userId);
        
        ShoppingCartMemento memento = cart.createMemento(description);
        history.push(memento);
        
        if (history.size() > MAX_HISTORY_SIZE) {
            Stack<ShoppingCartMemento> newHistory = new Stack<>();
            List<ShoppingCartMemento> mementos = new ArrayList<>(history);
            mementos.subList(1, mementos.size()).forEach(newHistory::push);
            cartHistories.put(userId, newHistory);
        }
    }
    
    public boolean undo(ShoppingCart cart) {
        String userId = cart.getUserId();
        Stack<ShoppingCartMemento> history = cartHistories.get(userId);
        
        if (history == null || history.size() <= 1) {
            System.out.println("没有可撤销的操作");
            return false;
        }
        
        history.pop(); // 移除当前状态
        ShoppingCartMemento previousState = history.peek();
        cart.restoreFromMemento(previousState);
        
        return true;
    }
    
    public void showHistory(String userId) {
        Stack<ShoppingCartMemento> history = cartHistories.get(userId);
        
        if (history == null || history.isEmpty()) {
            System.out.println("用户 " + userId + " 没有购物车历史");
            return;
        }
        
        System.out.println("=== 购物车操作历史 ===");
        List<ShoppingCartMemento> mementos = new ArrayList<>(history);
        
        for (int i = mementos.size() - 1; i >= 0; i--) {
            ShoppingCartMemento memento = mementos.get(i);
            String indicator = (i == mementos.size() - 1) ? " <- 当前" : "";
            System.out.println((mementos.size() - i) + ". " + memento.toString() + indicator);
        }
    }
}

/**
 * 备忘录模式使用示例
 */
public class MementoPatternExample {
    
    public void demonstrateMemento() {
        System.out.println("=== 备忘录模式演示 ===\n");
        
        // 1. 订单状态管理演示
        demonstrateOrderHistory();
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // 2. 购物车历史管理演示
        demonstrateShoppingCartHistory();
    }
    
    private void demonstrateOrderHistory() {
        System.out.println("1. 订单状态历史管理演示:");
        
        // 创建订单和历史管理器
        Order order = new Order("ORD001", "U001");
        OrderHistoryManager historyManager = new OrderHistoryManager();
        
        // 添加商品
        order.addItem(new OrderItem("P001", "iPhone 14", new BigDecimal("5999"), 1));
        order.addItem(new OrderItem("P002", "AirPods Pro", new BigDecimal("1899"), 1));
        historyManager.saveState(order, "订单创建");
        
        // 更新订单状态
        order.updateStatus(OrderStatus.PAID, "支付完成");
        order.setPaymentId("PAY001");
        historyManager.saveState(order, "支付完成");
        
        order.updateStatus(OrderStatus.SHIPPED, "商品发货");
        order.setTrackingNumber("TN12345");
        historyManager.saveState(order, "商品发货");
        
        order.updateStatus(OrderStatus.DELIVERED, "订单送达");
        historyManager.saveState(order, "订单送达");
        
        // 显示当前状态
        System.out.println("\n当前订单状态: " + order);
        
        // 显示历史记录
        System.out.println();
        historyManager.showHistory("ORD001");
        
        // 回滚测试
        System.out.println("\n执行回滚操作:");
        historyManager.rollback(order);
        System.out.println("回滚后状态: " + order);
        
        // 回滚到指定版本
        System.out.println("\n回滚到支付完成状态:");
        historyManager.rollbackToVersion(order, 2);
        System.out.println("回滚后状态: " + order);
        
        // 再次显示历史
        System.out.println();
        historyManager.showHistory("ORD001");
    }
    
    private void demonstrateShoppingCartHistory() {
        System.out.println("2. 购物车历史管理演示:");
        
        ShoppingCart cart = new ShoppingCart("U001");
        ShoppingCartHistoryManager historyManager = new ShoppingCartHistoryManager();
        
        // 空购物车状态
        historyManager.saveState(cart, "空购物车");
        
        // 添加商品
        cart.addItem(new OrderItem("P001", "iPhone 14", new BigDecimal("5999"), 1));
        historyManager.saveState(cart, "添加iPhone 14");
        
        cart.addItem(new OrderItem("P002", "AirPods Pro", new BigDecimal("1899"), 1));
        historyManager.saveState(cart, "添加AirPods Pro");
        
        cart.addItem(new OrderItem("P003", "MacBook Pro", new BigDecimal("15999"), 1));
        historyManager.saveState(cart, "添加MacBook Pro");
        
        // 显示当前购物车
        System.out.println("\n当前购物车:");
        cart.getItems().forEach(item -> System.out.println("- " + item));
        System.out.println("总金额: ¥" + cart.getTotalAmount());
        
        // 显示历史
        System.out.println();
        historyManager.showHistory("U001");
        
        // 撤销操作
        System.out.println("\n执行撤销操作:");
        historyManager.undo(cart);
        
        System.out.println("撤销后购物车:");
        cart.getItems().forEach(item -> System.out.println("- " + item));
        System.out.println("总金额: ¥" + cart.getTotalAmount());
        
        // 再次撤销
        System.out.println("\n再次撤销:");
        historyManager.undo(cart);
        
        System.out.println("撤销后购物车:");
        cart.getItems().forEach(item -> System.out.println("- " + item));
        System.out.println("总金额: ¥" + cart.getTotalAmount());
    }
}

### 19. 观察者模式 (Observer Pattern)

**场景**: 订单状态变更通知、库存变化监听、价格变动提醒

```java
/**
 * 订单观察者接口
 * 参考：Spring事件机制
 */
public interface OrderObserver {
    void update(OrderEvent event);
    String getObserverName();
}

/**
 * 订单事件
 */
public class OrderEvent {
    private String orderId;
    private String eventType;
    private OrderStatus oldStatus;
    private OrderStatus newStatus;
    private Object eventData;
    private LocalDateTime timestamp;
    
    public OrderEvent(String orderId, String eventType, OrderStatus oldStatus, 
                     OrderStatus newStatus, Object eventData) {
        this.orderId = orderId;
        this.eventType = eventType;
        this.oldStatus = oldStatus;
        this.newStatus = newStatus;
        this.eventData = eventData;
        this.timestamp = LocalDateTime.now();
    }
    
    // Getters
    public String getOrderId() { return orderId; }
    public String getEventType() { return eventType; }
    public OrderStatus getOldStatus() { return oldStatus; }
    public OrderStatus getNewStatus() { return newStatus; }
    public Object getEventData() { return eventData; }
    public LocalDateTime getTimestamp() { return timestamp; }
    
    @Override
    public String toString() {
        return String.format("OrderEvent{orderId='%s', eventType='%s', %s->%s, time=%s}",
            orderId, eventType, oldStatus, newStatus, 
            timestamp.format(DateTimeFormatter.ofPattern("HH:mm:ss")));
    }
}

/**
 * 订单主题 - 被观察者
 */
public class OrderSubject {
    private List<OrderObserver> observers;
    private String orderId;
    private OrderStatus currentStatus;
    
    public OrderSubject(String orderId) {
        this.orderId = orderId;
        this.observers = new ArrayList<>();
        this.currentStatus = OrderStatus.CREATED;
    }
    
    /**
     * 添加观察者
     */
    public void addObserver(OrderObserver observer) {
        observers.add(observer);
        System.out.println("添加观察者: " + observer.getObserverName());
    }
    
    /**
     * 移除观察者
     */
    public void removeObserver(OrderObserver observer) {
        observers.remove(observer);
        System.out.println("移除观察者: " + observer.getObserverName());
    }
    
    /**
     * 通知所有观察者
     */
    public void notifyObservers(OrderEvent event) {
        System.out.println("通知所有观察者: " + event);
        for (OrderObserver observer : observers) {
            try {
                observer.update(event);
            } catch (Exception e) {
                System.out.println("观察者 " + observer.getObserverName() + " 处理事件时发生异常: " + e.getMessage());
            }
        }
    }
    
    /**
     * 更新订单状态
     */
    public void updateOrderStatus(OrderStatus newStatus, Object eventData) {
        OrderStatus oldStatus = this.currentStatus;
        this.currentStatus = newStatus;
        
        // 创建事件并通知观察者
        OrderEvent event = new OrderEvent(orderId, "STATUS_CHANGE", 
            oldStatus, newStatus, eventData);
        notifyObservers(event);
    }
    
    /**
     * 触发特定事件
     */
    public void fireEvent(String eventType, Object eventData) {
        OrderEvent event = new OrderEvent(orderId, eventType, 
            currentStatus, currentStatus, eventData);
        notifyObservers(event);
    }
    
    public String getOrderId() { return orderId; }
    public OrderStatus getCurrentStatus() { return currentStatus; }
    public int getObserverCount() { return observers.size(); }
}

/**
 * 具体观察者 - 邮件通知服务
 */
public class EmailNotificationObserver implements OrderObserver {
    private String email;
    
    public EmailNotificationObserver(String email) {
        this.email = email;
    }
    
    @Override
    public void update(OrderEvent event) {
        switch (event.getEventType()) {
            case "STATUS_CHANGE":
                handleStatusChange(event);
                break;
            case "PAYMENT_SUCCESS":
                handlePaymentSuccess(event);
                break;
            case "SHIPMENT_CREATED":
                handleShipmentCreated(event);
                break;
            default:
                System.out.println("[邮件通知] 收到未知事件: " + event.getEventType());
        }
    }
    
    private void handleStatusChange(OrderEvent event) {
        System.out.println(String.format("[邮件通知] 发送状态变更邮件到 %s", email));
        System.out.println("  主题: 订单状态更新 - " + event.getOrderId());
        System.out.println("  内容: 您的订单状态已从 " + event.getOldStatus() + 
            " 更新为 " + event.getNewStatus());
    }
    
    private void handlePaymentSuccess(OrderEvent event) {
        System.out.println("[邮件通知] 发送支付成功邮件到 " + email);
        System.out.println("  主题: 支付成功确认");
        System.out.println("  内容: 订单 " + event.getOrderId() + " 支付成功");
    }
    
    private void handleShipmentCreated(OrderEvent event) {
        System.out.println("[邮件通知] 发送发货通知邮件到 " + email);
        System.out.println("  主题: 商品已发货");
        System.out.println("  内容: 订单 " + event.getOrderId() + " 已发货，物流信息: " + event.getEventData());
    }
    
    @Override
    public String getObserverName() {
        return "邮件通知服务(" + email + ")";
    }
}

/**
 * 具体观察者 - 短信通知服务
 */
public class SmsNotificationObserver implements OrderObserver {
    private String phoneNumber;
    
    public SmsNotificationObserver(String phoneNumber) {
        this.phoneNumber = phoneNumber;
    }
    
    @Override
    public void update(OrderEvent event) {
        // 短信通知只关心重要状态变更
        if ("STATUS_CHANGE".equals(event.getEventType())) {
            OrderStatus newStatus = event.getNewStatus();
            if (newStatus == OrderStatus.PAID || newStatus == OrderStatus.SHIPPED || 
                newStatus == OrderStatus.DELIVERED) {
                sendSms(event);
            }
        } else if ("PAYMENT_SUCCESS".equals(event.getEventType())) {
            sendPaymentSms(event);
        }
    }
    
    private void sendSms(OrderEvent event) {
        System.out.println("[短信通知] 发送短信到 " + phoneNumber);
        String message = getStatusMessage(event.getNewStatus(), event.getOrderId());
        System.out.println("  内容: " + message);
    }
    
    private void sendPaymentSms(OrderEvent event) {
        System.out.println("[短信通知] 发送支付成功短信到 " + phoneNumber);
        System.out.println("  内容: 订单" + event.getOrderId() + "支付成功，请等待发货。【商城】");
    }
    
    private String getStatusMessage(OrderStatus status, String orderId) {
        switch (status) {
            case PAID:
                return "订单" + orderId + "支付成功，正在准备发货。【商城】";
            case SHIPPED:
                return "订单" + orderId + "已发货，请注意查收。【商城】";
            case DELIVERED:
                return "订单" + orderId + "已送达，请确认收货。【商城】";
            default:
                return "订单" + orderId + "状态已更新为" + status + "。【商城】";
        }
    }
    
    @Override
    public String getObserverName() {
        return "短信通知服务(" + phoneNumber + ")";
    }
}

/**
 * 具体观察者 - 库存管理服务
 */
public class InventoryManagementObserver implements OrderObserver {
    private Map<String, Integer> inventory;
    
    public InventoryManagementObserver() {
        this.inventory = new HashMap<>();
        // 初始化库存
        inventory.put("P001", 100);
        inventory.put("P002", 50);
        inventory.put("P003", 200);
    }
    
    @Override
    public void update(OrderEvent event) {
        if ("STATUS_CHANGE".equals(event.getEventType())) {
            handleStatusChange(event);
        }
    }
    
    private void handleStatusChange(OrderEvent event) {
        OrderStatus newStatus = event.getNewStatus();
        
        switch (newStatus) {
            case PAID:
                // 支付成功后扣减库存
                deductInventory(event);
                break;
            case CANCELLED:
                // 订单取消后释放库存
                releaseInventory(event);
                break;
        }
    }
    
    private void deductInventory(OrderEvent event) {
        System.out.println("[库存管理] 订单支付成功，扣减库存");
        // 这里假设事件数据包含订单商品信息
        if (event.getEventData() instanceof List) {
            @SuppressWarnings("unchecked")
            List<OrderItem> items = (List<OrderItem>) event.getEventData();
            
            for (OrderItem item : items) {
                String productId = item.getProductId();
                int quantity = item.getQuantity();
                int currentStock = inventory.getOrDefault(productId, 0);
                
                if (currentStock >= quantity) {
                    inventory.put(productId, currentStock - quantity);
                    System.out.println("  扣减库存: " + productId + " -" + quantity + 
                        ", 剩余: " + inventory.get(productId));
                } else {
                    System.out.println("  警告: " + productId + " 库存不足，无法扣减");
                }
            }
        }
    }
    
    private void releaseInventory(OrderEvent event) {
        System.out.println("[库存管理] 订单取消，释放库存");
        // 实际实现中需要根据具体业务逻辑处理库存释放
    }
    
    @Override
    public String getObserverName() {
        return "库存管理服务";
    }
}

/**
 * 具体观察者 - 数据统计服务
 */
public class StatisticsObserver implements OrderObserver {
    private Map<OrderStatus, Integer> statusCounts;
    private Map<String, Integer> eventCounts;
    
    public StatisticsObserver() {
        this.statusCounts = new HashMap<>();
        this.eventCounts = new HashMap<>();
        
        // 初始化计数器
        for (OrderStatus status : OrderStatus.values()) {
            statusCounts.put(status, 0);
        }
    }
    
    @Override
    public void update(OrderEvent event) {
        // 统计事件类型
        eventCounts.merge(event.getEventType(), 1, Integer::sum);
        
        if ("STATUS_CHANGE".equals(event.getEventType())) {
            // 统计订单状态
            OrderStatus newStatus = event.getNewStatus();
            statusCounts.merge(newStatus, 1, Integer::sum);
            
            System.out.println("[数据统计] 记录状态变更: " + event.getOrderId() + 
                " -> " + newStatus);
        }
        
        // 输出统计信息
        if (eventCounts.values().stream().mapToInt(Integer::intValue).sum() % 5 == 0) {
            printStatistics();
        }
    }
    
    public void printStatistics() {
        System.out.println("=== 订单统计信息 ===");
        System.out.println("状态统计:");
        statusCounts.forEach((status, count) -> {
            if (count > 0) {
                System.out.println("  " + status + ": " + count + "次");
            }
        });
        
        System.out.println("事件统计:");
        eventCounts.forEach((event, count) -> 
            System.out.println("  " + event + ": " + count + "次"));
    }
    
    @Override
    public String getObserverName() {
        return "数据统计服务";
    }
}

/**
 * 观察者管理器
 */
public class ObserverManager {
    private Map<String, OrderSubject> subjects;
    private List<OrderObserver> globalObservers;
    
    public ObserverManager() {
        this.subjects = new HashMap<>();
        this.globalObservers = new ArrayList<>();
    }
    
    /**
     * 获取或创建订单主题
     */
    public OrderSubject getOrderSubject(String orderId) {
        return subjects.computeIfAbsent(orderId, OrderSubject::new);
    }
    
    /**
     * 添加全局观察者（监听所有订单）
     */
    public void addGlobalObserver(OrderObserver observer) {
        globalObservers.add(observer);
        System.out.println("添加全局观察者: " + observer.getObserverName());
        
        // 为所有现有订单主题添加此观察者
        subjects.values().forEach(subject -> subject.addObserver(observer));
    }
    
    /**
     * 移除全局观察者
     */
    public void removeGlobalObserver(OrderObserver observer) {
        globalObservers.remove(observer);
        subjects.values().forEach(subject -> subject.removeObserver(observer));
    }
    
    /**
     * 创建新订单主题时自动添加全局观察者
     */
    private void attachGlobalObservers(OrderSubject subject) {
        globalObservers.forEach(subject::addObserver);
    }
    
    /**
     * 更新订单状态
     */
    public void updateOrderStatus(String orderId, OrderStatus newStatus, Object eventData) {
        OrderSubject subject = getOrderSubject(orderId);
        if (subjects.get(orderId) == null) {
            // 新订单，添加全局观察者
            attachGlobalObservers(subject);
        }
        subject.updateOrderStatus(newStatus, eventData);
    }
    
    /**
     * 触发订单事件
     */
    public void fireOrderEvent(String orderId, String eventType, Object eventData) {
        OrderSubject subject = getOrderSubject(orderId);
        subject.fireEvent(eventType, eventData);
    }
    
    /**
     * 获取统计信息
     */
    public void printManagerStatistics() {
        System.out.println("=== 观察者管理器统计 ===");
        System.out.println("管理的订单数量: " + subjects.size());
        System.out.println("全局观察者数量: " + globalObservers.size());
        
        subjects.forEach((orderId, subject) -> {
            System.out.println("订单 " + orderId + ": " + subject.getObserverCount() + " 个观察者");
        });
    }
}

/**
 * 观察者模式使用示例
 */
public class ObserverPatternExample {
    
    public void demonstrateObserver() {
        System.out.println("=== 观察者模式演示 ===\n");
        
        ObserverManager manager = new ObserverManager();
        
        // 创建全局观察者
        StatisticsObserver statisticsObserver = new StatisticsObserver();
        manager.addGlobalObserver(statisticsObserver);
        
        // 创建订单特定的观察者
        EmailNotificationObserver emailObserver = new EmailNotificationObserver("user@example.com");
        SmsNotificationObserver smsObserver = new SmsNotificationObserver("13800138000");
        InventoryManagementObserver inventoryObserver = new InventoryManagementObserver();
        
        // 为特定订单添加观察者
        String orderId = "ORD001";
        OrderSubject orderSubject = manager.getOrderSubject(orderId);
        orderSubject.addObserver(emailObserver);
        orderSubject.addObserver(smsObserver);
        orderSubject.addObserver(inventoryObserver);
        
        System.out.println("\n=== 模拟订单处理流程 ===");
        
        // 模拟订单状态变更
        List<OrderItem> orderItems = Arrays.asList(
            new OrderItem("P001", "iPhone 14", new BigDecimal("5999"), 1),
            new OrderItem("P002", "AirPods Pro", new BigDecimal("1899"), 1)
        );
        
        // 1. 订单支付成功
        System.out.println("\n1. 订单支付成功");
        manager.updateOrderStatus(orderId, OrderStatus.PAID, orderItems);
        manager.fireOrderEvent(orderId, "PAYMENT_SUCCESS", "支付金额: ¥7898");
        
        System.out.println("\n" + "-".repeat(30));
        
        // 2. 订单发货
        System.out.println("\n2. 订单发货");
        manager.updateOrderStatus(orderId, OrderStatus.SHIPPED, orderItems);
        manager.fireOrderEvent(orderId, "SHIPMENT_CREATED", "物流单号: SF123456789");
        
        System.out.println("\n" + "-".repeat(30));
        
        // 3. 订单送达
        System.out.println("\n3. 订单送达");
        manager.updateOrderStatus(orderId, OrderStatus.DELIVERED, orderItems);
        
        System.out.println("\n" + "-".repeat(30));
        
        // 4. 处理第二个订单
        System.out.println("\n4. 处理第二个订单");
        String orderId2 = "ORD002";
        manager.updateOrderStatus(orderId2, OrderStatus.PAID, Arrays.asList(
            new OrderItem("P003", "MacBook Pro", new BigDecimal("15999"), 1)
        ));
        
        // 取消第二个订单
        manager.updateOrderStatus(orderId2, OrderStatus.CANCELLED, null);
        
        System.out.println("\n" + "=".repeat(50));
        
        // 显示最终统计
        manager.printManagerStatistics();
        statisticsObserver.printStatistics();
    }
}

### 20. 状态模式 (State Pattern)

**场景**: 订单状态流转、支付状态管理、物流状态跟踪

```java
/**
 * 订单状态接口
 * 参考：状态机模式实现
 */
public interface OrderState {
    void handlePayment(OrderContext context);
    void handleShipment(OrderContext context);
    void handleDelivery(OrderContext context);
    void handleCancellation(OrderContext context);
    void handleReturn(OrderContext context);
    
    OrderStatus getStatus();
    String getStatusDescription();
    List<OrderStatus> getAllowedTransitions();
}

/**
 * 订单上下文
 */
public class OrderContext {
    private String orderId;
    private String userId;
    private List<OrderItem> items;
    private BigDecimal totalAmount;
    private String paymentId;
    private String trackingNumber;
    private OrderState currentState;
    private List<StateTransition> stateHistory;
    private LocalDateTime createTime;
    private LocalDateTime updateTime;
    
    public OrderContext(String orderId, String userId, List<OrderItem> items) {
        this.orderId = orderId;
        this.userId = userId;
        this.items = items;
        this.totalAmount = calculateTotalAmount();
        this.stateHistory = new ArrayList<>();
        this.createTime = LocalDateTime.now();
        this.updateTime = LocalDateTime.now();
        
        // 初始状态
        setState(new CreatedState());
    }
    
    public void setState(OrderState newState) {
        OrderState oldState = this.currentState;
        this.currentState = newState;
        this.updateTime = LocalDateTime.now();
        
        // 记录状态转换历史
        if (oldState != null) {
            StateTransition transition = new StateTransition(
                oldState.getStatus(), 
                newState.getStatus(), 
                LocalDateTime.now()
            );
            stateHistory.add(transition);
            
            System.out.println(String.format("订单 %s 状态转换: %s -> %s", 
                orderId, oldState.getStatus(), newState.getStatus()));
        } else {
            System.out.println(String.format("订单 %s 初始化状态: %s", 
                orderId, newState.getStatus()));
        }
    }
    
    // 状态操作委托
    public void pay() {
        System.out.println("尝试支付订单: " + orderId);
        currentState.handlePayment(this);
    }
    
    public void ship() {
        System.out.println("尝试发货订单: " + orderId);
        currentState.handleShipment(this);
    }
    
    public void deliver() {
        System.out.println("尝试完成配送: " + orderId);
        currentState.handleDelivery(this);
    }
    
    public void cancel() {
        System.out.println("尝试取消订单: " + orderId);
        currentState.handleCancellation(this);
    }
    
    public void returnOrder() {
        System.out.println("尝试退货: " + orderId);
        currentState.handleReturn(this);
    }
    
    private BigDecimal calculateTotalAmount() {
        return items.stream()
            .map(item -> item.getPrice().multiply(BigDecimal.valueOf(item.getQuantity())))
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
    
    // Getters and Setters
    public String getOrderId() { return orderId; }
    public String getUserId() { return userId; }
    public List<OrderItem> getItems() { return items; }
    public BigDecimal getTotalAmount() { return totalAmount; }
    public String getPaymentId() { return paymentId; }
    public void setPaymentId(String paymentId) { this.paymentId = paymentId; }
    public String getTrackingNumber() { return trackingNumber; }
    public void setTrackingNumber(String trackingNumber) { this.trackingNumber = trackingNumber; }
    public OrderState getCurrentState() { return currentState; }
    public List<StateTransition> getStateHistory() { return new ArrayList<>(stateHistory); }
    public LocalDateTime getCreateTime() { return createTime; }
    public LocalDateTime getUpdateTime() { return updateTime; }
    
    public OrderStatus getCurrentStatus() {
        return currentState.getStatus();
    }
    
    public String getStatusDescription() {
        return currentState.getStatusDescription();
    }
    
    public List<OrderStatus> getAllowedTransitions() {
        return currentState.getAllowedTransitions();
    }
    
    public void printStateHistory() {
        System.out.println("=== 订单 " + orderId + " 状态历史 ===");
        System.out.println("创建时间: " + createTime.format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));
        
        for (StateTransition transition : stateHistory) {
            System.out.println(transition);
        }
        
        System.out.println("当前状态: " + currentState.getStatus() + " - " + currentState.getStatusDescription());
    }
}

/**
 * 状态转换记录
 */
public class StateTransition {
    private OrderStatus fromState;
    private OrderStatus toState;
    private LocalDateTime transitionTime;
    
    public StateTransition(OrderStatus fromState, OrderStatus toState, LocalDateTime transitionTime) {
        this.fromState = fromState;
        this.toState = toState;
        this.transitionTime = transitionTime;
    }
    
    public OrderStatus getFromState() { return fromState; }
    public OrderStatus getToState() { return toState; }
    public LocalDateTime getTransitionTime() { return transitionTime; }
    
    @Override
    public String toString() {
        return String.format("%s: %s -> %s", 
            transitionTime.format(DateTimeFormatter.ofPattern("HH:mm:ss")),
            fromState, toState);
    }
}

/**
 * 具体状态 - 已创建状态
 */
public class CreatedState implements OrderState {
    
    @Override
    public void handlePayment(OrderContext context) {
        System.out.println("处理支付...");
        
        // 模拟支付处理
        boolean paymentSuccess = processPayment(context);
        
        if (paymentSuccess) {
            context.setPaymentId("PAY" + System.currentTimeMillis());
            context.setState(new PaidState());
            System.out.println("支付成功，订单状态更新为已支付");
        } else {
            System.out.println("支付失败，订单保持创建状态");
        }
    }
    
    @Override
    public void handleShipment(OrderContext context) {
        System.out.println("错误: 未支付订单无法发货");
    }
    
    @Override
    public void handleDelivery(OrderContext context) {
        System.out.println("错误: 未支付订单无法完成配送");
    }
    
    @Override
    public void handleCancellation(OrderContext context) {
        System.out.println("取消未支付订单");
        context.setState(new CancelledState());
    }
    
    @Override
    public void handleReturn(OrderContext context) {
        System.out.println("错误: 未支付订单无法退货");
    }
    
    private boolean processPayment(OrderContext context) {
        // 模拟支付处理，90%成功率
        return Math.random() > 0.1;
    }
    
    @Override
    public OrderStatus getStatus() {
        return OrderStatus.CREATED;
    }
    
    @Override
    public String getStatusDescription() {
        return "订单已创建，等待支付";
    }
    
    @Override
    public List<OrderStatus> getAllowedTransitions() {
        return Arrays.asList(OrderStatus.PAID, OrderStatus.CANCELLED);
    }
}

/**
 * 具体状态 - 已支付状态
 */
public class PaidState implements OrderState {
    
    @Override
    public void handlePayment(OrderContext context) {
        System.out.println("错误: 订单已支付，无需重复支付");
    }
    
    @Override
    public void handleShipment(OrderContext context) {
        System.out.println("处理发货...");
        
        // 模拟发货处理
        String trackingNumber = generateTrackingNumber();
        context.setTrackingNumber(trackingNumber);
        context.setState(new ShippedState());
        
        System.out.println("发货成功，物流单号: " + trackingNumber);
    }
    
    @Override
    public void handleDelivery(OrderContext context) {
        System.out.println("错误: 订单尚未发货，无法完成配送");
    }
    
    @Override
    public void handleCancellation(OrderContext context) {
        System.out.println("取消已支付订单，处理退款...");
        
        // 模拟退款处理
        boolean refundSuccess = processRefund(context);
        
        if (refundSuccess) {
            context.setState(new CancelledState());
            System.out.println("退款成功，订单已取消");
        } else {
            System.out.println("退款失败，订单保持已支付状态");
        }
    }
    
    @Override
    public void handleReturn(OrderContext context) {
        System.out.println("错误: 订单尚未发货，无法退货");
    }
    
    private String generateTrackingNumber() {
        return "TN" + System.currentTimeMillis();
    }
    
    private boolean processRefund(OrderContext context) {
        // 模拟退款处理，95%成功率
        return Math.random() > 0.05;
    }
    
    @Override
    public OrderStatus getStatus() {
        return OrderStatus.PAID;
    }
    
    @Override
    public String getStatusDescription() {
        return "订单已支付，准备发货";
    }
    
    @Override
    public List<OrderStatus> getAllowedTransitions() {
        return Arrays.asList(OrderStatus.SHIPPED, OrderStatus.CANCELLED);
    }
}

/**
 * 具体状态 - 已发货状态
 */
public class ShippedState implements OrderState {
    
    @Override
    public void handlePayment(OrderContext context) {
        System.out.println("错误: 订单已发货，无需支付");
    }
    
    @Override
    public void handleShipment(OrderContext context) {
        System.out.println("错误: 订单已发货，无需重复发货");
    }
    
    @Override
    public void handleDelivery(OrderContext context) {
        System.out.println("确认收货，订单完成");
        context.setState(new DeliveredState());
    }
    
    @Override
    public void handleCancellation(OrderContext context) {
        System.out.println("错误: 订单已发货，无法取消。如需退货请联系客服");
    }
    
    @Override
    public void handleReturn(OrderContext context) {
        System.out.println("处理退货申请...");
        context.setState(new ReturnedState());
        System.out.println("退货申请已受理");
    }
    
    @Override
    public OrderStatus getStatus() {
        return OrderStatus.SHIPPED;
    }
    
    @Override
    public String getStatusDescription() {
        return "订单已发货，运输中";
    }
    
    @Override
    public List<OrderStatus> getAllowedTransitions() {
        return Arrays.asList(OrderStatus.DELIVERED, OrderStatus.RETURNED);
    }
}

/**
 * 具体状态 - 已送达状态
 */
public class DeliveredState implements OrderState {
    
    @Override
    public void handlePayment(OrderContext context) {
        System.out.println("错误: 订单已完成，无需支付");
    }
    
    @Override
    public void handleShipment(OrderContext context) {
        System.out.println("错误: 订单已完成，无需发货");
    }
    
    @Override
    public void handleDelivery(OrderContext context) {
        System.out.println("错误: 订单已送达");
    }
    
    @Override
    public void handleCancellation(OrderContext context) {
        System.out.println("错误: 订单已完成，无法取消");
    }
    
    @Override
    public void handleReturn(OrderContext context) {
        System.out.println("处理售后退货申请...");
        
        // 检查是否在退货期限内
        LocalDateTime now = LocalDateTime.now();
        LocalDateTime deliveryTime = context.getUpdateTime();
        long daysSinceDelivery = ChronoUnit.DAYS.between(deliveryTime, now);
        
        if (daysSinceDelivery <= 7) {
            context.setState(new ReturnedState());
            System.out.println("在退货期限内，退货申请已受理");
        } else {
            System.out.println("超出退货期限（7天），无法退货");
        }
    }
    
    @Override
    public OrderStatus getStatus() {
        return OrderStatus.DELIVERED;
    }
    
    @Override
    public String getStatusDescription() {
        return "订单已送达，交易完成";
    }
    
    @Override
    public List<OrderStatus> getAllowedTransitions() {
        return Arrays.asList(OrderStatus.RETURNED);
    }
}

/**
 * 具体状态 - 已取消状态
 */
public class CancelledState implements OrderState {
    
    @Override
    public void handlePayment(OrderContext context) {
        System.out.println("错误: 订单已取消，无法支付");
    }
    
    @Override
    public void handleShipment(OrderContext context) {
        System.out.println("错误: 订单已取消，无法发货");
    }
    
    @Override
    public void handleDelivery(OrderContext context) {
        System.out.println("错误: 订单已取消，无法配送");
    }
    
    @Override
    public void handleCancellation(OrderContext context) {
        System.out.println("订单已经是取消状态");
    }
    
    @Override
    public void handleReturn(OrderContext context) {
        System.out.println("错误: 订单已取消，无法退货");
    }
    
    @Override
    public OrderStatus getStatus() {
        return OrderStatus.CANCELLED;
    }
    
    @Override
    public String getStatusDescription() {
        return "订单已取消";
    }
    
    @Override
    public List<OrderStatus> getAllowedTransitions() {
        return Collections.emptyList(); // 终态，无法转换到其他状态
    }
}

/**
 * 具体状态 - 已退货状态
 */
public class ReturnedState implements OrderState {
    
    @Override
    public void handlePayment(OrderContext context) {
        System.out.println("错误: 订单已退货，无法支付");
    }
    
    @Override
    public void handleShipment(OrderContext context) {
        System.out.println("错误: 订单已退货，无法发货");
    }
    
    @Override
    public void handleDelivery(OrderContext context) {
        System.out.println("错误: 订单已退货，无法配送");
    }
    
    @Override
    public void handleCancellation(OrderContext context) {
        System.out.println("错误: 订单已退货，无法取消");
    }
    
    @Override
    public void handleReturn(OrderContext context) {
        System.out.println("订单已经是退货状态");
    }
    
    @Override
    public OrderStatus getStatus() {
        return OrderStatus.RETURNED;
    }
    
    @Override
    public String getStatusDescription() {
        return "订单已退货，退款处理中";
    }
    
    @Override
    public List<OrderStatus> getAllowedTransitions() {
        return Collections.emptyList(); // 终态，无法转换到其他状态
    }
}

/**
 * 订单状态机管理器
 */
public class OrderStateMachine {
    private Map<String, OrderContext> orders;
    
    public OrderStateMachine() {
        this.orders = new HashMap<>();
    }
    
    /**
     * 创建新订单
     */
    public OrderContext createOrder(String orderId, String userId, List<OrderItem> items) {
        OrderContext order = new OrderContext(orderId, userId, items);
        orders.put(orderId, order);
        System.out.println("创建新订单: " + orderId);
        return order;
    }
    
    /**
     * 获取订单
     */
    public OrderContext getOrder(String orderId) {
        return orders.get(orderId);
    }
    
    /**
     * 检查状态转换是否合法
     */
    public boolean isValidTransition(String orderId, OrderStatus targetStatus) {
        OrderContext order = getOrder(orderId);
        if (order == null) {
            return false;
        }
        
        return order.getAllowedTransitions().contains(targetStatus);
    }
    
    /**
     * 获取所有订单统计
     */
    public void printStatistics() {
        System.out.println("=== 订单状态机统计 ===");
        System.out.println("总订单数: " + orders.size());
        
        Map<OrderStatus, Long> statusCounts = orders.values().stream()
            .collect(Collectors.groupingBy(
                OrderContext::getCurrentStatus,
                Collectors.counting()
            ));
        
        System.out.println("状态分布:");
        statusCounts.forEach((status, count) -> 
            System.out.println("  " + status + ": " + count + "个"));
    }
}

/**
 * 状态模式使用示例
 */
public class StatePatternExample {
    
    public void demonstrateState() {
        System.out.println("=== 状态模式演示 ===\n");
        
        OrderStateMachine stateMachine = new OrderStateMachine();
        
        // 创建订单
        List<OrderItem> items = Arrays.asList(
            new OrderItem("P001", "iPhone 14", new BigDecimal("5999"), 1),
            new OrderItem("P002", "AirPods Pro", new BigDecimal("1899"), 1)
        );
        
        OrderContext order1 = stateMachine.createOrder("ORD001", "U001", items);
        
        System.out.println("\n=== 正常订单流程演示 ===");
        
        // 1. 支付订单
        System.out.println("\n1. 支付订单");
        order1.pay();
        
        // 2. 发货
        System.out.println("\n2. 发货订单");
        order1.ship();
        
        // 3. 确认收货
        System.out.println("\n3. 确认收货");
        order1.deliver();
        
        // 显示订单状态历史
        System.out.println();
        order1.printStateHistory();
        
        System.out.println("\n" + "=".repeat(50));
        
        // 异常情况演示
        System.out.println("\n=== 异常操作演示 ===");
        
        OrderContext order2 = stateMachine.createOrder("ORD002", "U002", items);
        
        // 尝试未支付发货
        System.out.println("\n1. 尝试未支付就发货");
        order2.ship();
        
        // 支付后尝试重复支付
        System.out.println("\n2. 支付后尝试重复支付");
        order2.pay();
        order2.pay();
        
        // 发货后尝试取消
        System.out.println("\n3. 发货后尝试取消");
        order2.ship();
        order2.cancel();
        
        // 申请退货
        System.out.println("\n4. 申请退货");
        order2.returnOrder();
        
        System.out.println("\n" + "=".repeat(50));
        
        // 订单取消流程演示
        System.out.println("\n=== 订单取消流程演示 ===");
        
        OrderContext order3 = stateMachine.createOrder("ORD003", "U003", items);
        
        // 支付后取消
        order3.pay();
        order3.cancel();
        
        System.out.println();
        order3.printStateHistory();
        
        // 显示最终统计
        System.out.println("\n" + "=".repeat(50));
        stateMachine.printStatistics();
    }
}

### 21. 策略模式 (Strategy Pattern)

**场景**: 支付策略选择、物流策略选择、定价策略

```java
/**
 * 支付策略接口
 * 参考：支付宝/微信支付架构
 */
public interface PaymentStrategy {
    PaymentResult processPayment(PaymentRequest request);
    boolean supportsCurrency(String currency);
    BigDecimal calculateFee(BigDecimal amount);
    String getPaymentMethodName();
    Map<String, Object> getPaymentConfig();
}

/**
 * 支付请求
 */
public class PaymentRequest {
    private String orderId;
    private String userId;
    private BigDecimal amount;
    private String currency;
    private Map<String, String> parameters;
    
    public PaymentRequest(String orderId, String userId, BigDecimal amount, String currency) {
        this.orderId = orderId;
        this.userId = userId;
        this.amount = amount;
        this.currency = currency;
        this.parameters = new HashMap<>();
    }
    
    // Getters and Setters
    public String getOrderId() { return orderId; }
    public String getUserId() { return userId; }
    public BigDecimal getAmount() { return amount; }
    public String getCurrency() { return currency; }
    public Map<String, String> getParameters() { return parameters; }
    public void setParameter(String key, String value) { parameters.put(key, value); }
    public String getParameter(String key) { return parameters.get(key); }
}

/**
 * 支付结果
 */
public class PaymentResult {
    private boolean success;
    private String transactionId;
    private String message;
    private BigDecimal actualAmount;
    private BigDecimal fee;
    private Map<String, Object> data;
    
    public PaymentResult(boolean success, String transactionId, String message) {
        this.success = success;
        this.transactionId = transactionId;
        this.message = message;
        this.data = new HashMap<>();
    }
    
    // Getters and Setters
    public boolean isSuccess() { return success; }
    public String getTransactionId() { return transactionId; }
    public String getMessage() { return message; }
    public BigDecimal getActualAmount() { return actualAmount; }
    public void setActualAmount(BigDecimal actualAmount) { this.actualAmount = actualAmount; }
    public BigDecimal getFee() { return fee; }
    public void setFee(BigDecimal fee) { this.fee = fee; }
    public Map<String, Object> getData() { return data; }
    public void setData(String key, Object value) { data.put(key, value); }
}

/**
 * 具体策略 - 支付宝支付
 */
public class AlipayStrategy implements PaymentStrategy {
    private String appId;
    private String privateKey;
    private String publicKey;
    
    public AlipayStrategy(String appId, String privateKey, String publicKey) {
        this.appId = appId;
        this.privateKey = privateKey;
        this.publicKey = publicKey;
    }
    
    @Override
    public PaymentResult processPayment(PaymentRequest request) {
        System.out.println("[支付宝] 处理支付请求");
        System.out.println("  订单号: " + request.getOrderId());
        System.out.println("  金额: " + request.getAmount() + " " + request.getCurrency());
        
        // 计算手续费
        BigDecimal fee = calculateFee(request.getAmount());
        BigDecimal actualAmount = request.getAmount().add(fee);
        
        // 模拟支付宝API调用
        boolean success = simulatePayment();
        
        PaymentResult result;
        if (success) {
            String transactionId = "ALIPAY_" + System.currentTimeMillis();
            result = new PaymentResult(true, transactionId, "支付成功");
            result.setActualAmount(actualAmount);
            result.setFee(fee);
            result.setData("paymentMethod", "支付宝");
            result.setData("appId", appId);
            
            System.out.println("  支付成功，交易号: " + transactionId);
        } else {
            result = new PaymentResult(false, null, "支付失败：余额不足或银行卡异常");
            System.out.println("  支付失败");
        }
        
        return result;
    }
    
    @Override
    public boolean supportsCurrency(String currency) {
        return Arrays.asList("CNY", "USD", "EUR").contains(currency);
    }
    
    @Override
    public BigDecimal calculateFee(BigDecimal amount) {
        // 支付宝手续费率：0.6%
        return amount.multiply(new BigDecimal("0.006")).setScale(2, RoundingMode.HALF_UP);
    }
    
    @Override
    public String getPaymentMethodName() {
        return "支付宝";
    }
    
    @Override
    public Map<String, Object> getPaymentConfig() {
        Map<String, Object> config = new HashMap<>();
        config.put("appId", appId);
        config.put("feeRate", "0.6%");
        config.put("supportedCurrencies", Arrays.asList("CNY", "USD", "EUR"));
        return config;
    }
    
    private boolean simulatePayment() {
        // 模拟90%成功率
        return Math.random() > 0.1;
    }
}

/**
 * 具体策略 - 微信支付
 */
public class WechatPayStrategy implements PaymentStrategy {
    private String mchId;
    private String appSecret;
    
    public WechatPayStrategy(String mchId, String appSecret) {
        this.mchId = mchId;
        this.appSecret = appSecret;
    }
    
    @Override
    public PaymentResult processPayment(PaymentRequest request) {
        System.out.println("[微信支付] 处理支付请求");
        System.out.println("  订单号: " + request.getOrderId());
        System.out.println("  金额: " + request.getAmount() + " " + request.getCurrency());
        
        BigDecimal fee = calculateFee(request.getAmount());
        BigDecimal actualAmount = request.getAmount().add(fee);
        
        boolean success = simulatePayment();
        
        PaymentResult result;
        if (success) {
            String transactionId = "WXPAY_" + System.currentTimeMillis();
            result = new PaymentResult(true, transactionId, "支付成功");
            result.setActualAmount(actualAmount);
            result.setFee(fee);
            result.setData("paymentMethod", "微信支付");
            result.setData("mchId", mchId);
            
            System.out.println("  支付成功，交易号: " + transactionId);
        } else {
            result = new PaymentResult(false, null, "支付失败：微信支付密码错误");
            System.out.println("  支付失败");
        }
        
        return result;
    }
    
    @Override
    public boolean supportsCurrency(String currency) {
        return "CNY".equals(currency); // 微信支付主要支持人民币
    }
    
    @Override
    public BigDecimal calculateFee(BigDecimal amount) {
        // 微信支付手续费率：0.5%
        return amount.multiply(new BigDecimal("0.005")).setScale(2, RoundingMode.HALF_UP);
    }
    
    @Override
    public String getPaymentMethodName() {
        return "微信支付";
    }
    
    @Override
    public Map<String, Object> getPaymentConfig() {
        Map<String, Object> config = new HashMap<>();
        config.put("mchId", mchId);
        config.put("feeRate", "0.5%");
        config.put("supportedCurrencies", Arrays.asList("CNY"));
        return config;
    }
    
    private boolean simulatePayment() {
        return Math.random() > 0.15; // 85%成功率
    }
}

/**
 * 具体策略 - 银行卡支付
 */
public class BankCardStrategy implements PaymentStrategy {
    private String bankCode;
    private String gatewayUrl;
    
    public BankCardStrategy(String bankCode, String gatewayUrl) {
        this.bankCode = bankCode;
        this.gatewayUrl = gatewayUrl;
    }
    
    @Override
    public PaymentResult processPayment(PaymentRequest request) {
        System.out.println("[银行卡支付] 处理支付请求");
        System.out.println("  银行: " + bankCode);
        System.out.println("  订单号: " + request.getOrderId());
        System.out.println("  金额: " + request.getAmount() + " " + request.getCurrency());
        
        BigDecimal fee = calculateFee(request.getAmount());
        BigDecimal actualAmount = request.getAmount().add(fee);
        
        boolean success = simulatePayment();
        
        PaymentResult result;
        if (success) {
            String transactionId = bankCode + "_" + System.currentTimeMillis();
            result = new PaymentResult(true, transactionId, "银行卡支付成功");
            result.setActualAmount(actualAmount);
            result.setFee(fee);
            result.setData("paymentMethod", "银行卡");
            result.setData("bankCode", bankCode);
            
            System.out.println("  支付成功，交易号: " + transactionId);
        } else {
            result = new PaymentResult(false, null, "支付失败：银行卡余额不足或网络异常");
            System.out.println("  支付失败");
        }
        
        return result;
    }
    
    @Override
    public boolean supportsCurrency(String currency) {
        return Arrays.asList("CNY", "USD", "EUR", "JPY").contains(currency);
    }
    
    @Override
    public BigDecimal calculateFee(BigDecimal amount) {
        // 银行卡支付手续费：固定2元 + 0.1%
        BigDecimal fixedFee = new BigDecimal("2.00");
        BigDecimal percentageFee = amount.multiply(new BigDecimal("0.001"));
        return fixedFee.add(percentageFee).setScale(2, RoundingMode.HALF_UP);
    }
    
    @Override
    public String getPaymentMethodName() {
        return "银行卡支付(" + bankCode + ")";
    }
    
    @Override
    public Map<String, Object> getPaymentConfig() {
        Map<String, Object> config = new HashMap<>();
        config.put("bankCode", bankCode);
        config.put("gatewayUrl", gatewayUrl);
        config.put("feeStructure", "固定2元 + 0.1%");
        config.put("supportedCurrencies", Arrays.asList("CNY", "USD", "EUR", "JPY"));
        return config;
    }
    
    private boolean simulatePayment() {
        return Math.random() > 0.2; // 80%成功率
    }
}

/**
 * 支付上下文 - 策略模式的上下文类
 */
public class PaymentContext {
    private PaymentStrategy paymentStrategy;
    private List<PaymentStrategy> availableStrategies;
    
    public PaymentContext() {
        this.availableStrategies = new ArrayList<>();
        initializeStrategies();
    }
    
    private void initializeStrategies() {
        // 初始化可用的支付策略
        availableStrategies.add(new AlipayStrategy("2021001234567890", "private_key", "public_key"));
        availableStrategies.add(new WechatPayStrategy("1234567890", "app_secret"));
        availableStrategies.add(new BankCardStrategy("ICBC", "https://gateway.icbc.com"));
        availableStrategies.add(new BankCardStrategy("CCB", "https://gateway.ccb.com"));
    }
    
    /**
     * 设置支付策略
     */
    public void setPaymentStrategy(PaymentStrategy strategy) {
        this.paymentStrategy = strategy;
        System.out.println("选择支付方式: " + strategy.getPaymentMethodName());
    }
    
    /**
     * 根据支付方式名称选择策略
     */
    public boolean selectPaymentMethod(String methodName) {
        for (PaymentStrategy strategy : availableStrategies) {
            if (strategy.getPaymentMethodName().contains(methodName)) {
                setPaymentStrategy(strategy);
                return true;
            }
        }
        System.out.println("未找到支付方式: " + methodName);
        return false;
    }
    
    /**
     * 智能选择最优支付策略
     */
    public void selectOptimalStrategy(PaymentRequest request) {
        System.out.println("智能选择最优支付策略...");
        
        PaymentStrategy bestStrategy = null;
        BigDecimal lowestFee = null;
        
        for (PaymentStrategy strategy : availableStrategies) {
            if (strategy.supportsCurrency(request.getCurrency())) {
                BigDecimal fee = strategy.calculateFee(request.getAmount());
                
                if (lowestFee == null || fee.compareTo(lowestFee) < 0) {
                    lowestFee = fee;
                    bestStrategy = strategy;
                }
                
                System.out.println("  " + strategy.getPaymentMethodName() + " 手续费: ¥" + fee);
            }
        }
        
        if (bestStrategy != null) {
            setPaymentStrategy(bestStrategy);
            System.out.println("选择最优策略: " + bestStrategy.getPaymentMethodName() + 
                " (手续费: ¥" + lowestFee + ")");
        } else {
            System.out.println("没有支持该币种的支付方式");
        }
    }
    
    /**
     * 执行支付
     */
    public PaymentResult executePayment(PaymentRequest request) {
        if (paymentStrategy == null) {
            return new PaymentResult(false, null, "未选择支付方式");
        }
        
        if (!paymentStrategy.supportsCurrency(request.getCurrency())) {
            return new PaymentResult(false, null, 
                "当前支付方式不支持币种: " + request.getCurrency());
        }
        
        return paymentStrategy.processPayment(request);
    }
    
    /**
     * 获取当前策略信息
     */
    public Map<String, Object> getCurrentStrategyInfo() {
        if (paymentStrategy == null) {
            return Collections.emptyMap();
        }
        
        Map<String, Object> info = new HashMap<>();
        info.put("name", paymentStrategy.getPaymentMethodName());
        info.put("config", paymentStrategy.getPaymentConfig());
        
        return info;
    }
    
    /**
     * 获取所有可用策略
     */
    public List<PaymentStrategy> getAvailableStrategies() {
        return new ArrayList<>(availableStrategies);
    }
    
    /**
     * 添加新的支付策略
     */
    public void addPaymentStrategy(PaymentStrategy strategy) {
        availableStrategies.add(strategy);
        System.out.println("添加新的支付策略: " + strategy.getPaymentMethodName());
    }
}

/**
 * 物流策略接口
 */
public interface LogisticsStrategy {
    BigDecimal calculateShippingCost(ShippingRequest request);
    int estimateDeliveryDays(String origin, String destination);
    String getLogisticsProvider();
    boolean supportsDestination(String destination);
}

/**
 * 物流请求
 */
public class ShippingRequest {
    private String origin;
    private String destination;
    private BigDecimal weight;
    private BigDecimal volume;
    private boolean isFragile;
    private boolean isUrgent;
    
    public ShippingRequest(String origin, String destination, BigDecimal weight, BigDecimal volume) {
        this.origin = origin;
        this.destination = destination;
        this.weight = weight;
        this.volume = volume;
    }
    
    // Getters and Setters
    public String getOrigin() { return origin; }
    public String getDestination() { return destination; }
    public BigDecimal getWeight() { return weight; }
    public BigDecimal getVolume() { return volume; }
    public boolean isFragile() { return isFragile; }
    public void setFragile(boolean fragile) { isFragile = fragile; }
    public boolean isUrgent() { return isUrgent; }
    public void setUrgent(boolean urgent) { isUrgent = urgent; }
}

/**
 * 具体物流策略 - 顺丰快递
 */
public class SFExpressStrategy implements LogisticsStrategy {
    
    @Override
    public BigDecimal calculateShippingCost(ShippingRequest request) {
        System.out.println("[顺丰快递] 计算运费");
        
        // 基础运费
        BigDecimal baseCost = new BigDecimal("15.00");
        
        // 重量费用：每公斤2元
        BigDecimal weightCost = request.getWeight().multiply(new BigDecimal("2.00"));
        
        // 体积费用：每立方分米0.5元
        BigDecimal volumeCost = request.getVolume().multiply(new BigDecimal("0.50"));
        
        BigDecimal totalCost = baseCost.add(weightCost).add(volumeCost);
        
        // 易碎品加收30%
        if (request.isFragile()) {
            totalCost = totalCost.multiply(new BigDecimal("1.30"));
        }
        
        // 加急服务加收50%
        if (request.isUrgent()) {
            totalCost = totalCost.multiply(new BigDecimal("1.50"));
        }
        
        return totalCost.setScale(2, RoundingMode.HALF_UP);
    }
    
    @Override
    public int estimateDeliveryDays(String origin, String destination) {
        // 顺丰速度较快
        if (isSameCity(origin, destination)) {
            return 1; // 同城当日达
        } else if (isSameProvince(origin, destination)) {
            return 2; // 省内次日达
        } else {
            return 3; // 全国3日达
        }
    }
    
    @Override
    public String getLogisticsProvider() {
        return "顺丰快递";
    }
    
    @Override
    public boolean supportsDestination(String destination) {
        // 顺丰覆盖全国
        return true;
    }
    
    private boolean isSameCity(String origin, String destination) {
        // 简化判断，实际应该根据城市编码
        return origin.substring(0, 2).equals(destination.substring(0, 2));
    }
    
    private boolean isSameProvince(String origin, String destination) {
        // 简化判断
        return origin.substring(0, 1).equals(destination.substring(0, 1));
    }
}

/**
 * 物流上下文
 */
public class LogisticsContext {
    private LogisticsStrategy logisticsStrategy;
    private List<LogisticsStrategy> availableStrategies;
    
    public LogisticsContext() {
        this.availableStrategies = new ArrayList<>();
        initializeStrategies();
    }
    
    private void initializeStrategies() {
        availableStrategies.add(new SFExpressStrategy());
        // 可以添加更多物流策略：圆通、中通、韵达等
    }
    
    public void selectOptimalLogistics(ShippingRequest request) {
        LogisticsStrategy bestStrategy = null;
        BigDecimal lowestCost = null;
        int fastestDelivery = Integer.MAX_VALUE;
        
        System.out.println("选择最优物流策略...");
        
        for (LogisticsStrategy strategy : availableStrategies) {
            if (strategy.supportsDestination(request.getDestination())) {
                BigDecimal cost = strategy.calculateShippingCost(request);
                int days = strategy.estimateDeliveryDays(request.getOrigin(), request.getDestination());
                
                System.out.println(String.format("  %s: 运费¥%s, 预计%d天送达", 
                    strategy.getLogisticsProvider(), cost, days));
                
                // 选择策略：优先考虑时效，其次考虑价格
                if (request.isUrgent()) {
                    if (days < fastestDelivery || (days == fastestDelivery && 
                        (lowestCost == null || cost.compareTo(lowestCost) < 0))) {
                        fastestDelivery = days;
                        lowestCost = cost;
                        bestStrategy = strategy;
                    }
                } else {
                    if (lowestCost == null || cost.compareTo(lowestCost) < 0) {
                        lowestCost = cost;
                        bestStrategy = strategy;
                    }
                }
            }
        }
        
        if (bestStrategy != null) {
            this.logisticsStrategy = bestStrategy;
            System.out.println("选择物流: " + bestStrategy.getLogisticsProvider());
        }
    }
    
    public BigDecimal calculateShippingCost(ShippingRequest request) {
        if (logisticsStrategy == null) {
            throw new IllegalStateException("未选择物流策略");
        }
        return logisticsStrategy.calculateShippingCost(request);
    }
}

/**
 * 策略模式使用示例
 */
public class StrategyPatternExample {
    
    public void demonstrateStrategy() {
        System.out.println("=== 策略模式演示 ===\n");
        
        // 1. 支付策略演示
        demonstratePaymentStrategy();
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // 2. 物流策略演示
        demonstrateLogisticsStrategy();
    }
    
    private void demonstratePaymentStrategy() {
        System.out.println("1. 支付策略演示:");
        
        PaymentContext paymentContext = new PaymentContext();
        
        // 创建支付请求
        PaymentRequest request = new PaymentRequest("ORD001", "U001", 
            new BigDecimal("999.00"), "CNY");
        
        // 场景1：手动选择支付方式
        System.out.println("\n场景1: 手动选择支付宝");
        paymentContext.selectPaymentMethod("支付宝");
        PaymentResult result1 = paymentContext.executePayment(request);
        System.out.println("支付结果: " + (result1.isSuccess() ? "成功" : "失败"));
        if (result1.isSuccess()) {
            System.out.println("交易号: " + result1.getTransactionId());
            System.out.println("手续费: ¥" + result1.getFee());
        }
        
        // 场景2：智能选择最优支付方式
        System.out.println("\n场景2: 智能选择最优支付方式");
        paymentContext.selectOptimalStrategy(request);
        PaymentResult result2 = paymentContext.executePayment(request);
        System.out.println("支付结果: " + (result2.isSuccess() ? "成功" : "失败"));
        
        // 场景3：测试不支持的币种
        System.out.println("\n场景3: 测试不支持的币种");
        PaymentRequest usdRequest = new PaymentRequest("ORD002", "U002", 
            new BigDecimal("100.00"), "GBP");
        paymentContext.selectPaymentMethod("微信支付");
        PaymentResult result3 = paymentContext.executePayment(usdRequest);
        System.out.println("支付结果: " + result3.getMessage());
        
        // 显示所有可用策略
        System.out.println("\n所有可用支付策略:");
        paymentContext.getAvailableStrategies().forEach(strategy -> {
            System.out.println("- " + strategy.getPaymentMethodName());
            System.out.println("  配置: " + strategy.getPaymentConfig());
        });
    }
    
    private void demonstrateLogisticsStrategy() {
        System.out.println("2. 物流策略演示:");
        
        LogisticsContext logisticsContext = new LogisticsContext();
        
        // 创建物流请求
        ShippingRequest request = new ShippingRequest("北京", "上海", 
            new BigDecimal("2.5"), new BigDecimal("0.1"));
        
        // 普通邮寄
        System.out.println("\n普通邮寄:");
        logisticsContext.selectOptimalLogistics(request);
        BigDecimal cost1 = logisticsContext.calculateShippingCost(request);
        System.out.println("运费: ¥" + cost1);
        
        // 易碎品加急邮寄
        System.out.println("\n易碎品加急邮寄:");
        request.setFragile(true);
        request.setUrgent(true);
        logisticsContext.selectOptimalLogistics(request);
        BigDecimal cost2 = logisticsContext.calculateShippingCost(request);
        System.out.println("运费: ¥" + cost2);
    }
}

### 22. 模板方法模式 (Template Method Pattern)

**场景**: 订单处理流程模板、支付流程模板、数据导出模板

```java
/**
 * 订单处理模板抽象类
 * 参考：电商订单处理标准流程
 */
public abstract class OrderProcessTemplate {
    
    /**
     * 模板方法 - 定义订单处理的标准流程
     */
    public final OrderProcessResult processOrder(Order order) {
        System.out.println("=== 开始处理订单: " + order.getOrderId() + " ===");
        
        try {
            // 1. 验证订单
            if (!validateOrder(order)) {
                return OrderProcessResult.failure("订单验证失败");
            }
            
            // 2. 检查库存（钩子方法，子类可选择是否实现）
            if (needInventoryCheck() && !checkInventory(order)) {
                return OrderProcessResult.failure("库存检查失败");
            }
            
            // 3. 计算价格
            BigDecimal finalPrice = calculatePrice(order);
            order.setFinalAmount(finalPrice);
            
            // 4. 处理支付
            if (!processPayment(order)) {
                return OrderProcessResult.failure("支付处理失败");
            }
            
            // 5. 更新库存
            if (!updateInventory(order)) {
                // 支付成功但库存更新失败，需要回滚
                rollbackPayment(order);
                return OrderProcessResult.failure("库存更新失败");
            }
            
            // 6. 创建发货任务（钩子方法）
            if (needShipping()) {
                createShippingTask(order);
            }
            
            // 7. 发送通知
            sendNotification(order);
            
            // 8. 记录日志
            logOrderProcess(order, "SUCCESS");
            
            System.out.println("订单处理完成: " + order.getOrderId());
            return OrderProcessResult.success(order.getOrderId());
            
        } catch (Exception e) {
            // 异常处理
            handleException(order, e);
            logOrderProcess(order, "ERROR: " + e.getMessage());
            return OrderProcessResult.failure("订单处理异常: " + e.getMessage());
        }
    }
    
    // 抽象方法 - 子类必须实现
    protected abstract boolean validateOrder(Order order);
    protected abstract BigDecimal calculatePrice(Order order);
    protected abstract boolean processPayment(Order order);
    protected abstract boolean updateInventory(Order order);
    protected abstract void sendNotification(Order order);
    
    // 具体方法 - 通用实现
    protected boolean checkInventory(Order order) {
        System.out.println("检查库存...");
        for (OrderItem item : order.getItems()) {
            // 模拟库存检查
            int availableStock = getAvailableStock(item.getProductId());
            if (availableStock < item.getQuantity()) {
                System.out.println("库存不足: " + item.getProductId() + 
                    ", 需要: " + item.getQuantity() + ", 可用: " + availableStock);
                return false;
            }
        }
        System.out.println("库存检查通过");
        return true;
    }
    
    protected void rollbackPayment(Order order) {
        System.out.println("回滚支付: " + order.getPaymentId());
        // 实际实现中需要调用支付系统的退款接口
    }
    
    protected void createShippingTask(Order order) {
        System.out.println("创建发货任务");
        String trackingNumber = "TN" + System.currentTimeMillis();
        order.setTrackingNumber(trackingNumber);
        System.out.println("物流单号: " + trackingNumber);
    }
    
    protected void logOrderProcess(Order order, String result) {
        System.out.println(String.format("日志记录: [%s] 订单:%s, 结果:%s, 时间:%s",
            this.getClass().getSimpleName(),
            order.getOrderId(),
            result,
            LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"))
        ));
    }
    
    protected void handleException(Order order, Exception e) {
        System.out.println("处理异常: " + e.getMessage());
        // 可以在这里实现通用的异常处理逻辑
    }
    
    // 钩子方法 - 子类可以重写来改变算法行为
    protected boolean needInventoryCheck() {
        return true; // 默认需要库存检查
    }
    
    protected boolean needShipping() {
        return true; // 默认需要发货
    }
    
    // 辅助方法
    private int getAvailableStock(String productId) {
        // 模拟从库存系统获取数据
        Map<String, Integer> mockStock = new HashMap<>();
        mockStock.put("P001", 100);
        mockStock.put("P002", 50);
        mockStock.put("P003", 200);
        return mockStock.getOrDefault(productId, 0);
    }
}

/**
 * 具体实现 - 普通商品订单处理
 */
public class RegularOrderProcessor extends OrderProcessTemplate {
    
    @Override
    protected boolean validateOrder(Order order) {
        System.out.println("[普通订单] 验证订单信息");
        
        // 基础验证
        if (order.getOrderId() == null || order.getOrderId().isEmpty()) {
            System.out.println("订单ID不能为空");
            return false;
        }
        
        if (order.getItems() == null || order.getItems().isEmpty()) {
            System.out.println("订单商品不能为空");
            return false;
        }
        
        if (order.getUserId() == null || order.getUserId().isEmpty()) {
            System.out.println("用户ID不能为空");
            return false;
        }
        
        System.out.println("订单验证通过");
        return true;
    }
    
    @Override
    protected BigDecimal calculatePrice(Order order) {
        System.out.println("[普通订单] 计算价格");
        
        // 计算商品总价
        BigDecimal total = order.getItems().stream()
            .map(item -> item.getPrice().multiply(BigDecimal.valueOf(item.getQuantity())))
            .reduce(BigDecimal.ZERO, BigDecimal::add);
        
        // 应用优惠券
        BigDecimal discount = BigDecimal.ZERO;
        if (order.getCouponId() != null) {
            discount = new BigDecimal("20.00"); // 模拟优惠券减免
            System.out.println("应用优惠券减免: ¥" + discount);
        }
        
        BigDecimal finalPrice = total.subtract(discount);
        System.out.println("价格计算完成: 原价¥" + total + ", 最终价格¥" + finalPrice);
        
        return finalPrice;
    }
    
    @Override
    protected boolean processPayment(Order order) {
        System.out.println("[普通订单] 处理支付");
        
        // 模拟支付处理
        String paymentId = "PAY" + System.currentTimeMillis();
        order.setPaymentId(paymentId);
        
        // 模拟90%支付成功率
        boolean success = Math.random() > 0.1;
        
        if (success) {
            System.out.println("支付成功, 支付ID: " + paymentId);
            order.setStatus(OrderStatus.PAID);
        } else {
            System.out.println("支付失败");
        }
        
        return success;
    }
    
    @Override
    protected boolean updateInventory(Order order) {
        System.out.println("[普通订单] 更新库存");
        
        for (OrderItem item : order.getItems()) {
            System.out.println("扣减库存: " + item.getProductId() + " -" + item.getQuantity());
        }
        
        System.out.println("库存更新完成");
        return true;
    }
    
    @Override
    protected void sendNotification(Order order) {
        System.out.println("[普通订单] 发送通知");
        System.out.println("发送订单确认短信给用户: " + order.getUserId());
        System.out.println("发送订单确认邮件");
    }
}

/**
 * 具体实现 - VIP订单处理
 */
public class VipOrderProcessor extends OrderProcessTemplate {
    
    @Override
    protected boolean validateOrder(Order order) {
        System.out.println("[VIP订单] 验证订单信息");
        
        // VIP订单需要额外验证
        if (!isVipUser(order.getUserId())) {
            System.out.println("非VIP用户不能使用VIP订单处理");
            return false;
        }
        
        // 基础验证
        if (order.getOrderId() == null || order.getItems() == null || order.getItems().isEmpty()) {
            System.out.println("订单信息不完整");
            return false;
        }
        
        System.out.println("VIP订单验证通过");
        return true;
    }
    
    @Override
    protected BigDecimal calculatePrice(Order order) {
        System.out.println("[VIP订单] 计算价格（含VIP折扣）");
        
        BigDecimal total = order.getItems().stream()
            .map(item -> item.getPrice().multiply(BigDecimal.valueOf(item.getQuantity())))
            .reduce(BigDecimal.ZERO, BigDecimal::add);
        
        // VIP用户享受95折
        BigDecimal vipDiscount = total.multiply(new BigDecimal("0.05"));
        System.out.println("VIP折扣: ¥" + vipDiscount);
        
        // 应用优惠券
        BigDecimal couponDiscount = BigDecimal.ZERO;
        if (order.getCouponId() != null) {
            couponDiscount = new BigDecimal("30.00"); // VIP用户优惠券减免更多
            System.out.println("VIP优惠券减免: ¥" + couponDiscount);
        }
        
        BigDecimal finalPrice = total.subtract(vipDiscount).subtract(couponDiscount);
        System.out.println("VIP价格计算完成: 原价¥" + total + ", 最终价格¥" + finalPrice);
        
        return finalPrice;
    }
    
    @Override
    protected boolean processPayment(Order order) {
        System.out.println("[VIP订单] VIP快速支付通道");
        
        String paymentId = "VIP_PAY" + System.currentTimeMillis();
        order.setPaymentId(paymentId);
        
        // VIP用户支付成功率更高（95%）
        boolean success = Math.random() > 0.05;
        
        if (success) {
            System.out.println("VIP支付成功, 支付ID: " + paymentId);
            order.setStatus(OrderStatus.PAID);
        } else {
            System.out.println("VIP支付失败");
        }
        
        return success;
    }
    
    @Override
    protected boolean updateInventory(Order order) {
        System.out.println("[VIP订单] 优先扣减库存");
        
        // VIP订单优先处理库存
        for (OrderItem item : order.getItems()) {
            System.out.println("优先扣减库存: " + item.getProductId() + " -" + item.getQuantity());
        }
        
        System.out.println("VIP库存更新完成");
        return true;
    }
    
    @Override
    protected void sendNotification(Order order) {
        System.out.println("[VIP订单] 发送VIP专属通知");
        System.out.println("发送VIP订单确认短信给用户: " + order.getUserId());
        System.out.println("发送VIP订单确认邮件");
        System.out.println("安排VIP专属客服跟进");
    }
    
    @Override
    protected void createShippingTask(Order order) {
        System.out.println("创建VIP优先发货任务");
        String trackingNumber = "VIP_TN" + System.currentTimeMillis();
        order.setTrackingNumber(trackingNumber);
        System.out.println("VIP物流单号: " + trackingNumber);
        System.out.println("标记为优先发货");
    }
    
    private boolean isVipUser(String userId) {
        // 模拟VIP用户检查
        return userId.startsWith("VIP") || userId.equals("U001");
    }
}

/**
 * 具体实现 - 虚拟商品订单处理
 */
public class VirtualOrderProcessor extends OrderProcessTemplate {
    
    @Override
    protected boolean validateOrder(Order order) {
        System.out.println("[虚拟商品] 验证订单信息");
        
        // 验证是否都是虚拟商品
        for (OrderItem item : order.getItems()) {
            if (!isVirtualProduct(item.getProductId())) {
                System.out.println("存在非虚拟商品，不能使用虚拟商品订单处理");
                return false;
            }
        }
        
        System.out.println("虚拟商品订单验证通过");
        return true;
    }
    
    @Override
    protected BigDecimal calculatePrice(Order order) {
        System.out.println("[虚拟商品] 计算价格");
        
        BigDecimal total = order.getItems().stream()
            .map(item -> item.getPrice().multiply(BigDecimal.valueOf(item.getQuantity())))
            .reduce(BigDecimal.ZERO, BigDecimal::add);
        
        System.out.println("虚拟商品价格计算完成: ¥" + total);
        return total;
    }
    
    @Override
    protected boolean processPayment(Order order) {
        System.out.println("[虚拟商品] 处理支付");
        
        String paymentId = "VIRTUAL_PAY" + System.currentTimeMillis();
        order.setPaymentId(paymentId);
        
        // 虚拟商品支付通常更稳定
        boolean success = Math.random() > 0.05;
        
        if (success) {
            System.out.println("虚拟商品支付成功, 支付ID: " + paymentId);
            order.setStatus(OrderStatus.PAID);
        }
        
        return success;
    }
    
    @Override
    protected boolean updateInventory(Order order) {
        System.out.println("[虚拟商品] 更新虚拟库存");
        
        for (OrderItem item : order.getItems()) {
            System.out.println("扣减虚拟库存: " + item.getProductId() + " -" + item.getQuantity());
        }
        
        return true;
    }
    
    @Override
    protected void sendNotification(Order order) {
        System.out.println("[虚拟商品] 发送激活码/下载链接");
        System.out.println("发送虚拟商品交付短信给用户: " + order.getUserId());
        System.out.println("发送激活码邮件");
        
        // 虚拟商品立即交付
        for (OrderItem item : order.getItems()) {
            String activationCode = generateActivationCode();
            System.out.println("商品 " + item.getProductId() + " 激活码: " + activationCode);
        }
    }
    
    // 重写钩子方法 - 虚拟商品不需要库存检查和发货
    @Override
    protected boolean needInventoryCheck() {
        return false; // 虚拟商品不需要物理库存检查
    }
    
    @Override
    protected boolean needShipping() {
        return false; // 虚拟商品不需要发货
    }
    
    private boolean isVirtualProduct(String productId) {
        // 模拟虚拟商品检查
        return productId.startsWith("V");
    }
    
    private String generateActivationCode() {
        return "ACT" + System.currentTimeMillis() + (int)(Math.random() * 1000);
    }
}

/**
 * 订单处理结果
 */
public class OrderProcessResult {
    private boolean success;
    private String orderId;
    private String message;
    private Map<String, Object> data;
    
    private OrderProcessResult(boolean success, String orderId, String message) {
        this.success = success;
        this.orderId = orderId;
        this.message = message;
        this.data = new HashMap<>();
    }
    
    public static OrderProcessResult success(String orderId) {
        return new OrderProcessResult(true, orderId, "订单处理成功");
    }
    
    public static OrderProcessResult failure(String message) {
        return new OrderProcessResult(false, null, message);
    }
    
    // Getters
    public boolean isSuccess() { return success; }
    public String getOrderId() { return orderId; }
    public String getMessage() { return message; }
    public Map<String, Object> getData() { return data; }
    public void setData(String key, Object value) { data.put(key, value); }
}

/**
 * 订单处理工厂
 */
public class OrderProcessorFactory {
    
    public static OrderProcessTemplate createProcessor(Order order) {
        // 根据订单类型选择不同的处理器
        if (isVipOrder(order)) {
            return new VipOrderProcessor();
        } else if (isVirtualOrder(order)) {
            return new VirtualOrderProcessor();
        } else {
            return new RegularOrderProcessor();
        }
    }
    
    private static boolean isVipOrder(Order order) {
        return order.getUserId().startsWith("VIP") || "VIP".equals(order.getOrderType());
    }
    
    private static boolean isVirtualOrder(Order order) {
        // 检查是否所有商品都是虚拟商品
        return order.getItems().stream()
            .allMatch(item -> item.getProductId().startsWith("V"));
    }
}

/**
 * 模板方法模式使用示例
 */
public class TemplateMethodPatternExample {
    
    public void demonstrateTemplate() {
        System.out.println("=== 模板方法模式演示 ===\n");
        
        // 1. 普通订单处理
        demonstrateRegularOrder();
        
        System.out.println("\n" + "=".repeat(60) + "\n");
        
        // 2. VIP订单处理
        demonstrateVipOrder();
        
        System.out.println("\n" + "=".repeat(60) + "\n");
        
        // 3. 虚拟商品订单处理
        demonstrateVirtualOrder();
        
        System.out.println("\n" + "=".repeat(60) + "\n");
        
        // 4. 使用工厂自动选择处理器
        demonstrateAutoSelection();
    }
    
    private void demonstrateRegularOrder() {
        System.out.println("1. 普通订单处理演示:");
        
        Order order = createRegularOrder();
        OrderProcessTemplate processor = new RegularOrderProcessor();
        OrderProcessResult result = processor.processOrder(order);
        
        System.out.println("处理结果: " + (result.isSuccess() ? "成功" : "失败"));
        if (!result.isSuccess()) {
            System.out.println("失败原因: " + result.getMessage());
        }
    }
    
    private void demonstrateVipOrder() {
        System.out.println("2. VIP订单处理演示:");
        
        Order order = createVipOrder();
        OrderProcessTemplate processor = new VipOrderProcessor();
        OrderProcessResult result = processor.processOrder(order);
        
        System.out.println("处理结果: " + (result.isSuccess() ? "成功" : "失败"));
    }
    
    private void demonstrateVirtualOrder() {
        System.out.println("3. 虚拟商品订单处理演示:");
        
        Order order = createVirtualOrder();
        OrderProcessTemplate processor = new VirtualOrderProcessor();
        OrderProcessResult result = processor.processOrder(order);
        
        System.out.println("处理结果: " + (result.isSuccess() ? "成功" : "失败"));
    }
    
    private void demonstrateAutoSelection() {
        System.out.println("4. 自动选择处理器演示:");
        
        // 创建不同类型的订单
        Order[] orders = {
            createRegularOrder(),
            createVipOrder(),
            createVirtualOrder()
        };
        
        for (Order order : orders) {
            System.out.println("\n处理订单: " + order.getOrderId());
            OrderProcessTemplate processor = OrderProcessorFactory.createProcessor(order);
            System.out.println("选择处理器: " + processor.getClass().getSimpleName());
            
            OrderProcessResult result = processor.processOrder(order);
            System.out.println("处理结果: " + (result.isSuccess() ? "成功" : "失败"));
        }
    }
    
    private Order createRegularOrder() {
        Order order = new Order("ORD001", "U001");
        order.addItem(new OrderItem("P001", "iPhone 14", new BigDecimal("5999"), 1));
        order.addItem(new OrderItem("P002", "AirPods Pro", new BigDecimal("1899"), 1));
        order.setCouponId("COUPON001");
        return order;
    }
    
    private Order createVipOrder() {
        Order order = new Order("VIP001", "VIP001");
        order.addItem(new OrderItem("P003", "MacBook Pro", new BigDecimal("15999"), 1));
        order.setCouponId("VIP_COUPON001");
        order.setOrderType("VIP");
        return order;
    }
    
    private Order createVirtualOrder() {
        Order order = new Order("VIRTUAL001", "U003");
        order.addItem(new OrderItem("V001", "电子书", new BigDecimal("29.90"), 1));
        order.addItem(new OrderItem("V002", "软件许可", new BigDecimal("199.00"), 1));
        return order;
    }
}

### 23. 访问者模式 (Visitor Pattern)

**场景**: 订单数据分析、商品统计报表、价格计算访问

```java
/**
 * 订单元素接口
 * 参考：访问者模式在报表系统中的应用
 */
public interface OrderElement {
    void accept(OrderVisitor visitor);
}

/**
 * 订单访问者接口
 */
public interface OrderVisitor {
    void visitOrder(Order order);
    void visitOrderItem(OrderItem orderItem);
    void visitCustomer(Customer customer);
    void visitPayment(Payment payment);
    void visitShipment(Shipment shipment);
}

/**
 * 订单类 - 实现OrderElement
 */
public class Order implements OrderElement {
    private String orderId;
    private String customerId;
    private List<OrderItem> items;
    private Payment payment;
    private Shipment shipment;
    private OrderStatus status;
    private LocalDateTime createTime;
    private BigDecimal totalAmount;
    
    public Order(String orderId, String customerId) {
        this.orderId = orderId;
        this.customerId = customerId;
        this.items = new ArrayList<>();
        this.createTime = LocalDateTime.now();
        this.status = OrderStatus.CREATED;
    }
    
    @Override
    public void accept(OrderVisitor visitor) {
        visitor.visitOrder(this);
        
        // 访问订单项
        for (OrderItem item : items) {
            item.accept(visitor);
        }
        
        // 访问支付信息
        if (payment != null) {
            payment.accept(visitor);
        }
        
        // 访问发货信息
        if (shipment != null) {
            shipment.accept(visitor);
        }
    }
    
    // 业务方法
    public void addItem(OrderItem item) {
        items.add(item);
        calculateTotalAmount();
    }
    
    private void calculateTotalAmount() {
        totalAmount = items.stream()
            .map(item -> item.getPrice().multiply(BigDecimal.valueOf(item.getQuantity())))
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
    
    // Getters and Setters
    public String getOrderId() { return orderId; }
    public String getCustomerId() { return customerId; }
    public List<OrderItem> getItems() { return items; }
    public Payment getPayment() { return payment; }
    public void setPayment(Payment payment) { this.payment = payment; }
    public Shipment getShipment() { return shipment; }
    public void setShipment(Shipment shipment) { this.shipment = shipment; }
    public OrderStatus getStatus() { return status; }
    public void setStatus(OrderStatus status) { this.status = status; }
    public LocalDateTime getCreateTime() { return createTime; }
    public BigDecimal getTotalAmount() { return totalAmount; }
}

/**
 * 订单项类 - 实现OrderElement
 */
public class OrderItem implements OrderElement {
    private String productId;
    private String productName;
    private String category;
    private BigDecimal price;
    private int quantity;
    private String supplierId;
    
    public OrderItem(String productId, String productName, String category, 
                    BigDecimal price, int quantity) {
        this.productId = productId;
        this.productName = productName;
        this.category = category;
        this.price = price;
        this.quantity = quantity;
    }
    
    @Override
    public void accept(OrderVisitor visitor) {
        visitor.visitOrderItem(this);
    }
    
    public BigDecimal getTotalPrice() {
        return price.multiply(BigDecimal.valueOf(quantity));
    }
    
    // Getters and Setters
    public String getProductId() { return productId; }
    public String getProductName() { return productName; }
    public String getCategory() { return category; }
    public BigDecimal getPrice() { return price; }
    public int getQuantity() { return quantity; }
    public String getSupplierId() { return supplierId; }
    public void setSupplierId(String supplierId) { this.supplierId = supplierId; }
}

/**
 * 客户类 - 实现OrderElement
 */
public class Customer implements OrderElement {
    private String customerId;
    private String name;
    private String email;
    private String phone;
    private String memberLevel;
    private LocalDateTime registerTime;
    private String region;
    
    public Customer(String customerId, String name, String memberLevel, String region) {
        this.customerId = customerId;
        this.name = name;
        this.memberLevel = memberLevel;
        this.region = region;
        this.registerTime = LocalDateTime.now();
    }
    
    @Override
    public void accept(OrderVisitor visitor) {
        visitor.visitCustomer(this);
    }
    
    // Getters
    public String getCustomerId() { return customerId; }
    public String getName() { return name; }
    public String getEmail() { return email; }
    public String getPhone() { return phone; }
    public String getMemberLevel() { return memberLevel; }
    public LocalDateTime getRegisterTime() { return registerTime; }
    public String getRegion() { return region; }
}

/**
 * 支付类 - 实现OrderElement
 */
public class Payment implements OrderElement {
    private String paymentId;
    private String paymentMethod;
    private BigDecimal amount;
    private String currency;
    private LocalDateTime paymentTime;
    private String status;
    
    public Payment(String paymentId, String paymentMethod, BigDecimal amount) {
        this.paymentId = paymentId;
        this.paymentMethod = paymentMethod;
        this.amount = amount;
        this.currency = "CNY";
        this.paymentTime = LocalDateTime.now();
        this.status = "SUCCESS";
    }
    
    @Override
    public void accept(OrderVisitor visitor) {
        visitor.visitPayment(this);
    }
    
    // Getters
    public String getPaymentId() { return paymentId; }
    public String getPaymentMethod() { return paymentMethod; }
    public BigDecimal getAmount() { return amount; }
    public String getCurrency() { return currency; }
    public LocalDateTime getPaymentTime() { return paymentTime; }
    public String getStatus() { return status; }
}

/**
 * 发货类 - 实现OrderElement
 */
public class Shipment implements OrderElement {
    private String trackingNumber;
    private String shippingCompany;
    private String origin;
    private String destination;
    private LocalDateTime shipTime;
    private String status;
    private BigDecimal shippingCost;
    
    public Shipment(String trackingNumber, String shippingCompany, 
                   String origin, String destination, BigDecimal shippingCost) {
        this.trackingNumber = trackingNumber;
        this.shippingCompany = shippingCompany;
        this.origin = origin;
        this.destination = destination;
        this.shippingCost = shippingCost;
        this.shipTime = LocalDateTime.now();
        this.status = "SHIPPED";
    }
    
    @Override
    public void accept(OrderVisitor visitor) {
        visitor.visitShipment(this);
    }
    
    // Getters
    public String getTrackingNumber() { return trackingNumber; }
    public String getShippingCompany() { return shippingCompany; }
    public String getOrigin() { return origin; }
    public String getDestination() { return destination; }
    public LocalDateTime getShipTime() { return shipTime; }
    public String getStatus() { return status; }
    public BigDecimal getShippingCost() { return shippingCost; }
}

/**
 * 具体访问者 - 订单统计访问者
 */
public class OrderStatisticsVisitor implements OrderVisitor {
    private int totalOrders = 0;
    private BigDecimal totalRevenue = BigDecimal.ZERO;
    private int totalItems = 0;
    private Map<String, Integer> categoryStats = new HashMap<>();
    private Map<String, Integer> paymentMethodStats = new HashMap<>();
    private Map<String, Integer> regionStats = new HashMap<>();
    private Map<String, BigDecimal> supplierStats = new HashMap<>();
    
    @Override
    public void visitOrder(Order order) {
        totalOrders++;
        totalRevenue = totalRevenue.add(order.getTotalAmount());
        System.out.println("访问订单: " + order.getOrderId() + ", 金额: ¥" + order.getTotalAmount());
    }
    
    @Override
    public void visitOrderItem(OrderItem item) {
        totalItems++;
        
        // 统计商品类别
        categoryStats.merge(item.getCategory(), item.getQuantity(), Integer::sum);
        
        // 统计供应商销售额
        if (item.getSupplierId() != null) {
            supplierStats.merge(item.getSupplierId(), item.getTotalPrice(), BigDecimal::add);
        }
        
        System.out.println("  商品: " + item.getProductName() + 
            ", 类别: " + item.getCategory() + 
            ", 数量: " + item.getQuantity());
    }
    
    @Override
    public void visitCustomer(Customer customer) {
        // 统计地区分布
        regionStats.merge(customer.getRegion(), 1, Integer::sum);
        
        System.out.println("  客户: " + customer.getName() + 
            ", 等级: " + customer.getMemberLevel() + 
            ", 地区: " + customer.getRegion());
    }
    
    @Override
    public void visitPayment(Payment payment) {
        // 统计支付方式
        paymentMethodStats.merge(payment.getPaymentMethod(), 1, Integer::sum);
        
        System.out.println("  支付: " + payment.getPaymentMethod() + 
            ", 金额: ¥" + payment.getAmount());
    }
    
    @Override
    public void visitShipment(Shipment shipment) {
        System.out.println("  发货: " + shipment.getShippingCompany() + 
            ", 运费: ¥" + shipment.getShippingCost());
    }
    
    /**
     * 打印统计报告
     */
    public void printStatisticsReport() {
        System.out.println("\n=== 订单统计报告 ===");
        System.out.println("总订单数: " + totalOrders);
        System.out.println("总收入: ¥" + totalRevenue);
        System.out.println("总商品数量: " + totalItems);
        
        if (totalOrders > 0) {
            BigDecimal avgOrderValue = totalRevenue.divide(
                BigDecimal.valueOf(totalOrders), 2, RoundingMode.HALF_UP);
            System.out.println("平均订单价值: ¥" + avgOrderValue);
        }
        
        System.out.println("\n商品类别统计:");
        categoryStats.forEach((category, count) -> 
            System.out.println("  " + category + ": " + count + "件"));
        
        System.out.println("\n支付方式统计:");
        paymentMethodStats.forEach((method, count) -> 
            System.out.println("  " + method + ": " + count + "笔"));
        
        System.out.println("\n地区分布统计:");
        regionStats.forEach((region, count) -> 
            System.out.println("  " + region + ": " + count + "个客户"));
        
        if (!supplierStats.isEmpty()) {
            System.out.println("\n供应商销售统计:");
            supplierStats.forEach((supplier, amount) -> 
                System.out.println("  " + supplier + ": ¥" + amount));
        }
    }
    
    // Getters for accessing statistics
    public int getTotalOrders() { return totalOrders; }
    public BigDecimal getTotalRevenue() { return totalRevenue; }
    public Map<String, Integer> getCategoryStats() { return new HashMap<>(categoryStats); }
}

/**
 * 具体访问者 - 订单验证访问者
 */
public class OrderValidationVisitor implements OrderVisitor {
    private List<String> validationErrors = new ArrayList<>();
    private boolean isValid = true;
    
    @Override
    public void visitOrder(Order order) {
        System.out.println("验证订单: " + order.getOrderId());
        
        if (order.getOrderId() == null || order.getOrderId().isEmpty()) {
            addError("订单ID不能为空");
        }
        
        if (order.getCustomerId() == null || order.getCustomerId().isEmpty()) {
            addError("客户ID不能为空");
        }
        
        if (order.getItems().isEmpty()) {
            addError("订单商品不能为空");
        }
        
        if (order.getTotalAmount().compareTo(BigDecimal.ZERO) <= 0) {
            addError("订单金额必须大于0");
        }
    }
    
    @Override
    public void visitOrderItem(OrderItem item) {
        if (item.getQuantity() <= 0) {
            addError("商品数量必须大于0: " + item.getProductName());
        }
        
        if (item.getPrice().compareTo(BigDecimal.ZERO) <= 0) {
            addError("商品价格必须大于0: " + item.getProductName());
        }
        
        if (item.getProductName() == null || item.getProductName().isEmpty()) {
            addError("商品名称不能为空");
        }
    }
    
    @Override
    public void visitCustomer(Customer customer) {
        if (customer.getName() == null || customer.getName().isEmpty()) {
            addError("客户姓名不能为空");
        }
        
        if (customer.getRegion() == null || customer.getRegion().isEmpty()) {
            addError("客户地区不能为空");
        }
    }
    
    @Override
    public void visitPayment(Payment payment) {
        if (payment.getAmount().compareTo(BigDecimal.ZERO) <= 0) {
            addError("支付金额必须大于0");
        }
        
        if (!"SUCCESS".equals(payment.getStatus())) {
            addError("支付状态异常: " + payment.getStatus());
        }
    }
    
    @Override
    public void visitShipment(Shipment shipment) {
        if (shipment.getTrackingNumber() == null || shipment.getTrackingNumber().isEmpty()) {
            addError("物流单号不能为空");
        }
        
        if (shipment.getDestination() == null || shipment.getDestination().isEmpty()) {
            addError("收货地址不能为空");
        }
    }
    
    private void addError(String error) {
        validationErrors.add(error);
        isValid = false;
        System.out.println("  验证错误: " + error);
    }
    
    public boolean isValid() {
        return isValid;
    }
    
    public List<String> getValidationErrors() {
        return new ArrayList<>(validationErrors);
    }
    
    public void printValidationResult() {
        System.out.println("\n=== 订单验证结果 ===");
        if (isValid) {
            System.out.println("订单验证通过");
        } else {
            System.out.println("订单验证失败，错误如下:");
            for (int i = 0; i < validationErrors.size(); i++) {
                System.out.println((i + 1) + ". " + validationErrors.get(i));
            }
        }
    }
}

/**
 * 具体访问者 - 订单导出访问者
 */
public class OrderExportVisitor implements OrderVisitor {
    private StringBuilder csvContent = new StringBuilder();
    private boolean headerWritten = false;
    
    @Override
    public void visitOrder(Order order) {
        if (!headerWritten) {
            // 写入CSV头部
            csvContent.append("订单ID,客户ID,订单状态,创建时间,总金额,商品名称,商品类别,商品数量,商品单价,支付方式,物流公司\n");
            headerWritten = true;
        }
    }
    
    @Override
    public void visitOrderItem(OrderItem item) {
        // 每个订单项一行数据
        // 这里需要从上下文获取订单信息，简化处理
    }
    
    @Override
    public void visitCustomer(Customer customer) {
        // 记录客户信息用于导出
    }
    
    @Override
    public void visitPayment(Payment payment) {
        // 记录支付信息用于导出
    }
    
    @Override
    public void visitShipment(Shipment shipment) {
        // 记录物流信息用于导出
    }
    
    /**
     * 导出订单数据
     */
    public void exportOrder(Order order) {
        // 重新访问订单以构建完整的导出数据
        for (OrderItem item : order.getItems()) {
            csvContent.append(order.getOrderId()).append(",")
                     .append(order.getCustomerId()).append(",")
                     .append(order.getStatus()).append(",")
                     .append(order.getCreateTime().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"))).append(",")
                     .append(order.getTotalAmount()).append(",")
                     .append(item.getProductName()).append(",")
                     .append(item.getCategory()).append(",")
                     .append(item.getQuantity()).append(",")
                     .append(item.getPrice()).append(",")
                     .append(order.getPayment() != null ? order.getPayment().getPaymentMethod() : "").append(",")
                     .append(order.getShipment() != null ? order.getShipment().getShippingCompany() : "")
                     .append("\n");
        }
    }
    
    public String getExportedData() {
        return csvContent.toString();
    }
    
    public void saveToFile(String filename) {
        System.out.println("导出订单数据到文件: " + filename);
        System.out.println("导出内容预览:");
        System.out.println(csvContent.toString());
    }
}

/**
 * 订单管理器 - 管理订单集合
 */
public class OrderManager {
    private List<Order> orders = new ArrayList<>();
    private Map<String, Customer> customers = new HashMap<>();
    
    public void addOrder(Order order) {
        orders.add(order);
    }
    
    public void addCustomer(Customer customer) {
        customers.put(customer.getCustomerId(), customer);
    }
    
    public Customer getCustomer(String customerId) {
        return customers.get(customerId);
    }
    
    /**
     * 接受访问者访问所有订单
     */
    public void acceptVisitor(OrderVisitor visitor) {
        for (Order order : orders) {
            // 访问客户信息
            Customer customer = customers.get(order.getCustomerId());
            if (customer != null) {
                customer.accept(visitor);
            }
            
            // 访问订单
            order.accept(visitor);
        }
    }
    
    public List<Order> getOrders() {
        return new ArrayList<>(orders);
    }
}

/**
 * 访问者模式使用示例
 */
public class VisitorPatternExample {
    
    public void demonstrateVisitor() {
        System.out.println("=== 访问者模式演示 ===\n");
        
        // 创建订单管理器和测试数据
        OrderManager orderManager = createTestData();
        
        // 1. 统计访问者演示
        demonstrateStatisticsVisitor(orderManager);
        
        System.out.println("\n" + "=".repeat(60) + "\n");
        
        // 2. 验证访问者演示
        demonstrateValidationVisitor(orderManager);
        
        System.out.println("\n" + "=".repeat(60) + "\n");
        
        // 3. 导出访问者演示
        demonstrateExportVisitor(orderManager);
    }
    
    private OrderManager createTestData() {
        OrderManager manager = new OrderManager();
        
        // 创建客户
        Customer customer1 = new Customer("C001", "张三", "VIP", "北京");
        Customer customer2 = new Customer("C002", "李四", "普通", "上海");
        Customer customer3 = new Customer("C003", "王五", "金牌", "广州");
        
        manager.addCustomer(customer1);
        manager.addCustomer(customer2);
        manager.addCustomer(customer3);
        
        // 创建订单1
        Order order1 = new Order("ORD001", "C001");
        order1.addItem(new OrderItem("P001", "iPhone 14", "数码电子", new BigDecimal("5999"), 1));
        order1.addItem(new OrderItem("P002", "AirPods Pro", "数码电子", new BigDecimal("1899"), 1));
        order1.setPayment(new Payment("PAY001", "支付宝", order1.getTotalAmount()));
        order1.setShipment(new Shipment("SF001", "顺丰快递", "深圳", "北京", new BigDecimal("15")));
        order1.setStatus(OrderStatus.DELIVERED);
        
        // 为订单项设置供应商
        order1.getItems().get(0).setSupplierId("SUP001");
        order1.getItems().get(1).setSupplierId("SUP001");
        
        // 创建订单2
        Order order2 = new Order("ORD002", "C002");
        order2.addItem(new OrderItem("P003", "Nike运动鞋", "服装鞋帽", new BigDecimal("699"), 2));
        order2.addItem(new OrderItem("P004", "运动T恤", "服装鞋帽", new BigDecimal("199"), 3));
        order2.setPayment(new Payment("PAY002", "微信支付", order2.getTotalAmount()));
        order2.setShipment(new Shipment("YTO001", "圆通快递", "广州", "上海", new BigDecimal("12")));
        order2.setStatus(OrderStatus.SHIPPED);
        
        // 为订单项设置供应商
        order2.getItems().get(0).setSupplierId("SUP002");
        order2.getItems().get(1).setSupplierId("SUP002");
        
        // 创建订单3
        Order order3 = new Order("ORD003", "C003");
        order3.addItem(new OrderItem("P005", "《Java编程思想》", "图书音像", new BigDecimal("89"), 1));
        order3.addItem(new OrderItem("P006", "《设计模式》", "图书音像", new BigDecimal("79"), 1));
        order3.setPayment(new Payment("PAY003", "银行卡", order3.getTotalAmount()));
        order3.setStatus(OrderStatus.PAID);
        
        // 为订单项设置供应商
        order3.getItems().get(0).setSupplierId("SUP003");
        order3.getItems().get(1).setSupplierId("SUP003");
        
        manager.addOrder(order1);
        manager.addOrder(order2);
        manager.addOrder(order3);
        
        return manager;
    }
    
    private void demonstrateStatisticsVisitor(OrderManager manager) {
        System.out.println("1. 订单统计访问者演示:");
        
        OrderStatisticsVisitor statisticsVisitor = new OrderStatisticsVisitor();
        manager.acceptVisitor(statisticsVisitor);
        
        statisticsVisitor.printStatisticsReport();
    }
    
    private void demonstrateValidationVisitor(OrderManager manager) {
        System.out.println("2. 订单验证访问者演示:");
        
        OrderValidationVisitor validationVisitor = new OrderValidationVisitor();
        manager.acceptVisitor(validationVisitor);
        
        validationVisitor.printValidationResult();
        
        // 测试无效订单
        System.out.println("\n测试无效订单:");
        Order invalidOrder = new Order("", ""); // 空订单ID和客户ID
        OrderValidationVisitor invalidValidator = new OrderValidationVisitor();
        invalidOrder.accept(invalidValidator);
        invalidValidator.printValidationResult();
    }
    
    private void demonstrateExportVisitor(OrderManager manager) {
        System.out.println("3. 订单导出访问者演示:");
        
        OrderExportVisitor exportVisitor = new OrderExportVisitor();
        
        // 导出所有订单
        for (Order order : manager.getOrders()) {
            exportVisitor.exportOrder(order);
        }
        
        // 保存到文件
        exportVisitor.saveToFile("orders_export.csv");
    }
}

---

## 总结

这个Java设计模式教程涵盖了所有23种经典设计模式，每种模式都结合了真实的电商场景：

### 创建型模式 (5种)
1. **单例模式** - 订单号生成器、连接池管理
2. **工厂方法模式** - 支付服务创建
3. **抽象工厂模式** - 多平台电商适配
4. **建造者模式** - 复杂订单构建
5. **原型模式** - 商品模板复制

### 结构型模式 (7种)
6. **适配器模式** - 第三方支付接口适配
7. **桥接模式** - 跨平台消息推送
8. **组合模式** - 商品分类树
9. **装饰器模式** - 价格计算链
10. **外观模式** - 订单处理统一接口
11. **享元模式** - 商品属性缓存
12. **代理模式** - 权限控制和缓存

### 行为型模式 (11种)
13. **责任链模式** - 订单审核流程
14. **命令模式** - 订单操作撤销/重做
15. **解释器模式** - 优惠规则表达式
16. **迭代器模式** - 订单列表遍历
17. **中介者模式** - 订单流程协调
18. **备忘录模式** - 订单状态回滚
19. **观察者模式** - 订单状态变更通知
20. **状态模式** - 订单状态流转
21. **策略模式** - 支付策略选择
22. **模板方法模式** - 订单处理流程模板
23. **访问者模式** - 订单数据分析

每个模式都包含：
- 真实的大厂应用场景
- 完整的Java代码实现
- 详细的注释说明
- 实际的使用示例
- 符合企业级开发标准的代码规范

这些设计模式是软件架构的基础，掌握它们对于编写高质量、可维护的企业级Java应用至关重要。
```

<function_calls>
<invoke name="TodoWrite">
<parameter name="todos">[{"id": "design-patterns-tutorial", "content": "创建完整的Java设计模式教程文档，包含23种设计模式的电商场景实现", "status": "completed", "priority": "high"}]