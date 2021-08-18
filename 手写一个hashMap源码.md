手写一个hashMap源码

```java
package futian.yu.com
    
public interface Map<K,V>{
    V put(K k,V v);
    V get(K k);
    int size();
    
    interface Entry<K,V>{
        K getKey();
        V getValue();
    }
}
```

```java
package futian.yu.com

public class HashMap<K,V> implements Map<K,V>{
    Entry<K,V> table[] = null;
    int size = 0;
    
    public HashMap(){
        table = new Entry[16];
    }
    
    /**
     * 通过key进行hash hashCode取模
     * 下标取数组下表对应的Entry对象
     * 对象是否为空？为空可以直接存储
     * 不为空 用链表来解决
     * 赋值返回
     *
     */
    @Override
    public V put(K k,V v){
        int index = hash(k);
        Entry<K,V> entry = table[index];
        if(enrty == null){
			//没有冲突 直接存
            table[index] = new Entry<>(k,v,index,null);
        }else{
            //冲突 链表存
            table[index] = new Entry<>(k,v,index,entry);
        }
        return table[index].getValue;
    }
    
    private int hash(K k){
        int i = k.hashCode % 15;
        return r>=0 ? i : -i;
    }
    
    @Override
    public V get(K k){
        if(size == 0){
            return null;
        }
        int index = hash(k);
        //根据key获取value
        Entry<K,V> entry = findValue(k,table[index]);
        return entry == null ? null : entry.getValue();        
    }
    
    public Entry<K,V> findValue(K k, Entry<K,V> entry){
        if(k.equals(entry.getKey) || k == entry.getKey){
            return entry;
        }else{
            if(entry.next != null){
                return findValue(k,entry.next);//递归调用
            }
        }
        return null;
    }
    
    @Override
    public int size(){
        return size;
    }
    
    class Entry<K,V> implements Map.Entry<K,V>{
        K k; V v; int hash; Entry<K,V> next;
        public Entry(K k, V v, int hash, Entry<K,V> next){
            this.K = k;
            this.V = k;
            this.hash = hash;
            this.next = next;
        }
        
        @Override
        public V getKey(){
        	return k;    
        }
        @Override
        public V getValue(){
            return v;
        }
        
    }
    
}
```

