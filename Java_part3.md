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
