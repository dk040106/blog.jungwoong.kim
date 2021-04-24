---
layout: article
title: Why you should not use .clone() on 2D array (Java)
tags:
  - Java
  - English
key: no-java-2d-clone
---

## Intro

오늘 수업시간에 자바로 코드를 짜는 중에 2D array를 사용하는 부분이 있었다. 그런데 내가 원하는 대로 작동하지 않고, 뭔가 array가 잘 복사되지 않는 듯 한 상황이 발생하는 것이었다. 이 글에서는 내가 오늘 겪은 상황에 대한 설명과 그 해결책을 제시한다.

Java의 Object class에는 .clone() 이라는 메소드가 존재한다. 이름에서 보면 알 수 있듯이 그 instance와 내용물을 복사하여 새로운 instance를 리턴하는 메소드이다. Object는 class이지만, Java에서는 array에서도 같은 이름의 메소드를 제공한다.

## 1차원 배열

1차원 배열의 경우 arr.clone()을 하면 내부의 데이터가 잘 복사된다.

```java
int[] arr1 = { 1, 2, 3, 4, 5 };
int[] arr2 = arr1.clone();

printArray(arr1);
printArray(arr2);
```

```
Arr1: 1 2 3 4 5
Arr2: 1 2 3 4 5
```

다음 코드를 보면, arr.clone()은 arr과 완전히 다른 배열이라는 것을 알 수 있다.

```java
arr1[1] = 100;
arr2[3] = -100;

printArray(arr1);
printArray(arr2);
```

```
Arr1: 1 100 3 4 5
Arr2: 1 2 3 -100 5
```

## 2차원 배열

1차원 배열에서 잘 작동했으니, 2차원 배열에서도 같은 방법으로 clone을 해보자. 역시 .clone()을 하면 잘 작동하는 것 처럼 보인다.

```java
int[][] arr3 = { { 1, 2, 3 }, { 2, 3, 4 } };
int[][] arr4 = arr3.clone();

printArray(arr3);
printArray(arr4);
```

```

Arr3:
1 2 3
2 3 4

Arr4:
1 2 3
2 3 4

```

그렇지만, 아래 코드를 보면 두 2차원 배열에서 .clone()이 예상한 대로 작동하지 않고 있다는 것을 알 수 있다. Deep copy를 하려고 했지만 Shallow copy가 되어 두 배열이 같은 값을 가리키고 있다.

```java
arr3[0][0] = 100;
arr4[1][2] = -100;

printArray(arr3);
printArray(arr4);
```

```
Arr3:
100 2 3
2 3 -100

Arr4:
100 2 3
2 3 -100
```

다음을 보면 arr3과 arr4는 서로 다른 2차원 배열을 가리키고 있다는 것을 알 수 있는데, 왜 .clone()은 원하는 대로 작동하지 않을 것일까?

```java
System.out.println(arr3);
System.out.println(arr4);
```

그것은 Java의 배열이 계층 구조로 이루어져 있기 때문이다. 2차원 배열은 element를 직접 가지고 있지 않고, 1차원 배열의 배열로 이루어져 있다. 다음을 보면 이 사실을 알 수 있다.

```java
for(int[] row: arr3) {
    System.out.println(row);
}
```

```
result:
[I@5acf9800
[I@4617c264
```

이것을 바탕으로, 우리가 .clone()을 한 배열을 살펴보자.

```java
System.out.println(arr3[0]);
System.out.println(arr4[0]);
```

```
result:
[I@5acf9800
[I@5acf9800
```

2차원 배열들은 결국 같은 1차원 배열을 가리키고 있는 것이다! 2차원 배열에 .clone()을 사용하면 1차원 배열들을 복사한 게 되어 1차원 배열들은 그대로 같은 element를 가리키게 되는 것이다.

## 결론

위에서 알 수 있듯이, .clone() 메소드는 배열 내부에 있는 모든 값을 복사하지 않고, 그 배열이 저장하는 값만 복사를 해 준다. 1차원 배열에서는 잘 작동하지만, 2차원 배열이나 class등 reference type을 저장하는 배열부터는 deep copy를 하는 것처럼 보이는 shallow copy를 하게 된다.

따라서 Java에서 2차원 배열을 deep copy할 때는 귀찮지만 하나하나 복사하자.
