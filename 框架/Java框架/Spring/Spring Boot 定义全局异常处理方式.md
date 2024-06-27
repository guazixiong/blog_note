# Spring Boot 定义全局异常处理方式

SpringBoot中，@ControllerAdvice 即可开启全局异常处理，使用该注解表示开启了全局异常的捕获，我们只需在自定义一个方法使用@ExceptionHandler注解然后定义捕获异常的类型即可对这些捕获的异常进行统一的处理。

## 定义异常基础服务接口类

```java
package common.exception;

/**
 * @author: pangdi
 * @Date: 2023/9/8 14:57
 * @Description: 异常基础服务接口类
 */
public interface BaseErrorInfoInterface {

    /**
     * 错误码
     *
     * @return 错误码
     */
    String getErrorCode();

    /**
     * 错误描述
     *
     * @return 错误描述
     */
    String getErrorMessage();
}
```

## 定义异常枚举类

```java
package common.exception;

/**
 * @author: pangdi
 * @Date: 2023/9/8 14:59
 * @Description: 数据操作错误定义枚举类
 */
public enum ExceptionEnum implements BaseErrorInfoInterface {

    /**
     * 成功!
     */
    SUCCESS("2000", "成功!"),
    /**
     * 请求的数据格式不符!
     */
    BODY_NOT_MATCH("4000", "请求的数据格式不符!"),
    /**
     * 请求的数字签名不匹配!
     */
    SIGNATURE_NOT_MATCH("4001", "请求的数字签名不匹配!"),
    /**
     * 未找到该资源!
     */
    NOT_FOUND("4004", "未找到该资源!"),
    /**
     * 服务器内部错误!
     */
    INTERNAL_SERVER_ERROR("5000", "服务器内部错误!"),
    /**
     * 服务器正忙，请稍后再试!
     */
    SERVER_BUSY("5003", "服务器正忙，请稍后再试!"),
    /**
     * 类型转换异常!
     */
    PARAMS_NOT_CONVERT("5101", "类型转换异常!");

    /**
     * 错误码
     */
    private final String errorCode;

    /**
     * 错误描述
     */
    private final String errorMessage;

    ExceptionEnum(String errorCode, String errorMessage) {
        this.errorCode = errorCode;
        this.errorMessage = errorMessage;
    }

    @Override
    public String getErrorCode() {
        return errorCode;
    }

    @Override
    public String getErrorMessage() {
        return errorMessage;
    }
}
```

## 自定义异常类

```java
package common.exception;

/**
 * @author: pangdi
 * @date: 2023/8/30 15:46
 * @description: 业务异常类
 */
public class BusinessException extends RuntimeException {

    private static final long serialVersionUID = -788314347541917090L;

    /**
     * 错误码
     */
    protected String errorCode;

    /**
     * 错误信息
     */
    protected String errorMessage;

    public BusinessException() {
        super();
    }

    public BusinessException(BaseErrorInfoInterface errorInfoInterface) {
        super(String.valueOf(errorInfoInterface.getErrorCode()));
        this.errorCode = String.valueOf(errorInfoInterface.getErrorCode());
        this.errorMessage = errorInfoInterface.getErrorMessage();
    }

    public BusinessException(BaseErrorInfoInterface errorInfoInterface, Throwable cause) {
        super(String.valueOf(errorInfoInterface.getErrorCode()), cause);
        this.errorCode = String.valueOf(errorInfoInterface.getErrorCode());
        this.errorMessage = errorInfoInterface.getErrorMessage();
    }

    public BusinessException(String errorMessage) {
        super(errorMessage);
        this.errorMessage = errorMessage;
    }

    public BusinessException(String errorCode, String errorMessage) {
        super(errorCode);
        this.errorCode = errorCode;
        this.errorMessage = errorMessage;
    }

    public BusinessException(String errorCode, String errorMessage, Throwable cause) {
        super(errorCode, cause);
        this.errorCode = errorCode;
        this.errorMessage = errorMessage;
    }

    public String getErrorCode() {
        return errorCode;
    }

    public void setErrorCode(String errorCode) {
        this.errorCode = errorCode;
    }

    @Override
    public String getMessage() {
        return errorMessage;
    }

    public void setErrorMessage(String errorMessage) {
        this.errorMessage = errorMessage;
    }

    @Override
    public Throwable fillInStackTrace() {
        return this;
    }
}
```

