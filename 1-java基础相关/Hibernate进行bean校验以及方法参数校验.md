**Hibernate进行bean校验以及方法参数校验**

​		具体使用可以参考bss-service大网项目.

### 准备

下载hibernate validator的依赖包，这里全部使用maven管理依赖。

```xml
<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>6.0.1.Final</version>
</dependency>
<dependency>
    <groupId>org.glassfish</groupId>
    <artifactId>javax.el</artifactId>
    <version>3.0.1-b08</version>
</dependency>
```

### 校验bean的属性

1. 先定义一个需要校验的类，这里采用hibernate validator官方的例子

```java
public class Car {
   @NotNull @Size(min = 2, max = 14)
   private String manufacturer;
   public Car(String manufacturer) {
      this.manufacturer = manufacturer;
   }
   public void drive(@Max(50) int speedInMph) {
     // ...
   }
  // getter setter
}
```

1. 获取Validator

```dart
  ValidatorFactory factory = Validation.byProvider(HibernateValidator.class)
                           .configure()
                           .buildValidatorFactory();
   Validator validator = factory.getValidator();
```

1. 使用Validator校验Car

```dart
  Car car = new Car(null);
  Set<ConstraintViolation<Car>> violations = validator.validate(car);
  assertEquals(1, violations.size());
```

​		对于bean的校验首先需要确定要校验哪些类，在这些类的属性添加各种约束（比如@NotNull），通过java.validation.Validation获取Validator，通过validator校验对象，获取校验的结果，其校验结果都是返回一个包含ConstraintViolation对象的集合。

### 校验方法中的参数

比如要校验Car中drive方法中的参数。

1. 首先获取ExecutableValidator

```undefined
ExecutableValidator executableValidator = validator.forExecutables();
```

通过之前获取的Validator得到ExecutableValidater.

1. 通过ExecutableValidator校验drive方法中的参数

```kotlin
Car object = new Car("Morris");
Method method = RacingCar.class.getMethod( "drive", int.class );
Object[] parameterValues = { 90 };
Set<ConstraintViolation> violations = executableValidator.validateParameters(
                object,
                method,
                parameterValues
        );
// assertEquals(1, violations.size() );
```

通过这个例子能够最bean的属性和方法进行简单的校验，但是我们经常遇到需要校验的场景有：

1. 对象的级联校验；
    比如Car中引用一个对象Driver，在校验Car的同时也需要校验Driver中的属性。

```java
class Car {
  @NotNull
   private String manufacturer;
   private Driver driver;
}
```

1. 对象中关联参数联合校验；
    比如Car有两个属性，座位数和乘客数，要求乘客数量不能大于座位数。

```java
class Car {
  @Max(20)
  private int seatCount;
  // passengers.size() <= seatCount
  private List<Passenger> passengers;
}
```

1. 方法中参数关联校验；
    比如Car对象的方法buildCar的签名如下：

```csharp
public Car buildCar(int seatCount, List<Passenger> passengers) {
  // ...
  return null;
}
```

要求乘客数量小于座位数。

1. 默认情况下，validator会对被校验对象的所有属性进行校验，能否只校验一部分？
2. 如何自定义约束呢？
3. 如何自定义校验结果中的message呢？
    下面一起来回答前面提到的几个问题。

### 级联校验

通过在属性上添加@Valid注解就可以进行级联校验了。如下：

```java
class Car {
  @NotNull
  private String manufacturer;
  @Valid
  private Driver driver;

  // getter setter
}
```

这样在校验Car的时候，也同时会校验Driver中的属性。

### 自定义约束

1.首先定义一个约束，约束是一个注解形式，相当于定义个注解，我们需要确定这个注解是用于属性、类还是方法上等，其次约束注解需要提供几个固定的方法，最后确定这个约束需要的自定义方法。例如要验证汽车的载客人数不能超过座位数的约束。

