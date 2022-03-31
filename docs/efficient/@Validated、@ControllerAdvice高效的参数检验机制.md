**`@Validated @ControllerAdvice` 高效的参数检验机制**

### 传统的参数校验实现

```java
 @GetMapping("normal")
    @ApiOperation("低效的参数校验")
    public ResponseResult normal(Photo photo) {
        if (StringUtils.isNotBlank(photo.getTitle())){
            return ResponseResult.failure(StatusCodeEnum.PARAM_ERROR.getCode(), "标题不能为空");
        }
        if (StringUtils.isNotBlank(photo.getContent())){
            return ResponseResult.failure(StatusCodeEnum.PARAM_ERROR.getCode(), "内容不能为空");
        }
        if (photo.getAddTime() != null){
            return ResponseResult.failure(StatusCodeEnum.PARAM_ERROR.getCode(), "添加时间不能为空");
        }
        log.info("信息：{}",photo);
        return ResponseResult.success(StatusCodeEnum.SUCCESS);
    }
```

​		需要自己一个个去写`if else`判断

### 高效的参数校验

- 定义实体类，并且添加判断注解

```java
package com.zjm.photo.model;

import java.math.BigDecimal;
import com.baomidou.mybatisplus.annotation.TableName;
import com.baomidou.mybatisplus.annotation.IdType;
import com.zjm.photo.model.base.BaseModel;
import com.baomidou.mybatisplus.extension.activerecord.Model;
import java.util.Date;
import com.baomidou.mybatisplus.annotation.Version;
import com.baomidou.mybatisplus.annotation.TableId;
import java.io.Serializable;
import lombok.Data;
import lombok.EqualsAndHashCode;
import lombok.experimental.Accessors;

import javax.validation.constraints.NotBlank;
import javax.validation.constraints.NotNull;

/**
 * <p>
 * 
 * </p>
 *
 * @author zjm
 * @since 2021-05-16
 */
@Data
@EqualsAndHashCode(callSuper = true)
@Accessors(chain = true)
@TableName("p_photo")
public class Photo extends BaseModel<Photo> {

    private static final long serialVersionUID = 1L;

    @TableId(value = "photo_id", type = IdType.AUTO)
    private Integer photoId;

    private String uuid;

    /**
     * 标题
     */
    @NotBlank(message = "标题不能为空")//添加参数校验判断
    private String title;

    /**
     * 内容
     */
    @NotBlank(message = "内容不能为空")//添加参数校验判断
    private String content;

    /**
     * 发布时间
     */
    @NotNull(message = "发布时间不能为空")//添加参数校验判断
    private Date addTime;

    /**
     * 地址
     */
    private String address;

    private BigDecimal lat;

    private BigDecimal lng;

    /**
     * 状态
     */
    private Integer statue;


    @Override
    protected Serializable pkVal() {
        return this.photoId;
    }

}

```

> 说明：此处实体类中加了三个判空的注解

- 通用返回结果模型

```java
package com.zjm.photo.model.dto;


import com.zjm.photo.enums.StatusCodeEnum;
import lombok.Data;

import java.io.Serializable;

/**
 * 类名称：ResponseResult
 * ********************************
 * <p>
 * 类描述：通用返回结果模型
 *
 * @author
 * @date 下午12:59
 */
@Data
public class ResponseResult<T> implements Serializable {
    /**
     * serialVersionUID
     */
    private static final long serialVersionUID = 7813356989387725160L;
    /**
     * 是否成功
     */
    private boolean success;
    private boolean isSuccess;
    /**
     * 编码
     */
    private String code;
    /**
     * 描述信息
     */
    private String msg;
    /**
     * 结果
     */
    private T data;

    /**
     * 数量
     */
    private Long count;

    private Long pages;

    /**
     * 成功
     * @param info
     * @param statusCodeEnum
     * @param <T>
     * @return
     */
    public static <T> ResponseResult<T> success(T info, StatusCodeEnum statusCodeEnum, Long count) {
        return success(info,statusCodeEnum.getCode(), statusCodeEnum.getMsg(),count);
    }

    public static <T> ResponseResult<T> success(T info,StatusCodeEnum statusCodeEnum) {
        return success(info,statusCodeEnum.getCode(), statusCodeEnum.getMsg(),null);
    }
    public static <T> ResponseResult<T> success(StatusCodeEnum statusCodeEnum) {
        return success(null,statusCodeEnum.getCode(), statusCodeEnum.getMsg(),null);
    }

    public static <T> ResponseResult<T> success(StatusCodeEnum statusCodeEnum,Long count) {
        return success(null,statusCodeEnum.getCode(), statusCodeEnum.getMsg(),count);
    }


    public static <T> ResponseResult<T> success(T info,StatusCodeEnum statusCodeEnum,Long count,Long pages) {
        return success(info,statusCodeEnum.getCode(), statusCodeEnum.getMsg(),count,pages);
    }

    /**
     * 成功
     * @param info
     * @param <T>
     * @return
     */
    public static <T> ResponseResult<T> success(T info,String code, String msg,Long count) {
        return success(info,code,msg,count,null);
    }

    public static <T> ResponseResult<T> success(T info,String code, String msg,Long count,Long pages) {
        ResponseResult<T> responseResult = new ResponseResult<>();
        responseResult.setSuccess(Boolean.TRUE);
        responseResult.setCode(code);
        responseResult.setMsg(msg);
        responseResult.setData(info);
        responseResult.setCount(count);
        responseResult.setPages(pages);
        return responseResult;
    }




    /**
     * 失败
     * @param statusCodeEnum
     * @param <T>
     * @return
     */
    public static <T> ResponseResult<T> failure(StatusCodeEnum statusCodeEnum) {
        return failure(null,statusCodeEnum.getCode(), statusCodeEnum.getMsg());
    }

    public static <T> ResponseResult<T> failure(String code,String msg) {
        return failure(null,code, msg);
    }

    public static <T> ResponseResult<T> failure(T info,StatusCodeEnum statusCodeEnum) {
        return failure(info,statusCodeEnum.getCode(), statusCodeEnum.getMsg());
    }

    /**
     * 失败
     * @param code
     * @param msg
     * @param <T>
     * @return
     */
    public static <T> ResponseResult<T> failure(T info,String code, String msg) {
        ResponseResult<T> responseResult = new ResponseResult<>();
        responseResult.setSuccess(Boolean.FALSE);
        responseResult.setCode(code);
        responseResult.setMsg(msg);
        responseResult.setData(info);
        return responseResult;
    }
}

```

