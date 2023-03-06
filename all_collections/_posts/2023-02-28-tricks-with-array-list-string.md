---
layout: post
title: Tổng Hợp một số Tricks Dùng cho xử lí mảng, list, và chuỗi trong Java
date: 2023-03-01
categories: ["java", "dev", "trick", "vi"]
---

Trước đây, chắc hẳn chúng ta đã nghe được những anh chị dev đi trước nói vể ngôn ngữ Java như là một thứ gì đó _rườm rà_, _dài dòng_, _cũ kĩ_, và _không linh họat_, etc. Quả thật, lúc mới học, khi nhìn vào một bộ [Java keywords](https://www.wikiwand.com/en/List_of_Java_keywords) chắc hẳn ai đó cũng đã choáng, trong đó có mình 😊. Lấy một vài ví dụ cụ thể hơn về những nhược điểm trên, để có thể khai báo hàm Main và in được dòng `Hello world` ra console, chúng ta cần phải khai báo những keywords như `public`, `static`, `void`, `main`, `(String[] args)`, `System`, `out`, và `println()`... Khá là dài dòng.

```java
class Main {
  public static void main(String[] args) {
    System.out.println("Hello world");
  }
}
```

Hay để convert một mảng thành list tương ứng, chúng ta đã từng phải viết một cái hàm dài như này 🥲.

```java
class Main {
    // hàm biến đổi mảng thành list dưới dạng generic
    public static <T> List<T> convertToList(T[] arr) {
        // tạo một list rỗng
        List<T> list = new ArrayList<>();

        // lưu tất cả phần từ có trong mảng vào list
        for (T i: arr)
            list.add(i);

        return list;
    }

    // driver
    public static void main(String[] args) {
        // cho
        String[] arr_str = { "A", "B", "C", "D" };

        // thực thi biến đổi
        List<String> list_str = convertToList(arr_str);

        // kiểm tra
        System.out.println(list_str);
    }
}
```

Tuy nhiên, từ phiên bản Java 8 trở đi tới giờ, Java đã được nâng cấp đến phiên bản thứ 17 (hình như sắp ra 19 rồi thì phải). Mỗi một bản nâng cấp thì ngôn ngữ cũng được cải thiện hơn về mặt cú pháp như tường minh hơn, ngắn gọn hơn, và nhũng thư viện tiện ích như [Stream API](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html) được ra đời, hay [Collections framework](https://docs.oracle.com/javase/8/docs/technotes/guides/collections/overview.html) cũng được nâng cấp để giúp công việc của Java developer thêm phần tiện nghi, và dễ dàng hơn.

Trong bài viết lần này, chúng ta sẽ sử dụng sự nâng cấp, cải tiến của những bản Java gần đây và thao tác một vài tricks hữu ích như xử lí mảng, list, và chuỗi để phục vụ công việc thường ngày nhé.

## Indexing

- [Biến đổi list thành mảng]()
- [Biến đổi mảng thày list]()
- [Gán giá trị mặc định cho mảng có size nhất định]()
- [Gán giá trị mặc định cho list có size nhất định]()
- [So sánh hai mảng]()
- [Sắp xếp mảng theo thứ tự tăng dần]()
- [Sắp xếp mảng theo thứ tự giảm dần]()
- [Sắp xếp list theo thứ tự tăng dần]()
- [Sắp xếp list theo thứ tự giảm dần]()
- [Tìm giá trị lớn nhất/nhỏ nhất trong list]()
- [Tìm giá trị lớn nhất/nhỏ nhất trong mảng]()
- [Tìm giá trị lớn nhất/nhỏ nhất trong custom objects]()
- [Gộp tất cả những phần từ chẵn/lẻ có trong mảng thành một list]()
- [Tách các phần tử trong mảng thành một list dựa trên một dãy pham vi [fromIndex,..,toIndex]]()
- [Tìm tổng của các phần từ có trong list trong dãy phạm vi [fromIndex,..,toIndex]]()
- [Tìm tổng của tất cả các phần từ có trong list]()
- [Biến đổi mảng hai chiều thành list hai chiều]()
- [Sắp xếp các chuỗi trong một list]()

## Getting Started

### Biến đổi list thành mảng

```java
List<String> list = Arrays.asList("Java", "is", "verbose");

String[] array = list.toArray(new String[list.size()]);
// output: ["Java", "is", "verbose"]
```

### Biến đổi mảng thành list

> Lưu ý: `Arrays.asList()` là bất biến (immutable), tức là sau khi được gán giá trị, nó sẽ không thể thêm, xóa, xửa các phần tử bên trong. Còn `collect(Collectors.toList())` là không bất biến (mutable), có thể thêm, xóa, và sửa.

```java
String[] arr_str = { "A", "B", "C", "D" };

// cách 1
List<String> list_str = Arrays.asList(arr_str);

// cách 2
List<String> list_str = Arrays.stream(arr_str)
                              .collect(Collectors.toList());

// output: ["A", "B", "C", "D"]
```

### Gán giá trị mặc định cho mảng có size nhất định

```java
// size là `5`
int[] arrayOfZeroes = new int[5];

// giá trị mặc định là `0`
Arrays.fill(arrayOfZeroes, 0);
// output: [0, 0, 0, 0, 0]
```

### Gán giá trị mặc định cho list có size nhất định

```java
// size là  `5`, giá trị mặc định là `0`
List<Integer> listOfZero = Collections.nCopies(5, 0)
                                      .stream()
                                      .map(e -> e)
                                      .toList();
// output: [0, 0, 0, 0, 0]
```

### So sánh hai mảng

```java
int[] arr = {1, 2, 3};
int[] arr1 = {1, 2, 3};

// so sánh
int result = Arrays.compare(arr, arr1);

System.out.println(result == 0 ? "equal" : "not equal");
// output: equal
```

### Sắp xếp mảng theo thứ tự tăng dần

```java
int[] arr = {2,1,3};

Arrays.sort(arr);
// output: [1,2,3]
```

### Sắp xếp mảng theo thứ tự giảm dần

```java
int[] arr = {2,1,3};

int[] result = Arrays.stream(arr)
                     .boxed()
                     .sorted(Collections.reverseOrder())
                     .mapToInt(Integer::intValue)
                     .toArray();
// output: [1,2,3]
```

### Sắp xếp list theo thứ tự tăng dần

```java
var list = Arrays.asList(2, 1, 3);

Collections.sort(list);
// output: [1, 2, 3]
```

### Sắp xếp list theo thứ tự giảm dần

```java
var list = Arrays.asList(2, 1, 3);

// sort nó trước
Collections.sort(list);

// sau đó đảo mảng sẽ thành giảm dần
Collections.sort(list, Collections.reverseOrder());
// output: [3, 2, 1]
```

### Tìm giá trị lớn nhất/nhỏ nhất trong list

```java
var list = Arrays.asList(1, 2, 3);

// get max
int max = Collections.max(list);
// output: 3

// get min
int min = Collections.min(list);
// output: 1
```

### Tìm giá trị lớn nhất/nhỏ nhất trong mảng

```java
int[] arr = {1, 2, 3};

// tìm min
int min = Arrays.stream(arr)
                .min()
                .getAsInt();
// output: 1

// tìm max
int max = Arrays.stream(arr)
                .max()
                .getAsInt();
// output: 3
```

### Tìm giá trị lớn nhất/nhỏ nhất trong custom objects

```java
public class Employee {
  String name;
  int age;

  // constructors, getters, and setters
}

public class Search {

  // tìm employee già nhất dựa trên tuổi
  public static Employee oldest(Employee[] employees) {
     Employee result = Arrays.stream(employees)
                           .max(Comparator.comparing(Employee::getAge))
                           .orElseThrow(NoSuchElementException::new);

      return result;
  }

  // tìm employee trẻ nhất dựa trên tuổi
  public static Employee youngest(Employee[] employees) {
      Employee result = Arrays.stream(employees)
                           .min(Comparator.comparing(Employee::getAge))
                           .orElseThrow(NoSuchElementException::new);

      return result;
  }
}
```

### Gộp tất cả những phần từ chẵn/lẻ có trong mảng thành một list

```java
var list = List.of(1, 2, 3, 4);

// lấy ra tất cả các phần tử chẵn
List<Integer> evenList = list.stream()
                             .filter(x -> { return x % 2 == 0; })
                             .toList();
// output: [2, 4]

// lấy ra tất cả các phần tử lẻ
List<Integer> oddList = listOf.stream()
                              .filter(x -> x % 2 != 0)
                              .collect(Collectors.toList());
// output: [1, 3]
```

### Tách các phần tử trong mảng thành một list dựa trên một dãy phạm vi [fromIndex,..,toIndex]

```java
int[] arr = {1, 2, 3, 4, 5};

// dãy phạm vi [1,..,3]
// lưu ý rằng index `3` là không bao gồm (exclusive)
int from = 1, to = 3;

var result = IntStream.range(from, to + 1)
                      .mapToObj(i -> arr[i])
                      .collect(Collectors.toList());
// output: [2, 3, 4]
```

### Tìm tổng của các phần từ có trong list trong dãy phạm vi [fromIndex,..,toIndex]

```java
int[] arr = {1, 2, 3, 4, 5};

// dãy pham vi [1,..,3]
int from = 1, to = 3;

var result = IntStream.range(from, to + 1)
                        .mapToObj(k -> nums[k])
                        .collect(Collectors.toList())
                        .stream()
                        .mapToInt(e -> e)
                        .sum();
// output: 9
```

### Tìm tổng của tất cả các phần từ có trong list

```java
var list = List.of(1, 2, 3);

int result = list.stream()
              .mapToInt(e -> e)
              .sum();
// output: 6
```

### Tìm tổng của các phần tử trong phạm vi

```java
// dãy phạm vi [1,..5]
int from = 1, to = 5;

int result = IntStream.range(from, to + 1)
                   .sum();
// output: 15
```

### Biến đổi mảng hai chiều thành list hai chiều

```java
// list hai chiều
List<List<Integer>> twoD_list = Arrays.asList(
      Arrays.asList(1, 1, 1),
      Arrays.asList(2, 2, 2),
      Arrays.asList(3, 3, 3)
);

// mảng hai chiều
int[][] twoD_arr = twoD_list.stream()
                  .map(l -> l.stream()
                             .mapToInt(Integer::intValue)
                             .toArray())
                  .toArray(int[][]::new);

// output: [[1, 1, 1], [2, 2, 2], [3, 3, 3]]
```

### Sắp xếp các chuỗi trong một list

```java
var words = Arrays.asList("bca", "acb", "cba");

// sắp xếp tất cả các chuỗi có trong list
List<String> result = words.stream()
                           .map(e -> Stream.of(e.split(""))
                                           .sorted()
                                           .collect(Collectors.joining()))
                           .collect(Collectors.toList());
// output: ["abc", "abc", "abc"]
```

## Conclusion

Như bạn có thể thấy, thông qua những cải tiến và sự ra đời của Stream API thì các công việc trước kia giờ đã trở lên ngắn gọn hơn khá nhiều. Hi vọng các bạn tìm được gì đó hữu ích trong bài viết này. Chúc các bạn học tập thành công 😊.
