---
title: "Vue3奇技淫巧-Hook篇"
date: "2022-12-29T22:54:10+08:00" 
---

## 前言

总有一些常用hook，很常见但是网上不一定有， 所以在此记录一些个人认为常用的
 

## 文件选择器
> 前端选择文件总是依赖input， 常常需要自定义

```typescript
import {
    onMounted,
    onUnmounted
} from 'vue' 
export const useFileSelector = () => {
    const selector: HTMLInputElement = document.createElement('input')

    selector.style.display = 'none'
    selector.type          = 'file'
    const chooseFile       = () => {
        return new Promise((resolve, reject) => {
                const onChange = () => {
                    const file = selector.files?.[0] || null
                    resolve(file)
                    selector.removeEventListener('change', onChange)
                }
                selector.addEventListener('change', onChange)
                selector.click()
            }
        )
    }
    const toURL            = (file: File) => {
        return URL.createObjectURL(new Blob([ file ]))
    }
    onMounted(() => document.body.append(selector))
    onUnmounted(() => selector.remove())

    return {
        chooseFile,
        toURL
    }
}
```

> 使用如上代码， 暴露一个chooseFile函数，
> 调用chooseFile函数，
> 等待异步返回的结果就是用户选择的文件

> 这里需要注意，如果用户没有选择文件，
> 而是关闭的对话框，
> 会返回null，所以需要特判



## UseCharts

```ts
import { onBeforeUnmount, onMounted, ref, Ref, watch } from "vue";
import { EChartsOption, type EChartsType, init } from "echarts";
import { useThemeStore } from "@/store";
import { useResizeObserver } from "@vueuse/core";

export const useCharts = (data: Ref, fn: (data: any) => EChartsOption) => {

    const themeStore = useThemeStore()
    const container = ref<HTMLElement>()
    let chartType: EChartsType | null = null

    const initChart = () => {
        if (container.value) {
            chartType = init(container.value, themeStore.isLight ? undefined : 'dark', { renderer: 'svg' })
        }

    }

    onMounted(() => {
        initChart()
        render()
    })
    const destroy = () => {
        if (chartType) {
            chartType.clear()
            chartType.dispose()
            chartType = null
        }
    }
    watch(themeStore, () => {
        destroy()
        initChart()
        render()
    })
    const resize = () => {
        chartType?.resize()
    }

    useResizeObserver(container, () => {
        resize()
    })

    const render = () => {
        if (!chartType || !fn || !data?.value) return
        const options = fn(data.value)
        if (!options) return;
        options.backgroundColor = ''
        chartType.setOption(options)

    }

    watch(data, () => {
        render()
    })
    onBeforeUnmount(() => {
        destroy()
    })
    return { container }

}
```



## axios 
```ts
import type { AxiosRequestConfig } from "axios";
import axios from "axios";
import { getToken } from "@/utils/authorization";
import { useMessageBox } from "@/utils/message";

interface HTTPResponse<T = any> {
    code: number,
    data: T,
    message: string
}

const createRequestInstance = (baseURL: string) => {
    const instance = axios.create({
        baseURL, // base url for proxy
        timeout: 1000 * 5 // wait for 5 second
    })

    const request = async <T = any>(config: AxiosRequestConfig): Promise<HTTPResponse<T>> => {
        const msg = useMessageBox()

        // const tokenKey =
        const token = getToken()
        // console.log(token)
        if (!config.headers) {
            config.headers = {}
        }
        if (token) {
            // console.log('set???')
            config.headers.Authorization = `${ token }`
        }
        const resp = await instance(config)
        if (resp.data.code == 200) {
            return resp.data
        } else {
            if (resp.data.message) {
                msg?.error(resp.data.message)
            }
            throw (new Error(resp.data.message || 'Unexpect error'))
        }
    }
    return { request }
}
const { request: v1 } = createRequestInstance("/api/v1")

export {
    v1
}
```