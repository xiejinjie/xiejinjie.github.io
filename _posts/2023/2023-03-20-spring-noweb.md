---
layout: post
title: "Springboot非Web模式启动"
date: 2023-03-04 10:10:00 +0800
categories: tech
---

SpringBoot可以使用SpringApplicationBuilder构建启动参数，通过.web可以设置非web模式，如下：

```
@SpringBootApplication
public class App {
    private static final Logger logger = LoggerFactory.getLogger(App.class);

    public static void main(String[] args) {
        new SpringApplicationBuilder()
                .sources(App.class)
                .web(WebApplicationType.NONE)
                .headless(false)
                .run(args);
        logger.info("(♥◠‿◠)ﾉﾞ  应用启动成功   ლ(´ڡ`ლ)ﾞ  ");
    }
}
```

其中有配置headless为false，这是针对需要使用awt的场景。springboot默认为非交互式的，禁止启动awt。