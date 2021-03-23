---
layout: post
title: Spring Technique
tags: Spring
---

## Spring Technique

Spring容器启动执行

```
//ApplicationContextEvent监听的是项目容器事件（启动、停止等）
//ContextRefreshedEvent是容器刷新（启动或说初始化）
@Component
public class Start implements ApplicationListener<ApplicationContextEvent> {
    public void onApplicationEvent(ApplicationContextEvent event) {
        if (event.getApplicationContext().getParent() == null) {
        
        }
    }
}
```