# 1.基本介绍

Java 8 Lambda 表达式的代码实际操作例子，主要是把jdk8里面的lambda常用的例子摆一摆，忘记了看一看就知道怎么使用，方便回忆。

# 2.怎么使用

### 2.1.使用的循环体字段的代码，也就是操作对象，数据结构之类的，这里是简单的 list 数据。

```
private final List<BigDecimal> prices = Arrays.asList(
            new BigDecimal("10"), new BigDecimal("30"), new BigDecimal("17"),
            new BigDecimal("20"), new BigDecimal("15"), new BigDecimal("18"),
            new BigDecimal("45"), new BigDecimal("12"));

    private final List<String> friends = Arrays.asList("Brian", "Nate", "Neal", "Raju", "Sara", "Scott");
    private final List<String> editors = Arrays.asList("Brian", "Jackie", "John", "Mike");
    private final List<String> comrades = Arrays.asList("Kate", "Ken", "Nick", "Paula", "Zach");
    private final List<Person> people = Arrays.asList(
            new Person(20, "John"),
            new Person(21, "Sara"),
            new Person(21, "Jane"),
            new Person(35, "Greg")
    );
```

### 2.2.初识，简单对比lambda和常规操作的不同

```
   /**
     * 假设超过20块的话要打九折
     */
    private void test11() {
        BigDecimal totalOfDiscountedPrices = BigDecimal.ZERO;
        for (BigDecimal price : prices) {
            if (price.compareTo(BigDecimal.valueOf(20)) > 0) {
                totalOfDiscountedPrices = totalOfDiscountedPrices.add(price.multiply(BigDecimal.valueOf(0.9)));
            }
        }
        System.out.println("Total of discounted prices: " + totalOfDiscountedPrices);
    }

    private void test12() {
        final BigDecimal totalOfDiscountedPrices =
                prices.stream()
                        .filter(price -> price.compareTo(BigDecimal.valueOf(20)) > 0)
                        .map(price -> price.multiply(BigDecimal.valueOf(0.9)))
                        .reduce(BigDecimal.ZERO, BigDecimal::add);
        System.out.println("Total of discounted prices: " + totalOfDiscountedPrices);
    }
```

运行结果：

![](/static/image/20200422141523610.png)

### 2.3.集合的使用

```
   /**
     * 集合的使用
     */
    @Test
    public void test2() {
        friends.forEach(new Consumer<String>() {
            @Override
            public void accept(final String name) {
                System.out.println(name);
            }
        });

        friends.forEach((final String name) -> System.out.println(name));

        friends.forEach((name) -> System.out.println(name));

        friends.forEach(name -> System.out.println(name));

        friends.forEach(System.out::println);
    }
```

运行结果：  
![](/static/image/2020042214154563.png)  
都是从传统的写法上，慢慢的过度到Lambda的写法。

### 2.4.列表的转化

```
 private void test31() {
        final List<String> uppercaseNames = new ArrayList<String>();
        for (String name : friends) {
            uppercaseNames.add(name.toUpperCase());
        }
        System.out.println(uppercaseNames);
    }

    private void test32() {
        final List<String> uppercaseNames = new ArrayList<String>();
        friends.forEach(name -> uppercaseNames.add(name.toUpperCase()));
        System.out.println(uppercaseNames);
    }

    /**
     * Steam的map方法可以用来将输入序列转化成一个输出的序列
     * map方法把lambda表达式的运行结果收齐起来，返回一个结果集
     */
    private void test33() {
        friends.stream()
                .map(name -> name.toUpperCase())
                .forEach(name -> System.out.print(name + " "));
        System.out.println();
    }

    /**
     *
     */
    private void test34() {
        friends.stream()
                .map(name -> name.length())
                .forEach(count -> System.out.print(count + " "));
    }

    /**
     *
     */
    private void test35() {
        friends.stream()
                .map(String::length)
                .forEach(count -> System.out.print(count + " "));
    }
```

运行结果：

