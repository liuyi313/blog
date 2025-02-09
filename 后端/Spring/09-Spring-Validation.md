# Spring Validation

Spring 为了给开发者提供便捷，对 hibernate validation 进行了二次封装，显示校验 validated bean 时，可以使用 spring validation 或者 hibernate validation。

## 注解

JSR 提供的校验注解：

| 注解                        | 说明                                                     |
| --------------------------- | -------------------------------------------------------- |
| @Null                       | 被注释的元素必须为 null                                  |
| @NotNull                    | 被注释的元素必须不为 null                                |
| @AssertTrue                 | 被注释的元素必须为 true                                  |
| @AssertFalse                | 被注释的元素必须为 false                                 |
| @Min(value)                 | 被注释的元素必须是一个数字，其值必须大于等于指定的最小值 |
| @Max(value)                 | 被注释的元素必须是一个数字，其值必须小于等于指定的最大值 |
| @DecimalMin(value)          | 被注释的元素必须是一个数字，其值必须大于等于指定的最小值 |
| @DecimalMax(value)          | 被注释的元素必须是一个数字，其值必须小于等于指定的最大值 |
| @Size(max=, min=)           | 被注释的元素的大小必须在指定的范围内                     |
| @Digits (integer, fraction) | 被注释的元素必须是一个数字，其值必须在可接受的范围内     |
| @Past                       | 被注释的元素必须是一个过去的日期                         |
| @Future                     | 被注释的元素必须是一个将来的日期                         |
| @Pattern(regex=,flag=)      | 被注释的元素必须符合指定的正则表达式                     |

Hibernate Validator 提供的校验注解：

| 注解                       | 说明                                   |
| -------------------------- | -------------------------------------- |
| @NotBlank()                | 验证字符串非 null，且长度必须大于 0    |
| @Email                     | 被注释的元素必须是电子邮箱地址         |
| @Length(min=,max=)         | 被注释的字符串的大小必须在指定的范围内 |
| @NotEmpty                  | 被注释的字符串的必须非空               |
| @Range(min=,max=,message=) | 被注释的元素必须在合适的范围内         |

首先定义 validated bean：

```java
@Data
public class User {

    @NotBlank(message = "用户名不能为空")
    private String name;

    @Min(value = 20, message = "年龄不能小于20")
    private Integer age;

    @NotBlank(message = "住址不能为空")
    private String address;

    @Email(message = "邮箱格式错误")
    private String eamil;

    @Pattern(regexp = "^1(3|4|5|7|8)\\d{9}$",message = "手机号码格式错误")
    @NotBlank(message = "手机号码不能为空")
    private String telphone;
}
```

在 Controller 中使用：

- @Validated：可以用在类型、方法和方法参数上。但是不能用在成员属性（字段）上，且提供分组功能
- @Valid：可以用在方法、构造函数、方法参数和成员属性（字段）上

```java
@RestController
@Slf4j
public class UserController {

    @PostMapping("/register")
    public void register(@Valid @RequestBody User user, BindingResult bindingResult) {
        if(bindingResult.hasErrors()) {
            log.error("【注册用户】参数不正确，user={}, msg={}", user, bindingResult.getFieldError().getDefaultMessage());
        }
    }
}
```

如果有多个参数需要校验，形式可以如下，即一个校验类对应一个校验结果：

```java
foo(@Validated Foo foo, BindingResult fooBindingResult ，@Validated Bar bar, BindingResult barBindingResult);
```

## 分组校验

如果同一个类，在不同的使用场景下有不同的校验规则，那么可以使用分组校验。

```java
@Data
public class User {

    @Min(value = 20, message = "年龄不能小于20", groups = {Adult.class})
    private Integer age;

    public interface Adult{}

    public interface Minor{}
}
```

在 Controller 中使用，注意使用分组功能必须使用 `@Validated` 注解：

```java
@RestController
@Slf4j
public class UserController {

    @PostMapping("/drink")
    public void drink(@Validated({User.Adult.class}) @RequestBody User user, BindingResult bindingResult) {
        if(bindingResult.hasErrors()) {
            log.error("参数不正确，user={}, msg={}", user, bindingResult.getFieldError().getDefaultMessage());
        }
    }

    @PostMapping("/live")
    public void live(@Validated @RequestBody User user, BindingResult bindingResult) {
        if(bindingResult.hasErrors()) {
            log.error("参数不正确，user={}, msg={}", user, bindingResult.getFieldError().getDefaultMessage());
        }
    }
}
```

参考文章：  
[使用 spring validation 完成数据后端校验](https://www.cnkirito.moe/spring-validation/)
