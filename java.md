## Debug java
```
-Djavax.net.debug=ssl:handshake:verbose
-Djavax.net.debug=all
```

## 泛型

extends 通配符 `add(List<? extends Number>)` 方法体内不能调用set(T)并传入引用，null除外。不能写。
super 通配符  `add(List< ? super Number>)` 方法体内不能调用T get()并返回T的引用，Object引用除外，不能读。
无限定通配符  `add(List<?>)` 方法体内既不能有extends的限制，又不能有super的限制。既不能读也不能写，只能做Null 判断。 <?>是所有<T>的超类，可以安全转型。
  
集合Connections的定义,dest只能写，src只能读。
```
public class Collections {
    public static <T> void copy(List<? super T> dest, List<? extends T> src) {
        for (int i=0; i<src.size(); i++) {
            T t = src.get(i); // src是producer
            dest.add(t); // dest是consumer
        }
    }
}
```

[Producer extends consumer super](https://www.liaoxuefeng.com/wiki/1252599548343744/1265105920586976)