```java
@Target({TYPE, ANNOTATION_TYPE})
@Retention(RUNTIME)
//必须添加下面这个注解，实际校验的时候将使用指定的Validator进行校验
@Constraint(validateBy = {ValidPassengerCount.Validator.class}) 
public @interface ValidPassengerCount {
// 固定需要添加的方法
  String message() default 'validatePassengerCount.message';
// 固定需要添加的方法
Class<?>[] groups() default {};
// 固定需要添加的方法
Class<? extends Payload> payload() default {};

class Validator implements ConstraintValidator<ValidPassengerCount, Car> {

        @Override
        public void initialize(ValidPassengerCount constraintAnnotation) {

        }
        @Override
        public boolean isValid(Car value, ConstraintValidatorContext context) {
            if (value.getPassengers().size() > value.getPassengers().size()) {
                return false;
            }
            return true;
        }
    }
```

上面自定义的约束没有添加自定义的业务属性，但可以添加任何自定义的方法，然后在Validator的initialize方法中，通过ValidatorPassengerCount获取自定义方法放回的结果保存在Validator中，然后在isValid方法使用；
 对于isValid方法的返回类型是boolean型，校验通过返回true，校验失败返回false。

 2.在需要约束的类定义中添加自定义的约束，已Car为例，如下:

```java
@ValidPassengerCount
class Car {
  @Min(4)
  private int seatCount;
  @NotNull
  private List<Passenger> passengers;
  // getter setter
}
```

### 约束分组（GROUP）

​		约束分组用来实现部分校验的功能，例如我们在Car的fields上添加了较多约束，但是在有些场景中我们只需要验证car的部分属性，虽然这种场景的使用应不多，但我们如何实现这种功能呢？
 通过前面的例子我们可以看到，在每一个约束中都包含一个groups的属性，返回class数组，Validator的validate方法也提供一个输入groups的参数，我想大家都明白groups是怎么用的了，对，我们就是可以使用groups实现之校验该跟分组的约束。示例如下：

```java
public class Driver  {

@NotNull
public String name;

    @Min(
            value = 18,
            message = "You have to be 18 to drive a car",
            groups = DriverChecks.class
    )
    public int age;

    @AssertTrue(
            message = "You first have to pass the driving test",
            groups = DriverChecks.class
    )
    public boolean hasDrivingLicense;

    public Driver(String name) {
       this.name = name;
    }

    public void passedDrivingTest(boolean b) {
        hasDrivingLicense = b;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

```csharp
public interface DriverChecks {
}
```

上面的示例代码中使用了两种group，DEFAULT和DriverChecks，Driver.name属于DEFAULT 组，Driver.age和Driver.hasDrivingLicense属于DriverChecks分组，如果只想校验Driver.name，只需要参照如下示例:

```css
validator.validate(driver, DEFAULT.class);
```

如果只想校验Driver.age和Driver.hasDrivingLicense，参考如下示例：

```css
validator.validate(driver, DriverChecks.class);
```

如果想同时Driver.name、Driver.age和Driver.hasDrivingLicense，参考如下示例：

```css
validator.validate(driver, DEFAULT.class, DriverChecks.class);
```

校验的顺序与group的先后顺序一致。

### 自定义message

自定义message可以通过多种方式来实现。

1. 在属性或者方法参数上添加约束注解时，可以在约束的message属性上设置自定义的message。如下：

```kotlin
class Car {
  @NotNull("car.menufacturer can't be null")
  private String manufacturer;
}
```

1. 通过在构建Validator实例的过程中，配置ValidatorFactory时，设置一个MessageInterpolater，如下：

```cpp
validator = Validation.byProvider(HibernateValidator.class)
                .configure()
                .messageInterpolator(new ResourceBundleMessageInterpolator(new PlatformResourceBundleLocator("message/validate_message")))
                .buildValidatorFactory()
                .getValidator();
