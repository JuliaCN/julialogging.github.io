# [如何轮换日志文件](@id How-to-rotate-log-files)

*日志轮换*对于长时间运行的应用程序很常见，例如网络服务器，请参见 [`logrotate`](https://linux.die.net/man/8/logrotate) 的 Linux 系统示例。日志轮换意味着根据某种标准替换日志文件。通常日志文件根据日期轮换，例如每天或每周，或根据文件大小轮换，例如将单个文件保持在 10MB 以下。

## 基于日期的日志轮换

[LoggingExtras.jl](@ref) 包实现了 [`DatetimeRotatingFileLogger`](@ref LoggingExtras.DatetimeRotatingFileLogger)，顾名思义，它是一个基于日期/时间的日志轮换记录器。日志轮换的频率是根据日期格式（请参阅 [`Dates.DateFormat`](https://docs.julialang.org/en/v1/stdlib/Dates/#Dates.DateFormat) 和 [`dateformat"..."`](https://docs.julialang.org/en/v1/stdlib/Dates/#Dates.@dateformat_str)）形式的输入文件名模式确定的。

让我们看一个初始示例：

```@example datetime-rotate
using Logging, LoggingExtras

# Directory for our log files
logdir = joinpath(@__DIR__, "logs")
logdir = joinpath(mktempdir(), "logs") # hide
mkpath(logdir)

# Filename pattern (see note below about character escaping)
filename_pattern = raw"yyyy-mm-dd-\w\e\b\s\e\r\v\e\r.\l\o\g"

# Create the logger
logger = DatetimeRotatingFileLogger(logdir, filename_pattern)

old_global_logger = global_logger() # hide
# Install the logger globally
global_logger(logger)
global_logger(old_global_logger) # hide
rm(logdir; recursive=true, force=true) # hide
```

这是一个每天轮换日志文件的记录器，因为“天”是文件名模式中最小的日期时间单位。

!!! note
    请注意，文件名模式中不属于日期时间模式的所有字符都将被转义。如果没有转义，这些字符也将被 [`Dates.DateFormat`](https://docs.julialang.org/en/v1/stdlib/Dates/#Dates.DateFormat) 解释。从技术上讲，并非所有字符都需要转义，例如 `w` 没有意义，但像上面的示例一样转义所有字符是最安全的。

经过几天的日志记录，我们会在日志目录中找到以下文件：

```bash
$ ls logs/
2021-11-12-webserver.log
2021-11-13-webserver.log
2021-11-14-webserver.log
2021-11-15-webserver.log
```

---

现在让我们通过添加两个 `logrotate` 中的常用的功能来改进记录器：文件压缩和文件保留策略。日志文件通常是可压缩的，添加压缩可以为我们节省一些空间。文件保留策略让我们将日志文件保留固定天数，例如 30 天，然后自动删除它们。对压缩和保留策略的支持不是内置的，但我们可以使用外部包来实现这些目的，并在回调函数中使用 `rotation_callback` 关键字参数实现此功能。`DatetimeRotatingFileLogger` 每次轮换日志文件时都会调用此函数。该函数的唯一参数是“旧”文件。

我们将通过 `Gzip_jll` 包使用 `gzip` 压缩，：

```@example datetime-rotate
using Gzip_jll

function logger_callback(file)
    # Compress the file
    Gzip_jll.gzip() do gzip
        run(`$(gzip) $(file)`)
    end
end
nothing # hide
```
对于文件保留策略，我们将使用 [FilesystemDatastructures](https://github.com/staticfloat/FilesystemDatastructures.jl) 包中的 `NFileCache`。这里我们创建了一个可以保存 30 个文件的文件缓存，：

```@example datetime-rotate
using FilesystemDatastructures

# Create a file cache that keeps 30 files
fc = NFileCache(logdir, 30, DiscardLRU();
                # Make sure only files ending with "webserver.log.gz" are included
                predicate = x -> endswith(x, r"webserver\.log\.gz")
)
nothing # hide
```

现在我们只需要修改上面的回调，将轮换和压缩的文件添加到缓存中：

```@example datetime-rotate
function logger_callback(file)
    # Compress the file
    Gzip_jll.gzip() do gzip
        run(`$(gzip) $(file)`)
    end
    # Add the compressed file to the cache (gzip adds the .gz extension)
    FilesystemDatastructures.add!(fc, file * ".gz")
end
nothing # hide
```

当第 31 个文件添加到缓存中时，将自动删除最旧的文件来为新文件腾出空间。让应用程序运行一段时间后检查日志目录：

```bash
$ ls logs/
2021-11-20-webserver.log.gz
2021-11-21-webserver.log.gz
2021-11-22-webserver.log.gz
[...]
2021-12-17-webserver.log.gz
2021-12-18-webserver.log.gz
2021-12-19-webserver.log.gz
2021-12-20-webserver.log
```

30 个压缩文件，由缓存管理，还有一个“活动”文件尚未压缩并添加到缓存中。

---

这是完整示例：

```@example datetime-rotate-complete
using Logging, LoggingExtras, Gzip_jll, FilesystemDatastructures

# Directory for our log files
logdir = joinpath(@__DIR__, "logs")
logdir = joinpath(mktempdir(), "logs") # hide
mkpath(logdir)

# Filename pattern
filename_pattern = raw"yyyy-mm-dd-\w\e\b\s\e\r\v\e\r.\l\o\g"

# File cache that keeps 30 files
fc = NFileCache(logdir, 30, DiscardLRU();
                # Make sure only files ending with "webserver.log.gz" are included
                predicate = x -> endswith(x, r"webserver\.log\.gz")
)

# Callback function for compression and adding to cache
function logger_callback(file)
    # Compress the file
    Gzip_jll.gzip() do gzip
        run(`$(gzip) $(file)`)
    end
    # Add the compressed file to the cache (gzip adds the .gz extension)
    FilesystemDatastructures.add!(fc, file * ".gz")
end

# Create the logger
logger = DatetimeRotatingFileLogger(
             logdir, filename_pattern;
             rotation_callback = logger_callback,
)

old_global_logger = global_logger() # hide
# Install the logger globally
global_logger(logger)
global_logger(old_global_logger) # hide
rm(logdir; recursive=true, force=true) # hide
```

!!! note
    上面的设置与 Julia 的包服务器使用的日志记录设置非常相似，请参阅 [JuliaPackaging/PkgServer.jl:bin/run_server.jl](https://github.com/JuliaPackaging/PkgServer.jl/blob/72e3f280ac0f91d9ded17f23920f0aa3e2a28a89/bin/run_server.jl#L24-L53)。

## 基于文件大小的日志轮换

对于基于文件大小的轮换，例如当文件大小达到特定阈值时进行文件轮换，请查看 [LogRoller.jl](@ref) 包。
