+++
title = "Java Optional 基本 & 心得分享"
author = "Allen Hsieh"
description = "最近我們部門在開發新的微服務都是使用 Java，但許多同事都是寫習慣 PHP 且之前沒有使用過 Java 。由於像 PHP 這種 Script Language 在處理 Undefined Variable 時，只是提示Warning，導致有的同事沒有良好的檢查 null 的習慣。有的同事抱怨 Null Pointer Exception 花了許多的時間在尋找，所以我決定在這次的新專案中，使用 Optional 來減少 Null Pointer Exception。"
featured = true
categories = ["JAVA"]
tags = [
    "JavaCore",
]
date = "2021-06-27"
aliases = ["java-optional-basic"]
images = ["images/java.jpeg"]
+++


最近我們部門在開發新的微服務都是使用 Java，但許多同事都是寫習慣 PHP 且之前沒有使用過 Java 。由於像 PHP 這種 Script Language 在處理 Undefined Variable 時，只是提示Warning，導致有的同事沒有良好的檢查 null 的習慣。有的同事抱怨 Null Pointer Exception 花了許多的時間在尋找，所以我決定在這次的新專案中，使用 Optional 來減少 Null Pointer Exception。這邊要特別注意，如果沒有正確的使用 Optional 的話，還是有可能會造成 Null Pointer Exception 喔！

> Null Reference is the Billion Dollar Mistake - Tony Hoare 

## 建立Optional 物件
---

大家認為下面的 Example1 運行的結果會是什麼樣的呢？

```java
import java.util.Optional;

public class Example1 {
    public static void main(String[] args) {
        String sentence = "I am Optional String";
        Optional<String> optionalSentence = Optional.of(sentence);
        System.out.println(optionalSentence.get());
    }
}
```

執行結果會是 "I am Optional String"，這邊可以看到 line 6 透過 of() method 建立了 一個 Optional 的物件。但如果 sentence 是 null 的時候，會怎麼樣呢？結果會是 Null Pointer Exception! 因為 of() method 這邊所帶入的 Value 必須是 non-null的，所以建議大家在使用 Optional 時，應該避開使用 of() method，而是使用 ofNullable() method。

現在我們將 Example1 中的 sentence 改成 null 並且使用 ofNullAble() method 來運行，這時大家覺得結果是怎樣呢？

```java
import java.util.Optional;

public class Example2 {
    public static void main(String[] args) {
        String sentence = null;
        Optional<String> optionalSentence = Optional.ofNullable(sentence);
        System.out.println(optionalSentence.get());
    }
}
```

大家是否會認為跟 Example1 一樣，會列印出"I am Optional String"？但實際執行的結果會是 NoSuchElementException : No value present，因為在 call get() method 時內部檢查 Value 是否存在，所以這邊建議在實際使用中，不要使用 get() method，除非你能確認 Value 一定不等於 null。
這邊要順便介紹 Optional 中特別的物件: Optional.empty() ，這邊要特別注意 Optional empty 還是 Optional 物件，只是裡面的 value 是 null，所以 Optional empty 跟 null 是不同的東西。

## Optional 安全取得Value
---

這邊一開始要介紹Optional 中的 isPresent()，透過 isPresent 我們可以安全地確認 Value 是否存在

```java
import java.util.Optional;

public class Example3 {
    public static void main(String[] args) {
        String sentence = "I am Optional String";
        Optional<String> optionalSentence = Optional.ofNullable(sentence);
        if (optionalSentence.isPresent()) {
            System.out.println(optionalSentence.get());
        } else {
            System.out.println("value is empty");
        }
    }
}
```

Example3 因為我們確認了 Value 已經存在 Optional 中了，所以會正常運行，但是否會覺得這樣寫法跟我們以前自己檢查一些值是否是 null 是一樣的呢？
這邊要介紹 orElse()

```java
import java.util.Optional;

public class Example4 {
    public static void main(String[] args) {
        String sentence = "I am Optional String";
        Optional<String> optionalSentence = Optional.ofNullable(sentence);
        String defaultValue = "value is empty";
        System.out.println(optionalSentence.orElse(defaultValue));
    }
}
```

大家可以看到 orElse() 當 Optional 物件的 value 是空的時候，會返回 defaultValue，這樣我們就可以不用寫 if else 了。
但有的人想 return 的 defaultValue 比較複雜怎麼辦? 我們也可以使用 orElseGet() 執行一個函示取得想要的預設值。

```java
import java.util.Optional;

public class Example5 {
    public static void main(String[] args) {
        String sentence = "I am Optional String";
        Optional<String> optionalSentence = Optional.ofNullable(sentence);
        System.out.println(optionalSentence.orElseGet(()-> getDefaultValue()));
    }
    private static String getDefaultValue() {
        return "I am default value";
    }
}
```

大家以前也一定會經常做一些必要的欄位檢查，如果都些值是 Null 就要丟特定的 Exception，這邊可以使用 orElseThrow()

```java
import java.util.Optional;

public class Example6 {
    public static void main(String[] args) throws Exception {
        String sentence = "I am Optional String";
        Optional<String> optionalSentence = Optional.ofNullable(sentence);
        System.out.println(optionalSentence.orElseThrow(()-> new myCustomException()));
    }
}
```

## Optional 安全執行特定函式
---

在之前的例子中，我們只是單純的想要正確的執行 System.out.println，這邊介紹ifPresent()。

```java
import java.util.Optional;

public class Example7 {
    public static void main(String[] args) throws Exception {
        String sentence = "I am Optional String";
        Optional<String> optionalSentence = Optional.ofNullable(sentence);
        optionalSentence.ifPresent(value -> {
            System.out.println(value);
        });
    }
}
```

