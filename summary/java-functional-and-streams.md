# Lambda、Optional 與 Stream

## Day105
#### 學習重點 : 關於Lambda中的Method Reference - 1
- 如何更進一步去簡化Lambda呢？ ⭐⭐⭐⭐
    - 在先前的學習當中，lambda用於funtional Interface當中，常見的形式 : `(參數) -> {主體}`。
    - 使用 () -> {}，實作後再傳入接收Interface物件的method作為參數使用。
    - 雖然易讀性高，但要更加簡化的話，參數也可以省略，讓 **主體的動作更加明確**，底下這個例子很明顯 : 
    ```java=
    List<Integer> numbers = List.of(1,2,3,4,5);
    // 有參數形式的lambda
    numbers.forEach(number -> System.out.println(number));
    // 主體明確展示出來，省略了搬number的表達式
    numbers.forEach(System.out::println);
    ```
- Method reference初見 ⭐⭐⭐⭐⭐
    - 從剛剛的程式範例可以感受到，Method **似乎變成了參數被傳入**，但其實只是省略了搬參數的動作，感覺好像真的只傳了個method而不是實作。
    - 這樣的動作 **取代** 傳統lambda所謂的 **怎麼做**（取參數->實作->物件傳入），而是 **作甚麼**（傳method）。
    - 由於這個部分還很多部分需要理解，我打算分個5、6天慢慢研究，今天先這樣！

## Day106
#### 學習重點 : 關於Lambda中的Method Reference - 2
- 如何實際應用Method Reference在類別層面上？ ⭐⭐⭐⭐
    - 先來看看底下的實作程式碼 : 
    ```java=
    public class Greeter {
        public void greet(String name){
            System.out.println("Hello " + name);
        }
    }
    ```
    ```java=
    Greeter greeter = new Greeter();
    List<String> names = List.of("Julian", "Bob", "Alice");
    names.forEach(greeter::greet);
    ```
    - 在實際學習應用時，我有一個疑問 : **greeter又不是介面**，為何可以使用Method Reference？ ⭐⭐⭐⭐⭐⭐⭐⭐
        - 當我們實際點進forEach方法時，可以發現長這樣 : 
        ```java=
        default void forEach(Consumer<? super T> action) {
            Objects.requireNonNull(action);
            for (T t : this) {
                action.accept(t);
            }
        }
        ```
        - 可以知道當我們傳方法參考進去後 --> 會變成Comsumer介面物件（action），而action的accept方法中的實作即為我們所寫入的greeter動作，型別由List<>決定！
    - 所以總結來說，當我傳方法（**提供動作**）時，並不是以方法所在地是不是介面來看，而是以forEach這種（**接收動作**）的所在地來看的，也就是Comsumer啦~

## Day107
#### 學習重點 : 關於Lambda中的Method Reference - 3
- 關於Map配對動作的使用 ⭐⭐⭐⭐⭐⭐
    - 在List中，可以使用List提供的map功能進行建構！那麼實際是怎麼建構的呢？
    - 底下是範例程式 : 
    ```java=
    List<String> family = Arrays.asList("Alice", "Bob", "Ken");
    List<Person> people = family.stream()
            .map(Person::new)
            .toList();
    ```
    - Person是自行建立的類別，建構時接收一個 `String name`，當我想要建構一系列Person物件時，需要一個名單 ➝ 這時靠著family提供名單，並藉由List的stream功能遍歷整個family List。
    - 由於Person接收String類別，因此透過Method Reference使用new去尋找對應單一String參數的建構子 **並建構** ➝ 接著再轉成一系列Person物件。
- 為何要這麼複雜，不是可以用for來建構嗎？ ⭐⭐⭐⭐
    - 這就是聲明式的優點，透過method來完成事情，比起使用for loop一個一個新增還來的清晰，儘管**效能比不上for loop**這種原生語法，且**不好做流程控制**，但在某些情境上使用聲明式會比for loop好上很多！

