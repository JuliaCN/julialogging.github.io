# [使用日志记录器](@id Working-with-loggers)

!!! tip
    跟随教程把代码片段复制粘贴到 Julia REPL 中执行，是个不错的注意！

在这篇教程中，我们将学习如何使用记录器，例如接收和处理 `@info` 及其朋友们（查看 [Logging 基本用法](@ref Logging-basics)）发出的日志消息的后端。

Julia 中的默认记录器是 [`ConsoleLogger`](@ref Logging.ConsoleLogger)，它打印日志消息到终端（具体来说，它打印到 [`stderr`](https://docs.julialang.org/en/v1/base/io-network/#Base.stderr)）。

```@repl
@info "Hello default ConsoleLogger!"
```

`ConsoleLogger` 只是记录器后端的其中一个实现，还有很多不同目的的其他实现，请查看 [Logging 包概述](@ref Logging—package-overview)。在此教程中，我们仅尝试 [Logging.jl](@ref) 标准库中定义的记录器，但是无论你使用那种记录器实现，使用记录器的函数都是相同的。

[`global_logger`](@ref Logging.global_logger) 函数用于获取或设置 *全局记录器*：

```@repl
using Logging
global_logger() = current_logger() # hide
global_logger() |> typeof
```

当前的全局记录器会被任何衍生任务继承，因此，如果你想要为你的程序设置一个自定义的记录器，通常更新全局记录器就足够了。这里是一个如何设置全局记录器为 [`NullLogger`](@ref Logging.NullLogger) 以禁言所有日志消息的示例：

```@repl
using Logging
logger = NullLogger();
old_logger = global_logger(logger); # save the old logger
with_logger(NullLogger()) do # hide
@info "This message goes to the new global NullLogger!"
end # hide
global_logger(old_logger); # reset to the old logger
@info "This message goes to the old logger again!"
```

正如你在此例中看到的，当设置一个新记录器时，`global_logger` 函数会返回旧记录器，然后我们重新设置全局记录器为旧记录器。对于一个程序/应用，这通常是你需要的一切--构造一个记录器或一个记录器组合，然后把它设置为全局记录器。

[`with_logger`](@ref Logging.with_logger) 函数可以为一个特定任务修改记录器。

```@repl
using Logging
logger = NullLogger();
with_logger(logger) do
    @info "This message goes to the temporary logger!"
end
@info "Now the logger is back to normal."
```

如你所见，在调用 `with_logger` 之后，记录器的状态未改变，日志消息如之前一样打印。