這邊想稍微提一下 eta-conversion 的概念，但細節大家可以參考這篇 [如何理解並使用Java中雙冒號(::)運算操作符](https://read01.com/zh-tw/BJaAPOg.html#.YTMyBtMzZhF)
根據eta-conversion 可以寫成以下code

```java
import java.util.Optional;

public class Example8 {
    public static void main(String[] args) {
        String sentence = "I am Optional String";
        Optional<String> optionalSentence = Optional.ofNullable(sentence);
        //原本
        optionalSentence.ifPresent(value -> {
            System.out.println(value);
        });
        //因為只執行一個函式，所以可以將 {} 拿掉
        optionalSentence.ifPresent(value -> System.out.println(value));
        //因為只有的函式只有一個參數，透過 eta-conversion 可以寫成
        optionalSentence.ifPresent(System.out::println);
    }
}
```

ifPresent() 只能在 Optional 有 value 時執行，有時候我們也希望如果 Optional 有 value 時執行 函式 A，沒有值時 執行函式 B，這時我們就可以透過 ifPresentOrElse()

```java
import java.util.Optional;

public class Example9 {
    public static void main(String[] args) {
        String sentence = "I am Optional String";
        Optional<String> optionalSentence = Optional.ofNullable(sentence);
        optionalSentence.ifPresentOrElse(Example9::methodA, Example9::methodB);
    }

    public static void methodA(String value) {
        //do something with value
    }

    public static void methodB() {
        //do something without value
    }
}
```

## Optional 進階函式
---

Filter : 如果 isContainOptional return false，filter 的結果會是 Optional empty

```java
import java.util.Locale;
import java.util.Optional;

public class Example10 {
    public static void main(String[] args) {
        String sentence = "I am Optional String";
        Optional<String> optionalSentence = Optional.ofNullable(sentence);
        System.out.println(optionalSentence.filter(Example10::isContainOptional));
    }

    public static boolean isContainOptional(String input) {
        return input.toLowerCase(Locale.ENGLISH).contains("optional");
    }
}

/* output
Optional[I am Optional String]
*/
```

Map: 如果 Optional 的 Value 有值，這邊才會執行你所定義的 Mapping Function，如果沒有就直接返回 Optional Empty。 我在工作中，經常需要將 DTO 轉換成 Response 物件，會使用 Map 來使用

```java
import lombok.AllArgsConstructor;
import lombok.Getter;

@AllArgsConstructor
@Getter
class Person {
    private String firstName;
    private String lastName;
}
```

```java
import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.ToString;

@AllArgsConstructor
@Getter
@ToString
public class FullNamePerson {
    private String fullName;
}
```

```java
import java.util.Optional;

public class Example11 {
    public static void main(String[] args) {
        Optional<Person> person = Optional.ofNullable(new Person("allen", "hsieh"));
        Optional<FullNamePerson> fullNamePerson = person.map(Example11::convert);
        fullNamePerson.ifPresent(System.out::println);
    }

    private static FullNamePerson convert(Person person) {
        return new FullNamePerson(person.getFirstName() + " " + person.getLastName());
    }
}

/* output
FullNamePerson(fullName=allen hsieh)
*/
```

flatMap: 這邊我比較常用在取得 Optional 物件中的 Optional

```java
import lombok.AllArgsConstructor;
import java.util.Optional;

@AllArgsConstructor
class Person {
    private String firstName;
    private String lastName;

    public Optional<String> getFirstName() {
        return Optional.ofNullable(firstName);
    }

    public String getLastName() {
        return lastName;
    }
}
```

```java
import java.util.Optional;

public class Example12 {
    public static void main(String[] args) {
        Optional<Person> person = Optional.ofNullable(new Person("allen", "hsieh"));
        System.out.println(person.flatMap(Person::getFirstName));
        System.out.println(person.map(Person::getLastName));
    }
}

/* output
Optional[allen]
Optional[hsieh]
*/
```

## Class Field 是否使用 Optional

一開始我正在嘗試使用 Optional 時，由於Lombok 的 Getter 不可以設定將正常的 Object 返回 Optional，那時我曾經有在考慮是否要將 Class Field 直接需告成 Optional。最後在研究了一番以後，我發現 class field 不應該須告成 Optional ，以下兩個原因
1. Optional is not Serializable，這邊有 [Stack Overflow](https://stackoverflow.com/questions/24547673/why-java-util-optional-is-not-serializable-how-to-serialize-the-object-with-suc) 的討論，大家可以參考
2. 你不能避免別人將 Instance variable set 成 null ，還是有可能造成 Null Pointer Exception

```java
import lombok.Data;
import java.util.Optional;

@Data
class Person {
    private Optional<String> name;

    Person() {
    }
}
```

```java
import java.util.Optional;

public class Example13 {
    public static void main(String[] args) {
        Person person = new Person();
        person.setName(null);

        Optional<String> optionalName = person.getName();
        optionalName.ifPresent(System.out::println);
    }
}
```

這邊由於 getName return 的 Optional 是由外面帶入的，並不能控制返回的物件一定是 Optional，所以在 Line 8 的 optionalName 會是 null。當 Line 9 的 ifPresent() 執行時，會造成 Null Pointer Exception。 Optional 的使用是為了減少 null 的情況，所以好的設計是在拿到 Optional 物件時，不需要在檢查是否是 null。這也同時表示函示的參數，不應該有 Optional 帶入，因為有可能會是 Null 的情況

## 總結
---
這邊小小的總結一下我目前使用 Optional 的規則
1. 避免使用 of() & get() 以免造成 null pointer exception
2. 函式返回的物件，有可能是 null 的情況，選擇使用 Optional 物件返回
3. Class Field 不使用 Optional
4. 函式參數不已 Optional 物件帶入
