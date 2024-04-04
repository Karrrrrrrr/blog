---

title: "SWC"

date: "2022-01-19T18:10:37+08:00"

categories: ["前端"]

tags: ["前端","ts","js","编译"]

---

新的ts编译工具

基本可以做到开箱即用, 从效率上看, 编译比tsc快, 比ts-node直接运行还快

但是无法进行类型检测, 这部分还是需要tsc执行

创建swc配置文件
```.swcrc```
```json
{
  "minify": true,
  "jsc": {
    "parser": {
      "syntax": "typescript",
      "tsx": true,
      "decorators": false,
      "dynamicImport": true
    },
    "transform": {
      "legacyDecorator": false,
      "decoratorMetadata": true
    },
    "target": "esnext",
    "keepClassNames": true,
    "loose": false
  },
  "module": {
    "type": "commonjs",
    "strict": false,
    "strictMode": true,
    "lazy": false,
    "noInterop": false
  },
  "sourceMaps": false
}
```

安装node依赖 
```shell 
pnpm add -D @swc/cli @swc/core
```

编译整个文件夹 
```shell
swc src -d dist
```
编译 单文件
```shell
swc app.ts -o dist/app.js
```

如果需要把依赖一起打包的话, swc无法实现,还是需要rollup或者esbuild

esbuild 打包依赖命令如下
```shell
esbuild index.ts --bundle --outfile=dist/bundle.js --platform=node --minify-whitespace --m inify-syntax --minify-identifiers
```