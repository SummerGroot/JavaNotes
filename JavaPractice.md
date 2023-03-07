```java
class Test {
    public static void main(String[] args) {
        System.out.println(new B().getValue());
    }
    static class A {
        protected int value;
        public A (int v) {
            setValue(v);
        }
        public void setValue(int value) {
            this.value= value;
        }
        public int getValue() {
            try {
                value ++;
                return value;
            } finally {
                this.setValue(value);
                System.out.println(value);
            }
        }
    }
    static class B extends A {
        public B () {
            super(5);
            setValue(getValue()- 3);
        }
        public void setValue(int value) {
            super.setValue(2 * value);
        }
    }
}
```

链接：https://www.nowcoder.com/questionTerminal/43e8e12bc30e412e92340d11dff6077d?
来源：牛客网



思考和解决这个题的主要核心在于对java多态的理解。个人理解时，执行对象实例化过程中遵循多态特性 ==> 调用的方法都是***将要实例化的子类中\***的重写方法，只有明确调用了super.xxx关键词或者是子类中没有该方法时，才会去调用父类相同的同名方法。 

###   Step 1: new B()构造一个B类的实例 

  此时super(5)语句调用显示调用父类A带参的构造函数，该构造函数调用setValue(v)，这里有两个注意点一是虽然构造函数是A类的构造函数，但此刻正在初始化的对象是B的一个实例，**因此这里调用的实际是B类的setValue方法**，于是调用B类中的setValue方法 ==> 而B类中setValue方法显示调用父类的setValue方法，将B实例的value值设置为2 x 5 = **10**。
 紧接着，B类的构造函数还没执行完成，继续执行setValue(getValue()- 3)   // 备注1语句。 

  先执行getValue方法，B类中没有重写getValue方法，因此调用父类A的getValue方法。这个方法比较复杂，需要分步说清楚： 

1.    调用getValue方法之前，B的成员变量value值为10。    
2.    value++ 执行后， B的成员变量value值为11，此时开始执行到return语句，将11这个值作为getValue方法的返回值返回出去    
3.    但是由于getValue块被try finally块包围，因此finally中的语句无论如何都将被执行，所以步骤2中**11这个返回值会先暂存起来**，到finally语句块执行完毕后再真正返回出去。    
4.    这里有很重要的一点：finally语句块中 this.setValue(value)方法调用的是**B类的setValue方法**。为什么？因为此刻正在初始化的是B类的一个对象（运行时多态），就像最开始第一步提到的一样(而且这里用了使用了this关键词显式指明了调用当前对象的方法)。因此，此处会再次调用B类的setValue方法，同上，super.关键词显式调用A的setValue方法，**将B的value值设置成为了2 \* 11 = 22**。    
5.    因此第一个打印项为22。      
6.    finally语句执行完毕 会把刚刚暂存起来的11 返回出去，也就是说这么经历了这么一长串的处理，getValue方法最终的返回值是11。   

  回到前面标注了 **//备注1** 的代码语句，其最终结果为setValue(11-3)=>setValue(8)
 而大家肯定也知道，这里执行的setValue方法，将会是B的setValue方法。 之后B的value值再次变成了2*8 = 16; 

###   Step2: new B().getValue() 

  B类中没有独有的getValue方法，此处调用A的getValue方法。同Step 1， 

1.    调用getValue方法之前，B的成员变量value值为**16**。    
2.    value++ 执行后， B的成员变量value值为17，此时执行到return语句，会将17这个值作为getValue方法的返回值返回出去    
3.    但是由于getValue块被try finally块包围而finally中的语句无论如何都一定会被执行，所以步骤2中**17这个返回值会先暂存起来**，到finally语句块执行完毕后再真正返回出去。    
4.    finally语句块中继续和上面说的一样: this.setValue(value)方法调用的是**B类的setValue()方法将B的value值设置成为了2 \* 17 = 34**。    
5.    因此第二个打印项为34。     
6.    finally语句执行完毕 会把刚刚暂存起来的17返回出去。    
7.    因此new B().getValue()最终的返回值是17.   

###   Step3: main函数中的System.out.println 

  将刚刚返回的值打印出来，也就是第三个打印项：17  

  最终结果为 **22 34 17**。 如果朋友们在看的过程中仍然有疑问，可以亲自把代码复制进去ide，在关键语句打下断点，查看调用方法的对象以及运行时的对象值，可以有更深刻的理解。 

```java
public class Example{
    String str=new String("tarena");
    char[]ch={'a','b','c'};
    public static void main(String args[]){
        Example ex=new Example();
        ex.change(ex.str,ex.ch);
        System.out.print(ex.str+" and ");
        System.out.print(ex.ch);
    }
    public void change(String str,char ch[]){
   //引用类型变量，传递的是地址，属于引用传递。
        str="test ok";
        ch[0]='g';
    }
}
```

链接：https://www.nowcoder.com/questionTerminal/4ffbf0f9f23a4d7f88577638e372cd6a?
来源：牛客网

String str=new String("tarena");
语句执行的时候，首先会检查字符串常量池有没有“tarena”，如果没有，就在字符串常量池创建一个“tarena”，然后根据字符串常量池的“tarena”，在堆中创建一个“tarena”对象，这个str指向堆中的“tarena”对象；如果字符串常量池有“tarena”，就直接在堆中创建一个“tarena”对象。
总的来说，就是在堆中有一个str成员变量，指向堆中的一个地址A，地址A中的值是“tarena”。且由于字符串底层是final类型的数组实现，那么这个地址A的值“tarena”是不变的。str的值始终是“tarena”，除非将str指向其他地址。

char[]ch={"a"’,"b","c"};
同样，上面语句在堆中创建一个成员变量ch，并创建一个地址B，B中的值是数组的元素："a"、"b"、"c"，该成员变量ch指向地址B

public void change(String str,char ch[]){
//引用类型变量，传递的是地址，属于引用传递。
str="test ok";
ch[0]="g";
}
上面语句执行的时候，在栈中创建2个局部变量str、ch，局部变量str被赋予A地址的副本，指向A地址。而局部变量ch被赋予地址B的副本，指向地址B。

当 str="test ok"; 执行的时候，我们知道地址A的内容是不会变的，那么即使局部变量str指向地址A，它的值变化也不会引起地址A的值变化，而是在栈中开辟一个地址C，地址C的值是“test ok”，局部变量str指向地址C，而成员变量str仍然指向地址A，且值是“tarena”。

当 ch[0]="g"; 执行的时候，由于地址B的值可变，那么局部变量ch的值的变化，会导致地址B的值的变化，地址B第一个值就从"a"变为"g"，因此成员变量ch指向地址B，它的值也变化，为 ["g","b","c"]
