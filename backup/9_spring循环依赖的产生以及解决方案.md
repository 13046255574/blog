# [spring循环依赖的产生以及解决方案](https://github.com/13046255574/blog/issues/9)

# spring循环依赖的产生以及解决方案

## 循环依赖的产生
以下面代码为例 A依赖B，B依赖A
```java
@Component
public class A {
  private B b;
  public void setB(B b) {
    this.b = b;
  }
}

@Component
public class B {
  private A a;
  public void setA(A a) {
    this.a = a;
  }
}
```
- Spring发现A对象，实例化A，没有放入spring对象池（一级缓存）
- 发现B属性，进行注入，去spring对象池寻找
- 没有找到，进入B属性，进行实例化B
- 然后在获取A属性的时候，也没有从Spring对象池中找到，这时又会进入A进行创建
- 至此，循环依赖产生了A->B->A->B...的过程

## 一二三级缓存介绍
- singletonObjects一级缓存：用于存放实例化并初始化好的bean
- earlySingletonObjects二级缓存：用于存放实例化，但没初始化好的bean引用
- singletonFactories三级缓存：用于存放实例化好但没有初始化的bean工厂

```java
	/** Cache of singleton objects: bean name to bean instance. */
	private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

	/** Cache of early singleton objects: bean name to bean instance. */
	private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);

	/** Cache of singleton factories: bean name to ObjectFactory. */
	private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);
```

## 三级缓存的解决思路：
还是以上面的A、B为例
- Spring发现A对象，实例化A，没有放入Spring对象池(一级缓存)，把A放进去三级缓存
- 发现B属性，去spring对象池去找
- 没有找到，进入B属性，实例化B，把B放入三级缓存
- B在获取A属性的时候，从三级缓存中获取到半成品A，将其属性注入，此时B完成实例化和初始化，放到一级缓存中，并返回
- 接着A获取到B对象，进行属性注入，于是A也完成实例化和初始化会放到一级缓存中

> 在B进行属性注入的时候，获取A对象时，会从三级缓存中先获取半成品A，只有实例化，还没初始化的A。这就打破了循环。

