### 1.`stream()` 方法 

​	Java 8 中引入的 Stream API 的一部分。它用于将集合（Collection）转换为一个流（Stream），以便进行各种操作，如过滤、映射、排序等。

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// 将列表转换为流
Stream<Integer> stream = numbers.stream();

// 过滤出偶数
stream.filter(num -> num % 2 == 0)
      // 对元素进行加倍
      .map(num -> num * 2)
      // 计算总和
      .reduce(0, Integer::sum);

http://wwwcurl "https://api.zsxq.com/v2/hashtags/48844541281228/topics?count=20" ^   -H "authority: api.zsxq.com" ^   -H "accept: application/json, text/plain, */*" ^   -H "accept-language: zh-CN,zh;q=0.9" ^   -H "cache-control: no-cache" ^   -H "origin: https://wx.zsxq.com" ^   -H "pragma: no-cache" ^   -H "referer: https://wx.zsxq.com/" ^   --compressed
```

不用parallelStream()的原因

​	在使用parallelStream()时，有一个常见的问题是重复消费。当在并行流中使用一些会改变流源的操作时（比如sorted()、distinct()等），如果这些操作在多个线程上并行执行，那么就有可能导致重复消费。这是因为并行流需要对数据进行[分片](https://so.csdn.net/so/search?q=%E5%88%86%E7%89%87&spm=1001.2101.3001.7020)，不同线程处理的分片可能会包含相同的元素，从而导致相同的元素被处理多次。