## 自定义响应类

```java
package common.core.domain;

import common.constant.HttpStatus;
import common.exception.BaseErrorInfoInterface;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.io.Serializable;

/**
 * @author: pangdi
 * @date: 2023/8/30 17:15
 * @description: 响应信息主体
 */
@Data
@NoArgsConstructor
public class Response<T> implements Serializable {
    private static final long serialVersionUID = 1L;

    /**
     * 成功
     */
    public static final int SUCCESS = 200;

    /**
     * 失败
     */
    public static final int FAIL = 500;

    private int code;

    private String msg;

    private T data;

    public static <T> Response<T> ok() {
        return restResult(null, SUCCESS, "操作成功");
    }

    public static <T> Response<T> ok(T data) {
        return restResult(data, SUCCESS, "操作成功");
    }

    public static <T> Response<T> ok(String msg) {
        return restResult(null, SUCCESS, msg);
    }

    public static <T> Response<T> ok(String msg, T data) {
        return restResult(data, SUCCESS, msg);
    }

    public static <T> Response<T> fail() {
        return restResult(null, FAIL, "操作失败");
    }

    public static <T> Response<T> fail(String msg) {
        return restResult(null, FAIL, msg);
    }

    public static <T> Response<T> fail(T data) {
        return restResult(data, FAIL, "操作失败");
    }

    public static <T> Response<T> fail(BaseErrorInfoInterface errorInfo) {
        Response<T> response = new Response<T>();
        response.setCode(Integer.parseInt(errorInfo.getErrorCode()));
        response.setMsg(errorInfo.getErrorMessage());
        return response;
    }

    public static <T> Response<T> fail(String msg, T data) {
        return restResult(data, FAIL, msg);
    }

    public static <T> Response<T> fail(int code, String msg) {
        return restResult(null, code, msg);
    }

    /**
     * 返回警告消息
     *
     * @param msg 返回内容
     * @return 警告消息
     */
    public static <T> Response<T> warn(String msg) {
        return restResult(null, HttpStatus.WARN, msg);
    }

    /**
     * 返回警告消息
     *
     * @param msg 返回内容
     * @param data 数据对象
     * @return 警告消息
     */
    public static <T> Response<T> warn(String msg, T data) {
        return restResult(data, HttpStatus.WARN, msg);
    }

    private static <T> Response<T> restResult(T data, int code, String msg) {
        Response<T> response = new Response<T>();
        response.setCode(code);
        response.setData(data);
        response.setMsg(msg);
        return response;
    }

    public static <T> Boolean isError(Response<T> ret) {
        return !isSuccess(ret);
    }

    public static <T> Boolean isSuccess(Response<T> ret) {
        return Response.SUCCESS == ret.getCode();
    }
}
```

## 自定义全局异常处理

