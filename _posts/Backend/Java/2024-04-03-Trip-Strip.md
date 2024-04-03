---
title: "trip 과 strip 두 메서드간의 차이"
date: 2024-04-03 20:50:30 +0900
categories: [Beckend, Java]
tags: [Java, String]
---

문자열 앞뒤의 공백을 제거하기 위해서 java.lang.String Class의 trim(), strip() 메서드를 사용할 수 있다.
하지만, 위 두 개의 메서드는 비슷한듯 하지만 차이점이 존재한다.

## trip()

---

java.lang.String 클래스의 trim() 메서드는 앞뒤 공백을 제거한 문자열의 복사본을 리턴한다.

```java
public class StringSpace {
    public static void main(String[] args) {
        // 앞뒤로 공백이 있는 문자열
        String str = "  Hi Anna!     ";

        // 공백 제거 (trim())
        String trimStr = str.trim();
        String stripStr = str.strip();

        // 공백 제거 문자열 출력
        System.out.println("strip 문자열 : '" + stripStr + "'");
        System.out.println("trim 문자열 : '" + trimStr + "'");
    }
}

strip 문자열 : Hi Anna!
trim 문자열 : Hi Anna!
```

결과만 놓고 보자면 앞뒤의 공백을 제거하는 것으로 별 차이가 없어 보이지만 두 메서드는 제거하는 공백의 종류가 다르다.

유니코드에는 우리가 일반적으로 많이 사용하는 스페이스('\u0020'), 탭('\u0009) 등 외에도 더 많은 종류의 공백 문자들이 있는데, 여기서 두 메서드의 차이를 알수 있다.

- trim( ) : '\u0020' 이하의 공백들만 제거
- strip( ) : 유니코드의 공백들을 모두 제거

<br />

## stripLeading(), stripTrailing()

---

Java 11 이후로는 stripLeading(), stripTrailing() 메서드도 이용할 수 있게 되었다.
stripLeading() 메서드는 문자열 앞의 공백을 제거 해주고, stripTrailing() 메서드는 문자열 뒤의 공백을 제거 해준다.

```java
public class StringSpace {
    public static void main(String[] args) {
        // 앞뒤로 공백이 있는 문자열
        String str = "  Hi Anna!     ";

        // 공백 제거 (stripLeading(), stripTrailing())
        String stripLeadingStr = str.stripLeading();
        String stripTrailingStr = str.stripTrailing();

        // 공백 제거 문자열 출력
        System.out.println("원본 문자열 : '" + str + "'");
        System.out.println("stripLeading 문자열 : '" + stripLeadingStr + "'");
        System.out.println("stripTrailing 문자열 : '" + stripTrailingStr + "'");

    }
}
```
