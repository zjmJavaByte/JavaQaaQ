**springboot接口参数时间转换、解决Date类型参数异常**

> 参考文章：[springboot接口参数时间转换、解决Date类型参数异常](https://mp.weixin.qq.com/s/PAfQLc_CogZaWpD0Lv61sA?)

### 默认情况

​		在默认情况下，如果前端传参为String类型，而Controller接口接收参数的类型是Date类型，在不加任何有关接收Date类型数据的配置的情况下，控制台出现以下异常：

```java
Failed to convert from type [java.lang.String] to type [java.util.Date] for value '2333333333'; nested exception is java.lang.IllegalArgumentException]]
```

​		也就是说，在Spring中，如果项目接口出现时间参数，前端传值时就容易出现问题，尤其是不同接口对应的前端传递的Date数据格式不同的情况。下面针对不同的情况来介绍一下解决办法

### 局部配置——使用`@DateTimeFormat`

​		第一种情况，仅需要对某个Bean类的Date类型字段进行转换，那么只需要在Bean类属性上增加`@DateTimeFormat()`注解，括号内 `pattern`为前端传递的日期格式。比如：

```java
@DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
Date createTime;
```

​		特点--缺陷：

​		按照如上配置，只能处理形如：`2018-11-2 2:22:2`这样固定格式的数据，无法处理时间戳和`2018-11-2`格式的数据。如果想要处理`2018-11-2`这样的时间，就必须把`pattern`改成`yyyy-MM-dd`。

​		可以对不同的属性赋予不同的`pattern`，但是对每个Date类型都要加上注解显得比较繁琐，也无法处理单个属性可能对应不同格式的值的情况。

​		更通用、有效的方式是定义一个时间转换类，并将其应用到所有的接口上。

### 全局配置——自定义时间转换器

分为两个步骤：

- 编写一个时间转换类，把可能的格式转换为Date类型 
- 通过某种方式，将时间转换应用到Spring的所有接口上

**时间转换类**

```java
import org.springframework.core.convert.converter.Converter;
import org.springframework.util.StringUtils;
import java.text.SimpleDateFormat;
import java.util.Date;
/**
 * 日期转换类
 * 将标准日期、标准日期时间、时间戳转换成Date类型
 */
public class DateConverter implements Converter<String, Date> {
    private static final String dateFormat = "yyyy-MM-dd HH:mm:ss";
    private static final String shortDateFormat = "yyyy-MM-dd";
    private static final String timeStampFormat = "^\\d+$";
    @Override
    public Date convert(String value) {
        if(StringUtils.isEmpty(value)) {
            return null;
        }
        value = value.trim();
        try {
            if (value.contains("-")) {
                SimpleDateFormat formatter;
                if (value.contains(":")) {
                    formatter = new SimpleDateFormat(dateFormat);
                } else {
                    formatter = new SimpleDateFormat(shortDateFormat);
                }
                return formatter.parse(value);
            } else if (value.matches(timeStampFormat)) {
                Long lDate = new Long(value);
                return new Date(lDate);
            }
        } catch (Exception e) {
            throw new RuntimeException(String.format("parser %s to Date fail", value));
        }
        throw new RuntimeException(String.format("parser %s to Date fail", value));
    }
}
```

**将时间转换类应用到接口上**

​		介绍两种方式：使用`@Component` + `@PostConstruct`或`@ControllerAdvice` + `@InitBinder`

​		第一种方式：`@Component` + `@PostConstruct`

​		首先介绍一下@PostConstruct注解，这个注解用于在生成对象时完成某些初始化操作，这些初始化操作依赖于依赖注入（DI）。我们可以定义一个配置类，通过@PostConstruct注解来实现时间转换机制的绑定。

​		执行顺序：Construct >> @Autowired >> @PostConstruct

​		注意：@PostConstruct注解不属于Spring，而是Java自带的。

​		下面来看示例代码：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.convert.support.GenericConversionService;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.support.ConfigurableWebBindingInitializer;
import org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter;
import javax.annotation.PostConstruct;
@Component
public class WebConfigBeans {
  @Autowired
  private RequestMappingHandlerAdapter handlerAdapter;
  @PostConstruct
  public void initEditableAvlidation() {
    ConfigurableWebBindingInitializer initializer = (ConfigurableWebBindingInitializer)handlerAdapter.getWebBindingInitializer();
    if(initializer.getConversionService()!=null) {
      GenericConversionService genericConversionService = (GenericConversionService)initializer.getConversionService();
      genericConversionService.addConverter(new DateConverterConfig());
    }
  }
}
```

​		第二种方式： `@ControllerAdvice` + `@InitBinder`

```java
import com.aegis.config.converter.DateConverter;
import com.aegis.model.bean.common.JsonResult;
import org.springframework.core.convert.support.GenericConversionService;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.WebDataBinder;
import org.springframework.web.bind.annotation.*;
@ControllerAdvice
public class ControllerHandler {
    @InitBinder
    public void initBinder(WebDataBinder binder) {
        GenericConversionService genericConversionService = (GenericConversionService) binder.getConversionService();
        if (genericConversionService != null) {
            genericConversionService.addConverter(new DateConverter());
        }
    }
}
```







