### 无参情况
无参情况就是说我们的方法内不接收参数。

#### Get请求
当我们只写RequestMapping，而不指定RequestMethod的时候。默认的method为一个get请求。
```java
@RequestMapping("/noArgs/getDemo")
public void noArgsGetDemo();
```

#### Post请求
```java
@RequestMapping(value = "/noArgs/postDemo",method = RequestMethod.POST)
public void noArgsPostDemo();
```
也可以直接使用PostMapping

```java
@PostMapping(value = "/noArgs/postDemo")
public void noArgsPostDemo();
```


### 单个参数的情况
方法内只有一个参数

#### Get请求
get请求方式接参，只能使用RequestParam注解
```java
@RequestMapping(value = "/singleArg/getDemo")
public void singleArgGetDemo(@RequestParam String name);
```
不写RequestMethod注解，默认就是get请求。

#### Post请求
post请求方式接参，可以使用三种方式，一种是不写，一种是RequestParam，一种是RequestBody。

#### RequestParam
先说说RequestParam这种方式。需要指明method，如果不指明则和上方一样了。默认是get。
```java
@RequestMapping(value = "/singleArg/PostDemo",method = RequestMethod.POST)
public void singleArgPostDemo(@RequestParam String name);
```

#### RequestBody
一旦使用RequestBody这种方式，他就是post请求，不用写method了。
```java
@RequestMapping(value = "/singleArg/PostDemo")
public void singleArgPostDemo(@RequestBody String name);
```
这个注解就很强势了，你写post，不写或者写get都没用，不会生效的，只要有这个方式那他就是Post请求了。

#### 啥也不写
```java
@RequestMapping(value = "/singleArg/PostDemo")
public void singleArgPostDemo(String name);
```
此时默认会在参数前加上RequestBody注解。然后就会变成Post请求。


### 多参
#### get请求
多个参数也是使用@RequestParam注解。

```java
@RequestMapping(value = "/moreArgs/getDemo")
public void moreArgGetDemo(@RequestParam String name,@RequestParam String sex);
```
使用了RequestParam注解，默认method就是get。

#### post请求
多个参数只能有一个是requestBody方式，其他应该使用requestParam方式。

```java
@RequestMapping(value = "/moreArgs/postDemo")
public void moreArgPostDemo(@RequestBody String name,@RequestParam String sex);
```
也可以全部使用RequestParam方式，但是要指定post。

```java
@RequestMapping(value = "/moreArgs/postDemo",method = RequestMethod.POST)
public void moreArgPostDemo(@RequestParam String name,@RequestParam String sex);
```
如果要是参数前，都没写注解，则会报错，因为会默认加上两个RequestBody。