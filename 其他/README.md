# 关于本篇

本篇是搜集记录通用的边角知识点。

# 闭包

## 作用

用于在全局想持续访问局部变量，将子函数作为一个变量返回，就可以通过这个返回值，获取到局部变量了。

## 举例

在javaScript中是通过函数嵌套来实现的，而在java中是通过“接口+内部类”来实现的。

```javascript
function aa() {
    let age = 16;
    return function bb() {
        return age;
    }
}

let userAge = aa()();/*两个括号表示调用返回的函数bb*/
console.log(userAge);
userAge = null;/*释放闭包，闭包会导致资源不被回收，因此要设置成null*/
```

```java
public class DemoClass1 {
    private int length =0;

    //private|public
    private class InnerClass implements ILog
    {
        @Override
        public void Write(String message) {
            //DemoClass1.this.length = message.length();
            length = message.length();
            System.out.println("DemoClass1.InnerClass:" + length);
        }
    }

    public ILog logger() {
        return new InnerClass();/*返回接口类似于js返回函数*/
    }

    public static void main(String[] args){
        DemoClass1 demoClass1 = new DemoClass1();
        demoClass1.logger().Write("abc");

        //.new
        DemoClass1 dc1 = new DemoClass1();
        InnerClass ic = dc1.new InnerClass();
        ic.Write("abcde");
    }
}

/*
结果如下：
DemoClass1.InnerClass:3
DemoClass1.InnerClass:58
*/
```

相当于其他函数能够通过这种方式，访问了类DemoClass1的private属性；

# 进栈顺序一定如何计算出栈次序的种类

卡特兰数：满足以下性质：

令h(0)=1,h(1)=1，catalan数满足递推式。h(n)= h(0)*h(n-1)+h(1)*h(n-2) + … + h(n-1)h(0) (n>=2)。也就是说，如果能把公式化成上面这种形式的数，就是卡特兰数。

可以简化成
$$
h(n) =  C{n\choose2n}/(n+1)
$$

$$
A(m,n) = n*(n-1)*(n-2)*···*（n-m+1）=n!/(m!*(n-m)!)
$$

$$
C(m,n) = A(m,n)/A(m,m)
$$

那么一个足够大的栈的进栈序列为1,2,3,⋯,n时有多少个不同的出栈序列？

我们可以这样想，假设k是最后一个出栈的数。比k早进栈且早出栈的有k-1个数，一共有h(k-1)种方案。比k晚进栈且早出栈的有n-k个数，一共有h(n-k)种方案。所以一共有h(k-1)*h(n-k)种方案。显而易见，k取不同值时，产生的出栈序列是相互独立的，所以结果可以累加。k的取值范围为1至n，所以结果就为h(n)= h(0)*h(n-1)+h(1)\*h(n-2) + … + h(n-1)h(0)

出栈入栈问题有许多的变种，比如n个人拿5元、n个人拿10元买物品，物品5元，老板没零钱。问有几种排队方式。熟悉栈的同学很容易就能把这个问题转换为栈。值得注意的是，由于每个拿5元的人排队的次序不是固定的，所以最后求得的答案要\*n!。拿10元的人同理，所以还要*n!。所以这种变种的最后答案为h(n)*n!*n!。

类似问题还有很多具体见[博客](http://lanqi.org/skills/10939/)



