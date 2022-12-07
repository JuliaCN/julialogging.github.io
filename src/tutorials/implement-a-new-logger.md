# 实现一个新的日志记录器

在此教程中，我们将会经历实现一个新记录器的必需步骤。这包括定义一个新的结构体，它是 `Logging.AbstractLogger` 的子类型，并且为此记录器接口实现必需的方法。

!!! note
    通常，除非你要实现一个新的记录器槽（logger sink)，否则不需要定义新的记录器来获取你想要的行为。[LoggingExtras.jl](@ref) 包提供了任意路由，转换，和过滤日志事件的记录器。例如，在此示例中实现的记录器使用 [`TransformerLogger`](@ref LoggingExtras.TransformerLogger) 实现起来很简单（请参阅[本页的最后一部分](@ref cipher-existing)）。

作为一个玩具示例，我们将实现一个加密记录器--一个使用 [Caesar cipher](https://en.wikipedia.org/wiki/Caesar_cipher)） 来“加密”消息的记录器。我们想要记录器接收任何日志事件，并且可以使用输出流进行配置。开始吧！

!!! tip
    跟随教程把代码片段复制粘贴到 Julia REPL 中执行，是个不错的注意！

第一步是加载 [Logging.jl](@ref) 标准库，它定义了记录日志的基础设置和必须方法，我们需要为新记录器进行扩展。然后我们定义新记录器的结构，确保使用 `Logging.AbstractLogger` 作为超类（supertype)：

```julia
using Logging

struct CipherLogger <: Logging.AbstractLogger
    io::IO
    key::Int
end

CipherLogger(key::Int=3) = CipherLogger(stderr, key)
```

输入到记录器结构的是 I/O 流和凯撒密码的密钥，所有的日志消息都会被写入此流，密钥只是一个整数。一个供外部使用的方便结构也会定义一些默认值：I/O 流默认为 `stderr`，密钥默认为 `3`。

下一步，我们需要为新的 `CipherLogger` 扩展必需的方法：

#### [`Logging.min_enabled_level`](@ref)

此方法应该返回日志记录器接收消息的最小等级。Julia REPL 中默认记录器只接收 `Logging.Info` 及以上级别的消息，例如，我们想要我们的记录器接收*任何消息*,因此我们简单的返回 `Logging.BelowMinLevel`，这是最小的合理级别：

```julia
Logging.min_enabled_level(logger::CipherLogger) = Logging.BelowMinLevel
```

#### [`Logging.shouldlog`](@ref)

此方法是过滤日志消息的下一个机会。输入参数是记录器，日志消息级别，被创建日志事件的所属模块，和唯一标识事件创建位置的日志事件 id。基于此信息，我们可以决定记录器是否应该接收消息并返回 `true`，或者它应该丢弃消息并返回 `false`。再一次，因为我们想让我们的记录器接收*任何消息*，无论输入参数是什么，我们都简单的返回 `true`： 

```julia
function Logging.shouldlog(logger::CipherLogger, level, _module, group, id)
   return true
end
```

#### [`Logging.catch_exceptions`](@ref)

此方法决定我们的记录器是否应该捕获来自日志记录系统的异常。这是可以做到的，例如，当生成日志消息时发生的异常。如果 `catch_exceptions` 返回 `true`，日志记录系统将发送一个 error 日志消息到记录器，否则不发送。让我们接收这些 error 日志消息：

```julia
Logging.catch_exceptions(logger::CipherLogger) = true
```

#### [`Logging.handle_message`](@ref)

此方法是日志事件最终到达的位置。输入参数是记录器，日志级别，消息，和关于消息来源位置的元数据（module, group, id, file, 和 line）。另外，日志事件可能有关键字参数，例如：

```julia
@info "hello, world" time = time()
```

记录器将会发送关键字参数 `time => time()`（详细信息请查看 [Logging 基本用法](@ref Logging-basics)教程）。基于此信息，我们现在将为我们的记录器生成一个日志消息，把它打印到记录器 I/O 流。当然，一般来说此方法不必写入常规流，它可以把日志事件发送为一个 HTTP 请求（和 [LokiLogger.jl](@ref) 类似），或者作为短信发送到你的手机。我们 `CipherLogger` 的 `handle_message` 函数是非常简单的：我们把消息加密，然后输出级别和加密后的消息：

