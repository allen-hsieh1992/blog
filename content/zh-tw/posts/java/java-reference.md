+++
title = "你知道 Java 的四個引用嗎？"
author = "Allen Hsieh"
description = "使用 Java 也有一段時間了，之前一直不知道有 java.lang.ref 這包 package 的存在，直到最近在研究 Java Garbage Collection 的時候，才剛好知道原來 Java 的引用還有細分成四個，分別由強到弱為強引用、軟引用、弱引用和虛引用"
featured = true
categories = ["JAVA"]
tags = [
"JavaCore"
]
date = "2022-01-27"
aliases = ["java-reference"]
images = ["images/java.jpeg"]
+++

使用 Java 也有一段時間了，一直不知道有 java.lang.ref 這包 package 的存在，直到最近在研究 Java [Garbage Collection]({{< relref "/content/zh-tw/posts/java/java-garbage-collection.md" >}}) 的時候，才剛好知道原來 Java 的引用還有細分成四個，分別由強到弱為:
* 強引用 (Strong Reference)
* 軟引用 (Soft Reference)
* 弱引用 (Weak Reference) 
* 虛引用 (Phantom Reference)

---

## 強引用
---
相信大家對強引用一定不陌生，我們正常建立新的物件時所使用的引用都是強引用。當記憶體中的物件有強引用時，Garbage Collection 就一定不會回收此物件。甚至當記憶體不足時 Java Virtual Machine 寧願噴出 OutOfMemoryError 的錯誤，也不會清除它。

```Java
package com.javacore.ref;

import java.util.ArrayList;
import java.util.List;

public class StrongRef {
    static class User {}

    public static void main(String[] args) {
        List<User> userList = new ArrayList<>();
        while (userList != null) {
            User user = new User();
            userList.add(user);
        }
    }
}
```

上面的 While Loop 的 User(Line12) 就是強引用，雖然 Loop 中 User 會一直指向新的物件，但是 Line13 將 User 加入 List 中也是強引用，所以所有的 User 都會一直保留在記憶體中。我使用 10 MB 的 Heap 執行 (jvm option -Xmx10m)，得到下面的結果，不出意外一下就空間不足了。

```bash
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at java.base/java.util.Arrays.copyOf(Arrays.java:3720)
	at java.base/java.util.Arrays.copyOf(Arrays.java:3689)
	at java.base/java.util.ArrayList.grow(ArrayList.java:238)
	at java.base/java.util.ArrayList.grow(ArrayList.java:243)
	at java.base/java.util.ArrayList.add(ArrayList.java:486)
	at java.base/java.util.ArrayList.add(ArrayList.java:499)
	at com.javacore.ref.StrongRef.main(StrongRef.java:13)
```

## 軟引用
---
軟引用跟強引用最大的不同之處就是，當 Garbage Collection 認為記憶體有壓力時，才會將軟引用的物件給清除。也就代表 Garbage Collection 執行時，不一定會從記憶體清除。<br>
注意:當記憶不足時，JVM 會先嘗試清除軟引用，但如果清除完以後還是不足，還是為噴 OutOfMemoryError

```Java
package com.javacore.ref;

import java.lang.ref.SoftReference;

public class SoftRef {

    static class User {}

    public static void main(String[] args) {
        System.out.println("Available Memory Size " + Runtime.getRuntime().maxMemory());
        System.out.println("Available Memory Size " + Runtime.getRuntime().totalMemory());

        User user = new User();
        SoftReference<User> userSoftReference = new SoftReference<>(user);
        user = null;

        System.out.println(userSoftReference.get());
        System.gc();
        System.out.println(userSoftReference.get());

        byte[] a = new byte[1024 * 1024 * 3];
        while (null != userSoftReference.get()) {
            System.out.println("Free Memory Size " + Runtime.getRuntime().freeMemory());
            byte[] b = new byte[1024 * 1000];
        }
        System.out.println("The soft reference " + userSoftReference.get());
    }
}
```

這邊使用 5MB 的 Heap 來執行(JVM Option -Xmx5m)，得到以下的結果。

```Bash
Available Memory Size 6291456
Available Memory Size 6291456
com.javacore.ref.SoftRef$User@71be98f5
com.javacore.ref.SoftRef$User@71be98f5
Free Memory Size 1222280
Free Memory Size 185952
Free Memory Size 185952
The soft reference null
```

從上面的 Code 來看
* Line13 一開始強引用 User
* Line14 建立一個新的軟引用到同一個 User
* Line15 將原本的強引用給釋放出來，所以 Line13 建立的 User ，已經沒有任何強引用，只剩下 Line14 的軟引用
* Line17 嘗試從軟引用拉出資料，執行的結果當下會拿到 User 的物件
* Line19 雖然 Line18 強制呼叫 JVM 執行了 Garbage Collection，但由於記憶體沒有壓力，所以還是取得到 User 的物件。
* Line21-25 嘗試給記憶體施壓，直到 Garbage Collection 將 User 從記憶體清除

## 弱引用
---
弱引用跟軟引用最大的不同點就是，當 Garbage Collection 產生時，不管記憶體是否足夠，都一定會將弱引用的物件給清除掉。

```Java
package com.javacore.ref;

import java.lang.ref.WeakReference;

public class WeakRef {
    static class User {}

    public static void main(String[] args) {
        User user = new User();
        WeakReference<User> userWeakReference = new WeakReference<>(user);
        user = null;

        System.out.println(userWeakReference.get());
        System.gc();
        System.out.println(userWeakReference.get());
    }
}
```