## Day108
#### 學習重點 : Optional
- 甚麼是Optional？ ⭐⭐⭐⭐⭐
    - 在製作SpringBoot小專案的時候，常常會看到Optional的身影，尤其是當我return一些東西時，編譯器總是叫我使用Optional來寫，因此我想說透過Method Reference的銜接來看看Optional的真面目！
    #### Optional解決NullPointerException
    - 這是Optional設計的初衷，一般來說，當我寫一個method會回傳物件時，物件是null就會發生 **NPE**，因此常會在呼叫method的同時先做 `if (XX != null)...` 的判斷。
    - 而Optional設計用來「包裝」一個物件，無論其是否是null都可以被包裝成Optional物件。
    - 以下是一個簡單的程式範例 : 
    ```java=
    public class OptionalT{
        public static void main(String[] args){
            Optional<Cat> optionalCat = findCatByName("John");
            if (optionalCat.isPresent()){
                System.out.println(optionalCat.get().getAge());
            }else{
                System.out.println(0);
            }
        }
        
        public static Optional<Cat> findCatByName(String name){
            // ...省略尋找喵的過程，反正最後會有個cat被找到，可能是null
            // ofNullable代表說這個Optional容器可能為空
            return Optional.ofNullable(cat);
        }
    }
    ```
    - 以上的範例似乎跟一般判斷是否是null很類似對吧~ 但透過使用Optional可以讓寫程式的人注意到這個method可能會出現NPE情況。
- 如何使用Method Reference在Optional上？ ⭐⭐⭐⭐
    - 首先若想要簡化if else的判斷，可以透過「聲明式」來取代流程控制，就像前幾天所看到List的動作。
    ```java=
    optionalCat.map(Cat::getAge).orElse(0);
    ```
    - 與昨天所使用的map很像，透過Cat型別以及getAge回傳的Integer型別形成mapper，接著若Optional是Empty則回傳空Optional，若有cat則回傳cat，並執行getAge動作。

## Day109
#### 學習重點 : Optional的應用
- 甚麼時候應避免使用Optional ⭐⭐⭐⭐
    - 使用Optional時，應避免用於以下部分 : **方法參數、成員變數**，還有更重要的是，**避免isPresent()+get()的流程搭配**，因為這樣跟判斷null本質上**都是命令式**，即失去了聲明式的用意。
    - 當用於「方法參數」時，除了處理Optional是否包含東西，還有可能整個Optional就是null --> 因此要避免。
    - 而用於「成員變數」時，若需要序列化，由於Serializable就像之前理解的，無法序列化Optional這種物件 --> 因此要避免。
- 如何應用於SpringBoot中RESTful API的場景？ ⭐⭐⭐⭐⭐⭐
    - 在Optional有個方法叫做filter，很適合用於Service判斷或者驗證，因此我們可以將filter以及map還有最後的orElseThrow結合形成聲明式的邏輯。
    ```java=
    public Response<User> login(UserRequest userRequest) {
        return implementUserDao
            .findUserByAccount(userRequest.getAccount())
            .filter(/*驗證Lambda表達式*/)
            .map(/*Response邏輯回應*/)
            .orElseThrow(/*丟出Exception*/));
    }
    ```
    - 利用我之前的小專案可以認知到 --> 透過聲明式搭配能更加看到動作流程。

## Day110
#### 學習重點 : Optional method理解
- filter ⭐⭐⭐⭐⭐
    - 深入來看filter的定義 : 
