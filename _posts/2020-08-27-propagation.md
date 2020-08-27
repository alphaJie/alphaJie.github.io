---
layout: post
title:  "Spring事务传播行为和隔离级别"
date:   2020-08-27 11:58:23 +0530
categories: Spring Transactional
---
## 事务传播行为
当我们在一个方法上增加 @Transactional 时，Spring就会使用AOP的思想，在方法执行前先开启事务，执行完毕后，根据方法是否抛出异常决定提交还是回滚事务。  
事务传播行为就是说，调用一个加了事务的方法A，或者方法A的代码内调用加了方法B，此时事务的边界是什么。Spring规定了事务传播行为有以下7种：  
- PROPAGATION_REQUIRED：如果当前没有事务，就创建一个新事务，如果当前存在事务，就加入该事务。默认传播行为，该设置是最常用的设置。
- PROPAGATION_REQUIRES_NEW：创建新事务，无论当前存不存在事务，都创建新事务。偶尔使用。
- PROPAGATION_NESTED：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则按REQUIRED属性执行。偶尔使用
- PROPAGATION_SUPPORTS：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就以非事务执行
- PROPAGATION_MANDATORY：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就抛出异常。
- PROPAGATION_NOT_SUPPORTED：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
- PROPAGATION_NEVER：以非事务方式执行，如果当前存在事务，则抛出异常。

## 代码说明
```java
public class PropagationTest{
    @Transactional(propagation = PROPAGATION_REQUIRED)    
    public void methodA(){
        doSomethingPre();
        methodB();
        doSomethingAfter();
    }
    @Transactional(propagation = PROPAGATION_REQUIRED)
    public void methodB(){
        //doSomething
    }   
}
public class ProController{
    public void doService(){
        methodA();
    }
}
```
**情况一 都使用REQUIRED**  
此种只会开启一个事务,步骤  
1. 开启一个事务
2. 执行methodA的代码，接着执行methodB的代码
3. 提交或回滚事务

**情况二 methodB使用REQUIRES_NEW**  
开启两个事务，两个事务互不影响，步骤  
1. 开启事务1
2. 执行methodA的代码doSomethingPre()
3. 开始事务2
4. 执行methodB里的代码
5. 提交或回滚事务2
6. 执行methodA的代码doSomethingAfter()
7. 提交或回滚事务1(此时methodB的事务已提交，不会再回滚)

doService()方法如果直接调用methodB，则开启一个事务，执行代码。

**情况三 methodB使用NESTED**  
开启两个事务，外层事务回滚影响内层事务，内层事务回滚不影响外层，步骤  
1. 开启事务
2. 执行methodA的代码doSomethingPre()
3. 设置回滚点，savePoint
4. 执行methodB里的代码
5. 如果methodB代码异常，回滚到savePoint位置
6. 执行methodA的代码doSomethingAfter()
7. 提交或回滚事务(如果回滚，methodA和methodB的执行结果都会回滚)  

doService()方法如果直接调用methodB，则开启一个事务，执行代码。

**情况四 methodB使用SUPPORTS**  
methodA使用默认事务，methodB使用SUPPORTS，步骤    
1. 开启一个事务
2. 执行methodA的代码，接着执行methodB的代码
3. 提交或回滚事务

doService()方法直接调用methodB，则执行methodB代码，不使用事务。

**情况五 methodB使用MANDATORY**  
methodA使用默认事务，methodB使用MANDATORY，步骤    
1. 开启一个事务
2. 执行methodA的代码，接着执行methodB的代码
3. 提交或回滚事务

doService()方法直接调用methodB，则抛出异常。

**情况六 methodB使用NOT_SUPPORTED**  
methodA使用默认事务，methodB使用MANDATORY，步骤  
1. 开启事务
2. 执行methodA的代码doSomethingPre()
3. 事务挂起
4. 执行methodB里的代码
5. 事务恢复
6. 执行methodA的代码doSomethingAfter()
7. 提交或回滚事务(methodB里数据库操作结果已生效，如果回滚，只回滚methodA的代码的操作)  

doService()方法如果直接调用methodB，则执行methodB代码，不使用事务。

**<font color=#FF0000>情况七 methodB使用NEVER</font>**  
methodA使用默认事务，methodB使用NEVER，步骤 
1. 开启事务
2. 执行methodA的代码doSomethingPre()
3. 抛出异常，不执行methodB
4. 回滚事务

doService()方法如果直接调用methodB，则执行methodB代码，不使用事务。

## 事务隔离级别
是指当多个事务同时执行，并且操作同一个数据时，事务对此数据的占有情况。并发可能会导致以下问题：  
**脏读（Dirty read）**：脏读发生在一个事务读取了被另一个事务改写但尚未提交的数据时。如果这些改变在稍后被回滚了，那么第一个事务读取的数据就会是无效的。  
**不可重复读（Nonrepeatable read）**：不可重复读发生在一个事务执行相同的查询两次或两次以上，但每次查询结果都不相同时。这通常是由于另一个并发事务在两次查询之间更新了数据。  
**幻读（Phantom reads）**：幻影读和不可重复读相似。当一个事务（T1）读取几行记录后，另一个并发事务（T2）插入了一些记录时，幻影读就发生了。在后来的查询中，第一个事务（T1）就会发现一些原来没有的额外记录。  

**隔离级别**  
默认 @Transactional(isolation = Isolation.DEFAULT)  
- READ_UNCOMMITTED：可读取其他事务未提交数据(会出现脏读, 不可重复读) 基本不使用    
- READ_COMMITTED：可读取其他事务已提交数据(会出现不可重复读和幻读) 
- SERIALIZABLE：串行化，独占数据，事务1提交之后，事务2才可执行。 
- REPEATABLE_READ：可重复读(会出现幻读)，MySQL默认级别   
- DEFAULT：默认使用后端数据库的隔离级别   

## 注意事项
- @Transactional 只能被应用到public方法上, 对于其它非public的方法,如果标记了@Transactional也不会报错,但方法没有事务功能.
- @Transactional只是标签，要使用事务需要开始事务功能\<tx:annotation-driven/>
- spring默认只有需要运行时异常才会回滚事务（即RuntimeException下子类），而其他受检异常不回滚，可通过rollbackFor参数指定回滚异常
- 推荐在具体的类（或类的方法）上使用 @Transactional 注解，因为注解不能被继承，在接口上使用@Transactional时，只有当你设置了基于接口的代理时它才生效
- 可以通过timeout参数设置事务超时时间，避免大事务，默认30秒