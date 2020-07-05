# mybatis-自动生成代码管理 最佳实践

<!--more-->

项目中的最佳实践是把自动生成的代码和手工写的代码放在不同的地方，xml和Mapper都分开放，方便管理。

比如：
* src/main/java/xxx/mapper/auto 
* src/main/java/xxx/mapper/manual

![pop](/images/202004/2020年4月8日10点25分2.png "image")

xml也是类似的，这样就不用担心重新生成时的覆盖问题了。
![pop](/images/202004/2020年4月8日10点25分1.png "image")


然后manual里可以继承auto中的结果映射关系, 示例:
```xml
<resultMap id="BaseResultMap" 
           type="com.company.xxx.model.SysPermission"
           extends="com.company.xxx.mapper.auto.SysPermissionMapper.BaseResultMap">
</resultMap>
```

## References

* [1](https://time.geekbang.org/course/detail/156-83259)
