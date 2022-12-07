# [如何记录日志到文件](@id How-to-log-to-a-file)

!!! tip
    跟随教程把代码片段复制粘贴到 Julia REPL 中执行，是个不错的注意！

通常需要把日志消息发送到日志文件，而不是记录到终端。来自 `Logging` 标准库的 [`ConsoleLogger`](@ref Logging.ConsoleLogger) 和 [`SimpleLogger`](@ref Logging.SimpleLogger) 接收一个 IO 流作为输入，因此，在这些记录器中插入一个文件流并记录到文件是很简单的。这是一个示例：

```@example log-to-file
using Logging

io = open("logfile.txt", "w")
logger = ConsoleLogger(io)

with_logger(logger) do
    @info "Message to the logfile"
end

close(io)
```

当读取文件并打印结果时，我们可以验证此文件包含预期的日志输入：

```@example
print(read("logfile.txt", String))
rm("logfile.txt") # hide
```

使用这种方法，您可能会注意到，由于 [IO buffering](https://en.wikipedia.org/wiki/Data_buffer)，消息将延迟写入文件，或者可能直到程序结束和文件关闭时才写入。此外，像上面的例子那样自己管理文件 IO 流有点烦人。由于这些原因，通常最好使用专门为文件写入而实现的记录器。

[LoggingExtras.jl](@ref)包提供了 [`FileLogger`](@ref LoggingExtras.FileLogger), 顾名思义，这是专门为将消息写入文件而实现的。`FileLogger` 打开文件并在处理每条消息后自动刷新流。以下是 `FileLogger` 的使用示例：

```@example filelogger
using Logging, LoggingExtras

logger = FileLogger("logfile.txt")

with_logger(logger) do
    @info "First message to the FileLogger!"
end
```
读取和打印内容：

```@example filelogger
print(read("logfile.txt", String))
```

默认情况下，当 `FileLogger` 打开文件流时，使用 “write” 模式（请参阅文档 [`open`](https://docs.julialang.org/en/v1/base/io-network/#Base.open)），这意味着如果文件存在并且有一些内容，它将被覆盖。可以通过传递 `append = true` 给构造函数来使用“追加”模式。这将保留文件中的内容并且仅在末尾追加。这是一个示例，打开和上面相同的文件，但使用 `append = true`：


```@example filelogger

logger = FileLogger("logfile.txt"; append = true)

with_logger(logger) do
    @info "Second message to the FileLogger?"
end

print(read("logfile.txt", String))
rm("logfile.txt") # hide
```

如您所见，第一条消息仍在文件中，我们只附加了第二条。

在 `FileLogger` 内部使用了 [`SimpleLogger`](@ref Logging.SimpleLogger)，因此输出格式与`SimpleLogger`。 如果你想控制输出格式，你可以使用[`FormatLogger`](@ref LoggingExtras.FormatLogger)（也来自 [LoggingExtras.jl](@ref)）代替。`FormatLogger` 接受一个格式化函数作为第一个参数。格式化函数有两个参数：(i) 写入格式化消息的 IO 流和 (ii) 日志参数（参见 [`LoggingExtras.handle_message_args`](@ref)）。这是一个例子：

```@example formatlogger
using Logging, LoggingExtras

logger = FormatLogger(open("logfile.txt", "w")) do io, args
    # Write the module, level and message only
    println(io, args._module, " | ", "[", args.level, "] ", args.message)
end

with_logger(logger) do
    @info "Info message to the FormatLogger!"
    @warn "Warning message to the FormatLogger!"
end

print(read("logfile.txt", String))
rm("logfile.txt") # hide
```

`FormatLogger` 需要我们使用推荐选项（`"w"`, `"a"`, 等）手动打开文件流，就像 `FileLogger` 一样，它会在每条消息后刷新流。

也可以看看：

  - [将消息发送到多个位置](@ref Send-messages-to-multiple-locations)，了解如何将日志消息发送到多个记录器，例如发送到文件和终端。
  - [如何轮换日志文件](@ref How-to-rotate-log-files) 获取更多文件日志记录选项。

