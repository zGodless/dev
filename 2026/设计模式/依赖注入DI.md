# ‌依赖注入‌是一种设计模式，用于解决“类与类之间的耦合”问题。
‌传统方式（硬编码）‌：类 A 内部直接 new 出类 B。A 强依赖 B，难以测试和替换。
‌DI 方式（控制反转）‌：类 A 不自己创建 B，而是告诉框架“我需要 B”，框架在运行时把 B “注入”给 A。

# 生命周期管理
    生命周期	场景示例	注意事项
‌Transient‌	无状态工具类、轻量级计算服务	每次请求都新建，开销小但频繁 GC
‌Scoped‌	‌DbContext‌、UnitOfWork、当前用户信息	‌每个 HTTP 请求一个实例‌，请求结束销毁
‌Singleton‌	缓存服务、配置读取器、日志记录器	全局唯一，线程安全要求高

# 捕获依赖错误‌：
禁止‌在 Singleton 服务中注入 Scoped 服务（会导致 Scoped服务变成全局单例，引发数据混乱）。比如全局缓存是Singleton服务，在其中使用了Scoped创建的用户对象，会导致所有用户请求进来都是同一个用户对象。
‌禁止‌在 Scoped 服务中注入 Transient 服务虽不报错，但可能违背预期。

# 为什么 .NET Core 强制使用 DI？
1. 解耦与可测试性
    ‌单元测试容易‌：测试 Controller 时，可以注入 Mock 对象代替真实数据库，无需启动 Web 服务器。
2. 统一管理资源
    容器负责创建和‌销毁‌对象（特别是实现了 `IDisposable接口的对象，如 DbContext），防止内存泄漏。
3. 灵活替换实现
    开发时用 InMemoryDatabase，生产用 SqlServer，只需修改 Program.cs 中的一行注册代码，无需改动业务逻辑。

# 第三方库集成
    大多数 .NET 库（如 Swagger, Serilog, AutoMapper, Consul）都提供 .AddXXX() 扩展方法，本质都是向 DI 容器注册服务：

    builder.Services.AddSwaggerGen(); // 注册 Swagger 服务
    builder.Services.AddAutoMapper(...); // 注册 AutoMapper

# 总结建议
    ‌默认首选 Scoped‌：对于大多数 Web API 业务服务（尤其是涉及数据库的）。
    ‌谨慎使用 Singleton‌：仅用于无状态、线程安全的基础设施。
    ‌尽量少用 Transient‌：除非对象非常轻量且无状态，否则频繁的 GC 会影响高性能场景下的表现。

