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
