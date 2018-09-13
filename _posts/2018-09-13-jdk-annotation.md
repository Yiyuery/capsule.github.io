---
title: JJDK 之 自定义注解 Annotation
date: 2018-09-13 23:22:00
categories:
- JDK
tags:
- clone
- Cloneable
---

JDK 通过annotation实现注解进行参数校验

---

# JDK 之 自定义注解 Annotation

## 参数校验

> 注解定义

`Validation:`

```Java
/*
 * @ProjectName: 编程学习
 * @Copyright:   2018 HangZhou Yiyuery Dev, Ltd. All Right Reserved.
 * @address:     http://xiazhaoyang.tech
 * @date:        2018/7/28 18:15
 * @email:       xiazhaoyang@live.com
 * @description: 本内容仅限于编程技术学习使用，转发请注明出处.
 */
package com.hikvision.cms.acs.common.web.validate;

import java.lang.annotation.*;

/**
 * <p>
 *    自定义注解校验
 * </p>
 *
 * @author xiachaoyang
 * @version V1.0
 * @date 2018年09月13日 13:41
 * @modificationHistory=========================逻辑或功能性重大变更记录
 * @modify By: {修改人} 2018年09月13日
 * @modify reason: {方法名}:{原因}
 * ...
 */
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Validation {

    /**
     * 允许为空
     * @return
     */
    boolean allowEmpty() default true;

    /**
     * 允许最大长度
     * @return
     */
    int maxLength() default 0;

    /**
     * 允许的最大最小值
     * @return
     */
    int[] allowRange() default {Integer.MIN_VALUE,Integer.MAX_VALUE};
}

```

> 反射解析注解

`Validator:`
```Java
/*
 * @ProjectName: 编程学习
 * @Copyright:   2018 HangZhou Yiyuery Dev, Ltd. All Right Reserved.
 * @address:     http://xiazhaoyang.tech
 * @date:        2018/7/28 18:15
 * @email:       xiazhaoyang@live.com
 * @description: 本内容仅限于编程技术学习使用，转发请注明出处.
 */
package com.hikvision.cms.acs.common.web.validate;

import com.hikvision.cms.acs.common.constant.ConstParamHttpErrorCode;
import com.hikvision.cms.acs.common.exception.ApiException;
import org.apache.commons.collections4.MapUtils;
import org.apache.commons.lang3.StringUtils;
import org.apache.commons.lang3.math.NumberUtils;
import org.springframework.util.CollectionUtils;

import java.lang.reflect.Field;
import java.lang.reflect.ParameterizedType;
import java.util.Collection;
import java.util.List;
import java.util.Map;

/**
 * <p>
 *  校验工具 基类
 * </p>
 *
 * @author xiachaoyang
 * @version V1.0
 * @date 2018年09月13日 14:32
 * @modificationHistory=========================逻辑或功能性重大变更记录
 * @modify By: {修改人} 2018年09月13日
 * @modify reason: {方法名}:{原因}
 * ...
 */
public class Validator<T> {

    private Class<T> entity;

    public Validator() {
    }

    /**
     * 核心校验方式
     */
    public void validate() {
        entity = (Class<T>) ((ParameterizedType) this.getClass().getGenericSuperclass())
                .getActualTypeArguments()[0];
        if (this.entity != null) {
            Field[] fields = entity.getDeclaredFields();
            for (Field field : fields) {
                field.setAccessible(true);
                //获取字段中包含fieldMeta的注解
                Validation v = field.getAnnotation(Validation.class);
                if (v != null) {
                    if(!v.allowEmpty()){
                        validateEmpty(field);
                    }
                    if(v.allowRange().length == 2 && (v.allowRange()[0] != Integer.MIN_VALUE || v.allowRange()[1] != Integer.MAX_VALUE)){
                        validateRange(field,v);
                    }
                    if(v.maxLength() > 0){
                        validateMaxLength(field,v);
                    }
                }
            }
        }
    }

    /**
     * 校验字符串长度、集合长度
     * @param field
     * @param v
     */
    private void validateMaxLength(Field field, Validation v) {
        try {
            Object val = field.get(this);
            boolean flag = true;
            if(val == null){
                return;
            }
            if(val instanceof String){
                if(String.valueOf(val).length() > v.maxLength()){
                    flag = false;
                }
            }
            if(val instanceof List){
                if(((List) val).size() > v.maxLength()){
                    flag = false;
                }
            }
            if(!flag){
                throw new ApiException(ConstParamHttpErrorCode.ACS_API_PARAM_RANGE_ERROR,String.format("Param '%s' is exceed max length[%d]!",field.getName(),v.maxLength()));
            }
        }catch (IllegalAccessException e) {
            throw new ApiException(ConstParamHttpErrorCode.ACS_HTTP_SYS_INNER_EXCEPTION_ERROR,"Other error.");
        }
    }

    /**
     * 校验数据范围 数值、集合大小
     * @param field
     * @param v
     */
    private void validateRange(Field field, Validation v) {
        try {
            Object val = field.get(this);
            boolean flag = true;
            if(val == null){
                return;
            }
            if(val instanceof Integer || NumberUtils.isDigits(val+"")){
                Integer value = (Integer) val;
                if(val == null || v.allowRange()[0] > value || v.allowRange()[1] < value || v.allowRange()[0] > v.allowRange()[1]){
                    flag = false;
                }
            }
            if(val instanceof List){
                if(val == null || ((List)val).size() == 0){
                    flag = false;
                }
            }
            if(!flag){
                throw new ApiException(ConstParamHttpErrorCode.ACS_API_PARAM_RANGE_ERROR,String.format("Param '%s' is out of range or illegal range configuration [%d,%d]!",field.getName(),v.allowRange()[0],v.allowRange()[1]));
            }
        }catch (IllegalAccessException e) {
            throw new ApiException(ConstParamHttpErrorCode.ACS_HTTP_SYS_INNER_EXCEPTION_ERROR,"Other error.");
        }
    }

    /**
     * 校验字符串、集合是否为空
     * @param field
     */
    private void validateEmpty(Field field) {
        try {
            Object val = field.get(this);
            boolean flag = true;
            if(val == null){
                throw new ApiException(ConstParamHttpErrorCode.ACS_HTTP_PARAM_VALIDATE_EMPTY_ERROR,String.format("Param '%s' is blank,but required!",field.getName()));
            }
            if(val instanceof String){
                if(StringUtils.isEmpty(String.valueOf(val)) || String.valueOf(val).toLowerCase().equals("null")){
                    flag = false;
                }
            }
            if(val instanceof Collection){
                if(CollectionUtils.isEmpty((Collection) val)){
                    flag = false;
                }
            }
            if(val instanceof Map){
                if(MapUtils.isEmpty((Map) val)){
                    flag = false;
                }
            }
            if(!flag){
                throw new ApiException(ConstParamHttpErrorCode.ACS_HTTP_PARAM_VALIDATE_EMPTY_ERROR,String.format("Param '%s' is blank,but required!",field.getName()));
            }
        } catch (IllegalAccessException e) {
            throw new ApiException(ConstParamHttpErrorCode.ACS_HTTP_SYS_INNER_EXCEPTION_ERROR,"Other error.");
        }
    }
}
```