```java
package com.pbad.customer.advice;

import common.core.domain.Response;
import common.exception.BusinessException;
import common.exception.ExceptionEnum;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.servlet.http.HttpServletRequest;

/**
 * @author: pangdi
 * @date: 2023/9/8 15:04
 * @description: 全局异常处理器，用于处理不同类型的异常情况。
 */
@ControllerAdvice
public class GlobalExceptionHandler {
    private static final Logger logger = LoggerFactory.getLogger(GlobalExceptionHandler.class);

    /**
     * 处理自定义的业务异常。
     *
     * @param req HTTP请求对象
     * @param e   业务异常对象
     * @return 包含错误信息的响应对象
     */
    @ExceptionHandler(value = BusinessException.class)
    @ResponseBody
    public Response<Void> bizExceptionHandler(HttpServletRequest req, BusinessException e) {
        logger.error("发生业务异常！原因是：{}", e.getMessage());
        return Response.fail(Integer.parseInt(e.getErrorCode()), e.getMessage());
    }

    /**
     * 处理空指针的异常。
     *
     * @param req HTTP请求对象
     * @param e   空指针异常对象
     * @return 包含错误信息的响应对象
     */
    @ExceptionHandler(value = NullPointerException.class)
    @ResponseBody
    public Response<Void> exceptionHandler(HttpServletRequest req, NullPointerException e) {
        logger.error("发生空指针异常！原因是:", e);
        return Response.fail(ExceptionEnum.BODY_NOT_MATCH);
    }

    /**
     * 处理其他异常。
     *
     * @param req HTTP请求对象
     * @param e   其他异常对象
     * @return 包含错误信息的响应对象
     */
    @ExceptionHandler(value = Exception.class)
    @ResponseBody
    public Response<Void> exceptionHandler(HttpServletRequest req, Exception e) {
        logger.error("未知异常！原因是:", e);
        return Response.fail(ExceptionEnum.INTERNAL_SERVER_ERROR);
    }

    /**
     * 处理类型转换异常。
     *
     * @param req HTTP请求对象
     * @param e   类型转换异常对象
     * @return 包含错误信息的响应对象
     */
    @ExceptionHandler(value = NumberFormatException.class)
    @ResponseBody
    public Response<Void> exceptionHandler(HttpServletRequest req, NumberFormatException e) {
        logger.error("发生类型转换异常！原因是:", e);
        return Response.fail(ExceptionEnum.PARAMS_NOT_CONVERT);
    }
}
```

## 自定义业务异常

```java
package com.pbad.customer.exception;

/**
 * @author: pangdi
 * @date: 2023/8/30 15:53
 * @description: 异常枚举类
 */
public enum ExceptionMsgEnum {

    /**
     * 获取银行卡信息失败
     */
    NOT_FOUND_BANK_CARD("10001","获取银行卡信息失败"),
    /**
     * 银行卡已绑定
     */
    BANK_CARD_HAVE_BIND("10002","银行卡已绑定"),
    /**
     * 客商编号为空
     */
    CUSTOMER_ID_IS_NULL("10003","客商编号为空"),
    /**
     * 操作人为空
     */
    OPERATE_IS_NULL("10004","操作人为空"),
    /**
     * 获取客商信息失败
     */
    NOT_FOUND_CUSTOMER_INFO("10005","获取客商信息失败"),
    ;

    /**
     * 错误编码
     */
    private final String errorCode;

    /**
     * 错误信息
     */
    private final String errorMessage;

    ExceptionMsgEnum(String errorCode, String errorMessage) {
        this.errorCode = errorCode;
        this.errorMessage = errorMessage;
    }

    public String getErrorMessage() {
        return errorMessage;
    }

    public String getErrorCode() {
        return errorCode;
    }
}
```

## 测试

```java
package com.pbad.customer.controller;

import com.pbad.customer.exception.ExceptionMsgEnum;
import common.core.domain.Response;
import common.exception.BusinessException;
import io.swagger.annotations.Api;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author: pangdi
 * @date: 2023/8/29 10:46
 * @description: 测试全局异常controller接口
 */
@Api(tags = "测试全局异常controller接口")
@RequestMapping(path = "/test/exception/")
@RestController
public class TestGlobalExceptionController {

    /**
     * 测试空指针异常.
     */
    @GetMapping("/test01")
    public Response<Void> test01() {
        String a = null;
        System.out.println(a + 1);
        return Response.ok();
    }

    /**
     * 测试自定义业务异常.
     */
    @GetMapping("/test02")
    public Response<Void> test02() {
        throw new BusinessException(
                ExceptionMsgEnum.CUSTOMER_ID_IS_NULL.getErrorCode(),
                ExceptionMsgEnum.CUSTOMER_ID_IS_NULL.getErrorMessage());
    }

    /**
     * 测试类型转换异常.
     */
    @GetMapping("/test03")
    public Response<Void> test03() {
        Integer.parseInt("asdf");
        return Response.ok();
    }

}
```

```bash
{
    "code": 4000,
    "msg": "请求的数据格式不符!",
    "data": null
}
```

## 总结

Gitee 地址 [base_project: 一个含开发风格的实例后端开发demo](https://gitee.com/guazixiong/base_project)

一个含开发风格的实例后端开发demo
