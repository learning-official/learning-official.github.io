# Spring Schedule 排程與分散式鎖

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