![img](/static/image/20200422141625541.png)  
都是从传统的写法上，慢慢的过度到Lambda的写法。

### 2.5.在集合中查找元素

```
 private void test41() {
        final List<String> startsWith = new ArrayList<String>();
        for (String name : friends) {
            if (name.startsWith("N")) {
                startsWith.add(name);
            }
        }
        System.out.println(startsWith);
    }

    /**
     * filter方法接收一个返回布尔值的lambda表达式。
     * 如果表达式结果为true，运行上下文中的那个元素就会被添加到结果集中;
     * 如果不是，就跳过它。最终返回的是一个Steam，它里面只包含那些表达式返回true的元素。
     * 最后我们用一个collect方法把这个集合转化成一个列表
     */
    private void test42() {
        final List<String> startsWith =
                friends.stream()
                        .filter(name -> name.startsWith("N"))
                        .collect(Collectors.toList());
        System.out.println(startsWith);
    }
```

运行结果：

![](/static/image/2020042214164436.png)

### 2.6.lambda表达式的重用

```
/**
     * lambda表达式带来的冗余
     */
    private void test51() {
        final long count1 = friends.stream().filter(name -> name.startsWith("N")).count();
        final long count2 = editors.stream().filter(name -> name.startsWith("N")).count();
        final long count3 = comrades.stream().filter(name -> name.startsWith("N")).count();
        System.out.println(count1 + " " + count2 + " " + count3);
    }

    /**
     * 重用
     */
    private void test52() {
        final Predicate<String> startsWith = name -> name.startsWith("N");
        final long count1 = friends.stream().filter(startsWith).count();
        final long count2 = editors.stream().filter(startsWith).count();
        final long count3 = comrades.stream().filter(startsWith).count();
        System.out.println(count1 + " " + count2 + " " + count3);
    }
```

运行结果：  
![](/static/image/20200422141726414.png)

### 2.7.闭包 使用词法作用域和闭包

```
  /**
     * 第一个predicate判断名字是否是以N开头的，第二个是判断是否以B开头的。
     * 我们把这两个实例分别传递给两次filter方法调用。
     * 这样看起来很合理，但是两个predicate产生了冗余，它们只是那个检查的字母不同而已。
     */
    private void test61() {
        final Predicate<String> startsWith1 = name -> name.startsWith("N");
        final Predicate<String> startsWith2 = name -> name.startsWith("B");
        final long count1 = friends.stream().filter(startsWith1).count();
        final long count2 = friends.stream().filter(startsWith2).count();
        System.out.println(count1 + " " + count2);
    }

    /**
     * filter可不是什么函数都接受的。它只接受只有一个参数的函数，那个参数对应的就是集合中的元素，返回一个boolean值，
     * 它希望传进来的是一个Predicate。
     */
    private void test62() {
        final long count1 = friends.stream().filter(checkIfStartsWith("N")).count();
        final long count2 = friends.stream().filter(checkIfStartsWith("B")).count();
        System.out.println(count1 + " " + count2);
    }

    public static Predicate<String> checkIfStartsWith(final String letter) {
        return name -> name.startsWith(letter);
    }
```

运行结果：

![](/static/image/20200422141746999.png)

### 2.8.Optional

```
 /**
     * Optional
     */
    @Test
    public void test7() {
        pickName1(friends, "N");
        pickName2(friends, "N");
    }

    public static void pickName1(List<String> names, String startingLetter) {
        String foundName = null;
        for (String name : names) {
            if (name.startsWith(startingLetter)) {
                foundName = name;
                break;
            }
        }
        System.out.println(String.format("A name starting with %s: %s", startingLetter, foundName));
    }

    public static void pickName2(List<String> names, String startingLetter) {
        final Optional<String> foundName = names.stream()
                .filter(name -> name.startsWith(startingLetter))
                .findFirst();
        System.out.println(String.format("A name starting with %s: %s", startingLetter, foundName.orElse("No name found")));
        foundName.ifPresent(System.out::println);
    }
```

