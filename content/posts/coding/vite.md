---
title: "Vite"
date: "2023-04-25T08:42:22+08:00"
---


## 常见配置
vite初始化完总是需要手动配置一些自定义参数,这里记录一些常用配置
```shell
pnpm add -D unplugin-vue-components unplugin-auto-import # 宏定义
pnpm add -D rollup-plugin-visualizer # 打包分析工具
pnpm add -D @types/node # 不影响编译, 给webstorm看的
```

当我们使用了 ```unplugin-auto-import```之后, 编译器会识别,但是编辑器不会识别,
所以我们需要手动把```auto-import.d.ts```加入到```tsconfig.json```

如果是vue项目, 通常需要以下若干个包
```shell
pnpm add pinia naive-ui axios vue-router vueuse
pnpm add echarts # 可选
pnpm add katex  # 可选

```



---
## vite.config.ts
```ts
// vite.config.ts
import {defineConfig} from 'vite'
import vue from '@vitejs/plugin-vue'
import AutoImport from 'unplugin-auto-import/vite'
import Components from 'unplugin-vue-components/vite'
import {NaiveUiResolver} from 'unplugin-vue-components/resolvers'
import vueJsxPlugin from "@vitejs/plugin-vue-jsx";
import createDemoPlugin from "./build/vite-plugin-demo"
import path from 'path'
import {visualizer} from "rollup-plugin-visualizer";


export default defineConfig({
    plugins: [
        createDemoPlugin(),
        // vue(),
        visualizer({
            gzipSize: true,
            brotliSize: true,
            emitFile: false,
            filename: "dist/test.html", //分析图生成的文件名
            open: false  //如果存在本地服务端口，将在打包后自动展示
        }),

        vueJsxPlugin(),
        AutoImport({
            imports: [
                'vue',
                'vue-router',
                {
                    'naive-ui': [
                        'useDialog',
                        'useMessage',
                        'useNotification',
                        'useLoadingBar'
                    ]
                }
            ]
        }),
        Components({
            resolvers: [NaiveUiResolver()]
        })
    ],
    resolve: {
        alias: {
            '@': path.resolve('./src'),
        },
        extensions: ['.mjs', '.ts', '.tsx', '.vue', '.js', '.jsx', '.json', '.md']
    },
    build: {
        rollupOptions: {
            output: {
                manualChunks(id, {getModuleInfo, getModuleIds}) {
                    const t2 = id.split('/node_modules/')
                    if (t2.length == 3) {
                        const strings = t2[2].split('/')
                        return strings[0]
                    }
                },
            },
        },
    },

})
```

---
## tsconfig.json

```json
// tsconfig.json

{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "module": "ESNext",
    "lib": [
      "ES2020",
      "DOM",
      "DOM.Iterable"
    ],
    "skipLibCheck": true,
    /* Bundler mode */
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "preserve",
    /* Linting */
    "strict": false, // 关闭严格模式, 没使用的变量不检查
    "noUnusedLocals": false,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": false,
    "allowJs": true, // ts中允许引入js
    "paths": {
      "@/*": [
        "src/*"
      ],
    },
    "allowSyntheticDefaultImports": true
  },
  "include": [
    "src/**/*.ts",
    "src/**/*.d.ts",
    "src/**/*.tsx",
    "src/**/*.vue",
    "src/**/*.tsx",
    "src/**/*.jsx",
    "src/**/*.md", // 当我们自定义一些后缀导入, 就需要在这里加
    "auto-imports.d.ts" // 同上, 宏定义的
  ],
  "references": [
    {
      "path": "./tsconfig.node.json"
    }
  ]
}

```


## tsconfig.node.json
```json
{
  "compilerOptions": {
    "composite": true,
    "skipLibCheck": true,
    "module": "ESNext",
    "moduleResolution": "bundler",
    "allowSyntheticDefaultImports": true
  },

  "include": [
    "build/**/*.ts", // 不同于tsconfig.json, 写在这里的配置文件影响对node的提示,也就是vite服务器端的提示
    "vite.config.ts"
  ]
}
```



## global-var.d.ts
```ts
declare module '*.md' {
    import type { ComponentOptions } from 'vue'
    const component: ComponentOptions
    export default component
}

declare module '*.vue' {
    import type { DefineComponent } from 'vue'
    const component: DefineComponent<{}, {}, any>
    export default component
}
```



```ts

import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import { Plugin as importToCDN } from 'vite-plugin-cdn-import'

// https://vitejs.dev/config/
export default defineConfig({
    plugins: [
        vue(),
        importToCDN({
            modules: [
                {
                    name: 'vue',
                    var: 'Vue',
                    path: 'https://cdn.staticfile.org/vue/3.3.4/vue.global.min.js'
                }, {
                    name: 'vue-router',
                    var: 'VueRouter',
                    path: 'https://cdn.staticfile.org/vue-router/4.2.5/vue-router.global.min.js'
                },
                {
                    name: 'vue-demi',
                    var: 'VueDemi',
                    path: 'https://cdn.staticfile.org/vue-demi/0.14.6/index.iife.min.js'
                },
                {
                    name: 'echarts',
                    var: 'echarts',
                    path: 'https://cdn.staticfile.org/echarts/5.4.3/echarts.min.js'
                },
                {
                    name: 'naive-ui',
                    var: 'naive',
                    path: 'https://cdn.staticfile.org/naive-ui/2.34.4/index.prod.min.js'
                },
                {
                    name: 'gsap',
                    var: 'gsap',
                    path: 'https://cdn.staticfile.org/gsap/3.12.2/gsap.min.js'
                },
                {
                    name: 'pinia',
                    var: 'Pinia',
                    path: 'https://cdn.staticfile.org/pinia/2.1.6/pinia.iife.prod.min.js'
                },
            ]
        })
        // AutoImport({
        //     // 这里除了引入 vue 以外还可以引入pinia、vue-router、vueuse等，
        //     // 甚至你还可以使用自定义的配置规则，见 https://github.com/antfu/unplugin-auto-import#configuration
        //     imports: ['vue', 'vue-router', 'pinia', '@vueuse/core'],
        //     dts: './src/auto-imports.d.ts',
        //     // 第三方组件库的解析器
        // }),
        // Components({
        //     // dirs 指定组件所在位置，默认为 src/components
        //     // 可以让我们使用自己定义组件的时候免去 import 的麻烦
        //     dirs: ['src/components/'],
        //     dts: './src/auto-components.d.ts',
        //     // 配置需要将哪些后缀类型的文件进行自动按需引入
        //     extensions: ['vue', 'md'],
        //     // 解析的 UI 组件库，这里以 Element Plus 和 Ant Design Vue 为例
        //     resolvers: [NaiveUiResolver()],
        // }),
    ],
    // alias: {}
    resolve: {
        alias: {
        }
    }
})


```