## Day255
#### 學習重點 : 關於使用者 - 管理員/賣家後台設計JPQL
- Query設計 ⭐⭐⭐
    - 由於在設計面板時，需要搜尋「待送貨訂單、營業總額...」等需要客製化SQL的動作，但由於我對SQL沒有很熟，所以請AI幫我了一下下w。
    - 而Query有一個需要注意的是 :  COALESCE、SUM的搭配。
    ```sql=
    COALESCE(SUM(o.quantity * m.price), 0.0)
    ```
    - 上面這段在定義的就是遍歷所有order中的quantity以及其merchandise的價格，並做乘積，那如果沒有找到，就會使用預設值"0.0"。
- 實作JPQL ⭐⭐⭐
    - 透過上述的兩個動作可以實現下列JPQL : 
    ![螢幕擷取畫面 2026-09-12 225943](https://hackmd.io/_uploads/r1XRGx7YGg.png)
    
## Day256
#### 學習重點 : 關於使用者 - 管理員/賣家後台設計Service結合前端UI
- 如何設計Service連接Controller？ ⭐⭐⭐⭐
    - 由於我設計了兩種DTO形式（Admin、Seller），因此在Controller設計getDashboard時，一個做PreAuthorize驗證ROLE_ADMIN，另一邊接收User直接進Service（因為人人都可以是Seller）。
    - 因此Service的getDashboard會有兩個形式，Dao也要有兩種版本 : ADMIN取得全域資訊、Seller取得區域資訊，因此Seller需要加一個 `WHERE user = :user...`，或者使用約定大於配置的函式聲明方式，加上 `AndOwner..(User user)`。
- Service設計 ⭐⭐⭐
    - 利用DTO加上 `@Builder`，可以更好的建立DTO。
    ![image](https://hackmd.io/_uploads/H1roQVEYfg.png)
    ![image](https://hackmd.io/_uploads/SJjn7NNYfx.png)
- UI設計 ⭐⭐
    - 這邊就請AI幫我結合兩種API形式的UI
    ![image](https://hackmd.io/_uploads/B1dE44EFGx.png)

## Day257
#### 學習重點 : 關於DTO - 使用mapStruct來轉換資料.1
- 關於MapStruct ⭐⭐⭐⭐⭐
    - 先說它與 `BeanWrppaer、PropertyDescriptor` 的差異 : 
        - BeanWrapper與PropertyDescriptor組合 : **使用Reflection**，不斷在 **Runtime時期** 取得Field資訊。
        - MapStruct : **在Compile時期**，根據Objects（source/target）之間的getter/setter做Mapping，可以自行設定ignore來忽略敏感資訊，防護資安。
        - 可以看出MapStruct省下了反射的動作，讓DTO與Entity之間轉換速度大增。
        - 且由於是在Compile時期，因此型別問題會直接報錯。
    - 它是一個Interface，而mapStruct會根據我們設定的Mapper annotation來自動生成一個 `XXXMapperImpl` 的實作類別，也是透過這樣的方式做映射。
- 實際操作MapStruct ⭐⭐⭐
    - 在這邊要注意的點是 : `MapStruct與Lombok` 的協同關係。
    - 由於MapStruct需要使用getter/setter，因此若有使用lombok的 `@Getter/@Setter` 註解，則需要讓maven配置時，將lombok置於mapstruct上面搶先啟動，同時還需要有lombok與MapStruct的橋接器。
    ```xml=
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.13.0</version>
        <configuration>
            <source>${java.version}</source>
            <target>${java.version}</target>
            <annotationProcessorPaths>
                <path>
                    <groupId>org.projectlombok</groupId>
                    <artifactId>lombok</artifactId>
                    <version>${lombok.version}</version>
                </path>
                <path>
                    <groupId>org.projectlombok</groupId>
                    <artifactId>lombok-mapstruct-binding</artifactId>
                    <version>${lombok-mapstruct-binding.version}</version>
                </path>
                <path>
                    <groupId>org.mapstruct</groupId>
                    <artifactId>mapstruct-processor</artifactId>
                    <version>${org.mapstruct.version}</version>
                </path>
            </annotationProcessorPaths>
        </configuration>
    </plugin>
    ```
    - 除了plugin的設定，dependency也要引入mapStruct : 
    ```xml=
   <dependency>
        <groupId>org.mapstruct</groupId>
        <artifactId>mapstruct</artifactId>
        <version>${org.mapstruct.version}</version>
    </dependency>
    ```
    - properties也要設定好version : 
    ```xml=
    <properties>
        <!-- ...省略... -->
        <org.mapstruct.version>1.6.3</org.mapstruct.version>
        <lombok-mapstruct-binding.version>0.2.0</lombok-mapstruct-binding.version>
    </properties>
    ```

## Day258
#### 學習重點 : 關於DTO - 使用mapStruct來轉換資料.2
- 實際設計Interface ⭐⭐⭐⭐⭐⭐
    - 我們可以透過加註 `@Mapper(componentModel = MappingConstants.ComponentModel.SPRING)` 來設計一個轉換DTO的Bean（因為設定ComponentModel為Spring，所以產生實作檔時會自動加上`@Component`）
    - 接著可以設計method接收source與target，其中target要加上　`@MappingTarget` 讓實作檔產生時，使用其setter來設定。
    - 若沒有加上註解，則mapStruct將其視為source並new出新的entity設定後回傳。
    - 而我們也可以利用 `@Mapping` 來設定ignore欄位，以下是完整Interface架構 : 
    ![image](https://hackmd.io/_uploads/ry9wqILYfl.png)
    - 而前往自動產生的實作檔如下 : 
    ![image](https://hackmd.io/_uploads/HJ1O9IIKGx.png)
    - 由於目前的UpdateRequest並沒有password等資訊，因此生成時，不會採取Mapping註解的動作（但Mapping還是加上去比較符合正確資安意識）。
- 應用於Service ⭐⭐
    - 接著就可以替換原本的BeanWrapper寫法啦～
    ![image](https://hackmd.io/_uploads/rkQd9ILFMe.png)

## Day259
#### 學習重點 : 關於商品 - 套用mapStruct用於編輯商品資訊
- 設計端點 ⭐⭐⭐⭐⭐
    - 首先先來設計 `[PATCH] /prod/{merchandiseId}`，透過該端點我們可以編輯商品的資訊 : `name、price、stock、description`。
    - 因此我們可以透過mapStruct加註 `nullProperty...IGNORE` 來忽略null欄位！
    - 但這邊要注意的是 : 使用primitive type，像是float、int等，就算前端沒填入，也**會被Java自動填入** `0.0f`、`0`，因此我們應該要包裝成 `Float、Integer` 來**避免沒填入卻被歸零的風險**。
    - 而在Controller則是需要先針對merchandiseId與User做驗證，確認該使用者是否能訪問資源。
- 實際實作 ⭐⭐
    - 底下是邏輯 : 
    ![image](https://hackmd.io/_uploads/rJKMejwtfe.png)
    ![image](https://hackmd.io/_uploads/HJcXeswKMl.png)
    ![image](https://hackmd.io/_uploads/Sy54xiDFze.png)
    - 再來是成果展示 : 
    ![image](https://hackmd.io/_uploads/BklLeoPFMl.png)

## Day260
#### 學習重點 : Swagger API建立
- Swagger API是甚麼？ ⭐⭐⭐
    - 簡單來說Swagger就是整合了專案中的api端點，變成一個文檔，供其他人使用，並提供路徑參數、RequestBody等設定以及範例。
    - 底下是我引入Swagger API dependency以及實際成果的圖 : 
    ![image](https://hackmd.io/_uploads/rJAEnbYKzx.png)
    - 我有順便更改了一些API路徑，使其更符合RESTful API的風格！
    - 由於今天比較忙，明天再來詳細探究這個東西吧！

## Day261
#### 學習重點 : Swagger API的authorize問題與解決
- 如何設置Swagger的authorize？ ⭐⭐⭐⭐
    - 一般情況下，Swagger在測試我們提供的API端點時，不會特別特別在Headers放Authorization的JWT token，因此這需要我們自行設定。
    - 我們會加入一個 `OpenApiConfig`，其中設定關於驗證的機制方案。
    ![image](https://hackmd.io/_uploads/SyYIcA9YMg.png)
    - 先向OpenAPI註冊一個components，標題為 `BearerAuth` 的驗證機制。接著指定要使用SecurityScheme.HTTP的方式，格式為JWT，且前綴會加上bearer。
    - 透過上述指定，到時候放入token就會根據 `Authorization: <scheme> <credentials>` 的方式去做存取！
- 成果展示 : ⭐⭐⭐ 
    - 在Swagger UI就會看到以下成果啦～
    ![image](https://hackmd.io/_uploads/SJmujC9KMx.png)
    ![image](https://hackmd.io/_uploads/SyAij0ctGl.png)

## Day262
#### 學習重點 : Response Code Enum最後完善、邏輯完善
- ResponseEnum的完善 ⭐⭐⭐⭐
    - 由於原本的Response Code大多都是Hardcod在Service當中，現在我將其完全抽離出來完整橫跨專案整體。
    - 同時在Response類中設計 
    ```java=
    public static <T> Response<T> of(ResponseEnum re);
    public static <T> Response<T> of(ResponseEnum re, T data)
    ```
    - 利用static factory可以多載建構式，我順便留了null data的設計，擴充Response code的靈活性！
- 邏輯完善 ⭐
    - 一般來說賣家不能夠新增自己的商品進自己的購物車，因此我在Service中加入了id檢查！
        - 原本是想針對「賣家不能新增自己的商品進自己的購物車」這件事作為PreAuthorize做資源攔截，但後來想想這樣要重複查兩次DB，且逾越PreAuthorize的職責（權限控制），因此選擇放入Service。

## Day263
#### 學習重點 : 針對超賣問題新增Retryable、樂觀鎖例外處理
- 超賣解決 ⭐⭐⭐⭐⭐⭐
    - 當兩個使用者同時下訂同個商品，可能會發生 **資源不足** 的問題，此時我們可以利用樂觀鎖加註Version的方式，看誰先搶到更新商品的Version，並讓沒搶到鎖的使用者rollback。
    - 但上述方式需要修正的情況在於 ➞ **當資源充足時**，兩人同時下訂，也會遇到Version衝突，此時某一方 **會被迫rollback**！但明明資源充足，應該兩人都可以順利下訂。
    - 因此我們需要設計retry，讓訂單送出時遇到的version問題不會因為一次衝突就馬上rollback，此時可以用Spring提供的 `@Retryable` 來設定嘗試次數...。
    ![image](https://hackmd.io/_uploads/r10tXWpYGx.png)
    - ❗注意 : createOrder的Version衝突例外是在 **整個事務結束後要commit上去時發生**，因此若Retry是內層而Transactional是外層，則commit時，Retry不會接收到例外，因此 **要注意階層關係**，所以我將Retry移至Controller層面做為外層處理。
- 樂觀鎖例外處理 ⭐
    - 我也在ExceptionHandler中攔截了 `ObjectOptimisticLockingFailureException`，讓衝突發生且Retry過後，可以正確回應前端的狀態！
    ![image](https://hackmd.io/_uploads/rJyZr-6Kfl.png)

## Day264
#### 學習重點 : 例外處理 - ErrorCodeEnum新增、ErrorCode介面化
- ErrorCode的修正與介面化 ⭐⭐⭐⭐⭐⭐⭐
    - 原本我的ErrorCode設計成像下方這樣 : 
    ```java=
    public enum ErrorCode{
        USER_NOT_FOUND(3001, "UserNotFoundError"),
        // ...
    }
    ```
    - 接收code、msg，但這就造成我需要在ExceptionHandler中比對各種errorCode再回傳不同HttpStatus，造成冗長的if-else語法，像下面那樣 : 
    ![image](https://hackmd.io/_uploads/HkV0oKAKzl.png)
    - 因此我設計將ErrorCode抽離成介面 : 
    ```java=
    public interface ErrorCode{
        String getErrorMessage();
        HttpStatus gethttpStatus();
    }
    ```
    - 並設計不同Exception的enum去實作ErrorCode，如 : `BusinessLogicErrorCode`、`ResourceErrorCode`，在其中定義例外訊息、狀態碼 : 
    ![image](https://hackmd.io/_uploads/r1vYjFAFzl.png)
    - 接著在Exception類別中建構式接收其enum，最後就可以在Handler輕鬆攔截並處理啦~
    ![image](https://hackmd.io/_uploads/HJIF2K0KGg.png)
    - 設計private方法接收ErrorCode介面並取得資訊
    ![image](https://hackmd.io/_uploads/Bkwr2t0KMl.png)
    - 攔截處理即可傳入「**有實作ErroCode介面**」的ExceptionEnum！
    ![image](https://hackmd.io/_uploads/H1g_3YCYzg.png)
    - 後來發現這種抽離功能，固定邏輯，只針對資訊上的更新就是所謂的OCP原則！

## Day265
#### 學習重點 : 使用者資料管控-電商系統實作（Part-3結業式🥳）
- 前言 ⭐⭐⭐
    - 算是把專案Part-3的部分完結啦～其實我本來考慮在這邊完結整個專案，但後來思考過後，我覺得可以再延伸一個Part-4，預計會針對單元測試、WebHook、Redis等方向來前進，但在這之前，先讓我來總結一下本次Part-3的學習心得！
    - 我打得不多，因為手好酸ww。
- Security、Schedule兩大元件的學習 ⭐⭐⭐⭐⭐⭐⭐
    - 藉著Part-3專案，我也跟著參加了IT鐵人賽的活動，雖然寫得很爛ww，但至少讓我重拾原本認真寫筆記的心態！
    - **Schedule的歷程** : 
        - 這條路很長，我從原生排程一路到Spring的Schedule，自己設計ThreadPoolConfig，到後來的Async設計，最後實作出自己的「**自動刪除未付款過期訂單**」，這也是Part-3第一個功能！
    - **Security的歷程** : 
        - 這條路更長ww，從Servlet到Controller其中經歷各種轉接的FilterChain，我算是一個個順過，而當然最終目的就捨棄自己的切面驗證，轉而使用更外層的Spring Security。
        - 最後也自己設計的Jwt的STATELESS驗證，以及配合Spring規定的UserDetails介面來實現統一格式。
        - 同時我也設計了Evaluator的分散式設計搭配Checker做驗證，擴充hasPermission的可用性！
    - 這邊我覺得最好玩的就是怎麼去解決兩個元件結合後的系統本人的驗證問題，是要給Role還是直接Bypass？
- 專案功能完善 ⭐⭐⭐⭐⭐⭐⭐⭐
    - 儘管這邊開始就沒有特定主題了，但我還是針對每個Entity著手完善詳細功能 : 
        - 購物車 : 新增/減少/清空
        - 訂單 : 取消
        - 商品 : 編輯內容細項
        - 通知 : 已讀
        - 分類 : 目錄
        - 使用者 : 個人資料寫入/更新、後台面板設計
        - 例外處理 : 完善例外、解耦
    - 其中我印象深刻的是在「Transational」跟「Lazy加載」的OSIV交互關係上，這邊對我來說超級燒腦ww。
    - 當然還有像是PATCH動作、mapStruct等也是很好的資源實作認識！
- 未來的想法 : ⭐
    - 我想，在進入Part-4最終收尾之前，我可能會先嘗試不同方向的學習路途，像是Oracle的題目練習(像OCA、OCP等)、Minecraft插件的設計(想做很久了owo)
    - 在最後100天嘗試換條路走走看！我應該會留30天來進行Part-4！

## Day266
#### 學習重點 : Part-4專案前的規劃、Oracle的考題練習
- 規劃 ⭐⭐⭐
    - 最近接了一些事務所以有點小忙.w.，所以在Part-4前我想分成兩個階段，一個是Oracle的題庫練習，另外一個是Minecraft插件的設計！
    - 而我應該會優先把Oracle題目練習好，這樣之後就可以去考證照了w，雖然證照聽說很花錢，但我還是打算試試看，畢竟學Java這麼久感覺得有個憑證才有成果。
    - 再來是Minecraft的插件，我打算花一個月來好好學好怎麼做插件，可能會做出一個小功能出來owo？
- Oracle題目分類 ⭐⭐⭐
    - 今天先不練習，因為我想去找書來看看(雖然很討厭看書w，我稍微整理了有關Oracle公司所提供的考試項目 : 
        - **Foundation** : 基本Java語法
        - Asscociate/SCJA/**OCA** : 物件導向語法、API使用（像String、ArrayList、Exception等）
        - Professional/SCJP/**OCP** : 至於OCP則會傾向Collection、Stream API、Concurrent還有Lambda、泛型等比較進階的設計。 
        - Expert/SCJE/**OCE** : 這個就超進階了，比較偏系統架構的設計，先不考慮w。
    - 我應該會從Foundation/OCA開始，基本上都使以Java SE為主，而題型會因版本不同而有差異，但我應該都會練練。

## Day267
#### 學習重點 : Java Foundation架構
- Java 簡介與環境基礎 ⭐⭐⭐
  - **核心觀念**：JVM / JRE / JDK 的關係與跨平台原理（Write Once, Run Anywhere）。
  - **程式結構**：基本 `main` 方法入口與編譯指令 (`javac` 轉為 Bytecode，`java` 執行)。
- 資料型別與運算子 ⭐⭐⭐⭐
  - **八大基本型別Primitive Types**：`byte`, `short`, `int`, `long`, `float`, `double`, `boolean`, `char`。
  - **運算與輸入**：算術/邏輯運算子優先權、`String` 基本操作，以及用 `Scanner` 讀取輸入。
- 流程控制 ⭐⭐⭐⭐
  - **條件判斷**：`if-else`、三元運算子、`switch-case`。
  - **迴圈結構**：`for`、`while`、`do-while` 走訪，以及 `break` / `continue` 的跳轉控制。
- 物件導向與基本 API (OOP & Basic APIs) ⭐⭐⭐⭐⭐
  - **OOP 核心**：類別 (Class)、物件 (Object)、建構子 (Constructor) 與封裝 (`private`)。
  - **資料集合與例外**：一維/多維陣列、`ArrayList` 基礎操作，以及 `try-catch` 例外處理。

## Day268
#### 學習重點 : Java Foundation測驗.1
- 測驗內容 ⭐⭐
    - 今天我去Oracle官網看Foundations第一課Explorer的課程，想說先試試看測驗，再看需不需要從頭開始學，結果好像蠻輕鬆就拿100分的ww，但我還是整理了一些比較好玩的問題 : 
    ![image](https://hackmd.io/_uploads/BkYRwGNqfe.png)
- switch的fall-through與case
    - 雖然都有概念了拉，但還是再複習一下！
    - 若多個case導向的statement相同，其實可以合併為 : 
    ```java=
    switch(variable){
        case 1: case 2: case 3:
            // ... statement
            break;
    }
    ```
    - 當然也可以用Java 14引進的 `case 1,2,3 ->` 的語句，但先用傳統的即可！
    - 而要注意的是switch的fall-through問題 : 
    ```java=
    switch(variable){
        case 1:
            // ...statement
        case 2:
            // ...statement
            break;
    }
    ```
    - 若case 1符合，但沒有break，則執行完statement，後續會繼續執行且「**無視條件**」，因此case 2也會被執行。
- 關於Helidon、OCI ⭐⭐
    - 我看Oracle一直推自己家的Java微服務框架Helidon，但生態系好像蠻小的ww
    - 關於跟OCI（Oracle Cloud Infrastructure）可能要再研究一下ouo。

## Day269
#### 學習重點 : Java Foundation測驗.2
- 測驗心得 ⭐⭐⭐⭐
    - 今天正式來測驗Java的Foundation題目，一樣是從Oracle官網上找的，也成功取得100分的成績，感覺我可以來挑戰看看OCA、OCP的題目ww，只是好像要買模擬題，所以我還在想。
    ![image](https://hackmd.io/_uploads/r1EcwLB5fe.png)
    - 今天一樣是遇到switch的時候有特別思考了一下，要注意的點在於 : 
    ```java=
    switch(variable){
        case 1 -> // ..statement
        case 2 -> // ..statement
    }
    ```
    - 如果沒加上default，則為一般陳述句，不回傳，通過編譯。
    - 然而若為Expression，將switch的結果回傳，則需要加上default確保switch有回傳值！

## Day270
#### 學習重點 : Java OCA測驗.1
- 測驗整理 ⭐⭐⭐⭐
    - 今天到enthuware的網站上找到了一些OCA模擬題來寫（測驗10題，有錯一題owo），應該之後會來買他們官網的模擬題集合來寫寫看！
    - 而我今天有遇到幾個不錯的題目 : 
- 關於interface的繼承與default問題 ⭐⭐
    - 底下是範例 : 
    ```java=
    public interface A{
        public default String getAddress(){
            return "...";
        }
    }

    public interface B extends A{
        public default String getAddress(){
            return "。。。";
        }
    }

    public class C implements A, B{}
    ```
    - 當我有兩個介面，且B繼承A時，C類別是可以同時實作AB父子介面的！
    - 「當我用C物件呼叫getAddress會怎樣？」
        - JVM執行程式時，會以子介面優先父介面的條件，因此會選擇使用B的default。
    - 「當C實作的兩個介面沒有繼承關係，但兩介面又有同名函式呢？」
        - 此時JVM會報錯，因為不知道要用or覆寫哪個介面！
- 關於Java針對成員變數的primitive type未初始化時的動作？ ⭐⭐⭐⭐⭐
    - boolean : 未初始化時，則 **分配false** 給它。
    - numeric type : **一律給予0**（包含char）
    - object : 給予null
    - 所以當我 `boolean[] ba = new boolean[10]` 時，ba中的每個元素由於未被初始化，因此一律設為false！
    - 注意⚠️，若是在「**函式**」中未初始區域變數，不會像成員變數那樣被Java自動設定預設值，而是直接編譯報錯！
- 關於main進入口的設定 ⭐⭐⭐
    - 若我寫 `public static Long main(String[] args){return 10L;}`
    - 上述是 **Compile正確，Runtime錯誤** : 
        - 在編譯時期，main只會被當作一般靜態函式做處理，因此不會出錯。
        - 但到了Runtime，JVM會因為找不到 `public static void main(String[] args)` 的函式進入口，而噴出 `NoSuchMethodError` 的錯誤。

## Day271
#### 學習重點 : Java OCP測驗.1
- 前言 ⭐⭐
    - 我已經買了Enthuware的1Z0-829（Java SE 17）模擬題owo，但我不知道貼上來當作筆記會不會有著作權的問題？ 所以我只會分享我的錯題修正重點，其他就不放上來了！
    - 雖然買了之後才發現好像應該要買Java 21 or 25的w，但買了就買了，先練完再說！
- 測驗修正重點 : ⭐⭐⭐⭐⭐⭐⭐⭐
    - Record的特性 : 
        - record類別會生成 `final類別、final成員、getter、覆寫Object`，注意getter生成時不會加上 `get`，而是直接以成員名稱命名。
        - **不允許自行宣告實例成員**，但可以設計「實例函式、靜態成員、靜態函式」
        - 允許實作介面，但不允許繼承(因為其本身就繼承Record類別)
        - 不能被宣告為抽象Record : `abstract record ...`
        - 可以設計建構子，但建構子參數列為空，只能用於校驗參數 : 
        ```java=
        public Student {
            if (id < 0) throw new IllegalArgumentException();
        }
        ```
    - String的分行 : 
        - 假設我有個String長這樣 : 
        ```java=
        String name = """
                      小八\
                      你好w"""
        ```
        - 其中的 `\` 代表的是 `不要換行`，因此實際print出來會長這樣 : `小八你好w`
    - Lambda的簡寫形式、型別問題 : 
        - 所謂的簡寫就是簡化了「**return、{}、分號**」的動作，如下方展示 : 
        ```java=
        // 兩者都可以
        Predicate<Data> p1 = (d) -> {return d.value>3;};
        Predicate<Data> p2 = (d) -> d.value>3;
        ```
        - 當然可以將 `(d)` 改成 `(Data d)` 這樣的宣告方式，但就是有點多餘拉w。
    - 繼承中，轉型後的成員指向 : 
        - 總結來說，轉型後 : 
            - 呼叫 **實例變數、靜態變數 & 方法** 以「當前型別」為主 -> Reference Type -> 編譯時即決定，不具備多型。
            - 呼叫 **實例方法** 則以「原本型別」為主 -> Actual Object Type -> 經轉型後，還是可以找到源頭，因此為多型特徵。
        - 假設 C繼承B，B繼承A，三者都有i成員跟getI函式，但i的值都不同。
        ```java=
        A o1 = new C();
        B o2 = (B) o1;

        System.out.print(o1.getI()); // o1原本是C，因此呼叫C的getI
        System.out.print(o2.i); // o2.i以當前型別B為主，因此找B的i
        ```

## Day272
#### 學習重點 : Java OCP測驗.2
- 測驗重點 🌟🌟🌟
    - Record的精簡建構子 ⭐⭐
        - 當我們在record中寫下 : 
        ```java=
        public record Test(int id, String name){
            public Test{
                this.id = id;
            }
        }
        ```
        - 是不合法的，因為record自動生成的final class檔中，就會使用對final int id賦值，若手動賦值就會違反final特性。
        - 但倒是可以這樣做 : 
        ```java=
        public record Test(int id, String name){
            public Test{
                id = id+1;
            }
        }
        ```
        - 因為這只是對「參數」做修改，而不是本身的成員變數。
    - Stream API的intermediate跟terminal ⭐⭐⭐⭐
        - 當我們利用Stream去做事情時，通常會有intermediate及terminal兩種形式。
        - 用直觀的說法 : 
            - `map、filter` 等會回傳Stream，等待繼續操作的屬於intermediate。
            - forEach、toList、findFirst等為terminal（可能是動作、回傳其他型別如Optional、List等）。
        - 而intermediate通常接收function類型的參數，如Function、Predicate等，這種屬於Lazy Evaluation，只會等到terminal敘述才會被觸發。

## Day273
#### 學習重點 : Java OCP測驗.3
- 測驗內容 🌟🌟🌟🌟🌟🌟
    - Java Module的由來 ⭐⭐⭐
        - 在Java 9之前，程式碼通常只會被簡單的分類（ClassPath單純根據路徑找檔案），因此當我宣告public class時，其他檔案都可以access，但有時候我們只希望該類別只被某些類別使用。
        - 在Java 9後，提供了一個module-info.java的模組配置檔，舉例來說 : 我在 `com.example` 路徑下加了module-info.java，等於告訴 `com.example` 底下的所有資料夾的所有類別都屬於「同個模組」。
        - 我可以在module-info.java中使用 :
            - `requires` : 告訴其他要使用 `com.example` 模組的人，使用這個模組依賴甚麼套件。
            - `exports` : 來允許哪些Package可被外界看到。
        - 如果我沒有exports某個類別，就算該類別是public，也只供模組內部使用，這就是所謂的「**強封裝Strong Encapsulation**」
    - Java ClassPath與Module的關係 ⭐⭐⭐⭐
        - 一般我們寫的Spring專案其實都還是維持ClassPath這種無模組化的專案，因為Spring使用大量的「反射機制」，這剛好是模組化專案預設不允許的。
        - 因此Java為了因應多數ClassPath架構，當執行ClassPath架構專案時，會自動將整個專案包裝成一個「Unnamed Module」，做個樣子ww。
    - Sealed Class ⭐⭐⭐⭐⭐⭐⭐
        - 而說到Sealed Class的特性，就剛好跟Module有關，在Sealed Class有個規定 : 
            - 當我有 `public sealed class A permits p2.B{}` 以及 `public final class B extends p1.A{}`，A在package1，B在package2，屬於不同Package，此時就會報錯。
            - 原因在於sealed class所允許的類別繼承動作，只能夠在 : 同模組 or 同package（Unnamed Module）
            - 因此我們可以將 **B移到p1**，或者，將 **A跟B作為同一module做打包**，因為Sealed允許同一module內不同package做繼承。
            - 而B也只能以 `sealed`、`non-sealed`、`final` 之間做修飾。

## Day274
#### 學習重點 : Java OCP測驗.4
- 測驗內容 🌟🌟🌟🌟🌟
    - 關於模組 ⭐⭐⭐⭐⭐
        - 當我原本有一個 `.jar` 檔，供其他公司的人使用，現在我希望將其模組化，但又希望最小化影響他人的程度，我該怎麼做？
        - 假設我有專案有 `org.example.bonds` 以及 `org.example.bonds.analytics` 的 packages。
        ```java=
        module bondanalytics{
            exports org.example.bonds;
            exports org.example.bonds.analytics; 
        }
        ```
        - 在Java的世界中，儘管analytics屬於bonds，但還是會被認為是不同的package，因此若我只exports bonds，而沒有bonds.analytics，則其他公司依賴analytics的部分就會出錯！
        #### 關於opens那回事
        - 由於exports不允許反射機制，因此我不能引入模組內類別後，在執行期存取private成員。
        - 這時候就出現了 opens 的語法，意即「開放該package給大家使用反射機制」。
        ```java=
        module myModule{
            // 開放給特定模組用
            opens org.example.bonds to spring.core;
        }
        ```
        - 當然也可以直接
        ```java=
        open module my.module {
        }
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

## Day277
#### 學習重點 : Java OCP測驗.7
- 測驗內容 🌟🌟🌟🌟🌟🌟
    - Interface的定義 ⭐⭐⭐⭐
        - 當我們把 `enum`、`class`、`interface` 塞在interface中，都會隱性加上 `public static`，而若是一般的 `method`，則是加上 `public`。
        - 若要實作method，則需要加上 `default`。而若在method前加上static、private，則必須實作。
        - Interace也能使用變數，會隱性加上 `private static final`，因此宣告時必須賦值。
    - Sealed class巢狀類別的繼承 ⭐⭐⭐⭐⭐⭐
        - 先看以下程式 : 
        ```java=
        public class Test{
            sealed class A{}
            
            non-sealed class B extends A{}
        }
        ```
        - 你會發現，A沒有加上permits關鍵字欸！這是因為當A類別發現沒有permits時，且又在Test內部時，會去看看有沒有同樣在Test類別中的 `final`、`non-sealed`、`sealed` 類別繼承A，若沒有則會報錯，若有則可以通過！
    - Java的指令應該如何下？ ⭐⭐⭐⭐⭐
        - 一般來說Java的指令格式如右 : `java [設定選項] [目標] [主程式入口] [參數]`。
        - 而就一般classpath來說，我們會用 `java -cp . Main` 做為一般的指令，意思是 「根據目前所在資料夾(.)，執行Main」，若Main有用到任何其他class，則從 `.` 開始找。
        - 而我也可以用 `;`(Windows)、`:`(macOS、Linux) 來加大目標範圍，像是 `java -cp .;org.example.jar Main` which means 「掃過目前資料夾以及org.example.jar的class檔」
    - 模組化的封包在一般classpath專案使用會有差嗎？ ⭐⭐⭐⭐⭐⭐⭐
        - Q : 像剛剛的org.example.jar，如果我丟到一般的classpath專案，會怎麼樣？
        - A : 其實不會，Java會自動無視模組化專案中的 `module-info.class`，將其視為一般的jar，因此我們也可以使用 `-cp` 直接執行一個模組化專案！

## Day278
#### 學習重點 : Java OCP測驗.8
- 測驗內容 🌟🌟🌟🌟
    - 關於ExecutorService ⭐⭐⭐⭐⭐⭐⭐
        - 當我們利用 `Executors.newSingleThreadExecutor` 來建立ExecutorService後，我們通常會發送任務，使用 `.submit`，而submit的任務又有幾種 `(Runnable task)`、`(Callable<T> task)`、`(Runnable task, T result)`。
        - 後兩者回傳值都是 `Future<T>`，前者是 `Future<?>`，而最後一個為何要回傳result呢？
            - 一般來說是因為 `做回傳標記`、`丟容器交給Runnable修改後再做為result傳出`。
            - 後者跟Callable還是有稍微不同。Callable通常是在 **內部實例化物件** 做運算後傳出; 而Runnable處理物件再藉著result傳出通常是去修改「**既有**」的物件！
            - 雖然還有一種可能是 **Runnable比較早誕生**，所以才有這種方法出現w。
        - 而我們在使用Callable時，有時候也沒有要用到回傳值，像是 `寄信+invokeAll(Collection<Callable<T>>)`，一次寄出大量信件又不希望獲得回傳值，則可以使用 `Callable<Void>` 來接收，當然，其中的實作最後就顯式 `return null`。
        - 而 `.submit()` 在提交任務時即在背景執行（Eager Execution，無延遲求值）。
        - `Future.get()` 呼叫時，會以 Blocking 卡住主執行緒，等待結果，若在子執行緒發生例外，會在呼叫 get() 時被包裝成 ExecutionException 拋出。

## Day279
#### 學習重點 : Java測驗OCP.9
- 測驗內容 🌟🌟🌟🌟🌟🌟🌟
    - 關於StringBuilder ⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐
        - 記得在Day199的時候，我就有寫到關於String家族的觀念，但沒有練習操作過StringBuilder的方法，所以今天在練習時就錯了ww。
        - 以下是StringBuilder的常用方法 : 
            - append、insert的綜合參數 : 通常會接收一個 `CharSequence s`，也就是要新增、插入...的字串，接著還有 `start、end` 是去 **切割**「CharSequence」，以上通常是StringBuilder的參數！
            - append : 
                - 基於當前字串，在後方加入字串（CahrSequence s），接著必須給予 `start、end`，一般來說都是 `inclusive to exclusive`。
                - 使用完後，回傳reference，但若 `start小於0、start大於end、end大於s.length` 則拋出 `IndexOutOfBoundsExecption`。
                - 範例 : `StringBuilder("0123456789").append("AAAXXX", 1, 4).toString();` --> `0123456789AAX`。
            - insert : 
                - 基於當前字串，從index為0做Offset，放入要插入之字串（會放在indexOffset **前** 的位置），接著一樣有start、end是切割要插入之字串。
                - 使用完後，回傳reference，但若 `start小於0、start大於end、end大於s.length` 則拋出 `IndexOutOfBoundsExecption`。
                - `Offset小於0或大於整個字串`，則拋出 `StringIndexOutOfBoundsException`。
                - 範例 : `StringBuilder("0123456789").insert(1, "AAAXXX", 0, 3).toString();` --> `0AAA123456789`。
            - replace : 
                - 這邊的start、end則是 **針對主字串本身**，選取一個範圍，替換成我們要的字串。
                - 範例 : `StringBuilder("0123456789").replace(0, 3, "AAAXX").toString();`，將0~2的位置替換成AAAXX。
            - substring : 
                - 回傳String，不會修改原本的StringBuilder，而接收start、end跟replace一樣，是選取範圍，但不做替換，而是 **轉成String後回傳**。
            - replace跟substring若start跟end有誤，則都拋出 `StringIndexOutOfBoundsException`。

## Day280
#### 學習重點 : Java OCP測驗.10
- 測驗內容 🌟🌟🌟🌟🌟🌟🌟
    - 關於Exception ⭐⭐⭐⭐⭐⭐
        - 首先要先理解 `System.out.print` 與 `printStackTrace` 的差別
            - 當我們print出一個例外時，Throwable所覆寫的toString方法是將 `className + message` 印出，因此大概會長這樣 : `exceptionT.MyException: Exception from foo`。
            - 而使用printStackTrace的話，從定義看 : `a complete chain of the names of the methods called, along with the line numbers, is printed`，表示整個源頭跟中間呼叫層都會跑出來。
        - 以上是Exception的印出方式，而接下來我遇到了一題是關於 `Exception` 繼承階層上的catch衝突
            - 當我寫下 : 
            ```java=
            try{
                // ...
            }catch (IOException | Exception e){
                // ...
            }
            ```
            - 則會報錯，原因在於IOEception與Exception存在「**繼承關係**」，此時我們應該選其一。
            - 且我們宣告的 `e` 具有final屬性，因此不能重新指派。

## Day281
#### 學習重點 : Java OCP測驗.11
- 測驗內容 🌟🌟🌟🌟
    - 關於編譯期與執行期 ⭐⭐⭐⭐
        - 先假設一個繼承結構如右 : subclass `D -> C -> B -> A` parent class 
        - 假設我們寫了以下程式 : 
        ```java=
        A a = new C();
        a.sleep();
        ```
        -  左邊的 `A a` 是在編譯期做**靜態宣告**，而右邊的 `new C()` 則會等到**執行期才做記憶體分配的實際型別**。
        -  在編譯期，java只會確認C是否繼承A，且A是否有sleep方法。
        -  若sleep是屬於C，則在編譯失敗（因為編譯期只看的到靜態的A），若sleep屬於A，則根據C是否覆寫，在執行期就以誰寫的來執行。
    -  關於instanceof
        - 在Java中，我想要知道 C跟A是否有「Is-a」的繼承關係，會使用 `instanceof` 來做確認，而instanceof又屬於 `執行期`，因此在instanceof會看物件實際型別為何去判斷。
        - 但在編譯期，若兩物件在繼承結構上沒有交集，則直接錯誤，不會進入執行期。

## Day282
#### 學習重點 : Java OCP測驗.12
- 測驗內容 🌟🌟🌟🌟🌟🌟
    - 關於Pass by value那回事 ⭐⭐⭐⭐⭐⭐⭐
        - Java官方所定義的函式參數傳入方式是Pass by value，而不是Pass by reference。
        - 這點用prmitive type來思考確實很合理。
        - 但當今天傳入的是「物件」，會發現我使用該物件的setter，是能影響到外部物件的。
        - 其實這點在我之前滑脆有思考過，我的想法是 : 
        > 物件所謂的pass by value，其中的value是原物件地址的「一份copy」，因此參數指向的是地址的value，但跟原物件變數是分開的。
        - 上述這段是我當時針對該議題所做的回應，意即我在函式內部，參數存的也是地址，但與外部物件不是同個東西，which means，兩變數所在的記憶體位置是不相同的，只有值是相同，因此當我看到以下程式碼 : 
        ```java=
        public class TestClass{   
            static String str = "Hello World";   
            public static void changeIt(String s){    
                s = "Good bye world";   
            }   
            public static void main(String[] args){     
                changeIt(str);     
                System.out.println(str);   
            } 
        }
        ```
        - s與str指向的值相同，但本身變數存的位置不同。
        - 也因此我改動s不會影響到str，因此輸出不會改變。
        - 當然，如果我使用StringBuilder這種以method去改動物件的成員，那當然可以使輸出有變化，但這就與setter是一個道理了（改動物件內部，而不是值本身）。

## Day283
#### 學習重點 : Java OCP測驗.13
- 測驗內容 🌟🌟🌟🌟
    - 關於Comparator的設計 ⭐⭐⭐⭐⭐
        - 雖然我在Day192有學習過Comparator的設計與概念，但可以再做一次延伸。
        - 要做出比較器會使用 `Comparator.comparing` 的overload
            - `comparing(Function keyExtractor)`
                - 傳入一個 `Function`（如方法參考 `Person::getName`）來抽取要比對的屬性。
                - 但該成員必須要實作Comparable。
            - `comparing(Function keyExtractor, keyComparator)`
                - 傳入 `Function` 抽取成員後，使用自訂比較器 `keyComparator` 進行比對。
                - 如果成員本身沒有實作Comparable，則使用此函式
            - 基本型態有其特殊的比較函式（為避免boxing降低效能）
                - `comparingInt(ToIntFunction<? super T> keyExtractor)`
                - `comparingLong(ToLongFunction<? super T> keyExtractor)`
                - `comparingDouble(ToDoubleFunction<? super T> keyExtractor)`

## Day284
#### 學習重點 : Java OCP測驗.14
- 測驗內容 🌟🌟🌟🌟🌟🌟🌟🌟
    - 淺談CopyOnWriteArrayList的設計邏輯 ⭐⭐⭐⭐⭐
        - 首先它是 `Thread-safe`，但究竟是哪裡Thread-safe呢？
        - 原因在這 : 當我們要針對該類別之物件做 `.add`、`.remove` 時，其內部維護的 `volatile Object[] array` 會複製一份去做更改，操作完後，再將原本變數指向的舊array改指向新array。
        - 且其本身有加上互斥鎖，因此同時間只能有一執行緒取得鎖。
        - 因此不會引發所謂的 `ConcurrentModificationException`。
        - 當然，由於每次更動都要做一次更改，會非常耗效能，因此該類別只適用於「讀多寫少」的情況。
    - CopyOnWriteArrayList的實際測試 ⭐⭐⭐⭐
        - 當我寫以下程式 : 
        ```java=
        public static void main(String[] args) {
            
            List<String> cwa = new CopyOnWriteArrayList<>();
            cwa.add("hello");
            cwa.add("bob");
            
            for (var a : cwa){
                if (a.equals("bob")) cwa.remove(a);
                if (a.equals("hello")) cwa.remove(a);
            }

            System.out.print(cwa);
        }
        ```
        - 若我使用一般的 `ArrayList` 實作，則其中的forEach就會報錯。
        - 然而使用 `CopyOnWriteArrayList` 則會再做remove時，先複製再改指向。
    - CopyOnWriteArrayList原理探究 ⭐⭐⭐⭐⭐⭐⭐⭐
        - 上一點所寫的forEach本質上底層由Iterator去做事，而一般的ArrayList會確保在迴圈時的array完整性，使得無法修改。
        - 然而在 `CopyOnWriteArrayList` 中，當我們開始迴圈，取得Iterator時，Iterator指向的List就是原本的List，跟我們後續作修改的完全無關，它會如實的走完全程。
        - 而我們在其中做的修改，會經歷幾個步驟 : 
            1. 複製陣列
            2. 修改
            3. 立刻改array指向（但Iterator還是指向最初的array）
        - 接著後續的更改就會拿著新的array去做修改。

## Day285
#### 學習重點 : Java OCP測驗.15
- 測驗內容 🌟🌟🌟🌟
    - 關於運算子優先順序與運算元求值順序的獨立性 ⭐⭐⭐⭐⭐⭐
        - 我們都知道運算子有不同的優先級，像是 `*、/` 大於 `+、-`，因此先乘除後加減 : `a + b * c`。
        - 但若a、b、c是取值的函式 : `a() + b() * c()`，這時候Java會嚴格依照 「由左至右取值」，**先取** 得a()，再取b()，最後是c()，**再來才是做先乘除後加減**的動作。
        - 而上述的 **前者取值** 與 **後者的乘除加減** 是兩套系統，互不干涉。
        ```java=
        int s = 5;         
        s += s + mx(s) + ++s; 
        // 可以寫成 s = s + s + mx(s) + ++s
        // 假設mx函式可以使(參數+3)
        ```
        - 上述是一個有點刁鑽的例子，尤其是後面那個++s我一直很疑惑ww，對我來說++s會先做運算更新s，然後其他s就會用新的s去運算，但顯然不是。
        - 真正是我們會先取得s=5，接開始由左至右取值，`s = 5 + 5 + 8 + 6`。
        - 接著等++s執行完，s才會更新成6，但前面早已取值完，因此不影響，進入運算階段。
        - 若要拿著6去做取值的話，可以改寫成 : `s = ++s + s + s + mx(s);`，這樣後面三個運算元都會拿到更新後的s。
        - 總結 : **先做運算元取值，再做實際運算**。

## Day286
#### 學習重點 : Java OCP測驗.16
- 測驗內容 🌟🌟🌟🌟
    - 關於 `?` 與 `Object` 的差別 ⭐⭐⭐⭐⭐⭐
        - 如果我宣告 `List<?>`，跟宣告 `List<Object>`，兩者意義上是表示任何型別，但是在檢查上卻相差巨大。
        - 所謂的 `List<Object>` 在宣告時，表示允許所有類別物件的傳入。
        - 然而 `List<?>` 一般不會用於宣告，原因在於當我宣告 `List<?>`，我對該容器不能做 `.add` 等動作，因為編譯器不知道型別為何，因此一般來說 `List<?>` 會用於參數、接收回傳等「讀」的情景。
        - 而讀出來的型別會被認為是Object，因為我們知道任何型別都繼承自Object，這也是為何有人會把 `List<?>` 跟 `List<Object>` 搞混！
        - 而當我使用 `? extends String`，這樣的方式，最後讀出來就會被認為是String。
        - 因此我可以這樣寫 : 
        ```java=
        List<? extends String> strList = ...;
        for (Sring str : strList){
            ...
        }
        ```
        - 而如果我寫 `? super String`，則是String以上的任何父類都可以。

## Day287
#### 學習重點 : Java OCP測驗.17
- 測驗內容 🌟🌟🌟🌟
    - 關於module系統（JPMS）的引入規則 ⭐⭐⭐⭐⭐
        - 雖然現在Swing、AWT好像已經過時了w，但還是會一些有關module的東西需要理解。
        - module預設引入的唯一例外：`java.base`
            - 在 Java 模組化架構下，只有 `java.base` 是唯一會被 **自動引入** 的模組（eg. `java.lang`、`java.util`...）。
            - 只要類別不屬於 `java.base`，即使它是 Java 官方標準內建的工具，都**必須**在 `module-info.java` 中寫 `requires`。
        - Swing / AWT？
            - 我們知道這兩個算是很早期的Java GUI工具。
            - 而其常見的 Package 像是 `javax.swing` 或 `java.awt`。
        - But... Package 名稱 ≠ Module 名稱
            - 而 `javax.swing` 這種，是所謂的Package，不是Module，因此不能直接做requires（只接收module）。
            - 而Java將GUI相關的套件，都打包在了 `java.desktop` 中，因此只要requires這個就行。
            ```java=
            module my.app {
                requires java.desktop;
            }
            ```
        - 不過要注意 `java.base` 是做「requires」，只是告訴外部該模組依賴哪些模組，因此內部還是得做import。

## Day288
#### 學習重點 : Java OCP測驗.18
- 測驗內容 🌟🌟🌟🌟
    - 關於instanceof的pattern變數作用域 ⭐⭐⭐⭐⭐
        - 我們知道有個語法是 : `if (obj instanceof String s)`，可以直接轉型並宣告變數。
        - 但是！要注意變數名稱重複宣告的編譯問題：
            ```java=
            static void processBase(Base base) {
                A a = null; 
                if (base instanceof A a) {
                    System.out.println(a.power());
                }
                System.out.println("a is " + a);
            }
            ```
            - 原因：`instanceof A a` 中的 `a` 本質上就是一個區域變數（Pattern Variable）。
            - Java 禁止區域變數遮蔽（Shadowing）同一個作用域中已存在的變數名稱。
            - 上面第一行已經宣告了 `A a`，後面的 `instanceof A a` 又想宣告一次 `a`，會引發重複宣告的機制，**無法通過編譯**。
        - Pattern Variable 的特性：
            - pattern變數 `a` 的生命週期只存在於「條件判定為 true」的區塊內部。
            - 若外層沒有宣告 `A a = null;`，但在 `if` 區塊外部呼叫 `System.out.println(a)`，同樣會引發編譯錯誤，因為離開 `if` 範圍後 `a` 就失效了。

## Day289
#### 學習重點 : Java OCP測驗.19
- 測驗內容 🌟🌟🌟🌟
    - 關於Deque的詳細使用方式 ⭐⭐⭐⭐⭐
        - 其實我對Deque已有初步的了解，它具有FIFO/LIFO的特性，且於Java中使用會優於Stack。
        - 但今天要探討的是其中的method差異。
        - 關於Queue的特性 : 
            - `offer`、`add` 都是將element加到rear，而 `poll`、`remove` 則是移除front的element並回傳其值。
            - 而 `add` 在容器空間不足時，會拋出exception，但 `offer` 不會，它會回傳false。
            - 另外 `remove` 在容器為空時會拋出exception，但 `poll` 不會，它會回傳null。
        - 關於Stack的特性 :
            - 具有LIFO特性，操作皆在front/head。
            - `push` 是加到front，`pop` 是從front移除並回傳。
        - 當要查閱前端元素時，`element` 在容器為空時會拋出exception，而 `peek` 則是回傳null。
        - 所謂的First是指front，Last是指rear，由於LIFO特性push跟pop只針對front，因此沒有所謂的pushLast、pushFirst..等等。但由於Deque可以說是雙向佇列，因此有offerFirst、addLast...等等。

## Day290
#### 學習重點 : Java OCP測驗.20
- 測驗內容 🌟🌟🌟🌟
    - 關於virtual call的概念 ⭐⭐⭐⭐⭐⭐⭐
        - 所謂的virtual call意思是指在「Runtime時期根據實際型別做呼叫」以此保證「多型」的樣態。
        - 基本上除了 `private`、`final`、`static`、`constructor` 之外的method，都是以virtual call作為呼叫辦法，相對的，若是上述三種修飾類型則會以 `classical call` 作為呼叫方式。
        - 若我們使用virtual method時，JIT發現該method並沒有被子類覆寫，則會由virtual轉為classical以加速執行速度。
        #### 為何 `private`、`final`、`static` 會是classical？
        - 原因是 : 三者都是在編譯型態就靜態綁定了。

## Day291
#### 學習重點 : Java OCP測驗.21
- 測驗內容 🌟🌟🌟🌟🌟
    - 關於Static Initializer Block ⭐⭐⭐⭐⭐⭐⭐
        - 當我們在一個class中寫下 `static{}`，就是在做靜態初始化。
        - 但究竟靜態初始化是如何運作的呢？
            - 當我們「第一次」呼叫該類別，無論是 `new`、`呼叫靜態成員`，就會以觸發靜態初始化，但後續再呼叫就不會。
            - 而靜態初始化作用是在Runtime時期，當其被載入JVM中時會觸發。
        - 但有些前提不會觸發靜態初始化 : ⭐⭐⭐⭐⭐⭐⭐
            ```java=
            class Parent{
                static String name = "Parent";
                
            }
            class Son extends Parent{
                static final String ID = "final Son";
                static {System.out.println("Son");}
            }
            ```
            - 如果藉著Son去呼叫Parent的成員，那JVM就不會初始化Son，也就不會對Son做靜態初始化。
            - 如果第一次呼叫Son是呼叫 `Son.ID`，則由於 `static final` 的特性，也不會加載Son的靜態初始化，原因是final使變數被在編譯期就固定了！因此不會做初始化。

## Day292
#### 學習重點 : Java OCP測驗.22
- 測驗內容 🌟🌟🌟🌟🌟
    - 關於Module系統（JPMS）的綜合整理 ⭐⭐⭐⭐⭐⭐
        - 之前看過 `exports` 跟 `requires` 的基本用法，但我想針對Module再做一次延伸。
        - 每個Module都必須有自己獨立的資料夾、jar。
            - 一個資料夾底下絕對不能同時塞兩個以上的Module（因為根目錄只能有一個 `module-info`）。
            - 不過！我們可以把不同的Module各自打包成獨立的 `.jar`，再把這些JAR檔全部丟進同一個資料夾，最後把這個資料夾設定到 `module path` 上。
        - Module可以指定Package，也可以指定Service
            - **針對Package**：用 `exports` 匯出、用 `requires` 引入。
            - **針對Service**：用 `provides` 提供實作、用 `uses` 宣告使用。
        - Module是透過Dependency Injection（DI）來連接Service的嗎？
            - Java原生的Module系統 **並沒有使用DI**！
            - 雖然Module確實允許Service consumer去找到Service的實作，但consumer必須自己用迴圈（透過 `ServiceLoader`）遍歷所有實作，再挑出需要的那一個，並不是自動注入的（不像Spring有DI的機制，可以使用Autowired）。
        - Module可以把 `sealed class`（密封類別）匯出給其他Module繼承嗎？
            - `sealed class` 的所有子類別都必須在「同一個Module」裡面！（如果使用unamed module，則必須在「同一個package」裡面）。
            - 因此，就算把 `sealed class` 給 `exports` 出去，外部的class也絕對不能 `extends` 它！

## Day293
#### 學習重點 : Java OCP測驗.23
- 測驗內容 🌟🌟🌟
    - 關於Stream的特性與Comparator ⭐⭐⭐⭐
        - 先看底下的程式碼：
            ```java=
            Comparator<Book> c1 = (b1, b2) -> b1.getGenre().compareTo(b2.getGenre()); // 1
            books.stream().sorted(c1.thenComparing(Book::getTitle)); // 2
            System.out.println(books);
            ```
        - 題目問最後會印出什麼？答案是：未排序過的books！
        - 為什麼沒有排序且改動原本books？
            - Stream **不會修改原始資料**：對Stream呼叫 `sorted()` 只是回傳一個新的Stream參考（which means Intermediate operation，也是我之前學習過的！），原本的 `books` 則不會改動。
            - 缺少Terminal operation：若要「排序」books，則需要使用Terminal operation才會，但Terminal operation也只是產生一個新結果（List、void...），並不會改動原本的books，因此若要使用排序過後的books，只能賦值改指向。

## Day294
#### 學習重點 : Java OCP測驗.24
- 測驗內容 🌟🌟🌟🌟🌟🌟
    - 關於abstract的一些特性 ⭐⭐⭐⭐⭐
        - 「abstract class內部可以有實作method」，意即我們可以在抽象類別中寫下具有 `{}` 區塊的method。
        ```java=
        abstract class Test{
            void Impl(){
                System.out.println("Hello World!");    
            }
        }
        ```
        - 當然，如果有實作，就不能加上abstract修飾該method。
        - 「一般class內部不能有abstract method」，意即不能在一般class中加入抽象方法。
        ```java=
        public class Test{
            abstract void test(); // 報錯！
        }
        ```
        - 而當然，有些修飾詞也會跟abstract method互斥 : 
            - `final` : final會禁止method被覆寫，不適用抽象類別。
            - `private` : 抽象類別要給子類實作，因此不適用。
            - `static` : 屬於編譯期綁定類別，不適用需要實作的抽象方法、多型應用。

## Day295
#### 學習重點 : Java OCP測驗.25
- 測驗內容 🌟🌟🌟🌟🌟
    - 關於Stream的reduce ⭐⭐⭐⭐⭐
        - 這個我之前有學過了，所以就把題目的問題整理一下就好w。
        - 當呼叫不帶初始值的 `reduce((a, b) -> a + b)` 時，回傳型別是 `Optional`。
        - 若filter條件篩除所有元素（如 `a -> a < 0`），reduce會回傳 `Optional.empty()`。
        - 而Optional.ifPresent所確認的是 : 
            - 有值執行傳入的Consumer動作。
            - 若為空，則略過不執行，且不會拋出Exception。
    - 關於JDBC PreparedStatement與executeUpdate機制 ⭐⭐⭐⭐⭐
        - 這題其實我看了很問號ww，但還是紀錄一下好了。
        - 參數綁定規則：
            - PreparedStatement 的參數索引是利用 `?` 作為placeholder，接著利用物件的setter，依照Placeholder的順序設定（從索引1開始）。
            - 設定NULL時，則需使用 `setNull(參數位置, java.sql.Types.型態)`。
        - executeUpdate() 的回傳：
            - 用於執行 `INSERT` / `UPDATE` / `DELETE` 的情況。
            - 其回傳值代表 「**受影響的資料列數**」，並非受影響的欄位數。
            - 成功新增一筆資料，因此 `ps.executeUpdate()` 回傳並印出 `1`。

## Day296
#### 學習重點 : Java OCP測驗.26
- 測驗內容 🌟🌟🌟🌟🌟🌟
    - 關於Java的Unreachable statement ⭐⭐⭐⭐⭐⭐⭐
        - 在Java中，有些statement會被判定成unreachable，導致Compilation Error。
        - 但有些條件是需要仔細看 : 
        ```java=        
        while (false){
            // statement
        }

        // --------------

        for (int i=0; false; i++){
            // statement
        }

        // --------------

        do{
            // statement
        }while (false);

        // --------------

        if (false){
            // statement
        }

        // --------------
        ```
        - 前兩者會發生Error，但後兩者不會。
        - 原因在於 : 
            - `do-while` 本身保證一定可以被reach一次，因此不會報錯。
            - `if` 則是因為JLS（Java語言規範）有個「條件編譯」的規定，當我們寫下 `if(false){}`，javac在編譯時，就不會將該段編譯進 `.class` 中，而這個規定可以供開發者在Debug時，決定是否要「開啟Debug mode」。
        - Debug mode的概念如下 : 
        ```java=
        public class Test{
            public static final boolean DEBUG = false;
            
            public static void main(String[] args){
                if (DEBUG){
                    System.out.print("使用者ID : " + user.getId());
                }
            }
        }
        ```
        - 有時候我們要進行除錯模式，但又不想要上線時要一行一行刪掉除錯程式碼，我們就會利用一個Debug flag來選擇是否進入Debug mode。
        - 當javac看到if(false)時，就不會報錯，而是直接略過。
        - 補充一下 : `while (true){}` 這類型的迴圈（內部又沒有跳脫機制），若在其後續加上程式碼，也會報錯(Unreachable statement)。

## Day297
#### 學習重點 : Java OCP測驗.27
- 測驗內容 🌟🌟🌟🌟🌟🌟🌟🌟
    - 關於Serializable ⭐⭐⭐⭐⭐
        - 當我們想要序列化一個物件時，會在類別的宣告後方加上 `implements Serializable`，然而當類別內部有「**非基本型別**」的成員（which means其他類別），我們要確保該成員也有實作Serializable，才不會發生 `NotSerializableException`。
        - 至於陣列，若是「基本型別陣列、ArrayList」也支援序列化，但若陣列元素是其他類別且未實作序列化，則也會報錯。
    - transient與Serializable的搭配 ⭐⭐⭐⭐⭐⭐⭐
        - 若我們想要某些成員 **不被序列化**，可以加上 `transient` 使其被忽略（static成員也會被忽略，所以不用再加上transient）。
        - 而如果遇到使用「第三方類別成員，且未實作序列化」的情況，但我們又希望保留該成員的狀態，該怎麼辦？
            - 此時可以先將其加上 `transient`，接著再寫兩個method : `private void writeObject(ObjectOutputStream oos)`、`private void readObject(ObjectInputStream ois)`，內部將該成員的個別資料也寫進去序列化資料中。
            - 這兩個method就是 **序列化會使用到的method**，原本是預設，而我們自行實作了，因此JVM在序列化時，就會走我們自訂的這條路（注意，**這不是覆寫**，是讓JVM在執行時，以反射找到Method Signature去執行的）。

## Day298
#### 學習重點 : Java OCP測驗.28
- 測驗內容 🌟🌟🌟
    - 探討修飾子的編譯問題
        - 首先要知道的是，一個 `.java` 檔，只允許有一個 `public` 的類別，其餘只能是 `default`，也不能是 `private`、`protected`。
        - 而我們也可以在 `method` 內部宣告 `class`、`enum`、`interface`、`record`，但不能加上access modifiers，因為這屬於「Local Type」，意即該類型只存活於method內部，因此無須加上access modifiers。
    - 關於enum的東西，我有在Day172寫過哩！就先跳過ww。

## Day299
#### 學習重點 : Java OCP測驗.29
- 測驗內容 🌟🌟🌟🌟🌟🌟🌟
    - Unchecked Exception 與 Checked Exception在方法與繼承關係上的問題 ⭐⭐⭐⭐⭐⭐⭐⭐
        - 我們知道在父子類的繼承結構上如果出現以下片段 : 
        ```java=
        class Great {     
            public void doStuff() throws FileNotFoundException{     
            }     
        }  

        class Amazing extends Great {   
            public void doStuff() throws IOException, IllegalArgumentException{   
            }     
        } 
        ```
        - 會出現以下問題 : 
            - 子類拋出了 `IOException` 範圍大於父類拋出的 `FileNotFoundException`，這樣在「編譯期」就會報錯，因為我們必須保證子類的覆寫函式所拋出的Checked Exception範圍必須小於等於父類的同函式（且類型必須隸屬於父類所丟出的）。
        - 而解決方式可以是 : 
            - 使父類Checked Exception部分大於等於子類，或者是子類的小於等於父類。
            - 另外，使子類不拋出Checked Exception也可以，其等於保證該函式不會拋出來，一定是沒有，或者內部處理完。
        - 可以發現 `IllegalArgumentException` 不會出問題，那是因為其繼承 `RuntimeException`，也就是 `Unchecked Exception`，而這個在編譯期可以安穩通過，「編譯期只會受理 `Checked Exception` 的範圍問題！」。
        - 補張圖幫助理解 : 
        ![image](https://hackmd.io/_uploads/BJYWb3n3Gl.png)
        [圖片來源 : Checked and Unchecked Exceptions in Java](https://medium.com/@ahmed.abdelfaheem/checked-and-unchecked-exceptions-in-java-6cb1c9815d32)