运行结果：  
![](/static/image/20200422141813398.png)

### 2.9.MapReduce map（映射）和reduce\(归约，化简）是数学上两个很基础的概念

```
private void test81() {
        System.out.println("Total number of characters in all names: " + friends.stream()
                .mapToInt(name -> name.length())
                .sum());
    }

    private void test82() {
        final Optional<String> aLongName = friends.stream().reduce((name1, name2) -> name1.length() >= name2.length() ? name1 : name2);
        aLongName.ifPresent(name -> System.out.println(String.format("A longest name: %s", name)));
    }

    private void test83() {
        final String steveOrLonger = friends.stream().reduce("Steve", (name1, name2) -> name1.length() >= name2.length() ? name1 : name2);
        System.out.println(steveOrLonger);
    }

    private void test84() {
        for (String name : friends) {
            System.out.print(name + ", ");
        }
        System.out.println();

        for (int i = 0; i < friends.size() - 1; i++) {
            System.out.print(friends.get(i) + ", ");
        }

        if (friends.size() > 0) {
            System.out.println(friends.get(friends.size() - 1));
        }

        System.out.println(String.join(", ", friends));

        System.out.println(friends.stream().map(String::toUpperCase).collect(Collectors.joining(", ")));
    }
```

运行结果：

![img](/static/image/20200422141838855.png)

### 2.10.字符串及方法引用

```
/**
     * 字符串及方法引用
     */
    @Test
    public void test9() {
        final String str = "w00t";
        str.chars().forEach(ch -> System.out.println(ch));

        str.chars().forEach(System.out::println);

        str.chars()
                .mapToObj(ch -> Character.valueOf((char) ch))
                .forEach(System.out::println);

        str.chars()
                .filter(ch -> Character.isDigit(ch))
                .forEach(ch -> printChar(ch));

    }

    private static void printChar(int aChar) {
        System.out.println((char) (aChar));
    }
```

运行结果：

![img](/static/image/20200422141856449.png)

### 2.11.Comparator

```
private void testA7(List<Person> people) {
        final Function<Person, Integer> byAge = person -> person.getAge();
        final Function<Person, String> byTheirName = person -> person.getName();
        printPeople("Sorted in ascending order by age and name: ",
                people.stream()
                        .sorted(comparing(byAge).thenComparing(byTheirName))
                        .collect(Collectors.toList()));
    }

    private void testA6(List<Person> people) {
        final Function<Person, String> byName = person -> person.getName();
        List<Person> list = people.stream()
                .sorted(comparing(byName))
                .collect(Collectors.toList());
        printPeople("Sorted in ascending order by name: ", list);
    }

    private void testA5(List<Person> people) {
        List<Person> list = people.stream()
                .sorted((person1, person2) ->
                        person1.getName().compareTo(person2.getName()))
                .collect(Collectors.toList());
        printPeople("Sorted in ascending order by name: ", list);
    }

    private void testA4(List<Person> people) {
        people.stream()
                .max(Person::ageDifference)
                .ifPresent(eldest -> System.out.println("Eldest: " + eldest));
    }

    private void testA3(List<Person> people) {
        people.stream()
                .min(Person::ageDifference)
                .ifPresent(youngest -> System.out.println("Youngest: " + youngest));
    }

    private void testA2(List<Person> people) {
        Comparator<Person> compareAscending = (person1, person2) -> person1.ageDifference(person2);
        Comparator<Person> compareDescending = compareAscending.reversed();
        List<Person> list = people.stream().sorted(compareAscending).collect(Collectors.toList());
        printPeople("Sorted in ascending order by age: ", list);

        list = people.stream().sorted(compareDescending).collect(Collectors.toList());
        printPeople("Sorted in descending order by age: ", list);
    }

    private void testA1(List<Person> people) {
        List<Person> ascendingAge = people.stream().sorted((person1, person2) -> person1.ageDifference(person2))
                .collect(Collectors.toList());
        printPeople("Sorted in ascending order by age: ", ascendingAge);

        List<Person> descendingAge = people.stream().sorted((person1, person2) -> person2.ageDifference(person1))
                .collect(Collectors.toList());
        printPeople("Sorted in descending order by age: ", descendingAge);
    }

    public static void printPeople(final String message, final List<Person> people) {
        System.out.println(message);
        people.forEach(System.out::println);
    }
```

