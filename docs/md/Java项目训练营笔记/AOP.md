aop 切面
动态代理项目中的方法
## java中分为jdk动态代理,cglib动态代理

- jdk只能代理接口
- cglib代理类，不能代理final类
- 两者本质上都是生成新的class类
### JDK动态代理

1. jdk 会实现接口,相当于一个新的实现类
2. jdk继承了Proxy类
### cglib

1. 继承了实现类
2. 实现了Factory类

cglib性能优于jdk

jdk依然使用了被代理类调用方法
而cglib则使用自己本身的方法
```java
public void test_speed() {
    int size = 500;

    long start = System.nanoTime();
    AccountServiceImpl[] unProxiedServices = new AccountServiceImpl[size];
    for (int i = 0; i < size; i++) {
        unProxiedServices[i] = new AccountServiceImpl();
    }
    long end = System.nanoTime();
    System.out.println("不使用创建代理耗时" + (end - start));

    start = System.nanoTime();
    AccountService[] jdkProxiedServices = new AccountService[size];
    AccountService accountService;
    for (int i = 0; i < size; i++) {
        accountService = new AnnotationConfigApplicationContext(JDKProxyConfig.class).getBean(AccountService.class);
        jdkProxiedServices[i] = accountService;
    }
    end = System.nanoTime();
    System.out.println("jdk动态代理创建代理耗时" + (end - start));

    start = System.nanoTime();
    AccountService[] cglibProxiedServices = new AccountService[size];
    for (int i = 0; i < size; i++) {
        accountService = new AnnotationConfigApplicationContext(CglibProxyConfig.class).getBean(AccountService.class);
        cglibProxiedServices[i] = accountService;
    }
    end = System.nanoTime();
    System.out.println("cglib动态代理创建代理耗时" + (end - start));

    execute_time(unProxiedServices);
    execute_time(jdkProxiedServices);
    execute_time(cglibProxiedServices);

}

private void execute_time(AccountService[] accountServices) {
    long start = System.nanoTime();
    Account account1 = new Account("account1", 101);
    Account account2 = new Account("account2", 102);
    for (int i = 0; i < accountServices.length; i++) {
        accountServices[i].transfer(account1, account2, 1);
    }
    long end = System.nanoTime();
    System.out.println(end - start);
}
```
