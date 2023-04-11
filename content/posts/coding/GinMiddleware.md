---
title: "Gin"
date: 2022-07-06T22:26:42+08:00
---

## Gin的一些常用中间件配置


### JWT 
```go

```


### Log 
```go
package middleware

import "github.com/gin-gonic/gin"
func ColorLogger() gin.HandlerFunc {
	return gin.LoggerWithConfig(gin.LoggerConfig{
		//Output: logger(),
		Formatter: func(param gin.LogFormatterParams) string {
			now := time.Now()
			format := now.Format(time.DateTime)
			var color string
			switch param.StatusCode / 100 {
			case 2:
				color = "\033[34m"
			case 3:
				color = "\033[32m"
			case 4:
				color = "\033[33m"
			case 5:
				color = "\033[31m"
			default:
				color = "\033[0m"
			}
			const reset = "\033[0m\n"
			return fmt.Sprintf("%s%s\t[%-5s]\t[%v]\t[%v]\t%-6s\t%s\t%s\t%s\t%s",
				color,
				format,
				param.Method,
				param.StatusCode,
				param.Request.RemoteAddr,
				param.Latency.String(),
				param.ClientIP,
				param.ErrorMessage,
				param.Path, reset)
		},
	})
}

```

## 自定义日志库 xlog
```go
package xlog

import (
	"encoding/json"
	"fmt"
	"go.uber.org/zap"
	"go.uber.org/zap/zapcore"
	"os"
	"runtime"
	"strings"
	"time"
)

const (
	green   = "\033[97;32m"
	white   = "\033[90;37m"
	yellow  = "\033[90;33m"
	red     = "\033[97;31m"
	blue    = "\033[97;34m"
	magenta = "\033[97;35m"
	cyan    = "\033[97;36m"
	reset   = "\033[0m"
)

var logger *zap.Logger

func EncodeLevel(level zapcore.Level, encoder zapcore.PrimitiveArrayEncoder) {
	//

	_, file, line, _ := runtime.Caller(6)
	file = fmt.Sprintf("%s:%d", file, line)
	format := time.Now().Format(time.DateTime)
	_ = format
	var color string
	switch level {
	case zapcore.DebugLevel:
		color = green
	case zapcore.InfoLevel:
		color = blue
	case zapcore.WarnLevel:
		color = yellow
	case zapcore.ErrorLevel:
		color = red
	case zapcore.DPanicLevel:
		color = cyan
	case 10:
		color = green
	}
	encoder.AppendString(fmt.Sprintf("%s[%-6s|%s| %s ]", color, level.String(), format, file))

}
func init() {
	encoderConfig := zap.NewProductionEncoderConfig()
	encoderConfig.EncodeTime = zapcore.ISO8601TimeEncoder

	// 自定义文件：行号输出项

	// 创建一个颜色输出器
	consoleEncoder := zapcore.NewConsoleEncoder(encoderConfig)
	colorEncoder := zapcore.NewConsoleEncoder(zapcore.EncoderConfig{
		MessageKey:    "msg",
		LevelKey:      "level",
		NameKey:       "logger",
		CallerKey:     "caller_line",
		StacktraceKey: "stacktrace",
		LineEnding:    zapcore.DefaultLineEnding,
		EncodeLevel:   EncodeLevel, // 设置颜色编码器
		EncodeTime:    zapcore.ISO8601TimeEncoder,
		EncodeCaller:  zapcore.FullCallerEncoder,
	})
	_ = consoleEncoder
	_ = colorEncoder
	core := zapcore.NewTee(
		zapcore.NewCore(colorEncoder, os.Stdout, zap.DebugLevel),
	)
	logger = zap.New(core)
}

func toString(message []any) string {
	var builder = strings.Builder{}
	for i := range message {
		marshal, _ := json.Marshal(message[i])
		builder.Write(marshal)
		builder.Write([]byte{'\t'})
	}
	return builder.String()
}
func toList(message []any) []any {
	var res = make([]any, 0, len(message))
	for i := range message {
		marshal, _ := json.Marshal(message[i])
		res = append(res, string(marshal))
	}
	return res
}

func Error(message ...any) {
	logger.Error(toString(message))
}
func Info(message ...any) {
	logger.Info(toString(message))
}
func Fatal(message ...any) {
	logger.Fatal(toString(message))
}
func Debug(message ...any) {
	logger.Debug(toString(message))
}
func Warn(message ...any) {
	logger.Warn(toString(message))
}
func ErrorF(format string, message ...any) {
	logger.Error(fmt.Sprintf(format, toList(message)...))
}
func InfoF(format string, message ...any) {
	logger.Info(fmt.Sprintf(format, toList(message)...))
}
func FatalF(format string, message ...any) {
	logger.Fatal(fmt.Sprintf(format, toList(message)...))
}
func DebugF(format string, message ...any) {
	logger.Debug(fmt.Sprintf(format, toList(message)...))
}
func WarnF(format string, message ...any) {
	logger.Warn(fmt.Sprintf(format, toList(message)...))
}

```