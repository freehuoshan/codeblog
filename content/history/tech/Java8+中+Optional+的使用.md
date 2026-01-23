---
title: Java8 中 Optional 的使用
id: 10
date: 2023-04-10 18:19:13
auther: FREEDOM
cover: https://codeblog.net/upload/2021/04/images-c1780dbfe6244362acdd60ccafba85a8.png
excerpt: 概覽該教程會展示　Java8 引入的 Optional 類如何使用。該類的目的在於爲“可選值”(值存在或者不存在)提供類級別的解決方案以替代 null 引用(也稱空引用)。　要想深入理解爲什麼我們需要關心　Optional 類，請查看　這篇 oracle 官方文章創建 Optional 對象創建 O
permalink: /archives/java8zhong-optionalde-shi-yong
categories:
 - code
tags: 
 - java8
---

1. ## 概覽


  該教程會展示　Java8 引入的 Optional 類如何使用。該類的目的在於爲“可選值”(值存在或者不存在)提供類級別的解決方案以替代 null 引用(也稱空引用)。

　要想深入理解爲什麼我們需要關心　Optional 類，請查看　這篇 oracle 官方文章



2. ## 創建 Optional 對象


  創建 Optional 對象有多種方式。創建一個空 Optional 對象，我們只需要使用它的一個靜態方法 empty: 

```java
@Test
public void whenCreatesEmptyOptional_thenCorrect() {
    Optional<String> empty = Optional.empty();
    assertFalse(empty.isPresent());
}
```

  注意: 上面我們使用了方法 isPresent() 檢查那個 Optional 對象裏是否包含一個值。只有當我們使用一個非空值去創建 Optional 它纔會包含一個值。我們會在下邊進一步介紹 isPresent 方法。

  我們也可以使用靜態方法 of 創建 Optional 對象。

```java
@Test
public void givenNonNull_whenCreatesNonNullable_thenCorrect() {
    String name = "baeldung";
    Optional<String> opt = Optional.of(name);
    assertTrue(opt.isPresent());
}
```

  但是，傳給方法 of() 的參數不能爲 null。否則我們會收到一個 NullPointerException 異常。

```java
@Test(expected = NullPointerException.class)
public void givenNull_whenThrowsErrorOnCreate_thenCorrect() {
    String name = null;
    Optional.of(name);
}
```

  但是如果我們希望它可以包含 null 值，我們可以使用方法 offNullable()。

```java
@Test
public void givenNonNull_whenCreatesNullable_thenCorrect() {
    String name = "baeldung";
    Optional<String> opt = Optional.ofNullable(name);
    assertTrue(opt.isPresent());
}
```

  在這種情況下，如果我們傳入一個 null 值，它不會拋出任何異常但是會返回一個空的 Optional 對象

```java
@Test
public void givenNull_whenCreatesNullable_thenCorrect() {
    String name = null;
    Optional<String> opt = Optional.ofNullable(name);
    assertFalse(opt.isPresent());
}
```


3. ## 二级标题檢查值是否存在: isPresent() 和 isEmpty()


  當有一個從方法返回的或者自己創建的 Optional 對象，我們可以使用方法 isPresent()查看它是否包含一個值:

```java
@Test
public void givenOptional_whenIsPresentWorks_thenCorrect() {
    Optional<String> opt = Optional.of("Baeldung");
    assertTrue(opt.isPresent());
 
    opt = Optional.ofNullable(null);
    assertFalse(opt.isPresent());
}
```

  如果包含的值不是 null,,它會返回 true

  同時，如果使用 Java 11,　與之相反我們可以使用　isEmpty 方法

```java
@Test
public　void　givenAnEmptyOptional_thenIsEmptyBehavesAsExpected() {
    Optional<String> opt = Optional.of("Baeldung");
    assertFalse(opt.isEmpty());
 
    opt = Optional.ofNullable(null);
    assertTrue(opt.isEmpty());
}
```



4. ## 二级标题條件操作和 ifPresent()


  ifPresent()方法允許我們對包裹的非 null 值執行一些操作。在引入 Optional 之前，我們是這樣做的:

