---
title: "vite-cdn"
date: "2023-09-24T21:00:00+08:00"
categories: ["代码", "前端", "CSS", "CDN" ]
tags: [ "前端", "CSS", "CDN"]
---

# vite-cdn 

## 需求
加速vite构建

---

## 解决方案

从以下cdn网站中找出对应的地址

BootCDN: www.bootcdn.cn

七牛云: www.staticfile.org
```ts
[
    // https://cdn.staticfile.org/vue/2.7.9/vue.min.js
    'https://cdn.staticfile.org/vue/3.3.4/vue.global.min.js', 
    
    'https://cdn.staticfile.org/gsap/3.12.2/gsap.min.js',
    'https://cdn.staticfile.org/vue-demi/0.14.6/index.iife.min.js',
    'https://cdn.staticfile.org/vue-router/4.2.5/vue-router.global.min.js',
    'https://cdn.staticfile.org/naive-ui/2.34.4/index.prod.min.js',
    'https://cdn.staticfile.org/echarts/5.4.3/echarts.min.js',
    'https://cdn.staticfile.org/pinia/2.1.6/pinia.iife.prod.min.js',
    'https://cdn.staticfile.org/vuetify/3.3.17/vuetify.min.js',
    'https://cdn.staticfile.org/flv.js/1.6.2/flv.min.js',
    'https://cdn.staticfile.org/axios/1.5.0/axios.min.js',
    'https://cdn.staticfile.org/highlight.js/11.8.0/es/highlight.min.js',
    'https://cdn.staticfile.org/markdown-it/13.0.1/markdown-it.min.js',
    'https://cdn.staticfile.org/jspdf/2.5.1/jspdf.umd.min.js',
    'https://cdn.staticfile.org/xlsx/0.18.5/xlsx.full.min.js',

    'https://cdn.staticfile.org/vuetify/3.3.17/vuetify.css',
    'https://cdn.staticfile.org/vuetify/3.3.17/vuetify-labs.min.css',
    'https://cdn.staticfile.org/react/18.2.0/cjs/react.production.min.js',
]
```

360: cdn.baomitu.com

字节跳动: cdn.bytedance.com

unpkg.com

```ts
[
    "https://unpkg.com/@vueuse/shared",
    "https://unpkg.com/@vueuse/core", 
]
```
    


```ts
import importToCDN from 'vite-plugin-cdn-import'
export default () => {
    return defineConfig({
        plugins: [
            vue(),
            vueJsx(),
            AutoImport({
                imports: [
                    'vue',
                    'vue-router',
                ]
            }),
            // visualizer(),
            importToCDN({
                modules: [
                    {
                        name: 'vue',
                        var: 'Vue',
                        path: 'https://cdn.jsdelivr.net/npm/vue@3.3.4'
                    },
                    {
                        name:'vue-demi',
                        var:'VueDemi',
                        path:'https://cdn.jsdelivr.net/npm/vue-demi@0.14.6'
                    },
                    {
                        name: 'echarts',
                        var: 'echarts',
                        path: 'https://cdn.jsdelivr.net/npm/echarts@5.4.3'
                    },
                    {
                        name: 'naive-ui',
                        var: 'naiveUI',
                        path: 'https://cdn.jsdelivr.net/npm/naive-ui@2.34.4'
                    },
                    {
                        name: 'gsap',
                        var: 'gsap',
                        path: 'https://cdn.jsdelivr.net/npm/gsap@3.12.2'
                    },
                    {
                        name: 'pinia',
                        var: 'Pinia',
                        path:'https://cdn.jsdelivr.net/npm/pinia@2.1.3'
                    },
                ]
            })
        ],
        resolve: {
            alias: {
                '@': fileURLToPath(new URL('./src', import.meta.url))
            }
        },
        build: {
            rollupOptions: {
                output: {
                    manualChunks(id: string) {
                        const t2 = id.split('/node_modules/')
                        if (t2.length == 3) {
                            const strings = t2[2].split('/')
                            return strings[0]
                        }
                    },
                },
            },
        },
        base: './'
    })
}
```