# Mybatis总结

## XML配置

### 标签

resultType，指定返回类，必须等于返回类。 

resultMap，对返回数据进行字段映射。内部可设置一对一、一对多。且返回类可以是映射字段的对象本身或对象的列表。



### 其他

```text
附：XML转义字符
&lt;     	<   	小于号   
&gt;     	>   	大于号   
&amp;     	&   	和   
&apos;     	’   	单引号   
&quot;     	"   	双引号
```



> Invalid bound statement \(not found\)

检查：mapper.xml；mapper接口；target打包文件；config是否加载；4步





