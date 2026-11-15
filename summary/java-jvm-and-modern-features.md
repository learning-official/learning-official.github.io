# JVM 根源、IO、String 與設計模式

## Day91
#### 學習重點 : 序列化與Serializable簡單理解
- 怎麼理解序列化？ ⭐⭐⭐⭐⭐
    - 序列化是一種轉譯的概念，在程式語言當中，物件是主角，序列化的功能就是將物件轉 **成電腦所認得的格式**（btye stream），其中當然就帶有變數、方法。
    - 尤其Java又是以物件導向為主軸，因此序列化在Java中算是很重要的一環。
    - 我自己感覺Serializable跟Java的 `.class檔` 處理模式蠻像的？都是轉譯成二進制，不過一個是在轉譯物件，一個用來轉譯源代碼。
- 現在Serializable遇到的問題 ⭐⭐⭐⭐⭐⭐⭐
    - 我在爬文的時候有得到一個不錯的結論 :
        - **更新問題** : 由於Serializable限制在Java程式之間的溝通，且受限在版本差異、函式庫差異等問題下，造成很多不必要的麻煩。
        - **支援問題** : 目前程式庫中也不一定支援與 `ObjectOutputStream`、`ObjectInputStream` 這兩個API互動，但這又是Serializable的核心，因此yoyoyo受限住。
        - **更好的格式** : 像是JSON就是目前最好用的格式，儘管Serializable可以更好儲存Java物件的完整性，但JSON還是利大於弊！

## Day92
#### 學習重點 : Java File IO
- 如何寫檔？ ⭐⭐⭐⭐
    - 使用 `BufferedWriter`，以FileWriter作為參數傳入BufferedWriter作為物件執行的的目標檔案！
    - 最後需要配上 `.close()` **關閉Buffer中的資料輸入狀態**！
    ```java=
    public class Main {
    
        public static void main(String[] args){
            try {
                BufferedWriter writer = new BufferedWriter(
                    new FileWriter("FileIO\\output.txt")
                );
                writer.write("Hello World!");
                writer.write("\nSecond line!");
                writer.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
    ```
- 如何讀檔？ ⭐⭐⭐⭐⭐
    - 以readLine作為讀取的主要功能，存入String line中，這邊的寫法很特別！先賦值在判斷是否終止迴圈！
    ```java=
    public class Main {
    
        public static void main(String[] args){
            try {
                BufferedReader reader = new BufferedReader(
                    new FileReader("FileIO\\output.txt"));
                
                String line;
                while ((line = reader.readLine()) != null){
                    System.out.println(line);
                }
                reader.close();
                
            }catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
    ```

## Day93
#### 學習重點 : Java IO - 甚麼是Stream?
- Stream是甚麼？ ⭐⭐⭐⭐⭐
    - 在Input跟Output中，程式的運作底層原理就是靠著Stream在運作！但到底甚麼是Stream（流）呢？
    - 所謂的Stream其實就是「一串長度不一的**Bytes資料序列**」，因此Input Stream讀取資料序列，而Output Stream就是將資料序列呈現or寫入！
    - 如果用簡單的比喻，像是**鍵盤**就是一個**Input Stream**，讀取使用者按下的按鍵，而**螢幕**則是**Output Stream**，呈現出使用者按下按鍵出現的畫面！
