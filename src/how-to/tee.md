# [将消息发送到多个位置](@id Send-messages-to-multiple-locations)

在本教程中，我们将了解如何将日志消息同时发送到多个位置。在[使用日志记录器](@ref Working-with-loggers)和[如何记录日志到文件](@ref How-to-log-to-a-file)中，我们看到了一些替代记录器（[`NullLogger`](@ref Logging.NullLogger)，[`FileLogger`](@ref LoggingExtras.FileLogger), 等）以及如何使用它们。但是，在这些示例中，我们仅将消息发送到新记录器。[LoggingExtras.jl](@ref) 包为此目的实现了一个记录器[`TeeLogger`](@ref LoggingExtras.TeeLogger)。该名称的灵感来自 [shell 工具
`tee`](https://en.wikipedia.org/wiki/Tee_(command)) ，它将命令输出写入标准输出和文件。

一个非常常见的日志记录设置是将日志写入 `stderr`（如默认记录器所做的那样）*和*一个文件。以下是我们如何使用 `TeeLogger`、`FileLogger`和 默认的 [`ConsoleLogger`](@ref Logging.ConsoleLogger)来做到这点：

```@example tee
using Logging, LoggingExtras

logger = TeeLogger(
    global_logger(),          # Current global logger (stderr)
    FileLogger("logfile.log") # FileLogger writing to logfile.log
)

nothing # hide
```

使用此记录器时，每条消息都将*同时*路由到默认记录器和文件。与日志消息过滤一起，可以创建任意日志消息路由，因为所有记录器都很好地组合并且可以嵌套。这在[如何过滤消息](@ref How-to-filter-messages)中有更多描述，但下面给出了一个简短的例子。

这是一个将消息写入下面三个记录器的记录器：

  - 默认的全局记录器（stderr），
  - `MinLevelLogger` 接受级别 >= `Info` 的任何消息，并使用 `FileLogger` 将它们写入文件 `"logfile.log"`，
  - `MinLevelLogger` 接受级别 >= `Debug` 的任何消息，并使用 `FileLogger` 将它们写入文件 `"debug.log"`。

```@example tee2
using Logging, LoggingExtras

logger = TeeLogger(
    # Current global logger (stderr)
    global_logger(),
    # Accept any messages with level >= Info
    MinLevelLogger(
        FileLogger("logfile.log"),
        Logging.Info
    ),
    # Accept any messages with level >= Debug
    MinLevelLogger(
        FileLogger("debug.log"),
        Logging.Debug,
    ),
)

nothing # hide
```
