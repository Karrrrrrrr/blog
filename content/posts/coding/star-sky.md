---
title: "js实现星空动画"
date: 2025-05-02T22:00:00+08:00
---
# 星空
灵感来自 <a href="https://lobehub.com/zh">lobehub</a>
![img.png](/template/img/lobehub.png)
## Canvas实现
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>星空动画</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
        }

        canvas {
            display: block;
        }
    </style>
</head>

<body>
<canvas id="starry-sky"></canvas>
<script>
    const canvas = document.getElementById('starry-sky');
    const ctx = canvas.getContext('2d');
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;

    const numStars = 150;
    const stars = [];

    class Star {
        constructor() {
            this.x = Math.random() * canvas.width;
            this.y = Math.random() * canvas.height;
            this.size = (Math.random() * 3 + 1) * .4;
            const colors = [
                '#fff',
                '#f0f',
                // '#0ff',
                '#ff0'
            ];
            this.color = colors[Math.floor(Math.random() * colors.length)];
            this.speedX = (Math.random() * .5 + 1) / 3
            this.speedY = (Math.random() * .5 + 1) / 3
            console.log(this.speedX, this.speedY)
        }

        move() {
            this.x += this.speedX;
            this.y -= this.speedY;
            if (this.x > canvas.width) {
                this.x = 0;
            }
            if (this.y < 0) {
                this.y = canvas.height;
            }
        }

        draw() {
            ctx.beginPath();
            ctx.arc(this.x, this.y, this.size, 0, Math.PI * 2);
            ctx.fillStyle = this.color;
            ctx.fill();
        }
    }

    function createStars() {
        for (let i = 0; i < numStars; i++) {
            stars.push(new Star());
        }
    }

    function drawBackground() {
        const gradient = ctx.createLinearGradient(0, 0, 0, canvas.height);
        gradient.addColorStop(0, '#000000');
        gradient.addColorStop(.25, 'rgb(21,18,27)');
        gradient.addColorStop(.5, 'rgba(32,28,42,1)');
        gradient.addColorStop(1, 'rgb(44,52,99)');
        // gradient.addColorStop(1, '#445299');
        // gradient.addColorStop(1, '#1a1a4d');
        ctx.fillStyle = gradient;
        ctx.fillRect(0, 0, canvas.width, canvas.height);
    }

    function animate() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        drawBackground();
        stars.forEach(star => {
            star.move();
            star.draw();
        });
        requestAnimationFrame(animate);
    }

    createStars();
    animate();

    // window.addEventListener('resize', () => {
    //     canvas.width = window.innerWidth;
    //     canvas.height = window.innerHeight;
    // });
</script>
</body>

</html>
```
<div style="width: 100%">
    <iframe src="/template/starsky-canvas.html" style="width: 100%; border: 0; height: 500px"></iframe>
</div>
## Dom实现

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>星空动画</title>
    <style>
        #starry-sky {
            width: 1920px;
            height: 1080px;
            background: linear-gradient(to bottom, #000000, #1a1a4d);
            position: relative;
            overflow: hidden;
        }

        .star {
            border-radius: 50%;
            position: absolute;
            box-shadow: 0 0 5px #fff;
        }
    </style>
</head>

<body>
    <div id="starry-sky"></div>
    <script>
        const starrySky = document.getElementById('starry-sky');
        const numStars = 200; // 星星数量

        function createStars() {
            for (let i = 0; i < numStars; i++) {
                const star = document.createElement('div');
                star.classList.add('star');
                // 随机初始位置
                const x = Math.random() * window.innerWidth;
                const y = Math.random() * window.innerHeight;
                star.style.left = x + 'px';
                star.style.top = y + 'px';
                // 随机大小
                const size = Math.random() * 3 + 1;
                star.style.width = size + 'px';
                star.style.height = size + 'px';
                // 随机颜色
                const colors = ['#fff', '#f0f', '#0ff', '#ff0'];
                const randomColor = colors[Math.floor(Math.random() * colors.length)];
                star.style.backgroundColor = randomColor;
                // 随机移动速度
                const speedX = Math.random() * 2 + 1;
                const speedY = Math.random() * 2 + 1;
                star.dataset.speedX = speedX;
                star.dataset.speedY = speedY;
                starrySky.appendChild(star);
            }
        }

        function moveStars() {
            const stars = document.querySelectorAll('.star');
            stars.forEach((star) => {
                const x = parseInt(star.style.left) + parseFloat(star.dataset.speedX);
                const y = parseInt(star.style.top) - parseFloat(star.dataset.speedY);
                if (x > window.innerWidth) {
                    star.style.left = '0px';
                } else {
                    star.style.left = x + 'px';
                }
                if (y < 0) {
                    star.style.top = window.innerHeight + 'px';
                } else {
                    star.style.top = y + 'px';
                }
            });
        }

        createStars();
        setInterval(moveStars, 50);
    </script>
</body>

</html>
```

<div style="width: 100%">
    <iframe src="/template/starsky-dom.html" style="width: 100%; border: 0; height: 500px"></iframe>
</div>