- 统一的状态码

```java
package com.zjm.photo.enums;

import lombok.AllArgsConstructor;
import lombok.Getter;

/**
 * 类名称：StatusCodeEnum
 * ********************************
 * <p>
 * 类描述：状态编码枚举
 *
 * @author
 * @date 2020/3/1 下午9:54
 */
@AllArgsConstructor
@Getter
public enum StatusCodeEnum {

    // 2*** 成功
    SUCCESS("200", "操作成功"),

    // 3*** 参数异常
    PARAM_ERROR("201", "参数异常"),
    PARAM_NULL("202", "参数为空"),
    PARAM_FORMAT_ERROR("203", "参数格式不正确"),
    PARAM_VALUE_ERROR("204", "参数值不正确"),

    // 4*** 系统异常
    SYSTEM_ERROR("401", "服务异常"),
    UNKNOWN_ERROR("402", "未知异常"),
    FILE_EMPTY("403", "文件不存在"),
    NO_RESULT("408", "暂无数据"),
    SO_MUCH_RESULT("409", "so much result"),
    SERVER_BUSY("410", "服务器繁忙，请稍后再试!"),

    // 5*** 业务异常
    XXX("500", "业务异常"),
    SYSTME_ERR("501", "业务异常"),
    INSERT_FAILURE("502", "新增失败"),
    UPDATE_FAILURE("503", "更新失败"),
    DELETE_FAILURE("504", "删除失败"),
    RATE_LIMIT_ERROR("505", "限流异常"),
    FILE_UPLOAD_FAILURE("506", "文件上传失败"),
    OTHER_ERROR("507", "其他错误"),
    ALERADY_HAVE("508", "该企业一添加至异常名录库"),

    NUMBER_FORMAT("601","数字转换异常"),


    ILLEG_STATUS("701","非法状态异常"),

    NO_POWER("801","此账号无登录权限"),
    GET_USER_FAIL("802","获取用户失败")

    ;


    /**
     * 错误编码
     */
    private String code;

    /**
     * 错误描述
     */
    private String msg;


}

```

- 定义全局处理器[@ControllerAdvice](https://dalin.blog.csdn.net/article/details/113979539)

```java
package com.zjm.photo.Exception;

import com.zjm.photo.enums.StatusCodeEnum;
import com.zjm.photo.model.dto.ResponseResult;
import lombok.extern.slf4j.Slf4j;
import org.springframework.validation.BindException;
import org.springframework.validation.BindingResult;
import org.springframework.validation.ObjectError;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.servlet.http.HttpServletRequest;
import java.util.List;
import java.util.stream.Collectors;

@Slf4j
@ControllerAdvice
public class GlobalExceptionHandler {


    /**
     * 参数校验异常
     * @param be
     * @return
     */
    @ResponseBody//返回json数据
    @ExceptionHandler(value = BindException.class)//异常处理器
    public ResponseResult bindException(BindException be) {
        log.error("捕捉BindException异常：", be);
        List<ObjectError> allErrors = be.getAllErrors();
        List<String> collect = allErrors.stream().map(ObjectError::getDefaultMessage).collect(Collectors.toList());
        return ResponseResult.failure(collect,
                StatusCodeEnum.SYSTEM_ERROR.getCode(),
                "捕捉BindException异常");
    }
}

```

- controller层代码

```java
@Slf4j
@RequestMapping("/test/")
@RestController
@Api(tags = "测试")
public class TestController {
    @GetMapping("test")
    @ApiOperation("高效的参数校验")
    public ResponseResult test(@Validated Photo photo) {
        log.info("信息：{}",photo);
        return ResponseResult.success(StatusCodeEnum.SUCCESS);
    }
}
```

> 说明：方法参数前面加了`@Validated`校验注解

- 测试

  > 当我们什么都不传

![image-20210621165407795](https://gitee.com/laoyouji1018/images/raw/master/img/20210621165409.png)

> 当我们只传标题时

![image-20210621165459017](https://gitee.com/laoyouji1018/images/raw/master/img/20210621165500.png)

- 结论

通过全局异常处理器`@ControllerAdvice`、参数校验器`@Validated`、判空器`@NotBlank`的结合，可以高效的进行参数校验

### 参数检验进阶

- [分组校验](https://zhuanlan.zhihu.com/p/338722029)

> 分组校验说明：比如说上面的`photo`实体类，字段title上加了`@NotBlank`注解，所有用到该实体类上作为参数的方法都会校验，但是，有的接口我想校验，有的不想校验，这时候该怎么办？就可以用分组校验这样的方式

- [嵌套校验](https://blog.csdn.net/qq_32352777/article/details/108424932)

> 嵌套检验说明：类似于实体类中属性字段是另外一个实体，两个实体类都需要做校验，此时可以用嵌套检验