```julia
function Logging.handle_message(logger::CipherLogger,
                                lvl, msg, _mod, group, id, file, line;
                                kwargs...)
    # Apply Ceasar cipher on the log message
    msg = caesar(msg, logger.key)
    # Write the formatted log message to logger.io
    println(logger.io, "[", lvl, "] ", msg)
end
```

现在唯一缺失的部分是 `caesar` 函数，它应该用记录器的密钥加密消息。这里的加密只应用到 ASCII 字母 `A-Z` 和 `a-z` 上：

```julia
function caesar(msg::String, key::Int)
    io = IOBuffer()
    for c in msg
        shift = ('a' <= c <= 'z') ? Int('a') : ('A' <= c <= 'Z') ? Int('A') : 0
        if shift > 0
            c = Char((Int(c) - shift + key) % 26 + shift)
        end
        print(io, c)
    end
    return String(take!(io))
end
```

就是这样 -- 我们实现了一个新的记录器！让我们去兜兜风吧。如果你一直跟着学习到了这儿，并且从开始就复制粘贴代码片段到 Julia REPL 中执行，你应该看到和下面相同的输出：

```julia-repl
julia> using Logging

julia> cipher_logger = CipherLogger(3); # new logger with 3 as the key

julia> global_logger(cipher_logger); # set the logger as the global logger

julia> @info "Hello, world!"
[Info] Khoor, zruog!

julia> @info "This is an info message."
[Info] Wklv lv dq lqir phvvdjh.

julia> @warn "This is a warning."
[Warn] Wklv lv d zduqlqj.

julia> @error "This is an error message."
[Error] Wklv lv dq huuru phvvdjh.
```

我们也可以确保我们的记录器接收比 `Logging.Info` 级别低的日志事件（默认的记录器是不支持的）：

```julia-repl
julia> @debug "Is this visible?"
[Debug] Lv wklv ylvleoh?
```

最后，我们来确保记录器也能捕获日志事件异常。例如，这里我们尝试创建一个带有未定义变量 `name` 的消息字符串：

```julia-repl
julia> @info "hello, $name"
[Error] Hafhswlrq zkloh jhqhudwlqj orj uhfrug lq prgxoh Pdlq dw UHSO[17]:1
```

你能破解密文并且理解其意思吗？

### [使用已存在的功能构建 `CipherLogger`](@id cipher-existing)

如本页开头所示，除非你想要对接一个新类型的记录器槽，通常无需实现自己的记录器。相反，最好是组合现有的日志记录器以实现路由、转换、和过滤日志事件。上面的 `CipherLogger` 可以像下面这样使用来自 [LoggingExtras.jl](@ref) 包的 [`TransformerLogger`](@ref LoggingExtras.TransformerLogger) 轻而易举地实现：

```julia
using Logging, LoggingExtras

encryption_logger = TransformerLogger(SimpleLogger(stderr)) do args
    message = caesar(args.message, 3)
    return (; args..., message=message)
end

global_logger(encryption_logger)
```

结果看起来像下面这样：

```julia-repl
julia> @info "hello, world"
┌ Info: khoor, zruog
└ @ Main REPL[5]:1
```
使用已存在的 `TransformerLogger` 的另一个好处是它构造的很好。因此，我们可以用另一层来解密消息：

```julia
decryption_logger = TransformerLogger(encryption_logger) do args
    message = caesar(args.message, -3) # to decrypt just negate the key
    return (; args..., message=message)
end

global_logger(decryption_logger)
```

现在消息再打印到终端之前被加密*和*解密，因此这种情况下，两个记录器只是相互撤销对方的行为。这是个非常没用的例子，但是可组合性对其他一些事情非常有用！

```julia-repl
julia> @info "hello, world"
┌ Info: hello, world
└ @ Main REPL[8]:1
```
