## ServiceLoader的使用

### ServiceLoader介绍
ServiceLoader这个类来从配置文件中加载子类或者接口的实现类，主要是从META-INF/services这个目录下的配置文件加载给定接口或者基类的实现，ServiceLoader会根据给定的类的full name来在META-INF/services下面找对应的文件，在这个文件中定义了所有这个类的子类或者接口的实现类，返回一个实例。
![](https://ws4.sinaimg.cn/large/006tKfTcly1ftflh5todgj30at04p3yz.jpg)
在此文件中定义一个或者多个实现类，如：

```java
com.pkuhit.iihip.rce.core.MessageRepositoryImp1
com.pkuhit.iihip.rce.core.MessageRepositoryImp2
```



### 调用方法
```java
//need to define related class full name in /META-INF/services/....
ServiceLoader<IService> serviceLoader = ServiceLoader.load(IService.class);
for (IService service : serviceLoader) {
    //循环出来的各个接口实现类
}
```



### 工具类例子

```java
import java.util.Iterator;
import java.util.ServiceLoader;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
public class ServiceLoaderUtils
{
    protected static Logger logger = LoggerFactory.getLogger(ServiceLoaderUtils.class);
/**
 * 使用Java标准的ServiceLoader方式加载给定Service的实现。
 * 加载时优先使用系统属性中设定的实现类。如果没有设定，则按照
 * Java标准ServiceLoader加载方式遍历系统中找到的实现，选择
 * 第一个返回。
 * 
 * 系统属性可以通过启动java命令时的-D×××=yyy的方式来设定，也
 * 可以通过编程的方式直接调用System.setProperty来设定。
 * 
 * @param clazz
 * @return
 */
@SuppressWarnings("unchecked")
public static <T> T getService(Class<T> clazz)
{
    String engineClazz = System.getProperty(clazz.getName());
    try
    {
        if(engineClazz != null)
            return (T) Class.forName(engineClazz).newInstance();
    }
    catch (InstantiationException | IllegalAccessException | ClassNotFoundException e)
    {
        throw new RuntimeException(e);
    }
    T instance = null;
    ServiceLoader<T> loader = ServiceLoader.load(clazz);
    Iterator<T> it = loader.iterator();
    while(it.hasNext())
    {
        instance = it.next();
        break;
    }
    if(instance == null)
        logger.error("没有找到{}的实现。", clazz.getName());
    return instance;
  }
}
```



