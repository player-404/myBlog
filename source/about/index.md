---
title: about
date: 2024-06-13 11:29:37
layout: about
---
   <style>
        .about-skill {
            width: 100%;
            display: flex;
            justify-content: center;
            align-items: center;
        }

        .skill {
            list-style: none;
            margin: 0;
            padding: 0;
            width: 80%;
        }

        .line {
            display: flex;
            flex-direction: row;
            justify-content: space-around;
            align-items: center;
            margin: 50px 0;
        }

        .logo {
            width: 10%;
            height: 10%;
        }

        .logo>img {
            width: 100%;
            height: 100%;
        }

        .title {
            width: 15%;
            height: 15%;
            font-size: 28px;
            text-align: center;
            font-weight: 600;
        }
        .project {
            width: 100%;
            margin: 100px 0;
        }
        .pro-title {
            font-size: 28px;
            text-align: center;
        }
        .myicon {
            font-size: 28px;
            margin-right: 4px;
        }
        a {
            color: #34495e;
            text-decoration: none;
        }
        a:hover, a:visited, a:link, a:active {
            color: #34495e;
        }
        .project-name {
            font-weight: 600;
            font-size: 16px;
        }
        .project-desc {
            color:#a2a2a2;
            font-size: 14px;
        }
        .item-line {
            display: flex;
            justify-content: space-around;
            margin: 20px 0;
        }
        .item-con {
            width: 50%;
        }
        .project-cover {
            width: 60px;
            height: 60px;
            margin: 0 auto 10px;
            border-radius: 50%;
            overflow: hidden;
        }
        .project-cover>img {
            width: 100%;
            height: 100%;
        }
        .project-name {
            text-align: center;
            margin-bottom: 10px;
        }
        .project-desc {
            text-align: center;
        }
    </style>

<body>
    <div class="about-skill">
        <ul class="skill">
            <li class="line">
                <div class="logo">
                    <img src="img/vue.png" alt="Vue">
                </div>
                <div class="logo">
                    <img src="./img/pinia.png" alt="Pinia">
                </div>
                <div class="logo">
                    <img src="img/logo-axios.png" alt="Axios">
                </div>
                <div class="logo">
                    <img src="img/logo-uniapp.png" alt="Uniapp">
                </div>
                <div class="logo">
                    <img src="img/logo-vite.png" alt="Vite">
                </div>
            </li>
            <li class="line">
                <div class="logo">
                    <img src="img/logo-webpack.png" alt="Webpack">
                </div>
                <div class="logo">
                    <img src="img/logo-ts.png" alt=""TypeScript">
                </div>
                <div class="title">
                    我会
                </div>
                <div class="logo">
                    <img src="img/logo-elementui.png" alt="element">
                </div>
                <div class="logo">
                    <img src="img/logo-node.png" alt="Node">
                </div>
            </li>
            <li class="line">
                <div class="logo">
                    <img src="img/logo-express.png" alt="Express">
                </div>
                <div class="logo">
                    <img src="img/logo-mongodb.png" alt="MongoDB">
                </div>
            </li>
        </ul>
    </div>
    <div class="project">
        <div class="pro-title">
            <i class="iconfont icon-project myicon"></i>
            我的项目
        </div>
        <ul class="project-items">
            <li class="item-line">
                <div class="item-con">
                    <div class="project-cover">
                        <img src="img/project.png" alt="pic"/>
                    </div>
                    <div class="project-name">
                        <a href="https://player-404.github.io/geoJson-china/">地图GeoJson数据下载</a>
                    </div>
                    <div class="project-desc">
                        提供最新的中国地图GeoJson数据下载，包括省级、市级、县级以及区级。
                    </div>
                </div>
                <div class="item-con">
                    <div class="project-cover">
                        <img src="img/project.png" alt="pic"/>
                    </div>
                    <div class="project-name">
                        <a href="https://player-404.github.io/login/">登录Demo</a>
                    </div>
                    <div class="project-desc">
                        一个登录界面demo，使用 animate.css 实现动画效果。
                    </div>
                </div>
            </li>
        </ul>
    </div>
</body>
