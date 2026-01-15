[English](README.md) | [Русский](README.ru.md)

# High-Performance Colorized Slog Handler

A fast, zero-allocation-focused `slog.Handler` for Go. This library provides beautiful colorized text output, optional asynchronous buffering to reduce syscalls, and advanced features like context-based attribute injection.

## ⚠️ Performance & Compatibility Note
* **Target Use Case**: This handler is specifically designed for **local development**. Due to the overhead of ANSI color processing, it is approximately **3x slower** than a standard non-colorized handler.
* **Production**: A high-performance **JSON version** (without color overhead) is planned for the next release.
* **slog Compatibility**: Supports all standard `slog.Logger` features (groups, attributes, context) **except for `slog.LogValuer`** (coming soon).

## 🚀 Key Features
* **Optimized Memory Management**: Uses a `sync.Pool` to minimize heap allocations.
* **Asynchronous Buffering**: Optional `bufio` integration with a background flusher goroutine for high-throughput environments.
* **Context-Aware Attributes**: Inject log attributes directly into `context.Context` (for Request IDs or Trace IDs).
* **Smart Group Flattening**: Automatically flattens nested `slog.Group` attributes into clean dot-notation keys (e.g., `db.conn.id=5`).
* **Thread-Safe & Graceful**: Atomic flags and mutex protection ensure safe operation and clean shutdowns.
* **Colorized Output**: High-visibility ANSI colors for timestamps, log levels, and metadata.

## 📦 Installation
```shell
go get github.com/ttrtcixy/color-slog-handler
```

## 🛠 Usage

```go
package main

import (
	"context"
	"log/slog"
	"os"
)

func main() {
	cfg := &Config{
		Level:          int(slog.LevelDebug),
		BufferedOutput: true, // Enables background flushing
	}

	handler := NewTextHandler(os.Stdout, cfg)
	logger := slog.New(handler)

	// Important: Close the handler to stop the flusher and flush remaining logs
	defer handler.Close(context.Background())

	logger.Info("User logged in", "user_id", 42, "status", "active")
}
```
Easily pass attributes through the context without modifying every function signature:
```go
// Add attributes to context
ctx := AppendAttrsToCtx(context.Background(), slog.String("trace_id", "af82-bx22"))

// The logger will automatically pick them up
logger.InfoContext(ctx, "processing request")
// Output: 14:05:01 | INFO | processing request trace_id=af82-bx22
```

## ⚙️ Configuration
The `Config` struct supports environment variables via tags:
* `Level`: Logging level (e.g., Debug=-4, Info=0).
* `BufferedOutput`: Enable/Disable 4KB buffer with periodic async flush.

## ⚠️ Important Note on Buffering
If `BufferedOutput`: true is set, you must call `handler.Close(ctx)`:
* It stops the background flusher goroutine.
* It ensures any remaining logs in the 4096-byte buffer are written to the output.

Calling `Close()` on a non-buffered handler will return `ErrNothingToClose`.

## 🛣 Roadmap
* JSON output support with same buffering logic.
* Full `slog.LogValuer` support.