```java
String name = "..."
if(name != null) {
    System.out.println(name.length());
}
```

  這段代碼在對 name 執行操作之前檢查 name 是否爲空。這種解決方案不經冗餘而且容易出錯。

  它是爲了確保後邊打印變量可以正確運行，我們以後不應該再使用這種解決方案，徹底忘記執行 null 檢查。 

  但是在這段代碼中如果不執行 null 檢查，如果 name 變量爲空會發生 NullPointerException 異常。

  如果程序運行失敗是由輸入引起的，這通常是糟糕的編程習慣造成的。

  作爲一種更好的編程習慣，Optional 強制我們顯式處理可能爲空的值。現在讓我們看看在 java8 中如何重構上邊的代碼。

  以典型的函數編程風格，我們對一個的確存在的對象執行操作。

```java
@Test
public void givenOptional_whenIfPresentWorks_thenCorrect() {
    Optional<String> opt = Optional.of("baeldung");
    opt.ifPresent(name -> System.out.println(name.length()));
}
```

  這個例子中我們使用兩行代碼實現了上邊四行代碼同樣的功能。一行代碼是講一個對象包裹到一個 Optional 對象中，第二行代碼除了執行打印代碼外還隱含有驗證。

  

5. ## 默認值和 orElse()


  方法 orElse() 用於獲取 Optional 實例中包裹的值。它需要一個參數作爲默認值。如果包裹的值存在，方法 orElse()返回包裹的值，否則返回參數:

```java
@Test
public void whenOrElseWorks_thenCorrect() {
    String nullName = null;
    String name = Optional.ofNullable(nullName).orElse("john");
    assertEquals("john", name);
}
```



6. ## 默認值和 orElseGet()


  方法 orElseGet() 類似於 orElse()。但是如果 Optional 包裹的值不存在不是提供一個默認值返回，取而代之的是一個提供一個函數式接口會被調用並返回調用的結果。

```java
@Test
public void whenOrElseGetWorks_thenCorrect() {
    String nullName = null;
    String name = Optional.ofNullable(nullName).orElseGet(() -> "john");
    assertEquals("john", name);
}
```



7. ## orElse 和 orElseGet 的不同


  很多程序員是新接觸 Optional 或者 Java8,對於 orElse() 和 orElseGet() 不同點不是非常清楚。實際上這兩個方法給人的印象是功能互相重疊。

  然而，這兩個方法有一些非常細微但是非常重要的區別，如果不是非常清楚的話會嚴重影響代碼的性能。

  讓我們在測試類中創建一個名爲 getMyDefault() 的方法，沒有參數，返回一個默認值。

```java
public String getMyDefault() {
    System.out.println("Getting Default Value");
    return "Default Value";
}
```

  讓我們看兩個測試並觀察他們的行爲以確定方法 orElse() 和 orElseGet() 的重疊和不同之處

```java
@Test
public void whenOrElseGetAndOrElseOverlap_thenCorrect() {
    String text = null;
 
    String defaultText = Optional.ofNullable(text).orElseGet(this::getMyDefault);
    assertEquals("Default Value", defaultText);
 
    defaultText = Optional.ofNullable(text).orElse(getMyDefault());
    assertEquals("Default Value", defaultText);
}
```

  在上邊的例子中，將一個 null 包裹到一個 Optional 對象中然後我們試圖獲取包裹的值。兩種方式輸入如下:

```java
Getting default value...
Getting default value...
```

  兩種方式都調用了 getMyDefault() 方法。當包裹的值不存在時， orElse() 和 orElseGet() 運行結果完全一樣。

  現在讓我們運行另一個測試，包裹的值非 null,理想情況下默認值不應該被創建。

```java
@Test
public void whenOrElseGetAndOrElseDiffer_thenCorrect() {
    String text = "Text present";
 
    System.out.println("Using orElseGet:");
    String defaultText  = Optional.ofNullable(text).orElseGet(this::getMyDefault);
    assertEquals("Text present", defaultText);
 
    System.out.println("Using orElse:");
    defaultText = Optional.ofNullable(text).orElse(getMyDefault());
    assertEquals("Text present", defaultText);
}
```

  上邊的代碼除了包裹的不是 null 外其他代碼保持一樣。現在讓我們再次看下運行結果:

