---
title: "Golang Captcha"
date: 2025-04-02T10:00:00+08:00
---

## golang创造验证码

网上的一些验证码库存在部分兼容性问题, 有的不支持拓展, 有的不支持中文, 有的不支持自定义字体等等
所以使用标准库和字体库自己生成更能适应需求, 比如数字加减法, 字体, 颜色等等

```go
package main

import (
	"bytes"
	"fmt"
	"image"
	"image/color"
	"image/draw"
	"image/png"
	"io/ioutil"
	"math/rand"
	"os"
	"time"

	"github.com/golang/freetype"
	"github.com/golang/freetype/truetype"
)

// 定义字符集（去除了容易混淆的字符）
var chars = []byte{
	'2', '3', '4', '5', '6', '7', '8', '9',
	'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H',
	'J', 'K', 'L', 'M', 'N', 'P', 'Q', 'R',
	'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z',
	'@', '#', '$', '%', '&', '*', '?', '!',
}

// CaptchaConfig 验证码配置
type CaptchaConfig struct {
	Width       int           // 图片宽度
	Height      int           // 图片高度
	Length      int           // 验证码长度
	NoiseCount  int           // 噪点数量
	LineCount   int           // 干扰线数量
	FontSize    float64       // 字体大小
	FontDPI     float64       // 字体DPI
	BgColor     color.RGBA    // 背景颜色
	FontColor   color.RGBA    // 字体颜色
	NoiseColor  color.RGBA    // 噪点颜色
	LineColor   color.RGBA    // 干扰线颜色
	FontFile    string        // 字体文件路径
}

// DefaultConfig 默认配置
func DefaultConfig() *CaptchaConfig {
	return &CaptchaConfig{
		Width:      240,
		Height:     80,
		Length:     5,
		NoiseCount: 50,
		LineCount:  3,
		FontSize:   32,
		FontDPI:    72.0,
		BgColor:    color.RGBA{R: 245, G: 245, B: 245, A: 255},
		FontColor:  color.RGBA{R: 30, G: 30, B: 30, A: 255},
		NoiseColor: color.RGBA{R: 180, G: 180, B: 180, A: 255},
		LineColor:  color.RGBA{R: 200, G: 200, B: 200, A: 255},
		FontFile:   "", // 需要指定字体文件路径
	}
}

// GenerateCaptcha 生成验证码
func GenerateCaptcha(config *CaptchaConfig) (string, *image.RGBA, error) {
	// 创建空白画布
	img := image.NewRGBA(image.Rect(0, 0, config.Width, config.Height))
	
	// 填充背景色
	draw.Draw(img, img.Bounds(), &image.Uniform{C: config.BgColor}, image.Point{}, draw.Src)
	
	// 生成随机验证码
	code := make([]byte, config.Length)
	for i := 0; i < config.Length; i++ {
		code[i] = chars[rand.Intn(len(chars))]
	}
	captchaText := string(code)
	
	// 加载字体
	fontBytes, err := ioutil.ReadFile(config.FontFile)
	if err != nil {
		return "", nil, fmt.Errorf("读取字体文件失败: %v", err)
	}
	
	font, err := truetype.Parse(fontBytes)
	if err != nil {
		return "", nil, fmt.Errorf("解析字体失败: %v", err)
	}
	
	// 绘制验证码文本
	err = drawText(img, captchaText, font, config)
	if err != nil {
		return "", nil, err
	}
	
	// 添加干扰线
	addLines(img, config.LineCount, config.LineColor)
	
	// 添加噪点
	addNoise(img, config.NoiseCount, config.NoiseColor)
	
	return captchaText, img, nil
}

// drawText 绘制文本
func drawText(img *image.RGBA, text string, font *truetype.Font, config *CaptchaConfig) error {
	c := freetype.NewContext()
	c.SetDPI(config.FontDPI)
	c.SetFont(font)
	c.SetFontSize(config.FontSize)
	c.SetClip(img.Bounds())
	c.SetDst(img)
	c.SetSrc(image.NewUniform(config.FontColor))
	
	// 计算字符间距
	charWidth := int(config.FontSize * 0.8)
	startX := (config.Width - charWidth*len(text)) / 2
	startY := int(float64(config.Height)/2 + config.FontSize/3)
	
	// 绘制每个字符
	for i, ch := range text {
		// 轻微随机偏移
		x := startX + i*charWidth + rand.Intn(5) - 2
		y := startY + rand.Intn(5) - 2
		
		pt := freetype.Pt(x, y)
		if _, err := c.DrawString(string(ch), pt); err != nil {
			return err
		}
	}
	
	return nil
}

// addLines 添加干扰线
func addLines(img *image.RGBA, count int, lineColor color.Color) {
	for i := 0; i < count; i++ {
		x1 := rand.Intn(img.Bounds().Dx())
		y1 := rand.Intn(img.Bounds().Dy())
		x2 := rand.Intn(img.Bounds().Dx())
		y2 := rand.Intn(img.Bounds().Dy())
		
		drawLine(img, x1, y1, x2, y2, lineColor)
	}
}

// drawLine 绘制直线
func drawLine(img *image.RGBA, x1, y1, x2, y2 int, color color.Color) {
	dx := abs(x2 - x1)
	dy := abs(y2 - y1)
	sx, sy := 1, 1
	if x1 >= x2 {
		sx = -1
	}
	if y1 >= y2 {
		sy = -1
	}
	err := dx - dy
	
	for {
		img.Set(x1, y1, color)
		if x1 == x2 && y1 == y2 {
			break
		}
		e2 := 2 * err
		if e2 > -dy {
			err -= dy
			x1 += sx
		}
		if e2 < dx {
			err += dx
			y1 += sy
		}
	}
}

// addNoise 添加噪点
func addNoise(img *image.RGBA, count int, noiseColor color.Color) {
	for i := 0; i < count; i++ {
		x := rand.Intn(img.Bounds().Dx())
		y := rand.Intn(img.Bounds().Dy())
		img.Set(x, y, noiseColor)
	}
}

func abs(x int) int {
	if x < 0 {
		return -x
	}
	return x
}

func main() {
	// 初始化随机种子
	rand.Seed(time.Now().UnixNano())
	
	// 使用默认配置
	config := DefaultConfig()
	config.FontFile = "arial.ttf" // 替换为你的字体文件路径
	
	// 生成验证码
	text, img, err := GenerateCaptcha(config)
	if err != nil {
		fmt.Println("生成验证码失败:", err)
		return
	}
	
	fmt.Println("验证码:", text)
	
	// 保存为PNG文件
	file, err := os.Create("captcha.png")
	if err != nil {
		fmt.Println("创建文件失败:", err)
		return
	}
	defer file.Close()
	
	// 编码为PNG
	var buf bytes.Buffer
	if err := png.Encode(&buf, img); err != nil {
		fmt.Println("编码PNG失败:", err)
		return
	}
	
	if _, err := file.Write(buf.Bytes()); err != nil {
		fmt.Println("写入文件失败:", err)
		return
	}
	
	fmt.Println("验证码图片已保存为 captcha.png")
}
```