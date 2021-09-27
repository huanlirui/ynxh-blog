loading:
<h1 data-text="玩命加载中...">玩命加载中...</h1>
h1 {
    position: relative;
    color: rgba(0, 0, 0, .3);
    font-size: 5em
}
h1:before {
    content: attr(data-text);
    position: absolute;
    overflow: hidden;
    max-width: 7em;
    white-space: nowrap;
    color: #fff;
    animation: loading 8s linear;
}
@keyframes loading {
    0% {
        max-width: 0;
    }
}


链接：https://zhuanlan.zhihu.com/p/391561733


聚光灯：
   <h1 dot-light="LIZHENWEN">LIZHENWEN</h1>

   body {
    margin: 0;
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: #222;
    min-height: 100vh;
}

h1 {
    font-size: 9rem;
    font-family: Helvetica;
    letter-spacing: -.3rem;
    color: #333;
    position: relative;
}

h1::after {
    content: attr(dot-light);
    position: absolute;
    top: 0;
    left: 0;
    color: transparent;
    clip-path: ellipse(100px 100px at 0% 50%);
    animation: SpotLight 5s infinite;
    background-image: url("https://images.unsplash.com/photo-1568279898331-4870e84677dd?ixid=MnwxMjA3fDB8MHxzZWFyY2h8MTh8fGxpbmVhcnxlbnwwfHwwfHw%3D&ixlib=rb-1.2.1&auto=format&fit=crop&w=800&q=60");
    background-size: 150%;
    background-position: center center;
    -webkit-background-clip: text;
    background-clip: text;
}

@keyframes SpotLight {
    0% {
        clip-path: ellipse(100px 100px at 0% 50%);
    }

    50% {
        clip-path: ellipse(100px 100px at 100% 50%);
    }

    100% {
        clip-path: ellipse(100px 100px at 0% 50%);
    }
}