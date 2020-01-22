---
layout: post
title: Ticket Management System
tags: Project
---
## 票券管理系统

### 需求

Redis展示账户统计表和明细表

### 设计

JDK使用Redis，完成增删改查

## Redis

### 配置

- port：端口
- logfile：配置日志地址
- dir：配置工作地址，用于存放DB和Appendonly文件
- slaveof：主从配置
- masterauth：主从密码，同时修改masterauth_s
- requirepss：登录密码，同时修改requirepass_s

### 使用

- JedisPoolConfig
    - maxTotal：资源池最大连接数
    - maxIdle：资源池最大空闲连接数
    - minIdle：资源池最小空闲连接数
    - testOnBorrow：向资源池借用连接时，是否做有效性检测(ping)
    - testOnReturn：向资源池归还连接时，是否做有效性检测(ping)
    - testWhileIdle：空闲资源是否监控 
- JedisPool
    - pool.getResource()：得到连接Jedis
    - jedis.pinelined()：开启管道，顺序，但**不是原子操作允许失败，而且允许穿插命令**
    - pipeline.hgetAll(key)：操作(通过管道)
    - pipeline.syncAndReturnAll()：执行管道
    - pipeline.close：关闭管道
    - jedis.get(key)：操作(不通过管道)，是**原子操作**
    - jedis.close：关闭连接
- lua脚本：两种方式，**非原子操作中间错误则退出lua脚本，但无其他命令穿插**
	- jedis.eval(lua, keys, values)
    - jedis.evalsha(jedis.scriptLoad(lua), keys, values)：jedis.scriptLoad(lua)得到sha值，keys和values为list
    - **Twemproxy**：twitter开源，第一个key进行分片操作
    - lua脚本中redis.call('set', key, val);

- Bean中set所有值

```
public static Object mapToBean (Map<String, String> map, Class<?> clazz) {
    if(map == null)
        return null;
    Object obj = clazz.newInstance();
	for( ;clazz != Object.class; clazz = clazz.getSuperClass())//循环向上
	    Field[] fields = obj.getDeclaredFields();//所有field，不含父类继承的
	    for(Field field : fields) {
	        if(map.get(field.getName()) == null)
	            countinue;
            // field.getModifiers(); // 得到修饰词，可累加
	        field.setAccesible(true);// 访问private字段
	        field.set(obj, map.get(field.getName()));
	    }
    }
    return obj;
}
```
