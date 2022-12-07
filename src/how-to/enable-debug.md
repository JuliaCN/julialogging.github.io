# [如何启用 `@debug` 消息](@id How-to-enable-debug-messages)

默认 debug 级日志消息默认是不可见的。这是因为默认的 `ConsoleLogger` 接收的最低日志级别是 `Logging.Info` -- 任何比它低的都将被丢弃。

启用所有 `@debug` 级消息的最简单方式是，创建一个接收所有消息的新记录器。示例如下：

```@example enable-debug
using Logging

# New ConsoleLogger that prints to stderr and accept messages with level >= Logging.Debug
debug_logger = ConsoleLogger(stderr, Logging.Debug)
nothing # hide
```

新的记录器现在可以用于替代任务的本地记录器或全局记录器，请参考[使用日志记录器](@ref Working-with-loggers)。示例如下：

```@repl enable-debug
begin # hide
debug_logger = ConsoleLogger(stderr, Logging.Debug) # hide
with_logger(debug_logger) do # Enable the debug logger locally
     @debug "This is visible now!"
end
end # hide

begin # hide
old_global_logger = global_logger() # hide
debug_logger = ConsoleLogger(stderr, Logging.Debug) # hide
global_logger(debug_logger); # Enable the debug logger globally
global_logger(old_global_logger) # hide
end # hide
with_logger(ConsoleLogger(stderr, Logging.Debug)) do # hide
@debug "This is visible now!"
end # hide
```

此方法适用于任何接收日志级别作为参数的日志记录器，然而，情况并非总是如此。启用 debug 消息的另一个更具组合性的选择是，使用基于日志级别的消息过滤，[如何过滤消息](@ref How-to-filter-messages)章节是这种方法的更详细描述，但是这里给出的日志级别过滤示例使用了 [`MinLevelLogger`](@ref LoggingExtras.MinLevelLogger)。

`MinLevelLogger` 记录器是另一个记录器的封装，但是只让足够高级别的消息传入给被封装的记录器。这个示例中，我们封装了一个接收每个消息级别（`Logging.BelowMinLevel`）的 `ConsoleLogger`。

```@example enable-debug2
using Logging, LoggingExtras

begin # hide
logger = MinLevelLogger(
    ConsoleLogger(stderr, Logging.BelowMinLevel),
    Logging.Debug,
)

with_logger(logger) do
    @debug "This is visible!"
end
end # hide
```

要选择性地启用来自某些模块或包的调试消息，或基于日志级别等事情进行过滤，请查看[如何过滤消息](@ref How-to-filter-messages)章节。
To selectively enable debug messages from e.g. certain modules or packages, or filtering
based on things other than the log level, see [How to filter messages](@ref).

### `JULIA_DEBUG` 环境变量

另一个“快速又随性”地启用 debug 消息的方式是，使用`JULIA_DEBUG` (请查看 [Logging/Environmental variables]
(https://docs.julialang.org/en/v1/stdlib/Logging/#Environment-variables))环境变量。此变量可以设置为 `all` 以启用所有调试消息，或设置为一个或多个以逗号分隔的模块名列表来选择性启用某些模块的调试消息。然而，使用 `JULIA_DEBUG` 组合的并不像使用适当的日志消息过滤那样好（请查看[如何过滤消息](@ref How-to-filter-messages)）。请查看此问题的示例以获取更多讨论和信息：
[JuliaLogging/LoggingExtras.jl#20](https://github.com/JuliaLogging/LoggingExtras.jl/issues/20)。