![螢幕擷取畫面 2026-04-20 203047](https://hackmd.io/_uploads/rJ-Mo_e-Ge.png)
    - 基本上 --> 🌟會需要使用到filter，就是需要進行驗證or判斷時🌟，且最重要的是要「**跟Optional容器內的T型別有關**」的判斷 --> which means 當我的邏輯跟 `<T>` 無關時，就不太需要寫在Optional聲明式當中。
    ![螢幕擷取畫面 2026-04-20 203847](https://hackmd.io/_uploads/Byizsug-Gx.png)
    - 像是上面的程式，tpu跟 `Optional<User>` 無關，因此不需filter，而是直接if else，再進入聲明式。
- map ⭐⭐⭐⭐⭐
    - map的定義長這樣 :  
    ![螢幕擷取畫面 2026-04-20 204945](https://hackmd.io/_uploads/ry87iOgWfl.png)
    - 看得很亂，我嘗試釐清，T源自於一開始包裝的 `Optional<T>`，而U呢??
    - U源自於「泛型方法」所定義，這個看似很玄學，但回到 [Day27 : Generics - 2](https://hackmd.io/@learning-official/Java_learning##Day27)中，似乎就很清楚了。
    - 而map定義回傳的是 `Optional<U>`，因此我們在實作mapper時，最終回傳的型態即是U。
    - 在小專案當中，U就是Response，因此可以實作mapper，並加入更新邏輯。

## Day157
#### 學習重點 : var與Collectors簡單理解與應用
- 甚麼是var？ ⭐⭐⭐
    - 簡單來說，var用於宣告時，根據「右值型態」決定變數型態的一個關鍵字。
    - 簡易範例如下 : 
    ```java=
    var list = new ArrayList<String>("Alice", "Bob", "Carol");
    ```
    - 透過範例可以知道原本是以 `List<String>` 宣告，現在變成了var代替，由於我們知道函式回傳的是 `List<T>`，因此這邊用var來接收很剛好！
    - 但是！如果今天是是這樣 
    ```java= 
    var unknown = Yeah.getInstance();
    ```
    - 我根本不知道Yeah.getInstance要回傳甚麼，這時候用var就不太合適！
- Collectors是甚麼？ ⭐⭐⭐⭐⭐⭐
    - Collectors在stream當中是一個很重要的類別，如字面意思，它是用於「收集」stream內的元素。
    - 但是為甚麼要從一個箱子收集到另外一個箱子呢？
        - 重點在於👉若我們今天遇到 **業務邏輯** 要先經過「**驗證、審核、過濾**」等條件時，就可以很簡單的做出過濾箱子，而不需要再自行建箱子+跑迴圈加進去。
    - 以下是「直接過濾」vs「跑迴圈過濾」的比較 : 
        - 迴圈過濾 : 
        ```java=
        var list = Arrays.asList("Alice", "Bob", "Carol");
        var fileterList = new ArrayList<String>();
    
        for (var e : list){
            if (e.length() >= 4){
                fileterList.add(e);
            }
        }
        System.out.println(fileterList);
        ```
        - stream.filter.collect : 
        ```java=
        var list = Arrays.asList("Alice", "Bob", "Carol");
        var filterList = list
                .stream()
                .filter(s -> s.length() >= 4)
                .collect(Collectors.toList());
        System.out.println(filterList);
        ```

## Day158
#### 學習重點 : Collectors的Map
- Map的應用 ⭐⭐⭐⭐⭐
    - 當我們想利用物件成員做配對，如 `"名字" : 年齡` or `"名字" : "居住地"`，可以直接寫 : 
    ```java=
    User user = new User("小八", "東京");
    Map<String, String> map = new HashMap<>();
    map.put(userTest.getName(), userTest.getCity());
    ```
    - 但如果今天是一坨拉庫的User呢？ 總不可能一個一個put吧！
    - 這時候Collectors.toMap就派上用場了 : 
    - 仔細看，toMap的運作方式 : 兩個參數都是 `Function`，代表我們可以實作！
    ![image](https://hackmd.io/_uploads/SyOEA2zWGx.png)
        ```java=
        Map<String, String> userMap = users.stream()
            .collect(Collectors.toMap(
                User::getName, // Key
                User::getCity // Value
        ));
        ```
        - 上述是一般實作，也就是簡單取出成員！但也可以做變化，如下 :
        ```java=
        Map<String, String> userMap = users.stream()
            .collect(Collectors.toMap(
                User::getUUID, // Key
                u -> {
                    return u.getCity() != null ? u.getCity() : "無家可歸";
                } // Value
        ));
        ```
        - 當使用者是流浪漢時，就幫他加註「無家可歸」😵‍💫
- 使用toMap要注意的事情 : ⭐⭐⭐⭐⭐⭐ 
    - 1️⃣ 不允許NPE的出現，因此如果偵測到null，務必做邏輯處理(像Optional那樣)
    - 2️⃣ 由於key是unique的，因此不能使用getName作為Key，應該使用唯一鍵。
    - 如果今天Key真的遇到重複時，那該怎麼辦呢？
    ![image](https://hackmd.io/_uploads/SyFbG6GWMg.png)
    - 這時就需要加入mergeFunction，這個也是toMap的一種形式，用於決定 **要保留衝突鍵的舊鍵還是新鍵**。
    - 此時程式如下 : 
    ```java=
    Map<String, String> userMap = users.stream()
        .collect(Collectors.toMap(
            User::getUUID, // Key
            u -> {
                return u.getCity() != null ? u.getCity() : "無家可歸";
            }, // Value
            (existing, replacement) -> existing // 保留舊鍵
    ));
    ```

## Day159
#### 學習重點 : Collectors的group
- Collectors.groupingBy ⭐⭐⭐⭐⭐⭐⭐⭐
    - 這個用法用在Map鍵值對，根據 **成員屬性** 進行分組，常見範例如下 : 
    ```java=
    // 基本的分類
    Map<String, List<User>> groupByCity = users.stream
        .collect(Collectors.groupingBy(
                User::getCity
            )    
    )
    // 多級分組
    Map<String, Map<String, List<User>>> groupByCityAndDpt = users.stream
        .collect(Collectors.groupingBy(
            User::getCity,
            Collectors.groupingBy(
                User::getDepartment    
            )
    ))
    ```
    - 由多級分組可以知道 👉 groupingBy的第二個參數可以接收一個Collectors，而收集的「根據」就是上一層分類好的 `Map<User::getCity, List<User>>`，因此帶著前一層的資料進到下游分類區時，會長這樣 👉 `<User::getCity, Map<User::getDepartment, List<User>>>`。
    - 這邊調出srouceCode來看 : 
    ![image](https://hackmd.io/_uploads/rJt81XN-Me.png)
    ![image](https://hackmd.io/_uploads/SJHOJm4-Gx.png)
    - downstream回傳 `<T, A, D>`，從第一張圖來看就是 `<T, ?, Map>`，因此作為下游分類好的Map被丟回了上游時變成了D(value)，而key則是第一層所根據的User::getCity。
- 流程 : ⭐⭐⭐⭐⭐⭐
    - 由於上述都是以泛型來看，很燒腦ww，我稍微理了一下思緒 : 
        1. 呼叫上游函式時，根據City分類好箱子
        2. 以大箱子City的下去細分，再根據每個箱子去做第二層分類
        3. 最後做總包裝

## Day160
#### 學習重點 : Collectors的group、count與partition
- group的mapping
    - mapping在group當中就是將分類後的資料做加工變形！
    - 一般來說，當我們grouping完後，回傳的是User的物件地址，但若我想要以呈現使用者名字，就可以使用mapping來「變形」。
    ```java=
    Map<String, Set<String>> groupMap = users.stream()
        .collect(
        Collectors.groupingBy(
            User::getCity,
            Collectors.mapping(
                User::getName, 
                Collectors.toSet()
            )
        )
    );
    ```
    - 這邊Set亦即當某City中有重複的名稱，只許出現一次，也順便複習了之前學的Set！
- counting ⭐⭐⭐⭐⭐
    - 在Collectors當中，有個很好用的工具 : 計數器，原本我不太知道為甚麼要有它（我覺得可以直接用List.size()就好了owo）。
    - 但後來試想了一下，grouping時都是「以Key + toList在做分類」，若有數百萬筆資料，且我只想知道每組筆數時，還要先toList在getSize的話會 **效能爆炸**🤯
    - 因此今天如果只是要計數時，**不需要再建空間**，直接讓counting把關即可！
    - 如下方範例 : 
    ```java=
    Map<String, Map<String, Long>> countByDept = users.stream
        .collect(Collectors.groupingBy(
            User::getCity,
            Collectors.groupingBy(
                User::getDepartment,
                Collectors.counting()
            )
        )
    );
    ```
    - 根據城市與部門分組，計每個城市中的不同部門人數。
- partition ⭐⭐⭐⭐
    - 用於做「**布林分組**」，亦即只有true或false，也算一種grouping ouo? 因此跟grouping一樣可以加入Collectors的工具，像是多級grouping或者counting。
    - 以下是範例 : 
    ```java=
    Map<Boolean, Map<String, List<User>>> partition = users.stream()
        .collect(
            Collectors.partitioningBy(
                u -> u.getAge() >= 20,
                Collectors.groupingBy(
                    User::getCity
            )
        )
    );
    ```

## Day161
#### 學習重點 : Collectors的reducing
- reducing是甚麼？ ⭐⭐⭐
    - reducing的中文意思是「歸納...減少」，因此可以理解成「集合所有元素成一體」，亦即我們可以利用stream的元素來做集合動作，先來看看sourceCode！
- 原始碼解析與實作 ⭐⭐⭐⭐⭐⭐⭐⭐
    - 以下是單參數版本 : 
    ![螢幕擷取畫面 2026-06-10 222215](https://hackmd.io/_uploads/HJrdqxvbfe.png)
    - `BinaryOperation` 又繼承FunctionalInterface之 `BiFunction`，如下 : 
    ![螢幕擷取畫面 2026-06-10 222351](https://hackmd.io/_uploads/HJ2U5xv-fl.png)
    - 很明白的表示我們需要「實作」兩個參數的函式，因此才叫做"Bi"嘛~
    - 因此我透過user.age相加來實際測試看看 : 
        ```java=
        Optional<Integer> ss = users.stream()
                .map(User::getAge)
                .collect(
                Collectors.reducing(
                    (a, b) -> a + b
                )
            );
        System.out.println(ss.get());
        ```
        - 使用Optional是因為有可能沒有元素導致回傳null。
        - 由於原始碼中的邏輯應用到我的測試是以users內元素的User作為T值，但我們要拿的是Integer，因此要先用map取出正確型態的元素。
    - 以下是三參數版本 : 
    ![螢幕擷取畫面 2026-06-10 221228](https://hackmd.io/_uploads/By-r9gvWMg.png)
        - Identity作為「初始值」，亦即當集合內沒有元素的話，會回傳初始值。
        - mapper也就是我可以先利用函式取值，並回傳Identity型態的值，這代表我可以代替上述用stream.map的過程。
        ```java=
        Integer ss = users.stream()
            .collect(
            Collectors.reducing(
                0,
                User::getAge,
                (a, b) -> a + b
            )
        );
        System.out.println(ss);
        ```
        - 由於有Identity的初始化，因此不須使用Optional！

## Day162
#### 學習重點 : Collectors的filtering
- filtering ⭐⭐⭐⭐⭐⭐
    - 今天有點不蘇胡，先做個比較簡單的filter功能就好ㄌ。
    ![image](https://hackmd.io/_uploads/BJyIsBdZzl.png)
    - 透過原始碼可以知道 --> 接受一個Predicate，亦即一個我們預期的判斷式，符合預期則true。
    - 若為true則被downstream接受進行後續步驟。
    - 簡易範例 : 
        ```java=
        Long agefilter = users.stream().collect(
                Collectors.filtering(
                    u -> u.getAge() >= 20, 
                    Collectors.counting()
            )
        );
        ```
    #### 為何不用stream的filter就好？
    - 當我建立箱子時，「若箱子為空 **是否** 要移除這個箱子」？這個問題可以確認是否要用stream.filter。
    - 若我在進Collectors前就先過濾（stream.filter.collect），代表我後續在分組時，很有可能會導致箱子缺少。
    - 舉個例子 : 按照班級分組，過濾掉男生，若某個班級都是男生，等於那個班級的箱子直接消失！這時候就必須使用collectors.filtering，先建箱子再過濾！這樣那個箱子會是空列表！

## Day163
#### 學習重點 : java.util的function介面
- function介面群 ⭐⭐
    - 今天稍微來看點特別的東西 : function介面，JDK定義一些比較常用的介面給開發者使用，讓開發者不需要再自行寫介面！
    - 常見的有Consumer、Function、Predicate、Supplier。
    - 今天先寫個Consumer就好owo。
- Consumer ⭐⭐⭐⭐
    - 接收一個物件，並執行，實際寫寫看！
    ```java=
    import java.util.function.*;

    public class Test{

        public static void main(String[] args){
            
            // 直接implement後呼叫
            Consumer<String> testConsumer = System.out::println;
            testConsumer.accept("Hello World");
            
            // 藉由method接收後來做！
            String msg = "WoW";
            testConsumerVoid(System.out::println, msg);

        }
        
        // 實際method搭配Generics的method語法
        public static <T> void testConsumerVoid(Consumer<T> c, T msg){
            c.accept(msg);
        }
    }
    ```

## Day164
#### 學習重點 : java.util的function介面群之Function
- Function架構與實作 ⭐⭐⭐⭐⭐
    - 從名字可以知道，是一個函數！因此跟數學式一樣，接收一個x，回傳一個y，`y = f(x)`，在程式當中，兩者型態不一定要相同。
    - 從介面定義來看 : 
    ![image](https://hackmd.io/_uploads/rkX3PYqbze.png)
    - 接收一個T物件做完邏輯處理後回傳R物件。
    - 這讓我聯想到了之前一直在用的 **Optional.map功能** : `取出user進行業務處理後return一個Response`，這就是實踐Function介面的一個實例。
    - 我也列出了透過兩種不同實踐的方式以及Optional的研究 :
    ![image](https://hackmd.io/_uploads/HyT60Kqbfg.png)
- compose與andthen用法 ⭐⭐⭐⭐⭐
     - 兩者基本上就是數學式中的 `y = f(g(x))`！因此在泛型當中會是 `先<V, T> 再 <T, R>`，因此T是中繼。
     - 舉個例子 : `User接getAge，再接判斷age > 20` 判斷回傳boolean，這樣子就會是 `<User, Integer>、<Integer, Boolean>`，以下是我的演練 : 
     ![image](https://hackmd.io/_uploads/SJOx83c-Mx.png)
     - 我這邊用if name是小八來做測試w。
     - 基本讓andthen就是先處理外再處理內，compose是先處理內再處理外，因此按照我的測試，x會先被處理，y其次。

## Day165
#### 學習重點 : java.util的function介面群之Predicate
- Predicate ⭐⭐⭐⭐⭐
    - 基本上Predicate就是在做預測，因此會回傳boolean值。
    - 跟Function一樣，也有一些default method，像是not、negate、or、and。
    - 比較特別的是它有一個static的method : `isEqual(Object obj)`，簡單來說它可以擋掉NPE問題，不須自己再處理。
    ![螢幕擷取畫面 2026-06-14 095126](https://hackmd.io/_uploads/SJ6OX-3bzx.png)
    上面實作了一下Predicate的default用法。

## Day166
#### 學習重點 : java.util的function介面群之Supplier
- Supplier ⭐⭐⭐⭐⭐⭐
    - Supplier是四個介面群我最問號的w，它看起來就很多餘，先來看看原始碼。
    ![image](https://hackmd.io/_uploads/rk-4_caZze.png)
    - 但經過我仔細研究後發現了一個重大秘密 : 它可以「**延遲加載動作**」！
    - 回想之前在用Optional時，都會在最後寫 `orElseThrow()` 或 `orElseGet()`，參數都是 `() -> ...`，其實這就是在實作Supplier。
    #### 但為甚麼不直接寫 `orElse()` 就好呢？
    - **Eager Evaluation** vs. **Lazy Evaluation**
    - 原因在 : 當我寫下 `orElse(動作)` 時，Optional會「**先執行該動作並取得值**」，再看Optional元素是否為null決定是否回傳該值。這種先執行取值的概念叫做 **Eager Evaluation**。
    - 儘管它在小型求值很簡便，如 `username.orElse("未知使用者")`，但如果遇到 `user.orElse(new User("訪客", "0001", ....))` 這種需建構的動作時(耗效能)，則可以改用orElseGet來 **Lazy Evaluation**，也就是延遲加載 : 元素為null才執行，因此參數會先以Supplier「接住」該動作，待元素為null再觸發該動作！

## Day276
#### 學習重點 : Java OCP測驗.6
- 測驗內容 🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟
    - Stream 之 IntStream ⭐⭐⭐⭐⭐⭐⭐
        - IntStream的出現是為了解決 `Stream<Integer>` 的問題。
        - 由於 `Stream<T>` 是處理物件，因此 `一組primitive` 在做運算時都要先做 boxing（`Integer.valueOf`） 再做 unboxing（`intValue`），十分耗資源，而且不支援「**計算**」。
        - 因此Java提供了 `IntStream` 直接使用原生 `int[]` 做存取運算，更加有效率！
        - 而基本的用法如下 : 
            - `.range(startInclusive, endExclusive)` : 接收兩個參數，以step為1生成一組int，開頭包含 & 結尾不包含，回傳IntStream。
            - `.rangeClosed(startInclusive, endInclusive)` : 跟上一個類似，但結尾包含。
            - `.concat(intStream1, intStream2)` : 將兩個int陣列做結合**標記**，**屬於Lazy Evaluation**，等到Terminal才真正意義上將其結合。
            - `.parallel()` : 他會將IntStream以 **多執行緒** 下去做後續動作，因此當我Terminal使用 `.forEach` 時，會分配不同執行緒對IntStream元素做操作（但不保證元素處理順序），其本質上還是IntStream，只是內部在操作時會以多執行緒下去做。
        - 而最後我們還是可以使用 `.boxed` 將其包裝成 `Stream<Integer>`，做後續動作！（裝箱後就失去了IntStream設計的精隨，因此放到最後在裝箱才是真理！）
