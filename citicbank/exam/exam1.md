# 双重检查锁的单例模式
```java
public class Singleton {
        private static volatile Singleton singleton;

        private Singleton() {
        }

        public static Singleton getInstance() {
            if (singleton == null) {
                synchronized (Singleton.class) {
                    if (singleton == null) {
                        singleton = new Singleton();
                    }
                }
            }
            return singleton;
        }

    }
```
# HashMap 取值  一个value有三个abc字符串
```java

public class CountABC {
    public static void main(String[] args) {
        HashMap<Integer, String> map = new HashMap<>();
        map.put(1, "abcdabcabc");
        map.put(2, "abcdabc2abc");
        map.put(3, "abcdabc3232abc");
        map.put(4, "123abcdabcabc213123");
        map.put(5, "asdsfasdf");
        map.put(6, "asdabc,dfasabcabcasds");
        System.out.println(countABC(map));
    }

    private static int countABC(HashMap<Integer, String> map) {
        int i = 0;
        // HashMap 遍历方式
        // Iterator iterator = map.keySet().iterator();
        // while (iterator.hasNext()) {
        //     Integer key = (Integer) iterator.next();
        //     String value = map.get(key);
        //
        // }
        // // 遍历方式2  
        Iterator<Map.Entry<Integer, String>> iterator1 = map.entrySet().iterator();
        while (iterator1.hasNext()) {
            Map.Entry<Integer, String> entry = iterator1.next();
            String value1 = entry.getValue();
            int j;
            //  我想的一个方法是每遇到一个abc就把字符串截取一下。
            for (j=0; j < 3; j++) {
                int result = value1.indexOf("abc");
                if (result != -1) {
                    value1 = value1.substring(result + 3);
                } else {
                    break;
                }
            }
            if (j == 3) {
                i++;
            }
        }
        // 遍历方式3  jdk1.8
        // map.forEach((key,value)->{
        //     Integer key3 = key;
        //     System.out.println(key);
        //     String value3 = value;
        //     System.out.println(value3);
        //
        //
        // });
        return i;
    }
}
```