從上面的 Code 來看
* Line9 一開始強引用 User
* Line10 建立一個新的弱引用到同一個 User
* Line11 將原本的強引用給釋放出來，所以 Line9 建立的 User ，已經沒有任何強引用，只有弱引用
* Line13 如果 Garbage Collection 還沒跑的話，弱引用會拿到 User 的物件，所以當拿到 Null 時不要感到意外，因為已經 GC 掉了。
* Line15 這時弱引用正常會拿到 Null ，因為我們在 Line14 請JVM 執行了 GC

由於這邊不管記憶體大小都會清除，所以就沒特別限制 Heap 的Size，Code 執行結果如下
```Bash
com.javacore.ref.WeakRef$User@3f0ee7cb
null
```

## ReferenceQueue
---
在介紹虛引用前，這邊想先介紹 ReferenceQueue。ReferenceQueue 是當物件被 GC 以後，Reference 會被放到這個 Queue。是不是有點抽象，我們用以下的例子來看

```Java
package com.javacore.ref;

import java.lang.ref.ReferenceQueue;
import java.lang.ref.WeakReference;

public class ReferenceQueueExample {
    static class User {
    }

    public static void main(String[] args) {
        User user = new User();
        ReferenceQueue<User>  queue = new ReferenceQueue<>();
        WeakReference<User> weakReference = new WeakReference<>(user, queue);

        System.out.println("try to pull from queue " + queue.poll());
        user = null;
        System.gc();
        Object result = null;
        while (result == null) {
            System.out.println("trying to get from queue");
            result = queue.poll();
        }
        System.out.println("try to pull from queue " + result);
        System.out.println("try to get from weakReference " + weakReference.get());
    }
}
```

執行結果可以看出，有時 GC 並不是這麼即時，所以會有需要等一下之後才會放到Queue中。

```Bash
try to pull from queue null
trying to get from queue.
trying to get from queue.
trying to get from queue.
trying to get from queue.
try to pull from queue java.lang.ref.WeakReference@7e0ea639
try to get from weakReference null
```

## 虛引用
---
虛引用跟其他引用不同，它就跟名字如同虛設，在 get() 函式永遠回返回 Null。那它究竟有什麼用？ 它跟前面介紹的 [ReferenceQueue](#referencequeue) 是一起使用的，可以用來追中物件的回收的生命週期。在 Java9 時 finalize 的函式已經被deprecated，建議使用 虛引用加上 ReferenceQueue 取代

```Java
package com.javacore.ref;

import java.lang.ref.PhantomReference;
import java.lang.ref.Reference;
import java.lang.ref.ReferenceQueue;

public class PhantomRef {
    static class TraceObject {}

    static class PhantomTraceObject extends PhantomReference<TraceObject> {
        PhantomTraceObject(TraceObject traceObject, ReferenceQueue<? super  TraceObject> queue) {
            super(traceObject, queue);
        }

        public void cleanup() {
            System.out.println("trace object is cleaning up ");
        }
    }

    public static void main(String[] args) throws InterruptedException {
        ReferenceQueue<TraceObject> referenceQueue = new ReferenceQueue();
        TraceObject data = new TraceObject();
        PhantomTraceObject phantomTraceObject = new PhantomTraceObject(data, referenceQueue);
        data = null;

        System.gc();
        Reference<? extends TraceObject> referenceFromQueue;
        if ((referenceFromQueue = referenceQueue.remove()) != null) {
            ((PhantomTraceObject) referenceFromQueue).cleanup();
        }
    }
}
```

執行結果如下，line28 可以看到我使用 ReferenceQueue 中的 remove() 而非之前 [ReferenceQueue](#referencequeue) 中的 poll()。 remove() 和 poll() 最大的差別在於，remove() 會卡在那邊等 queue 裡面有資料而 poll() 是拉當下的結果。

```bash
trace object is cleaning up 
```

## WeakHashMap
---
這邊最後要介紹一個特別的 HashMap WeakHashMap，看到Weak大家一定就知道他是弱引用，它是將 Key 當成弱引用，所以存在 Map 裡面的值是會被 GC 的。

```Java
package com.javacore.ref;

import java.util.WeakHashMap;

public class WeakHashMapExample {
    static class Process {}
    static class Thread {}

    public static void main(String[] args) {
        Process process = new Process();
        Thread thread = new Thread();

        WeakHashMap<Process, Thread> map = new WeakHashMap<Process, Thread>();
        map.put(process, thread);
        process = null;

        System.out.println("Does map contain the thread: " + map.containsValue(thread));
        System.gc();
        while (map.containsValue(thread)) {
            System.out.println("map still contains the value");
        }
        System.out.println("The map now is empty " + map.isEmpty());
    }
}
```

執行的結果如下，從結果可以看出，當 line15 process 沒有強引用以後，map 裡面的內容就會消失。 
```bash
Does map contain the thread: true
map still contains the value
The map now is empty true
```

## 總結
---
雖然 Java 裡面有弱引用和軟引用的方式可以當作短暫的緩存，但緩存的時間是不可控的，因為是控制在 Java 的 Garbage Collection。而且大量的緩存也會造成更頻繁的GC。
目前還沒有在實務上使用的經驗，所以如果將來有幸有用到再來分享更多細節。不過目前個人觀點，這應該只適合用在 client Side，Web Server 還是建議使用像 Redis 或 Memcache 這些 Cache 的技術。
