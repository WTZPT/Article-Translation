本篇翻译Mohamed Taman的[Optional12种实践建议](https://blogs.oracle.com/javamagazine/12-recipes-for-using-the-optional-class-as-its-meant-to-be-used) 

# Recipe 1: 不要给optional 变量赋null值

有时候，当开发人员在处理数据库以查询一个employee时，会设计一个方法来返回Optional<Employee>;如果没有结果返回从数据库，有一些开发人员仍然返回null，，例如:
```Java
1 public Optional<Employee> getEmployee(int id) {
2    // perform a search for employee 
3    Optional<Employee> employee = null; // in case no employee
4    return employee; 
5 }
```
上面的代码是不正确的，应该完全避免它。应该用下面的行替换第3行，使用Optional自带的empty()来初始化Option
```java
Optional<Employee> employee = Optional.empty();
```
Optional是一个可以保存值的容器，用null初始化它是没有用的。
**API note: empty()方法适用Java8及以上**

# Recipe 2: 不要直接调用get()

考虑下面的代码段。有什么问题吗?
```java
Optional<Employee> employee = HRService.getEmployee();
Employee myEmployee = employee.get();
```
 是否猜到Optional的“employee”可能为空，直接调用get()可能抛出java.util.NoSuchElementException?如果是的话，你是对的。如果你认为调用get()会让你一天过得很愉快，那你就错了。首先应该使用isPresent()方法检查值是否存在，如下所示:
```javascript
if (employee.isPresent()) {
    Employee myEmployee = employee.get();
    ... // do something with "myEmployee"
} else {
    ... // do something that doesn't call employee.get()
}
```
注意，上面的写法只是样例，并不可取，下面会有几种代替isPresent()/get()方法的
**API note: isPresent()/get()方法适用Java8及以上**
# Recipe 3:使用Optional时，当Optional值为空引用时不要直接使用null
在某些情况下，Optional的值为null，此时不要直接使用null，而是应该使用orElse(null)
考虑下面调用方法类的反射API的invoke()方法的示例。它在运行时调用该方法。如果调用的方法是静态的，则第一个参数为null;否则，它传递包含类实例的方法。
```java
1 public void callDynamicMethod(MyClass clazz, String methodName) throws ... {
2    Optional<MyClass> myClass = clazz.getInstance();
3    Method method = MyClass.class.getDeclaredMethod(methodName, String.class);
4    if (myClass.isPresent()) {
5        method.invoke(myClass.get(), "Test");
6    } else {
7        method.invoke(null, "Test");
8    }
9 }
```
通常，应该避免使用orElse(null)，但是在这种情况下，使用orElse(null)比使用上面的代码更可取。因此，您可以用以下简洁的代码行替换第4行到第8行:
```
4    method.invoke(myClass.orElse(null), "Test");
```
**API note: orElse()方法适用Java8及以上**

上面讨论在使用Optional时如何避免null引用问题。下面讨论在Optional中设置和返回数据的不同方法。
# Recipe 4:  避免使用isPresent()/get()去操作value
考虑下面的代码。你能改变什么使它更优雅和有效?
```java
public static final String DEFAULT_STATUS = "Unknown";
...
public String getEmployeeStatus(long id) {
    Optional<String> empStatus = ... ;
    if (empStatus.isPresent()) {
        return empStatus.get();
    } else {
        return DEFAULT_STATUS;
    }
}
```

与Recipe #3相似，使用orElse()代替isPresent()/get()，如下：
```java
public String getEmployeeStatus(long id) {
    Optional<String> empStatus = ... ;
    return empStatus.orElse(DEFAULT_STATUS); 
}
```
这里需要考虑的一个非常重要的问题-性能损失:无论optional的value是否存在，orElse()返回的value始终会被计算。因此，这里的规则是，当您已经有预先构造的值并且不使用昂贵的计算值时，使用orElse()。
**API note: orElse()方法适用Java8及以上**

# Recipe 5: 不要使用orElse()返回计算值
考虑下面的代码片段：
```java
Optional<Employee> getFromCache(int id) {
    System.out.println("search in cache with Id: " + id);
    // get value from cache
}

Optional<Employee> getFromDB(int id) {
    System.out.println("search in Database with Id: " + id);    
    // get value from database
}

public Employee findEmployee(int id) {        
    return getFromCache(id)
            .orElse(getFromDB(id)
                    .orElseThrow(() -> new NotFoundException("Employee not found with id" + id)));}
```
首先，代码尝试从缓存中获取具有给定ID的employee，如果该employee不在缓存中，则尝试从数据库中获取。然后，如果employee不在缓存或数据库中，代码将抛出NotFoundException。如果您运行这段代码，并且雇employee在缓存中，则会打印以下内容:
```
Search in cache with Id: 1
Search in Database with Id: 1
```
即使employee将从缓存中返回，数据库查询仍然被调用。这很贵，对吧?相反，我将使用orElseGet(Supplier<? extends T> supplier)  这个有点像 orElse() 但有点不同，  orElseGet()允许当optional是empty时候去调用你定义的supplier 方法，这有利于提高性能。
现在考虑使用orElseGet()，重构上面的代码
```java
public Employee findEmployee(int id) {        
    return getFromCache(id)
        .orElseGet(() -> getFromDB(id)
            .orElseThrow(() -> {
                return new NotFoundException("Employee not found with id" + id);
            }));
}
```
这一次，你将得到你想要的和性能改进:代码将只打印以下内容:
```
Search in cache with Id: 1       
```
**API note: orElseGet()方法适用Java8及以上**
# Recipe 6: 在没有值的条件下抛出 exception

在某些情况下，您需要抛出异常，以表明某个值不存在。这通常发生在您开发与数据库或其他资源交互的服务时。使用Optional，很容易做到这一点。考虑下面的例子:
```
public Employee findEmployee(int id) {        
    var employee = p.getFromDB(id);
    if(employee.isPresent())
        return employee.get();
    else
        throw new NoSuchElementException();
}
```

优雅实现：
```
public Employee findEmployee(int id) {        
    return getFromDB(id).orElseThrow();
}
```
**API note:orElseThrow()适用 Java 10.  如果使用 Java 8 or 9, 参考recipe #7.**
# Recipe 7: 如何在没有值的时候显示抛出异常?
在Recipe #6，只能抛出一种隐式的异常-NoSuchElementException。但是这样的异常不足以向客户报告问题的描述性和相关性信息。如果你还记得的话，Recipe #5使用了orElseThrow(Supplier<? extends X> exceptionSupplier)方法，如果Optional没有值，orElseThrow方法能够抛出显示传给它的异常。
重构下面的代码片段
```java
@GetMapping("/employees/{id}")
public Employee getEmployee(@PathVariable("id") String id) {
    Optional<Employee> foundEmployee = HrRepository.findByEmployeeId(id);
    if(foundEmployee.isPresent())
        return foundEmployee.get();
    else
        throw new NotFoundException("Employee not found with id " + id);
}
```
重构后：
```java
@GetMapping("/employees/{id}")
public Employee getEmployee(@PathVariable("id") String id) {
    return HrRepository
    .findByEmployeeId(id)
    .orElseThrow(
        () -> new NotFoundException("Employee not found with id " + id));
}
```
此外，如果你只对抛出空异常感兴趣，像这样:

```
return status.orElseThrow(NotFoundException::new);
```
**API note:如果传入null参数给orElseThrow()方法，当没有值的时候会抛出NullPointerException。orElseThrow(Supplier<? extends X> exceptionSupplier)适用java8及以上**
# Recipe 8: 如果希望仅在Optional存在可选值时执行操作，则不要使用isPresent()-get()。
 有时， 希望仅在Optional存在可选值存在时执行操作，而在不存在可选值时不执行操作。这时应当使用ifPresent(Consumer<? super T> action)方法。下面的代码应当避免

1 Optional<String> confName = Optional.of("CodeOne"); 2 if(confName.isPresent()) 3    System.out.println(confName.get().length());
 使用ifPresent() 只需将上面第2、3行替换为一行，如下:
```
confName.ifPresent( s -> System.out.println(s.length()));
```
**API note:ifPresent()方法适用Java8及以上**
# Recipe 9:  如果值不存在，不要使用 isPresent()-get()操作
有时候，开发人员会编写针对Optional value存在和不存在的代码，如下：
```java
1 Optional<Employee> employee = ... ;
2 if(employee.isPresent()) {
3    log.debug("Found Employee: {}" , employee.get().getName());
4 } else {
5    log.error("Employee not found");
6 }
```
 注意，ifPresentOrElse()类似于ifPresent()，唯一的区别是它也覆盖了else分支。因此，您可以将第2行到第6行替换为:

```
employee.ifPresentOrElse(
emp -> log.debug("Found Employee: {}",emp.getName()), 
() -> log.error("Employee not found"));
```
**API note：ifPresentOrElse()方法适用Java9及以上**
# Recipe 10:当 Optional不存在值时返回另一个Optional。

在某些情况下，当Optional value是存在情况下会返回一个描述该value的Optional，否则，会返回一个由supplying function生成的Optional。
应当避免下面操作：
```java
Optional<String> defaultJobStatus = Optional.of("Not started yet.");
public Optional<String> fetchJobStatus(int jobId) {
    Optional<String> foundStatus = ... ; // fetch declared job status by id
    if (foundStatus.isPresent())
        return foundStatus;
    else
        return defaultJobStatus; 
}
```
 不要过度使用orElse()或orElseGet()方法来完成此操作，因为这两个方法都返回一个未封装的值。
所以也不要这样做:

```java
public Optional<String> fetchJobStatus(int jobId) {
    Optional<String> foundStatus = ... ; // fetch declared job status by id
    return foundStatus.orElseGet(() -> Optional.<String>of("Not started yet."));
}
```
完美而优雅的解决方案是使用or (Supplier<? extends Optional<? extends T>> supplier)方法，像下面：
```java
1 public Optional<String> fetchJobStatus(int jobId) {
2    Optional<String> foundStatus = ... ; // fetch declared job status by id
3    return foundStatus.or(() -> defaultJobStatus);
4 }
```
或者 在开始时没有定义defaultJobStatus可选，你也可以用以下代码替换第3行代码:
```
return foundStatus.or(() -> Optional.of("Not started yet."));
```
**API note：or()在supplying function是null或者结果为null值时会抛出NullPointerException 。or()方法适用Java9及以上**
# Recipe 11:  在不关注Optional Value是否为null情况下获取该Optional的状态

自Java11起，判断Optional是否为空可以使用isEmpty()方法，isEmpty()会在Optional为空时返回true。所以，可以代替下面的代码片段：
```java
1 public boolean isMovieListEmpty(int id){
2    Optional<MovieList> movieList = ... ;
3    return !movieList.isPresent();
4 }
```
 可以用下面的行替换第3行，使代码更易读:

```
return movieList.isEmpty();
```

**API note：isEmpty()方法适用Java11及以上**
# Recipe 12: 不要过度使用Optional.

有时候开发人员会倾向过度使用喜欢的东西，Optional就是其中之一。使用过程应当考虑清晰性、内存占用和简洁。
应该避免下面的代码
```java
1 public String fetchJobStatus(int jobId) {
2    String status = ... ; // fetch declared job status by id
3    return Optional.ofNullable(status).orElse("Not started yet.");
4 }
```
直接用下面这行代码替换第3行:
```
return status == null ? "Not started yet." : status;
```