- Java的IO Stream分類 ⭐⭐⭐
    - Java中，Input Stream與Output Stream，**都是抽象類別**！因此會有許多實作，像是昨天看到的**FileWriter、BufferedWriter都是**！底下這張圖就是Stream的架構！
    ![image](https://hackmd.io/_uploads/HkdZ0LTjWg.png)
    - Made By [Kody Simpson](https://www.youtube.com/watch?v=7dmIVusn8mk&list=PLfu_Bpi_zcDO4CdNYNS2Wten1vLuQfgp7&index=1)

## Day94
#### 學習重點 : Java IO - OutputStream與PrintStream
- OutputStream介紹 ⭐⭐⭐⭐
    - 在寫程式之前，必須先理解輸出流是怎麼運作的，首先當我們write資料序列進輸出流的時候，資料會先「**流**」進Buffer區，等到flush後，資料才會被「**沖出**」至指定區域！
- PrintStream是甚麼？ ⭐⭐⭐
    - 其實它就是我們最常使用的 `System.out...` 啦~ 可以說PrintStream就是OutputStream的一個實作子類別，它指定flush沖出的地方是「Terminal」！ 
    - 因此平常我們所使用的輸出形式 `System.out.println(參數)` 就是將 **參數丟進Buffer後再flush出來的**！
- 如何實作呢？ ⭐⭐⭐⭐⭐⭐
    - 以下是我整理的三個重點 : 
    ```java=
    package FileIO;

    import java.io.IOException;

    public class OutputStreamT{

        public static void main(String[] args){
        
            // 重點一 : 根據ASCII table輸出形式
            int thing = 75;
            System.out.write(thing);
            System.out.flush();
            
            // 重點二 : byte array被write進Buffer區再一次flush出來
            for (int i=32; i<127; i++){
                System.out.write(i);
            }
            System.out.flush();

            // 重點三 : 使用wrtie(byte[] b)時，需加try-catch
            // 由於該方法並沒有被PrintStream先攔截，因此必須自行攔截
            try{
                String name = "JavaLearningEveryday";
                byte[] bytes = name.getBytes();

                System.out.write(bytes);
                System.out.flush();
            }catch (IOException ex){
                System.out.println(ex);
            }

        }
    }
    ```

## Day95
#### 學習重點 : Java IO - InputStream
- InputStream是甚麼概念？ ⭐⭐⭐
    - 記得在剛開始學習Java的時候，有使用到一個Scanner的物件！而建構式當中所放入的就是 **輸入流** `System.in`！
    - 因此InputSream相對應OutputStream就是在 **讀取** 而不是寫入！
    - 昨天我在write放參數，這個參數本身就是已經被建置好的資料序列，而輸入流就是 **讀取序列後放入write** 當中作為參數。
- 如何使用InputStream？ ⭐⭐⭐
    - 首先當然就是利用 `System.in` 來做事啦～
    - 基本架構如下 : 
    ```java=
    try{
        int[] array = new int[10];
        for (int i=0; i < array.length; i++){
            // 讓使用者於Terminal輸入字串，一個一個wrtie進Buffer區
            array[i] = System.in.read();
            System.out.write(array[i]);
        }
        // 最後一次flush出來
        System.out.flush();
    }catch (IOException ex){
            System.out.println(ex);
    }
    ```
- 如何搭配FileInputStream？ ⭐⭐⭐⭐
    - 首先要先宣告FileInputStream物件來作為讀取File的主要工具。
    ```java=
    FileInputStream input = new FileInputStream("output.txt");
    // 這邊使用的available可以讀取檔案中有多少的字元
    byte[] arrayOfData = new byte[input.available()];
    // 將讀取到的字元放進array中
    input.read(arrayOfData);
    ```
    - 接著將讀取到的byte陣列放入write參數當中，再沖出至Terminal。
    ```java=
    System.out.write(arrayOfData);
    System.out.flush();
    ```

## Day96
#### 學習重點 : Java IO - FileStream
- FileStream如何使用？ ⭐⭐⭐
    - 透過前幾天的IO練習，其實可以大概知道怎麼用，首先當然就是先建立FileIO的物件，接著透過讀取與與寫入進Buffer區，再來就是flush啦~
    - 實際架構還是得搭配try-catch，因為對於write、read來說需要有IO例外的攔截！
    ```java=
    FileInputStream fin = null;
    FileOutputStream fout = null;
    try{
        fin = new FileInputStream(new File("fin.txt"));
        // 這邊設置append屬性是true，亦即不是覆蓋而是新增
        fout = new FileOutputStream(new File("fout.txt"), true);
        
        // 這邊使用readAllBytes直接抓取檔案所有的字元
        // 若擔心檔案過大，也可以使用while(data != -1)來做偵測是否為檔案結尾
        byte[] text = fin.readAllBytes();
        for (int i=0; i<text.length; i++){
            fout.write(text[i]);
        }

        fout.flush();
        
    }catch (IOException ex){
        System.out.println(ex);
    }finally{
        try{
            if (fin != null) fin.close();
            if (fout != null) fout.close();
        }catch (IOException ex){
            System.out.println(ex);
        }
    }
    ```
- 關於Buffer區域問題 ⭐⭐⭐⭐⭐⭐⭐
    - 我在學習FileIO的時候有遇到一個問題 --> **Buffer區容易搞混**，所以我稍微列出了關於Stream的Buffer區種類 : 
    - **PrintStream** : 處理 `System.out` 的緩衝區。
    - **InputStream** : 處理 `System.in`，當我read時會去Terminal拿取資料序列，待flush被呼叫或者close呼叫才會清空Buffer。
    #### 關於FileOutputStream的Buffer區
    - 在程式碼當中 `fout.flush` 其實沒什麼作用！在FileOutputStream中，**並沒有Override flush方法**，也沒有自己的Buffer區，所以寫這行只是一個習慣而已w
    - 深究該類別的寫入方式，其實就是單純**讀一個byte寫一個byte**，因此在使用fout.write時，可以想像他自動幫我們flush進去了！

## Day97
#### 學習重點 : Java IO - FilterStream
- 甚麼是FilterStream？ ⭐⭐⭐⭐
    - 當我們針對IO Stream做讀取寫入時，可能想對某些字元做更改，或者說「**攔截**」，這時候會靠著**FilterStream來完成**。
    - 儘管也可以直接在Main檔案中進行實作，但經過SpringBoot的洗禮，我也大概知道為何要有這種「職責分離」的概念啦～
- 如何使用FilterStream？ ⭐⭐⭐⭐
    - 在Stream大分類中，Filter（IO）Stream是一個父類，其中包含許多子類是專門處理特別的過濾，像是大約一個禮拜前所看過的Buffered（IO）Stream就是一種過濾類別！
    - 而我們也可以自行繼承Filter（IO）Stream來「客製化」過濾流。
    ```java=
    public class CustomFilterStream extends FilterOutputStream{
        
        // 因為有繼承，所以會用到super啦~~
        // 因為要過濾，當然要有「被過濾者」out本人傳進來
        public CustomFileStream(OutputStream out){
            super(out);
        }
        
        // 這邊Override父類的寫入功能，並加上自己的過濾邏輯
        @Override
        public void write(int b) throws IOException{
            if (b >= '0' && b <= '9'){
                super.write(b);
            }else{
                super.write('?');
            }
        }
        
        @Override
        public void write(byte b[], int off, int len) 
            throws IOException{
                for (int i=off, i < off+len; i++){
                    this.write(b[i]);
                }
            }
    }
    ```
- 在Main檔使用FilterStream ⭐⭐⭐⭐
    - 在Main檔案中，自然會需要兩個物件，一個是寫入流，一個是過濾流！
    ```java=
    public class FilterStreamT {

        public static void main(String[] agrs){

            FileOutputStream fout = null;
            CustomFilterStream filter = null;
            try{
                fout = new FileOutputStream("filter.txt");
                filter = new CustomFilterStream(fout);
                int i;
                while ((i = System.in.read()) != 'x'){
                    filter.write(i);
                }
            }catch (IOException ex){
                System.out.println(ex);
            }finally{
                try{
                    if (filter != null){
                        filter.close();
                    }
                }catch (IOException ex){
                    System.out.println(ex);
                }
            }
        }
    }
    ```
    - 但這邊要注意的是，過濾流中所連結的Buffer區，其實就是寫入流的Buffer區！
    - 我覺得這張圖畫得很好，偷過來用一下w。
    ![螢幕擷取畫面 2026-04-07 133952](https://hackmd.io/_uploads/Hk-y3_xWMl.png)
    [- Made By Kody Simpson](https://www.youtube.com/watch?v=W5OChVAoYm0&list=PLfu_Bpi_zcDO4CdNYNS2Wten1vLuQfgp7&index=5)
- 明天我應該會先把BufferedStream完結之後再來研究try-resources的用法！

## Day98
#### 學習重點 : Java IO - BufferedStream
- BufferedStream的用處 ⭐⭐⭐⭐⭐⭐⭐⭐
    - 回到剛學IOstream的時候，我對於BufferedStream的理解就是讀取寫入。
    - 但經歷過這禮拜的學習，我對於Buffer與Filter有更深的理解了！
    - 所謂的BufferedStream，對於電腦讀取與寫入有專門處理方式，增加處理效能！
- BufferedStream是怎麼運作的？
    - 1️⃣ 首先，BufferedStream在物件建立的時候，會開啟一個byte array用於「暫存資料」。
    - 2️⃣ 當我們執行read、write等動作，BufferedStream會先一次性將一坨資料序列先送進byte array，接著再交由read、write讀寫。
    #### 為甚麼要這樣做？
    - 會這樣做的原因很簡單 : 「省去 **來回** 硬碟拿取的次數」。
    - **硬體原因** : 由於「從硬碟拿資料」 與 「從RAM拿資料」有**實質上的速度差異**，因此一次從硬碟拿一整坨byte丟到RAM會比一個一個拿還快上N倍！
- 流程圖 : ⭐⭐⭐⭐⭐⭐
    - 我又偷了兩張圖來這邊ww，這個人真的講得很好，推一個！
    - 當我們針對Output的Buffered類別過濾時，會先 **等byte array滿了自動flush** 或者 **被手動flush後**，才會再去硬碟拿資料！
    ![螢幕擷取畫面 2026-04-08 222343](https://hackmd.io/_uploads/r1j3jOgbzl.png)
    - 這是Input的部分 : 
    ![螢幕擷取畫面 2026-04-08 222433](https://hackmd.io/_uploads/ry4TouxbGl.png)
    [Made By - Kody Simpson](https://www.youtube.com/watch?v=baHz_RmMt5I&list=PLfu_Bpi_zcDO4CdNYNS2Wten1vLuQfgp7&index=6)
- 實際程式架構 : ⭐⭐
    - 其實跟昨天的Filter一樣，只是名稱換一下，過濾功能不同而已w
    ```java=
    public class BufferedStreamT{

        public static void main(String[] args){

            BufferedInputStream input = null;
            BufferedOutputStream output = null;
            try{
                input = new BufferedInputStream(
                    new FileInputStream("bufferIn.txt")
                );
                output = new BufferedOutputStream(
                    new FileOutputStream("bufferOut.txt")
                );

                int in;
                while ((in = input.read()) != -1){
                    output.write(in);
                }
                output.flush(); // 雖然會close的時候會flush，但還是寫一下w
            } // ...後續catch、finally省略
        }
    }
    ```

## Day99
#### 學習重點 : try-with-resource
- 它是甚麼？ ⭐⭐⭐⭐⭐
    - 簡單來說，它是來 **簡化finally的語法**！
    - 在Java中，有個功能介面叫做 `AutoCloseable`，像是IOstream要關閉時，會使用 `.close()`，而該方法其實就是實作了 `AutoCloseable` 介面。
    - 而在try-with-resources當中，我們會在 **try當中傳入參數** --> `try(參數)`，而該參數就是實作了 `AutoCloseable` 的**物件**！在try區塊中執行完後，會 **自動** 執行close方法，不需要再寫finally來close！
- 它除了簡化用法還可以幹嘛？ ⭐⭐⭐⭐
    - **異常覆蓋** : 當我們使用一般的try-catch-finally時，會在finally又寫一個try-catch去抓close的報錯，這樣當主try中報錯時，而close又報錯，噴出的Exception會覆蓋掉了原本try當中的異常，只顯示close的異常。
    - **逆序關閉** : 當使用多層Stream嵌套時，若先關閉內層再關閉外層會產生錯誤，但如果是在try-with-resource中，會確保最外層的Stream先關閉，**依序往內關**！
- 實際怎麼寫？ ⭐⭐
    - 實際寫法很簡單 : 
    ```java=
    try(BufferedInputStream input = new BufferedInputStream(
            new FileInputStream("bufferIn.txt"))){
        
        // ...內容省略
        // 最後會自動close : bufferedstream --> filestream
        
    }catch (IOException ex){
        System.out.println(ex);
    }
    ```

## 🎉Day100
#### 學習重點 : BufferedReader & Writer & IOstream總結
- Reader & Writer 與 IOstream的差別在哪？ ⭐⭐⭐⭐
    - 首先必須認知到，兩者最大的差別在於 **讀取與寫入格式的不同**。
    - 由前幾天所認識到的IOstream可以知道，它是以「byte stream」做讀取寫入。
    - 而RW卻是以「character stream」在做處理的，其中靠著 `InputStreamReader`、`OutputStreamWriter` 將byte stream利用charset（字元集）轉成character後再丟進輸入輸出流當中。
- 關於編碼這回事 ⭐⭐⭐⭐⭐⭐
    - 在利用Charset轉碼時，Reader又是如何知道該怎麼轉呢？
    - 在現今，編碼系統常用Unicode作為語言綜合體（甚至是符號、emoji），UTF-8是最常被提及的，因為它綜合了不同語言的編碼。
    - Reader透過Unicode編碼的前輟去偵測要一次取幾個bytes來轉碼，像是中文字是3bytes，英文是1byte，此時利用英文跟中文的前輟不同，來去決定後續要再多取幾個byte！
- 最終型態 : BufferedRW ⭐⭐⭐
    - 綜合上週跟這週所學，並回到最初的起點，可以發現一開始用的BufferedRW就是結合了IOstream與Reader/Writer家族的型態。
    - 當然，在這邊的BufferedRW又會跟BufferedIO所開的buffer區不太一樣，BufferedRW的緩衝區會利用 `char[]` 來儲存，很好理解嘛！
    - 透過try-with-resource，簡化形式後，基本上就會長成下面這個模樣 : 
    ```java=
    public class Main {

        public static void main(String[] args){
            try(BufferedWriter writer = new BufferedWriter(
                new FileWriter("output.txt"))){

                writer.write("Hello World!");
                writer.write("\nSecond line!");

            }catch (IOException e) {
                System.out.println(e);
            }

            try(BufferedReader reader = new BufferedReader(
                new FileReader("output.txt"))){

                String line;
                while ((line = reader.readLine()) != null){
                    System.out.println(line);
                }

            }catch (IOException ex){
                System.out.println(ex);
            }
        }
    }
    ```
- Java IO總結 ⭐⭐
    - 稍微總結下這幾天的學習 : 
        - **分類** : Java IO中有兩大分類IOstream與Reader&Writer，兩者的差異在於讀取寫入的格式不同。
        - **流程** : 讀取將資料序列讀取後會流進指定區域（如Teminal、File），寫入會將資料序列流入「緩衝區」，待flush後流進指定區域。
        - **過濾器** : 繼承FilterStream後可以覆寫write、read等檔案，亦可使用內建的過濾器。
        - **Buffer過濾** : 該功能建立原因在於可以「減少來往硬碟讀取寫入的次數」，與output當中的buffer不同。 
        - **try-with-resource** : 簡化finally用法，讓實作autocloseable的物件可以自動.close，且可以追溯所有異常，不會被覆蓋。

## Day101
#### 學習重點 : Exception Handling（例外處理）簡介
- 例外處理簡介 ⭐⭐⭐
    - 其實在前100天，我已經寫了許多跟例外例外處理相關的程式碼！但都沒有認真的認識Java的Exception類別，藉著這幾天來好好的研究研究！
    - 首先，在Java中，任何的例外都繼承了 `Throwable` 類別，而其中有兩類，一類是不可被捕捉（catch）的**Error類別**，另一類就是可被catch的**Exception類別**！
    - 一般來說，Error之所以不被catch是因為這類別的問題通常是**不可預期的**，較不常發生在一般程式運行當中。
- Exception分類 ⭐⭐⭐
    - 在Exception中，有分成Unchecked Exception（未受檢例外）跟Checked Exception（受檢例外）。
    - 所謂的未受檢例外意旨當程式撰寫適當，**不需要catch也可以運作**，而受檢例外相對應的**必須在編譯前**，就**寫出catch敘述**！
    - 常見的未受檢例外就是Runtime Exception，而受檢例外就像是前幾天所看到的IOException！

## Day102
#### 學習重點 : Exception Handling（例外處理）實作上該注意甚麼？
- 縮小捕捉範圍 ⭐⭐⭐⭐
    - 當我們在使用catch捕捉例外時，若不使用精確的例外類別，而使用像Exception這種大範圍捕捉，儘管在自己測試上沒有問題，但要是放到與他人協作或者偵測特殊異常時，就會一個頭n個大了~
    - **錯誤捕捉** : 除了上述的問題之外，大範圍捕捉可能會捕捉到我們「不希望」捕捉到的例外，有些例外，我們會希望丟到外部處理，而不是在內部解決。像Dao層是負責與資料庫溝通，總不可能丟出一個 `SQLException` 給前端吧！因此我們會在Service處理過後丟出例外，再讓Spring利用ExceptionHandler截獲處理！
    - 作為一個開發者，要讓人讀得懂比寫code這件事還要麻煩的多，但這卻是不可避免的過程！
- printStackTrace的危險性 ⭐⭐⭐⭐⭐
    - 這是在Throwable所寫的方法，之所以會危險的原因有幾個 : 資訊洩漏、不好紀錄。
    - 資訊洩漏 : 當我們在catch中使用printStackTrace時，若錯誤發生，他會將發生錯誤的class名稱、method名稱都print出來，這樣被攔截後，**架構都被看光光了**ouo。
    - 不好紀錄 : 由於printStackTrace是印在Terminal，因此**不會被記錄在log日誌**，且程式關閉後就會被清理掉。
- Throw early、Catch late ⭐⭐⭐⭐
    - 若要做好一個例外拋出與攔截的程式，就應該要在程式前面先對可能產生的錯誤進行判斷是否拋出（Throw early），並交由後續的攔截系統統一處理（Catch late）！

## Day103
#### 學習重點 : Exception Handling（例外處理）CustomException
- 如何自製一個Exception？ ⭐⭐⭐⭐
    - **定義** : 首先要先定義這個例外是因「**甚麼錯誤**」而需要丟出例外。這邊假設傳入一個年齡參數，若年齡為負，則拋出例外（`AgeLessThanZeroException`）
    - **繼承** : 再來就是將類別製作出來並「繼承Exception」: 
    ```java=
    public class AgeLessThanZeroException extends Exception{}
    ```
    - **建構子多載** : 接著會利用到建構子多載以及繼承super去覆寫父類的建構子 : 
    ```java=
    public class AgeLessThanZeroException extends Exception{

        public AgeLessThanZeroException(){}

        public AgeLessThanZeroException(String msg){
            super(msg);
        }

        public AgeLessThanZeroException(Throwable cause){
            super(cause);
        }

        public AgeLessThanZeroException(String msg, Throwable cause){
            super(msg, cause);
        }
    }
    ```
    - 這邊可以看到多載建構子，msg很簡單嘛！當被throw new的時候，可以傳入訊息像是 : `Age must greater than 0！`，但Throwable物件又是甚麼呢？
    #### 例外引起例外 : Throwable多型應用
    - 這幾天的研究可以知道 `Throwable` 是整個例外處理類別的最上層，因此當我們傳入 `Throwable` 時，可以想到是利用 `Polymorphism 多型` 的概念吧~
    - 我們傳入的這個Throwable物件（a）代表著當前的例外是被（a）例外所引起的！
- 實際操作例外 ⭐
    - 這邊寫了個簡單的測試檔案 : 
    ```java=
    public class Main{

        public static void main(String[] args) 
            throws AgeLessThanZeroException{
            
            ageValidation(-1);
        }

        public static void ageValidation(int age) 
            throws AgeLessThanZeroException{
            
            if (age < 0){
                throw new AgeLessThanZeroException(
                    "Age must greater than zero!", 
                    new RuntimeException()
                );
            }
        }
    }
    ```
    - 拋出時，可以於參數加上其他例外，以及訊息，這樣若例外被丟出時，會在Terminal印出這樣的訊息 : 
    ![螢幕擷取畫面 2026-04-13 224605](https://hackmd.io/_uploads/HyOOjug-Gx.png)
- 更貼近的例外繼承 ⭐⭐⭐⭐⭐
    - 若我們繼承Exception，是否代表我們也可以繼承上層Throwable或者繼承下層更細的例外？
    - 可以！但不建議往上層進繼承，盡量 **繼承最貼近自身的例外**，像這邊可以繼承 `IllegalArgumentException` 取代 `Exception`，仔細看這個例外的簡介 : 
    ![螢幕擷取畫面 2026-04-13 225258](https://hackmd.io/_uploads/BJg5Ys_x-fx.png)
    - 很符合我們不希望傳入負的年齡的概念！

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