```java
Using orElseGet:
Using orElse:
Getting default value...
```

  注意到當使用 orElseGet() 獲取包裹的值時，由於包裹的值是存在的，方法 getMyDefault() 甚至沒有被調用。

  然而當使用 orElse() ，無論包裹的值是否存在，默認值總是會被創建。所以這種情況下，我們創建了一個從未使用的多餘對象。

  在上邊這個簡單的例子中，創建一個默認對象成本不高，但是如果創建默認值的方法例如這裏的 getMyDefault() 

需要調用 web 服務或者查詢數據庫，那麼成本就高多了。



8. ## 異常和 orElseThrow()


  方法 orElseThrow() 類似於 orElse() 和 orElseGet() 爲當值缺失時提供了一種新的解決方案。當值不存在時不是返回值而是拋出一個異常:

```java
@Test(expected = IllegalArgumentException.class)
public void whenOrElseThrowWorks_thenCorrect() {
    String nullName = null;
    String name = Optional.ofNullable(nullName).orElseThrow(
      IllegalArgumentException::new);
}
```

  這裏用到了 Java8 中的方法引用，向其傳入一個異常構造器



9. ## 返回值和 get()


  獲取包裝值的最終方法是 get():

```java
@Test
public void givenOptional_whenGetsValue_thenCorrect() {
    Optional<String> opt = Optional.of("baeldung");
    String name = opt.get();
    assertEquals("baeldung", name);
}
```

  然而，不像上邊三個方法，get() 只有當包裹的值非 null 纔會返回一個值，否則拋出一個 NoSuchElementException 異常

```java
@Test(expected = NoSuchElementException.class)
public void givenOptionalWithNull_whenGetThrowsException_thenCorrect() {
    Optional<String> opt = Optional.ofNullable(null);
    String name = opt.get();
}
```

  這是 get() 方法的主要缺陷，很明顯　Optional 應該幫助我們避免這樣的意外異常。因此這種方式違背 Optional 的初衷，很有可能將來會被棄用

  因此建議使用其它方式，其他方式讓我們能夠以顯式的方式處理值爲空的情況



10. ## 根據條件返回和 filter()


  我們可以使用　filter 方法對包裹的值進行測試並過濾。它需要傳入一個 Predicate(斷言-Java8提供了一個 Predicate 函數接口，用於判斷和測試) 參數，返回一個 Optional 對象,如果包裹得值通過了這個斷言的測試，那麼被測試的 Optional 被原樣返回。

  然而，如果該斷言返回爲 false,　它會返回一個空的 Optional 對象。

```java
@Test
public　void　whenOptionalFilterWorks_thenCorrect() {
    Integer year = 2016;
    Optional<Integer> yearOptional = Optional.of(year);
    boolean　is2016 = yearOptional.filter(y -> y == 2016).isPresent();
    assertTrue(is2016);
    boolean　is2017 = yearOptional.filter(y -> y == 2017).isPresent();
    assertFalse(is2017);
}
```

  filter 方法通常用來基於指定規則驗證包裹的值。例如，我們可以使用它驗證郵箱地址的格式或者密碼的安全性。

  讓我們看另一個有意思的例子。假設我們想要買一個路由器而且我們只關心價格問題。我們從某個網站接收推送並存儲到這些對象中:

```java
public　class　Router {
    private　Double price;
 
    public　Router(Double price) {
        this.price = price;
    }
    // standard getters and setters
}
```

  然後我們在以下代碼使用這些對象，其唯一目的是檢查該路由器的價格是否在我們的預算內  

  讓我們看下如果不使用 Optional:

```java
public boolean priceIsInRange1(Router router) {
    boolean isInRange = false;
 
    if(router != null && router.getPrice() != null && (router.getPrice() >= 10
        && router.getPrice() <= 15)) {
 
        isInRange = true;
    }
    return isInRange;
}
```

  請注意我們爲了實現這一目的寫了多少代碼，尤其是 if 條件語句。只有最後檢查價格範圍是和業務相關的，其他代碼只是爲了避免發生空指針異常

```java
@Test
public　void　whenFiltersWithoutOptional_thenCorrect() {
    assertTrue(priceIsInRange1(new　Modem(10.0)));
    assertFalse(priceIsInRange1(new　Modem(9.9)));
    assertFalse(priceIsInRange1(new　Modem(null)));
    assertFalse(priceIsInRange1(new　Modem(15.5)));
    assertFalse(priceIsInRange1(null));
}
```

  除此之外，很有可能某一天你忘記了 null 檢查，編譯時不會報任何錯誤。

  現在讓我們使用 Optional 的 filter 方法實現同樣的功能。

