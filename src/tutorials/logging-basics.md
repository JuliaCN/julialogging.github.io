# [Logging 基本用法](@id Logging-basics)

在此教程中，我们将学习发出*日志消息*和*日志事件*的基本用法。我们也会学习一点关于每条 log 消息由哪些信息组成以及当 log 消息发出后发生了什么。如果你要写一个脚本或者包，此章节应该覆盖了你需要的所有信息。

!!! tip
    跟随教程把代码片段复制粘贴到 Julia REPL 中执行，是个不错的注意！

### 基本日志事件

log 事件中大部分重要的信息是*日志消息*和*日志级别*。日志消息通常是一个信息量很大的文本字符串，日志级别是个严重性级别，用来向读者说明 log 消息的重要性。Julia的 `Base` 模块默认是可用的，它提供了日志宏 `@ingo`, `@warn`, `@error`,和 `@debug` 来创建日志消息。它们对应到常见的日志级别：

- **Info**: 有用的信息但是一点也不严重
- **Warning**: 一些某事可能不正确的信息
- **Error**: 某事发生了错误的信息
- **Debug**: 在调试时很有用的附加信息

让我们看看如何使用上面提到的宏创建一些日志消息：

```@repl
@info "This is an info message, carry on as usual."
@warn "This is a warning message, something might be wrong?"
@error "This is an error message, something is not working as it should!"
@debug "This is a debug message, it is invisible!"
printstyled("\njulia>"; color=:green, bold=true) # hide
```

正如你看到的，简单的用法只需要宏，它确定了日志级别和消息字符串。默认的日志后端负责处理我们生成的日志消息，像上面示例展示的一样打印已格式化，色彩化的消息到终端。格式默认是由日志级别决定的： `@info` 消息不输出像 `@warn` 和 `@error` 一样的位置信息（模块，文件，行号）。你也可以注意到 `@debug` 消息默认情况下是关闭的。通常在你调试代码中的问题需要更多信息时，才需要开启它们。


### 向日志事件添加额外元数据

在上述示例中，我们传给日志宏的唯一信息是日志消息字符串。可以通过在消息字符串后面附加额外事物来传递更多信息。额外信息可以使用 `key = value` 语法添加：

```@repl
@info "hello, world" x = [1, 2, 3] y = "something else"
```

或者仅附加一个变量:

```@repl
x = [3, 4, 5];
@info "hello, world" x
```

在这两个示例中，你可以看打日志后端很好的格式化了日志消息字符串后面的额外信息。添加额外信息在一些情况下是很有用的，特别是如果你想保持消息字符串不变，只传递一些动态信息。


### 日志级别

四个日志级别 `Info`, `Warn`, `Error`, 和 `Debug` 只是特定数字级别的别名，允许的日志级别范围正如你在下表看到的：

| Level    | Alias                   | Comment                         |
|:--------:|:------------------------|:--------------------------------|
| -1000001 | `Logging.BelowMinLevel` | (below) lowest possible level   |
| -1000    | `Logging.Debug`         | log level for `@debug` messages |
| 0        | `Logging.Info`          | log level for `@info` messages  |
| 1000     | `Logging.Warn`          | log level for `@warn` messages  |
| 2000     | `Logging.Error`         | log level for `@error` messages |
| 1000001  | `Logging.AboveMaxLevel` | (above) highest possible level  |

Logging 模块是 Julia 自带的，提供了 [`@logmsg`](@ref Logging.@logmsg) 宏，它是目前为止提到的其他宏的泛化。使用此宏时除了消息字符串，它还要求传递一个日志级别，但除此之外，`@logmsg` 的工作原理是一样的，这是一些示例：

```@repl logmsg
using Logging
@logmsg Logging.Info "Info message from @logmsg."
@logmsg Logging.Error "Error message from @logmsg." x = [1, 2, 3]
```

`@logmsg` 宏也可以使用 `LogLevel` 构造函数创建任何级别的日志消息：

```@repl logmsg
@logmsg Logging.LogLevel(123) "Log message with log level 123."
@logmsg Logging.LogLevel(1234) "Log message with log level 1234."
@logmsg Logging.LogLevel(2345) "Log message with log level 2345."
```

从彩色输出我们可以推断出日志级别 123 是一个 `info` 消息，日志级别 1234 是一个 `Warn` 消息，日志级别 2345 是一个 `Error` 消息。你也可以在上表的帮助下看到这些：一个具有等级 `X` 的日志消息是一个：
 - debug message if `Logging.Debug <= X < Logging.Info`,
 - info message if `Logging.Info <= X < Logging.Warn`,
 - warn message if `Logging.Warn <= X < Logging.Error`,
 - error message if `Logging.Error <= X < Logging.AboveMaxLevel`.

像这样自定义的日志级别不常见，但是有时它很方便，并且具有一些更细粒度的控制。


### 位置元数据

在上面的日志输出中你看到一些其他元数据附加到消息。特别是，日志事件的来源模块，文件名和行号被打印。因为代码是在 Julia REPL 中运行的，模块是 `Main`，文件是 `REPL[..]`，行号为 `1`。每个上面提到的宏生成的日志事件都和以下元数据关联：
 - 来源模块，
 - 来源文件，
 - 来源行号，
 - 一个日志事件 id（位置独有的），
 - 一个日志事件组（默认为文件名）。

由日志后端选择如何处理此信息。例如，Julia REPL 中默认的记录器后端对于 `@info` 消息不显式元数据，对于其他消息仅显式模块，文件，和行号。也可以覆盖默认设置，例如改变来源位置。你需要传递关键字参数到日志消息宏来做到：

```@repl
@warn "Overriding the source module" _module = Base
@warn "Overriding the source file" _file = "example.jl"
@warn "Overriding the source line" _line = 123
@warn "Overriding the log event group" _group = :example
@warn "Overriding the log event id" _id = :id
```

因此默认的记录器后端仅显式模块，文件，和行号，其他的覆盖是不可见的。
