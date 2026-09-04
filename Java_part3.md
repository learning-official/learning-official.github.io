## Day172
#### 學習重點 : Enum
- Enum概念 ⭐⭐⭐⭐⭐⭐
    - 雖然我對Enum有些許的理解與實際應用過，但我認為把它提出來做個筆記還是必要的！
    - 首先當然是認識它的意義，其實 **Enum就是一種class**，但它的存在是去定義**狀態、常值**等**不變**的東西，因此可以理解Enum是一種加上了final關鍵字的class。
    - 不過Enum**還是存在建構行為**，但外部不能「直接」用new去建構（Enum預設建構子是private），這樣才能保證Enum的原始狀態。真正建構物件是在Enum內執行的，且這些成員物件都是以final狀態出生的，因此Enum內部不會有Setter的出現！
    - Enum物件都是在編譯期即鎖死於Enum，並作為public final的成員被外部調用，這也是為何我們都是呼叫 `Enum.STATUS...` 類似這種格式。
    - 由於Enum是在編譯期即建構好物件，因此Enum實現建構式的方式就像下面這樣 : 
    ![image](https://hackmd.io/_uploads/Bynz68HGfe.png)
    - 這邊會報錯也是因為Enum被實現的動作只能由JVM實現，因此自行extends Enum是不行的！

## Day173
#### 學習重點 : Enum的實際架構
- 真實的Enum ⭐⭐⭐⭐⭐
    - 在宣告Enum時，通常會有以下過程 : 列舉 --> 宣告成員 --> 建構式 --> Getter。
    - 一個狀態是一個物件，而物件成員透過列舉放的參數進行建構，最後加上Getter，實際架構如下 : 
    ```java=
    public enum EnumT {
        
        // 列舉(成員參數)
        A("A", 0),
        B("B", 1);
        
        // 成員
        private final String a;
        private final int b;
    
        // 參數建構
        EnumT(String a, int b){
            this.a = a;
            this.b = b;
        }
        
        // Getter
        public String getA(){
            return this.a;
        } 

        public int getB(){
            return this.b;
        }
    }
    ```
- 隱藏的成員 ⭐⭐
    - 像昨天所寫，其實Enum本身是一個Class，當我們宣告一個A作為列舉檔時，A會在編譯時自動轉成class，並繼承「Enum」，而在Enum中，會記錄列舉的name、ordinal，如下所示 : 
    ![image](https://hackmd.io/_uploads/SytR45UMGe.png)
    - 會印出「A0」。

## Day174
#### 學習重點 : java.lang的Object
- Object的地位 ⭐⭐⭐⭐⭐⭐
    - Object在Java中是主宰類別的最高存在，只要建立一個class，這個class就會「隱性」繼承Object這個類別。
    - 即便是Interface，雖然不是class，但在Java規範中，Interface會以Implicit Declaration的方式賦予與Object這個類別相同的抽象方法，因為介面最後**還是會被實例化**，所以最後這些抽象方法也會因繼承Object而被自動實作。
    - 一般來說Object會處理 : 物件的身分(記憶體)、反射、物件的生命週期(執行緒)、物件的複製。
    - 因此我們平常在用的equals、getClass()、toString其實都是出自於Object之手！
- 簡單equals、toString、hashCode實作 ⭐⭐⭐⭐⭐⭐
    - equals是甚麼？
        - 其在Object的功能是 --> **比對物件的邏輯上的一致性**，它的預設是去 `比對記憶體位置是否相同（用==來比對）`，這就是其中一種邏輯一致。
        - 而我們也可以覆寫成 `根據成員去比對`。
    - 以下是我覆寫Object的部分method : 
    ![image](https://hackmd.io/_uploads/r10qlovMzg.png)
    - 這邊要注意！覆寫equals就必須覆寫hashCode！反之亦然，因為在做物件存取比對時，會經過兩道手續 : 
        - 1️⃣ 比較物件經hash後的index是否相同。
        - 2️⃣ index相同則以equals判斷兩者是否為同一物件 --> 若不同則collision，執行後續hash邏輯; 若相同則代表兩者為同一物件。
    - 透過上述兩個步驟可以知道 --> hashCode、equals必須實作概念必須相同(如同時使用name、age)。
    #### 為甚麼不先equals看看table中有沒有相同記憶體位置？
    - 雜湊表底層也是一個陣列，若要一個個equals，則為線性搜尋 --> O(n)
    - 若使用hash index可以直接定位 --> O(1)

## Day175
#### 學習重點 : java.lang的Object之clone
- 留言 ⭐
    - 之後的這20天，我的更新進度會慢許多，由於很多活動都卡在這個時段，因此我預計會把觀念再切碎的更多，當然，還是會保持每天更新的節奏ouo。
- clone ⭐⭐⭐⭐⭐
    - 今天來討論Object中的clone！儘管clone大部分時間都不會被「真的」拿來用，但還是要了解一下！
    - 首先要注意，clone的用途是 --> 代替建構子，copy出新物件，但級別屬於「**Shallow Copy**」，因此針對Wrapper member，只會copy reference。
    - 在覆寫clone時，需要在class中加上 `implements Cloneable`，使這個覆寫合法（雖然Cloneable沒有定義clone，反而是在Object中定義的，但Java說要加ouo）。
    - 且在clone覆寫中，必須throws Clone的例外。
    ![image](https://hackmd.io/_uploads/HygJp8tfzg.png)

## Day176
#### 學習重點 : java.lang的Object之deep copy
- 續看clone ⭐⭐⭐⭐⭐⭐
    - 接續昨天的clone，今天來討論將clone實作成deep copy的形式。
    - 一般使用super.clone()，意即去呼叫父類的clone，此時其實是以子類的身分 : `this指向子類`，到父類去做事，但由於super的關係，使多型暫時失效，轉以父類執行clone。
    - 但由於Object定義clone是以 `native` 定義，亦即使用原生語言(C++)，此時 C++ **找的是this所在的記憶體位置**，因此super多型遮罩在 C++ 不管用，這也是為甚麼明明寫super.clone，卻是去複製自己，而不是複製父類(**原因就在this的歸屬**)。
    - 這邊有找到幾個詞彙，之後可以研究研究 : `Klass、invokevirtual/invokespecial、Field Hiding`
- Deep copy的實作 ⭐⭐⭐
    - 其原理很簡單，使用super.clone()後，還需要針對本身的Wrapper成員再進行一次clone。
    ![image](https://hackmd.io/_uploads/S1TKpjqffl.png)
    - 這邊加了個ObjectCloneTest的Wrapper成員，因此要再針對它再進行一次clone。

## Day177
#### 學習重點 : java.lang的Object之執行緒
- wait/notify/notifyAll ⭐⭐⭐⭐⭐⭐⭐⭐⭐
    - 簡單來說，這三者專門處理Object這個資源的控管！
    - 以搶票系統來看 : 若一坨人要搶A區的票，A區作為一個Object紀錄剩餘票數，若發現票已售完，後續排隊用戶的進入購票環節時會因 `synchronized` 緣故，依序取用A區的資源，若發現沒有票則執行 `(A區)this.wait()`，意即進到A區資源的等待區，並交出自身的對A區的「**資源鎖**」給下一個執行緒。
    - 等有使用者釋票後，就會發出 `A區(this).notify()` 去「喚醒」其中一個在等待區的執行緒。
    - 而notifyAll就只是一次喚醒「所有」等待的執行緒而已。
    - 不過這產生一個問題 : 如果排頭檢查票數發現是0，並到等待區等待，結果第二個人進來時，剛好有票了，第二個人就順勢購票了。
        - 因此synchronized這種monitor lock又被稱為「**Non-fair Lock**」
        - 此時改用ReentrantLock會更加FAIR！！
- 為何wait/notify要在Object中？ ⭐⭐⭐
    - 一般來說，在討論執行緒時，都會以Thread/Runnable等為主，但是為甚麼wait/notify這些東西會出現在Object中呢？
    - 主要原因 : Object凌駕於所有class之上，且任何class都可以是「資源」，若以執行緒(使用者)去呼叫wait，會導致JVM `不知道這個執行緒是要進到哪個資源的等待區？`，這也是為何不會直接以Thread呼叫wait！

## Day178
#### 學習重點 : java.lang的Record
- 基本概念 ⭐⭐⭐⭐
    - 基本上Record在Java中是一個很方便建立model等的關鍵字。
    - 於Java 16引入，非常方便用於設計一些「不可變」的類別。
    - 其簡化了我們寫參數建構子、Getter、覆寫Object功能的過程。
    - 因為是不可變，因此當我們給定成員後，會自動變成final形式(class本身也是final)，且沒有Setter。
    - 由於record類別是去繼承java.lang的Record抽象類別，因此寫record不能再繼承其他類別，也不能被其他類別繼承！
- 實際範例 ⭐⭐⭐
    - 十分簡單易懂ww
    ![image](https://hackmd.io/_uploads/rJZtYWafMe.png)
    - 當然，也可以實作 : 
    ![image](https://hackmd.io/_uploads/SJqYKWpzfx.png)
    - 這邊就用前幾天碰到的Cloneable來示範實作ouo。
    - 但其實簡化成這樣，底層還是會還原成 : 
    ![image](https://hackmd.io/_uploads/Sk45F-Tffx.png)

## Day179
#### 學習重點 : java.lang的Record複製
- record的copy ⭐⭐⭐⭐
    - 今天看個簡單的主題，Record的複製！
    - 一般在class當中，我們可能會寫下一些成員，他們不是被參數指派，隨機產生、根據時間不同而有不同數值。像是下面這個 : 
    ```java=
    public class User {
        private String name;
        private long createTime;

        public User(String name) {
            this.name = name;
            this.createTime = System.currentTimeMillis();
        }    
    }
    ```
    - 此時若我們建構一個新的user，但copy了另一個User的name，此時用equals判斷會不一樣。
    - 但如果我們使用record，由於定義不可變及可預測性，我們只要透過`R newR = new R(oldR.name(), oldR.age()...)` 建構完畢後，newR與oldR做equals必是True。

## Day180
#### 學習重點 : java.lang的Record簡便用法
- 函式內宣告 ⭐⭐⭐
    - 就像匿名類別一樣，Record也可以被簡單實現在函式內部中被「單次」使用。
    - 以下是範例 : 
    ```java=
    public void calculate(List<Merchandise> list) {
        // ...建立List的過程
        record TempTotal(Long id, double finalPrice) {}

        List<TempTotal> result = list.stream()
            .map(m -> new TempTotal(m.getId(), m.getPrice() * 0.8))
            .toList();
    }
    ```

## Day181
#### 學習重點 : Java是甚麼？
- 前言 ⭐
    - 我想，這段時間有太多事情，學習語法或者框架可能會造成學習成效不好，因此我打算來好好認識一下Java ( 學習這麼久，但從來沒有認真了解過Java的背景ww
- JVM ⭐⭐⭐⭐⭐
    - Java Virtual Machine，意即Java虛擬機器，其提供了完整的處理器、Stack、Register等。
    - JVM接收了Java bytecode來執行。因此，只要能成功編譯成java bytecode(.class)的程式碼，**理論上都可在JVM上執行**。
- WORA ⭐⭐
    - Write Once, Run Anywhere，意即上述的JVM接收Java bytecode，使我們不需重新編譯Java程式碼，**只需編譯一次，即可在任何支援平台上執行。**

## Day182
#### 學習重點 : JDK與JRE
- JDK ⭐⭐⭐⭐
    - Java Development Kit，是一套很完整的Java開發套件，像我們常常使用的javac，就是JDK的套件。
    - 其內部也包含JVM、編譯、執行、打包jar檔等完整組件
    - 而對於一般使用者，無須下載JDK去執行Java程式 --> 因為只要取得bytecode即可！
    - 但對於程式開發者來說，JDK需要手動安裝。
    - 而一般都是以Java SE作為標準版來使用。
- JRE ⭐⭐⭐⭐
    - Java Runtime Environment，由JVM以及Java 核心API組成，當使用者想執行Java程式(bytecode)時，可以只使用JRE來執行，無需另外加入JDK。
    - 因此可以說JRE就是被砍掉編譯功能的JDK！

## Day183
#### 學習重點 : JRE的成分
- JRE解析 ⭐⭐⭐⭐
    - 在JRE中，基本上存在幾個部分 : JVM核心元件、核心類別庫、部屬工具。
    - 核心元件 : ClassLoader、Bytecode驗證、解讀器。
    - 類別庫 : XML分析、.lang、.io、.security等...
    - 部屬工具 : javapackager...
- JRE常見的綜合庫類別 ⭐⭐
    - 通訊 : JDBC、JNDI、JWS...
    - 視窗介面 : AWT、Swing...

## Day184
#### 學習重點 : JIT與Hotpot JVM
- JIT、Hotpot JVM ⭐⭐⭐⭐⭐
    - Just-in-time是指稱一種動態(即時)編譯器，其可將Java bytecode轉換成機器碼 --> 且可以抓取常用程式區塊，使其成為本地機器碼，提高程式執行效率。
    - JIT是Hotpot JVM執行模式下的其中一種模式，另一種是Interpretation
    - 註記 : Hotpot JVM是JVM中很受歡迎的虛擬機，除了JIT的貢獻之外，Hotpot也有引入GC的垃圾回收機制！
    - 這張圖我覺得不錯 : 
    ![image](https://hackmd.io/_uploads/HJTwT1SQMx.png)

## Day185
#### 學習重點 : JIT Compiler
- 介紹 ⭐⭐⭐
    - 如昨天所介紹，JIT是Hotspot JVM的核心技術之一，其技術精華在於以「熱點」偵測哪些程式被頻繁執行，並將其編譯、優化。
    - JIT的架構可以看底下這張圖 : 
    ![image](https://hackmd.io/_uploads/S1I71sImzx.png)
- C1、C2 compiler ⭐⭐⭐⭐⭐
    - C1編譯器主要負責的是需快速啟動、編譯的程式，由於其編譯的速度很快且延遲小，很適合需短期、快速運行的程式。
    - C2編譯器屬於高級編譯器，當熱點偵測到程式被重複執行多次後，會將其從C1轉交給C2編譯器管理，將其編譯、優化，儘管花的時間較長，但在需長期運行的程式當中，十分有利。
    - C2編譯的方式，就很適合後端的程式使用，因為其在伺服端上會很常被呼叫，這也是為何C2叫做Server JIT Compiler！

## Day186
#### 學習重點 : Runtime Data Area
- 基本架構 ⭐⭐⭐⭐⭐
    - 前面說的JIT是存在於Execute engine區塊中，屬於JVM的其中一塊。接下來，我要來探究Runtime Data Area的部分！
    ![image](https://hackmd.io/_uploads/SkLcd1dQGl.png)
    - 所謂的Thread私有區域代表當Thread生成時一起出現，Thread消失時跟著不見。而共享就如同static一樣，是跟著JVM的生命週期存活！
- Program counter ⭐⭐⭐
    - 每個Thread在建立時，都有一個自己的計數器，處理下一條要執行的指令為何？
    - 由於這部分感覺有點接近計概了，就先淺淺帶過就好ww。
- JVM Stack ⭐⭐⭐⭐⭐⭐
    - JVM Stack在Thread被建立時，會同時建立一個Stack Frame專門記錄方法的執行過程，並儲存區域變數表、參數、return後的記憶體位置。
    - 舉個例子 : 在A函式呼叫B函式時，會建立Stack Frame推入B函式執行時的區域變數、參數，最終函示執行完畢後，pop出B函式，並回到A函式被中斷的位置繼續執行A函式的Stack Frame！

## Day187
#### 學習重點 : Runtime Data Area
- Native Method Area ⭐⭐⭐⭐
    - 儘管兩者都屬於Thread執行時去服務方法的流程記憶體堆疊，但其管理的是以「native」關鍵字的「原生method」。
- Thread 資源共享區域 ⭐⭐⭐⭐⭐⭐
    - JVM在執行時，會建立一個Heap的區域，與JVM開啟結束共進退，管理物件、資料的生成。
    - 由於Thread共享，因此就可能會有資源性問題出現，這也是為甚麼會有lock、synchronized出現！
    - GC : 也出現在Heap區域，由於Heap存放物件、資料，因此進行資料管理是十分重要的，優化記憶體也是靠著GC的技術去達成！
    #### Method Area
    - 在Java1.8後就變成了Metaspace，存放Constant、Static不變的資料，因此不歸GC管，像是JIT編譯後的機器碼也是存於此。
    - 其中的Runtime Constant Pool即是存取.class經ClassLoader載入的資料。

## Day188
#### 學習重點 : JNI是甚麼？
- Native？ ⭐⭐⭐⭐⭐⭐
    - 在認識Object class的時候，我發現像是hashCode等都會加上native關鍵字，亦即方法以介面呈現在Java中，實作則是以效率較快的C++去寫，故此方法被稱為原生語法。
    - 一般原生語法會使用loadLibrary去載入預先編譯好的C++檔(.dll、.so)。
    - 而native註冊又有「動態」、「靜態」之分。
- JNI ⭐⭐⭐⭐
    - 至於JNI則是作為 `C++` 與`Java` 的中間人，其將 `Java` 的資料型態、GC機制等轉換成 `C++` 的邏輯。
    - 而JNI也能夠反向轉換 --> Callback，將C++轉換成Java！

## Day189
#### 學習重點 : Void在Java中的用處？
- Void？ ⭐⭐⭐⭐
    - Void object屬於Java.lang，當我們希望一件任務被執行後，不回傳任何物件，也就是null，此時我們會利用java.lang所定義的Void object，去表示一個null回傳的任務。
    - 而在Void object定義中，我們的constructor會是private的，主要是不希望Void被實例化，而失去了其null的特性！

## Day190
#### 學習重點 : Void跟null的關係？
- Void代表null？ ⭐⭐⭐⭐
    - 在一般情況中，null值被回傳是空的概念，若我們本來就不須回傳或者只單純執行任務的method設定Generics為Void，Object為Void，而不是單純的值為空。
    - 以上設定才能讓泛型的邏輯成立！

## Day191
#### 學習重點 : Callable搭配Void
- 常見例子
    - 在執行緒當中，我們有時候會搭配Void object來完成一些task。
    ```java=
    import java.util.concurrent.Callable;

    public class VoidT implements Callable<Void> {
        @Override
        public Void call() throws Exception {
            System.out.println("執行中...");
            return null;
        }
    }
    ```
    - 我們知道Void object可以有null Reference，且不能被實例化，因此唯一能回傳的就是null，但其指向的形態還是Void物件。

## Day192
#### 學習重點 : Comparable
- 甚麼是Comparable？ ⭐⭐⭐
    - 在集合框架中，我們時常會對物件進行排序，但排序的規則為何？
    - 這時我們就會實作Comparable介面來定義sort時，是按照什麼樣的邏輯下去做的！
- Comparable實作 ⭐⭐⭐⭐
    - 我們會覆寫一個叫做compareTo的method。
    ```java=
    public class Person implements Comparable<Person> {
        private String name;
        private int age;

        public Person(String name, int age) {
            this.name = name;
            this.age = age;
        }

        @Override
        public int compareTo(Person other) {
            return this.age - other.age;  // 依年齡排序
        }
    }
    ```
    - 會發現其回傳的是int，也就是根據正數負數決定是否要交換兩者順序，按照**自然排序** --> 小的往前排，大的往後。
    - 因此若this大於other則this往後，反之則往前。

## Day193
#### 學習重點 : Comparator
- 甚麼是Comparator？ ⭐⭐⭐⭐
    - 針對一個類別，像是Person，通常會實作compareTo，因此我們可以利用Collection的sort來比較。
    - 但如果我們不想使用類別原生的自然排序呢？
    - 這時我們可以按照我們自己的邏輯來實作比較器！而這個可以讓我們自己設計比較邏輯的工具就是Comparator。
- Comparator怎麼被運用？ ⭐⭐⭐⭐⭐⭐⭐
    - 在使用 `List<String> list` 存放字串時，String內建的compareTo方法是按照ASCII字典順序來排的。
    - 但當我們今天想要將list依照 **字串長度** 來排序呢？
    - 這時就可以使用Comparator介面！
    ```java=
    public static void main(String args[]){
        List<String> list = new ArrayList<>();
        list.sort((s1, s2) -> s1.length() - s2.length())
    }
    ```
    - 上述的 `(s1, s2)...` 就是在實作Comparator介面！

## Day194
#### 學習重點 : Comparator工具
- comparing ⭐⭐⭐⭐
    - 簡單來說，我們可以打包一個排序邏輯工具物件，以Comparator.comparing來生成一個排序物件，可以放進Collections的sort功能中！
    ![image](https://hackmd.io/_uploads/BkKmJdG4zx.png)
- naturalOrder ⭐⭐
    - 當然，Comparator也有naturalOrder的排序方法
    ![image](https://hackmd.io/_uploads/H1fE-dfEMg.png)

## Day195
#### 學習重點 : Collections的功能
- Collections的sort ⭐⭐⭐⭐⭐
    - 在集合框架中，可以使用Collections的sort，其中可以放入單純list（有實作過Comparable介面），或者list搭配comparator的比較器！
    - 像是Person自己有設計過compareTo，但我還是可以塞比較器！
    ![image](https://hackmd.io/_uploads/H1B46oQVfg.png)
    ![image](https://hackmd.io/_uploads/B1Q_6iX4Ge.png)
    - 我順便覆寫了toString讓輸出比較好看！

## Day196
#### 學習重點 : Deque
- 甚麼是Deque？ ⭐⭐⭐⭐⭐
    - 別於一般的queue，deque指的是雙向佇列，他擁有了FIFO及LIFO的特性，亦即push、pop都可以作用於front跟tail。
    #### 實際例子？
    - 一般常用的ctrl+z回溯功能，都是使用stack概念，push操作，pop出top形成回溯，但若一直操作而不回溯，stack就會overflow！
    - 此時若採用deque的概念，一樣push操作，但當操作堆疊達到一定數量後，pop掉tail的操作，此時結合stack可以pop出top(或者說tail)以及queue可以pop出front(或者說bottom)的功能達到最佳回溯功能！
- Java中實際使用deque ⭐⭐⭐⭐⭐⭐⭐
    - Java官方明確表示 : `A more complete and consistent set of LIFO stack operations is provided by the Deque interface and its implementations, which should be used in preference to this class(Stack).`，這也是因為Stack繼承了Vector，使得其Stack嚴謹度降低，且Vector是Thread-safe，亦即在單一執行緒下執行vector會有無用的鎖在拉低效能！
    - 而deque是一個介面，開發者可以選擇無鎖的ArrayDeque或者有鎖的LinkedBlockingDeque，這樣的兼容性使得deque優於stack！
    - 而實際的實作如下 : 
    ![image](https://hackmd.io/_uploads/By8qlxrEzx.png)

## Day197
#### 學習重點 : Deque的結構
- 深入Deque ⭐⭐⭐⭐
    - 昨天只有稍微帶到Deque的意義以及使用方法，今天則是來看看Deque在Java的繼承&實作架構！
    ![image](https://hackmd.io/_uploads/ByWrC88NMe.png)
    - 簡單來說，Deque建立在Queue之下，只是其內部在多定義了addFirst(針對front進行push)、pollLast(針對tail進行poll)，這種只有雙向Queue才會出現的功能！
    - 而在實作當中，則是由ArrayDeque與LinkedList為主。
    ![image](https://hackmd.io/_uploads/B1buGwUVfe.png)
- Deque的特色 ⭐⭐⭐⭐
    - ArrayDeque實作不接受null元素，因此若在ArrayDeque當中放入null會拋出NPE。但在LinkedList當中是可以的！但官方還是不建議在Deque中加入null。
    - 再來，其Array長度是可伸縮的，因此沒有固定大小。
    - 最後，Deque與Vector不同在於其不是thread-safe，因此不允許多執行緒的同時取用，也因此ArrayDeque這種實作只適用於單執行緒！

## Day198
#### 學習重點 : Deque作為Stack的應用
- Stack應用 ⭐⭐⭐⭐⭐⭐⭐
    - 在Java官方的建議中，通常會使用Deque作為代替Stack的選項，因此我找了個十分常見的範例 --> 瀏覽歷史紀錄，來做為Deque(Stack)的一種實作途徑。
    - 首先是定義好三個功能 : 訪問、上一頁、下一頁，上一頁與下一頁是一條路，而訪問代表按照過去的路往前開闢另一條路，因此當使用訪問時，下一頁的stack會被清空。
    - 以下是我的實作 : 
    ![image](https://hackmd.io/_uploads/H1hyDcDVGl.png)
    - 然後我寫了個簡單的測試 : 
    ![image](https://hackmd.io/_uploads/Bk6mt9wEze.png)
    - 透過上述實作可以達到以下成果 : 
    ![image](https://hackmd.io/_uploads/r1YDK5P4Ml.png)

## Day199
#### 學習重點 : String家族
- String ⭐⭐⭐⭐
    - 我們知道，String是immutable，因此翻開String類別，可以發現是以 `private final byte[] value;` 宣告的(Java8以前是char[])。
    - 而StringBuilder以及StringBuffer則是移除了 `final`，並加入了append、insert等修改字串的功能！
    #### 為何要修改字串？
    - 當我們在做一般字串拼接時、`substring` 等動作時，其實String會直接new一個新的String出來，而原本的字串則被遺棄在Heap區等待GC回收！十分浪費記憶體！
    - 如下方程式碼 : 
    ```java=
    for (int i = 0; i < 1000; i += 1) {
        str = str + arr[i] + ',';
    //  上方在String類別內部會改成下面這個
    //  str = new StringBuilder()
    //            .append(str)
    //            .append(arr[i])
    //            .append(",")
    //            .toString();
    }
    ```
    - 上述展示了當拼接1000次，會產出 `1000個StringBuilder + 1000個String臨時物件`，因此當我們要修改字串時，應該使用Builder或Buffer會更好一些！

## Day200
#### 學習重點 : StringBuilder vs. StringBuffer
- StringBuilder vs. StringBuffer ⭐⭐⭐
    - 一般來說，Builder跟Buffer都是實作了charSequence作為存取方式，但前者為 `non thread-safe`，意即沒有加上synchronized鎖住執行緒，也因此StringBuilder適合用於單執行緒。
    - 在JDK1.5之後，Java引入了StringBuilder，由於StringBuilder不像Buffer那樣有鎖，因此執行效率快了很多。
- String使用 `+` 做拼接 ⭐⭐⭐⭐⭐
    - 其實它是一種 **語法糖**，當我們針對兩個String進行拼接時，會以  `.append` 作為拼接方法（當然JDK1.5以前是 `StringBuffer.append`，之後是 `StringBuilder.append`！
    - 因此如果我們要做多次拼接，直接使用StringBuilder物件.append修改底層記憶體，比起new一堆垃圾物件還來的好！
- StringBuffer的toStringCache ⭐⭐⭐⭐⭐⭐⭐
    - 在StringBuffer物件中，若多次呼叫 `.toString`，Buffer會在第一次呼叫後將其存入toStringCache成員當中(它是String物件)，以便多個執行緒要讀取Buffer時，不用一直toString（複製陣列、new一個String物件...。
    - 這樣設計的原因也是因為 --> 通常執行緒「讀取頻率大於修改頻率」，因此設計Cache可以讓讀取更加快速，而又因較常讀取，因此Cache才有存在的意義！
    - 當然，若某一執行緒修改了Buffer，則Cache被清空，待下一次toString時賦予新的String！
    
## Day201
#### 學習重點 : Method chain - Builder pattern 
- Builder是甚麼？ ⭐⭐⭐⭐⭐⭐⭐⭐
    - Builder是一種設計模式，由於一般我們在建立物件時，若成員過多，會在建構式當中丟入太多參數，造成冗長的初始化。
    - 建構式多載？ --> 儘管我們利用了建構式多載來設計各種初始化參數接收，但在參數順序上，還是得依序填入，缺少「選擇性」，且過於「制式化」。
    - JavaBean？ --> 這也是一種設計方式，利用setter來設計一個物件，但這就使得物件的immutable性質消失了！
    - Builder的出現！ --> 為了保持選擇性 + 不可變，我們在A類別內部加入一個 `static class Builder`，成員與A類別相同，並在其中設計每個成員配一個函式去賦值，同時A類別的建構式中接收Builder物件，最後設計build函式呼叫A類別的建構式。
    - 而Builder的設計模式在初始化時，就是method chain的概念，最後使用build回傳物件！
- 實際的長相 : ⭐⭐⭐⭐
    - 我以Person作為模型加入了Builder的設計模式並用其建構出一個簡單的Person物件！
    ![image](https://hackmd.io/_uploads/ryYv0isVMl.png)
    - 而Builder內部如下 : 
    ![image](https://hackmd.io/_uploads/SyFoAijNMl.png)
    - 透過以上的實作也讓我釐清了在SpringBoot當中一直使用到的build跟body等功能是如何被建立的啦！

## Day202
#### 學習重點 : lombok的Builder
- Builder註解 ⭐⭐⭐⭐
    - lombok中也有設計一個註解叫做 `@Builder`，直接讓class免去寫Builder的架構，只需要在class上方加註即可！
    - 如下 : 
    ![image](https://hackmd.io/_uploads/SJoe-thEMg.png)
    - 而使用起來與我自己實作的一模一樣！
- @Builder.Default ⭐⭐⭐⭐
    - 在設計成員時，我希望若某些成員在build時沒有傳入值，則使用給定的預設值。
    - 由於我們是先設定Builder，最後在以.build一一對應到類別建構式當中。假設沒有在method chain設定a值，即便我們直接在類別當中指派5給a，最後還是會被建構式當中的 `this.a = builder.a` 覆蓋掉。
    - 舉個例子 : 
    ![image](https://hackmd.io/_uploads/Hye8dVt24Mx.png)
    ![image](https://hackmd.io/_uploads/SJcF4Kh4Mx.png)
    ![image](https://hackmd.io/_uploads/SkhKEY2NMl.png)
    - 在沒有傳入age的情況下，最後在建構時，會使用int的預設 `0` 傳入。
    - 為了設計預設值，Builder有一個功能是default，只需要在field上加入該註解即可。
    ![image](https://hackmd.io/_uploads/SJp1St2VMl.png)
    - 以上，即可解決預設的問題！
 
## Day203
#### 學習重點 : Builder的問題
- 關於null的問題 ⭐⭐⭐⭐⭐⭐⭐⭐⭐
    - 昨天練習預設值時，若沒有在method chain放入age時，會預設0或者按照Builder.Default來初始。
    - 但當我建立person物件時，傳入age是null，並不會觸發預設值，此時我們需要有相對應策略來應對null問題。
    - 直接設計一個偵測null給預設值？ --> 我們可以這樣做 : 
    ```java=
    int age = 5;
    if (person.getAge() != null){
        age = person.getAge();
    }
    ```
    - 取出person後再做一道防線，若null則給5，就跟昨天沒設定一樣的概念！
    - **但是**，這就違反了物件導向設計的精神 --> 物件本身的 **invariant**(不變量)，意即 `物件age若null要給5` 這件事應該封裝在Person當中，不應該交由外部再做處理！這也是DTO跟model之間的一個區別，當DTO到model之間應該要先做好null check，使得 **model有完整的invariant特性**！
- 解決
    - 我們可以在Person當中設計特殊getter，當要取出null的成員時，丟出預設值 :
    ![image](https://hackmd.io/_uploads/SJ8P1B0NGl.png)
    - 以上是其中一種改良方法(直接在model中設計邏輯)！
      
## Day204
#### 學習重點 : Builder的null check
- final的意義 ⭐⭐⭐⭐
    - 在設計物件時，我們有時會讓成員加上final修飾詞，使得物件的狀態可以被相信(決不被竄改)。
    - 當遇到Builder的null問題時，其實我們除了修改getter這個方法之外，也可以在Builder當中預設age，使得最後傳到Person建構式時，即便沒有呼叫過age就build，age也還是有值！
    ![image](https://hackmd.io/_uploads/SyvCiPyrMe.png)

## Day205
#### 學習重點 : Jakarta EE
- 何謂Jakarta EE？ ⭐⭐⭐⭐⭐⭐
    - Jakarta EE是Java SE的擴充版本，屬於企業級版本，其與SE不同的點在於 --> 其制定了一套網頁API，從資料庫到網頁溝通都有一套標準，但其為 **介面規範**，因此有許多不同的實作工具。
    - 而其中有兩個元素很重要 : Web、EJB。
    - Web包含兩個核心 : JSP、Servlet，前者是內嵌Java程式的HTML，後者則是負責**HTTP請求**。
    - 而EJB（Enterprise Java Bean）與Spring Bean其實是一個概念，但Spring將其設計的很方便使用，相較起來EJB就顯得複雜！
- Web container
    - 所謂的網頁容器就是指「**執行Java Web程式的平台**」，而常見的容器像是Tomcat、Jetty等。
    - 當我們撰寫完Java網頁程式後，打包成 `.war` 檔，並放置到Container中解壓縮即可！
    - 小知識 : Spring Boot是直接內嵌Tomcat並打包成可執行的 `.jar` 檔！
- MVC架構 ⭐⭐⭐
    - 還記得在N天前，我曾經有嘗試理解MVC架構，但似乎缺乏實作導致觀念混在一起，但現在我有一些實作經驗，想藉著理解Java EE的網頁原理再次釐清！
    - 首先是關於Web的部分，其下的JSP與Servlet可以對應到View與Controller，而Model呢？則是對應到Java Bean。
    - Servlet作為Controller，亦即針對Request、Response做處理與轉發，而其中一種轉發就是 **轉發至JSP去處理前端的呈現方式**。

## Day206
#### 學習重點 : Session與Cookie
- Session探究 ⭐⭐⭐⭐
    - Session是一段時間內的「**狀態**」，一般的Http是不會存取狀態的，因此我們需要以Session來確保狀態的「**持續**」。
    - 以客戶端來看，打開一個分頁可以說是Session的開始，刪除分頁Session就消失。
- Session與Cookie？ ⭐⭐⭐⭐
    - 在資料傳遞的過程當中，當我們發起Request至伺服端，伺服端並不會記得我們是誰，因此我們會透過Session來記起使用者。
    - Session就像JWT token一樣，在傳統Web領域中，伺服端以Response Header方式於客戶端設定其domain的cookie，並存放Session id，同時Web container後端會空一塊記憶體存取session id及資訊。
    - 因此Session依賴於cookie，使用者後續動作都是根據cookie中的Session id在辨別身分！
    - 而Session在伺服端也是有時效性，通常會設置Session Timeout來讓資料不持續占用記憶體。

## Day207
#### 學習重點 : Tomcat環境建置
- Tomcat建置 ⭐⭐⭐⭐⭐
    - 為了防止未來的我癡呆，這邊紀錄一下基本安裝要素ww。
    - 如同之前的Spring Boot，Tomcat也需要有其執行的環境，因此我們需要先去apache下載Tomcat的執行環境 --> [點我前往Apache Tomcat®](https://tomcat.apache.org/)
    - 待解壓縮後，到IDEA去設置Configurations
    ![image](https://hackmd.io/_uploads/S1aUS9QHfg.png)
    - 新增設置後，需要加入Deployments，使得網頁專案被部屬上去
    ![image](https://hackmd.io/_uploads/SkaWU57rMg.png)
    - 同時，也要在Project Structure中的Module、Facets新增Web
    ![image](https://hackmd.io/_uploads/rkKsIqQrfg.png)
    - Module中引入Web就如同我在dependency引入套件一樣，而Facets則是代表Web專案的架構及打包的設定。
- 測試 ⭐
    - 在web資料夾下建置一個index.jsp並執行，看看是否有成功部屬！
    ![image](https://hackmd.io/_uploads/B18rd5QSGl.png)
    - 成功！
    ![image](https://hackmd.io/_uploads/SkoPO5QHMe.png)

## Day208
#### 學習重點 : Tomcat Web專案打包與執行
- Tomcat Web專案打包 ⭐⭐⭐⭐⭐
    - 為了打包成.war檔，我們需要在Project Structure中設置artifact -> 選擇Type:archive --> 接著選擇我們要打包的原始專案(exploded)。
    ![image](https://hackmd.io/_uploads/H1HKwkSBfx.png)
    - 接著在Build中選擇archive的artifact。
    ![image](https://hackmd.io/_uploads/SJaM_1BSGl.png)
- Tomcat Server啟動
    - 在tomcat的資料夾中，有個webapps的資料夾，內部裝的是Web專案，而我們的.war檔即是放在其中。而藉由 `tomcat/bin` 下的startup開啟Tomcat server後，會自動解壓縮.war檔案成為Web專案。
    - 啟動！ : 開啟 **終端機** 後進入bin目錄，並輸入 `./startup.bat` 來啟動Tomcat server。
    - 若遇到 `Neither the JAVA_HOME nor the JRE_HOME environment variable is defined At least one of these environment variable is needed to run this program`，這種訊息，則需要去環境變數新增JDK的路徑 --> ![image](https://hackmd.io/_uploads/rk5anJSrMl.png)
- Web.xml ⭐⭐
    - 在這個檔案中，有個很重要的部分 --> servlet-mapping : 以name連結classes跟url-pattern，意即當我輸入網址時會導向哪個java程式！

## Day209
#### 學習重點 : JSP結合java語法
- JSP內嵌Java語法 ⭐⭐⭐
    - 在JSP檔中，我們可以利用 `<% %>` 內部放入Java語法來實現動態的語法邏輯，範例如下 : 
    ```html=
    </head>
    <body>
    <%
        String name = "小八";
        out.println("Hello!" + name);
    %>
    </body>
    </html>
    ```
    - 可以看到我只寫了out，並沒有加入System，這是因為jsp有個隱含元素功能可以省略。 
- JSP轉換至Servlet ⭐⭐⭐
    - 在執行JSP的程式碼時，其實它會先被轉換成Servlet，以Servlet組織包裝過後再輸出HTML檔至前端，這也就是為何JSP是動態的網頁。
    - 像是out在jsp經轉換後會變成PrintWriter的物件，並輸出HTML程式。

## Day210
#### 學習重點 : JSP - Implicit object
- 何謂Implicit object？ ⭐⭐⭐⭐
    - 在JSP當中，有一些元素時常被使用到，像是輸出out、請求request...。
    - 而這些元素早已被Tomcat宣告初始化過，因此我們只需要使用名稱，而不需再自行宣告物件。
    - 常見像是 `HttpServletRequest`、`HttpSession` 等等。
- Implicit object簡單實作 ⭐⭐⭐⭐
    - 我跟著教學簡單設計了一個form的框框可以填入數字，並submit後加一！
    ```html=
    <body>
    <%
        String num = request.getParameter("number");
        Integer i = 0;
        if (num != null){
            i = Integer.valueOf(num);
            i++;
        }
    %>
    <form action="/Tomc_Web_exploded/index.jsp" method="post">
        <input type="text" name="number" value="<%=i%>"/>
        <input type="submit" value="Submit"/>
    </form>
    </body>
    ```
    ![image](https://hackmd.io/_uploads/rJ94PdwSMg.png)

## Day211
#### 學習重點 : JSP - Directives
- 指示元素是甚麼？ ⭐⭐⭐⭐⭐
    - 回想一下，JSP是由HTML與Java組成的，當JSP被轉換至Servlet時，需要有一些設定，此時就須靠著指示元素來設定JSP的轉換機制。
    - 而指示元素有三個 : `page、include、taglib`。
- page ⭐⭐⭐
    - 用於指示Tomcat應該如何處理這個JSP檔，因此會包含像是編碼、Java類別引入、contentType等等。
    - 通常會將其放在 `<html>` 之外，作為設定功能，將其放在檔案頂部。
- include ⭐⭐⭐
    - include指示詞在JSP中有分為靜態引入、動態引入，現在要學的是靜態引入，而引入表示copy其他file的jsp檔，合併到目前的檔案。
    - 由於靜態引入是在編譯時期完成的，因此變數、HTML等都會複製過去。
- 實作 `page + include` ⭐⭐⭐⭐
    - 透過基本指示詞語法 : `<%@ directives attribute="value" %>`，來完成。
    ![image](https://hackmd.io/_uploads/Bk5_TT_Szg.png)
    ![image](https://hackmd.io/_uploads/HJXFa6OSzg.png)
    - 實際成果！
    ![image](https://hackmd.io/_uploads/Hkccaa_Sfg.png)

## Day212
#### 學習重點 : JSP - Action Elements
- 甚麼是Action Elements？ ⭐⭐⭐⭐
    - 昨天看的是指示元素 --> 指的是轉成Servlet的規格書。
    - 然而Action Elements如同其名稱一樣，是在 **執行期間動作**的。
    - 它的出現可以大幅減少Java內嵌在JSP當中，使得後續維護更方便。
    - 其型式如右 --> `<jsp:{elements} {attribute key} = {attribute value} />`
    - 而常見的elements有 --> include、useBean...。
- include ⭐⭐⭐⭐
    - 在action elements當中，include就是 **動態引入**，因此相較於Directives是合併jsp檔後編譯執行，它是分開編譯後再做執行。
    - 這種動態引入有很多好處 --> 根據使用者點哪個按鈕決定要導向(引入)哪個jsp、重新編譯檔案時不會牽動到其他jsp...。
    - 當然，如果是引入固定不動的JSP檔案，如:css、模板格式等，可以用Directives即可。
- useBean ⭐⭐⭐⭐⭐
    - 在action elements當中，可以搭配表單輸入 & Bean來設定model。
    - 這邊的Bean泛指POJO(Plain Old Java Object)，且實作Serializalbe的類別。
    - 而useBean有三個要素 : id、class、scope，意即mapping、Bean檔、Bean的生命週期與存活範圍。
- 以下是簡單實作 : ⭐⭐⭐
    - 利用useBean + Property搭配表單來設定UserBean。
    ![image](https://hackmd.io/_uploads/HJVHXMcBfg.png)
    ![image](https://hackmd.io/_uploads/HJdSQzqSzg.png)
    - 以下是成果展示！
    ![image](https://hackmd.io/_uploads/rkRw7f5Bfg.png)

## Day213
#### 學習重點 : JSTL標準標籤使用與客製化
- 甚麼是JSTL？ ⭐⭐⭐
    - JSP Standard Tag Library，而不像Directives與Action Elements那樣直接使用，JSTL需要被引入！
    - 因此需要先去pom.xml加入jakarta.servlet的jstl dependency。
- JSTL的意義與使用 ⭐⭐⭐⭐
    - JSTL與Action Elements都是為了減少Java程式碼內嵌於JSP的情況。
    - 而相較Action Elements專注在jsp檔的流轉; JSTL則是補足了流程控制、集合等Java的功能，但是是以「**標籤**」來寫。
    #### 與taglib的合作
    - 在Directives中還有一個元素前天沒有介紹到 --> taglib。
    - 而它就是為了JSTL而生的東西 --> 設置函式庫前輟。
    - 以下是範例 : 
    ![image](https://hackmd.io/_uploads/r1rMbwoBGe.png)
    - 透過設定前輟，可以讓JSP知道當我們使用 `c:...` 時，會自動找到tags下的core功能。
    - 而core提供了包含if、forEach等語法。
- tags自製結合JSTL實作 ⭐⭐
    - 除了使用官方提供的JSTL之外，我們也可以自製tags，而其中的原理我就不深究了ww。
    - 以下是整個結合後的成果 : 
    ![image](https://hackmd.io/_uploads/ryAYfPsrzx.png)
    ![image](https://hackmd.io/_uploads/BJbiGDsBGg.png)

## Day214
#### 學習重點 : Servlet生命週期、結合JSP實作MVC架構
- 自行設計Servlet（**Controller**） ⭐⭐⭐⭐
    - 前幾天都是在jsp中寫java及內嵌標籤，今天則獨立出來自行設計Servlet。
    - 而基本的Servlet架構如下 : 
    ![image](https://hackmd.io/_uploads/SJ6eYt2HGg.png)
    - 注意這邊的url設定可以利用WebServlet完成，但也可以在web.xml做mapping，**兩者擇其一** 即可！
    - 在HttpServlet當中，覆寫一般常用的兩個動作，**接收request、response作為與前端溝通的橋梁**。
    - 而 `HttpServlet` 的父類別是 `GenericServlet`，其中管理了Servlet的出生到死亡，而我們可以透過init、destroy函式來看看Servlet是何時誕生與關閉的！（**這也是IoC範疇**）
    ![image](https://hackmd.io/_uploads/ry9T5FhBGg.png)
- 結合 **Model** ⭐⭐⭐⭐⭐⭐⭐
    - 我跟著教學實作了一下Service，以空氣汙染為題，自行設置一個 `AirService` 處理資料處理的部分，同時設定一個Report類別作為Entity存取資料。
    - 接著我在加入 `AirServlet`，並在doGet的部分設定request設置Attribute取得AirService處理Report後的空汙縣市資料集合。
    ![image](https://hackmd.io/_uploads/BkLZ6FhSMl.png)
- **View** 的接收 ⭐⭐
    - 接著利用AirJsp搭配CSS模板(我請AI幫我生成的w)，並藉由core的forEach功能一一取出request的空汙集合，並呈現於Table上！
    ![image](https://hackmd.io/_uploads/SkJ5nY3HGx.png)
- 成果展示！ ⭐⭐
    - 由於我沒有串接API，所以先自行設計幾筆Report，看起來有模有樣就好ww。
    ![image](https://hackmd.io/_uploads/B1w03K3Sfe.png)

## Day215
#### 學習重點 : Spring 使用者資料管控專案Part-3 TODO list
- 前言 ⭐
    - 這應該是今年我的小專案最後一partㄌ，我打算後面的時間拿來複習 + 資料庫的探索。另外我其實蠻想深入了解Minecraft插件的w，以前有玩過，但一直沒機會完整研究過，我想找時間來深入探究。
    - 另外其實我最近有想要考考看Oracle的證照，所以可能會每個禮拜選個一天來看看Oracle的題目owo。
- 小專案Part-3 TODO list ⭐⭐
    - 0️⃣ Schedule排程（ScheduleExcutorService、Spring @schedule）
    - 1️⃣ Spring security

## Day216
#### 學習重點 : 關於原生Java的Schedule - 1
- 排程的重要性 ⭐⭐⭐
    - 在看Java原生排程之前，先來看看後端為何要有排程。
    - 1️⃣ 排程可以消化大量了耗時請求，以及重複的動作（Batch Processing），來保持主服務穩定。
    - 2️⃣ 排程可以 **定期** 清理垃圾資源，或者過期的Session、Token等等。而針對電商系統，更有金流對帳等資源上定期核對的需求。
- 排程的問題 ⭐⭐⭐
    - 在併發下，要如何確保排程不會因多台伺服器導致重複執行(如重複發送折價券)。
    - 是否會有多排程同時爭奪資源的問題。
    - 資源訪問 --> 其沒有JWT的限制，因此需要注意權限的分配。
    - 資料庫的問題 --> 資源保護及撈資料須以Pagination作處理。
- Timer vs. ScheduledExecutorService ⭐⭐⭐⭐
    - 在早期的Java中，Timer是主要的排程類別，但其有個硬傷 --> 單一執行緒。
    - 因此後來的JUC（java.util.concurrent）引入後，出現了ScheduledExecutorService（我後續簡稱SES好了w），解決了單一執行緒的問題。
    #### 兩者的詳細比較
    - Timer由於只有 **單一執行緒**，因此處理排程任務會堆在一起。然而SES可以用 **ThreadPool** 作配置，因此可以分配多執行緒來處理多排程。
    - Timer無法處理Uncaught Exception，一旦排程出問題，整個執行緒就bye bye。然而SES的ThreadPool可以捕捉異常，其他排程可繼續工作。
    - Timer取消任務的機制十分笨重，且不支援任務的回傳。然而SES支援 **Callable**，且排程啟動後的回傳物件即為 **ScheduleFuture**，因此可以利用 `.cancel`、`.get` 來取消、取得任務結果。

## Day217
#### 學習重點 : 關於原生Java的Schedule - 2
- 深入看ScheduleExcutorService（我簡稱SES） ⭐⭐⭐⭐
    - 打開SES的原始碼，可以看到它是繼承ExcutorService，因此可以知道當我新建一個排程時，任務一樣是被丟到ThreadPool去執行，這也是昨天我比較Timer的一大重點。
    - 而排程的特性有幾個 : **Period、Delay、Future**。
    - 不同Period的任務可能會 **分配至不同的Thread執行**。
- 怎麼建立一個ScheduleExcutorService？ ⭐⭐⭐
    - 首先我們給定 `corePoolSize` : 
    ![image](https://hackmd.io/_uploads/BkM-kpx8Ge.png)
    - 當然，也有其他參數建構式，但這邊就用最簡單的就好了w
- 任務完成後的回傳 : ScheduleFuture ⭐⭐⭐⭐⭐⭐
    - 當我們打開原始碼，可以發現每一個排程函式，都會回傳 `ScheduleFuture<>`，連Runnable排程函式都有，但我們都知道Runnable不會回傳，那為何需要Future？
        - 這是因為 ➝ ScheduleFuture取得物件後，可以使用 `.cancel` 來 **取消任務的排程**（當然這建立在任務還沒被執行啦w，若已經開始了，則會嘗試使用interrupt去觸發try-catch）。
    - 而在 `<>` 也分為 `<?>`（for Runabble，無回傳） 及 `<V>`（for Callable）。
- scheduleAtFixedRate vs. scheduleWithFixedDelay ⭐⭐⭐⭐
    - 關於scheduleAtFixedRate，其是週期性執行排程任務，然而當任務delay則，後續排程也會跟著delay（當前任務耗時大於Period，一結束前任務，後任務立刻執行），但一般情況下，其本質上就是 **定期做某事** 的函式。
    - 關於scheduleWithFixedDelay，其則是等前一個任務執行完畢後，根據設定的delay再多延遲幾秒，預留時間，最後才執行下一任務，確保 **兩任務絕對不重複**。
    - 以下是基本範例 : 
    ```java=
    schedule.scheduleWithFixedDelay(
        () -> {
            long seconds = System.currentTimeMillis() / 1000;
            System.out.println("Excuted at : " + seconds + " seconds");
            try{
                Thread.sleep(3000);
            }catch (InterruptedException e){
                System.out.print(e.getMessage());
            }
        },
        3, 3, TimeUnit.SECONDS
    ); // 意即每次執行任務間隔「至少」6秒
    ```

## Day218
#### 學習重點 : Schedule常用表達式Cron
- 甚麼是Cron？ ⭐⭐⭐
    - Cron（chronos），是排程的 **時間表達式**，用於設定週期性、間隔的時間。
    - 原本是類Unix作業系統的任務時間系統，但因後續沿用到了許多工具中，如Spring Boot的Schedule，也是本次我想攻略的部分！
    - 而我們以cron設定的排程任務，又稱為「cron jobs」。
- Cron的格式 ⭐⭐⭐⭐⭐⭐⭐⭐
    - 基本的格式 ➝ `(分 時 日 月 星期)`。
    - 如 : `(59 23 * * *)` 表示每天23時59分執行...。
    - 透過上面的範例可以知道 `*` 表示 `任意、所有` 的意思。
    - --
    | 符號 | 意義 |
    | --- | --- |
    | `*` | **所有** ➝ `(0 12 * * *)` ➝ 每天 12 點 |
    | `-` | **指定範圍** ➝ `(0-5 * * * *)` ➝ 每時 0 到 5 分 |
    | `,` | **指定數字** ➝ `(0 12,15,18 * * *)` ➝ 每天 12 點、15 點、18 點 |
    | `/` | **每（步長）** ➝ `(10/5 * * * *)` ➝ 每時的 10 分開始每 5 分鐘執行一次 |
    | `?` | **不指定** ➝ Spring 為解決「日/星期」衝突問題（僅限日與星期） |
- Linux及Spring的cron差異 ⭐⭐⭐⭐
    - Linux採用基本的5個單位格式（上述的格式），而Spring則採用6個單位格式 ➝ `(秒 分 時 日 月 星期)`。
    #### `?` 的意義
    - 當我寫下 `(* 12 15 * *)` (Spring記得多加一個秒單位)。
        - Linux ➝ 是每月15號（不管星期幾）的12點。
        - Spring ➝「星期日(0)到星期六(6)」 & 「15號」的12點（邏輯衝突），此時會報錯。
        - 這是因為Linux的 `*` 是任意。而Spring的 `*` 是所有。
    - Spring的 `?`
        - 因此Spring加了 `?` 來針對日、星期表達「忽略、不管」的概念，因此當我們在寫Spring的cron格式時，需注意 「日、星期」**必須** 有一個是 `?`。

## Day219
#### 學習重點 : Spring Boot的@Schedule架構與Unit Test
- Schedule類別的架構 ⭐⭐⭐⭐⭐⭐
    - 首先，先到Spring Boot專案當中加入一個排程類別，我是在我的電商專案的 `task/` 中加入ScheduleTask。
    - 由於排程也需要被丟進Spring容器當中，因此需要以 `@Component` 將其設為Spring Bean。
    - 在類別內，我們建立排程函式，我這邊參考[Spring官方的範例](https://spring.io/guides/gs/scheduling-tasks)設一個定期回傳時間的函式。
    - 以下是完整架構 : 
    ```java=
    @Component
    public class ScheduleTask{
        
        private static final Logger log = 
            LoggerFactory.getLogger(ScheduleTask.class);

        private static final SimpleDateFormat dateFormat =
            new SimpleDateFormat("HH:mm:ss");
        
        @Scheduled(fixedRate = 5000)
        public void reportCurrentTime(){
            log.info(
                "The time is now {}",
                dateFormat.format(new Date())
            );
        } 
    }
    ```
    - 最後，務必在Spring的啟動類別（也就是 `SpringApplication.run` 的類別）加上 `@EnableScheduling`。
- 關於 `@Scheduled` 的參數 ⭐⭐⭐
    - 在Scheduled的註解中，通常會寫三種時間格式 ➝ `fixedRate`、`fixedDelay`、`cron`，而這也是我在Java原生Schedule有看到的東西～
- Unit Test ⭐⭐⭐⭐⭐⭐⭐
    - 我們可以在SpringBootTest類別中測試排程是否有正確啟動、按照時間呼叫函式。
    - 而一般我們利用assertion會針對如 : `某段週期是否執行至少n次`、`是否正確更新資料庫`...。
    - 而這需要用到 `Awaitility` 這個工具，使用 **polling機制**，主執行緒去polling背景執行排程的非同步執行緒，確認assertion是否成功，一旦成功則pass。
    #### Awaitiliy vs. Thread
    - 傳統上可能會使用Thread.sleep使主執行緒等待 **特定時間**，待時間結束再確認背景排程的執行狀況。然而這可能因「主機不同」、「啟動時間長短」導致「單元測試時間」拉長，規模放大後則可能導致嚴重問題 ➝ 這種易受影響的Test又稱 **Flaky Test**。
    - 而Awaitility則是專門來 **測試非同步的工具**，他能夠在主執行緒「定時監控」，也就是polling～且 **不需要等待特定時間**，而是 **一旦成功則pass**。
    ```java=
    @SpringBootTest
    public class ScheduleTaskTest {

        @MockitoSpyBean
        public ScheduleTask scheduleTask;

        @Test
        public void reportCurrentTime(){
            Awaitility.await()
                    .atMost(Durations.TEN_SECONDS)
                    .untilAsserted(() -> {
                        verify(scheduleTask, atLeast(2))
                            .reportCurrentTime();
            });
        }
    ```

## Day220
#### 學習重點 : 關於Spring的TaskScheduler與TaskExecutor
- @Scheduled的追本溯源 ⭐⭐⭐⭐⭐
    - 當我們使用了 `@Scheduled` 來設定排程時，Spring內部其實偷偷將其管交給了 `TaskScheduler` 來運作（像是計時cron、觸發trigger）。
    - 而這個 `TaskScheduler` 是Spring設計用於排程的一個介面，而一般預設使用的實作類別是 `ThreadPoolTaskScheduler`，很熟悉吧～
    - 看到ThreadPool，自然而然會想到ExecutorService，而加上Schedule就變成了 ➝ `ScheduledExecutorService`！BANG！
    - 正如名字所示，ThreadPoolTaskScheduler內部的原理即是採用了Java原生的ScheduleExecutorService！
    - 那為何Spring不直接使用原生的就好了，還要自己包裝？主要有以下原因 : 
        - 1️⃣ **支援Cron時間表達式**。原生的排程只有Period、Delay，可沒有Cron！
        - 2️⃣ **生命週期的控管**。原生需要自行shutdown，然而Spring中，我們將其視為Bean，因此可以直接IoC！
        - 3️⃣ 對於異常更加靈敏，**自動包裝try-catch**，或者使用自訂ExceptionHandling。原生的儘管有應對方式，然而需自行寫try-catch，稍不注意則排程dead。
- TaskExecutor是甚麼？ ⭐⭐⭐⭐⭐
    - TaskExecutor是Spring自行設計的介面。而當然，它繼承自Executor。
    - 因此它的工作就是分配背景（非同步）執行緒，意即多執行緒！
        - 我們一般會以 `@Async` 作為使用TaskExecutor實作的一個方式（`.execute`）。這個我先不深入研究，先單就TaskExecutor與ThreadPoolTaskScheduler的關係為主。
    - 在ThreadPoolTaskScheduler中，除了實作TaskScheduler之外，還有實作了TaskExecutor，因此其 **具備了多排程的功能**！
    - 到這邊應該對TaskExecutor與ThreadPoolTaskScheduler的關聯有概念了，就讓我來實作看看吧！
- 實作多排程 ⭐⭐⭐⭐⭐
    - 首先先讓我們在ScheduleTask中設定兩個(以上)的排程，來看看default的設置。
    ```java=
    @Scheduled(fixedRate = 5000L)
    public void reportCurrentTime_1() {
        log.info("Schedule_1 reportTime : {}", dateFormat.format(new Date()));
    }

    @Scheduled(fixedRate = 2000L)
    public void reportCurrentTime_2() {
        log.info("Schedule_2 reportTime : {}", dateFormat.format(new Date()));
    }
    ```
    - log結果 : 
    ```
    2026-08-08T21:34:07.008+08:00  
        INFO 25876 --- [mall] [scheduling-1] org.system.task.ScheduleTask             : Schedule_1 reportTime : 21:34:07
    2026-08-08T21:34:07.009+08:00  
        INFO 25876 --- [mall] [scheduling-1] org.system.task.ScheduleTask             : Schedule_2 reportTime : 21:34:07
    ```
    - 若沒有特別設置排程的poolSize，預設就是1，因此排程之間有「排隊」的概念 ➝ 因此在毫秒上會有誤差(一個是.008，一個是.009)。
    - 此時我們在application.properties加入poolSize的設定
    ```properties=
    spring.task.scheduling.pool.size=2
    ```
    - 接著再執行一次，就會看到 : 
    ```
    2026-08-08T21:38:08.775+08:00  
        INFO 3212 --- [mall] [scheduling-2] org.system.task.ScheduleTask             : Schedule_1 reportTime : 21:38:08
    2026-08-08T21:38:08.775+08:00  
        INFO 3212 --- [mall] [scheduling-1] org.system.task.ScheduleTask             : Schedule_2 reportTime : 21:38:08
    ```
    - 兩者被分配到了不同的執行緒！時間上沒有影響了！（但要注意若同一任務排程之間有Delay則需要加上@Async，讓自己可以跟自己也達到非同步，但這是後續我的研究目標，這邊先打住。）
- 小小總結 : ⭐⭐⭐⭐⭐
    - 因此當我們在寫 `@Scheduled` 時，要注意執行緒的default是1，因此需要再注意多排程的設定！
    - 透過上述可以了解到 ➝ 
        - `TaskScheduler` 注重 **任務的觸發與時間**。
        - `TaskExecutor` 注重 **執行緒的調度與執行**！
        - `ThreadPoolTaskSchedule` 作為 `@Schedule` 的預設實作則 **綜合了兩者的概念**！

## Day221
#### 學習重點 : Schedule的Configuration與Bean
- SchedulerConfiguration ⭐⭐⭐⭐⭐
    - 昨天我是在 `application.properties` 中設定poolSize，這個其實是Spring提供的便捷方式。不過我們也可以 **自己寫一個配置檔**，**更精細化** 的設定參數。
    - 首先，我們會在專案下建立 `config/SchedulerConfiguration` 專門設定自己的排程。
    - 而在配置檔中，我們需要實作 `SchedulingConfigurer` 這個介面，並註冊我們設置好的TaskScheduler，這樣Spring才會接收並使用我們設置的TaskScheduler作為後續執行排程的元件。
    - 以下是範例 : 
    ```java=
    @EnableScheduling
    @Configuration
    public class SchedulerConfiguration implements SchedulingConfigurer {

        @Override
        public void configureTasks(ScheduledTaskRegistrar scheduledTaskRegistrar){
            ThreadPoolTaskScheduler threadPoolTaskScheduler = new ThreadPoolTaskScheduler();
            // 精細配置
            threadPoolTaskScheduler.setPoolSize(3);
            threadPoolTaskScheduler.setThreadNamePrefix("scheduling-task-");
            threadPoolTaskScheduler.initialize(); 
            // 註冊
            scheduledTaskRegistrar.setTaskScheduler(threadPoolTaskScheduler);
        }
    }
    ```
    - 我後來研究發現 `@EnableScheduling` **不一定** 要寫在 `SpringApplication.run` 的類別中，所以就搬到配置檔裡面ㄌ～
    - 在Spring啟動時，就會讀取我們設置的配置檔，並將我們設置的TaskScheduler **註冊** 至ScheduledTaskRegister，作為Spring元件（注意！TaskScheduler不是Bean，**只是供排程運作的一個元件**）
- 將TaskScheduler作為Bean ⭐⭐⭐⭐⭐⭐
    - 若我們想要在Service中注入TaskScheduler，來實現 「動態排程」➝ 意即不使用 `@Scheduled` 作為伺服器預設排程，而是根據使用者or函式呼叫時「動態」新增排程。
    - 我們可以將配置檔寫為以下的樣子 : 
    ```java=
    @EnableScheduling
    @Configuration
    public class SchedulerConfiguration implements SchedulingConfigurer {
        
        // ThreadPoolTaskScheduler作為Bean
        @Bean
        public ThreadPoolTaskScheduler taskScheduler(){
            ThreadPoolTaskScheduler threadPoolTaskScheduler = new ThreadPoolTaskScheduler();
            threadPoolTaskScheduler.setPoolSize(3);
            threadPoolTaskScheduler.setThreadNamePrefix("scheduling-task-");
            threadPoolTaskScheduler.initialize();
            return threadPoolTaskScheduler;
        }

        @Override
        public void configureTasks(ScheduledTaskRegistrar scheduledTaskRegistrar){
            scheduledTaskRegistrar.setTaskScheduler(taskScheduler());
        }
    }
    ```
    - 這樣，我們即可在Service中注入TaskScheduler啦～

## Day222
#### 學習重點 : Spring Schedule電商專案實作 - 定期處理未付款訂單.1
- OrderSchedule ⭐⭐⭐
    - 在 `task/` 目錄下，我新建了一個OrderSchedule，處理Order相關排程函式。
    - 而首先我要處理的是「**未付款訂單**」的事項 ➝ 當訂單呈現未付款狀態超過特定時長，我們需要將這些訂單取消，以 **避免庫存被占用**。
    - 因此我在OrderSchedule中建立了一個排程 `expiredOrderScheduling` 處理此問題。
- expiredPaymentScheduling函式處理 ⭐⭐⭐⭐⭐
    - 在函式處理之前，我先在OrderDao中加入一個Query : 
    ```java=
    // OrderDao.java
    @Query("select order from Order order WHERE order.status = 'UNPAID' AND order.create_time <= :expiredTime")
    List<Order> findAllExpiredUnpaidOrder(@Param("expiredTime") Instant expiredTime);
    ```
    - 這樣我們取出資料時就已經先過濾過一遍了，以此省略Task的工作。
    - 接著在排程中取出過期訂單的狀態，並修正為 `CANCELED`。
    ```java=
    @Transactional
    // 每分鐘polling一次
    @Scheduled(cron = "0 */1 * * * ?")
    public void expiredOrderScheduling(){
        // 這邊先以1分鐘就過期測試
        List<Order> orderList = orderDao.findAllExpiredUnpaidOrder(Instant.now().minus(Duration.ofMinutes(1)));
        if (orderList.isEmpty()) return;

        log.info("Found {} expired orders", orderList.size());
        int canceledCount = 0;
        for (Order order : orderList){
            if (order.getStatus() != Order.STATUS.UNPAID) continue;
            order.setStatus(Order.STATUS.CANCELED);
            canceledCount++;
        }
        orderDao.saveAll(orderList);
        log.info("Successfully canceled {} orders", canceledCount);
    }
    ```
    - 這邊再次過濾狀態，同時加上Transactional以 **保持資料一致性**。
    - 最後利用log紀錄取消訂單數量。
- 我遇到的問題與解決 ⭐⭐⭐⭐⭐⭐
    - 首先是關於排程 : 
        - 我原本的想法是 : 「當使用者下訂單後，新增動態排程，自訂單建立後經特定時長觸發 `CANCELED`」。
        - 問題在於 : 
            - **動態排程**十分吃資源，且不適用於這種整體金流事務的工作上。
            - 再來是**延遲**，新增排程後，此函式的交易就結束了，但背景執行緒還在等特定時段後處理訂單，一旦執行 `.save`，並不會rollback
            - 最後是**重啟**，一旦伺服器重啟，背景排程會消失，此時該訂單的「超時排程」就不見了，等於該訂單超時免疫了ww。
        - 綜合上述問題，我將排程修正為以cron設定定期抓資料，且分離出OrderService的payment過程，獨立出一條新的交易線（排程）。
- 未處理部分 ⭐⭐
    - 這邊留了幾個問題 : 
        - **庫存**的修正rollback。
        - 通知賣家與使用者。
        - Batch Update，批此處理未付款訂單。

## Day223
#### 學習重點 : Spring Schedule電商專案實作 - 定期處理未付款訂單.2
- @Transactional與@Scheduled並用問題 ⭐⭐⭐⭐⭐
    - 昨天，我將「**交易**」、「**排程**」的註解都加在了同一函式上，但這會造成一個問題 ➝ 排程占用 **交易資源連線過久**，使得其他功能 **無法取得該資源鎖**。
    - 為了解決這個問題，我將兩者處理的範圍拆開。將Schedule留在 `OrderSchedule`，而Transactional則丟到新建的一個 `OrderCancelService`。透過排程呼叫Service來處理CANCEL事宜。
- BATCH Update ⭐⭐⭐⭐⭐
    - 為了不要一次「交易」過多資源導致資源鎖占用，因此我們可以 **批次處理**，為此我設定了一個BATCH_SIZE來限制一次處理多少資源。
    ```java=
    private final int BATCH_SIZE = 10;
    
    @Scheduled(cron = "0 */1 * * * ?")
    public void expiredOrderScheduling(){

        List<Order> orderList = orderCancelService.getExpiredOrderList();
        if (orderList.isEmpty()) return;

        log.info("Found {} expired orders", orderList.size());
        int canceledCount = 0;
        int batch = 1;
        for (int i=0; i<orderList.size(); i+=BATCH_SIZE){
            List<Order> ordersubList = orderList.subList(i, Math.min(i+BATCH_SIZE, orderList.size()));
            // 藉由try-catch來防止某批次出問題而無法執行後續批次
            try {
                canceledCount += orderCancelService.expiredOrderScheduling(ordersubList);
                batch++;
            }catch (Exception e){
                log.info("Failed to cancel {} batch", batch);
            }
        }

        log.info("Successfully canceled {} orders", canceledCount);
    }
    ```
    - 利用for迴圈，並切 **割資源集合成子集**，分成BATCH_SIZE大小，再送進「交易」，這樣每次占用交易線的時間可以縮短許多～
- Join Fetch與庫存rollback ⭐⭐⭐⭐⭐⭐
    - 在處理庫存rollback之前，要先來理解何謂Join Fetch。
    #### 何謂Join Fetch？
    - 當我們在Dao層做Query取出orderlist時，若**沒有**將每個order的商品Entity實例化，會發生N+1問題，N+1這部分我不多寫了，之前有研究過了ww。
    - 因此我們需要在SQL語法中加入 `JOING FETCH order.merchandise`，也就是在Query過程，一併取出order本身關聯merchandise的實體並實例化。儘管我們可能設定Fetch Type是Lazy，但針對Merchandise這個實體則會 **預先載入**。
    - 完整的Query語法如下 : 
    ```sql=
    select order from Order order JOIN FETCH order.merchandise WHERE order.status = 'UNPAID' AND order.create_time <= :expiredTime
    ```
    #### 庫存的rollback
    - 有Order包著預載入Merchandise，我們就可以寫一個restock函式透過取得merchandise的stock，加上order本身的quantity，最後在回存至merchandise即可！
    ```java=
    // OrderCancelService.java
    
    // 此函式由schedule呼叫
    public List<Order> getExpiredOrderList(){
        return orderDao.findAllExpiredUnpaidOrder(Instant.now().minus(Duration.ofMinutes(1)));
    }
    
    // 批次處理交易，回傳成功交易筆數給schedule
    @Transactional
    public int expiredOrderScheduling(List<Order> orderList){

        int canceledCount = 0;
        List<Merchandise> merchandiseList = new ArrayList<>();
        for (Order order : orderList){
            if (order.getStatus() != Order.STATUS.UNPAID) continue;
            order.setStatus(Order.STATUS.CANCELED);
            merchandiseList.add(restock(order));
            canceledCount++;
        }
        orderDao.saveAll(orderList);
        merchandiseDao.saveAll(merchandiseList);
        return canceledCount;
    }
    
    // 補貨機制，由JOIN FETCH解決N+1問題，由交易函式呼叫
    private Merchandise restock(Order order){
        Merchandise merchandise = order.getMerchandise();
        merchandise.setStock(merchandise.getStock() + order.getQuantity());
        return merchandise;
    }
    ```

## Day224
#### 學習重點 : ShedLock與OptimisticLock
- OptimisticLock（樂觀鎖） ⭐⭐⭐⭐⭐⭐
    - 何謂樂觀鎖？其實就是在取得資源時，認為資料不常更新，樂觀認為不會有人改動資料，因此就不對該資源上鎖（而相反的，悲觀鎖即是在SELECT時就先上鎖了）。
        - 當我們在SELECT、saveAll時，都還是以JPA實體存在，直到函式區塊結束才會統一update並commit至DB佔用鎖。因此占用鎖的期間是「update到鎖釋放的階段」。
    - 樂觀鎖的原理在於針對某Entity加上一個欄位 `version`，當取得資源時，不上鎖，但在 **version欄位標記**，若最後commit上去時發現先看看version是否一致，若不一致，表示這期間有人動過資料，此時就不更新，反之則更新。
    #### 樂觀鎖的優缺點
    - 先說優點 : 
        - 樂觀鎖 **不會造成執行緒堵塞**（因為version衝突的資源一般不會retry，只有version正確才會待在執行緒commit）。
        - 適合讀取多，寫入少的時候，因此SELECT時不鎖，讓其他人也能讀取到資源。
    - 缺點 : 
        - 在高併發環境下，像是搶票，若大家同時訂票，只會有一個人的version是正確的，此時 **明明庫存還有票**，但 **卻無法下訂**。這時候就應該利用悲觀鎖的排隊機制！
- ShedLock ⭐⭐⭐⭐
    - 樂觀/悲觀鎖是針對單一資源，而ShedLock則是針對排程。
    - 其原理在於可以限制多排程下，只有一個排程可以執行，其餘會被擋下來直接return，而不執行業務邏輯，類似「**鎖住**」排程的感覺。
    - 一般會設置 `AtLeastFor`、`AtMostFor` 來限制這個排程鎖占用的最大時間跟最小時間。
    - 以下是簡單範例 : 
    ```java=
    @Scheduled(cron = "0 */1 * * * ?")
    @SchedulerLock(
        name = "expiredOrderSchedulingLock",
        lockAtMostFor = "50s",
        lockAtLeastFor = "10s"
    )
    public void expiredOrderScheduling(){
        // ...省略
    }
    ```
- 關於昨天程式修正 ⭐⭐⭐
    - 由於計算批次的batch完全可以被取代，因此我改成直接利用 `i`、`BATCH_SIZE` 來計算當前batch。
    - 另外，由於subList存的只是原本List的子集合（View），因此改動到subList也會影響原List。為了獨立subList，我們可以再new一塊空間把subList複製成獨立的List，這樣對獨立List增刪則不會影響原List！
    ```java=
    for (int i=0; i<orderList.size(); i+=BATCH_SIZE){
        List<Order> ordersubList = new ArrayList<>(orderList.subList(i, Math.min(i+BATCH_SIZE, orderList.size())));
        try {
            canceledCount += orderCancelService.expiredOrderScheduling(ordersubList);
        }catch (Exception e){
            log.error("Failed to cancel batch {}", (i / BATCH_SIZE)+1, e);
        }
    }
    ```

## Day225
#### 學習重點 : Spring Schedule電商專案實作 - 定期處理未付款訂單.3
- 樂觀鎖應用 ⭐⭐⭐⭐⭐⭐
    - 由於 **一般的Transactional事務是沒有上鎖的**，它只幫忙開啟交易，最後commit，若出錯則rollback。
    - 而昨天我探討了樂觀鎖的概念，今天就將它應用到「取消訂單事務」的情況，顯而易見，取消訂單這件事「不是高併發」，訂單資源只對一賣家、一買家、系統三者有關連，使用樂觀鎖是較正確的作法。
    - 而首先我先針對Order、Merchandise兩者的DB新增了version欄位 ➝ 
    ```sql=
    ALTER TABLE orders ADD version INT NOT NULL DEFAULT 0;
    ALTER TABLE merchandise ADD version INT NOT NULL DEFAULT 0;
    ```
    - 接著在Entity中加入 : 
    ```java=
    @Version
    @Column(name = "version")
    private Integer version;
    ```
    - 最後在原本取消訂單Schedule當中的try-catch加入 `OptimisticLockingFailureException` :
    ```java=
    catch (OptimisticLockingFailureException lockingFailureE){
        log.error(
            "Optimistic lock conflict detected in batch {}...", 
            (i / BATCH_SIZE)+1
        );
    ```
- ShedLock應用 ⭐⭐⭐⭐⭐⭐
    - 昨天只有在Schedule中設定參數，但沒有開啟ShedLock以及關聯表。
    #### 關聯表
    - 先看為何要有關聯表 ➝ 因為shedlock是針對「**多台主機對一排程**」，主機之間不知道誰先取得排程鎖，此時就得靠shedlock的關聯表做事。
    - 而關聯表需要有四個欄位 ➝ 鎖開始時間、鎖釋放時間、鎖的名稱、鎖的主機擁有者。
    - 當排程執行時，每台主機會利用我設定的name作為Primary Key嘗試寫入，**由於name的唯一性**，因此只有一台主機成功寫入，其餘則會報錯 `DuplicateKeyException`，而不執行排程。
    - 當後續排程再次執行時，主機則會嘗試UPDATE，並根據「鎖釋放時間是否小於現在時間』，若成立則更新 **並將鎖釋放時間延後**，而較慢發送SQL的主機做判斷時，則會因「鎖釋放時間延後而大於現在時間」被擋下來。
    ```sql=
    create table shedlock(
        name      varchar(64)   not null primary key,
        lock_until timestamp(3) not null,
        locked_at  timestamp(3) default CURRENT_TIMESTAMP(3) not null,
        locked_by  varchar(255) not null
    );
    ```
    #### EnableShedLock
    - 簡單來說就是開啟分散式鎖定的機制，而需要藉由 `@EnableSchedulerLock` 來開啟，同時，我也建立一個Configuration給ShedLock作設定。
    ```java=
    @Configuration
    // 設定當Server崩潰時，鎖最長占用時間
    @EnableSchedulerLock(defaultLockAtMostFor = "1m")
    public class ShedLockConfiguration {

        @Bean // 設定排程鎖元件，跟TaskScheduler是一樣的概念
        public LockProvider lockProvider(DataSource dataSource) {
            return new JdbcTemplateLockProvider(
                    JdbcTemplateLockProvider.Configuration.builder()
                            .withJdbcTemplate(new JdbcTemplate(dataSource))
                            .usingDbTime()
                            .build()
            );
        }
    }
    ```

## Day226
#### 學習重點 : Spring Schedule電商專案實作 - 定期處理未付款訂單.final
- 處理最後的通知part ⭐⭐⭐
    - 這個部分跟Schedule就比較沒有關係啦～只是單純新增了一個Notification API，並新增一個通知表，藉由CancelService與NotificationService來存取使用者的通知訊息！
- 以下是我的實作過程 : ⭐⭐⭐⭐⭐⭐⭐
    - 1️⃣ 建立Notification通知表。本來想說通知跟使用者呈現「**多對多**」，可以建立「通知-使用者關聯表」，但想說 **先以一對多方式處理**，較省時且方便。
    ```sql=
    create table notification(
        id   varchar(100) not null primary key,
        user_id  varchar(100) not null,
        msg      varchar(100) not null,
        is_read  tinyint(1) default 0 not null,
        create_time timestamp not null,
        constraint fk_userid_notification
            foreign key (user_id) 
            references users (id)
            on delete cascade
    );
    ```
    - 2️⃣ 建立Controller-Service-Dao三層結構。接著就是在Controller新增 `/notification` 端點，藉著GET取得使用者的訊息集合。
    - 3️⃣ 接著在 `OrderCancelService` 注入` NotificationDao`，並在排程執行時，設定「取消訊息」並 `.saveAll` 進通知表。
    ```java=
    // OrderCancelService.java
    @Transactional
    public int expiredOrderScheduling(List<Order> orderList){
        // ...省略
        List<Notification> notificationList = new ArrayList<>();

        for (Order order : orderList){
            // ...省略    
            Notification noti = new Notification();
            noti.setUser(order.getUser());
            noti.setMsg(order.getId() + " has been canceled");
            noti.setCreate_time(Instant.now());
            noti.setIs_read(false);
            notificationList.add(noti);
            // ...省略
        }
        notificationDao.saveAll(notificationList);
        // ...省略
    ```
    - 4️⃣ 設定NotificationResponse回傳至前端，再由前端的 `index.html` 接收並呈現！
    ```java=
    // NotificationService.java
    public Response<List<NotificationResponse>> getNotifications(String account){
        User user = userDao.findByAccount(account).orElseThrow(() -> ResourcesException.of(ErrorCode.USER_NOT_FOUND));
        List<Notification> notiList = notificationDao.findAllByUser(user);
        if (notiList.isEmpty()) return new Response<>("0", "No notification", Collections.emptyList());

        List<NotificationResponse> notiResList = notiList
                .stream()
                .map(NotificationResponse::new)
                .toList();

        return new Response<>("0", "Successfully", notiResList);
    }
    ```
- 成果展示如下 : ⭐⭐⭐
    - 下圖是呈現的結果，當排程掃描到過期訂單，則將其取消並通知使用者。
    ![image](https://hackmd.io/_uploads/B1lWav3Lfl.png)
    - 而在訂單頁面則呈現「已取消」。
    ![image](https://hackmd.io/_uploads/HkHUTwhLzg.png)

## Day227
#### 學習重點 : Spring Security與AOP驗證
- 原本的驗證方式 ⭐⭐⭐
    - 原本我的登入與後續驗證邏輯 : 
        - `/login` API帶著account、password
        - `jwtUtil` 以account簽署並生成token放入 `ResponseBody`
        - 後續打API會以 `jwtAuth` 之AOP接收token並驗證（以 `RequestContextHolder` 取得request中的token）
- 為何要用Spring Security？ ⭐⭐⭐⭐
    - **攔截位置** : AOP是在進Controller後做攔截，然而Security是在DispatcherServle前建立一道Filter chain（最外層）來過濾資訊。
    - **攻擊** : 當有惡意伺服器發送大量請求時，若進到AOP，則需要耗費大量資源。
    - **請求範圍** : AOP僅限於Spring Bean的範圍，然而Security是Servlet Filter，因此可以攔截所有請求。
    - **驗證規模** : 日後專案規模擴展到第三方登入、OAuth等功能，使用AOP等於要自己手刻。然而Security有提供資源可以直接使用。
    - **資安** : 最後是資安防護，遠遠不只JWT驗證這麼簡單，這時就不要再重複造輪子寫AOP了！

## Day228
#### 學習重點 : Spring Security - 何謂UserDetails？
- Principal、Credentials、Authorities ⭐⭐⭐⭐⭐⭐
    - 在理解UserDetails之前，要先理解Principal、Credential、Authority。
    - 這三者也是Spring Security的核心原理！
        - Principal : 表明你是誰？ ➝ **username、account**等都是。
        - Credential : 證明你真的是你 ➝ **password、token**等。
        - Authorities : 你的角色/權限有？ ➝ 像是**買家、賣家、可以下單、可以修改訂單**等。
    - 在Authentication（驗證）Login階段 : 透過account、password驗證，此時Principal為account、Credential為password。驗證後，清空Credential，並取得Authorities，封裝成User物件。
        - 在Controller當中，除了上述驗證過程，還會以Principal、Authorities簽署JWT token並放入ResponseBody回傳至前端！（這部分就可以用AOP來做）
        - 當帶著JWT token請求時，經FilterChain驗證後包裝成User物件丟進 `SecurityContextHolder` 中，供當前請求的上下文作用（當前API）。
    - 而上述的過程就需要藉由UserDetails作為標準User介面格式來給Security驗證！
- UserDetails介面的成員 ⭐⭐⭐
    - 作為User標準格式，需要有存取Principal、Credential、Authority的功能，因此介面圍繞著這三者轉！
    ```java=
    public interface UserDetails extends Serializable {
        Collection<? extends GrantedAuthority> getAuthorities();
        String getPassword();
        String getUsername();
        boolean isAccountNonExpired();
        // 帳號是否鎖住了？ 像是密碼輸錯過多次等 
        boolean isAccountNonLocked();
        boolean isCredentialsNonExpired();
        boolean isEnabled();
    }
    ```
    - 而我們會藉由Spring給定的UserDetailsService介面中的 `loadUserByUsername` 作為標準取得資料庫中User的途徑！
    - 而當然，UserDeatils需要被實作，因此可以自行加更多成員上去。
    - 而UserDetailService也是，由於Spring不知道我們使用的DB為何，因此我們需要實作Service。
- AuthenticationManager介面 ⭐⭐⭐⭐⭐⭐
    - 上面介紹了UserDetails介面核心的要素，以及Authentication的流程，那麼實際上驗證的工具是誰呢？ ➝ `AuthenticationManager`！
    - AuthenticationManager機制是這樣 ➝ 「LoginRequest的account與password」跟「UserDetailsService撈出來的UserDetails」，交由 `PasswordEncoder` 比對明文與加密密碼比對驗證過後，給出一個 `UsernamePasswordAuthenticationToken` 作為Authentication物件，最後再存入SecurityContextHolder！
    - 而同時也會拿著authentication去產生JWT token並回傳，作為後續使用者打api時的身分證。
- JWT token跟SecurityContextHolde的關聯 ⭐⭐⭐⭐⭐⭐
    - JWT作為「**Stateless**」的一方，是供使用者可以拿著token打api，再經FilterChain的驗證後，解密token對應的UserDetails物件，最後存入SecurityContextHolder，接著才進到Controller去執行該api的動作！

## Day229
#### 學習重點 : Spring Security - 解析SecurityConfig
- 舊版的Config長怎樣？ ⭐⭐⭐⭐
    - 在舊版的SecurityConfig中，我們需要繼承 `WebSecurityConfigurerAdapter` 並配合 `@EnableWebSecurity` 才能啟用Spring Security。
    - 而在繼承Adapter中，一般會需要覆寫configure，包含 `AuthenticationManagerBuilder`、`HttpSecurity`、`WebSecurity`。
        - HttpSecurity就是針對Http請求的部分，包含 `API驗證`、 `session設定(設定JWT無狀態)`、`CSRF啟用/關閉`、`FilterChain的設定`，可以看出Http請求會經過一連串的檢查才進入Controller。
        -  而WebSecurity相比HttpSecurity，其不會走FilterChain這條路，而是繞過直達Controller，因此一般來說只會讓靜態資源走這條路。
        -  最後是 `AuthenticationManagerBuilder`，其根本上就是組裝成AuthenticationManager的配置類別，也就是組裝 `passwordEncoder`、`UserDetailsService` 的配置。
-  現今的Spring Security Configuration ⭐⭐⭐⭐⭐
    -  現今的配置檔 **不需要再繼承Adapter**，而是以宣告一個配置SecurityFilterChain的函式並回傳，加上 `@Bean` 放入Spring容器作為元件。
    -  而透過舊式Config可以知道，FilterChain屬於HttpSecurity，因此內容就是在配置一般Http請求走的路線。
    -  以下是我的實作 : 
    ```java=
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity httpSecurity) throws Exception{
        httpSecurity
            .csrf(AbstractHttpConfigurer::disable)
            // 這邊將SessionPolicy宣告成STATELESS
            // 意即不會利用Session儲存使用者登入資訊，也因此不會有SessionID存入Cookie
            // 這也是為何要把CSRF關掉，因為根本沒用
            .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                    .requestMatchers("/login", "/registration", "/forgetPassword", "/prod/**", "/webhook/**").permitAll()
                    .anyRequest().authenticated()
            );
        return httpSecurity.build();
    }
    ```
    - 而WebSecurity也是按照上述寫一個Bean丟給Spring管理 : 
    ```java=
    @Bean
    public WebSecurityCustomizer webSecurityCustomizer() {
        return (web) -> web.ignoring().requestMatchers(
                "/css/**", "/js/**", "/images/**", "/favicon.ico", "/swagger-ui/**",  "/*.html", "/*.png", "/static/**"
        );
    }
    ```
    - 最後是 `AuthenticationManagerBuilder`，在現今版本中，我們通常會宣告 `passwordEncoder`、`UserDetailsService` 為Bean，Spring就會自動組裝成AuthenticationManager供我們的Controller使用拉~ 而這部份我想說留到明天來做好了！

## Day230
#### 學習重點 : Spring Security - 探討AuthenticationManager與實作UserDetails
- 完善Security Config ⭐⭐⭐⭐⭐⭐
    - 首先先把 `PasswordEncoder` 以及 `AuthenticationManager` 作為Bean丟給Spring容器管理！
    - 而這邊就不得不提到 `AuthenticatoinManager` 的驗證實作流程拉~
    #### ProviderManager、AuthenticationProvider
    - 簡單來說，在AuthenticationManager這個介面中，最常實作的類別是 `ProviderManager`，也是預設使用的類別。
    - 它專門維護Provider，而Provider就是 `驗證方式`，由於驗證方式有很多種，如 : 帳密登入、第三方登入、指紋登入...，因此若都塞在AuthenticationManager，會變得不好維護。
    - 在ProviderManager中，其委派 `AuthenticationProvider` 介面做驗證處理
    ```java=
    public interface AuthenticationProvider {

        // 驗證邏輯
        Authentication authenticate(Authentication authentication) throws AuthenticationException;

        // 是否能處理XXX驗證方式的集合
        boolean supports(Class<?> authentication);
    }
    ```
    - 因此實作AuthenticationProvider的類別才是真正處理驗證邏輯的地方！
    - 而最常使用的Provider則是 `DaoAuthenticationProvider`
    - 以下是來自 [Spring官網的流程圖](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/dao-authentication-provider.html) : 
    ![image](https://hackmd.io/_uploads/BkbGK3bvzg.png)
    - 因此可以知道 `DaoAuthenticationProvider` 就是結合UserDetailsService以及PasswordEncoder去做驗證。
- UserDetails、UserDetailsService實作 ⭐⭐⭐⭐
    - 簡單來說就是在原本我設計的User、UserService後面加上 `implements`，然後覆寫方法！
    - 比較酷的是關於User的Authorties覆寫方式 : 
    ```java=
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        if (role == null) return List.of();
        return List.of(new SimpleGrantedAuthority("ROLE_" + role.getName()));
    }
    ```
    - 由於我本來就有設定Role類別，因此可以直接以role.getName來設計Authorities。
    - 而UserDetails的部分則是實作loadUserByUsername : 
    ```java=
    @Override
    public UserDetails loadUserByUsername(String account) throws UsernameNotFoundException{
        return userDao.findByAccount(account).orElseThrow(() -> new UsernameNotFoundException("User not found: " + account));
    }
    ```
    - 這樣寫，當我們在做login驗證時，就可以使用 `AuthenticationManager` 在Controller做驗證啦！

## Day231
#### 學習重點 : Spring Security - 實作JwtAuthenticationFilter
- 自訂義Filter？ ⭐⭐⭐⭐⭐⭐⭐
    - 在看自訂義Filter之前，要先知道Spring Security是如何操控Filter的。
    - FilterChain的邏輯是這樣的 : 每個Filter接收 `request`、`response`、`filterChain`，經過本身的過濾邏輯後，再把request、response塞入filterChain丟給下一個Filter。
    - 因此通常在Filter定義中會看到 : 
    ```java=
    public final void doFilter(ServletRequest request, 
                               ServletResponse response, 
                               FilterChain filterChain
    ) throws ServletException, IOException {
        //...省略
    }
    ```
    - 而我們若要自訂義Filter，通常會去繼承 `OncePerRequestFilter`，它是一個抽象類別，而我們需要覆寫其中的doFilterInternal來設計過濾邏輯！但先讓我來釐清一下何謂 `OncePerRequestFilter` : 
    #### OncePerRequestFilter
    - 如它的名字，就是針對同一Http請求，只會「執行一次Filter邏輯」。
    - 但是為何要有這種功能呢？ ➝ 
        - 當今天我們遇到了 `轉發`、`錯誤處理` 等在 `同一Http請求下` 的動作，會觸發 **多次** Filter。
        - 這時候，我們不希望同一請求重複過濾（因為同一請求不需要過濾超過一次），就可以利用 `OncePerRequestFilter` 這個類別，利用標記法，先「確認該請求是否過濾過」，來決定是否放行！
- 設計JwtAuthenticationFilter ⭐⭐⭐⭐⭐
    - 在了解了FilterChain的邏輯後，就可以來專心寫Jwt過濾邏輯拉～
    - 步驟如下 : 
        - 首先針對「**是否有帶JWT token的邏輯判斷**」
            - 若沒帶token則不屬於JwtAuthenticationFilter的過濾範疇，因此放行給後續的Filter去偵測。 
        - 再來進入「**token的verify**」
            - 若取出的token內涵account 且 SecurityContextHolder中的authentication是空的，表示還沒設定過，則可繼續執行設定。
        - 接著進入「**設定Context環節**」
            - 透過token中的account與UserDetailService找到User，形成authentication物件，最後放入 `SecurityContextHolder` 完成設定。
        - 最後「**交由下一個Filter動作**」
            - 使用 `filterChain.doFilter(request,response)` 交給下一個Filter執行。
    - 以下是完整程式碼 : 
    ```java=
    @Component
    public class JwtAuthenticationFilter extends OncePerRequestFilter {

        private final UserService userService;

        @Value("${secret}")
        private String secret;

        @Autowired
        public JwtAuthenticationFilter(UserService userService){
            this.userService = userService;
        }

        @Override
        protected void doFilterInternal(HttpServletRequest request,
                                        HttpServletResponse response,
                                        FilterChain filterChain) throws ServletException, IOException{

            //* 取出原始token
            String rawToken = request.getHeader("Authorization");
            
            //* 若token有問題，交由後續Filter決定是否放行
            if (rawToken == null || !rawToken.startsWith("Bearer ")) {
                filterChain.doFilter(request, response);
                return;
            }
            String token = JwtUtil.decode(rawToken);

            try{
                JwtUtil.verify(token, secret);
                String account = JwtUtil.getString(token, "account");
                if (account != null && SecurityContextHolder.getContext().getAuthentication() == null){
                    UserDetails userDetails = userService.loadUserByUsername(account);
                    UsernamePasswordAuthenticationToken authentication = new UsernamePasswordAuthenticationToken(
                            userDetails,
                            null,
                            userDetails.getAuthorities()
                    );
                    authentication.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                    SecurityContextHolder.getContext().setAuthentication(authentication);
                }

            }catch (Exception e){
                System.out.println("Exception : " + e.getMessage());
            }

            filterChain.doFilter(request, response);
        }
    }
    ```

## Day232
#### 學習重點 : Spring Security - 淺談FilterChain的順序
- 將JwtAuthenticationFilter加入進Security Filter Chain ⭐⭐⭐⭐⭐
    - 在寫完Jwt的過濾邏輯之後，當然就是要將其加入進Security中啦~
    - 在SecurityFilerChain的配置方法中，我們需要在httpSecurity的建置過程中，將Jwt填入FilterChain中！
    ```java=
    // 先將JwtFilter印入
    private final JwtAuthenticationFilter jwtAuthenticationFilter;

    // 並加進初始化中
    public SecurityConfiguration(JwtAuthenticationFilter jwtAuthenticationFilter) {
        this.jwtAuthenticationFilter = jwtAuthenticationFilter;
    }

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity httpSecurity) throws Exception{
        httpSecurity
                // ...省略
                .addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class);
        return httpSecurity.build();
    }
    ```
    - 但這邊產生一個問題 ➝ Spring Security的Filter Chain是怎麼設置的？
- SecurityFilterChain ⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐
    #### 請求進入到Servlet的動作
    - 一個請求在進入Servlet時，並不知道要走Spring的Filter Chain，而是直接進入Servlet的Filter。
    - 這時就需要利用 `DelegatingFilterProxy`，它可以將請求**轉接**到Spring的FilterChain中。
    - 因此 `DelegatingFilterProxy` 就是將原本Servlet的Filter邏輯委派給Spring Bean（SecurityFilter）來代理！
    - 而身為SpringSecurityFilter的代理人就是 `FilterChainProxy`，這時請求就正式進入了Spring FilterChain的領域啦~
    > HTTP Request → DelegatingFilterProxy → FilterChainProxy → Controller
    #### FilterChain的過濾順序（我簡略很多w，只講重點）
    - 整個FilterChain的第一線就是 `FilterChainProxy` 本身！
        - 它會管理多組FilerChains ➝ `List<SecurityFilterChain>`，並針對不同URL使用不同的FilterChain。
    - 再來是 `SecurityContextHolderFilter`，載入上下文，掌管FilterChain的開始與結束。
        - 一般來說會從 `SecurityContextRepository` 載入之前的資訊，像是Session，並放入ThreadLocal。
        - 若找不到Session，代表可能是STATELESS的Jwt，則Context留空，待進入我的JwtFilter時設定Context為authentiction。
    - 接著進入資安部分的Filter
        - 如 : `CorsFilter`、`CsrfFilter`、`HeaderWriterFilter`、`LogoutFilter`，這邊就不多贅述，直接進入到重點。
    - 再來就是重點 : `AuthenticationFilter` ➝ 「**決定你是誰**」
        - 傳統的login作法，是利用配置中啟用 `.formLogin()` 的登入功能，去攔截 `[POST] /login`，並取得username、password，委派給AuthenticationManager。這樣的功能是寫在 `UsernamePasswordAuthenticationFilter` 中，這是STATEFUL的部分，也就是身分存儲於Session中。
        - 但我的login是獨立在Controller中，透過Service簽發JWT token，使登入狀態變成STATELESS，因此在配置時，我一開始就放行login，使其不在FilterChain被擋下來，而是直接進入Controller。
        - 因此昨天設計的JwtFilter，就是針對STATELESS的Filter。
        - 所以回到上一大點的程式碼 : `.addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class);`，我將設置Context的動作交由Jwt實施，而由於沒有在配置檔中啟用 `formLogin`，因此UsernamePasswordAuthenticationFilter根本就不會被放進FilterChain中！
    - 最後是 `AuthorizationFilter` ➝ 「**決定你這個身分能去哪**」
        - 在配置中設定的 `permitAll`、`authenticated` 都是在這層認證Filter中去做動作，以下是簡略圖 : 
        ``` 
                 Request 到達 AuthorizationFilter
                                 │
                                 ▼
                拿URL去比對你在 SecurityConfig 寫的規則
                                 │
           ┌─────────────────────┴───────────────────────┐
           ▼                                             ▼
       命中 permitAll() 規則                   命中 .authenticated() 規則
           │                                             │
      【直接放行！】                          檢查 SecurityContextHolder 
           │                                 有沒有已認證的 Authentication？
           │                                             │
           │                               ┌─────────────┴─────────────┐
           │                               ▼                           ▼
           │                         【有（已登入）】             【未登入/Token無效】
           │                               │                           │
           │                          【放行通過！】             【拋出 403 / 401】
           │                               │                           │
           └─────────────────┬─────────────┘                           │
                             ▼                                         ▼
                    順利進入 Controller                           中斷請求，返回錯誤
        ```
    - 今天先差不多寫到這！明天繼續完善後續進入Controller的 `PreAuthorize` 等AOP機制！

## Day233
#### 學習重點 : Spring Security - Method Security簡介
- 甚麼是Method Security？ ⭐⭐⭐⭐
    - 前幾天我一直研究的屬於 `WebSecurity` 的範疇，而經過FilterChain正式進入Controller時，就屬於 `MethodSecurity` 的部分啦～
    - 為何要有Method Security？ 
        - 可以說，Web防禦屬於「**粗粒度**」的過濾，而Method防禦屬於「**細粒度**」的過濾。
        - `WebSecurity` 負責處理URL訪問權限、身分及使用者驗證等。而 `MethodSecurity` 則可以結合使用者與方法參數加上Service Bean，判斷是否有XX權限。
        - 因此MethodSecurity的細粒度在於**結合了業務邏輯去檢查**，提升防禦性！
    - 在最基本的Method Security中，我們會透過驗證過的UserDetails的Authorities內涵的 `ROLE_{身分}` 及 `{權限}` 去判斷使用者是否能進入該Method，作為進入Method前的防線。
- Method Security怎麼使用？ ⭐⭐⭐⭐
    - 還記得在剛學實作Spring AOP時，我設計了一個 `@RequirePermission` 去判斷使用者是否含有某個Permission。像是 : `@RequirePermission(needPermissionOf = PermissionCode.GET_USERS_LIST)`
    - 而在MethodSecurity，則是可以使用Spring Security內涵的四個註解去實現判斷 : 
        - `PreAuthorize`、`PostAuthorize`、`PreFilter`、`PostFilter`
    - 但我應該會專注在 `PreAuthorize` 上去做研究，因為它適用於大部分場景！
    - 在進入註解使用之前，需要先在SecurityConfig上啟用MethodSecurity，因為Spring Security預設是不會啟用方法攔截的！
    ```java=
    @Configuration
    @EnableWebSecurity
    @EnableMethodSecurity // <-- here
    public class SecurityConfiguration {
        //...省略
    }
    ```
- Method Security的基本使用方式（`PreAuthorize` 示範） ⭐⭐⭐⭐⭐
    - 一般來說，我們會將 `@PreAuthorize` 加註在Contoller的端點Method前面。
    - 而參數會以字串形式放入 : `hasRole`、`hasAuthority`、`hasPermission` 等。
        - 如 : `@PreAuthorize("hasRole('ADMIN')")`
    - 以上面的例子，當呼叫帶有 `PreAuthorize...` 的方法時，MethodSecurity就會以 **SpEL**（Spring Expression Language）去解析字串內的敘述，並到UserDetails中搜尋 `ROLE_ADMIN` 的身分，若找不到則丟出 `AccessDeniedException`，找到則表示通過驗證進入Method中！
    - 這就是一般MethodSecurity的解析與驗證方式！

## Day234
#### 學習重點 : Spring  Security - 深入探討@PreAuthorize.1
- 前言 ⭐
    - 昨天有簡單講過Method Security註解的使用方式，而今天將要深入探討PreAuthorize的詳細用法！
- hasRole vs. hasAuthority ⭐⭐⭐⭐⭐
    - 這兩者的差別在於 : 是否有加 `ROLE_`。
    - 在UserDetails中存的Authrities，內涵帶有 `ROLE_` 前輟的身分字串，以及一般的權限字串。
        - 而當初我在實作UserDetails介面時，我在getAuthorities的函式內寫了這個 : 
        ```java=
        //...省略
        return List.of(new SimpleGrantedAuthority("ROLE_" + role.getName()));
        ```
        - 由於我有自行建立Role類別，包含ADMIN跟USER，在藉由Role類別對應到Permission類別，因此一般來說，我的User中不會帶有權限字串，而只有身分字串！
        - 因此我後續實作一般會使用hasRole而不是hasAuthority！
    - 若我寫hasRole('ADMIN')，Spring在解析時，會在前面自動加上 `ROLE_` 再去做運算。
- 關於使用SpEL的解析 與 簡單實作 ⭐⭐⭐⭐⭐
    - 一般來說，當我寫下 `@PreAuthorize("...")` 時，Spring會做以下事情 : 
       - 注入 `authentication` (當前登入者身分)
       - 注入 `principal` (User物件)
       - 注入 `#方法參數 (如 #id, #account)`
   - 因此我們一般會透過上述三者注入來實作驗證
   - 以下是我針對 「**刪除使用者**」 的API端點實作進入前的審查 ➝ 是否為 `ADMIN / 是否是使用者本人`？
   ```java=
    //* 刪除使用者
    @PreAuthorize("hasRole('ADMIN') or #deleteUserRequest.id == authentication.principal.id")
    @DeleteMapping("/users")
    public Response<UserResponse> delete(@AuthenticationPrincipal User user, @RequestBody DeleteUserRequest deleteUserRequest){
        return userService.delete(user.getAccount(), deleteUserRequest);
    }
   ```
   - 針對 `hasRole` 解析時，就會去User物件找到Authority並檢查是否有 `ROLE_ADMIN` 的字串
   - 而針對刪除方法傳入的 `DTO資料`，可以用 `#` 讓解析去找到DTO物件，並透過DTO.id與principal.id去比對！
- 這跟我自行建立的 `@RequirePermission` 有甚麼差？ ⭐⭐⭐⭐
    - Security內建的AOP，是直接取用FilterChain所打包好的UserDetails物件，**不需再去DB搜尋**。
        - 不像自行建立的AOP，需要先透過AuthAOP解析JWT找到account，接著PermissionAOP再透過 `RequestContextHolder` 的request找到attribute，取出account並從DB去找身分...。

## Day235
#### 學習重點 : Spring Security - 深入探討@PreAuthorize.2
- 何謂Safe Navigation？ ⭐⭐⭐⭐
    - 其實在其他語言中，也有Safe navigation的蹤跡，簡單來說，它就是負責處理null出現時的情況。
    - 在SpEL中，當我們想要呼叫物件的成員or函式時，也是跟一般Java語法相同 ➝ `物件.成員`。然而當物件本身是null時，SpEL無法像Java語法一樣使用if來null check。
    - 此時我們就會用到 Safe Navigation `?.` 來避免NPE出現。
    - 實際用法如右 : `#user?.role?.id == "R0001"`。
    - 邏輯如右 : 當 `?.` 前者的物件為null，呼叫後者成員時，不會拋出NPE，而是直接將該運算式回傳null。
    - ⚠️然而，需要注意的是，當運算左右值兩者都掛上了Safe Navigation，很有可能會出現 `null == null` 反而使運算式出現非預期性成立的情況發生！
- Custom Security Service ⭐⭐⭐⭐⭐
    - 當單純用字串無法完整表達驗證邏輯時，我們就需要依靠自製的Service來驗證，而在SpEL就只需要呼叫Service Bean的判斷方法即可！
    - 一般來說語法如右 : `@Bean.方法(參數)`。
    - 雖然說目前我可以單純透過hasRole去過濾，但當日後有其他權限判斷時，也可以使用自製的Service來做PreAuthorize。
    - 下面是我簡單修改原本的RequirePermission的AOP，變成Security Service的形式 : 
    ```java=
    // 設置Bean名稱為ps
    @Service("ps")
    public class PermissionService {

        private final RolePermissionDao rolePermissionDao;

        @Autowired
        public PermissionService(RolePermissionDao rolePermissionDao){
            this.rolePermissionDao = rolePermissionDao;
        }

        public boolean hasPerm(PermissionCode permissionCode){
            if (permissionCode == null) return false;
            
            Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
            
            // null、principal check
            if (authentication == null || !(authentication.getPrincipal() instanceof User user)) return false;
            if (user.getRole() == null) return false;
            
            return rolePermissionDao.existsByRoleAndPermissionId(user.getRole(), permissionCode.getCode());
        }
    }
    ```
    - 由於在FilterChain中，有個預設Filter叫 `AnonymousFilter`，是當使用者打API時，沒有帶Jwt，表示訪客，此時就會將 `"anonymousUser"` 字串並放入Context中。因此要先檢查authentication帶有的Pricipal是不是User。
    - 接著就可以在SpEL寫下 : `@ps.hasPerm(T(org.system.enums.PermissionCode).GET_USERS_LIST))`。
    - 這邊之所以要寫這麼複雜，是因為SpEL支援Enum的寫法，需要以 `T(Enum路徑).成員` 來描述，因此寫成這樣。

## Day236
#### 學習重點 : Spring Security - hasPermission的深入解析.1
- 為何要有hasPermission？ ⭐⭐⭐⭐⭐
    - 或許在我自行實作的PermissionService可以應付一般的RBAC系統，然而針對某個特定資源要確認是否能存取的話，卻不是那麼容易寫。
        - 讓我們來看看如果要「**刪除訂單id#0001**」這件事要怎麼以PermissionService應付
            - 我需要建立 `ps.canDeleteOrder(principal, #orderId)`
        - 那如果要「**編輯訂單id#0001**」呢？
            - 我需要建立 `ps.canEditOrder(principal, #orderId)`
    - 透過上述例子可以發現PermissionService應對Permission的方式，是一直建立Method去應付各種狀況。十分不理想。
    - 因此針對「**特定資源的動作驗證**」，我們可以利用Spring設計的`hasPermission` 來做，其規定了參數的格式必須是 `hasPermission(目標物件, 操作動作)`、`hasPermission(目標ID, 實體型態, 操作動作)`，很能夠看出其對**資源**訪問的細粒程度。
- SpEL的接收 ⭐⭐⭐⭐⭐⭐
    - 而在真正了解hasPermission之前，需要先來看看SpEL是怎麼被接收並解析的。
    #### MethodSecurityExpressionHandler
    - 在寫下 `@PreAuthorize()` 時，Spring會將內部的SpEL交由 `MethodSecurityExpressionHandler` 來解析文字！
    - 而 `MethodSecurityExpressionHandler` 是介面，因此會藉由 `DefaultMethodSecurityExpressionHandler` 這個預設實作來解析文字。
    - 內部包含了 `hasRole`、`hasAuthority` 等解析函式。而 `hasPermission` 則是委派給 「**PermissionEvaluator**」來解析。
    - 然而預設的PermissionEvaluator介面實作是 `DenyAllPermissionEvaluator`，與字面意思相同，就是遇到任何Permission都回傳false。
    - 因此我們需要自行建立 `CustomPermissionEvaluator` 來實作PermissionEvaluator，並自行設計屬於自己的Permission過濾器，最後再注入Hanlder當中完成註冊。
    - 而我們需要實作的PermissionEvaluator是基與Spring設計的統一格式，因此需要實作以下函式 : 
    ```java=
    public interface PermissionEvaluator extends AopInfrastructureBean {
        boolean hasPermission(Authentication authentication, Object targetDomainObject, Object permission);
        boolean hasPermission(Authentication authentication, Serializable targetId, String targetType, Object permission);
    }
    ```
    - 後續部分留到明天繼續研究吧！

## Day237
#### 學習重點 : Spring Security - hasPermission的深入解析.2
- 兩個hasPermission函式的不同 ⭐⭐⭐⭐⭐⭐
    - 先來看看hasPermission函式為何要有兩種不同的形式
    - 1️⃣ 功能性權限 :
        - `hasPermission(auth, targetObj, permission)`，針對目標實體（DTO）該使用者是否有某permission（如DELETE、EDIT等）。
        - 功能性權限通常會用 `@PostAuthorize` 去看看回傳的物件能否被使用者看到（可能包含隱私成員）。
    - 2️⃣ 資源性權限 : 
        - `hasPermission(auth, targetId, targetType, permission)`，針對某Type的資料(ID)，使用者是否有某permission（如DELETE、EDIT等）。
        - 一般來說會需要去DB搜尋該資料並判斷，通常在 `@PreAuthorize` 中攔截id與使用者做權限判斷。
        - 這就跟自製的PermissionService有極高相似之處，若想要統一規格，使用hasPermission語法會較好，若需要高度自由性，則自製Service較優。
- 分發權限架構設計（Strategy Pattern） ⭐⭐⭐⭐⭐
    - 由於一個專案中，會有許多實體，因此衍生出不同資源、不同訪問動作。
    - 我們可以自行建立一個 `DomainPermissionChecker` 介面，並由不同 `資源PermissionChecker` 去實作 ➝ 形成一組CheckerBeans。
    - 最後自製 `GlobalPermissionEvaluator` 去實作 `PermissionEvaluator`，其中注入 `Map<String, DomainPermissionChecker>`，Map由 `targetTypeName 對 CheckerBean`。
    - 流程如右 : 使用hasPermission字句 ➝ 呼叫Evaluator的hasPermission ➝ 根據傳入的targetTypeName找到相對應的CheckerBean ➝ 呼叫實際該CheckerBean的 `hasPermission`。
    > Controller --> PreAuthorize --> @hasPermission --> GlobalEvaluator --> DomainPermissionChecker --> EntityPermissionChecker.hasPermission

## Day238
#### 學習重點 : Spring Security - Evaluator Strategy Pattern實作.1
- DomainPermissionChecker ⭐⭐⭐⭐⭐
    - 首先是關於每個實體CheckerBean必須實作的一個介面，`DomainPermissionChecker`，跟Spring設計的有些許不同，我 **將Authentication的部分改成了User**，且 **針對資源性權限參數，省略了Type的傳入**。
    - 以下是介面 : 
    ```java=
    public interface DomainPermissionChecker {
        String getTargetType();
        // 功能性權限
        boolean hasPermission(User user, Object targetDomainObject, String permission);
        // 資源性權限
        boolean hasPermission(User user, Serializable targetId, String permission);
    }
    ```
    - getTargetType是用於取得實體的型別名稱。
    - 而每個實作 `DomainPermissionChecker` 的實體Checker，都需冠上 `@Component` 的註解，來使其成為Bean。
- GlobalPermissionEvaluator ⭐⭐⭐⭐⭐
    - `DomainPermissionChecker` 是關於實體實作的介面，而 `GlobalPermissionEvaluator` 則是實作 `PermissionEvaluator`，並在內部注入 `Map<String, DomainPermissionChecker>`，此時Spring就會去容器中找到有實作 `DomainPermissionChecker` 的Bean（這也是為何前一點說要對實體Checker冠上Component的原因）。
    - 以下是架構 : 
    ```java=
    @Component
    public class GlobalPermissionEvaluator implements PermissionEvaluator {
        
        // 管理一組CheckerBeans，key是bean的實體名稱，value就是checkerBean本身
        private final Map<String, DomainPermissionChecker> permissionCheckerMap;

        @Autowired
        public GlobalPermissionEvaluator(List<DomainPermissionChecker> domainPermissionCheckerList){
            this.permissionCheckerMap = domainPermissionCheckerList.stream().collect(
                    Collectors.toMap(
                            dpc -> dpc.getTargetType().toUpperCase(),
                            dpc -> dpc
                    )
            );
        }

        @Override
        public boolean hasPermission(Authentication authentication, Object targetDomainObject, Object permission){
            // 先做好null防線，並針對Authentication預先取得User
            if (authentication == null || !(authentication.getPrincipal() instanceof User user) || targetDomainObject == null || permission == null){
                return false;
            }
            
            // 傳入的targetObject先透過getClass取得實體型態名稱
            String targetType = targetDomainObject.getClass().getSimpleName().toUpperCase();
            // 藉由targetType作為key去找checkerBean
            DomainPermissionChecker dpc = permissionCheckerMap.get(targetType);
            if (dpc == null) return false;
            // 利用剛剛null check的user物件傳入自行設計的DomainPermissionChecker介面
            return dpc.hasPermission(user, targetDomainObject, permission.toString());
        }

        @Override
        public boolean hasPermission(Authentication authentication, Serializable targetId, String targetType, Object permission){
            if (authentication == null || !(authentication.getPrincipal() instanceof User user) || targetId == null || permission == null){
                return false;
            }
            
            // 針對資源性權限可以直接利用參數的targetType去搜尋鍵值對
            DomainPermissionChecker dpc = permissionCheckerMap.get(targetType.toUpperCase());
            if (dpc == null) return false;

            return dpc.hasPermission(user, targetId, permission.toString());
        }
    }
    ```
- 實際流程 ⭐⭐⭐⭐⭐
    - 透過下圖可以清楚知道分流，而實際CheckerBean則留到明天來實作吧！
    ![image](https://hackmd.io/_uploads/r1A6wthvzl.png)

## Day239
#### 學習重點 : Spring Security - Evaluator Strategy Pattern實作.2
- DomainPermissionChecker介面實作 - Order ⭐⭐⭐⭐⭐⭐
    - 今天先從OrderPermissionChecker開始著手設計。
    - 針對兩個hasPermission我目前 **先採用相同permission檢查設計**。
    - 先來看看針對Order有什麼樣的操作（permission）出現 : 
        - `READ、PAY、CANCELED、DELETE`
    - 進一步的解析，可以歸類成「Owner可操作」 or 「ROLE_ADIMN可操作」
        - 這邊就融合了**ABAC**(基於屬性存取 : Owner)、**RBAC**(基於角色存取 : ADMIN)
        - 而Owner可操作的有 `READ、PAY、CANCELED(有限)、DELETE(有限)`
        - 而ADMIN可操作的有 `READ、CANCELED、DELETE`
        - 那麼這邊我 **先不加入Seller**，帶整體完善後再來補強！
    - 透過上述的權限設定，可以用 **swtich-case來做分流** : 
    ```java=
    return switch (permission) {
        case "READ" -> isOwner || isAdmin;
        case "PAY" -> isOwner;
        case "REFUND" -> isAdmin;
        case "DELETE", "CANCELED" -> isAdmin || (isOwner && order.getStatus() == Order.STATUS.UNPAID);
        default -> false;
    };
    ```
- 實作類別OrderPermissionChecker展示
    - 以下是完整code : 
    ```java=
    @Component
    public class OrderPermissionChecker implements DomainPermissionChecker {

        private final OrderDao orderDao;

        @Autowired
        public OrderPermissionChecker(OrderDao orderDao){
            this.orderDao = orderDao;
        }

        public String getTargetType(){return "ORDER";}

        @Override
        public boolean hasPermission(User user, Object targetDomainObject, String permission){
            if (!(targetDomainObject instanceof Order order)) return false;
            return checker(user, permission, order);
        }

        @Override
        public boolean hasPermission(User user, Serializable targetId, String permission){
            Order order = orderDao.findById(targetId.toString()).orElseThrow(() -> ResourcesException.of(ErrorCode.ORDER_NOT_FOUND));
            return checker(user, permission, order);

        }

        private boolean checker(User user, String permission, Order order){
            boolean isOwner = user.getId().equals(order.getUser().getId());
            boolean isAdmin = user.getRole().getId().equals(RoleID.ADMIN.getID());

            return switch (permission) {
                case "READ" -> isOwner || isAdmin;
                case "PAY" -> isOwner;
                case "REFUND" -> isAdmin;
                case "DELETE", "CANCELED" -> isAdmin || (isOwner && order.getStatus() == Order.STATUS.UNPAID);
                default -> false;
            };
        }
    }
    ```

## Day240
#### 學習重點 : Spring Security - Evaluator Strategy Pattern實作.final
- 剩餘Contoller與實體 ⭐⭐⭐⭐
    - 列出目前Controller : `Notification`、`Cart`、`Merchandise`、`User`、`Category`、`Webhook`
    - 然而其中的Webhook不進入Spring Security的PreAuthorize中，原因如下 : 
        - 第三方發起API請求時不會帶有JWT，也沒有User物件。
        - 且在FilterChain當中，通常會設置成permit，不經過FilterChain。
        - 一般來說，Webhook會直達Controller並透過Service去處理「**檢查碼**」來確認身分。
    - 其餘實體搭配permission設計，就可以完善Checker啦~
- 實體CheckerBean完善 ⭐⭐⭐
    - 在完善Bean之前，我想要微調昨天寫的邏輯 : 
        - 針對targetID not found時，不丟出Exception，而是直接 `return false`，**更符合Security的規範**，也不會造成資安問題(狀態洩漏)。
        - 而在checker method的部分，則是加強null check的部分。
    - 接下來就是每個實體的基本permission設計啦 
        - Notification : `READ, EDIT, DELETE`
        - Cart : `READ, ADD, EDIT, DELETE, CLEAR`
        - Merchandise : `READ, EDIT, DELETE`
        - User : `READ, EDIT, DELETE`
        - Category : `READ, CREATE, EDIT, DELETE`
    - 而最後，則可以根據OrderPermissionChecker的邏輯，將以上實體都完成實作  `DomainPermissionChecker` 的動作！
    - 但由於程式碼過於冗長，我就不放上來了！
- 明天預計 : ⭐
    - 我會開始整合關於Schedule、Security的執行邏輯，並將原本AOP驗證的功能交由PreAuthorize來完成！

## Day241
#### 學習重點 : Spring Security - PreAuthorize全面上線！
- 前置作業 ⭐⭐⭐⭐
    - 在我們寫好 `GlobalPermissionEvaluator` 後，還需要將其註冊到SecurityConfig中！否則Spring會使用預設的Evaluator，其中委派的PermissionEvaluator實作是 `DenyAllPermissionEvaluator`。
    - 而註冊的方式就是設置最頂層的MethodSecurityExpressionHandler Bean : 
    ```java=
    // 加入我們的實作
    private final GlobalPermissionEvaluator globalPermissionEvaluator;

    // 初始化
    public SecurityConfiguration(JwtAuthenticationFilter jwtAuthenticationFilter, GlobalPermissionEvaluator globalPermissionEvaluator) {
        this.jwtAuthenticationFilter = jwtAuthenticationFilter;
        this.globalPermissionEvaluator = globalPermissionEvaluator;
    }
    
    // 設置Bean
    @Bean
    public MethodSecurityExpressionHandler methodSecurityExpressionHandler(){
        DefaultMethodSecurityExpressionHandler expressionHandler = new DefaultMethodSecurityExpressionHandler();
        expressionHandler.setPermissionEvaluator(globalPermissionEvaluator);
        return expressionHandler;
    }
    ```
- 針對原本AOP的替換 ⭐⭐⭐⭐⭐
    - 原本我的自製AOP是 `RequirePermission` ➝ 針對PermissionCode做檢查。
    - 但為了因應RBAC及PreAuthorize系統，我改成 `PreAuthorize("hasRole('ADMIN')")` 形式。
- 針對純傳入ID執行動作 ⭐⭐⭐⭐⭐⭐⭐
    - 有些動作只需要我們傳入ID即可執行，如 : 商品的刪除。
    ```java=
    @DeleteMapping("/prod/{merchandiseId}")
    public Response<MerchandiseResponse> deleteById(@PathVariable String merchandiseId){
        return merchandiseService.deleteById(merchandiseId);
    }
    ```
    - 此時就可以利用PreAuthorize針對資源性的檢查 : 
    ```java=
    // 加入hasPermission
    @PreAuthorize("hasPermission(#merchandiseId, 'MERCHANDISE', 'DELETE')")
    @DeleteMapping("/prod/{merchandiseId}")
    public Response<MerchandiseResponse> deleteById(@PathVariable String merchandiseId){
        return merchandiseService.deleteById(merchandiseId);
    }
    ```
    - 上述針對ID的應用是屬於「會經過FilterChain」，因此Authentication會驗證的。
    - 若是像 `getCategoryById` 這種看類別功能，訪客也能請求的路徑，一般會permitAll，因此不符合PreAuthorize的概念，也自然不能使用。
- 業務邏輯簡化 ⭐⭐⭐⭐⭐⭐
    - 由於PreAuthorize可以幫我們攔截非資源訪問許可，因此Service層就不需要再針對isOwner、isAdmin等去判斷，也可以移除掉！
    - 原本我的deleteUser Service長這樣 : 
    ```java=
    @Transactional
    public Response<UserResponse> delete(String account, DeleteUserRequest deleteUserRequest){
        String id = deleteUserRequest.getId();

        User operator = userDao.findByAccount(account).orElseThrow(() -> new AuthException(4001, "UserTokenError"));
        User target = userDao.findById(id).orElseThrow(() -> ResourcesException.of(ErrorCode.USER_NOT_FOUND));

        boolean isSelf = operator.getId().equals(target.getId());
        boolean hasDeletePermission = rolePermissionDao.existsByRoleAndPermissionId(operator.getRole(), PermissionCode.DELETE_USER.getCode());
        if (!(isSelf || hasDeletePermission)) return new Response<>("1", "InValidOperation", null);

        UserResponse userResponse = new UserResponse(target);

        cartDao.findByUser(target).ifPresent(cartDao::delete);
        userDao.delete(target);
        return isSelf ? new Response<>("0", "Successfully delete " + target.getUsername() + " By " + target.getUsername(), userResponse)
                : new Response<>("0", "Successfully delete " + target.getUsername() + " By ADMIN", userResponse);
    }
    ```
    - 而現在可以改成這樣 : 
    ```java=
    @Transactional
    public Response<UserResponse> delete(DeleteUserRequest deleteUserRequest) {
        User target = userDao.findById(deleteUserRequest.getId())
                .orElseThrow(() -> ResourcesException.of(ErrorCode.USER_NOT_FOUND));
        
        cartDao.findByUser(target).ifPresent(cartDao::delete);
        userDao.delete(target);

        return new Response<>("0", "Successfully delete " + target.getUsername(), new UserResponse(target));
    }    
    ```

## Day242
#### 學習重點 : Spring Security - FilterChain的Exception Handling
- 權限不足、Token過期的處理 ⭐⭐⭐⭐
    - 在處理Schedule身分問題之前，先要來處理回傳給前端的Http code。
    - 由於FilterChain是 **處理進入Controller前** 的請求，因此攔截的Exception自然會寫在Spring Config當中（而不會用在RestControllerAdvice），且不是用自訂的業務邏輯回傳碼，而是用Http status code。
    - 我們一般會實作兩個介面
        - `AuthenticationEntryPoint` : 負責驗證時的例外，像是token過期、未帶token去打受保護之API...。
        - `AccessDeniedHandler` : 負責被PreAuthrorize擋下來的請求，因此是跟「權限、角色」相關。
- 實作於SecurityConfig ⭐⭐⭐⭐
    - 我們可以直接用lambda實作接在httpSecurity的設置形式中 : 
    ```java=
    httpSecurity
        .csrf(//...)
        .sessionManagement(//...)
        .authorizeHttpRequests(//...)
        .addFilterBefore(//...)
        .exceptionHandling(ex -> ex
            // 401: token過期 or 未登入去打受保護API
            .authenticationEntryPoint((request, response, authException) -> {
                response.setContentType("application/json;charset=UTF-8");
                response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
                response.getWriter().write("{\"code\":401,\"message\":\"未登入或憑證已失效，請重新登入\"}");
            })
            // 403: 權限不足（被 @PreAuthorize 擋下）
            .accessDeniedHandler((request, response, accessDeniedException) -> {
                response.setContentType("application/json;charset=UTF-8");
                response.setStatus(HttpServletResponse.SC_FORBIDDEN);
                response.getWriter().write("{\"code\":403,\"message\":\"權限不足，拒絕訪問\"}");
            })
        );
    ```
- 實際成果 : ⭐⭐
    - 透過上述exceptionHandling的實作，加上原本設計的PreAuthorize，可以看到以下成果 : 
    ![image](https://hackmd.io/_uploads/rJEp0q-Ofx.png)
    - 當我想新增商品分類類別時，由於不是ADMIN，因此被PreAuthorize擋下來，並藉由accessDeniedHandler回傳前端。

## Day243
#### 學習重點 : Schedule與Security的AOP身分Bypass
- 為何Schedule要Bypass ⭐⭐⭐
    - 雖然在現在的架構設計都是在Controller中加上PreAuthorize來驗證請求，但業務邏輯Service方法**本身也可以加上PreAuthorize**，此時我們設計的排程系統可能就會受到影響。
    - 因此我們有幾種方式能夠讓排程方法不受影響執行某些功能 : 
        - 設計**函式多載**，讓排程跟使用者針對某函式走不同路線，但結果相同。
        - 在排程加入SecurityContextHolder，給予身分，使其通過權限檢查。
        - 設計AOP，針對需要的**排程系統加上註解**，設定身分參數，來通過權限檢查。
    - 一般來說，只在Controller設計PreAuthorize的專案，是不是需要設計這種bypass的，然而當架構越來越複雜時，可能就需要函式多載或者藉由設計身分給予功能來通過檢查。
- 設計AOP ⭐⭐⭐⭐
    - 我們可以針對Schedule的方法來設計一個注入系統身分的切面。
    - 而首先要先設計註解 : 
    ```java=
    @Target(value = ElementType.METHOD)
    @Retention(RetentionPolicy.RUNTIME)
    public @interface RunAsSystem {
        String role() default "ROLE_ADMIN";
    }
    ```
    - 接著可以來處理放入身分的切面設計 : 
    ```java=
    @Aspect
    @Order(1)
    @Component
    public class RunAsSystemAspect {
    
        // 限定只在task中生效
        @Around("@annotation(org.system.aop.annotation.RunAsSystem) && execution(public * org.system.task.*.*(..))")
        public Object grantedSystemIdentification(ProceedingJoinPoint joinPoint) throws Throwable {
            
            MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
            RunAsSystem asSystem = methodSignature.getMethod().getAnnotation(RunAsSystem.class);
            String role = asSystem.role();

            SecurityContextHolder.getContext().setAuthentication(
                new UsernamePasswordAuthenticationToken(
                        "SYSTEM_SCHEDULER",
                        null,
                        // 給予指定的role
                        List.of(new SimpleGrantedAuthority(role))
                )
            );

            try {
                return joinPoint.proceed();
            }finally {
                // 無論執行結果，最後一定要清空Context
                SecurityContextHolder.clearContext();
            }
        }
    }
    ```
    - 由於排程執行緒本身就與請求的執行緒錯開，所以不用擔心Context混用問題，但最後一定要clearContext，以避免下次請求帶有上次身分的問題！
 
## Day244
#### 學習重點 : Spring Security - 如何解決非同步執行緒的身分問題？
- 關於身分的傳遞 ⭐⭐⭐⭐
    - 在Spring中，當我們帶著Http請求進入Spring中，會有一條執行緒處理該請求，然而當該請求的業務邏輯中有「非同步執行緒處理其餘事項」的時候就會導致「身分丟失」！
    - 我們可以利用兩種方式來解決 : 
        - 直接把User當參數傳入
        - 利用 `DelegatingSecurityContextAsyncTaskExecutor` 來完成SecurityContext包裝。
    - 儘管第一種方式就很好了，但在某些情況還是會用到第二種方式。
- 何謂DelegatingSecurityContextAsyncTaskExecutor？ ⭐⭐⭐⭐⭐
    - 簡單來說，它可以將呼叫Async標註之方法的身分塞入子執行緒的SecurityContextHolder中。
    - 而我們會透過設置AsyncConfig時，以其作為實作TaskExecutor的Bean。
        - 預設是一般的ThreadPoolTaskExecutor，但我們用 `DelegatingSecurityContextAsyncTaskExecutor` 再包裝一層。
    ```java=
    @EnableAsync
    @Configuration
    public class AsyncConfig {

        @Bean("taskExecutor")
        public Executor taskExecutor(){

            ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
            taskExecutor.setCorePoolSize(5);       
            taskExecutor.setMaxPoolSize(10);       
            taskExecutor.setQueueCapacity(100);    
            taskExecutor.setThreadNamePrefix("async-task-");
            taskExecutor.initialize();
            
            return new DelegatingSecurityContextAsyncTaskExecutor(taskExecutor);
        }
    }
    ```
- 簡易實作 ⭐⭐⭐⭐
    - 我這邊用原本「建立訂單產生通知」的業務邏輯來開刀！
    - 原本的流程 : 送出訂單 ➝ 產生訂單 ➝ 送出訂單成立通知
    - 現在我將產生訂單後的「通知」作為子執行緒分出去，並使用SecurityContextHolder取出user。
    ```java=
    // 這邊使用taskExecutor的Bean作為處理非同步執行緒的動作
    @Async("taskExecutor")
    public void sendNotifications(String msg, Boolean is_read, Instant create_time){
        notificationDao.save(Notification.builder()
                        .user((User) SecurityContextHolder.getContext().getAuthentication().getPrincipal())
                        .msg(msg)
                        .is_read(is_read)
                        .create_time(create_time)
                        .build());
    }
    ```
    - 當我送出訂單時呼叫sendNotifications會走以下流程 : 
        - 1️⃣ 發現 `@Async` 將該方法包裝成Runnable
        - 2️⃣ 送交 `DelegatingSecurityContextAsyncTaskExecutor` 取出原執行緒的身分並送交TaskExecutor執行。
        - 3️⃣ 進入子執行緒，利用SecurityContextHolder取出身分後執行業務邏輯。
    - 如果要取得任務回傳的話，則得使用 `CompletableFuture` 來包裝。
    - 然而一般通知可以使用 `void` 即可。
- 實測結果 : ⭐⭐
    - 成功寄送通知到正確的user身上。
    ![image](https://hackmd.io/_uploads/rJm3rrNdGx.png)

## Day245
#### 學習重點 : Spring Schedule、Security兩大元件心得總結
- 關於Schedule 🌟🌟🌟🌟🌟🌟🌟
    - 在前半個月，我花了一些時間在處理ScheduleExecutorService的原生設計，並延伸到了Spring中所設計的TaskScheduler、TaskExecutor介面。
        - TaskScheduler : 排程介面，預設時作為 `ThreadPoolTaskSchduler`。
        - TaskExecutor : 執行緒介面，接收、執行任務，常用實作 `ThreadPoolTaskExecutor`。
    - 進入到 `@Scheduled` 後，則是注重在ScheduleConfig的設置，並以「訂單未付款」為題，將Schedule搭配Service來自動取消（以cron排程）。
        - 其中有運用到Join Fetch預載入的功能！使取消訂單後在restock商品時，不會產生 `N+1` 問題！
        - 最後收尾則是利用Optimistic Lock、ShedLock來處理資源鎖與排程鎖的問題！
        - **基本上三層架構就是** : ShedLock處理重複排程執行問題 ➝ Join Fetch + Batch批次處理訂單 ➝ Optimistic Lock處理資源Version問題。
    - 接著處理了關於Transactional與Scheduled的標註分離，讓單批次資源rollback不影響到其他批次。
    - 最後自己再設計一個Notification，當訂單取消後可以寄送通知到使用者通知欄中！
- 關於Security 🌟🌟🌟🌟🌟🌟🌟
    - 其實Security真的超乎我的想像，生態系有夠龐大，我感覺再學30天也學不完，可能要專門找老師來，不然很難自己學會owo。 
    - 首先我先比對自己設計的 `切面` 與 Spring Security的設計範圍 : 
        - 自己設計的切面驗證 : 在Contoller前後設下檢查，但格式不統一，且每種驗證需要自己設計。
        - Spring Security : 請求進入Spring後即攔截，經歷FilterChain（WebSecurity），MethodSecurity，直到出Controller都有防線，每種驗證都有介面與實作，**擴充性跟統一性高**。
    - 接著花了些時間在身分的驗證與包裝 `UserDetails`，以及 `Authentication`。
    - 再來處理了自己的Jwt驗證方式，並理過一次FilterChain的順序。
    - 最後進入MethodSecurity，這邊其實跟自製AOP很像ww，但格式的設計更加統一，且有處理到資源性問題。
    - 最後利用MethodSecurity的 `PreAuthorize` 來設計權限過濾！
        - 其中設計的Evaluator則是將驗證流在業務邏輯外，減少DB查詢次數，**使職責、關注點分離**，降低單元測試複雜度。
        - 接著加上FilterChain的Exception攔截，抓取Filter層的驗證/權限例外！
- 心得 🌟🌟🌟🌟🌟🌟🌟
    - 很高興有一起參與鐵人賽30天的活動，不然平常自己寫筆記都隨便亂寫ww，這次活動讓我重拾過去的熱情，而學習的旅途不會停止，我會繼續完成學Java的系列！

## Day246
#### 學習重點 : 關於購物車 - 設計新增/減少物品數量
- 前言 ⭐
    - 從鐵人賽回歸專案設計後，我會把每個實體的功能都盡量完善，把最後的專案part-3收尾！接著就可以來準備朝其他面向、框架學習了！
- 設計 : 新增/減少物品數量 ⭐⭐⭐⭐
    - 當物品加入購物車後，我希望能夠在購物車頁面中，設計加減API來直接針對商品做動作。
    - 由於我們有Security提供的AuthenticationPrincipal，因此不用再做userDao的find動作哩~
    - 以下是我的設計 : 
    ```java=
    // CartController
    @PutMapping("/item")
    public Response<CartResponse> changeCartItemQuantity(@AuthenticationPrincipal User user, @RequestBody ChangeCartItemQuantityRequest changeCartItemQuantityRequest){
        return cartService.changeCartItemQuantity(user, changeCartItemQuantityRequest);
    }
    ```
    ```java=
    // CartService
    @Transactional(rollbackFor = Exception.class)
    public Response<CartResponse> changeCartItemQuantity(User user, ChangeCartItemQuantityRequest changeCartItemQuantityRequest){
        if (changeCartItemQuantityRequest.getCart_item_id() == null || changeCartItemQuantityRequest.getQuantity() == null) return new Response<CartResponse>("1", "Failed", null);
        
        Cart cart = cartDao.findByUser(user).orElseThrow(() -> ResourcesException.of(ErrorCode.CART_NOT_FOUND));
        CartItem  cartItem = cart.getCartItemList().stream()
                .filter(ci -> changeCartItemQuantityRequest.getCart_item_id().equals(ci.getId()))
                .findFirst()
                .orElseThrow(() -> ResourcesException.of(ErrorCode.CART_ITEM_NOT_FOUND));

        int newQuantity = cartItem.getQuantity() + changeCartItemQuantityRequest.getQuantity();

        if (newQuantity > cartItem.getMerchandise().getStock()) {
            return new Response<>(CartResponseEnum.OUT_OF_STOCK, new CartResponse(cart));
        }

        if (newQuantity <= 0) {
            cart.getCartItemList().remove(cartItem);
            Cart savedCart = cartDao.save(cart);
            return new Response<>(CartResponseEnum.SUCCESSFULLY_REMOVED, new CartResponse(savedCart));
        }

        cartItem.setQuantity(newQuantity);
        Cart savedCart = cartDao.save(cart);
        return new Response<>(CartResponseEnum.SUCCESSFULLY_ADD, new CartResponse(savedCart));
    }
    ```
- 關於OSIV與Transactional的那回事 ⭐⭐⭐⭐⭐⭐⭐⭐⭐
    - 若我們沒有加上 `@Transactional`，則執行完 `cartDao.findByUser` 後，資料庫連線就會斷開，使得後續 `cart.getCartItemList().remove(cartItem);` 會報錯 `LazyInitializationException: : could not initialize proxy - no Session`。
        - **每次find** 都會經歷一次資料庫連線生命週期（建立、斷開）。
        - 因此一個方法中可能會多次建立、斷開連線。
    - 當我們針對一個實體的Lazy成員加載（getter）時，會觸發資料庫查詢，但資料庫連線早早在find完就關閉連線了，因此報錯。
    - 然而Spring有一個特殊的功能 `OSIV(Open Session in View)`，確保請求開始到結束途中，資料庫都保持連線（Session），因此就算我們不加交易事務註解，也不會報錯。
    - 然而然而，OSIV在一般企業級專案中，都會 **選擇關閉**，這是因為其十分占用資源！
    - 因此我們一定得確保 `@Transactional` 有加註在含有Lazy觸發的method上，確保呼叫該method時，**選擇保持連線**。
- 成果展示 : ⭐
    - 現在有UI按鈕可以呼叫 `/item` APIㄌ。
    ![image](https://hackmd.io/_uploads/BkqZkJPuzg.png)

## Day247
#### 學習重點 : 關於購物車 & 訂單 - 設計清空購物車與取消訂單
- 清空購物車、取消訂單 ⭐⭐
    - 功能本身就不多寫了，反正就是那樣，給大家看我的邏輯就好，把重點放在一些設計巧思比較重要owo。
    - 關於清空購物車 : 
    ![image](https://hackmd.io/_uploads/HJVPkE_uMl.png)
    - 關於取消訂單 : 
    ![image](https://hackmd.io/_uploads/Hke21EOuzl.png)
    ![image](https://hackmd.io/_uploads/BJJsJNu_zx.png)
- 巧思 - ResponseEnum介面化、OrderResponseEnum設計 ⭐⭐⭐⭐⭐
    - 首先是關於Response的多載，在我的Response中，有兩種函式 : 
        - `Response<T>(String rc, String rm, T data)`
        - `Response<T>(CartResponseEnum cre, T data)`
    - 原本只是設計給Cart用，但是我發現Enum可以實作介面，因此我將Response改成 : 
        - `Response<T>(ResponseEnum re, T data)`
        - 設計一個ResponseEnum介面 : 
        ![image](https://hackmd.io/_uploads/HJppgVuuzx.png)
        - 再由實體ResponseEnum去實作 : 
        ![image](https://hackmd.io/_uploads/S1MWbNuOfl.png)
    - 而透過上述的介面，我就可以去寫每個實體的回傳結果拉～
    - OrderResponseEnum : 
    ![image](https://hackmd.io/_uploads/ByC4ZEuOfx.png)
- 成果展示 : ⭐
    - UI多了「清空購物車」、「刪除訂單」:
    ![image](https://hackmd.io/_uploads/B1VF-Vu_Mg.png)
    ![image](https://hackmd.io/_uploads/Hy8FZEO_Mg.png)
