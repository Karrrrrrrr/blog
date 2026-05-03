---
title: "Github提交日历"
date: 2025-05-08T12:00:00+08:00
---
# 模拟github提交日历
## 代码
 
```html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8"/>
    <title>GitHub 风格提交日历 -- 过去一年</title>
    <style>
        #app {
            font-family: Arial, sans-serif;
            /*max-width: 900px;*/
            margin: 20px auto;
        }

        .calendar {
            display: flex;
            gap: 4px;
            user-select: none;
        }

        .week {
            display: flex;
            flex-direction: column;
            gap: 4px;
        }

        .day {
            width: var(--day-width);
            aspect-ratio: 1/1;
            border-radius: 3px;
            cursor: default;
            transition: background-color 0.3s ease;
            /* 默认浅灰 */
            background-color: #ebedf0;
            position: relative;
        }

        .day.empty {
            background-color: transparent;
            cursor: default;
        }

        .day.in-year:after {
            transition: opacity 0.2s ease;
            content: attr(data-tip);
            position: absolute;
            bottom: 125%;
            left: 50%;
            transform: translateX(-50%);
            background: rgba(0, 0, 0, 0.75);
            color: white;
            padding: 3px 6px;
            border-radius: 4px;
            font-size: 12px;
            white-space: nowrap;
            pointer-events: none;
            z-index: 10;
            opacity: 0;
        }

        .day.in-year:hover::after {
            opacity: 1;
        }

        /* 顶部月份行 */
        .months {
            display: flex;
            gap: 4px;
            margin-bottom: 4px;
        }

        .month-label {
            font-size: 12px;
            color: #555;
            text-align: center;
            user-select: none;
            line-height: 14px;
            /* 宽度与日格宽度加间隙对应 */
            display: inline-block;
        }

        /* Legend */
        .legend {
            display: flex;
            align-items: center;
            margin-top: 12px;
            gap: 4px;
            font-size: 12px;
            color: #555;
        }

        .legend-color {
            width: 14px;
            height: 14px;
            border-radius: 3px;
            background-color: #ebedf0;
            border: 1px solid #ddd;
        }

    </style>
</head>
<body>

<div id="app">
    <submit-calender :start-date="beginDate1" :end-date="endDate1"></submit-calender>
    <submit-calender :start-date="beginDate2" :end-date="endDate2"></submit-calender>
</div>

<!-- Vue3 + Dayjs -->
<script src="https://lf3-cdn-tos.bytecdntp.com/cdn/expire-1-M/vue/3.2.31/vue.global.min.js"></script>
<script src="https://lf9-cdn-tos.bytecdntp.com/cdn/expire-1-M/dayjs/1.10.8/dayjs.min.js"></script>

<script>
    const {
        createApp,
        reactive,
        computed
    } = Vue;

    // JS setup中新增代替插件的判断函数
    function isSameOrBefore(d1, d2) {
        return d1.isBefore(d2) || d1.isSame(d2);
    }

    function isSameOrAfter(d1, d2) {
        return d1.isAfter(d2) || d1.isSame(d2);
    }

    const d = dayjs()
    d.__proto__.isSameOrBefore = function (rhs) {
        return isSameOrBefore(this, rhs)
    }
    d.__proto__.isSameOrAfter = function (rhs) {
        return isSameOrAfter(this, rhs)
    }

    const SubmitCalender = {
        props: ['endDate', 'startDate', 'dayWidth', 'gap'],
        setup(props) {
            const dayWidth = Number(props.dayWidth || 15);
            const gap = Number(props.gap || 4);

            // 结束日期是今天
            const endDate = dayjs(props.endDate).startOf('day');
            // 365天前的日期
            const startDate = dayjs(props.startDate).startOf('day')

            // 模拟随机提交数据，仅针对时间范围内
            const contributionsData = reactive({});
            let dateCursor = startDate;
            while (dateCursor.isBefore(endDate) || dateCursor.isSame(endDate)) {
                const key = dateCursor.format('YYYY-MM-DD');
                // 激活数据，模拟0~10次提交
                contributionsData[key] = Math.floor(Math.random() * 10);
                dateCursor = dateCursor.add(1, 'day');
            }

            // 计算日历开始：startDate最早所在周的周日（前推到周日开始）
            let calendarStart = startDate;
            while (calendarStart.day() !== 0) {
                calendarStart = calendarStart.subtract(1, 'day');
            }
            // 计算日历结束：endDate所在周的周六（后推到周六结束）
            let calendarEnd = endDate;
            while (calendarEnd.day() !== 6) {
                calendarEnd = calendarEnd.add(1, 'day');
            }

            const weeksCount = Math.ceil(calendarEnd.diff(calendarStart, 'week', true));

            // 构造weeks数组，每周列，7天一列，包含当天日期，是否在范围内标记inRange
            const weeks = computed(() => {
                const arr = [];
                for (let w = 0; w <= weeksCount; w++) {
                    const week = [];
                    for (let d = 0; d < 7; d++) {
                        const date = calendarStart.add(w, 'week').add(d, 'day');
                        const dateStr = date.format('YYYY-MM-DD');
                        const inRange = date.isSameOrAfter(startDate) && date.isSameOrBefore(endDate);
                        const count = inRange ? (contributionsData[dateStr] ?? 0) : 0;
                        week.push({
                            date: dateStr,
                            count,
                            inRange
                        });
                    }
                    arr.push(week);
                }
                return arr;
            });

            // 颜色等级 - GitHub样式
            const colorScale = [
                '#ebedf0',
                '#c6e48b',
                '#7bc96f',
                '#239a3b',
                '#196127'
            ];

            function getColor(count) {
                if (count <= 0) return colorScale[0];
                if (count <= 2) return colorScale[1];
                if (count <= 4) return colorScale[2];
                if (count <= 7) return colorScale[3];
                return colorScale[4];
            }

            function formatTooltip(day) {
                const d = dayjs(day.date);
                return `${d.year()}年${d.month() + 1}月${d.date()}日: ${day.count} 次提交`;
            }

            // 计算月份左对齐显示
            // 思路：找calendarStart后的每周第一天，根据其所属月份确定标签起点（第几周）
            const months = computed(() => {
                // 取每周第一天的日期列表
                const weekFirstDays = [];
                for (let w = 0; w <= weeksCount; w++) {
                    weekFirstDays.push(calendarStart.add(w, 'week'));
                }

                // 找出月份中第一个“显示在calendar里的日期”的周索引
                // 遍历过去一年内出现的月份（起始至结束）
                const monthInfos = [];
                let cursor = dayjs(startDate);
                let lastMonth = null;
                const seenMonths = new Set();

                while (cursor.isSameOrBefore(endDate)) {
                    const monthStr = cursor.format('YYYY-MM');
                    if (!seenMonths.has(monthStr)) {
                        // 找这个月第一天在calendar中的周索引
                        const firstDayOfMonth = dayjs(monthStr + '-01');
                        // 先确保首日不早于calendarStart，否则用calendarStart
                        let labelDate = firstDayOfMonth.isBefore(calendarStart) ? calendarStart : firstDayOfMonth;

                        // 只考虑labelDate在范围内
                        if (labelDate.isAfter(calendarEnd)) break;

                        // 找周索引
                        let startWeek = 0;
                        for (let i = 0; i < weekFirstDays.length; i++) {
                            const weekStart = weekFirstDays[i];
                            const weekEnd = weekFirstDays[i + 1] ? weekFirstDays[i + 1].subtract(1, 'day') : calendarEnd;
                            if (labelDate.isSameOrAfter(weekStart) && labelDate.isSameOrBefore(weekEnd)) {
                                startWeek = i;
                                break;
                            }
                        }

                        monthInfos.push({
                            name: labelDate.format('M月'),
                            startWeek
                        });

                        seenMonths.add(monthStr);
                    }
                    cursor = cursor.add(1, 'month').startOf('month');
                }

                return monthInfos;
            });

            return {
                startDate,
                endDate,
                weeks,
                getColor,
                colorScale,
                formatTooltip,
                months,
                dayWidth,
                gap
            };
        },
        template: `
          <div style="width: fit-content">
            <div class="months" style="position: relative; height: 20px;">
              <div v-for="month in months" :key="month.name"
                   class="month-label"
                   :style="{ position: 'absolute', left: (month.startWeek * (dayWidth + gap)) + 'px',  }">
                {{ month.name }}
              </div>
            </div>
            <div class="calendar" :style="{'--day-width': dayWidth + 'px'}" style="width: fit-content">
              <div v-for="(week, wIndex) in weeks" :key="wIndex" class="week">
                <div v-for="day in week" :key="day.date"
                     :class="['day', {'empty': !day.inRange}, {'in-year': day.inRange}]"
                     :style="{backgroundColor: day.inRange ? getColor(day.count) : 'transparent', }"
                     :data-tip="day.inRange ? formatTooltip(day) : null">
                </div>
              </div>
            </div>
            <div style="display: flex; justify-content: right">
              <div class="legend" style="margin-top: 16px;">
                <span>少</span>
                <span class="legend-color" :style="{backgroundColor: colorScale[0]}"></span>
                <span class="legend-color" :style="{backgroundColor: colorScale[1]}"></span>
                <span class="legend-color" :style="{backgroundColor: colorScale[2]}"></span>
                <span class="legend-color" :style="{backgroundColor: colorScale[3]}"></span>
                <span class="legend-color" :style="{backgroundColor: colorScale[4]}"></span>
                <span>多</span>
              </div>
            </div>
          </div>`
    }

    createApp({
        components: {
            SubmitCalender
        },
        setup() {

            const now = dayjs()
            const endDate1 = now.startOf('day')
            const beginDate1 = endDate1.subtract(364, 'day').startOf('day')
            const endDate2 = now.startOf('year').subtract(-364, 'day')
            const beginDate2 = now.startOf('year')
            return {
                endDate1,
                beginDate1,
                endDate2,
                beginDate2
            }
        }
    }).mount('#app');
</script>

</body>
</html>


```

## 预览效果

<div style="width: 100%;overflow: hidden;">
    <iframe src="/template/Github Calander.html" style="width: 100%;border: 0; height: 400px"></iframe>
</div>