## vue3为什要用proxy
**Object.defineProperty的缺点**
1. Object.defineProperty的第一个缺陷,无法监听数组变化。 但是vue中是可以监听数组的变化的，那他是怎么实现的呢？使用了以下八种方法：
```javascript
push()
pop()
shift()
unshift()
splice()
sort()
reverse()
```
2. Object.defineProperty的第二个缺陷,只能劫持对象的属性,因此我们需要对每个对象的每个属性进行遍历，如果属性值也是对象那么需要深度遍历,显然能劫持一个完整的对象是更好的选择。

除了上面讲，在数据量庞大的情况下Object.defineProperty的两个缺点外，Object.defineProperty还有以下缺点。

1、无法监听es6的Set、WeakSet、Map、WeakMap的变化；

2、无法监听Class类型的数据；

3、属性的新加或者删除也无法监听；

4、数组元素的增加和删除也无法监听

**vue3为什么用proxy？**

1. proxy可以直接监听数组的变化；
2. proxy可以监听对象而非属性.它在目标对象之前架设一层“拦截”，外界对该对象的访问，都必须先通过这层拦截，因此提供了一种机制，可以对外界的访问进行过滤和改写。
Proxy直接可以劫持整个对象,并返回一个新对象。

**总结**

- Proxy返回的是一个新对象,我们可以只操作新的对象达到目的,而Object.defineProperty只能遍历对象属性直接修改；
- Proxy作为新标准将受到浏览器厂商重点持续的性能优化，也就是传说中的新标准的性能红利
- 当然,Proxy的劣势就是兼容性问题,而且无法用polyfill实现
