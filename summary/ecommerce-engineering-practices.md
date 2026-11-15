# 電商專案實戰（Part 2 ~ Part 3）

## Day111
#### 學習重點 : 小專案銜接
- Service層Optional轉換 ⭐⭐
    - 我今天將Service層的所有業務邏輯都轉成了Optional寫法，但我也發現了Optional的一些利弊 --> 由於是鍊式寫法，因此在throw不同例外時，有些不能夠寫在一起，這時候可能就得與其他語法綜合使用~
    - 今天把註冊、登入、密碼更新、重設密碼、信箱發送等都使用聲明式都轉好了，明天預計來發想新功能！
- 預計學習重點 : ⭐
    - 1️⃣ ORM架構
    - 2️⃣ Kafka套件
    - 3️⃣ 完善Response回應

## Day112
#### 學習重點 : 功能製作與model思考
- 購物功能 ⭐⭐
    - 我目前想到可以利用購物這個功能來熟悉架構，因此延伸出來看的話可以想到幾個必要model --> 商品、購物車、訂單、賣家、買家等。
    - 我覺得賣家買家可以獨立出來，同樣是User，但面向不同。
- model建立 ⭐⭐
    ![image](https://hackmd.io/_uploads/BkJ72DIabe.png)
    目前的流程是 : 使用者登入 --> 找到商品（確保商品存在）--> 檢查庫存 --> 更新購物車。
    - 先做出這個流程，之後再做其他的！

## Day113
#### 學習重點 : 認識ORM架構
- ORM是甚麼？ ⭐⭐⭐⭐⭐
    - 我覺得在正式實作多model之前，先來理解一下ORM架構可能對之後數據操作會更流暢一些！
    - ORM是（Object-Relational Mapping）的縮寫，亦即**物件關聯資料庫** --> 也就是我做的model會連結到DB當中的欄位。
    - 🌟JDBC的流程 : 寫好SQL語句 -> 使用Mapper對應值 --> 根據Mapper以及SQL語法利用update更新資料庫。
    - 🌟ORM的流程 : 對model class下註解 --> 使用方法加上參數對應資料庫 --> ORM框架 **自動生成SQL語法做映射**。 
    - 從上方的流程差異，可以發現透過ORM框架，封裝物件來操作資料庫比起純SQL操作更能簡化程式，且不需關心SQL語法怎麼寫，而是專心在怎麼操作物件方法。
- JPA（Java Persistence API） ⭐⭐⭐⭐
    - 它是Java官方用來針對ORM框架所提出的規範（因為它就是個API嘛ww），也因此JPA是一個介面。
    - Hibernate就是一個實作JPA的工具。
    - 在Spring當中，會使用Spring Data JPA，其底層透過Hibernate來實作，但因為封裝了JPA並擴展與簡化部分功能，因此能更快上手且省略部分流程。

## Day114
#### 學習重點 : Spring Data JPA - 概念與引入
- Spring Data JPA的架構 ⭐⭐⭐
    - `@Entity` --> 當我們使用Enitity註解model類別時，意旨該類別為數據類型，其屬性都會連結到資料表欄位中。
    - `Repository` --> 用來操作CRUD，通常會寫一個介面來繼承Repository，並指定要操作的實體model類別。
    - **約定大於配置** : `Query Methods` --> 一種約定自動生成SQL語法的方法命名規則，我們只要在Repository中寫出相對應的method以及參數就可以了！
- Maven & Application配置 ⭐⭐
    - 引入Spring Data JPA並設置application來實現自動建表的功能。
    ```xml=
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    ```
    ```properties=
    spring.jpa.hibernate.ddl-auto=update
    spring.jpa.show-sql=true    
    spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect
    spring.jpa.properties.hibernate.format_sql=true
    ```
    - show-sql能夠將產生的SQL語法顯示在控制台，最後一段format可以將SQL語法整理成縮排格式，明確看出語法。
    - 在正式測試時，通常會將show-sql與format關掉，且ddl-auto那段的update會改成validate來保證資料庫的完整與正確性，但這個目前我應該不太會用到w，先不深入理解！

## Day115
#### 學習重點 : Spring Data JPA - @Entity與Repository
- 建立新專案 ⭐
    - 我今天開了一個新的project針對JPA來練習！
    - 以下是我的專案結構 : 
    ![image](https://hackmd.io/_uploads/HkAqEB5Tbl.png)
- 資料實體@Entity ⭐⭐⭐⭐
    - 我在User當中以Lombook搭配JPA架構來撰寫 : 
    ![image](https://hackmd.io/_uploads/SydRNS5aWl.png)
    - `@TABLE` --> 將User實體連結到users資料表
    - `@Id` --> 指定account作為「**Primary Key**」
    - `@Column` --> 也就是欄位啦~
- Repository繼承 ⭐⭐⭐⭐⭐⭐
    - 在Dao層中，我建立了UserDao介面，而其需繼承JpaRepository<>作為JPA可操作的介面，而<>中需放入 --> User作為資料來源，String作為Primary Key。
    - 實際長這樣 : 
    ![image](https://hackmd.io/_uploads/H1pILr9aWx.png)
    - 「find-User-By-Account」就是Spring中所謂的約定大於配置。
    - 因account具唯一性，我 **選擇回傳Optional** 而不是List（因為最多只有一筆資料！）
- 實際成果 ⭐⭐⭐
    - 因為剩下Controller跟Service的東西都是Optional聲明式語句，沒什麼好看的w，直接看成果！
    ![image](https://hackmd.io/_uploads/HJ2lOHc6We.png)
    ![image](https://hackmd.io/_uploads/HkRNOrq6be.png)
    - 若帳號不存在，則 `return "查無此人"`。
    ![image](https://hackmd.io/_uploads/ryqPuHqTZe.png)

## Day116
#### 學習重點 : Spring Data JPA - CRUD簡易實作
- CRUD功能 ⭐⭐⭐⭐
    - 針對小專案的功能，我從註冊、更新資料下手，當然都是很簡易的流程w
    ```java=
    public String registration(UserRequest userRequest){
        try {
            User user = new User();
            // user設定...
            // 使用userDao繼承的Repository的內建功能.save
            userDao.save(user);
            return "新增成功";
        }catch (Exception e){
            System.out.println(e.getMessage());
            return "新增失敗";
        }
    }

    public String update(UserRequest userRequest){
        return userDao
                .findUserByAccount(userRequest.getAccount())
                .map(user -> {
                    // 更新名稱與密碼...
                    userDao.save(user);
                    return "更新成功";
                })
                .orElse("更新失敗");
    }
    ```
    - 這邊利用了 `.save` 來做 **建立與更新** 的動作。
- CustomQuery ⭐⭐⭐⭐
    - 除了用JpaRepository內建生成的SQL語法之外，我們也可以自己寫，就像JDBC那樣，但寫法截然不同。
    - JDBC的寫法如下 : 
    ![image](https://hackmd.io/_uploads/HyVqW_j6Wg.png)
    - 而Jpa CustomQuery如下 : 
    ![image](https://hackmd.io/_uploads/B1qR-Oia-g.png)
        - 在@Query當中所寫的user並不是資料表，而是Entity，直到user.account時，JPA才會去user所連結的TABLE尋找。
        - 在@Param當中，會指定 `:account` 所代表的變數是誰。

## Day117
#### 學習重點 : Spring Data JPA - Merchandise CR（~~UD~~）
- 商品資料表建立 ⭐⭐⭐
    - 我覺得我該來好好練習SQL語法了，很多都不太知道要怎麼寫，只能求助AI，QAQ。
    - 目前的建立方式如下 : 
    ![螢幕擷取畫面 2026-04-27 212846](https://hackmd.io/_uploads/SyEaqdeWfe.png)
- 商品三層架構 ⭐⭐⭐⭐
    #### GenerativeValue
    - 一樣透過Controller-Service-Dao來做分層，其中Dao連結Merchandise的Model，而商品的Model有個特別之處在於會需要利用GenerativeValue給定的方式來生成ID : 
    ![螢幕擷取畫面 2026-04-27 214002](https://hackmd.io/_uploads/HkxRqdgbMx.png)
    - 這邊利用 `IDENTITY` 意旨交給MySQL來自動遞增ID（`AUTO_INCREMENT`），當我們使用 `.save` 存入商品物件（不含ID）時，函式會回傳商品物件（含ID），此時可以接收該物件進行後續動作。
    #### 上架與查詢
    - 這邊參考 PCHome 24H 的網址來模仿，目前將CR做完，明天來試試看將User連結到商品！
    ![螢幕擷取畫面 2026-04-27 214312](https://hackmd.io/_uploads/rJ2Cq_ebGx.png)

## Day118
#### 學習重點 : Spring Data JPA - @ManyToOne、Cart建立
- @ManyToOne ⭐⭐⭐⭐⭐⭐
    - 甚麼是多對一？
        - 當「多位買家」都加入「A商品」進購物車時，就是多對一！
        - 當「多個商品」在購物車中都指向「同一買家」，就是多對一！
    - 透過以上的概念，可以知道買家與商品呈現 **多對多** 的情況，因此需要建立中間表（Cart）來連結兩個實體。
- Cart購物車建立 ⭐⭐⭐⭐⭐
    - 指定Merchandise與User為ManyToOne，並加入id與quantity變數。
    ![螢幕擷取畫面 2026-04-28 113642](https://hackmd.io/_uploads/BJj9cOxbfe.png)
    - 這邊先暫時用 `@JsonIgnore` --> 當取得購物車時，會過濾敏感的User隱私資訊。
- User_Id重建 ⭐⭐⭐
    - 原本User是利用 `AUTO_INCREMENT` 遞增id，但若應用在多資料庫時，可能因為 **id重複導致出錯**，且id若為自增，就 **有資安疑慮**，因此使用UUID隨機碼更為適合！
    ![螢幕擷取畫面 2026-04-28 114204](https://hackmd.io/_uploads/B1dicOe-Ml.png)
- 實際測試 ⭐⭐
    - 我利用自建的商品list功能+User註冊功能先取得UUID以及商品ID，由於我還沒寫加入購物車的功能，先暫時手動加入w
    - 實際取得User購物車內容長這樣 : 
    ![螢幕擷取畫面 2026-04-28 114512](https://hackmd.io/_uploads/HykncOgbGx.png)

## Day119
#### 學習重點 : Spring Data JPA - 購物車功能實作
- 購物車新增商品功能 ⭐⭐⭐⭐
    - 這個功能有幾個步驟需要釐清 ：
    - 取得商品ID --> 根據使用者ID尋找其購物車，並檢查是否有該商品的購物車資料 --> 檢查庫存是否大於使用者所選數量（含購物車原本可能有的） --> 若大於則.save，若小於則拋出例外。
    ![螢幕擷取畫面 2026-04-29 231451](https://hackmd.io/_uploads/H1WuqdgWGx.png)
    ![螢幕擷取畫面 2026-04-29 231504](https://hackmd.io/_uploads/r1du5OxWMg.png)
    - 目前code是沒問題了，但實測還需要在驗證，但這是明天的事了www，最近太累了QAQ。

## Day120
#### 學習重點 : Spring Data JPA - 加入購物車功能dto
- dto檔 ⭐⭐⭐⭐⭐
    - 由於一開始設計的AddToCart需要接收過多資訊，我將其整合後改成Request來傳入，避免不必要資訊傳入。
    - 目前設計傳入userId、merchandiseId、quantity，後續可以再新增~
    ![image](https://hackmd.io/_uploads/Skbweg-C-l.png)
- CartController ⭐⭐
    - 目前只單純回傳String來判斷是否加入，雖然好像不會有false的情況（因為我都先throw new RuntimeException🤣），之後整合進小專案後，再加入Response<>的功能！
- 業務邏輯問題 ⭐
    - 我目前還有個問題未解決 --> 一般來說，加入購物車不會影響到stock，只會檢查stock是否大於購物車+quantity而已，這部分的邏輯明天得再修改一下！

## Day121
#### 學習重點 : 小專案整合計畫
- 加入購物車功能完結 ⭐⭐⭐
    - 目前將addToCart功能的邏輯修正後，基本上應該可以使用了！
    - 接下來我要將Spring Data JPA的測試檔案整合進使用者管理系統小專案，這部份是不小的工程，我先整理出需要更動的部分！
- 專案整合 ⭐⭐
    - **Dao層** : JDBC全面改為Spring Data JPA。
    - **Service層** : 新增商品、購物車Service。
    - **Controller層** : 新增商品、購物車Controller。
    - **Model** : 除了商品購物車，需要新增User的ID欄位。
    - **Route** : 修正網址太散亂問題。

## Day122
#### 學習重點 : 小專案整合-Part1
- 類別更新 ⭐
    - 將Merchandise、Cart、User更新至小專案中。
    - Dao層改成Interface的JPA版本。
    - DTO更新AddToCart的Request請求。
    - Service、Controller加入Merchandise、Cart類別。
- 資料庫更新 ⭐
    - 原本使用的myjdbc棄用，更新為mall。
    ![image](https://hackmd.io/_uploads/Hym2pPmAbl.png)
- User更新 ⭐
    - 加入UUID成員。
- Github更新 ⭐⭐
    - 由於我一直不是很知道怎麼做版本控制，想說透過小專案來學習一下w，我先建了一個Repository，存了第一版的小專案！
    ![image](https://hackmd.io/_uploads/Hyx8aD7C-l.png)

## Day123
#### 學習重點 : 上架商品流程概念與實作
- 流程分析 ⭐⭐⭐⭐⭐
    - 我稍微理了一下路線 : User登入 ⮞ 取得Authorization標籤 ⮞ 送至JwtUtil驗證 ⮞ 驗證成功 ⮞ 取下Bearer中的account ⮞ 連帶原本商品資訊送進merchandiseService ⮞ 透過account找到User ⮞ 將owner設置為user的id ⮞ `.save(merchandise)`。 
- 實作 ⭐⭐⭐
    - 我先在dto檔中建立了一個request專門處理上架商品的資訊類別 :
    ![image](https://hackmd.io/_uploads/SJ012o7R-g.png)
    - 接著在Controller中設置驗證與取Bearer流程 : 
    ![image](https://hackmd.io/_uploads/S1rI2smRbx.png)
    - 由於Controller不能跨到Dao層，因此account先傳入再找使用者，以免使用者資訊洩漏 : 
    ![image](https://hackmd.io/_uploads/HyrnhsQAWe.png)
- 未做部分 ⭐⭐
    - 目前的request有點太過冗長，感覺可以找方法簡化？
    - Response尚未加入，而且Exception處理需要應用上去，不然有點奇怪！

## Day124
#### 學習重點 : BeanUtils簡化 & 商品Response回應
- BeanUtils ⭐⭐⭐
    - 這個工具看名字就知道是針對Bean在做事的ww，但我沒有要深入研究它，會使用其中的method就好。
    - **copyProperties** : 放入source、target即可以達成複製屬性的動作，儘管好用 ⮞ 但要注意該方法是shallow copy，亦即target物件內的成員指向的記憶體位置與source是相同的，但因為我只是要更新數據，目前不會影響到。
- Response回應 ⭐⭐⭐
    - 我將包含cart、merchandise的回應改為Response<>類別，並統一使用ExceptionHandler來處理，但目前還有些問題有待明後天解決！
    ![image](https://hackmd.io/_uploads/Bk-NMVLCWg.png)

## Day125
#### 學習重點 : Response回應完善
- 完善購物車查詢的回應 ⭐⭐⭐
    - 原本回應購物車前端的是單純的 `List<Cart>`，但我想在查詢後取得的是一個Response以統一回應格式，並且順便加入狀態。
    - 以下是我改之後的結果 : 
    ![螢幕擷取畫面 2026-05-05 232551](https://hackmd.io/_uploads/BJjotdg-Mx.png)
    - 實測成功！
    ![螢幕擷取畫面 2026-05-05 232846](https://hackmd.io/_uploads/BJ8hKOl-Mg.png)
    ![螢幕擷取畫面 2026-05-05 233029](https://hackmd.io/_uploads/BygFnFdxZfg.png)

## Day126
#### 學習重點 : 商品類別新增、FetchType淺分析
- Categories新增 ⭐
    - 今天開始研究商品類別部分，有點棘手，但感覺很好玩ww
```SQL=
CREATE TABLE categories(

    id          VARCHAR(100) NOT NULL PRIMARY KEY,
    name        VARCHAR(36)  NOT NULL UNIQUE,
    parent_id   VARCHAR(100),

    CONSTRAINT fk_parent_category
        FOREIGN KEY (parent_id)
        REFERENCES categories(id)
        ON DELETE SET NULL

)
```
- FetchType分析 ⭐⭐⭐⭐⭐⭐
    - 在JPA當中，當我們要跟資料庫要資料時會有兩種型態 : Eager 以及 Lazy。
    - Eager就像當我們查詢購物車時，若User被設置為Eager，則會抓取User內的所有成員。
    - 而Lazy則是只會儲存Cart所存的userId，其餘account、password都是null，等到使用user.getAccount()之類的才會再使用SQL語法做查詢。
    #### N+1 query problem
    - 這個問題出現在使用Eager loading時，若我們想查詢N個購物車資料，一般來說只需要 `SELECT * FROM cart` （一次查詢）即可，但若是User設為Eager，則會觸發N次查詢User，造成N+1 problem。
    #### N+1 problem solving
    - 解方有兩種，一種是利用Lazy loading代替Eager loading，當我們不需要取得User其他資料時可用。
    - 另一種則是Join fetch，針對需要關聯其他資料表時，才做查詢，這個需要自行寫JPQL，之後來研究，聽說很強大！

## Day127
#### 學習重點 : Category三層架構、N+1問題實例與解決
- 今日工程 ⭐
    - 今天我大概花了近2個小時都在處理Category的架構以及搜尋Category時的連鎖反應。
    - 我盡量縮小篇幅讓筆記更清晰。
- Category三層架構、N+1問題實例與解決 ⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐
    - **Dao層** : findById、findByParent_id
    - **Service層** :
        -  **🚨搜尋時的連鎖N+1結構問題** : 搜尋一個category時，會列出id、name，還有其「parent的category」
        
            **源頭** : model存的是 ⮞ `@ManyToOne Category category`，但parent也是一個Category，因此有可能當我查D分類，結果把D的祖宗十八代都挖了出來。
            
            **防線不足** : 雖然FetchType為Lazy，但當RestController將物件序列化成JSON格式時，還是會呼叫所有的Getter，又變回Eager。
            
            **解方** : 在DTO加入CategoryResponse讓Category成員在Service層先只抓取id而不傳整個category。
            ![image](https://hackmd.io/_uploads/HkzODX90-x.png)
    - 透過上述的結構處理，後續的應對我都用同一套邏輯下去寫，就萬事OK啦~
    - 針對Parent_id查找時需列出List問題（這也是N+1問題來源，但被我用CategoryResponse解決啦~），我利用for迴圈來做。
    ![image](https://hackmd.io/_uploads/rJ2FqmcRbg.png)
    - **Controller層** : /add、/parent/{parent_id}、/{category_id} 三個Endpoint資源請求。
    ![image](https://hackmd.io/_uploads/HyRVo75Cbx.png)
- PostMan實測
    ![image](https://hackmd.io/_uploads/SJHnp7qRZe.png)
    ![image](https://hackmd.io/_uploads/rkofRQ90-x.png)

## Day128
#### 學習重點 : 商品的Category分類問題、Pageable物件
- 商品的所屬「分類」屬性 ⭐⭐⭐⭐⭐⭐⭐
    - 原本👉Merchandise當中有個屬性是 `String category`，單純存商品所屬分類。
    - 現在👉我改成 `@ManyToOne Category category` 對應Category的category_id外鍵欄位。
    - 但這衍生幾個問題 : 
        - Q1 : 當我使用`[POST] /list` 上架商品時，輸入的category_id若使用 `categoryDao.findCategoryById` 來找的話，並回傳Category的話，很有可能在最後回傳 `Response<Merchandise>`時觸發Getter導致N+1問題。
        - Q2 : 如果我要「利用category_id找商品」時，若輸入的是parent，那我是不是要列出所有子分類的所有商品呢？
    - 我今天 **只針對Q1做解決** 
        - S1 : 建立 `MerchandiseResponse`，其中category_id就**單純用String存取**，且利用categoryDao要取出分類時，只使用 🌟**getReferenceId**🌟（Proxy自動生成只含id的Category物件）而不使用findCategoryById。
        ![image](https://hackmd.io/_uploads/S1mKwPjRWl.png)
- Pageable簡單使用 ⭐⭐⭐⭐⭐
    - 我目前想單純根據category_id找尋商品，原本是想用List儲存一連串商品，但後來發現似乎用分頁物來做更適合，之後應該會花些時間來學習。
    - 目前先單純利用範例程式來玩玩看w
    ![image](https://hackmd.io/_uploads/B1d08Ps0be.png)
    - 其中的id會根據Merchandise所寫的 `@Id` 變數來連結（注意!!不是連結TABLE而是類別變數）

## Day129
#### 學習重點 : 解開我腦袋的結、解決昨日Q2
- 腦袋打結部分 ⭐⭐⭐⭐⭐⭐⭐⭐
    - Q : 我在Dao層寫findByCategory，JPA **是怎麼透過Category「物件」** 去找商品的？
        - 1️⃣ JPA會到Category找 `@Id` 所對應的的變數，並取出該id的值，
        - 2️⃣ SQL語法連結Merchandise的category_id連結剛剛的id ⮞ `SELECT * FROM merchandise WHERE category_id = ?（此處填入取出之id）`
    - Q : 我寫的 `getReferenceById` 如果 **填入假id會怎樣**？
        - 1️⃣ JPA完全信任該id，捏出一個Category物件只有id有值（我填入的），其餘屬性都是null。
        - 2️⃣ 當我將假Category填入Merchandise後並 `.save`，就會報錯，以下是我的測試報錯紀錄 : 
        ![image](https://hackmd.io/_uploads/rytxfV30Zg.png)
        JPA跑去Category想找該id，結果找不到。
        - 3️⃣ 解決 : 先findById檢查Optional是否為空再進行後續動作。
- 解決昨日Q2 ⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐
    - 我的初始想法 : 
        > 由於Category有parent的結構，因此findCategoryByCategory時，可以列出parent的下一代（但不列出更後代）
        > 
        > 我希望透過搜尋Category_id列出其下所有 **最後一代** 的商品（因為商品一定是leaf分類）
    - 我的邏輯 : 
        > 使用for loop 不斷 findCategoryByCategory，直到回傳Response中的data是[]，則該Response中的Category為leaf。
    - 但這個邏輯會造成SQL多次查詢影響效能，因此在AI的幫忙中，我選擇以下思考邏輯 :
    - 最終邏輯 : 
        - 1️⃣ 於Category新增 `@OneToMany List<Category> chidren` 結構，實現parent-children雙向結構，似乎跟Doubly Linked List扯上關係了ww。
        - 2️⃣ 於MerchandiseService當中寫遞迴方法來找尋leaf
        ![image](https://hackmd.io/_uploads/SyZvsD30bl.png)
        - 3️⃣ 於MerchandiseDao修改Page方法為 : 
        ![image](https://hackmd.io/_uploads/H1YTov20-x.png)
        加上In關鍵字讓SQL由 `WHERE category_id = ?` 改成 `WHERE category_id IN (?, ?, ?, ...)`

## Day130
#### 學習重點 : Pageable類別
- `PageRequest.of(int pageNumber, int pageSize, Sort sort)` ⭐⭐⭐
    - 這個方法是最常被拿出來使用的，其接收了建立Pageable物件的參數，包含頁碼、大小、Size排序方式。
    - 一般來說會在Controller使用 `@RequestParam` 接收客戶端傳來的參數去決定Pageable的物件內容。
    - ine pageNumber、int pageSize意即頁碼與每頁筆數，不太需要說明ww。
    - 而Sort物件會決定排序方式 : 根據 `Sort.by(String... properties)`。
    ![image](https://hackmd.io/_uploads/S1SPZZ0AZl.png)
    - 一般Sort建立物件時會default Direction（如上圖） : ASC，意即升序，會根據properties來排序，SQL語法即 : `ORDER BY ... ASC`。
    #### 若propeties是List會怎麼排？
    - 當properties是多個的話，排序「根據」會由index 0開始排。
    - 假設 `Sort.by("price", "id")`，會先根據price排序，排完若價格重複再根據id去排 : **這很重要**。
    - 若只根據price去排，可能 **100個商品都是87元**，翻到第二頁可能就會看到重複的商品（因為 **每次翻頁都是不同pageable物件**，意即 **重排**，若沒有固定順序，則有可能遇到重複）。
- Pageable簡易練習實作 ⭐⭐⭐
    - 我想說拿User來開刀一下ww，寫一個可以列出使用者的端點。
    - 我寫了甚麼？ 
        - 按照前幾天的學習，我同樣新增了一個UserResponse專門來過濾掉敏感資訊。
        - 接著我在UserService當中寫一個listUsers的method : 
        ![image](https://hackmd.io/_uploads/rJZDKbCCWx.png)
        - 單純先練習，之後有時間再加上權限驗證ww。
        ![image](https://hackmd.io/_uploads/SyrUO-AAZg.png)
        ![image](https://hackmd.io/_uploads/rJZTYbRAZx.png)

## Day131
#### 學習重點 : Delete實作與問題解決
- delete實作 ⭐⭐⭐⭐⭐
    - Merchandise的刪除問題
        - 若要賣家想刪除商品必須考慮到一個問題 : 若有使用者加入該商品進購物車，則 **必須先移除** 購物車資料庫中所有該商品連結的資料。
        - 若直接刪除商品，會 **有Foreign Key的限制** 而無法執行delete。
        - 以下是我的實作部分 : 
        ![螢幕擷取畫面 2026-05-11 230923](https://hackmd.io/_uploads/H1svF_eWGl.png)
        - 其中先執行刪除購物車的 `deleteAll(cartDao.findByMerchandise(m));`
        - 再來才是刪除商品，後回傳MerchandiseResponse。
    - User刪除實作
        - 這部分我沒有做很細，只是單純透過id去刪除，之後可能需要加個驗證之類的w
        ![螢幕擷取畫面 2026-05-11 231903](https://hackmd.io/_uploads/ry4OY_eZfe.png)

## Day132
#### 學習重點 : 權限管理系統設計-1
- 權限管理系統初見（Role、Permission、Role_Permission三劍客）⭐⭐⭐⭐⭐⭐
    - 在使用者管理系統當中，權限需要 **利用多對多的關聯資料表** 來做處理，讓不同使用者有不同的ROLE，又不同的ROLE有不同的PERMISSION列表。
    - 因此在Entity的部分會需要針對User成員再新增一個Role來授予角色。
    - 而針對儲存權限要有Permission實體，接著透過Role_Permission實體連結Role達成權限系統與使用者系統的連接。
    ```SQL=
    CREATE TABLE role_permission(
        id VARCHAR(100) PRIMARY KEY,
        role_id VARCHAR(100) NOT NULL,
        permission_id VARCHAR(100) NOT NULL,

        CONSTRAINT fk_role FOREIGN KEY (role_id) 
            REFERENCES role(id) 
            ON DELETE CASCADE ,
            
        CONSTRAINT fk_permission FOREIGN KEY (permission_id) 
            REFERENCES permission(id) 
            ON DELETE CASCADE,
            
        CONSTRAINT uq_role_permission 
        UNIQUE (role_id, permission_id)
    );
    ```
    - 這邊透過CONSTRAINT約束ID，並透過ON DELETE CASCADE當role或permission被刪除時同時刪除這張表有關連的資料。
    - 並限制UNIQUE關聯role_id與permission_id，限制一role與一permission最多只有一次對應。

## Day133
#### 學習重點 : 權限管理系統設計-2
- 權限管理實體操作 ⭐⭐⭐⭐⭐
    - 針對三劍客的新增就很簡單，Role與Permission都是id、name。而RolePermission則是利用與Cart相同的 `@ManyToOne` 來連結兩者，至於FetchType一樣設成Lazy。
    - **RolePermission.Dao層** : 我加入了 `boolean existsByRoleAndPermissionId(Role role, String permissionId)` 作為尋找該role是否有對應之權限。
    - 這邊很神奇的是可以寫String PermissionId作為參數，而不是Permission物件（這樣還要自行建物件），這是因為JPA在method語法發現是Permission「的」Id，因此自動找到 Permission實體 `@Id` 註解的String id，這樣超級省空間的啦~
- 實際User刪除功能應用 : ⭐⭐⭐⭐⭐
    - 流程 : 透過account找操作者（operator），透過DTO設計的刪除Request找目標刪除者（target）➝ 判斷操作是否為本人or具刪除權限之人 ➝ 若不是則拋出例外，若是則執行操作並連帶購物車一同刪除 ➝ 回傳Response。
    ![螢幕擷取畫面 2026-05-13 225249](https://hackmd.io/_uploads/Bk9BFdebzx.png)
    - 中間註解的部分是原本問AI時建議列出的權限List，一個一個match看看是否有符合刪除權限的ID，但總覺得要把整張表翻出來，還不如精準打擊。

## Day134
#### 學習重點 : 權限管理系統設計-3、Method重構
- UserRegistration重構 ⭐⭐⭐⭐⭐
    - 在前期，我都是利用Optional的聲明式寫法來完成，但在UserRegistration中，其實適合流程寫法，因此我修正了下，加上Role設定，並移除了BeanUtils在註冊時的使用（避免資安疑慮）。
    - 不過至於Temp_password的部分我還需要再新增欄位做判斷，這邊先擱置w。
     ![image](https://hackmd.io/_uploads/rJA-9DXyMg.png)
- UserLogin重構 ⭐⭐⭐
    - 由於Login改成回傳UserResponse（為隔離User的敏感資訊），因此前端所設置的token要挪到Service來做，因此稍微改了下流程。
    ![image](https://hackmd.io/_uploads/H15C5PQyGx.png)

## Day135
#### 學習重點 : 權限管理系統設計-4
- 新增更多權限應對不同操作 ⭐⭐⭐
    - 根據不同API，就會有不同的權限對應，因此我針對不同功能設計了一些權限（但尚未完善ww）
    - 以下是我改良的userList列出所有使用者的分頁查詢功能。
    ![image](https://hackmd.io/_uploads/S1WGr24yMl.png)

## Day136
#### 學習重點 : 完善臨時密碼功能的流程、User剩餘Response回應完善
- 臨時密碼完善 ⭐⭐⭐
    - 由於在之前寫registration 以及 login時，都是暫時先以「TempPassword是否為null」來偵測是否為使用臨時密碼登入。
    - 但在一般情況中，新用戶註冊時，密碼會先被放置在TempPassword欄位，待登入後再轉至password。
    - 因此我根據這個邏輯重構了下registration以及login的程式。
    ![image](https://hackmd.io/_uploads/Bk0Q1M8yzg.png)
    ![image](https://hackmd.io/_uploads/ByCZyMLyGg.png)
- User剩餘Response完善 ⭐⭐⭐
    - 由於上次專案結束前，沒有加入Response東西有點多，包含忘記密碼的兩個method，因此我針對這部分改成了 `Response<UserResponse>`。
    ![image](https://hackmd.io/_uploads/S1ReeM8kfe.png)
    ![image](https://hackmd.io/_uploads/Hk1llGUkGg.png)

## Day137
#### 學習重點 : User全面進化
- User成員新增 ⭐⭐⭐⭐⭐
    - 我在User成員中，新增了 `last_login_time` 以及`password_expired_at` 的變數，用於存取登入時間以及密碼過期時間（密碼過期訂90天）。
    - 移除 `first_time_login` 欄位，改用 `last_login_time` 是否為null來做判斷。
    - 由於model以及邏輯更動過大，導致login流程需要大改，我乾脆把login method **分離成幾個小method** 來做驗證 & 登入流程。
- Login流程職責分離 ⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐
    - 簡單來說有三步 : 取得登入狀態（使用原密碼or臨時密碼or錯誤密碼?）➝ 臨時密碼淨空 ➝ 執行登入流程。
    ![image](https://hackmd.io/_uploads/rJHsOXPyfg.png)
    - 前面兩者的邏輯都很簡單，不多贅述，而最後的流程需要考慮到多種面向，我以Response Code來做分流，如上圖紅字所示。
    - 至於動作層級部分則是 : 
        ```
        TOP : 偵測臨時密碼登入，導向改密碼頁面
            - 若用戶未登入過，回傳001
            - 若是一般用戶，回傳01
            
        SECOND : 偵測密碼過期登入
            - 無論用戶是否登入過，都導向改密碼頁面，回傳02
            
        THIRD : 偵測首次登入，回傳00
        
        LAST : 常規登入，回傳0
        ```
    - 按照上述狀態做偵測，層級我排了好久QAQ，主要是要應對各種奇葩登入狀態，像是「註冊後不登入，等到密碼過期才登入」、又或是「註冊後不登入，密碼過期後，還忘記密碼，使用臨時密碼登入🤡」。
    - 根據上述層級安排，我將proccessLogin的流程獨立出來，這邊就不放了w。
- UserResponse建構式新增 ⭐⭐⭐
    - 由於一般我都是利用User的資料過濾敏感資訊後放到UserResponse物件。
    - 那麼我不如在UserResponse寫個建構式是接收User物件再把相對應的資訊copy過去即可！
    ![image](https://hackmd.io/_uploads/SJaf0QwJfl.png)
- JWT token位置更換 ⭐⭐⭐
    - 根據網友的建議，我將原本放在Response Header的token改放在Response Body。
    - 而我的Response Body又是跟著 `Reponse<>` 走的，因此我加入了 `LoginResponse` 的DTO檔，裡面包含一般UserResponse還有一個token。
    - 實測長這樣 : 
    ![image](https://hackmd.io/_uploads/HkqqaXvyGe.png)

## Day138
#### 學習重點 : 購物車系統更新流程
- 「新增商品至購物車」流程 ⭐⭐⭐⭐⭐
    - 原版程式 : User_Id & Merchandise_Id找Cart，接著根據quantity是否超過stock決定要不要更動DB（只有兩種狀態），最後利用BeanUtils複製Merchandise到Response作為回應。
    - 更新後程式 : 透過**User** & Merchandise找Cart，呼叫判斷Quantity函式來決定更新狀態 : （由於User狀態碼是0開頭，我想說Cart給2開頭好了）
    ![image](https://hackmd.io/_uploads/HJgEKFdkGg.png)
    ![image](https://hackmd.io/_uploads/B1YStt_1ze.png)

## Day139
#### 學習重點 : 訂單系統成立
- Order Model建立 ⭐⭐⭐
    - 首先是關於Model的成員，由於訂單的狀態有很多種，因此我想說利用Enum來完成，不過總感覺還得再修，這樣寫不太正式w。
    - 除了Status之外，成員還有 : 建立時間、使用者、商品、商品下訂數量。
    ![image](https://hackmd.io/_uploads/S1xT415kfe.png)
    - 圖中的 `@EnumerateType(EnumType.STRING)` 亦即將Enum的狀態以成員名稱存入。
    - 其餘其實沒什麼好說的w，就跳過。
- Order與Cart的狀態交互疑問 ⭐⭐⭐
    - 我目前是偵測當Cart中該商品符合可成立訂單的邏輯（也就是購物車商品數量大於Stock為原則）即 --> 在Order DB留下UNPAID的訂單紀錄。
    - 至於後續若訂單因付款逾時或者包裹問題等需要再找辦法解決。
    - 目前Code如下 : 
    ![image](https://hackmd.io/_uploads/B1PsVy9yGl.png)

## Day140
#### 學習重點 : 購物車關聯商品問題
- Cart問題 ⭐⭐⭐⭐
    - 在我實作訂單系統時遇到一個問題，就是建立訂單是根據購物車內容，但是一般來說一個購物車不只裝「一種商品」，這代表我的Cart model是有問題的，我應該使用 `List<Merchandise>`，而非單純的Merchandise。
    - 我必須再建立一個CartItem model去對應單一商品及數量，而Cart就會存取多個CartItem，透過CartItem作為中間表來連結Cart與Merchandise。
    - 以上是我的想法，由於今天比較沒什麼時間，明天再來做！

## Day141
#### 學習重點 : CartItem model設計
- Cart與CartItem的連結 ⭐⭐⭐⭐
    - 由於Cart所關聯的部分太多，我可能會拆分成幾天來做，今天先重建了Cart的model型態，另外新增了CartItem model。
    - 對於關聯的性質我以結構圖列出 : 
        ```
        - Cart 
            - id
            - user
            - @OneToMany cartItemList --> 多組CartItem
                          |
                          |
        - CartItem -------|
            - id
            - @ManyToOne merchandise
            - @ManyToOne cart
            - quantity
        ```
    - 原本 : 我的Cart是帶有Quantity以及Merchandise的單一對應，搜尋時需要利用for迴圈跑每個關聯該User的Merchandise。
    - 現在交給CartItem處理，而Cart連結User並帶有OneToMany的CartItem，這樣我們只需要掌握 `List<CartItem>` 即可啦~
    - 至於實際的Service更改，我明天再來完成！

## Day142
#### 學習重點 : CartItem與Cart的級聯關係
- 級聯問題 ⭐⭐⭐⭐
    - 透過昨天的圖表，可以寫出以下的程式 : 
    ![image](https://hackmd.io/_uploads/SJ-fSeRkfe.png)
    - 其中的orphanRemovel表示說若我們針對list刪除某個item後save Cart時，item也會被自動於item DB中抹除。
    - 問題 : 當我針對cartItemList做 **新增** 動作再save時，卻無法正確將item儲存到item DB當中，這個問題真的超級困擾，我目前已經嘗試過各種辦法都會報錯QAQ，希望明天能夠找到正解！

## Day143
#### 學習重點 : CartService完善、Response & 例外處理Enum化、Order設置
- 昨日級聯問題解決 ⭐⭐⭐⭐⭐
    - **解決辦法** : 原來是我眼殘加上腦袋當機，一直以為我有加Cascade，然後我又神奇的沒發現這個問題，問AI又不夠精準沒找到問題ww，感謝網友們的幫忙啦~
    - 最後於Cart中CartItem加上Cascade.ALL即可正常運作ㄌ。
- Exception、Response改為Enum初始化 ⭐⭐⭐⭐⭐⭐
    - 由於每次都要打errorCode、returnCode之類的東西，一來不嚴謹、二來很煩人，因此我獨立了出這些「狀態」成enum檔，如下所示 : 
    ![image](https://hackmd.io/_uploads/Sy_4me1xfx.png)
    - Response也是相同概念，根據不同檔案寫出相對應的Enum檔，包含rc、rm的常數定義。
- CartService完善 ⭐⭐⭐⭐⭐⭐⭐⭐⭐
    - 經過昨天與今天的修改及問題解決，我終於將Cart全面更新為Cart與CartItem的組合，讓Cart連結CartItem與User，而CartItem連結商品與Cart。
    - 至於addToCart的method經過修改後如下所示 : 
    ![image](https://hackmd.io/_uploads/Sk4BfZ1gGe.png)
    ![image](https://hackmd.io/_uploads/rkctsW1eze.png)
    - 我想要紀錄一個東西 : 
        - **Cart NPE問題** : 在Cart中所連結的CartItem是使用LAZY加載，因此在撈出Cart時，內部的CartItemList是個沒有被初始化的代理物件。
            當「原有商品」被撈出來加數量or刪數量時，若沒有 **特地加載** `item.setCart(cart)`，就直接save的話，會因CartItemList的缺失導致Response轉換時出現NPE問題。
- Order訂單建立 ⭐⭐⭐⭐⭐⭐
    - 基於一般商城都是「**選取**」cartItem進行結帳，因此我使用 `List<String>` 的DTO形式，讓使用者於前端送出要下訂的cartItem ID列表，再作後續邏輯處理。
    - 首先，先跑過所有cartItem，檢查是否都符合下訂標準(下訂時，庫存大於下訂數量)，只要 **其中有一個不符合** 的就直接throw Exception，意即 **不執行下訂流程**。
    - 因此傳入「訂單建立方法的參數」的itemList都是通過檢查的，根據cartItemList一個一個建立Order，再使用removedItemList記錄被刪除的item，最後再一次性removeAll。
    ![image](https://hackmd.io/_uploads/SkO-FWygzl.png)
    ![image](https://hackmd.io/_uploads/SkHzKZyxzx.png)

## Day144
#### 學習重點 : Order頁面取得與Webhook概念
- Order頁面取得 ⭐⭐⭐⭐
    - 原本我的OrderResponse是取得MerchandiseId，但我覺得改成MerchandiseResponse比較好ouo。
    ![image](https://hackmd.io/_uploads/ry8-Q8egMe.png)
- Webhook概念 ⭐⭐⭐⭐⭐⭐
    - 由於訂單有許多狀態，像是付款、運送中、包裹作業中...等等，我們需要藉由一個「中間人」來發送API告知Controller去改狀態。
    - 換另外一種講法 : 若今天外送員接到單時，總不可能一直跟店家確認餐點是否做好（這種稱為Polling輪詢），一定是店家做好餐點後「主動」告知外送員。
    - 以上述情境來看，原本作為接收端的店家反而主動告知，這就是webhook的邏輯。
    - 不讓client端一直Polling，而是待店家/物流中心..藉由webhook發送API告知Controller更新狀態。
    - 所以Webhook是甚麼？ : 一個通訊概念，當A系統發生變動，主動發送API給B系統做更新。

## Day145
#### 學習重點 : Webhook實作、綠界API串接
- Webhook API實作 ⭐⭐⭐⭐
    - 為了對應外部系統，我們需要設計一些endpoints來給外部Webhook呼叫，因此我獨立出了一個WebhookController。
    ![螢幕擷取畫面 2026-05-25 190946](https://hackmd.io/_uploads/ByLtrdx-fx.png)
    - 由於串接的是外部系統，因此不能使用Response這種自製的物件作回傳，因此我回歸到使用ResponseEntity這種正規的Http status code回傳。
    - 而我在OrderService層當中寫了一個處理付款更新狀態的method : 
    ![螢幕擷取畫面 2026-05-25 191651](https://hackmd.io/_uploads/rynTSdlZzx.png)
- 綠界串接 ⭐⭐⭐⭐⭐⭐
    - 由於綠界的付款API測試無法直接連到localhost，因此需要先使用 **ngrok**，它可以生成一個臨時網址連到本地端，以供外部系統連結我設計的API。
    - 以下是ngrok在終端機執行的圖 : 
    ![螢幕擷取畫面 2026-05-25 192030](https://hackmd.io/_uploads/BkbSDueZMe.png)
    - 下面是綠界串接的流程 : 填入訂單編號（MerchantTradeNo）、串接API網址（ReturnURL）
    ![螢幕擷取畫面 2026-05-25 192545](https://hackmd.io/_uploads/rJ9rvdlWGx.png)
    ![螢幕擷取畫面 2026-05-25 192600](https://hackmd.io/_uploads/By48P_gbfx.png)
    - 接著我在自己一個與綠界對接DTO檔，getMerhantTradeNo放入orderDao去尋找即可啦！
- 實測結果
    - 以下是付款成功後，**ngrok、綠界、本地端DB** 的結果圖
    ![螢幕擷取畫面 2026-05-25 193003](https://hackmd.io/_uploads/rJR8wdxbGe.png)
    ![螢幕擷取畫面 2026-05-25 193018](https://hackmd.io/_uploads/SyEvD_eWzg.png)
    ![螢幕擷取畫面 2026-05-25 192844](https://hackmd.io/_uploads/rkdvD_lZMl.png)

## Day146
#### 學習重點 : @Transactional基礎
- 甚麼是Transactional？ ⭐⭐⭐⭐⭐
    - @Transactional註解常用於「交易」，當Service在執行業務邏輯時，若出現Runtime Exception或Error，則會進行Rollback(回溯)，讓整筆交易能夠保持一致性 ➝ 資料庫**不**會出現 : 「買家確實付完錢，但賣家卻沒收到錢」的情況。
    - 只有兩種情況 : 全部成功、全部失敗。
- 如何使用Transactional？ ⭐⭐⭐⭐⭐⭐
    - 使用Transactional有兩個準則 : 只能掛在public方法、只掛可能需要rollback的method，且掛在依賴的最外層即可。
    - 為甚麼只要掛在最外層呢？因為Transactional是根據Exception來做rollback的，只要依賴的某一層出現規定例外則會自動rollback。
    - 例子 : 像是我在刪除使用者時，會先刪除使用者的購物車相關資料，再刪除使用者。若在其中兩行執行的過程出錯了，我不應該讓資料出現殘缺，因此需掛上Tansactional。
    ![螢幕擷取畫面 2026-05-26 100405](https://hackmd.io/_uploads/r13aPOg-fg.png)
    ![螢幕擷取畫面 2026-05-26 100719](https://hackmd.io/_uploads/H1bAwOxWfg.png)
- 實測 ⭐⭐⭐⭐⭐⭐
    - 若我沒有掛上@Transactional（故意註解掉），又故意在兩行之間報錯就會發生以下事情 :
    ![螢幕擷取畫面 2026-05-26 101454](https://hackmd.io/_uploads/ryKADdl-ze.png)
    ![螢幕擷取畫面 2026-05-26 101436](https://hackmd.io/_uploads/B1A0vdl-ze.png)
    - 結果 : cart的user資料消失了，但使用者沒消失，這就是不一致性。
    - 若我掛上Transactional，則就會自動rollback。

## Day147
#### 學習重點 : rollback分離解決不一致性、kafka概念
- rollback問題 ⭐⭐⭐
    - 我們如果連結外部交易系統時出問題，Transactional無法管到外部系統，因此必須「分離」訂單的建立、外部系統的webhook交易。
    - 總結來說 : 建訂單 & webhook交易必須分離。
- kafka ⭐⭐⭐⭐
    - 初見 : 簡單來說，kafka是用於管理事件流的傳遞，增刪等的平台，我們可以建立事件並傳至其他系統當中。
    - 認識 : 而kafka有分為三個角色 : Producer、Broker、Consumer，分別為建立事件、管理事件、接收訂閱事件。
    - 管理 : kafka中，所有資料都是事件相關，存於Broker的Topic當中，且資料有持久性。
    - 我想說之後剛好配合後續mailService改良，所以先來看看這個酷東西！

## Day148
#### 學習重點 : kafka引入
- kafka套件引入 ⭐⭐
    - 首先當然是在 `pom.xml` 當中加入dependency啦~
    ```xml=
    <dependency>
        <groupId>org.springframework.kafka</groupId>
        <artifactId>spring-kafka</artifactId>
    </dependency>
    ```
    - 接著是在application設定Port與序列化參數
    ```yml=
    spring.kafka.bootstrap-servers=localhost:9092
    
    spring.kafka.consumer.group-id=my-group
    spring.kafka.consumer.auto-offset-reset=earliest
    
    spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
    spring.kafka.consumer.value-deserializer=org.apache.kafka.common.serialization.StringDeserializer

    spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
    spring.kafka.producer.value-serializer=org.apache.kafka.common.serialization.StringSerializer
    ```
- Service層的設置 ⭐⭐
    - 首先當然是設定Producer與Consumer的基本送訊息與接訊息的設定。
    - 在Producer當中設定 Template建立事件、在Consumer當中設定 Listener監聽事件。
    ![image](https://hackmd.io/_uploads/ry8JCRSefg.png)
    ![image](https://hackmd.io/_uploads/rkIS00BlGe.png)
    - 由於這幾天太忙了，我稍微分割一下進度，不然太難受了QAQ。

## Day149
#### 學習重點 : kafka部屬與測試
- 部屬 ⭐⭐⭐⭐⭐⭐
    - 簡單來說就是要讓Kafka server運行在本地端隨時監聽事件，而一般選用的是`port:9092`
    - 問題 : 我明明是去官網下載，但不知道為甚麼部屬的時候就是一直出錯QAQ，只能求助AI，雖然還是看不太懂。
    - 我這邊先簡單總結問題點（但我還沒研究，只是先暫時請AI讓我能正常部屬QAQ） : 
        - 使用.bat生成隨機UUID時，路徑太常無法正常執行。
        - Kafka 4.X版本之後淘汰了Zookeeper並將Kraft的資料夾拆掉放到config的server.properties中，所以中間路徑都要修改。
        - 最後的報錯 : `Because controller.quorum.voters is not set...` 似乎跟Windows腳本有問題相關，AI給出的解方式直接使用Java環境跟Kafka核心溝通，跨過腳本的步驟，最後即正常執行~
 - 實際測試 : ⭐⭐⭐⭐ 
     ![image](https://hackmd.io/_uploads/B1AoMVwefe.png)

## Day150
#### 學習重點 : kafka正式取代runnable發送信件
- kafkaProducer、kafkaConsumer ⭐⭐⭐⭐
    - 首先是處理事件流的部分，先將Template改成接收 `<String, DTO>`，這邊接收DTO的原因在於可以將mailAddress、subject、body包裝起來，這樣日後只需修改物件即可。
    ![image](https://hackmd.io/_uploads/BkONAf_xMe.png)
    ![image](https://hackmd.io/_uploads/SkbfkQugzx.png)
- DTO : passwordResetMailRequest ⭐⭐⭐⭐
    - Lombok註解 : `@AllArgsConstructor`、`@NoArgsConstructor`，方便做建構動作。
        ![image](https://hackmd.io/_uploads/BkGsxXugGx.png)
- Jackson 正/反序列化 ⭐⭐⭐⭐
    - 簡單說Jackson是Java中正反序列化JSON與物件的 **重要中間人**。
    - 當Kafka收到一個DTO物件時，會先序列化成JSON格式，跟著Template一起送到目標Topic中。
    - 此時監聽收到一坨序列化後的JSON格式並指明DTO物件，Jackson嘗試轉回DTO時，會先建立一個空殼DTO物件，再靠著 **反射** 取得所有成員名稱，再根據按照Key一一對應Set上去。
    - 這就是為何需要加上Lombok的建構，因為若只有參數建構式，則Jackson在建立空殼時會報錯！
- 實測結果 : ⭐⭐
    - 成功！
    ![螢幕擷取畫面 2026-05-30 165840](https://hackmd.io/_uploads/S1FNPX_efe.png)

## Day151
#### 學習重點 : 信件模板及圖片檔附件
- Mail模板 ⭐⭐⭐⭐⭐⭐
    - 一開始我所設計的臨時密碼信件都只有簡單一句話，因此使用SimpleMailMessage套參數 : `mail, subject, body` 即可。
    - 但在一般常見的公司信件系統當中，都是利用HTML搭配圖片及按鈕等所組成。
    - 此時不能使用SimpleMailMessage來應付，而是需要使用MimeMessage來設定。
    #### 甚麼是MIME？ ⭐⭐⭐⭐⭐⭐⭐
    - 其全名為Multipurpose Internet Mail Extensions，亦即可支援多型態的檔案，文字、圖片、音訊等...。
    - 而在Java當中，有個MimeMessage的類別是專門在處理MIME的包裝，一般來說以多個MimeMultiPart組成MimeMessage。
    - 但是這個原生的套件有個問題，就是要自己手刻很多部分，很麻煩，而且序列化時可能會出錯。
- Spring **MimeMessageHelper** ⭐⭐⭐⭐⭐⭐
    - 由Spring設計的MIME包裝幫手，我們只需要以丟body參數即可，它會自動包裝成MultiBodyPart。
    - 實際長這樣 : 
    ```java=
    public void sendHtmlMail(
        String receiver, 
        String subject, 
        String content, 
        DataSource attachment){
        
        try {
            // 建立多型態信件
            MimeMessage message = javaMailSender.createMimeMessage();
            
            // 建立Mime包裝幫手，設置true代表是以多part組成
            MimeMessageHelper helper = new MimeMessageHelper(
                message, 
                true, 
                "utf-8");
            
            // 設置傳送對象、主旨、body(html要設置true才會以html讀取)
            helper.setTo(receiver);
            helper.setSubject(subject);
            helper.setText(content, true);
            
            // 設置圖片的BodyPart
            if (attachment != null) 
                helper.addInline(attachment.getName(), attachment);

            message.saveChanges();
            javaMailSender.send(message);
        } catch (MessagingException e){
            throw new RuntimeException(e);
        }
    }
    ```
    - 設置好Mail後，接著要設定的是「如何 **處理圖片** 自resources/static中讀取」的問題。
    - 由於最後專案被打包成jar時，**實體路徑會消失**，因此不能使用File類別找路徑的方式，而是使用的是Spring的ClassPathResource（ClassLoader），可以直接針對resources下的檔案位置做讀取，而不受打包的影響。
    ![image](https://hackmd.io/_uploads/BJjpSqFezx.png)
    - 在其中的ByteArrayDataSource是指我們在執行打包檔時，若要讀取，需指定型態（image/png）及讀取的數據流。
- 執行測試 ⭐⭐⭐⭐
    ![image](https://hackmd.io/_uploads/H1FrfqFlfx.png)
    ![image](https://hackmd.io/_uploads/rkayM5Kgfg.png)

## Day152
#### 學習重點 : AOP基本框架、JWT驗證加入AOP
- 回憶 ⭐⭐
    - 在 [Day40](https://hackmd.io/@learning-official/Java_learning#Day40) 的時候，我曾經有概略的帶過AOP的概念(好懷念ww)，而今天我把AOP的概念實作到了小專案當中。
    - 以簡單理解AOP的概念來看👉創造切面➝建立Pointcut➝執行邏輯
- 實際架構 ⭐⭐⭐⭐⭐⭐
    - 建立Aspect class，建立Pointcut，確認切入點 : 
    ![螢幕擷取畫面 2026-06-01 193543](https://hackmd.io/_uploads/SJr4udlbMx.png)
    - 我以User的JWT來練習，當偵測到非(Login、Registration、forgetPassword)的方法被execute時，會先到jwtAuth的切面來做驗證，不過這個偵測邏輯應該會在幾天內被我改掉ww，因為似乎可以直接設計一個註解來擋掉不需驗證的User功能。
    #### 甚麼是RequestContextHolder？
    - 簡單來說，當API打進來後，Spring會建立一個執行緒ThreadLocal記錄下請求的資訊，含有HttpServletRequest的資訊。
    - 而當我在切面當中呼叫RequestContextHolder.currentRequestAttribute時，會取得ThreadLocal的請求資訊。
    - 接著把request取出來後即可找到JWT資訊~
    - 而最後驗證完，再將取出的token的account設置回去(這樣才不需要Controller重複呼叫JWTUtil)

## Day153
#### 學習重點 : AOP註解優化、Around的Log紀錄
- UnAuth註解新增 ⭐⭐⭐⭐⭐⭐
    - 由於昨天的execution需要先排除「指定」的method，像是login、registration等。
    - 而今天我建立了一個註解(可以說是標記)，讓這個註解作用於需被排除的方法上。
    ![螢幕擷取畫面 2026-06-02 134107](https://hackmd.io/_uploads/SkrKd_xZfx.png)
    ![螢幕擷取畫面 2026-06-02 134149](https://hackmd.io/_uploads/rk2YuugZfe.png)
    - 透過標記method，讓我們可以直接在Before的偵測內容中，排除被 `@UnAuth` 標記的method。
    ![螢幕擷取畫面 2026-06-02 134310](https://hackmd.io/_uploads/SkBcuueZfx.png)
- Log切面設計 ⭐⭐⭐⭐⭐⭐
    - 在日誌紀錄中，通常會記錄一個method的開始事件與結束事件，因此會用Around來做，但由於around會經歷一個method的開始與結束，因此會需要透過ProceedingJoinPoint來「控制方法的動作」。
    - Around使用的ProceedJoinPoint有點像是上帝視角？除了取得參數、名稱之外，還能夠使用 `.proceed()` 來啟動method。
    ![螢幕擷取畫面 2026-06-02 135425](https://hackmd.io/_uploads/Hylidul-Me.png)
    - 由於呼叫方是切面class，接收最後的Response也會丟給proceed方法，因此需要用Object接住後，再做回傳。
- 順序問題 ⭐⭐⭐
    - 假設某個方法會被攔截做JWT驗證，又需要Log做紀錄，那誰會先攔截到呢?是Before還是Around？
    - 這時候不確定攔截的順序就會造成很多問題，因此會用 `@Order(級別)` 來確立順序。
    - 簡單來說，加上Order後，LogAspect跟AuthAspect就會按照順序來進行攔截。
    ![螢幕擷取畫面 2026-06-02 140044](https://hackmd.io/_uploads/rkl3_OxZMg.png)
    ![螢幕擷取畫面 2026-06-02 140030](https://hackmd.io/_uploads/Bkm2dOlbzx.png)
    - 這邊將驗證優先，再來才做log，可以避免掉一些無效訪問導致Log資料量過大的問題。

## Day154
#### 學習重點 : 完整驗證切面、權限碼獨立
- Controller層的驗證 ⭐⭐⭐
    - 今天把Controller的method都審視過，並根據是否有HttpsRequest決定加不加 `@UnAuth` 的驗證Bypass。
    - 另外，由於Service中部分程式的Permission偵測沒有加上，我今天也順便把權限驗證的部分完成了！
- PermissionCode ⭐⭐⭐
    - 由於Permission的Code如果直接寫在Service層做偵測，可能有Hard coding的問題，因此我獨立了一個enum出來專門處理PermissionCode。
    ![螢幕擷取畫面 2026-06-03 213728](https://hackmd.io/_uploads/SJSgFugWfg.png)

## Day155
#### 學習重點 : 權限切面建立與註解應用、專案Part-2收尾
- PermissionAspect ⭐⭐⭐⭐⭐⭐⭐
    - 由於很多API是串接給管理員的功能，若要在Service「重複寫驗證權限邏輯」（找userRole跟Permission是否有對應欄位），十分的麻煩。
    - 因此我建立了權限切面來統一管理驗證邏輯。
    ![image](https://hackmd.io/_uploads/SkSvgzJZze.png)
    - 也給定一個註解專門處理PermissionCode
    ![image](https://hackmd.io/_uploads/SkFiDf1ZMx.png)
    #### joinPoint.getSignature()
    - 前幾天研究切面的時候好像沒有紀錄到關於「joinPoint.getSignature的用法」，總之這個方法可以 **取得對應method的metadata**。
    - 透過今天的切面實作再來簡單理解一次 : 
    ![image](https://hackmd.io/_uploads/B1X6GGy-zl.png)
    - 由於我們需要取得method頭上的註解參數，因此透過joinPoint拿到metadata後利用反射機制取出method數據，再把參數PermissionCode取出來。
    - 接著拿著取出來的permissionCode與ThreadLocal中的request.account做核對！
- 實際註解 : ⭐⭐⭐⭐⭐⭐⭐ 
    - 這邊紀錄一個地方，若做了切面驗證，無論是Jwt還是Permission，原本的HttpServletRequest參數就可以拔掉，因為request本來就存在（ThreadLocal），若不需要在Controller使用，就可以拔掉。
    ![image](https://hackmd.io/_uploads/Sk-gdz1ZMg.png)
- 其餘雜項完善 ⭐⭐
    - 購物車刪除物品Response回傳有問題，我稍微調整了一下。
    - BCrypt的驗證在login當中進行了三次驗證，可能會搞爆CPU ww，所以我稍微修正了下，讓驗證只剩兩次（主密碼or臨時密碼）。
    - JWT secretKey在@Value中我直接把明文寫出來了ww，趕快修正回來哩！

## Day156
#### 學習重點 : 使用者資料管控-電商系統實作（Part-2結業式🥳）
- 功能綜整與總結 ⭐⭐⭐⭐⭐⭐⭐⭐
    - 總結來說，這次的專案part規模比上次大太多了ouo，所以沒辦法一一列出詳細部分，因此我特地錄了影片+請AI簡短總結我實作的功能！
    - 這裡有專案展示影片 : [點我到Youtube查看！](https://youtu.be/CXSSZ0BkeKE)
    ### 📁 商品分類樹 (Category Management)
    支援樹狀分層結構的商品分類，可動態擴展。
    - 實作與概念 : [Day127](https://hackmd.io/@learning-official/Java_learning_2#Day127)
    *   **新增分類**：`POST /category/add` (需權限 `ADD_CATEGORY`)
        *   **用法**：提交 `{ name, parent_id }`，若無上層分類則 `parent_id` 填寫 `"root"`。
    *   **查詢子分類清單**：`GET /category/parent/{parent_id}`
        *   **用法**：拉取指定父節點下的所有子分類列表（支援 UUID 或 `"root"`）。
    *   **按 ID 查詢單一分類**：`GET /category/{category_id}`
        *   **用法**：獲取特定分類詳細資訊。
    ### 📦 商品發佈與瀏覽 (Merchandise Management)
    提供商品的上架管理、分頁查詢與移除功能。
    - 實作與概念 : [Day117](https://hackmd.io/@learning-official/Java_learning_2#Day117)
    *   **商品發佈上架**：`POST /list` (需認證)
        *   **用法**：提交 `{ name, price, stock, category_id, description }`。
    *   **按分類瀏覽商品**：`GET /prod/{categoryId}?page={num}`
        *   **用法**：獲取特定分類下的商品分頁列表。
    *   **刪除/下架商品**：`DELETE /prod/{merchandiseId}` (需認證)
        *   **用法**：由商品持有人執行商品下架。
    ### 🛒 購物車緩衝區 (Cart Management)
    快取使用者欲採購的商品明細。
    - 實作與概念 : [Day118](https://hackmd.io/@learning-official/Java_learning_2#Day118)、[Day141](https://hackmd.io/@learning-official/Java_learning_2#Day141)
    *   **查看購物車明細**：`GET /cart/` (需認證)
        *   **用法**：獲取當前登入者購物車內的商品清單、單價、數量與總計金額。
    *   **加入商品至購物車**：`POST /cart/add` (需認證)
        *   **用法**：提交 `{ merchandiseId, quantity }`。
    ### 📋 訂單與交易處理 (Order Management)
    將購物車商品轉為訂單，並記錄歷史交易。
    - 實作與概念 : [Day139](https://hackmd.io/@learning-official/Java_learning_2#Day139)
    *   **建立選取商品訂單**：`POST /order/` (需認證)
        *   **用法**：提交包含已選取購物車明細 ID 的陣列：`{ cartItemList: ["id1", "id2", ...] }`。
    *   **獲取個人訂單列表**：`GET /order/list` (需認證)
        *   **用法**：載入當前使用者名下的所有交易紀錄，包含訂單狀態、商品內容與金額。
    ### 🔌 綠界支付 Webhook (EcPay Webhook Callback)
    - 實作與概念 : [Day144](https://hackmd.io/@learning-official/Java_learning_2#Day144)
    *   **綠界付款成功回呼**：`POST /webhook/api/ecpay/payment` (Form URL-Encoded)
        *   **用法**：由第三方金流（綠界）傳送付款通知以更新訂單狀態為 `PAID`。

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

## Day248
#### 學習重點 : 關於通知 - 設計已讀
- 已讀的請求 ⭐⭐⭐⭐⭐⭐⭐
    - 由於「已讀」屬於針對Notification的「is_read」成員狀態做更新，因此不該使用 `PUT`，而應該使用 `PATCH`。
    - 讓我來釐清這兩個動作的差異 : 
        - `PUT` : RequestBody傳入整個DTO資料，少任一欄位，則該欄位變null，因此屬於「**更新整體資料，後端覆蓋掉原本資料**」。
        - `PATCH` : 針對資源的單一成員狀態、資料做修改，RequestBody只針對單一欄位傳入，因此屬於「更新部分資料，**後端取出原有資料並對成員做更新**」。
    - 回到已讀的狀態更新，由於前端對已讀API的動作設計為PATCH，因此我們只需要針對某一id資源設計一個URL : `/notification/{id}/read`。
    - 並利用 `hasPermission檢查資源訪問可行性`，最後後端取出資料做部分更新即可！
- 程式邏輯與成果展示 ⭐
    - 以下是Service邏輯（ResponseCode暫時先不寫ww，因為想不到） : 
    ![image](https://hackmd.io/_uploads/BJO48hYuzl.png)
    - 以下是成果展示 : 
    ![image](https://hackmd.io/_uploads/B1XwLnYOGg.png)
    - 可以點擊「標記已讀」來執行PATCH動作。

## Day249
#### 學習重點 : 關於分類 - 設計總目錄.1
- 總分類目錄的結構 ⭐⭐⭐⭐⭐
    - 在實作這個端點前，要先知道分類結構應該怎麼看。
    - 每個分類各有 : `id`、`name`、`parent`、`children`
        - 這邊利用分類Response存取 `String id, String name, String parent_id, List<CategoryResponse> children`。
    - 當我們想要搜尋整體分類結構要分成底下的步驟 : 
        - 1️⃣ JPA搜尋所有Category ➝ 取得 `allCategories`。
        - 2️⃣ 遍歷allCategories，組裝成 `Map<String, CategoryResponse>`，String是分類id。
        - 3️⃣ 建立空列表 `List<CategoryResponse> rootList`，存所有根分類。
        - 4️⃣ 遍歷allCategories，利用當前分類id丟進Map取得Rsponse。
            - 若分類parent是null，則存進rootList。
            - 若分類有parent，則找到parent並 `parentResponse.getChildren.add(分類)`。
        - 5️⃣ 最後回傳 rootList。
    - 實際結構如下 : 
    ```json=
    {
        "rc" : "...",
        "rm" : "...",
        "data" : [
            {
                "id" : "...",
                "name" : "...",
                "parent_id" : null,
                "children" : [
                    {...},{...}
                ]
            },
            {...}
        ]
    }
    ```
    - 實際實作留給明天吧！

## Day250
#### 學習重點 : 關於分類 - 設計總目錄.1
- 實作總目錄搜尋邏輯 ⭐⭐⭐⭐⭐⭐
    - 先來看看CategoryResponse的設計 : 
    ```java=
    public class CategoryResponse {

        private String id;
        private String name;
        private String parent_id;

        // 當children為empty或者null時，則序列化時排除它不顯示
        @JsonInclude(JsonInclude.Include.NON_EMPTY)
        private List<CategoryResponse> children;

        public CategoryResponse(Category c){
            this.id = c.getId();
            this.name = c.getName();
            this.parent_id = c.getParent() == null ? null : c.getParent().getId();
            this.children = new ArrayList<>();
        }
    }
    ```
    - 底下根據昨天設計的目錄邏輯設計 : 
    ```java=
    private Response<List<CategoryResponse>> getCategoryTree(){
        List<Category> categoryList = categoryDao.findAll();
        Map<String, CategoryResponse> categoryResponseMap = new HashMap<>();
        // 根據分類填入id對照其Response
        categoryList.forEach(c -> categoryResponseMap.put(c.getId(), new CategoryResponse(c)));

        // 最後回傳的是rootList，存取根目錄的Response
        // 由於每個Response都帶有children，因此可以藉由children展開後續結構
        List<CategoryResponse> rootList = new ArrayList<>();
        for (Category c : categoryList){

            // 透過Map取得當前Response
            CategoryResponse currentDTO = categoryResponseMap.get(c.getId());
            String parentId = currentDTO.getParent_id();

            // 若當前DTO為根分類，則放入rootList
            if (parentId == null){
                rootList.add(currentDTO);
            } else {
                // 若不為根分類，則將其放入其父母的childrenList中
                CategoryResponse parentDTO = categoryResponseMap.get(parentId);
                if (parentDTO != null) {
                    parentDTO.getChildren().add(currentDTO);
                } else {
                    // 避免有孤立分類出現(分類有parentId，但沒出現在Map中)，因此直接加到根分類
                    rootList.add(currentDTO);
                }
            }
        }

        return new Response<>("0", "Successfully", rootList);
    }
    ```
- 結合API呈現 ⭐⭐
    - 由於gerTree這種屬於「**非驗證型端點**」，因此先在SecurityConfig放行（permit）該端點。
    - 再來結合前端可以呈現如下圖 : 
    ![](https://hackmd.io/_uploads/HkDMu6odze.png)

## Day251
#### 學習重點 : 關於使用者 - 設計個人頁面讀取/更新寫入.1
- Patch與DTO的問題 ⭐⭐⭐
    - 我們知道Patch是應用於「**部分欄位**」的覆蓋動作，不像PUT是直接覆蓋「整個」Entity後寫入。
    - 而使用者資料更新時，可以更改像是 : `nickname`、`phone_number`、`email`...，這些都會變成一個DTO類別 : `UserUpdateRequest`。
    - 但當我傳入一個DTO類別又使用**PATCH**動作時，DTO中的欄位就可能有null出現。
    - 因此我在設計DTO搭配PATCH的流程時，必須要先找出哪些欄位是null並過濾，到時候saveDB時才能避免寫入null！ 
- BeanWrapper、PropertyDescriptor ⭐⭐⭐⭐⭐⭐
    - 在寫個人資料更新時，由於PATCH與DTO的搭配，使得我需要先找出null欄位並過濾，此時就需要利用到小節標題的兩個類別 : `BeanWrapper、PropertyDescriptor`。
    - BeanWrapper : 作為Spring底層設計的**Bean包裝類別**。
        - 內部會包含一組「**Bean內部的屬性資料**」，也就是metadata！
        - 而BeanWrapper同時也可以**根據屬性名稱**去取得該屬性的type、value等。
    - PropertyDescriptor : 剛剛說的 **屬性資料**，就是存在PropertyDescriptor中。
        - 因此Bean內部的每個屬性都會變成PropertyDescriptor物件，其中包含屬性的name、type，以及屬性在Bean中的getter、setter方法。
    - 透過上述兩者的介紹，可以知道其實BeanWrapper以及PropertyDescriptor就是幫我 **做完反射的工作**，我們就不需要再去自己寫Field、Method來取得屬性資料了。
    - 接著我們就只需要抓到Bean的屬性陣列後，找出每個屬性的value，確認是否為null，並做標記，就可以達到在saveDB之前過濾null的動作啦~

## Day252
#### 學習重點 : 關於使用者 - 設計個人頁面讀取/更新寫入.2
- BeanUtils的複製方式 ⭐⭐⭐⭐⭐
    - 一般的BeanUtils.copyProperties，接收source objetc、target object，複製前者的屬性並對應到後者。而第三個參數（忽略欄位陣列）可以選擇是否填入。
    - 我們就可以自行設計一個String陣列，其中放入我們要「忽略的欄位」，也就是選擇不複製哪些欄位到目標物件中。
    - 這時就會用到昨天所構思的「**找出哪些為null的欄位**」，接著把null欄位名稱存成一個Array，並丟進BeanUtils.copProperties的第三個參數當中。
- 設計 `getNullPropertyValues`
    - 首先當然是先將srouce物件存成BeanWrapper物件，接著遍歷其中的 `PropertyDescriptor[]`，檢查每個屬性資料的values是否為null，是的話則存進最終String array當中。
    ```java=
    private String[] getNullPropertyValues(Object source){
        BeanWrapper beanWrapper = new BeanWrapperImpl(source);
        PropertyDescriptor[] pds = beanWrapper.getPropertyDescriptors();

        Set<String> nullSet = new HashSet<>();
        for (PropertyDescriptor pd : pds){
            Object src = beanWrapper.getPropertyValue(pd.getName());
            if (src==null || (src instanceof String str && str.trim().isEmpty())) nullSet.add(pd.getName());
        }

        return nullSet.toArray(new String[0]);
    }
    ```
- 搭配Service、Controller、UI作呈現 ⭐⭐⭐⭐
    - 接著Service設計一個處理取得個人頁面的method : 
    ```java=
    @Transactional(rollbackFor = Exception.class)
    public Response<UserResponse> updateUserProfile(User user, UserUpdateProfileRequest userUpdateProfileRequest){
        // 由於Authentication驗證的user不在當前事務當中，因此還是得打一次DB去確認為最新user
        User userDB = userDao.findById(user.getId()).orElseThrow(() -> ResourcesException.of(ErrorCode.USER_NOT_FOUND));
        BeanUtils.copyProperties(userUpdateProfileRequest, userDB, getNullPropertyValues(userUpdateProfileRequest));
        User saved = userDao.save(userDB);
        return new Response<>("0", "Successfully updated user's profile", new UserResponse(saved));
    }
    ```
    - 前端可呈現為下方模樣 : 
    ![image](https://hackmd.io/_uploads/BkZA7HCOGx.png)

## Day253
#### 學習重點 : 關於使用者 - 管理員/賣家後台設計構思
- 後台設計 ⭐⭐⭐
    - 這兩者都屬於數據上的統計回傳，因此需要設計DTO來應對，且驗證身分與資源確保該身分有權限訪問，避免IDOR情況發生。
    - 但電商專案有一個問題，當賣家/管理員刪除商品時，不能直接 `DELETE`。應該是將商品狀態改為 `status = DELETED`，藉此確保與該商品關聯的訂單(紀錄)不會變成孤兒資料。
    - 而管理員操控面板時，也應該確保雙重驗證，尤其是關於「刪除」需要特別加上保險。
    - 最後則是Log紀錄的部分，我個人的想法是可以透過切面針對面板操作的事件紀錄 : 「**時間、身分、動作、資源ID、狀態**」

## Day254
#### 學習重點 : 關於使用者 - 管理員/賣家後台DTO設計
- 前言 ⭐
    - 最近比較忙owo，進度會慢一些QAQ。
- DTO設計 ⭐⭐⭐
    - 我目前的想法是先將管理員&賣家的DTO合在一起，設計成官方直營的概念，後面在拆開成兩種DTO。
    - 我設計幾個欄位 : `revenue`、`orderCount`、`userCount`、`productCount`，可以再分為 `當日`、`全部`，但目前只需要提取全部即可，因為有些實體沒有設計時間的欄位ww。
    - 所以設計下來可以長這樣 : 
    ![image](https://hackmd.io/_uploads/Skrl1sZKGe.png)
    - 而在JPA查詢時，我們就得善用Query，來客製化SQL查詢語句！就交由明天來研究吧！

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
