---
updated: 2022-12-10
created: 2022-12-10
title: Stream流
categories: Java
tags:
  - Java
excerpt: ''
cover: https://img.tt98.com/d/file/tt98/2020032519004984/5e7b285f92300.jpg
---

#### Stream流的创建

##### Collection.stream()

~~~java
List<String> list = Arrays.asList("a", "b", "c");
// 创建一个顺序流
Stream<String> stream = list.stream();
// 创建一个并行流
Stream<String> parallelStream = list.parallelStream();
~~~

##### Arrays.stream(array)

~~~java
int[] array={1,3,5,6,8};
IntStream stream = Arrays.stream(array);
~~~

##### 使用Stream的静态方法

~~~java
Stream<Integer> stream = Stream.of(1, 2, 3, 4, 5, 6);

Stream<Integer> stream2 = Stream.iterate(0, (x) -> x + 3).limit(4);//从0开始
stream2.forEach(System.out::println);

Stream<Double> stream3 = Stream.generate(Math::random).limit(3);
stream3.forEach(System.out::println);
~~~

#### 遍历/匹配(foreach/find/match)

~~~java
public class StreamTest {
    
    @Test
	public void demo1{
        List<Integer> list = Arrays.asList(7, 6, 9, 3, 8, 2, 1);

        // 遍历输出符合条件的元素
        list.stream().filter(x -> x > 6).forEach(System.out::println);
        // 匹配第一个,串行里面findFirst和findAny一样
        Optional<Integer> findFirst = list.stream().filter(x -> x > 6).findFirst();
        // 匹配任意（适用于并行流）
        Optional<Integer> findAny = list.parallelStream().filter(x -> x > 6).findAny();
        // 是否包含符合特定条件的元素
        boolean anyMatch = list.stream().anyMatch(x -> x > 6);
        System.out.println("匹配第一个值：" + findFirst.get());
        System.out.println("匹配任意一个值：" + findAny.get());
        System.out.println("是否存在大于6的值：" + anyMatch);
    }
}

~~~

#### 筛选(filter)

> 按照一定的规则校验流中的元素，将符合条件的元素提取到新的流中的操作。

~~~java
public class StreamTest {
    @Test
    public void demo2(){
        List<Integer> list = Arrays.asList(7, 6, 9, 3, 8, 2, 1);
        list.stream().filter(i->i>7).forEach(System.out::println);
    }
}
~~~

##### 案例(**筛选员工中工资高于8000的人，并形成新的集合。** 形成新集合依赖`collect`（收集）)

~~~java
public class StreamTest {
	public static void main(String[] args) {
		List<Person> personList = new ArrayList<Person>();
		personList.add(new Person("Tom", 8900, 23, "male", "New York"));
		personList.add(new Person("Jack", 7000, 25, "male", "Washington"));
		personList.add(new Person("Lily", 7800, 21, "female", "Washington"));
		personList.add(new Person("Anni", 8200, 24, "female", "New York"));
		personList.add(new Person("Owen", 9500, 25, "male", "New York"));
		personList.add(new Person("Alisa", 7900, 26, "female", "New York"));

		List<String> fiterList = personList.stream().filter(x -> x.getSalary() > 8000).map(Person::getName)
				.collect(Collectors.toList());
		System.out.print("薪资高于8000美元的员工：" + fiterList);
	}
}
~~~

#### 聚合(max/min/count)

~~~java
public class StreamTest {
	public static void main(String[] args) {
        //Integer同理max((x,y)->y.length()-x.length())返回一个int值
        //List<String> list = Arrays.asList("adnm", "admmt", "pot", "xbangd", "weoujgsd");
        //Optional<String> max = list.stream().max((x,y)->y.length()-x.length());
        //System.out.println("最长的字符串：" + max.get());
        
        //Integer同理max(Comparator.comparingInt(Integer::intValue))里面是函数类型
		List<String> list = Arrays.asList("adnm", "admmt", "pot", "xbangd", "weoujgsd");
		Optional<String> max = list.stream().max(Comparator.comparing(String::length));
		System.out.println("最长的字符串：" + max.get());
	}
}
~~~

