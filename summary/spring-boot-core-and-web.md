# Spring Boot 核心與 Servlet / Tomcat

## Day33
#### 學習重點 : Spring Boot - 環境建置
- Maven簡介 ⭐⭐⭐⭐⭐
    - **Maven是？** : 它是一種自動化專案管理工具，讓各式各樣的檔案之間能順暢溝通。
    - 它靠著 `pom.xml` 這個核心檔案來描述版本、依賴關係、封包路徑等...。
    - 指令設置 : 利用maven的特性，用指令來輕鬆建置專案，常見的有 :
    ```maven=
    mvn compile
    mvn -cp 目標資料夾 目標類別檔
    ```
- Maven環境設置 ⭐⭐⭐⭐⭐⭐⭐
    - `mvn` 指令生效
        - 像終端機上的指令 `javac`、`pip`、`git` 一樣，我們要讓系統認得mvn這個指令，就要到電腦的 **編輯系統環境變數** 加入maven的bin檔(指令核心)位置，這樣在terminal輸入mvn時，系統就會 **自動導航** 到指定的位置去執行指令。
    - `pom.xml` 核心設置
        - 基本上maven就是靠著pom中的設定去自動化建置、生成檔案
        ```html=
        <!-- 首先先宣告這個檔案是用Maven的namespace下去跑的 -->
        <project xmlns="https://maven.apache.org/POM/4.0.0">
            <!-- pom.xml的格式版本 -->
            <modelVersion>4.0.0</modelVersion>
            <!-- 封包路徑 -->
            <groupId>test</groupId>
            <artifactId>web</artifactId>
            <!-- 專案版本，自己寫w -->
            <version>1.0.0</version>
            <properties>
                <!-- 編譯時使用的編碼 : 常見UTF-8 -->
                <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
                <!-- 編譯時使用的java編譯器版本(javac -version查看) -->
                <maven.compiler.source>21</maven.compiler.source>
                <!-- 程式最低可以執行的環境版本(java -version查看) -->
                <maven.compiler.target>21</maven.compiler.target>
            </properties>
        </project>
        ```
