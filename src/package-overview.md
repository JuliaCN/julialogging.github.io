# [Logging 包概述](@id Logging—package-overview)

此章节包含 Julia 中和 logging 相关的一些包的概述。大部分的包和来自 `Base` 的标准 logging 前端宏 `@debug`，`@info`，`@warn`, `@error` 及 [Logging.jl](@ref)  标注库提供的抽象整合在一起。

!!! note
    如果一些 logging 相关的包不在下列列表中，不要犹豫，请立刻把它添加进来。
    （译注：请至[原文](https://julialogging.github.io/package-overview/)添加）。


## [Logging.jl](@id logging-overview)

[![](https://img.shields.io/badge/-code%20repository-blue)](https://github.com/JuliaLang/julia/tree/master/stdlib/Logging)
[![](https://img.shields.io/badge/-external%20docs-blue)](https://docs.julialang.org/en/v1/stdlib/Logging/)

Logging 标准库提供了很多 logging 基础设施，大多数其他的 logging 包都建立在它的基础上。Logging 提供了[`global_logger`](@ref Logging.global_logger) 和 [`with_logger`](@ref Logging.with_logger) 来设置全局/局部（global/local）日志记录器，`AbstractLogger` 接口及以下三个记录器：
 - [`ConsoleLogger`](@ref Logging.ConsoleLogger): Julia REPL 中的默认记录器
 - [`SimpleLogger`](@ref Logging.SimpleLogger): [`ConsoleLogger`](@ref Logging.ConsoleLogger) 的基础版
 - [`NullLogger`](@ref Logging.NullLogger): 等同于 [`/dev/null`](https://en.wikipedia.org/wiki/Null_device)的记录器.

Logging 的功能被用于本文档中的大部分教程及 How-to。

详细信息请查看 [Logging API reference](@ref Logging.jl)。


## [LoggingExtras.jl](@id loggingextras-overview)

[![](https://img.shields.io/badge/-code%20repository-blue)](https://github.com/JuliaLogging/LoggingExtras.jl)
[![](https://img.shields.io/badge/-external%20docs-blue)](https://github.com/JuliaLogging/LoggingExtras.jl/blob/master/README.md)

LoggingExtras 包提供了一些 Logging 标准库的必要扩展。例如用于消息过滤的记录器：
[`MinLevelLogger`](@ref LoggingExtras.MinLevelLogger),
[`EarlyFilteredLogger`](@ref LoggingExtras.EarlyFilteredLogger),
[`ActiveFilteredLogger`](@ref LoggingExtras.ActiveFilteredLogger);用于任意消息转换的
 [`TransformerLogger`](@ref LoggingExtras.TransformerLogger);用于消息路由的 [`TeeLogger`](@ref LoggingExtras.TeeLogger); 以及三个不同的记录器接收器(logger sink):用于记录到磁盘文件的 [`FileLogger`](@ref LoggingExtras.FileLogger),用于自定义日志输出格式的 [`FormatLogger`](@ref LoggingExtras.FormatLogger), 以及用于根据日期轮换记录到磁盘文件的
[`DatetimeRotatingFileLogger`](@ref LoggingExtras.DatetimeRotatingFileLogger)。

LoggingExtras 的功能被用于大部分的 How-to 指南，因此，请参考这些示例。

详细信息请查看 [LoggingExtras API reference](@ref LoggingExtras.jl)。


## [LoggingFormats.jl](@id loggingformats-overview)

[![](https://img.shields.io/badge/-code%20repository-blue)](https://github.com/JuliaLogging/LoggingFormats.jl)
[![](https://img.shields.io/badge/-external%20docs-blue)](https://github.com/JuliaLogging/LoggingFormats.jl/blob/master/README.md)

LoggingFormats 包提供了一些预定义的日志格式，用于和 `LoggingExtras.jl` 包中的 `FormatLogger`, `DatetimeRotatingFileLogger` 一起使用：
 - [`LoggingFormats.JSON`](@ref LoggingFormats.JSON): 输出序列化为 JSON 格式的日志消息。
 - [`LoggingFormats.LogFmt`](@ref LoggingFormats.LogFmt): 输出格式化为 [logfmt](https://brandur.org/logfmt) 格式的日志消息。
 - [`LoggingFormats.Truncated`](@ref LoggingFormats.Truncated): 类似 `ConsoleLogger` 的格式, 但是长消息会被截断。

详细信息请查看 [LoggingFormats API reference](@ref LoggingFormats.jl)。


## [TerminalLoggers.jl](@id terminalloggers-overview)

[![](https://img.shields.io/badge/-code%20repository-blue)](https://github.com/JuliaLogging/TerminalLoggers.jl)
[![](https://img.shields.io/badge/-external%20docs-blue)](https://julialogging.github.io/TerminalLoggers.jl/stable/)

TerminalLoggers 包提供了 [`TerminalLogger`](@ref TerminalLoggers.TerminalLogger)，它是更高级的的记录器，提供日志记录基于终端的漂亮输出。特别是它支持 Markdown 格式的日志消息，以及进度条（建立在 [ProgressLogging](@ref progresslogging-overview) 包之上）。

详细信息请查看 [TerminalLoggers API reference](@ref TerminalLoggers.jl)。


## [ProgressLogging.jl](@id progresslogging-overview)

[![](https://img.shields.io/badge/-code%20repository-blue)](https://github.com/JuliaLogging/ProgressLogging.jl)
[![](https://img.shields.io/badge/-external%20docs-blue)](https://julialogging.github.io/ProgressLogging.jl/stable/)

ProgressLogging 包提供了一些很方便的前端宏，包括使跟踪循环结构进度的日志记录变得更简单的 `@progress`。

详细信息请查看 [ProgressLogging API reference](@ref ProgressLogging.jl)。


## [LogRoller.jl](@id logroller-overview)

[![](https://img.shields.io/badge/-code%20repository-blue)](https://github.com/JuliaLogging/LogRoller.jl)
[![](https://img.shields.io/badge/-external%20docs-blue)](https://github.com/JuliaLogging/LogRoller.jl/blob/master/README.md)

LogRoller 包提供了当日志文件达到大小限制时轮换日志文件的功能。特别是 `IO` [`RollingFileWriter`](@ref LogRoller.RollingFileWriter) (可以和其他的记录器组合使用) 以及 [`RollingLogger`](@ref LogRoller.RollingLogger).

详细信息请查看 [LogRoller API reference](@ref LogRoller.jl)。


## [SyslogLogging.jl](@id logroller-overview)

[![](https://img.shields.io/badge/-code%20repository-blue)](https://github.com/JuliaLogging/SyslogLogging.jl)
[![](https://img.shields.io/badge/-external%20docs-blue)](https://github.com/JuliaLogging/SyslogLogging.jl/blob/master/README.md)

SyslogLogging 包提供了 [`SyslogLogger`](@ref SyslogLogging.SyslogLogger)，它会把消息写到 [syslog](https://en.wikipedia.org/wiki/Syslog)。

详细信息请查看 [SyslogLogging API reference](@ref SyslogLogging.jl)。


## [Logging2.jl](@id logging2-overview)

[![](https://img.shields.io/badge/-code%20repository-blue)](https://github.com/JuliaLogging/Logging2.jl)
[![](https://img.shields.io/badge/-external%20docs-blue)](https://github.com/JuliaLogging/Logging2.jl/blob/master/README.md)

Logging2 包提供了重定向 `stdout` 和 `stderr` 输出到日志系统的工具。

详细信息请查看 [Logging2 API reference](@ref Logging2.jl)。


## [TensorBoardLogger.jl](@id tensorboardlogger-overview)

[![](https://img.shields.io/badge/-code%20repository-blue)](https://github.com/PhilipVinc/TensorBoardLogger.jl)
[![](https://img.shields.io/badge/-external%20docs-blue)](https://philipvinc.github.io/TensorBoardLogger.jl/stable/)

TensorBoardLogger 包可以作为后端把结构化数值型数据记录到 [TensorBoard](https://www.tensorflow.org/tensorboard)。


## [LokiLogger.jl](@id lokilogger-overview)

[![](https://img.shields.io/badge/-code%20repository-blue)](https://github.com/JuliaLogging/LokiLogger.jl)
[![](https://img.shields.io/badge/-external%20docs-blue)](https://github.com/JuliaLogging/LokiLogger.jl/blob/master/README.md)

LokiLogger 包提供了 [`LokiLogger.Logger`](@ref) 记录器，它把日志消息通过 HTTP 发送到一个 [Grafana Loki](https://grafana.com/oss/loki/) 服务器。

详细信息请查看 [LokiLogger API reference](@ref LokiLogger.jl)。


## [LogCompose.jl](@id logcompose-overview)

[![](https://img.shields.io/badge/-code%20repository-blue)](https://github.com/JuliaLogging/LogCompose.jl)
[![](https://img.shields.io/badge/-external%20docs-blue)](https://github.com/JuliaLogging/LogCompose.jl/blob/master/README.md)

LogCompose 包提供了声明的记录器配置和相关的 `.toml` 文件格式。

详细信息请查看 [LogCompose API reference](@ref LogCompose.jl)。

## [MiniLoggers.jl](@id miniloggers-overview)

[![](https://img.shields.io/badge/-code%20repository-blue)](https://github.com/JuliaLogging/MiniLoggers.jl)
[![](https://img.shields.io/badge/-external%20docs-blue)](https://github.com/JuliaLogging/MiniLoggers.jl/blob/master/README.md)

MiniLoggers 包提供了 Julia 日志记录器的最小设置和简单而强大的日志字符串格式。它允许构建自定义的和紧凑的日志记录，支持色彩，输出到外部文件，时间戳等更多设置。

详细信息请查看 [MiniLoggers API reference](@ref MiniLoggers.jl)。