##### 案例(**获取员工薪资最高的人**)

~~~java
public class StreamTest {
	public static void main(String[] args) {
		List<Person> personList = new ArrayList<Person>();
		personList.add(new Person("Tom", 8900, 23, "male", "New York"));
		personList.add(new Person("Jack", 7000, 25, "male", "Washington"));
		personList.add(new Person("Lily", 7800, 21, "female", "Washington"));
		personList.add(new Person("Anni", 8200, 24, "female", "New York"));
		personList.add(new Person("Owen", 9500, 25, "male", "New York"));
		personList.add(new Person("Alisa", 7900, 26, "female", "New York"));

		Optional<Person> max = personList.stream().max(Comparator.comparingInt(Person::getSalary));
		System.out.println("员工薪资最大值：" + max.get().getSalary());
	}
}
~~~

##### 案例(计算集合中大于6的元素的个数)

~~~java
public class StreamTest {
	public static void main(String[] args) {
		List<Integer> list = Arrays.asList(7, 6, 4, 8, 2, 11, 9);

		long count = list.stream().filter(x -> x > 6).count();
		System.out.println("list中大于6的元素个数：" + count);
	}
}
~~~

#### 映射(map/flatMap)

映射，将一个流的元素按照一定的映射规则映射到另一个流中。分为map和flatMap

map：接收一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新的流

flatMap：接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连接成一个流。

~~~java
public class StreamTest {
	public static void main(String[] args) {
		String[] strArr = { "abcd", "bcdd", "defde", "fTr" };
		List<String> strList = Arrays.stream(strArr).map(String::toUpperCase).collect(Collectors.toList());

		List<Integer> intList = Arrays.asList(1, 3, 5, 7, 9, 11);
		List<Integer> intListNew = intList.stream().map(x -> x + 3).collect(Collectors.toList());

		System.out.println("每个元素大写：" + strList);
		System.out.println("每个元素+3：" + intListNew);
	}
}
~~~

##### 案例(将所有user年龄增加10)

~~~java
@Test
    public void demo4(){
        List<User> list=new ArrayList<>();
        list.add(new User(1,"aaa","aaa","aaa",18,"女","aaa"));
        list.add(new User(1,"bbb","bbb","bbb",17,"女","bbb"));
        List<User> collect = list.stream().map(user -> {
            user.setAge(user.getAge() + 10);
            return user;
        }).collect(Collectors.toList());
        collect.stream().forEach(System.out::println);
    }
~~~

##### 案例(将两个字符数组合并成一个新的字符数组)

~~~java
public void demo5(){
        List<String> list = Arrays.asList("a,b,c,d", "e,f,g,h");
//        List<Stream<String>> collect = list.stream().map(i -> {
//            String[] split = i.split(",");
//            return Arrays.stream(split);
//        }).collect(Collectors.toList());
        List<String> collect1 = list.stream().flatMap(i -> {
            String[] split = i.split(",");
            return Arrays.stream(split);
        }).collect(Collectors.toList());
        System.out.println(collect1);
    }
~~~

#### 收集(collect)

> collect主要依赖java.util.stream.Collectors类内置的静态方法。

##### 归集(toList/toSet/toMap)

因为流不存储数据，那么在流中的数据完成处理后，需要将流中的数据重新归集到新的集合里。`toList`、`toSet`和`toMap`比较常用，另外还有`toCollection`、`toConcurrentMap`等复杂一些的用法。