运行结果：

```
Sorted in ascending order by age: 
Person{age=20, name='John'}
Person{age=21, name='Sara'}
Person{age=21, name='Jane'}
Person{age=35, name='Greg'}
Sorted in descending order by age: 
Person{age=35, name='Greg'}
Person{age=21, name='Sara'}
Person{age=21, name='Jane'}
Person{age=20, name='John'}
Sorted in ascending order by age: 
Person{age=20, name='John'}
Person{age=21, name='Sara'}
Person{age=21, name='Jane'}
Person{age=35, name='Greg'}
Sorted in descending order by age: 
Person{age=35, name='Greg'}
Person{age=21, name='Sara'}
Person{age=21, name='Jane'}
Person{age=20, name='John'}
Youngest: Person{age=20, name='John'}
Eldest: Person{age=35, name='Greg'}
Sorted in ascending order by name: 
Person{age=35, name='Greg'}
Person{age=21, name='Jane'}
Person{age=20, name='John'}
Person{age=21, name='Sara'}
Sorted in ascending order by name: 
Person{age=35, name='Greg'}
Person{age=21, name='Jane'}
Person{age=20, name='John'}
Person{age=21, name='Sara'}
Sorted in ascending order by age and name: 
Person{age=20, name='John'}
Person{age=21, name='Jane'}
Person{age=21, name='Sara'}
Person{age=35, name='Greg'}
```

### 2.12.收集器

```
   /**
     * 收集器
     */
    @Test
    public void testB() {
        testB1();
        testB2();
        testB3();
        testB4();
        testB5();
        testB6();
    }

    private void testB6() {
        Comparator<Person> byAge = Comparator.comparing(Person::getAge);
        Map<Character, Optional<Person>> oldestPersonOfEachLetter =
                people.stream()
                        .collect(Collectors.groupingBy(person -> person.getName().charAt(0),
                                Collectors.reducing(BinaryOperator.maxBy(byAge))));
        System.out.println("Oldest person of each letter:");
        System.out.println(oldestPersonOfEachLetter);
    }

    @Test
    public void testB5() {
        Map<Integer, List<String>> nameOfPeopleByAge =
                people.stream()
                        .collect(
                                Collectors.groupingBy(Person::getAge, Collectors.mapping(Person::getName, Collectors.toList())));

        System.out.println("People grouped by age: " + nameOfPeopleByAge);
    }

    /**
     * to Map<Integer, List<Person>>
     */
    @Test
    public void testB4() {
        Map<Integer, List<Person>> peopleByAge =
                people.stream()
                        .collect(Collectors.groupingBy(Person::getAge));
        System.out.println("Grouped by age: " + peopleByAge);
    }

    /**
     * to list
     */
    private void testB3() {
        List<Person> olderThan20 =
                people.stream()
                        .filter(person -> person.getAge() > 20)
                        .collect(Collectors.toList());
        System.out.println("People older than 20: " + olderThan20);
    }

    /**
     * to list
     */
    private void testB2() {
        List<Person> olderThan20 =
                people.stream()
                        .filter(person -> person.getAge() > 20)
                        .collect(ArrayList::new, ArrayList::add, ArrayList::addAll);
        System.out.println("People older than 20: " + olderThan20);
    }

    /**
     * to list
     */
    private void testB1() {
        List<Person> olderThan20 = new ArrayList<>();
        people.stream()
                .filter(person -> person.getAge() > 20)
                .forEach(person -> olderThan20.add(person));
        System.out.println("People older than 20: " + olderThan20);
    }
```

运行结果：

![](/static/image/20200422140914515.png)