```java
public boolean priceIsInRange2(Modem modem2) {
     return Optional.ofNullable(modem2)
       .map(Modem::getPrice)
       .filter(p -> p >= 10)
       .filter(p -> p <= 15)
       .isPresent();
 }
```

  上邊調用了 map 方法是用於將一個值轉換爲另一個值,但是請注意，這個操作不會修改原始值。

  在我們這裏，我們從一個 Router 對象獲取其價格。在下面我們會詳細介紹 map() 方法。

  首先，如果向上面的方法傳入一個 null 對象，我們不希望出現任何錯誤或者異常。

  其次，方法中代碼邏輯就是方法名描述的那樣:價格範圍檢查。Optional 會處理其他事情。

```java
@Test
public void whenFiltersWithOptional_thenCorrect() {
    assertTrue(priceIsInRange2(new Modem(10.0)));
    assertFalse(priceIsInRange2(new Modem(9.9)));
    assertFalse(priceIsInRange2(new Modem(null)));
    assertFalse(priceIsInRange2(new Modem(15.5)));
    assertFalse(priceIsInRange2(null));
}
```

  之前的解決方案除了檢查價格範圍還必須做更多的檢查以防禦其固有的脆弱性。因此，我們可以使用 filter 方法替代不必要的 if 語句和排除不想要的值。



11. ## 轉換值和 map()


  在上面，我們知道了如何使用 filter 拒絕或者接受一個值。我們可以使用相似的語法使用 map() 方法轉換一個 Optional 值

```java
@Test
public void givenOptional_whenMapWorks_thenCorrect() {
    List<String> companyNames = Arrays.asList(
      "paypal", "oracle", "", "microsoft", "", "apple");
    Optional<List<String>> listOptional = Optional.of(companyNames);
 
    int size = listOptional
      .map(List::size)
      .orElse(0);
    assertEquals(6, size);
}
```

   在這個例子中，我們將一個字符串列表包裹在一個 Optional 對象中然後使用它的 map 方法對其包含的 list 執行操作。這裏我們執行的操作是獲取列表的大小。

   map 方法返回的計算結果包裹在 Optional 中，我們必須調用返回的 Optional 相應的方法獲取它的值。

   注意， 方法 filter 對其值執行簡單的檢查然後返回一個 boolean 值。而 map 方法是對該值執行計算然後將計算結果包裹在一個 Optional 對象中。

```java
@Test
public　void　givenOptional_whenMapWorks_thenCorrect2() {
    String　name = "baeldung";
    Optional<String> nameOptional = Optional.of(name);
 
    int　len = nameOptional
     .map(String::length)
     .orElse(0);
    assertEquals(8, len);
}
```

  我們可以將 map 和 filter 鏈接到一起執行更強大的操作。

  現在假設我們想要檢查用戶輸入密碼的正確性，我們可以先使用 map 清洗一下這個密碼，然後使用一個 filter 檢查其是否正確:

```java
@Test
public void givenOptional_whenMapWorksWithFilter_thenCorrect() {
    String password = " password ";
    Optional<String> passOpt = Optional.of(password);
    boolean correctPassword = passOpt.filter(
      pass -> pass.equals("password")).isPresent();
    assertFalse(correctPassword);
 
    correctPassword = passOpt
      .map(String::trim)
      .filter(pass -> pass.equals("password"))
      .isPresent();
    assertTrue(correctPassword);
}
```

  就像我們看到的，如果不首先清洗輸入，它會被過濾掉－>用戶輸入的有可能帶有前置空格和後置空格。所以我們在過濾掉不正確的之前先使用一個 map 將髒密碼轉換爲乾淨的



12. ## 轉換值和　flatMap()


  就像 map() 方法，我們還有一個　flatMap() 作爲轉換值的選擇。不同點在於 map 轉換的值未被包裹，而 flatMap 處理被包裹的值會在轉換之前解包。

  之前，我們做過將簡單的字符串或者 Integer 對象包裹到一個 Optional 實例中。但是通常我們是從一個複雜對象的訪問器獲取這些對象。

  爲了搞清楚這兩個方法的不同，我創建了一個 Person 對象，它包含 name,age,password 字段:

