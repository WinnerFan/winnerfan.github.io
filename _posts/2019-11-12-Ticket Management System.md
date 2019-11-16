### 票券管理系统

#### 需求
Redis展示账户统计表和明细表

#### 设计
JDK使用Redis，完成增删改查

### Redis

#### 配置
- port：端口
- logfile：配置日志地址
- dir：配置工作地址，用于存放DB和Appendonly文件
- slaveof：主从配置
- masterauth：主从密码，同时修改masterauth_s
- requirepss：登录密码，同时修改requirepass_s

#### 使用
- JedisPoolConfig
    - maxTotal：资源池最大连接数
    - maxIdle：资源池最大空闲连接数
    - minIdle：资源池最小空闲连接数
    - testOnBorrow：向资源池借用连接时，是否做有效性检测(ping)
    - testOnReturn：向资源池归还连接时，是否做有效性检测(ping)
    - testWhileIdle：空闲资源是否监控 
- JedisPool
    - pool.getResource()：得到连接Jedis
    - jedis.pinelined()：开启管道，顺序，但**不是原子操作**(允许个别失败)
    - pipeline.hgetAll(key)：操作(通过管道)
    - pipeline.syncAndReturnAll()：执行管道
    - pipeline.close：关闭管道
    - jedis.get(key)：操作(不通过管道)，是**原子操作**
    - jedis.close：关闭连接
- lua脚本
    - jedis.evalsha(sha, keys, values)：sha为String，keys和values为list，是**原子操作**
    - **Twemproxy**：twitter开源，第一个key进行分片操作
    - lua脚本中redis.call('set', key, val);

### Bean类
set所有值

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
            //field.getModifiers();//会累加public=1, private=2, protected=4, static=8, final=16, synchronized=32, volatile=64, transient=128, native=256, interface=512, abstract=1024, strict=2048
	        field.setAccesible(true);//访问private字段
	        field.set(obj, map.get(field.getName()));
	    }
    }
    return obj;
}
```