~~~java
public class StreamTest {
	public static void main(String[] args) {
		List<Integer> list = Arrays.asList(1, 6, 3, 4, 6, 7, 9, 6, 20);
		List<Integer> listNew = list.stream().filter(x -> x % 2 == 0).collect(Collectors.toList());
		Set<Integer> set = list.stream().filter(x -> x % 2 == 0).collect(Collectors.toSet());

		List<Person> personList = new ArrayList<Person>();
		personList.add(new Person("Tom", 8900, 23, "male", "New York"));
		personList.add(new Person("Jack", 7000, 25, "male", "Washington"));
		personList.add(new Person("Lily", 7800, 21, "female", "Washington"));
		personList.add(new Person("Anni", 8200, 24, "female", "New York"));
		
		Map<?, Person> map = personList.stream().filter(p -> p.getSalary() > 8000)
				.collect(Collectors.toMap(Person::getName, p -> p));
		System.out.println("toList:" + listNew);
		System.out.println("toSet:" + set);
		System.out.println("toMap:" + map);
	}
}
~~~

##### 接合(joining)

~~~java
public class StreamTest {
	public static void main(String[] args) {
		List<Person> personList = new ArrayList<Person>();
		personList.add(new Person("Tom", 8900, 23, "male", "New York"));
		personList.add(new Person("Jack", 7000, 25, "male", "Washington"));
		personList.add(new Person("Lily", 7800, 21, "female", "Washington"));

		String names = personList.stream().map(p -> p.getName()).collect(Collectors.joining(","));
		System.out.println("所有员工的姓名：" + names);
		List<String> list = Arrays.asList("A", "B", "C");
		String string = list.stream().collect(Collectors.joining("-"));
		System.out.println("拼接后的字符串：" + string);
	}
}
~~~

#### 排序(sorted)

~~~java
public class StreamTest {
	public static void main(String[] args) {
		List<Person> personList = new ArrayList<Person>();

		personList.add(new Person("Sherry", 9000, 24, "female", "New York"));
		personList.add(new Person("Tom", 8900, 22, "male", "Washington"));
		personList.add(new Person("Jack", 9000, 25, "male", "Washington"));
		personList.add(new Person("Lily", 8800, 26, "male", "New York"));
		personList.add(new Person("Alisa", 9000, 26, "female", "New York"));

		// 按工资升序排序（自然排序）
		List<String> newList = personList.stream().sorted(Comparator.comparing(Person::getSalary)).map(Person::getName)
				.collect(Collectors.toList());
		// 按工资倒序排序
		List<String> newList2 = personList.stream().sorted(Comparator.comparing(Person::getSalary).reversed())
				.map(Person::getName).collect(Collectors.toList());
		// 先按工资再按年龄升序排序
		List<String> newList3 = personList.stream()
				.sorted(Comparator.comparing(Person::getSalary).thenComparing(Person::getAge)).map(Person::getName)
				.collect(Collectors.toList());
		// 先按工资再按年龄自定义排序（降序）
		List<String> newList4 = personList.stream().sorted((p1, p2) -> {
			if (p1.getSalary() == p2.getSalary()) {
				return p2.getAge() - p1.getAge();
			} else {
				return p2.getSalary() - p1.getSalary();
			}
		}).map(Person::getName).collect(Collectors.toList());

		System.out.println("按工资升序排序：" + newList);
		System.out.println("按工资降序排序：" + newList2);
		System.out.println("先按工资再按年龄升序排序：" + newList3);
		System.out.println("先按工资再按年龄自定义降序排序：" + newList4);
	}
}
~~~

#### 提取/组合(distinct,skip,limit)

~~~java
public class StreamTest {
	public static void main(String[] args) {
		String[] arr1 = { "a", "b", "c", "d" };
		String[] arr2 = { "d", "e", "f", "g" };

		Stream<String> stream1 = Stream.of(arr1);
		Stream<String> stream2 = Stream.of(arr2);
		// concat:合并两个流 distinct：去重
		List<String> newList = Stream.concat(stream1, stream2).distinct().collect(Collectors.toList());
		// limit：限制从流中获得前n个数据
		List<Integer> collect = Stream.iterate(1, x -> x + 2).limit(10).collect(Collectors.toList());
		// skip：跳过前n个数据
		List<Integer> collect2 = Stream.iterate(1, x -> x + 2).skip(1).limit(5).collect(Collectors.toList());

		System.out.println("流合并：" + newList);
		System.out.println("limit：" + collect);
		System.out.println("skip：" + collect2);
	}
}
~~~