- 專案架構 ⭐⭐⭐⭐⭐⭐⭐⭐
    - 引用 [iT邦幫忙 Day20 Maven簡介](https://ithelp.ithome.com.tw/articles/10303335)
    ```
    專案 :
    |-- pom.xml (Maven 設定檔)
    |-- src
    |   |-- main (開發主目錄)
    |   |   |-- java (原始碼)
    |   |   |   -- test (groupId)
    |   |   |       -- web (artifactId)
    |   |   |               -- Main.java (程式檔)
    |   |   |-- resources (各種靜態資源/設定檔)
    |   |-- test (測試主目錄)
    |       -- java
    |           -- test
    |                -- web
    |                    -- MainTest.java
    |-- target (打包主目錄)
    |   -- classes
    |       -- Main.class
    ```
    - `src / main / java` 基本上是Maven的固定架構，不能隨便更改。
- 程式封包、指令呼叫 ⭐⭐⭐⭐
    - 開頭必須給予封包路徑
    ```java=
    package test.web;    
    ```
    - 當使用 `mvn compile` 編譯檔案時，程式會被編譯成.class並丟到target的classes資料夾中。
    - 當我們要執行class時，需要以 `mvn -cp ./target/classes` 為起點 -> 去找 `test.web.Main`，這樣分開寫的原因是要 **明確表示封包路徑完整呈現出** `test.web` 而不是在設定起點時就放在classes後面。
    - 最後指令長這樣 : 
    ```maven=
    mvn -cp ./target/classes test.web.Main
    ```

## Day34
#### 學習重點 : Spring Boot - 基本程式框架
- `pom.xml` 連結Spring Boot  ⭐⭐⭐⭐⭐⭐⭐
    - 昨天我只有針對pom.xml做基本專案設置，並沒有引入Spring Boot的架構，今天就來完善它吧！
    - 在 `pom.xml` 中，要加入幾個東西 : `parent`、`dependency`、`build`
    - `parent` 會取得 Spring Boot 的pom檔，內部包含複雜的依賴管理跟插件配置。
    `dependency` 選擇依賴 Spring Boot 的網頁工具。
    `build` 就是幫忙打包的部分啦~一樣是使用 Spring Boot 的工具。
    ```html=
    <!-- 繼承Spring Boot的pom檔 -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-stater-parent</artifactId>
    </parent>
    <!-- 依賴的web套件都在這引入 -->
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
    <!-- 以maven打包成Fat jar(整合所有使用到的套件，包含Spring Boot)-->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
    ```
- 基本Spring Boot網頁程式框架 ⭐⭐⭐⭐⭐⭐
    - 這邊引入了包含Spring的web伺服器、類別讀取、環境設定等
    ```java=
    // 環境設定、類別讀取、依賴性檢查等
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;

    // 讀取網頁類別並寫入到網頁
    import org.springframework.web.bind.annotation.RestController;
    import org.springframework.web.bind.annotation.GetMapping
    
    @SpringBootApplication
    @RestController
    public class Main{
        public static void main(String[] args){
            SpringApplication.run(Main.class, args);
        }

        @GetMapping("/") // 寫入根目錄，也就是首頁
        public String home_page(){
            return "Hello Spring boot!";
        }
    }
    ```
- 網頁呈現 ⭐⭐⭐
    - 首先是利用 `mvn compile` 編譯後，再利用我們在pom寫的dependency (Spring Boot plugin) `mvn spring-boot:run`        
    - 伺服器上線至本機後，使用`localhost:8080` or `127.0.0.1:8080` 找到自己publish上去的網頁。

## Day35
#### 學習重點 : Spring Boot - 靜態檔處理、路由與路徑
- 靜態檔是甚麼？ ⭐⭐⭐⭐⭐
    - 當不同人對網頁做出請求時，伺服器所做的動作 **都相同，都會導引至相同檔案**，且這些請求 **不須經過Java運算**，而是靠著當初啟動時所設定的方式就稱為靜態。
    - 在網頁設計當中，靜態檔就包含了美術、介面設計、按鈕效果等，因此常見的 
    **靜態HTML、CSS、JavaScript** 都是靜態檔的一員！
- 如何處理以及取得靜態檔？ ⭐⭐⭐⭐⭐⭐
    - 首先是靜態檔的位置 : 按照我在Day33 ~~從IT邦幫忙偷來w~~ 的結構圖中可以看到靜態檔被放在resources目錄當中，但我們還得新增一個 **static子目錄**，並把檔案塞進static當中，才能夠被讀取到！
    - 在啟動網頁後，我們需要在 `localhost:8080/` 後面加上靜態檔位置，感謝Spring Boot的幫忙，因此我們 **只需要打上static後的檔案位置** 即可！
    - 結果圖(~~其實我只是想放小八~~) : 
    ![DAY35_xiaoba](https://hackmd.io/_uploads/HJVJYGbGfe.png)
- 路由是甚麼？ ⭐⭐⭐⭐
    - 簡單來說，像是A網址到B網址的決定、跳轉，以及要怎麼回應B網址的內容都是靠路由的運作。
    - 在Spring Boot當中，我們可以 **引入路徑參數**，並寫一個 **路由函式** 來 **回應** 該網址的內容。
    ```java=
    import //...省略

    // 重點在這
    import org.springframework.web.bind.annotation.PathVariable;
    
    @RestController
    @SpringBootApplication
    public class Main{
        public static void main(String[] args){
            SpringApplication.run(Main.class, args)
        }
        
        // 填入路徑參數name建立路由函式，並以@PathVariable註解，達成動態
        @GetMapping("/test/{name}")
        public String set(@PathVariable String name){
            return "Hello " + name;
        }
    }
    ```

## Day36
#### 學習重點 : Spring Boot - RequestParam參數查詢、JSON格式
- 網址符號簡介 ⭐⭐⭐⭐⭐⭐
    - 這個東西算是回答了我一直以來困惑很久的問題 : 網址到底是怎麼寫的？尤其是一些甚麼 `?`、`=`、`&` 等等的符號，而在今天，我終於看懂了！
    - 在參數查詢當中，我們會用到 `路徑 ? 參數1=value & 參數2=value`，以 `?` 分隔路徑與參數，使Mapping透過路徑對應到函式後，將參數放進函式中。
    ```java=
    import //...省略
        
    // 引入參數查詢annotation
    import org.springframework.web.bind.annotation.RequestParam;

    @SpringBootApplication
    @RestController
    public class Main{
        public static void main(String[] args){
            SpringApplication.run(Main.class, args);
        }

        @GetMapping("/echo")
        public String echo(
            // 對echo路徑新增兩個參數，n1以及n2
            @RequestParam int n1, @RequestParam int n2
        ){
            return "Sum is " + (n1 + n2); 
        }
    }
    ```
    - 輸入 `http://localhost:8080/echo?n1=1&n2=2` ->
    ![DAY36_route_parameter](https://hackmd.io/_uploads/BkiWFzWMzg.png)
- 不同格式回傳 ⭐⭐⭐⭐⭐⭐
    - Spring Boot可以接受許多形式的回傳型態，如Array、Map、List等，甚至是自己定義的類別(要有getter)，它都會以 **Json資料格式** 放至前端，以下是實際操作 : 
    ```java=
    import //...省略
        
    @RestController
    @SpringBootApplication
    public class Main{
        
        public static void main(String[] args){
            SpringApplication.run(Main.class, args);
        }
        
        @GetMapping("/test")
        public String[] nameList(){
            String[] nameL = new String[]{"Alice", "Bob", "Carol"};
            return nameL;
        }
        
        @GetMapping("/test/map")
        public Map<String, Integer> map(){
            Map<String, Integer> mp = Map.of("x", 1, "y", 4, "z", 3);
            return mp;
        }
        
        // 自訂義類別(有getter)
        @GetMapping("/test/point")
        public Point point(){
            return new Point(3,4);
        }
    }
    ```
    ```java=
    // Point類別
    package test.web;

    public class Point {
        private int x;
        private int y;
        public Point(int x, int y){
            this.x = x;
            this.y = y;
        }
        public int getX(){
            return this.x;
        }
        public int getY(){
            return this.y;
        }
    }
    ```

## Day37
#### 學習重點 : Spring Boot - Mapping差異、(IoC、DI、Bean) 三兄弟
- **GetMapping** 以及 **ReqeustMapping** 差異 ⭐⭐⭐⭐
    - 在上網查請求網址寫法時，常常會看到Get跟Request的出現，我就很好奇這兩位同學差在哪？
    - 簡單來說，**Get是Request的簡化版**，它只處理 **method上的請求業務**，而Request可以對Class、method作用，由於Request比較廣泛，因此參數需要放得比較多。
    #### RequestMapping用法
    - 當作用在Class上時，意即該類別下的method都會從參數網址為起點開始，直接看code比較清楚！　
    ```java=
    import //..省略
    import org.springframework.web.bind.annotation.RequestMapping;

    @SpringApplication
    @RestController
    // 使Main類別都是從/test出發
    @RequestMapping("/test")
    public class Main{
        public static void main(String[] args){
            SpringApplication.run(Main.class, args);
        }
        
        // 網址為 localhost:8080/test/
        @GetMapping("/")
        public String home(){
            return "Hello test!";
        }
    }
    ```
    - 當作用在method上時，除了要一樣要加入目錄之外，還要選擇該網址的 **動作類型**，像是 `GET`、`POST`、`PUT` 等...。
    但如果我寫 `@RequestMapping(value="/yeah", method=RequestMethod.GET)`
    那其實就跟GetMapping長一模一樣了。
    #### 總結
    整體來說，GetMapping要比RequestMapping來的**更精確**，但當我們有許多class要做**網址分類時**，利用RequestMapping來寫會更好，還有當我們需求**不只GET時**，也要用RequestMapping來選擇Method！
---
- IoC （Inversion of Control 控制反轉）⭐⭐⭐⭐⭐⭐⭐⭐
    - 先下結論，它就是把 **控制權交給中心統籌管理**、**提高維護性、降低依賴性**。
    #### 仔細思考
    - 假設B、C實作了O介面，且A類別中需要使用O介面的功能，我們會在A類別中建立B物件（假設使用B物件實作的功能），但當我想換成C時，必須砍掉所有A物件，這樣 **不好維護程式**，且 **A對B、C的依賴性or相關性過高**。
    - 此時我們會在A類別中單純宣告O介面的變數，**但不指派**，而是將B與C交給中心管理，當我們要使用B實作的功能時，只需要將O變數指向中心的B，而C也是如此。
- DI（Dependency Injection 依賴注入） ⭐⭐⭐⭐⭐⭐⭐⭐
    - 在討論IoC時其實就有用到DI的概念，當我們自A類別宣告介面變數，並在建構時，==指派== 一個實作介面的物件，就是 ==注入==，而依賴就是 **A類別依賴O介面的意思**。
    - 我覺得這篇文章講的很好，「[淺入淺出 Dependency Injection](https://medium.com/wenchin-rolls-around/%E6%B7%BA%E5%85%A5%E6%B7%BA%E5%87%BA-dependency-injection-ea672ba033ca)」想像插頭是O介面，插座是A類別，可以有很多插頭實作O介面，但可能一個是電腦插頭，另一個是電風扇插頭，功能不同，當建立A物件時，就等於插頭與插座接上了（注入）!
    - 讓我們用code來實際操作看看 :
    ```java=
    //-------------插頭介面檔-------------------
    public interface ElectricalPlug{ // 插頭介面
        void Connect();
    }
    //---------------插座檔--------------------
    public class Socket{ //使用插頭的「插座」類別
        private ElectricalPlug ep;
        // 在初始化時，"注入" 實作插頭介面的類別
        public Socket(ElectricalPlug ep){
            this.ep = ep;
        }
        public void SendPower(){
            this.ep.Connect();
        }
    }
    //----------------------------------------
    //實作插頭介面
    public class AEP implements ElectricalPlug{
        @Override
        void Connect(){
            System.out.println("這是A插頭");
        }
    }
    ```
- Bean （~~豆豆先生~~）⭐⭐⭐⭐⭐⭐⭐⭐
    - 其實Bean就是實作O介面的類別**物件**啦~這些Beans都會被存放於Spring中心，透過Spring來管理這些物件的生命週期。
    - 我們不需要自行new物件做管理，而是丟給Spring IoC中心管理(做為一個Bean存起來)，而在需要用的時候，利用DI概念就可以啦~

## Day38
#### 學習重點 : Spring Boot - Bean創建與呼叫 
- Bean的創建 ⭐⭐⭐⭐
    - 在昨天學習中，我們知道，Bean就是是 **實作介面的類別** 存在Spring IoC中。
    - 我們在建立Bean時會用 `@Component` 來註解類別，而注入時會利用 `@Autowired`
    ```java=
    // A插頭實作類別
    @Component // 利用Component註解該類別為Bean，因此會交由Spring管理
    public class AEP implements ElectricalPlug{
        @Override
        public void Connect(){
            System.out.println("這是A插頭");
        }
    }
    ```
    ```java=
    // 插座類別
    @RestController // 多功能註解(包含component)
    public class Socket{
        
        // 利用Autowired去Spring中心找有實作ep的類別(A插頭)
        // 並注入進去
        @Autowired 
        private ElectricalPlug ep;
        
        @GetMapping("/power")
        public String sendpower(){
            this.ep.Connect();
            return "Power on";
        }
    }
    ```
    - 當Main檔案執行時，在 `localhost:8080/power` 就會看到以下畫面 : 
    ![image](https://hackmd.io/_uploads/rJ9rKGbfMe.png)
    #### 使用Autowired要注意的事情 ⭐⭐⭐⭐
    - 我們必須確保某類別使用Autowired時，該類別**本身也得是一個Bean**.. 為甚麼呢？ 
    - 原因在於 : 當該類別也是Bean時，**Spring才能管理這個類別**，在建構該類別時，才能順便檢查類別有沒有需要做甚麼(像是Autowired等)，也就是說，我們必須讓Spring管理插頭以及 **插座**，這樣才能保證這個插座會接哪些插頭。
    #### RestController也可以宣告Bean ⭐⭐⭐
    - 其實RestController本身是繼承自Controller，而Controller **又繼承自Component**，因此RestController就是一個 **多功能註解**，不只能宣告Bean，還能讓Spring去解讀其他東西，因此在Socket中，就不單純只寫Component，而是寫RestController（因為還有Mapping要被解讀）。
- 多Beans的使用 （**@Qualifier**）⭐⭐⭐⭐⭐
    - 如果有多個類別實作了插頭介面，而我們又使用Autowired的話，就會出錯，因為當Spring去找Bean時就會遇到 **要選哪個Bean?** 的難題，這時候就是Qualifier上場的時候啦~
    - 假設我有三個實作 `AEP`、`BEP`、`Celectrical`
    ```java=
    @RestController
    public class Socket{
        
        @Autowired
        @Qualifier("BEP")
        private ElectricalPlug ep;
        // ...省略
    }   
    ```
    - 透過Qualifier就可以選擇要取得的實作啦~
    #### Bean的名稱🚨
    - 這邊要注意的是，當Bean的名稱是像BEP這種 **前兩個字母是大寫**，那麼在Qualifier中寫BEP沒問題，但如果像Celectrical的話，在Qualifier就要寫 `celectrical`，第一個大寫要轉成小寫。
        ##### 為甚麼這麼麻煩？
        - 其實這跟Java的傳統有關，一般來說類別首字母都是大寫，在建構物件變數時，首字母反而會寫成小寫。
        - 由於Bean本身是物件，所以就理所當然地把Qualifier首字母限制成小寫~(如果類別前兩個字母是大寫，那Spring就會認為它是縮寫詞，而不過度干預！)

## Day39
#### 學習重點 : Spring Boot - PostConstruct 與 Value取得設定檔
- Bean的初始化 ⭐⭐⭐⭐⭐⭐
    - 若類別在實作介面時，本身也有成員變數，我們就需要利用初始化來賦值，由於 `@Component` 的關係，管理權在Spring身上，因此我們要利用 `@PostConstruct` 來告訴Spring要初始化。
    - **初始化的重要性🚨** : 如果是一般的初始化，其實可以直接在宣告時賦值，但若涉及到複雜的初始化，像是 **檢查是否有注入成功**，這些不能賦值但又很重要，因此需要靠初始化來完成。
    ```java=
    @Component
    public class AEP implements ElectricalPlug{
        
        private int durability;
        
        @PostConstuct
        public void init(){
            durability = 100;
        }
        
        @Override
        public void Connect(){
            if (this.durability>0){
                this.durability--;
                System.out.println(
                    "已連接A插頭，目前耐久度 : " + this.durability
                );
            }else{
                System.out.println("A插頭已損壞");
            }
        }
    }
    ```
    - 這邊給予插頭一個耐久度，**使用100次就會壞掉**，那麼就需要對耐久度做初始化（雖然可以直接賦值啦ww 但想要展示一下PostConstruct怎麼用）
    #### PostConstruct的一些須知🚨🚨
    - 首先就是一定是標註在 `public void` 的函式上。
    - 標註的函式**不能**有參數。
    - 盡量一個類別只放一個PostConstruct即可。
    #### 初始化檢查是否有注入 - （PostConstruct真正派上用場之地）
    ```java=
    @RestController
    public class Socket{
        
        @Autowired
        @Qualifier("AEP")
        private ElectricalPlug ep;
        
        @PostConstruct
        public void init(){
            if (ep == null){
                System.out.println("沒有插頭耶~");
            }else{
                System.out.println("好耶，有插頭!");
            }
        }
    }
    ```
    #### Spring存Bean的順序（超重要！！）
    - 1️⃣ 執行建構子
    - 2️⃣ 注入變數
    - 3️⃣ 執行PostConstruct
    - 這也難怪可以在PostConstruct做一些檢查（因為都建構完了嘛～）
- Value 與 Spring Boot設定檔 ⭐⭐⭐⭐⭐⭐
    - 我們可以利用 `@Value` 來讀取 Spring Boot 的設定檔中設定的值，這應該也算一種初始化？
    #### application.properties設定檔
    - [Day33](##Day33) 有放結構，忘了再去看w，反正這個設定檔會是放在resources中，這個檔案都是**拿來存值**的，也難怪它被放在靜態資料夾。
    - 它的語法是Spring自己的格式，使用 `Key=Value` 來作為宣告（注意不能有空白like key = value），且不需要型別，以下是範例 :
    ```python=
    durability=100
    # 註解跟python一樣，使用「#」
    my.name=小八 # 可以用「.」當作「的」，將name作為my的子屬性
    ```
    - 當然Spring除了可以讀properties檔，也可以讀常見的yml檔，我相信你很熟悉它，就不講了~
    #### Value取值
    - 在Bean中就可以取出設定檔中設定的值並👉注入👈在變數上，注意型別要對的上，而使用的方法如下 : 
    ```java=
    @Component
    public class AEP implements ElectricalPlug{
        
        @Value("${durability}") // 名稱不一定要跟變數一樣
        private int durability;
        
        // ...省略
    }
    ```
    - 除了在Bean中使用，也可以在有 `@Configuration` 的類別中使用，不過目前還不知道它是甚麼東東，先記在心裡就好w
    - 而Value也可以設置預設值，像這樣 `"${durability:50}"`，用**冒號**代表賦值，當Sprint在設定檔中 **找不到durability**，就會使用預設值。

## Day40
#### 學習重點 : Spring Boot - AOP（Aspect-Oriented-Programming）　
- AOP - 切面導向程式設計? ⭐⭐⭐⭐⭐⭐⭐⭐
    - 從字面上看根本看不懂🫠，但我找到了一張圖不錯。
    取自 [[AOP系列] 簡單介紹AOP的概念](https://ithelp.ithome.com.tw/articles/10229664)
    ![image](https://hackmd.io/_uploads/HyY7jMDv-l.png)        ----> **(圖一)**
    ![image](https://hackmd.io/_uploads/H1DEozPwWe.png)    ----> **(圖二)**
    - 從圖一可以知道，若n個方法除了本身的功能，還要做同樣的邏輯(權限check、資料check、錯誤log)，等於要寫n次邏輯。
    - 但當我們將方法共同邏輯轉90度，與方法本身的功能垂直後，變成像是 **切開** 來一樣，**這時就是AOP的用處了** -> 輕鬆管理相同邏輯、減少運算時間。
- 引入Spring AOP ⭐⭐
    - 在 `pom.xml` 中加入aop的dependency : 
    ```html=
    <depenency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-stater-aop</artifactId>
    </depenency>
    ```
- Spring AOP怎麼運用？ ⭐⭐⭐⭐⭐⭐
    - 在Spring中，`@Aspect` 是宣告類別為切面的註解，而切面本身也必須是Bean，**若**其不在Spring IoC中，內部的檢查機制都不會被讀取，也自然不能達到管理方法的功能了。
    - 由於Aspect本身是Bean，因此其管理的方法所在的類別也必須是Bean。
    - 切面類別寫法 :
    ```java=
    import org.aspectj.lang.annotation.Aspect;
    import org.aspectj.lang.annotation.Before;

    @Aspect // 創建切面
    @Component // 使切面為Bean
    public class AspectTime{
        
        // 在切入點方法之前執行
        @Before("execution(* test.web.ElectricalPlug.Connect(..))")
        public void beforeConnect() {
            System.out.println("準備連接插頭...");
        }
    }
    ```
    - Pointcut（切入點）📌
        - 在Before參數中，我們要設置 **切入點**，讓Spring知道用作於哪個方法之前。
        - 而參數有幾個要素 `execution(修飾詞 回傳型態 封包名.類別名.方法名(參數))`。
        - 修飾詞**可省略**，`*` 代表**隨便**，`隨便` 可以作用在路徑中以及回傳型態，而方法參數用（..）來代表。
        - 若不想這麼麻煩，也可以自定義annotation，並以「帶有該註解的方法」來作為切入點。
    - 其餘用法還有 `@After` : 作用於方法之後，跟Before一樣作法。`@Around` : 作用於方法前後，稍複雜，就不多細探討了~

## Day41
#### 學習重點 : Spring Boot - Spring MVC（Model-View-Controller）
- 甚麼是Spring MVC？ ⭐⭐⭐⭐⭐⭐⭐⭐
    - 簡單來說，它就是連結前端(View)與後端(Model、Controller)的橋梁，其實在前幾天我就做過MVC了。
    - 當我們在網址欄（**前端**）輸入 `http://localhost:8080/power` 這樣的動作時，實際上就是透過MVC告訴插頭要接電（**後端**），由此可知，MVC就是 `Mapping` 、`Controller` 等來實踐的！
- Http架構 ⭐⭐⭐⭐
    - Http協議是來規範前後端溝通的格式，其必包含`Request`、`Response`，這樣才能成為**有效的溝通**
    #### Request內容
    - **Http method** : 包含GET、POST、PUT等，在 [Mapping比較](##Day37) 中我有寫過！
    - Url : 就是那串 `http://localhost:8080/power`。
    - Header : 基本上就是一些通用資訊（使用者瀏覽器資訊等），可以利用 `@RequestHeader` 來取得資訊。
    - Body : 用在POST、PUT等身上，以參數請求回傳給後端，通常會是一些隱私的資訊。
    Header跟Body可以比喻成包裹，**寄件人、收件人、地址**都是Header，**內容物**是Body。
    #### Response內容
    - Http status code : 這個很常見，像是404Error一樣，前面的數字就是狀態碼 : ![image](https://hackmd.io/_uploads/SJ8wKDOPZx.png)
    - Header : 與Request header一樣都是放通用資訊。
    - **Body** : 後端要傳給前端的數據會放在這，當請求通過時，數據會整合前端顯示給使用者。

## Day42
#### 學習重點 : Spring Boot - Url格式、POST
- Url的結構 ⭐⭐⭐⭐⭐⭐
    - 在Url中，分為幾個部分 -> `協議://域名:埠號/路徑?查詢#片段ID`。
    - **協議** : 一般常用的就是http協議
    - **域名** : 就是主機的IP位置(一般來說不會直接寫IP啦w)
    - **埠號** : 選填，它作為主機的起點與終點，可以分流不同資訊，不同協定會指定不同連接埠，這樣當電腦遇到埠號時，就知道要以什麼樣的方式處理它。
    想像**域名是一家公司**，當送貨員要送貨時，根據貨上的樓層(**埠號**)搭到該樓交給人員(**使用者**)。
    - ---
    - **路徑** : 這裡就是Spring的天下啦~ 根據使用者輸入的路徑，Spring就會去找相對應的Mapping，像是前幾天寫過的RequestMapping以及GetMapping！
    - **查詢** : 像是RequestParam這樣的查詢參數，用於動態導向。
    - **片段** : 這邊只專屬於前端，因此並不會與後段有聯繫，像是點擊我的筆記Day1的話，網址就是`https://hackmd.io/@learning-official/Java_learning#Day1`，後面的Day1**不會傳到後端**，而是單純讓使用者電腦**自動滑到Day1的頁面**而已。

- POST概念 ⭐⭐⭐⭐
    - 昨天統整的Http架構中，Body就是給POST包裝後傳遞到後端的東西，由於它是隱蔽的，不能放在Header由GET傳遞，因此POST在MVC當中是很重要的東西，一些私人的參數都要靠它來傳遞。
- **@RequestBody**、網站測試 ⭐⭐⭐⭐⭐⭐⭐
    - 透過前幾天寫的[Point類別](##Day36)，我們可以以X、Y值為隱密資訊透過POST方法傳遞，其中需要用到 `RequestBody` 來標註Point物件 : 
        ```java=
        // ...省略
        import org.springframework.web.bind.annotation.RequestHeader;

        @SpringBootApplication
        @RestController
        @RequestMapping("/test")
        public class Main{
            
            // ...省略
            @RequestMapping("/post")
            public String post(@RequestBody Point point){
                System.out.println(point.getX());
                System.out.println(point.getY());
                return "請求成功";
            }
        }
        ```
        ![image](https://hackmd.io/_uploads/BkbvBCFvZe.png)
        ![image](https://hackmd.io/_uploads/H1NuHAKv-x.png)

## Day43
#### 學習重點 : Spring Boot - RequestHeader、API概念
- RequestHeader用法 ⭐⭐⭐⭐
    - 昨天認識了RequestBody的用法，那麼今天來學Header吧！儘管Header似乎不會拿來傳重要參數，但也可以模擬一下，學習如何使用RequestHeader註解！
    - Header本身可以用在 **各種Http method**，不像Body只能用在POST & PUT。
    ```java=
    import org.springframework.web.bind.annotation.RequestHeader;

    @SpringBootApplication
    @RestController
    public class Main{
        
        public static void main(String[] args){
            SpringApplication.run(Main.class, args);
        }
        
        @RequestMapping("/header")
        public String header_test(@RequestHeader String info){
            System.out.println(info);
            return "請求成功";
        }
    }
    ```
- API概念 ⭐⭐⭐⭐⭐⭐
    - 老實說，這個東西從我以前開始就是一個似懂非懂的東西，想說藉著Java學習，來了解API真正的意義！
    #### API （Application Programming Interface）
    - 字面上來看，叫做**應用程式介面**，直白的講，就是應用程式之間的溝通！
    - 想像API是服務生，而我們看著menu(應用程式)勾選完料理後，由服務生(API)送到廚房(應用程式)製作美食！因此可以說API是串接使用者與後台數據處理的重要成員！
    #### API實際應用
    - 當要比較飯店價格時，就會找~~trivago~~，它其實就是把各大飯店的API接到自己的網站上，當我們查詢空房、價位時(**Request**)，它就會將我們的要求利用飯店API取得資訊(**Response**)。
    #### 總結
    - API就是一個**窗口**，連接了使用者與伺服端，使用者**不須**知道伺服器端怎麼處理的，只需要得到一個結果即可，很大程度的減少使用者的困惑程度ww

## Day44
#### 學習重點 : Spring Boot - RESTful API
- Representational State Transfer （具象狀態傳輸）⭐⭐⭐⭐
    - REST本身是一種架構，使傳輸過程更有效率，它規定了資料應該以什麼樣的格式傳送、儲存...。
    - 因此當我們要寫一個REST協定的傳輸、儲存功能時，就會說RESTful API(充滿REST風格的服務員？)
    - 而REST的細節可以利用**CRUD**(增查改刪)來表示，也就是**Create、Read、Update、Delete**。
- RESTful API實際作法 ⭐⭐⭐⭐⭐⭐
    - RESTful API會以各種Http method表示要對資料庫做甚麼事，而所謂的CRUD就會對應到**POST、GET、PUT、DELETE**。
    #### Url的路徑階層表示
    - 當我們選用Http method時，就代表要對資料庫做某某事，而路徑已`/`作為分隔，就可以讓後端知道前端要針對`哪個資料中 的 哪個資料中的...`，其中 `的` 就代表 `/`，底下有個明確的圖來表示[（感謝古古的筆記圖w）](https://kucw.io/blog/springboot/21/)。
    ![image](https://hackmd.io/_uploads/r18yqf-MGx.png)
    #### Response Body用JSON格式
    - 在RESTful API的製作中，會以JSON作為主要格式，因此在RESTful API中會使用 `@RestController`。
    #### RESTful API選用時機
    - 若是**單純的操作資料庫**、**跨平台服務時**，以RESTful API絕對是很好的選擇。
        - 尤其在跨平台時，若為了Response，要分別寫網頁版、手機版的格式，但以RESTful API回傳都是JSON格式的情況下，就可以避免這種麻煩事！
    - 若涉及到**複雜的運算** 或是 **隱私資料**，以RESTful API這種顯示在網址上的作法似乎就不是很好。

## Day45
#### 學習重點 : Spring Boot - Mapping的種類、實作RESTful API
- Mapping種類 ⭐⭐⭐⭐
    - 除了 `@GetMapping`，還有 `@PostMapping`、`@PutMapping`、`@DeleteMapping`，這些讓我們可以對同一資源，同一網址，做不同的操作！
    - 以往都是利用 `RequestMapping("路徑", method = RequestMethod.POST)` 醬子去選擇，十分麻煩，因此後來多了四個直接寫XXXMapping的方法，很省事！
- RESTful API實作 ⭐⭐⭐⭐⭐⭐
    - 其實就是**把CRUD功能做出來**！因此會用到**四大Http method**，但因為我還沒學資料庫所以單純把之前學的annotation放上來而已，至於真的要更新到資料庫中...還要一段時間w
    - 這邊用我之前寫的Point類別當作資料操作。
    ```java=
    // ...省略
    public class PointTest{
        // ...省略
        @PostMapping("/points")
        public String create(@RequestBody Point point){
            this.point = point;
            return "點創建成功";
        }

        @GetMapping("/points/{point}")
        public String get(@PathVariable String point){

            if (this.point == null){
                return "點不存在";
            }else if (this.point.getName().equals(point)){
                return "點查詢成功 " + this.point.getX() + "," + this.point.getY();
            }else{
                return "點查詢失敗";
            }  
        }

        @PutMapping("/points/{point}")
        public String put(@PathVariable String point, @RequestBody Point p){
            if (this.point == null){
                return "點不存在";
            }else if (!this.point.getName().equals(point)){
                return "點更新失敗";
            }else{
                this.point = p;
                return "點更新成功";
            }
        }

        @DeleteMapping("/points/{point}")
        public String delete(@PathVariable String point){
            if (this.point == null){
                return "點不存在";
            }else if (this.point.getName().equals(point)){
                this.point = null;
                return "點刪除成功";
            }else{
                return "點刪除失敗";
            } 
        }
    }
    ```
    #### REST細節注意🚨🚨
    - 這邊要注意，雖然網址路徑相同，但「**動作**」不同，這是REST中很厲害的地方 -> 當我們操作Http method時，可以避免網址**濫用** -> 利用**單一網址**對上不同資源的操作。我覺得可以把這個主題放到明天來研究，今天先這樣！

## Day46
#### 學習重點 : Spring Boot - RESTful API 深入研究
- 名詞解釋 
    - 由於很多名詞一直出現，但又搞不清楚，所以我稍微統整了一下！
    #### Resources（資源）⭐⭐⭐⭐
    - 它就是被Http method操作的東西，可以是物件、狀態、紀錄等，只要跟前後端操作有關的，都可以說它是一種資源，其只會有一個**唯一Url指向**，操作可以不同（這就剛好符合REST風格的寫法）。
    #### Endpoint ⭐⭐⭐⭐⭐⭐
    - 在API當中，是一種**特定**的URL，其中可能有特定的參數、動作等...，它是處理前端與後端的接頭。
    - 它與一般URL不一樣的點在於 -> 一般URL可能長這樣 `/api/points`，但enpoint是 `[GET] /api/points`、`[POST] /api/points`，**這些都是不同的endpoint，但卻是同一個URL**。
    #### Richardson Maturity Model ⭐⭐⭐⭐⭐⭐⭐⭐
    - 該模型用來評估一個API是否真的具有REST風格，共分為四級。
    這邊引用 [Shiny的軟件開發的瑣碎閒聊(8)](https://www.threads.com/@shiny2024hk/post/DBJcneBzDHt?xmt=AQF0iJ7Ox1Pt_h19Kb1YUQe4HTeUjCyxjeivOSmaxgy1UQ) 所展示的四個REST風格等級。
    🟥等級０：單一 Endpoint 配以參數來識別操作。
    🟨等級１：將單一 Endpoint 拆分成多個資源導向的 Endpoint。
    🟦等級２：引入標準的 HTTP 動詞（GET、POST、PUT、DELETE）。
    🟩等級３：支持 HATEOAS，提供清晰的資源導航。
    - 至於甚麼是HATEOAS？其實就是提供資源操作之外，還會提供**多餘**的連結，讓使用者可以導向其他頁面！（至於內部細節，可以安排到後面來寫寫看！）

## Day47
#### 學習重點 : Spring Boot - Http Status Code
- 狀態碼介紹 ⭐
    - 甚麼是狀態碼呢？在前幾天有介紹狀態碼是**Response**的其中一個部分，也代表這次**請求的狀態**，這邊就來仔細看看每個狀態碼的涵義！
- 常見狀態碼 
    #### 1XX : 資訊回應 ⭐⭐
    - 1開頭不常看到，就不寫太多了。
    - **100** : `Continue`，讓使用者繼續請求。
    #### 2XX : 成功回應 ⭐⭐⭐⭐⭐⭐
    - 一般來說，2開頭的都是跟Pass✅有關，看到這個要開心啊w
    - **200** : `OK`，成功的概念。
    - **201** : `Created`，通常用於POST請求
    - **202** : `Accepted`，表示該請求已經通過，**但** 後續處理尚未完成，有可能是導向其他伺服器，或者本身伺服器有其他工作要處理。
    #### 3XX : 重新導向訊息 ⭐⭐⭐⭐⭐⭐
    - **300** : `Multiple Choices`，重新導向後，可能會有多個URL供使用者選擇，像是選擇語言之類的~
    - **301** : `Moved Permanently`，基本上就是URL**永久搬家**了！舊網址收到301，就會**重新導向**至Response header中Location的新URL。
    - **302** : `Found`，他是**臨時搬家**，與301一樣都會導向新網址！
    對於301、302除了搬家模式不同，流量也會不同，301因為是永久搬家，因此新網址會**繼承舊網址的SEO排名**，但302因為是暫時搬家，因此不會繼承。
    #### 4XX : 前端請求錯誤 ⭐⭐⭐⭐⭐⭐⭐⭐
    - **400** : `Bad Request`，前端的參數請求錯誤，像是下面的範例，若請求參數中填入**字串**，就會回傳400
        ```java=
        @GetMapping("/echo")
        public String ec(@RequestParam int n1, @RequestParam int n2){
            return "Sum is " + (n1 + n2); 
        }
        ```
        ![image](https://hackmd.io/_uploads/HJo-9MWffg.png)
    - **401** : `Unauthorized`，跟字面上的意思差不多，就是**未通過驗證**，像是帳號密碼打錯啦，token錯誤之類的～
    - **403** : `Forbidden`，權限不足的概念！像是我不是Youtube Premium會員就不能背景播放（嗚嗚嗚嗚嗚嗚嗚嗚），通常會把401跟403綁在一起看，因為兩個很像！
    - **404** : `Not Found`，這個應該算是最為人知的一個狀態碼了w，基本上就是前端網址輸入錯誤啦～
    #### 5XX : 後端處理問題 ⭐⭐⭐⭐⭐⭐⭐⭐
    - **500** : `Internal Server Error`，基本上只要後端程式錯誤有Bug，都會回傳500！
    - **503** : `Service Unavailable`，在伺服器維護期間去call API的話，會回傳503，表示現在伺服器無法處理請求，也有可能是因為流量過大，太過忙碌導致。
    - **504** : `Gateway Timeout`，如果後端處理該請求過久，就會被認定超時，有可能程式不小心進到無限Loop了w。

## Day48
#### 學習重點 : Spring Boot - MVC與API的關係釐清、Spring JDBC介紹
- MVC與API ⭐⭐⭐⭐⭐⭐⭐
    - 其實這兩個名詞一直在我腦海中不斷交錯在一起，因為我常常會搞混他們（看來是學不精的關係🤦‍♂️🤦‍♀️）
    - 我稍微統整了一下這兩個東西的差別 : 
    #### 模式不同
    - MVC主要是以拆分前後端為架構，將重點交由不同部門處理（關注點分離），且Model與資料庫溝通，View負責處理前端畫面並呈現數據，Controller則**充當Model、View的橋樑**，處理使用者與View的互動。
    - 而API本身是**一種介面協議**，而Controller只是一種實作API的方法，API不與View結合，只回傳純數據結構，如JSON、XML。
- 資料庫操作 ⭐⭐⭐⭐⭐⭐⭐
    - 在Spring Boot當中，後端與資料庫的互動需要藉由一些工具來操作，常見的有 `Spring JDBC、Hibernate、Spring Data JPA` 等等，而其中可分為 : **SQL操作DB**、**ORM概念操作DB**，JDBC屬於前者，後面兩個屬於後者。
    - ORM概念不會直接使用SQL語法，而是利用Java語法操作。
    - 現在先來看JDBC是甚麼，ORM之類再來研究！
    #### Spring JDBC介紹
    - JDBC（Java Database Connectivity），就是一種利用Java與Database連結的技術，由於單純用Java操作資料庫會很麻煩，因此Spring JDBC簡化了中間步驟，只需要專心寫SQL語法即可。

## Day49
#### 學習重點 : Spring Boot - Spring JDBC建置
- `pom.xml` ⭐⭐
    - 建置Spring JDBC要先引入Spring打包好的JDBC啦~ 另外還要加入SQL資料庫！
    ```html=
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>
    <dependency>
        <groupId>com.mysql</groupId>
        <artifactId>mysql-connector-j</artifactId>
        <version>8.0.33</version>
    </dependency>
    ```
- `application.properties` ⭐⭐⭐⭐
    - 接下來要與SQL資料庫做連線！ 
    ```=
    // 使用MySQL驅動程式
    spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
    
    // 連接到自己的電腦SQL資料庫，並指定資料庫的位置myjdbc，並把編碼與時區都調整為常用
    spring.datasource.url=jdbc:mysql://localhost:3306/myjdbc?serverTimezone=Asia/Taipei&characterEncoding=utf-8
    
    // 放入SQL的帳密
    spring.datasource.username=root
    spring.datasource.password=springboot
    ```
    - 基本上在application與pom中設定好，就可以開始操作SQL語法啦~不過因為我之前有學過一些些的SQL所以還蠻好理解的w

## Day50
#### 學習重點 : Spring Boot - SQL資料庫建置、實際程式操作
- SQL資料庫建置 ⭐⭐⭐⭐
    - 由於我幾年前有稍微碰過SQL！所以還算熟悉SQL語法操作，這邊就先利用Intellij的內建功能來連結SQL吧~
    - 由於這幾天都很忙，我先以教學的範例做為參考，做一個student的table出來 ->
    ```SQL=
    CREATE TABLE student (
        id INT PRIMARY KEY ,
        name VARCHAR(30)
    )
    ```
    - 基本上就是建置出一個ID與Name的pair，ID具**唯一性**(PRIMARY key)，Name每個字元數不超過30。
    - 建置完的Table如下
    ![image](https://hackmd.io/_uploads/rk2B1_7uZx.png)
- 實際程式操作（**C**reate）⭐⭐⭐⭐⭐⭐
    - 今天先單純利用Create（POST）方法來嘗試建立資料(Resources)。
    #### NamedParameterJdbcTemplate
    - 他是一種模板，用於與SQL資料庫溝通，而我們會在程式中**注入**它，當成Bean來使用。
    ```java=
    @Autowired
    private NamedParameterJdbcTemplate namedParameterJdbcTemplate;

    @PostMapping("/jdbc")
    public String insert(){
        
        String sql = "INSERT INTO student(id, name) VALUES (3, 'John')";

        Map<String, Object> map = new HashMap<>();
        namedParameterJdbcTemplate.update(sql, map);

        return "INSERT passed";
    }
    ```
    - 老實說，它其實就是把SQL語法塞進字串ww。
    #### map問題
    - 要注意！map的出現是因為後續有動態寫法，可以在SQL語法中寫入動態參數，並以map的key-value pair對應到 id-name的pair，但目前先以固定寫法，因此map是空的，只是裝飾品w。
    #### 今日心得
    - 今天一直在重複POST error羅生門，後面先暫時把程式移到Main區執行，這個問題之後得好好來解決一下QAQ。

## Day51
#### 學習重點 : Spring Boot - 資料庫動態參數使用
- SQL動態參數 ⭐⭐⭐⭐⭐⭐
    - 在SQL當中，使用 `:` 來宣告變數，這樣就可以跟昨天的Map搭配啦！
    ```java=
    String sql = "INSERT INTO student(id, name) 
        VALUES (:studentId, :studentName)";
    ```
    - 表示studentId存入student table中的id鍵，以此類推。
- Map搭配 ⭐⭐⭐⭐⭐⭐
    - 而Map本身就是存取key-value pair的最佳工具！我們利用HashMap建構後，將參數輸入的id & name以及getter放入key-value pair，實作如下 : 
    ```java=
    @PostMapping("/jdbc")
    public String insert(@RequestBody Student student){
        
        String sql = "INSERT INTO student(id, name) VALUES (:studentId, :studentName)";

        Map<String, Object> map = new HashMap<>();
        // 這邊放入的key會根據其指向的值，對上SQL的變數值，並放入table
        map.put("studentId", student.getId());
        map.put("studentName", student.getName());
        namedParameterJdbcTemplate.update(sql, map);

        return "INSERT passed";
    }
    ```
    - 這邊就利用前幾天學的POST（Create）的**RequestBody來放入參數**，而這邊利用Student類別的範例來做參考。
    ```java=
    public class Student {
        
        private Integer id;
        private String name;

        public Integer getId() {
            return id;
        }

        public void setId(Integer id) {
            this.id = id;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }
    }
    ```
    #### 疑問
    - 這兩天都在處理Update的東西，我發現Map本身存的就是SQL中的參數映射值，因此一次的Create只會有一組數據做上傳，但若是SQL語法有多組，是不是可以在Map中放入多組SQL參數映射呢？還是這樣就會放在不同的路徑當中？

## Day52
#### 學習重點 : Spring Boot - UPDATE、DELETE實作
- 實作 ⭐⭐⭐⭐
    - 其實Update與Delete的概念與前幾天的CRUD實作相同，只是SQL語法會稍做更動。
    - 這邊SQL語法的重點在於 `WHERE` 的條件限制，不然很容易就毀了整個Table。
    - 這邊利用了 `namedParameterJdbcTemplate.update()` 回傳 **更動資料筆數** 來判斷操作是否執行成功。
    #### UPDATE
    - 語法 : `UPDATE {TABLE} SET 欄位 = 值 WHERE 條件`
    ```java=
    @PutMapping("/jdbc")
    public String update(@RequestBody Student student){

        String sql = 
            "UPDATE student SET name = :name WHERE id = :id";

        Map<String, Object> map = new HashMap<>();
        map.put("id", student.getId());
        map.put("name", student.getName());
        
        int count = namedParameterJdbcTemplate.update(sql, map);

        if (count > 0) {
            return "已更新 " + count + " 筆資料";
        } else {
            return "找不到 ID 為 " + student.getId() + " 的學生";
        }
    }
    ```
    - 這邊把SQL動態參數、map的key變數名稱 **統一** 跟student table的 **欄位名稱相同**！這樣日後在debug可以更加迅速！
    #### DELETE
    - 語法 : `DELETE FROM {TABLE} WHERE 條件`
    ```java=
    @DeleteMapping("/jdbc/{id}")
    public String delete(@PathVariable Integer id){

        String sql = "DELETE FROM student WHERE id = :id";

        Map<String, Object> map = new HashMap<>();
        map.put("id", id);
        
        int count = namedParameterJdbcTemplate.update(sql, map);

        if (count > 0){
            return "DELETE Successful";
        }else{
            return "DELETE Failed";
        }
    }
    ```

## Day53
#### 學習重點 : Spring Boot - READ（GET）實作
- query ⭐⭐⭐
    - 在CRUD中，READ不屬於Update的範圍，因為它並沒有實際更動到資料，因此會使用**query來做查詢操作**，而其回傳的是 `List<T>` 型態，這邊的T自然宣告為Student型態！
- RowMapper ⭐⭐⭐⭐⭐
    - 我們知道 `String sql = "SELECT ..."` 是針對TABLE做查詢，而後續則會回傳**ResultSet這個原始數據**，此時靠著RowMapper即可將ResultSet數據處理成Student的型態！
    - 但由於RowMapper只是一種概念，我們要將其實作，底下是實作 : 
    ```java=
    public class StudentRowMapper implements RowMapper<Student>{

        @Override
        public Student mapRow(ResultSet rs, int rowNum) 
            throws SQLException{

            Student student = new Student();
            student.setId(rs.getInt("id"));
            student.setName(rs.getString("name"));
            return student;
        }
    }
    ```
- READ實作
    - 實作完Mapper的部分就可以來看主程式了！
    ```java=
    @GetMapping("/jdbc")
    public List<Student> query(){
        String sql = "SELECT id, name FROM student";
        Map<String, Object> map = new HashMap<>();
        StudentRowMapper rowMapper = new StudentRowMapper();

        List<Student> list = 
            namedParameterJdbcTemplate.query(sql, map, rowMapper);
        return list;
    }
    ```
    #### 流程
    - 1️⃣ SQL語法取得ResultSet數據。
    - 2️⃣ Mapper將Row的key-value pair轉成Student物件並回傳。
    - 3️⃣ 利用 `namedParameterJdbcTemplate.query()` 回傳List型態。
- 總結CRUD實作 ⭐⭐
    - 透過這幾天的歷程，我能夠感受到前後端的分工了，但是對於程式分區以及除錯得需要再來研究研究！

## Day54
#### 學習重點 : Spring Booot - Controller-Service-Dao
- MVC 與 Controller-Service-Dao ⭐⭐⭐⭐⭐⭐
    - MVC本身就是一種架構概念而已，並不會影響程式本身，只是將程式分類，以便日後較好維護。
    - 而在Spring Boot中，會利用Controller-Service-Dao來實踐MVC的概念。
    - 擷取自 [古古的後端筆記-Day28](https://kucw.io/blog/springboot/28/)，我覺得這張圖寫得很清楚！
    ![image](https://hackmd.io/_uploads/S11O9fWfzg.png)
    #### MVC 與 Controller-Service-Dao 的實際對應
    - View在Spring Boot實踐中並不需要處理，因為它是屬於**前端框架**的部分。
    - Controller 就是對應到 Controller層，像是RequetMapping、RequestParam、PathVariable等與前端溝通的都屬於Controller層。
    - 而Model所對應的就是**Service與Dao層**，但這兩層的分別十分重要。
- 三層架構實際實作 ⭐⭐⭐⭐⭐
    #### Controller層
    ```java=
    @SpringBootApplication
    @RestController
    public class StudentController{
        
        public static void main(String[] args){
            SpringApplication.run(StudentController.class, args);
        }
        
        @Autowired
        private StudentService studentService;
        
        // GetMapping屬前端溝通
        @GetMapping("/jdbc")
        public Student query(@PathVariable Integer id){
            return studentService.getById(id);
        }
    }
    ```
    #### Service層
    ```java=
    @Component
    public class StudentService {

        @Autowired
        private StudentDao studentDao;

        public Student getById(Integer id){
            List<Student> list = studentDao.getById(id);
            if (!list.isEmpty()){
                return list.getFirst();
            }else{
                return null;
            }
        }
    }
    ```
    #### Dao層
    ```java=
    @Component
    public class StudentDao{

        @Autowired
        private NamedParameterJdbcTemplate 
            namedParameterJdbcTemplate;

        public List<Student> getById(Integer id){
            // SQL語法
            String sql = 
                "SELECT id, name FROM student WHERE id = :id";

            Map<String, Object> map = new HashMap<>();
            map.put("id", id);

            return namedParameterJdbcTemplate.query(
                 sql, map, new StudentRowMapper()
             );
        }
    }
    ```
    - 可以注意到三層都必須是Bean，才可連貫注入。
    - Dao層專注在**SQL語法與資料庫溝通**、Service層專注在**商業邏輯處理**、Contorller專注在與前端溝通（**資料呈現、選擇View**）。

## Day55
#### 學習重點 : Spring Boot - ResponseEntity
- ResponseEntity介紹 ⭐⭐⭐⭐
    - 在Controller決定對前端顯示何訊息時，其中同時包含 : `狀態碼`、`Response Header`、`Response Body`，之前在寫的時候，我都只有 `return String`，因此狀態碼除非網址輸錯，不然都是顯示 ==200ok==。
    - 而ResponseEntity就可以解決這個問題，他的工作就是負責設定上述的三個東西。
- ResponseEntity實際作用 ⭐⭐⭐⭐⭐⭐
    - 我稍微整理了一些我遇到的錯誤狀態碼 : 
        - 404 : 輸入id後 **查詢不到** 資料 | 放入student資料update，但 **找不到** id更新。
        - 409 : student id **重複** 更新、創建。
        - 500 : SQL語法錯誤 | DB連線中斷 | nullPointerException。
    - 我將昨天簡易的分層架構變成 : 
        - 💡**Dao**層 : 回傳更動資料筆數。
        - 💡**Service**層 : "暫時" 單純return Dao層回傳的筆數。
        - 💡**Controller**層 : 判斷筆數並使用ResponseEntity做更精確的前端呈現。
    - 一般來說，500會被放在try-catch中的Exception，因此架構如下 : 
    ```java=
    @GetMapping("/jdbc/{id}")
    public ResponseEntity<?> query(@PathVariable Integer id){
        try{
            List<Student> student = studentService.getById(id);
            if (!student.isEmpty()){
                return ResponseEntity.status(200).
                    body(student.getFirst());
            }else{
                return ResponseEntity.status(404).
                    body("Student ID : " + id + " Not Found");
            }
        }catch (Exception e){
            return ResponseEntity.status(500).
                body("Internal Service Error");
        }
    }
    ```
    - 其餘丟到github比較省空間 - [Student Controller-Service-Dao實作](https://github.com/learning-official/JavaLearning/tree/main/SpringBoot/src/main/java/test/web/student)

## Day56
#### 學習重點 : Spring Boot - Controller-Service-Dao整體實作 - 1
- 圖書管理系統分析 ⭐⭐⭐
    - 按照 [古古的後端筆記 Spring Boot Day29](https://kucw.io/blog/springboot/29/)，裡面有詳細的程式碼，我也跟著實作了一下，其實跟Student的實作並沒有太大差異，但某些部份可以拿出來研究。
- SQL : Auto increment ⭐⭐⭐
    -  若在創建TABLE時，給予像id這種INT形態一個 `AUTO INCREMENT` 的key word。當我們新增數據時，則 **不需提供id**，數據庫會自動新增，並 **依序遞增**。
    #### Java中取得遞增id方法 : KeyHolder ⭐⭐⭐⭐⭐⭐
    - 我們會利用KeyHolder來取得自動遞增的Key
    ```java=
    KeyHolder keyHolder = new GeneratedKeyHolder();
    namedParameterJdbcTemplate.update(
        sql, new MapSqlParameterSource(map), keyHolder
    );
    int book_id = keyHolder.getKey().intValue();
    ```
    #### ❓疑問區 : 為何是MapSqlParameterSource(map)
    - 仔細看update方法內部的流程 : 
    ![image](https://hackmd.io/_uploads/B15ccfWzMe.png)
    - 🔑前提 : 
        - 從圖中會發現共有四種update method（**method overloading**）
        - 且當我們傳map進去後，其實也會被Spring放進MapSqlParameterSource中，可以說它是做為跟 **SQL參數對應的工具包**。
    - 當今天搭配KeyHolder時，Spring就沒有幫忙做放入工具包的動作了，因此要自行打包 `new ...Source(map)`。 
    - 若有多個 `auto increment`，則可以在參數放入 `String[] id = {"book_id"}` 去指定要抓取的欄位。

## Day57
#### 學習重點 : Spring Boot - Controller-Service-Dao整體實作 - 2
- MapSqlParameterSource省略map ⭐⭐⭐⭐
    - 該類別提供一個方法 `addValue` 可以直接加入key-value pair，省去**創建**HashMap的動作。
    ```java=
    namedParameterJdbcTemplate.update(
        sql,
        new  MapSqlParameterSource()
                .addValue("title", book.getTitle())
                .addValue("author", book.getAuthor())
                .addValue("image_url", book.getImage_url())
                .addValue("price", book.getPrice())
                .addValue("published_date", book.getPublished_date())
                .addValue("created_date", now)
                .addValue("last_modified_date", now),
        keyHolder
    );
    ```
- Dao介面、實作（解耦）⭐⭐⭐⭐⭐⭐
    - 如 [Day38](##Day38) 所介紹，Bean是由Spring IoC管理的物件，可**降低依賴、提高維護性**。
    - 其中的精隨就在於我們注入的是 **介面變數**，使用Qualifier選擇實作的類別，而 **不是直接自行建立** 實作類別的物件變數。
    ```java=
    public interface BookDao {
        public Integer insert(Book book);
        public Book query(Integer book_id);
        public void update(Integer book_id, Book book);
        public void delete(Integer book_id);
    }
    ```
    - 這樣Service在呼叫Dao層時，**不需知道內部邏輯**，只知道有insert、query、update、delete四種方法，若日後有多個實作Dao介面時，只需要利用Qualifier選擇即可。
    - 而Service層以及Controller層也可使用同種方式進行！

## Day58
#### 學習重點 : Spring Boot - Controller-Service-Dao整體實作 - 3
- Inject Clock ⭐⭐⭐⭐⭐⭐
    - ❓問題 : 幹嘛要用Clock？為何不用原本的new Date即可？
    - 💬回答 : 
        - 假設圖書系統中借書的時長是30天，且逾期未還則立即註銷圖書會員。
        - 若我們在 **測試檔** 中，**預期** `2026-03-01 00:00:00` 借書後，超過 `2026-03-31 00:00:00` 就立刻註銷會員，且不因資料庫延遲或時區等問題。
        - 當使用new Date時，系統會抓取電腦當前時區時間，若其中慢了幾秒或是測試地區不同，導致與預期註銷時間不符，則測試不通過。
        - 但當我們使用clock後，能夠自行設置一個固定時區時間，讓測試能夠按照流程，最後通過。

- Clock注入 ⭐⭐⭐⭐⭐⭐
    - 如同上面所說，注入clock後，就可以在測試檔中直接測試，而注入方法如下。
    ```java=
    package test.web.library.config; // 重要

    @Configuration
    public class AppConfig{
        @Bean
        public Clock clock(){
            return Clock.systemDefaultZone();
        }
    }
    ```
    ```java=
    package test.web.library.serivce;

    @Component
    public class BookService {
        
        // 注入clock
        @Autowired
        private Clock clock;
            
        public Integer insert(BookRequest book){
            Date now = Date.from(clock.instant());
            return bookDao.insert(book, now);
        }
        
        public void update(Integer bookId, BookRequest book){
            if (bookDao.query(bookId) == null) return;
            
            Date now = Date.from(clock.instant());
            bookDao.update(bookId, book, now);
        }
    }
    ```
    ```java=
    @SpringBootTest
    public class BookServiceTest {
        // 尚未學習w
    }
    ```
    #### 實際測試流程
    - 1️⃣ 開始測試，測試檔寫一個固定時區時間，並丟給Clock。
    - 2️⃣ BookService向Clock要時間，Clock給他1️⃣設置的。
    - 3️⃣ BookService注入clock變數並且放入BookDao method中。
    - 4️⃣ BookDao操作資料庫時，放入1️⃣設置的時間。

## Day59
#### 學習重點 : Spring Boot - 測試檔建置、Unit Test - 1
- `pom.xml` 引入 ⭐⭐
    ```html=
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    ```
    - scope的存在是因為該依賴檔只需要在測試時使用，於正式打包後，不會被加進來，省下空間。
- 檔案結構 ⭐⭐
    ```
    | -- test
    |     -- java
    |        -- test
    |           -- web
    |              -- library
    |                  -- BookServiceTest.java
    ```
- 測試檔程式結構 ⭐⭐⭐⭐⭐
    - 在測試檔中，需要對class做註解 `@SpringBootTest`，才能夠做為測試的單元。
    ```java=
    @SpringBootTest
    public class BookServiceTest{
        
        @Autowired
        private BookService bookService;
        
        @MockitoBean
        private Clock clock;
        
        @Test
        public void testInsert(){
            // ...
        }
    }
    ```
    #### MockBean
    - 這邊使用的MockitoBean亦即 **模擬一個靜止的clock bean**，而該clock則是 **由test檔所操控**，相對應的AppConfig中的clock則是在實際run後所使用之 **會動的時鐘**。
    - 雖然說今天有實際測試過testInsert，但未認真認識內部的使用方法，而是注重在如何讓測試環境通過，因此先不放上來！
- 時間微調 ⭐⭐⭐⭐
    - 在Mapper中，需要將 `rs.getDate` 改成 `rs.getTimestamp`，因為當初建立TABLE時，是利用TIMESTAMP建立的，若使用getDate，則會捨去 `HH:mm:ss` 這段，這對test檔中測試時間正確性有極大影響。
    ```java=
    book.setPublished_date(rs.getTimestamp("published_date"));
    book.setCreated_date(rs.getTimestamp("created_date"));
    book.setLast_modified_date(rs.getTimestamp("last_modified_date"));
    ```

## Day60
#### 學習重點 : Spring Boot - Unit Test - 2
- Instant類別、與Date差異 ⭐⭐⭐
    - 簡單來說，Instant是Java的一個時間類別，相較於Date來說，Instant較統一，以UTC作為基準，全世界使用Instant都會得到同一時間。
    - Instant取得時間是以 `1970-01-01 00:00:00 UTC 起算的秒數 + 奈秒數`（Unix時間原點），十分精確。
    - 適合用於**存取資料庫時間**、**跨時區**的時間同步。
- Instant實際操作、與Clock關係 ⭐⭐⭐
    - 透過Instant的內部邏輯，可以知道 :
    - `Instant.now()` 會 `return Clock.currentInstant()`
    ```java=
    Instant instant = Instant.now();
    ```
    - 設定時間（字串）
    ```java=
    Instant fixedInstant = Instant.parse("2026-03-01T00:00:00Z");
    ```
    - T作為分隔日期與時間，Z是零時區(格林威治時間)。
    - 透過上述，可以知道，Clock是一個**鐘**(可以自由調整指針)，而Instant則是**取得時間點**（自Unix時間開始）。
- 測試檔操作時間 ⭐⭐⭐⭐
    ```java=
    // 建立時間點
    Instant fixedInstant = Instant.parse("2026-03-01T00:00:00Z");
    // 固定鐘，利用Instant參數 + 時區
    Clock fixedClock = Clock.fixed(fixedInstant, ZoneId.of("UTC"));
    ```

## Day61 
#### 學習重點 : Spring Boot - MockitoBean
- 甚麼是MockitoBean？⭐⭐⭐⭐
    - 它的前身是MockBean，但在SpringBoot3.4後改為MockitoBean，本質上是一樣的用法。
    - 簡單來說，MockitoBean就是來 **模擬假的Bean**，但是為甚麼呢？
        #### 依賴問題
        - 假設Bean A會使用到Bean B跟Bean C，但B、C又依賴其他服務，而我們想專注在A收到B、C返回值後的動作，此時可自行模擬Bean B、C增加效率、除錯。
        #### Exception處理
        - 在服務上線後，最怕的就是各種神奇的Bug沒被修復，因此在測試時，可以利用MockitoBean的方法來設置各種輸入、輸出。
    - 總結來說，Mock過後的 **MockBean會取代** 原本在Spring IoC容器中 **真正的Bean**，因此在測試時呼叫該Bean方法或成員時，Spring會調用我們自行定義的MockBean。
- 如何操作Mockito？ ⭐⭐⭐
    - 使用 `Mockito.when().thenReturn()`來 **模擬呼叫與回傳**。
    - 以我之前寫的Service層來說，當insert中呼叫了clock.instant()去取得當前時間時，我們在when中可以放入clock.instant()去偵測誰呼叫了這個東西。
    ```java=
  Mockito.when(clock.instant()).thenReturn(fixedInstant);
    // getZone本身會跟著instant一起被呼叫，儘管service沒有 "直接調用"
  Mockito.when(clock.getZone()).thenReturn("UTC");
    ```

## Day62
#### 學習重點 : assert斷言
- 斷言的意義 ⭐⭐⭐⭐
    - 斷言通常被用在程式測試當中，只要回傳為False則立刻 `throws AssertionError`，讓我們提早知道錯誤。
- 斷言用於Unit Test當中 ⭐⭐⭐⭐
    - 當我們想要偵測自BookService取得的clock **是否** 為我們宣告的fixedInstant，就可以利用assertEquals這個函式來偵測。
    ```java=
    BookRequest request = new BookRequest();
    request.setTitle("Time");
    // ..省略set
    Integer id = bookService.insert(request);

    assertEquals(
        Date.from(fixedInstant).getTime()
                 ,bookService.query(id).getCreated_date().getTime()
        );
    ```
- 其餘Assert用法 ⭐⭐
    - 其他還有assertTrue（**boolean**），用於斷言該陳述句是否為True，若False則 `throws AssertionFailedError`。

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