```java
public class Person {
    private String name;
    private int age;
    private String password;
 
    public Optional<String> getName() {
        return Optional.ofNullable(name);
    }
 
    public Optional<Integer> getAge() {
        return Optional.ofNullable(age);
    }
 
    public Optional<String> getPassword() {
        return Optional.ofNullable(password);
    }
 
    // normal constructors and setters
}
```

  我們通常會創建一個這樣的對象然後將其包裹到一個 Optional 對象中，就像我們之前包裹字符串那樣，或者也可以調用另一個方法:

```java
Person person = new Person("john", 26);
Optional<Person> personOptional = Optional.of(person);
```

   注意，現在我們包裹一個 Person 對象，會包含嵌套的 Optional 實例:

```java
@Test
public void givenOptional_whenFlatMapWorks_thenCorrect2() {
    Person person = new Person("john", 26);
    Optional<Person> personOptional = Optional.of(person);
 
    Optional<Optional<String>> nameOptionalWrapper  
      = personOptional.map(Person::getName);
    Optional<String> nameOptional  
      = nameOptionalWrapper.orElseThrow(IllegalArgumentException::new);
    String name1 = nameOptional.orElse("");
    assertEquals("john", name1);
 
    String name = personOptional
      .flatMap(Person::getName)
      .orElse("");
    assertEquals("john", name);
}
```

  這裏，我們試圖獲取 Person 對象的 name 屬性，然後執行一個斷言

  注意我們是如何在第七行使用 map 實現此目的的，然後注意後邊如何使用 flatMap() 方法實現同樣的目的。

  方法引用 Person::getName 類似於我們上邊去除密碼前後空格使用的 String::trim

  唯一不同點是不像 trim 返回字符串， getName 返回的是一個 Optional ，加之，map 轉換將結果包裝在一個Optional 對象中所以導致了嵌套　Optional。

  這種情況下，如果使用 map() 方法，在轉換值之前需要先將其中嵌套的　Optional 解出其值－>這個操作　flatMap 會隱身執行。



13. ## Java8 中的 Optionals 鏈


  有時，我們可能會需要從多個 Optional 中獲取第一個非空的　Optional 對象。使用像 orElseOptional()會非常方便。不幸的是，Java8 並沒有直接提供這樣的方法。

　這裏我們介紹一種新的方式:  

```java
private Optional<String> getEmpty() {
    return Optional.empty();
}
 
private Optional<String> getHello() {
    return Optional.of("hello");
}
 
private Optional<String> getBye() {
    return Optional.of("bye");
}
 
private Optional<String> createOptional(String input) {
    if (input == null || "".equals(input) || "empty".equals(input)) {
        return Optional.empty();
    }
    return Optional.of(input);
}
```

  爲了鏈接起多個 Optional 對象然後獲得第一個非空 Optional，在 java8 我們可以使用 Stream API:

```java
@Test
public void givenThreeOptionals_whenChaining_thenFirstNonEmptyIsReturned() {
    Optional<String> found = Stream.of(getEmpty(), getHello(), getBye())
      .filter(Optional::isPresent)
      .map(Optional::get)
      .findFirst();
     
    assertEquals(getHello(), found);
}
```

  這種方式的缺陷是無論非空的 Optiona 出現在　Stream 中哪個位置，所有 get 方法都會執行。

  如果我們希望傳遞給　Stream.of() 這些方法懶惰執行，我們需要使用方法引用和 Supplier 接口:

```java
@Test
public void
  givenThreeOptionals_whenChaining_thenFirstNonEmptyIsReturnedAndRestNotEvaluated() {
    Optional<String> found =
      Stream.<Supplier<Optional<String>>>of(this::getEmpty, this::getHello, this::getBye)
        .map(Supplier::get)
        .filter(Optional::isPresent)
        .map(Optional::get)
        .findFirst();
 
    assertEquals(getHello(), found);
}
```

  如果我們需要方法帶有參數，必須藉助　Lambda 表達式:

