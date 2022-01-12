**Map类**

Map是java中的一个类，**Map.Entry是Map内部的一个接口。**

Map提供了一些常用方法，如keySet()、entrySet()等方法。


keySet()方法返回值是Map中key值的集合；entrySet()的返回值也是返回一个Set集合，此集合的类型为Map.Entry。


Map.Entry是Map声明的一个内部接口，此接口为泛型，定义为Entry<K,V>。**它表示Map中的一个实体（一个key-value对）**。接口中有getKey(),getValue方法。

```java
  Map<String, String> map = new HashMap<String, String>();    
  map.put("key1", "value1");    
  map.put("key2", "value2");    
  map.put("key3", "value3");    
      
  //第一种：普遍使用，二次取值    
  System.out.println("通过Map.keySet遍历key和value：");    
  for (String key : map.keySet()) {    
   System.out.println("key= "+ key + " and value= " + map.get(key));    
  }    
      
  //第二种    
  System.out.println("通过Map.entrySet使用iterator遍历key和value：");    
  Iterator<Map.Entry<String, String>> it = map.entrySet().iterator();    
  while (it.hasNext()) {    
   Map.Entry<String, String> entry = it.next();    
   System.out.println("key= " + entry.getKey() + " and value= " + entry.getValue());    
  }    
      
 //第三种：推荐，尤其是容量大时</span>    
  System.out.println("通过Map.entrySet遍历key和value");    
  for (Map.Entry<String, String> entry : map.entrySet()) {    
   System.out.println("key= " + entry.getKey() + " and value= " + entry.getValue());    
  }    
    
  //第四种    
  System.out.println("通过Map.values()遍历所有的value，但不能遍历key");    
  for (String v : map.values()) {    
   System.out.println("value= " + v);    
  }
```

————————————————
原文链接：https://blog.csdn.net/yaomingyang/article/details/78748130