```

然后在classpath的message目录下添加validate_message.properties文件，将要翻译的信息添加到该文件中：

![img](https:////upload-images.jianshu.io/upload_images/3116850-9b297af39917dd20.png?imageMogr2/auto-orient/strip|imageView2/2/w/656/format/webp)



这个是hibernate-validator jar中ValidationMessages.properties文件中的内容，默认key的规则都是以定义的约束的class的全限量名加".message"组成。

1. 自定义约束设置message
    自定义约束时，在约束的message属性上设置default值，如下：

```dart
String message() default "{javax.validation.constraints.NotNull.message}";
```

1. 在指定约束的校验器中，覆盖默认message，如下：

```bash
constraintContext.disableDefaultConstraintViolation();           
constraintContext.buildConstraintViolationWithTemplate( "{com.mycompany.constraints.CheckCase.message}"  ).addConstraintViolation();       
 }
```

hibernate-validator还包含一些其他的特性，就不细说了。下面我们看看如何将hibernate-validator应用到实际的开发工作中去。

### 应用例子

对于使用springMvc框架的，要在Controller中校验方法参数只需要在参数上注解@Valid和一些约束注解就可以了。
 在使用一些rpc通讯框架时，一般这些rpc框架都不会集成一些参数校验的组件，需要我们自己写，这个时候我们就可以采用hibernate-validator组件了，这个相比自己去写校验组件真是快多了。
 一般采用rpc做服务实现时，在服务实现的第一层，我们通过配置aop的方式对服务实现类进行代理，在代理中添加校验的逻辑。如下：

1. 写一个通过hibernate-validator进行校验的类，如下：

```tsx
public class HibernateValidateService implements ValidateService {

    private static HibernateValidateService INSTANCE;

    private static ExecutableValidator validator;

    static {
        validator = Validation.byProvider(HibernateValidator.class)
                .configure()
                .failFast(true)
                .ignoreXmlConfiguration()
                .parameterNameProvider(new ParanamerParameterNameProvider())
                .messageInterpolator(new ResourceBundleMessageInterpolator(new PlatformResourceBundleLocator("message/validate_message")))
                .buildValidatorFactory()
                .getValidator()
                .forExecutables();
    }

    public <T> void validate(T obj, Method method, Object[] parameters) throws OspException {
        Set<ConstraintViolation<T>> violations = validator.validateParameters(obj, method, parameters);
        if (!violations.isEmpty()) {
            ConstraintViolation<T> violation = violations.iterator().next();
            throw new IllegalArgumentException(buildErrorMsg(violation));
        }
    }

    private <T> String buildErrorMsg(ConstraintViolation<T> violation) {
        Iterator<Path.Node> propertyNodes = violation.getPropertyPath().iterator();
        // skip method name
        propertyNodes.next();

        StringBuilder sb = new StringBuilder();
        while (propertyNodes.hasNext()) {
            sb.append(propertyNodes.next().getName());
            if (propertyNodes.hasNext()) {
                sb.append(".");
            } else {
                sb.append(" ");
            }
        }
        return sb.append(violation.getMessage()).toString();
    }

    public static ValidateService getInstance() {
        if (INSTANCE == null) {
            synchronized (HibernateValidateService.class) {
                if (INSTANCE == null) {
                    INSTANCE = new HibernateValidateService();
                }
            }
        }
        return INSTANCE;
    }

}
```

1. 编写aop advice

```java
public class ValidateBeforeAdvice implements MethodBeforeAdvice {
    private ValidateService validateService = HibernateValidateService.getInstance();
    @Override
    public void before(Method method, Object[] args, Object target) throws Throwable {
        validateService.validate(target, method, args);
    }
}
```

​	2.配置服务实现层（api层）代理

```csharp
 <bean id="validateBeforeAdvice" class="advice.ValidateBeforeAdvice" />
<aop:config proxy-target-class="false">
        <aop:pointcut id="pointcut" expression="execution(* api.validate..*(..))" />
        <aop:advisor advice-ref="validateBeforeAdvice" pointcut-ref="pointcut" />
    </aop:config>
```

这里有个注意点就是api层抛出了什么类型的异常，在写校验的时候也应该抛出什么类型的异常。