> 使用方式

```Java
/*
 * @ProjectName: 编程学习
 * @Copyright:   2018 HangZhou Yiyuery Dev, Ltd. All Right Reserved.
 * @address:     http://xiazhaoyang.tech
 * @date:        2018/7/28 18:15
 * @email:       xiazhaoyang@live.com
 * @description: 本内容仅限于编程技术学习使用，转发请注明出处.
 */
package com.example.chapter3.validate;

import lombok.AllArgsConstructor;
import lombok.Data;

import java.util.List;

/**
 * <p>
 *
 * </p>
 *
 * @author xiachaoyang
 * @version V1.0
 * @date 2018年09月13日 14:15
 * @modificationHistory=========================逻辑或功能性重大变更记录
 * @modify By: {修改人} 2018年09月13日
 * @modify reason: {方法名}:{原因}
 * ...
 */
@Data
@AllArgsConstructor
public class ParamsQo extends Validator<ParamsQo>{

    @Validation(allowEmpty = false,maxLength = 5)
    private String name;
    @Validation(allowRange = {1,20})
    private Integer age;
    @Validation(allowRange = {100,200})
    private int num;
    @Validation(maxLength = 2)
    private List<Integer> nums;

}
```

`ApiException:`自定义异常类

```Java
/*
 * @ProjectName: 编程学习
 * @Copyright:   2018 HangZhou Yiyuery Dev, Ltd. All Right Reserved.
 * @address:     http://xiazhaoyang.tech
 * @date:        2018/7/28 18:15
 * @email:       xiazhaoyang@live.com
 * @description: 本内容仅限于编程技术学习使用，转发请注明出处.
 */
package com.example.chapter3.validate;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

/**
 * <p>
 *
 * </p>
 *
 * @author xiachaoyang
 * @version V1.0
 * @date 2018年09月13日 14:49
 * @modificationHistory=========================逻辑或功能性重大变更记录
 * @modify By: {修改人} 2018年09月13日
 * @modify reason: {方法名}:{原因}
 * ...
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
public class ApiException extends RuntimeException {

    private Integer errorCode;

    private String errorMsg;

    public ApiException(String errorMsg) {
        super();
        this.errorMsg = errorMsg;
    }

}
```

`Test:`
```Java
public class ValidationTest {

    @Test
    public void test(){
        ParamsQo paramsQo = new ParamsQo("xx",5,111, Lists.newArrayList(1,5));
        paramsQo.validate();//对应错误抛异常
    }
}

```


## REFRENCES

1. [java自定义注解](https://blog.csdn.net/important0534/article/details/54020691)
2. [java自定义注解实现前后台参数校验](http://www.cnblogs.com/softidea/p/5709397.html)
3. [一小时搞明白自定义注解（Annotation）](https://blog.csdn.net/u013045971/article/details/53433874)
4. [springboot~为Money类型添加最大值和最小值的注解校验](https://www.cnblogs.com/lori/p/9021681.html)