```java
@Test
public void
givenTwoOptionalsReturnedByOneArgMethod_whenChaining_thenFirstNonEmptyIsReturned() {
    Optional<String> found = Stream.<Supplier<Optional<String>>>of(
        () -> createOptional("empty"),
        () -> createOptional("hello")
      )
      .map(Supplier::get)
      .filter(Optional::isPresent)
      .map(Optional::get)
      .findFirst();
 
    assertEquals(createOptional("hello"), found);
}
```

  通常，如果鏈中所有 Optional 都是空的我們希望返回一個默認值，可以像下面通過調用 orElse() 或 orElseGet():

```java
@Test
public　void　givenTwoEmptyOptionals_whenChaining_thenDefaultIsReturned() {
    String found = Stream.<Supplier<Optional<String>>>of(
      　() -> createOptional("empty"),
      　() -> createOptional("empty")
    　 )
      .map(Supplier::get)
      .filter(Optional::isPresent)
      .map(Optional::get)
      .findFirst()
      .orElseGet(() -> "default");
 
    assertEquals("default", found);
}
```



14. ## JDK9 Optional API


  Java9 中向　Optional API添加了新方法:

- or() 方法我提供者創建一個可選的 Optional
- ifPresentOrElse() 允許如果該 Optional 值存在執行一些行爲如果不存在執行其他行爲
- stream() 方法用於將 Optional 轉換爲 Stream

　這裏是[完整的文章](完整的文章)



15. ## Optional 的誤用


  最後，讓我們看使用　Optional 　一種危險的方式: 傳遞　Optional 參數到一個方法:

  想象我們有一個　Person 列表，想要根據給定的名字搜索列表，同時如果指定一個值，我們希望匹配的結果年齡至少大於該值，該指定的值應該是可選的，所以我們寫了這段代碼:

```java
public List<Person> search(List<Person> people, String name, Optional<Integer> age) {
    // Null checks for people and name
    people.stream()
      .filter(p -> p.getName().equals(name))
      .filter(p -> p.getAge() >= age.orElse(0))
      .collect(Collectors.toList());
}
```

  我們發佈該方法後，另一個開發者這樣使用它:

```java
someObject.search(people, "Peter", null);
```

  現在，如果運行他的代碼，或拋出 NullPointerException 異常。

  所以我們必須對該可選參數執行空檢查，但是這違背了使用 Optional 的初衷，因爲使用 Optional 就是爲了避免 null 檢查。

  下面是我們處理該問題可能用到的更好的方法:

```java
public List<Person> search(List<Person> people, String name, Integer age) {
    // Null checks for people and name
 
    age = age != null ? age : 0;
    people.stream()
      .filter(p -> p.getName().equals(name))
      .filter(p -> p.getAge() >= age)      
      .collect(Collectors.toList());
}
```

  這裏，該參數依然是可選的，但是我們只用了一行代碼處理檢查。另一種可能的方式是創建兩個重載方法:

```java
public List<Person> search(List<Person> people, String name) {
    return doSearch(people, name, 0);
}
 
public List<Person> search(List<Person> people, String name, int age) {
    return doSearch(people, name, age);
}
 
private List<Person> doSearch(List<Person> people, String name, int age) {
    // Null checks for people and name
    return people.stream()
      .filter(p -> p.getName().equals(name))
      .filter(p -> p.getAge() >= age)
      .collect(Collectors.toList());
}
```

  這中方式我們提供過了清晰的 API ，通過兩個方法做不同的事(儘管他們共享實現)。

  所以，避免方法參數使用 Optional。發行 Optional 時 Java 的意圖是將其用作返回類型，用於標識一個方法可能返回一個空值。因此，在實踐中不推薦將 Optional 作爲方法參數  discouraged by some code inspectors.



16. ## 總結


  這篇文章我們涵蓋了 Java8 Optional類最重要的功能

  我們簡單的探討了爲什麼應該使用 Optional 代替顯示 null 檢查和輸入驗證

  我們也知道了當一個 Optional 是空時如何使用 get()、orElse()、orElseGet() 獲取一個默認值

  然後我們瞭解了如何使用 map()、flatMap() 和 filter() 過濾社映射 Optional

  我們也瞭解了使用流暢的 Optional API 將多個操作鏈接起來多麼容易

  最後我們瞭解了將　Optional 作爲方法參數是多麼糟糕的主意，應該避免它

  這篇文章的所有源代碼在 這裏

　　

翻譯自: https://www.baeldung.com/java-optional#conclusions

