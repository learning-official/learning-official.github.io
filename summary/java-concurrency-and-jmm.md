# 多執行緒、鎖與 JMM 記憶體模型

## Day81
#### 學習重點 : Thread & Runnable 基本理解
- 前言 ⭐
    - 從以前到現在，我對於這兩個詞都很不能理解，儘管知道是可以讓程式同時運作，但終究沒有實作經驗，因此我打算花個幾天來認真研究一下。
- Thread基本架構 ⭐⭐⭐⭐
    - 首先，需要建一個class用於繼承Thread，並 Override父類的 `run()` method。
    ```java=
    // Test.java
    public class Test extends Thread{
        
        @Ovveride
        public void run(){
            System.out.print("Thread test!");
        }
    }
    ```
    - 可以想像run是 **一條執行緒的進入點**(?
    - 接著在Main檔建立Test物件，並使用 `start()` 來 **啟動執行緒**。
    ```java=
    // Main.java
    public class Main{
        
        public static void main(String args[]){
            Test test = new Test();
            test.start();
        }
    }
    ```
- Thread - 暫停當前執行緒用法 ⭐⭐⭐⭐⭐⭐
    - 當我們有多條執行緒時，可以使用 `sleep()` 來暫停當前執行緒。
    ```java=
    // Test.java
    public class Test extends Thread{
        
        @Override
        public void run(){
            System.out.println("Thread test!");
            
            // 由於執行緒在暫停期間，可被中斷，因此要catch被中斷後該做甚麼動作
            try{
                Thread.sleep(1000); // 1000 ms = 1s
            }catch(InterruptedException e){}
        }
    }
    ```

## Day82
#### 學習重點 : Runnable的運用方式
- Thread跟Runnable的差別在哪？ ⭐⭐⭐⭐⭐⭐⭐⭐
    - 其實點進去這兩個.class檔就會發現，Runnable是一個 `FunctionalInterface`，而Thread是實作Runnable的一個檔案。
    #### Runnable比Thread好用的點在哪？
    - 1️⃣ **靈活性問題** 
        - Runnable是介面，由於Java不允許多重繼承，但 **可以多重實作**，因此使用implements相較於extends能增加類別的靈活性。
    - 2️⃣ **多工處理，可共用資料**
        - 若我有一個工作 (Task) 想要丟給多個執行緒去跑同一個Task物件。
        - 若繼承Thread，必需建立多個Task物件，按照 `task1.start()、task2.start()...` 這樣子。
        - 若我實作Runnable，則可以只建立一個task1，並 `new Thread(task1).start();` 很多次即可，其實質上只使用了task1一個物件。
    - 3️⃣ **關注點分離**
        - Thread 關心的是 **執行緒本身**。
        - Runnable 關心的是要 **執行的工作內容**。
    - 4️⃣ **關於Thread pool存放問題**
        - 它有點像我之前學的Spring IoC容器（? 在這邊，我們會將實作Runnable的 **類別丟到Thread pool裡面去存放**，後續的動作，我還沒學到，之後再來~
- Runnable的實際code寫法 ⭐⭐⭐⭐
    - 跟Thread大同迷你異，只差在它需要丟到Thread去start。
    ```java=
    public class Main {
    
        public static void main(String[] args) {
            for (int i=0; i<5; i++){
                // 這樣子是建立五個RunnableTest物件
                RunnableTest rt = new RunnableTest(i);
                new Thread(rt).start();
            }
        }
    }
    ```
    ```java=
    public class Main {
    
        public static void main(String[] args) {
            // 這樣子是則是建立一個物件，讓5個執行緒去跑同一個物件
            RunnableTest rt = new RunnableTest(1);
            for (int i=0; i<5; i++){
                new Thread(rt).start();
            }
        }
    }
    ```

## Day83
#### 學習重點 : Callable簡單理解使用方法
- Callable是甚麼？⭐⭐⭐⭐
    - Callable基本上跟Thread、Runnable是做同樣的事情 -> 多執行緒！
    - 但差別在於Callable有回傳值，且可以自定義型態，也就是泛型！
    - 直接來看程式碼比較清楚 : 
    ```java=
    import java.util.concurrent.Callable;

    public class CallableTest {
        
        // 這邊傳入整數的執行緒工作
        Callable<Integer> callable = () -> {
            int sum = 0;
            for (int i = 1; i <= 10; i++) {
                sum += i;
            }
            return sum;
        };

        public static void main(String[] args) {
            CallableTest ct = new CallableTest();
            try {
                // 使用call啟動執行緒！
                System.out.println(ct.callable.call());
            } catch (Exception e) {}
        }
    }
    ```

## Day84
#### 學習重點 : Callable的用法
- Callable與Runnable不同之處 ⭐⭐
    - 儘管Callable與Runnable都是用於多執行緒，但Callable可以在task結束後，**回傳自訂義型別的值or物件**。
- Callable回傳以及後續用法 ⭐⭐⭐⭐
    - 當我們需要取得Callable實作下所回傳的東西時，需要用到Future類別。
    ```java=
    Callable<Integer> c = () -> {實作內容省略...}
    Future<Integer> f = new Executors.
        newSingleThreadExecutor().submit(c)
    ```
    - 先將Callable送進 **執行緒池** 中（Executor這部分先省略，我之後再來研究ww），接著將Callable的 **回傳的結果送進Future物件** 中。
    - 接著就可以利用Future物件去取值啦~
    ```java=
    System.out.println(f.get()); // 回傳的型態由Callable決定
    ```

## Day85
#### 學習重點 : Executors處理
- 甚麼是ExecutorService？⭐⭐⭐
    - 可以想像ExecutorService是一家公司，專門 **接收並管理Task**，而這個接收Task的地方就是前兩天一直提到的ThreadPool。
    - 當我們建立了Callable物件時，需要submit該物件到ThreadPool當中排隊等待工作。
    ```java=
    public static void main(String[] args) throws Exception{
        
        // 該方法在Executors中會回傳Service物件，不須再自行new！
        ExecutorService service = Executors.newFixedThreadPool(3);
        
        // 使用List存放Callable所回傳的Future物件
        List<Future<Integer>> list = new ArrayList<>();

        for (int i=0; i<5; i++){
            CallableTest<Integer> ct = new CallableTest<>(i+10);
            list.add(service.submit(ct));
        }
        
        service.shutdown();
        
        for (Future<Integer> future : list){
            System.out.println(future.get());
        }
    }
    ```
    - 注意！儘管執行緒task會結束，但Pool不會關，因此我們需要自行shutdown該Service，否則它會一直占用資源。

## Day86
#### 學習重點 : 甚麼是Synchronize？
- Synchronize（同步）的意義 ⭐⭐⭐⭐
    - 在前幾天有提及到Runnable可以建立同一個物件 **但建立多執行緒** 運行同一個物件，若今天需對成員變數做更改時，**同時讀取、更動、寫入** 的時間有 **可能會衝突**，造成資料錯誤，此時就需要靠Synchronize來協調！
- Synchronize如何使用？ ⭐⭐⭐⭐
    - 它就像final、abstract一樣，直接加在method的前面！
    ```java=
    public class RunnableTest implements Runnable{ 
        // ...省略
        public synchronized void add() {
            this.num++;
        }

        @Override
        public void run(){
            // ...省略
        }
    }
    ```
    - 而它的作用性就在於 : 當 `thread_1` 進入到同步方法後，`thread_2、thread_3...` 若也要進入同步方法時，將需要排隊等待，以避免資料同時讀取寫入，造成錯誤！

## Day87
#### 學習重點 : Concurrency - 資源可見性問題 & Lock解方
- 甚麼是資源可見性問題？ ⭐⭐⭐⭐⭐⭐⭐⭐
    - 當利用Concurrency概念實踐Multithread的時候，會遇到一個問題 ➞ 當執行緒在**修改「共享資源」時**，其他執行緒能否 **「即時看見」這個修改成果**？
    - 一般來說，執行緒在操作共享資源時，會先從主記憶體 **複製一份** 到自己的工作區，運算完畢後，再更新到主記憶體 。
        - 但這會衍生出「**更新延遲**」的問題。
        - **更新延遲** ➞ 亦即昨天所理解到的，同時讀取運算造成的時間差，使得資料更新上出現重疊、錯誤 ➞ 進而衍生出了資源可見性問題。
        - 我們可以說操作共享資源的程式片段是 **Critical Section**。
- Lock解方 ⭐⭐⭐
    - 為了因應多執行緒操作共享資源所造成的可見性問題，Java提供了 `Synchonized` 關鍵字作為一種 **鎖住Critical Section** 的方法，使得該區塊 **同時間只能夠讓單一執行緒操作**。
    - **若Lock範圍過大**，反而失去了Concurrency的優點，因此通常在寫Synchronized時，只會針對「取用共享資源」的部分鎖住。
    ```java=
    public void add(){
        // 針對nun++上鎖，但method本身不是鎖
        synchronized(this){
            this.num++;
            System.out.println("task " + this.num);
        }
    }
    ```

## Day88
#### 學習重點 : Concurrency - 原子性問題
- 甚麼是原子性問題？ ⭐⭐⭐⭐⭐⭐
    - 在程式中，式子像是 `i++`、`new Thread()` ..雖然只有一行，但轉譯到組語中，就會變成像 : 「加載變數至CPU cache ➞ 計算、建立... ➞ 賦值 ➞ 回傳」這樣的步驟，由於那些式子「**在程式碼中**」已是**不可分割**，因此稱其為**原子性**。
    - 而問題就出在，當今天需要使用Concurrency多執行緒去操作同一物件時，就會有原子性問題出現，底下這張圖我覺得不錯！
    ![image](https://hackmd.io/_uploads/SkWFc3UiWl.png)
        - [(取自Java Concurrency #1: Concurrency 基礎)](https://medium.com/bucketing/java-concurrency-1-%E5%9C%A8%E9%96%8B%E5%A7%8B%E5%AF%ABcode%E5%89%8D%E5%85%88%E4%BA%86%E8%A7%A3%E4%B8%80%E4%B8%8Bconcurrency%E7%9A%84%E5%9F%BA%E7%A4%8E-8d1a6694eeff)
    - 由於在程式碼中，已經不能再切分 `cnt++` 了，因此在Java Memory Model中，會使用像前幾天所寫的Synchronized來將某些式子鎖住，流程不與其他執行緒交互作用！
- Java Memory Model（JMM）的概念 ⭐⭐⭐
    - JMM代表著「JVM如何管理Thread與Memory的互動」。
    - 1️⃣ Happen Before Order : 亦即當Write Action執行完畢後，Read Action能夠取得正確的變數值，此時就會利用到 `volatile` 關鍵字來 **確保可見性**，不過使用volatile會限制變數於主執行緒中操作，且不能解決原子性問題，因此使用不廣。
    - 2️⃣ 第二個概念也就是Synchronized的概念拉~ 我就不打了。

## Day89
#### 學習重點 : 應用Callable、ExecutorService、Future
- 搜尋File中的關鍵字 ⭐⭐⭐⭐⭐
    - 我請AI給我了一個功能以實踐我這幾天學習的執行緒功能！
    - 功能實作步驟如下 : 
        - 1️⃣ 建立 **搜尋** File中關鍵字數量的檔案（這邊不是主要學習部分，直接請AI給code）
        - 2️⃣ 建立Scanner要求使用者給予要搜尋的關鍵字，並建立 `Callable<Integer>` 後，submit該task後，以 `Future<Integer>` 接收。
        - 3️⃣ 建立ArrayList **接收多個Future**，使用 `future.get()` 等待並取得在該檔案中搜到的關鍵字數量，最後加總。
- 實際code展示 ⭐⭐
    - 這邊只展示了應用的部分，搜關鍵字的部分這邊先不研究w
    ```java=
    public class Main {

        static final ExecutorService service = 
            Executors.newFixedThreadPool(4);

        public static void main(String[] args){

            String[] files = {
                // ...省略
            };

            Scanner scanner = new Scanner(System.in);
            List<Future<Integer>> list = new ArrayList<>();

            System.out.println("請要尋找的關鍵字：");
            String keyword = scanner.next();

            for(String path : files){
                list.add(service.submit(new FileSearchTask(path, keyword)));
            }

            int count = 0;

            try{
                for (Future<Integer> future : list){
                    count += future.get();
                }
                System.out.println(
                    "這10個檔案共包含 " + count + " 個 " + keyword
                );
            }catch (InterruptedException iException){
                iException.printStackTrace();
            }catch (Exception e){
                e.printStackTrace();
            }

            service.shutdown();
            scanner.close();
        }
    }
    ```

## Day90
#### 學習重點 : 執行緒應用 - 小專案寄發mail解決
- 寄發mail問題 ⭐⭐⭐⭐
    - 在我之前的小專案實作中，寄發信箱這部分有個問題點，那就是當我POST後，頁面會進入待機狀態 ➞ **等成功sendmail後** 才會跳轉去更新密碼頁面。
    - 而多執行緒剛好可以解決這個問題，一部份讓主執行緒去做跳轉，再另外開出一個 **子執行緒去做sendmail** 的動作！
- ExecutorService小專案mail寄發應用 ⭐⭐⭐⭐
    - 我這邊使用了Runnable介面搭配ExecutorService來開子執行緒進行mail寄發。
    ```java=
    public UserService(){
        // ...省略
        this.mailExecutors = Executors.newFixedThreadPool(10);
    }

    Runnable r = () -> {
        // 子執行緒 : 寄發原始隨機密碼到使用者信箱
         try{
            mailService.sendMail(account, "Temporary password", rsg);
         }catch (Exception e){
             System.out.println("寄信失敗 : " + e.getMessage());
         }
    };
    mailExecutors.submit(r);
    ```

## Day275
#### 學習重點 : Java OCP測驗.5
- 測驗內容 🌟🌟🌟🌟🌟
    - 關於執行緒 ⭐⭐⭐⭐⭐⭐⭐
        - 先看底下程式碼 : 
        ```java=
        class A extends Thread {
            boolean flag = true;
            public void run() {
                System.out.println("Starting loop");
                while( flag ){ };   
                System.out.println("Ending loop");
            }
        }
        public class TestClass {
            public static void main(String args[]) throws Exception {
                A a = new A();
                a.start();
                Thread.sleep(1000);

            }
        }
        ```
        - 若我在TestClass利用 `a.flag = false` 想要嘗試跳脫執行緒迴圈，會發現執行緒沒有立即反應這個變化！
        - 這是因為「Happens-Before」特性，由於JVM將 `flag=true` 的 **狀態快取** 到了CPU暫存器中，因此就算在TestClass修改 `a.flag`，也不會讓 `a.flag` 立即更新使其跳出迴圈。我們可以針對flag加上 `volatile` 的修飾詞，使其每次被讀取都是直接從主記憶體抓取，**確保資源可見性**！
        - 因此能夠中斷無限迴圈的方式只有 `a.interrupt()`，但要注意在 `sleep`、`wait`、`join` 使用interrupt會出現InterruptedException。
