# [如何过滤消息](@id How-to-filter-messages)

在此教程中，我们将看到如何基于元数据（level, module等）和日志消息内容来过滤消息。在[如何启用 `@debug` 消息](@ref How-to-enable-debug-messages)和
[发送消息到多个位置](@ref Send-messages-to-multiple-locations)中有如何基于日志级别过滤消息的示例。

在日志系统消息管道中，可以分两个阶段过滤消息。在第一阶段，只有元数据是已知的（level, module, group, 和 id）。特别是消息字符串本身尚未构建，如果创建日志消息的成本很高，则在此阶段进行过滤可能会更有效。例如，在
```julia
@info "The value of some_expensive_call is: $(some_expensive_call(args...))"
```
中，在早期阶段还未发生对 `some_expensive_call` 的调用。当有更多信息可用时，例如完整的消息字符串、文件和行以及任何关键字参数，消息也可以稍后过滤。

### 早期过滤使用 `EarlyFilteredLogger`

可以使用 [LoggingExtras.jl](@ref) 包中的 [`EarlyFilteredLogger`](@ref LoggingExtras.EarlyFilteredLogger) 来完成早期阶段的过滤。`EarlyFilteredLogger` 需要一个谓词函数和一个记录器作为输入参数。如果谓词返回 `true`，则消息被传递给包装的记录器，否则被忽略。谓词函数的唯一输入是一个命名元组，请参阅 [`LoggingExtras.shouldlog_args`](@ref)。

这是一个记录器的示例，它只接受 (i)`Logging.Info` 级别（`Logging.Info <= level < Logging.Warn`）的消息和 (ii) 来自 `Foo` 模块的消息：

```@example filtering
using Logging, LoggingExtras

# Define the Foo module
module Foo
    info() = @info "Information from Foo"
    warn() = @warn "Warning from Foo"
end
using .Foo

begin # hide
# Create the logger
global_logger() = current_logger() # hide
logger = EarlyFilteredLogger(global_logger()) do args
    r = Logging.Info <= args.level < Logging.Warn && args._module === Foo
    return r
end

# Test it
with_logger(logger) do
    @info "Information from Main"
    @warn "Warning from Main"
    Foo.info()
    Foo.warn()
end
end # hide
```

如您所见，唯一没有被过滤器丢弃的是来自 `Foo` 模块的 `@info` 消息！

!!! note
    `MinLevelLogger` 只是 `EarlyFilteredLogger` 的一个特例，它只检查消息的日志级别是否高于配置的级别。

和 [`TeeLogger`](@ref LoggingExtras.TeeLogger) 一起，我们现在可以为消息创建任意路由。这是一个更复杂的例子：

```@example filtering2
using Logging, LoggingExtras

module WebServer end # hide
logger = TeeLogger(
    global_logger(),
    EarlyFilteredLogger(
        args -> args._module === WebServer,
        TeeLogger(
            EarlyFilteredLogger(
                args -> args.level < Logging.Info,
                FileLogger("debug.log"),
            ),
            EarlyFilteredLogger(
                args -> Logging.Info <= args.level < Logging.Warn,
                FileLogger("info.log"),
            ),
            EarlyFilteredLogger(
                args -> Logging.Warn <= args.level,
                FileLogger("warnings_and_errors.log"),
            ),
        )
    ),
)
nothing # hide
```

此记录器将所有消息发送到当前的全局记录器和 `EarlyFilteredLogger`. 然后 `EarlyFilteredLogger` 丢弃所有不是来自 `WebServer` 模块的消息，并将剩余的消息发送到新的 `TeeLogger`. 此`TeeLogger` 会将消息发送到三个 `EarlyFilteredLoggers`，它们只保留特定级别的消息并将这些消息发送到对应于该级别的 `FileLogger`。这意味着，`"debug.log"` 只有来自 `WebServer`（第一个过滤器）并且具有 debug 日志级别的消息。类似地， `"info.log"` 中只会有 info 日志级别的消息，任何更高级别（warn, error）的消息都将位于 `"warnings_and_errors.log"` 中。

### 后期过滤使用 `ActiveFilteredLogger`

如果基于级别、模块、组和 id 的过滤不够，可以使用 [`ActiveFilteredLogger`](@ref LoggingExtras.ActiveFilteredLogger)，它也来自 [LoggingExtras.jl](@ref) 包。此记录器类似于 `EarlyFilteredLogger`，唯一的区别是传给谓词函数的命名元组包含更多数据，请参阅 [`LoggingExtras.handle_message_args`](@ref)。以下是根据消息字符串内容进行过滤的记录器示例：

```@example filtering3
using Logging, LoggingExtras

begin # hide
global_logger() = current_logger() # hide
logger = ActiveFilteredLogger(global_logger()) do args
    return args.message == "Hello there!" || args.message == "General Kenobi!"
end

with_logger(logger) do
    @info "I find your lack of faith disturbing."
    @info "Hello there!"
    @info "General Kenobi!"
    @info "Power! Unlimited power!"
end
end # hide
```
