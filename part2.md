## 目錄
- [Spring 「使用者資料管控專案Part1」 - Day 66~80](##Day66)
- [Multi-Thread - Day 81~90](##Day81)
- [Serializeable - Day91](##Day91)
- [Java IO - Day92~100](##Day92)
- [Exception Handling - Day101~103](##Day101)
- [HashSet - Day104](##Day104)
- [Lambda（Method Reference） - Day105~107](##Day105)
- [Optional淺入淺出 - Day108~110](##Day108)
- [Spring 「使用者資料管控-電商小專案Part2」 - Day111~156](##Day111)

## Day66
#### 學習重點 : JWT token生成與驗證 - 2
- 密碼儲存業務邏輯 ⭐⭐⭐⭐
    - 透過與大神的溝通稍微整理了下更新密碼的業務邏輯 : 
    - 1️⃣ 建立帳號時，首次密碼**存取於temp_password欄位**。
    - 2️⃣ **第一次**登入時，比對temp_password，若成功則放進password欄位儲存。
    - 若忘記密碼，可發送臨時密碼(temp)至使用者email，待登入後選取更改密碼功能，若無，則**清空temp**。
- 密碼更新功能Controller測試 ⭐⭐⭐⭐⭐
    - **Token認證** : 以token作為認證使用者方法，因此要在Request headers放入token。
    - **密碼dto** : 建立PasswordUpdate接收RequestBody傳入的新舊密碼，舊密碼作為認證，若成功則將password欄位改為新密碼。
    - JwtUtil實作認證 : 
    ```java=
    public static void verify(String token, String secret) {
        Algorithm algorithm = Algorithm.HMAC256(secret);
        JWTVerifier verifier = JWT.require(algorithm).build();
        verifier.verify(token);
    }
    ```
    - 密碼更改功能於Controller測試 : 
    ```java=
    @PutMapping("/users")
    public void update(@RequestBody PasswordUpdate passwordUpdate
            , HttpServletRequest request){

        String token = request.getHeader("Authorization");
        JwtUtil.verify(token, secret);
    }
    ```
    - `Bearer <token>` 為固定的Authorization格式，因此在Header時需用此格式才能代表JWT驗證。

## Day67
#### 學習重點 : JWT token生成與驗證 - 3
- 密碼更新Payload解析 ⭐⭐⭐⭐
    - 透過在Jwt的sign中claim的account名稱，去資料庫搜尋使用者，並認證舊密碼是否跟傳進來的oldPassword相符。
    - 不過我目前都是在Dao層判定，好像有點問題ww
    - JwtUtil建立取得payload的資訊 :
    ```java=
    public static String getString(String token, String key) {
        Claim claim = JWT.decode(token).getClaim(key);
        return claim.asString();
    }
    ```
    ```java=
    userService.updatePassword(
        passwordUpdate, 
        JwtUtil.getString(token, "account")
    );
    ```
- 判定測試 ⭐⭐⭐⭐⭐⭐
    - 現在都先用簡單的分析來讓功能先做出來，之後再作優化！
    - 這邊是流程 : 
        - 1️⃣ 利用account取得使用者當前密碼
        - 2️⃣ 比對oldPassword與當前密碼
        - 3️⃣ 若pass則寫入新密碼，若fail則不動作
    ```java=
    String sql = "SELECT * FROM user WHERE account = :account";
        // ...省略
    User user = namedParameterJdbcTemplate.query(
        sql, 
        map, 
        new UserMapper()
    ).getFirst();

    if (Objects.equals(user.getPassword(), passwordUpdate.getOldPassword())){

        user.setPassword(passwordUpdate.getNewPassword());
        String updateSql = 
            "UPDATE user SET password = :password WHERE account = :account";
        // ...省略
        namedParameterJdbcTemplate.update(updateSql, param);
    }
    ```

## Day68
#### 學習重點 : JWT token生成與驗證 - 4
- 密碼更新邏輯（Service層）⭐⭐⭐⭐⭐⭐
    - 1️⃣ 以payload傳進來的account資訊呼叫 `userDao.findUserByAccount()`
    - 2️⃣ 接收到User後，確認是否為null，若否.則進入比較階段，若是.則回傳false。
    - 3️⃣ 比較成功，則呼叫 `dao.updatePassword()` 傳入user並更新。
    ```java=
    public boolean updatePassword(PasswordUpdate passwordUpdate, String account){
        
        // user接收
        User realUser = implementUserDao.findUserByAccount(account);
        if (realUser == null) return false; // 確認狀態
        // 比較password
        if (Objects.equals(realUser.getPassword(), passwordUpdate.getOldPassword())){
            
            // 成功則setPassword並傳入dao層
            realUser.setPassword(passwordUpdate.getNewPassword());
            implementUserDao.updatePassword(realUser);
            return true;
        }
        return false;
    }
    ```
- 密碼更新邏輯（Dao層) ⭐⭐
    - 這邊就很簡單，Dao層 **只負責與資料庫溝通**，因此不參與比對環節。
    ```java=
    @Override
    public void updatePassword(User user){

        String updateSql = "UPDATE user SET password = :password WHERE account = :account";
        BeanPropertySqlParameterSource param = new BeanPropertySqlParameterSource(user);
        namedParameterJdbcTemplate.update(updateSql, param);

    }
    ```

## Day69
#### 學習重點 : Java BCrypt Introduction
- Bcrypt架構 ⭐⭐⭐⭐⭐⭐
    - 分層分為 -> alg、cost factor、salt、hashed
    - 與Jwt的驗證一樣都有分層！但實際結構還是有些許不同，主要是Jwt跟BCrypt實際用處也不太一樣。
    ![image](https://hackmd.io/_uploads/S16_1Z6tZx.png)
    - **alg** : 演算法的種類。
    - **cost factor** : 重複加密幾次 -> 越多次越安全，但加密時間越長。
    - **salt** : 每次**隨機產生**，增加多樣性，驗證機制 - 從密文拿出salt區塊，加上使用者提供的密碼進行雜湊，若與hash區域相同則通過。
    - **hash** : 由salt + password進行雜湊出來的東西，會儲存在使用者password欄位。
- 實際Java加密與驗證 ⭐⭐⭐
    - 加密、驗證，兩個步驟，Java BCrypt其實很易懂，就稍微寫了下。
    ```java=
    public class BcryptUtil {

        public static String genHashedPassword(String password){
            int logsalt = 12;
            String salt = BCrypt.gensalt(logsalt);
            return BCrypt.hashpw(password, salt);
        }

        public static boolean verify(String insertPassword, String hashedPassword){
            return BCrypt.checkpw(insertPassword, hashedPassword);
        }
    }
    ```
## Day70
#### 學習重點 : BCrypt應用至Service層
- 生成加密密碼於Service ⭐⭐⭐⭐
    - 在Service層中，我加了一個 `userBCryptSettings` 來處理傳進來的使用者數據。
    ```java=
    public static void userBCryptSettings(User user){
        String hashedP =
            BcryptUtil.genHashedPassword(user.getPassword());
        user.setPassword(hashedP);
    }
    ```
    - 由於後續可能會在user型別中加入其他需加密的東西，所以settins就不單純接收String而是User了。
    #### 注意🚨🚨
    - 這邊可以看到傳入的是User物件，也就是所謂的 **call by reference**，傳進來的是地址，並沒有 `new` 的動作，因此地址是原本外部的，更改的user也自然是外部的user。
- 修改Object.equals ⭐⭐⭐⭐
    - 原本我是利用Object的比較method，**但** 現在是利用BCrypt的verify功能，直接將oldPassword與realUser的加密密碼進行驗證！
    - 我感覺我很容易就忘記如何verify，這邊先寫下來好了w : 
        - 1️⃣ 從雜湊密碼中拿出中間段的salt。
        - 2️⃣ 將 **salt** 與傳進來 **要驗證的密碼** 進行雜湊。
        - 3️⃣ 比對 **驗證密碼的雜湊值** 與 **原本資料庫存的雜湊密碼**。
    ```java=
    public User login(UserRequest userRequest){
        User user = new User();
        // ...省略設定

        User realUser = implementUserDao.login(user);
        if (BcryptUtil.verify(user.getPassword(), realUser.getPassword())){
            return realUser;
        }else{
            return null;
        }
    }
    ```
    - updatePassword也是同樣的道理，這邊就不放上來了。

## Day71
#### 學習重點 : 忘記密碼功能實作（隨機密碼產生）
- 引入 `apache.commons-text` ⭐
    - 簡單來說就是引入一個能夠生成隨機密碼的東西。
    - 原本是想要手刻一個啦w 但感覺目前先專注在系統的實作，之後有時間再來研究！
    ```html=
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-text</artifactId>
        <version>1.10.0</version>
    </dependency>
    ```
- 基本架構 ⭐⭐⭐
    - 按照apache提供的文檔，我稍微模仿了一下 : 
    ```java=
    String rsg = new RandomStringGenerator.Builder()
            .withinRange('0', 'z')
            .filteredBy(Character::isLetterOrDigit)
            .build().generate(12);
    ```
    #### Method reference (::)
    - 我記得遙遠的Day?? 我有學習過lambda的用法，簡單來說就是interface的實作簡寫用法。
    - 而這邊的method refernece就是lambda的更簡寫用法，連參數都省掉了，我自己覺得有點太簡化了w。

## Day72
#### 學習重點 : 忘記密碼功能實作（mail發送）
- 忘記密碼功能想法 ⭐⭐⭐
    - 由於忘記密碼功能通常是 : 
        - 1️⃣ 點選忘記密碼功能 
        - 2️⃣ 寄送臨時密碼 or 更改密碼按鈕至使用者設定之信箱 
        - 3️⃣ 使用臨時密碼登入後再更改 or 按按鈕後更改
    - 我這邊選擇配合我的 `updatePassword` 與 `temp_password` 的 **臨時密碼登入**。
- 郵件工具建立
    - 這邊需要利用spring.mail的工具
    - `pom.xml` 引入 ⭐
    ```html=
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-mail</artifactId>
    </dependency>
    ```
    - application.properties 設定郵件伺服器、傳送者等。 ⭐⭐⭐⭐
    ```properties=
    spring.mail.host=smtp.gmail.com
    spring.mail.port=587
    spring.mail.username= 我的gmail帳號
    spring.mail.password= 到安全性步驟中申請應用程式密碼
    spring.mail.properties.mail.smtp.auth=true
    spring.mail.properties.mail.smtp.starttls.enable=true
    ```
    - 接著是基本架構 (**mailService**) : ⭐⭐⭐⭐⭐⭐
    ```java=
    @Service
    public class MailService {
        
        
        // 注入springboot的mailSender
        @Autowired
        JavaMailSender javaMailSender;

        public void sendMail(String mailReceiver,
                             String subject,
                             String body){
            // 設定內容
            SimpleMailMessage smm = new SimpleMailMessage();
            smm.setTo(mailReceiver);
            smm.setSubject(subject);
            smm.setText(body);

            javaMailSender.send(smm);
        }
    }
    ```
    - 再來是搭配Service當中的forgotPassword ⭐⭐⭐⭐⭐⭐⭐⭐
    ```java=
    public void forgetPassword(String account){
        
         // 先利用account找到使用者數據
         User user = implementUserDao.findUserByAccount(account);
         // 生成隨機密碼
         String rsg = new RandomStringGenerator.Builder()
                            .withinRange('0', 'z')
                            .filteredBy(Character::isLetterOrDigit)
                            .build().generate(12);
         
         // 加密隨機密碼 
         String hashedRandomPassword = 
             BcryptUtil.genHashedPassword(rsg);

         // 將加密密碼與帳號交由dao層放入資料庫
         implementUserDao.
             forgetPassword(account, hashedRandomPassword);
        
         // 交由 mailService 處理寄發mail的事情
         mailService.sendMail(account, "Temporary password", rsg);
    }
    ```
    - **實際成果如下**，接下來就是處理temp_password登入後刪除臨時密碼的功能啦~不過今天先這樣就好！
    ![螢幕擷取畫面 2026-03-13 161831](https://hackmd.io/_uploads/SkWOFH-qWx.png)
    
## Day73
#### 學習重點 : 忘記密碼功能實作（臨時密碼登入）
- 密碼驗證 - 新增可變參數動態陣列、方法性質修正 ⭐⭐⭐⭐⭐
    - 在進入臨時密碼登入前，我先對BCryptUtil的verify功能做了稍微更改 : 
    ```java=
    // public改private、static移除 : 由於service是bean，不應有類別屬性
    private boolean verify(
        String insertPassword, String... hashedPassword){
        
        // 迴圈跑過所有加密密碼
        for (String hp : hashedPassword){
            // 避免發生nullPointerExeption，我加了個保險
            if (hp == null) continue;
            if (BCrypt.checkpw(insertPassword, hp)){
                return true;
            }
        }
        return false;
    }
    ```
    - 由於有臨時密碼，若在Service層一直用 **if-else重複呼叫verify** 會 **很繁瑣**，又剛好我之前有學過Varargs的用法，想說剛好派上用場ww
- 臨時密碼登入 ⭐⭐⭐⭐⭐
    - 當我忘記密碼並成功寄發臨時密碼後，temp_password欄位更新，我就可以 **接收temp_password** 並用於login的verify上了，以下是我的邏輯 : 
    ```java=
    public User login(UserRequest userRequest){

        User realUser = implementUserDao
            .findUserByAccount(userRequest.getAccount());

        boolean isValid = BcryptUtil.verify(
            userRequest.getPassword(), 
            // 看看原本password或者temp_password哪個可以對上
            realUser.getPassword(), 
            realUser.getTemp_password()
        );

        // 【確認」登入成功後，將temp設為null保持淨空並更新至資料庫
        if (realUser.getTemp_password() != null && isValid){
            realUser.setTemp_password(null);
            implementUserDao.updatePassword(realUser);
        }

        return isValid ? realUser : null;
    }
    ```
## Day74
#### 學習重點 : 刪除帳號實作
- 刪除帳號功能 ⭐⭐
    - 其實也沒有甚麼好講的ww，就是單純的刪除動作，只是一樣要利用JWT token驗證，以下我在Controller中的實作。
    ```java=
    @DeleteMapping("/users")
    public ResponseEntity<?> delete(HttpServletRequest request){

        String token = request.getHeader("Authorization");
        JwtUtil.verify(token, secret);
        String account = JwtUtil.getString(token, "account");
        
        userService.deleteAccount(account);
        
        return ResponseEntity.ok().build();
    }
    ```
    - 對於某些部分，我想要在明後天做個修正 : 
    - 1️⃣ JWT驗證需要有exception做攔截，因為現在沒有細分錯誤。
    - 2️⃣ BCrypt的Bearer前輟要補上，並且在傳進token時再做字串分析。
    - 3️⃣ token要加上更多訊息，像是exp之類的。
    - 我有看到一張圖去解釋status code對應的token錯誤 : 
     ![image](https://hackmd.io/_uploads/By4GbGEcbe.png)

## Day75
#### 學習重點 : JWT解析token、token過期時間新增
- Bearer標記 ⭐⭐⭐⭐⭐
    - Bearer是一種認證機制，表示token是透過JWT格式加密的，前面也實作了一堆JWT的東西，這邊就不贅述了w
    - 在後端，我新增了Bearer標籤功能，當signature製作完畢後，我讓token加上 `Bearer` 字串，**表示是JWT形式的token**。
    - 而解析的方式目前若發現是非Bearer就回傳null，目前尚未加入throws功能。
    ```java=
    // 目錄 : Util/JwtUtil
    public static String authorization(String rawCode){
        return String.format("Bearer %s", rawCode);
    }

    public static String decode(String code){
        if (code.startsWith("Bearer ")) {
            return code.substring(7);
        }else {
            return null;
        }
    }
    ```
- 加入exp - token到期時間 ⭐⭐
    - 這邊使用簡單的Date形式，並作相加
    ```java=
    // 目錄 : Util/JwtUtil
    private static final long EXPIRED_TIME = 1000*10; // 10秒

    public static String sign(User user, String secret) {
        Algorithm algorithm = Algorithm.HMAC256(secret);
        long time = System.currentTimeMillis();

        return JWT.create()
                .withClaim("account", user.getAccount())
                .withIssuedAt(new Date(time))
                // 讓現在時間 + 簽署開始後多久到期，由EXPIRED_TIME定義
                .withExpiresAt(new Date(time+EXPIRED_TIME)) 
                .sign(algorithm);
    }
    ```

## Day76
#### 學習重點 : ControllerAdvice - 例外處理
- 簡介 ⭐⭐⭐⭐
    - 建立Exception的資料夾，在其中需要有**Handler**以及**例外的種類**。
    - 1️⃣ 先定義例外的類型，並且定義errorCode以及errorMessage。
    - 2️⃣ Handler專門處理例外的各種問題，決定要回應什麼樣的status code以及訊息。
- 程式架構 ⭐⭐⭐⭐⭐⭐
    - 例外類型定義（以驗證來做測試）
    ```java=
    package org.system.exception;

    public class AuthException extends RuntimeException{

        private int errorCode;
        private String errorMessage;

        public AuthException(int errorCode, String errorMessage){

            this.errorCode = errorCode;
            this.errorMessage = errorMessage;

        }
        // getter & setter
    }
    ```
    - Handler處理
    ```java=
    package org.system.exception.handler;

    // ...省略
    @ControllerAdvice
    public class testExceptionHandler {

       @ExceptionHandler({AuthException.class})
       public ResponseEntity<?> handleAuthException(AuthException e){

           if (e.errorCode == 1001){
               return //...省略
           }
           // ...省略
       }
    }
    ```
    - 由於今天真的超級忙，明後天再來完整處理其他錯誤w

## Day77
#### 學習重點 : ControllerAdvice - 例外處理分類
- 簡單分類 : ⭐⭐⭐⭐⭐⭐
    - 我稍微統整了一下可能會出現的錯誤 : 
        - **驗證錯誤類** : token過期、token有誤、Bearer錯誤、。
        - **業務邏輯類** : 註冊時帳號已存在、登入時帳密有誤、更新密碼時舊密碼有誤。
        - **資源找不到類** : 忘記密碼時填入的信箱並不在DB中。
    - 由於 **驗證錯誤類** 會交由auth0內建的 `JWTVerificationException`，因此我分了 `BusinessLogicException`、`ResourcesNotFoundException`。
- 業務邏輯類 : ⭐⭐⭐
    - 以下是我的分類程式 : 
    ```java=
    @ExceptionHandler({BusinessLogicException.class})
    public ResponseEntity<?> logicHandle(BusinessLogicException e){

        if (e.getErrorCode() == 1001){
            // 當註冊帳號時，帳號已存在
            return ResponseEntity.status(409).build();
        }else if (e.getErrorCode() == 1002){
            // 重設密碼時，舊密碼不符
            return ResponseEntity.status(400).build();
        }else if(e.getErrorCode() == 1003){
            // 登入時帳密有誤
            return ResponseEntity.status(401).build();
        }else{
            // 未知錯誤先回傳BAD_REQUEST
            return ResponseEntity.status(400)
                .body(e.getErrorMessage());
        }
    }
    ```
    - 我利用registration來加上錯誤引流 : 
    ```java=
    public void registration(UserRequest userRequest){
        try{
            //...省略
            if (userDao.findUserByAccount(userRequest.getAccount()) 
                != null){
                throw new BusinessLogicException(1001, 
                                        "Failed registration");
            }
            userBCryptSettings(user);
            implementUserDao.registration(user);
        }catch (BusinessLogicException e){
            throw e;
        }catch (Exception e){
            System.out.println("錯誤訊息 : " + e.getMessage());
            throw new BusinessLogicException(
                9999, 
                "系統出現未知錯誤，請稍後再試...。"
            );
        }
    }
    ```
    - 明天繼續努力將其完善！

## Day78
#### 學習重點 : 剩餘例外處理分流、Controller & Service層簡潔化
- 話說我覺得程式碼好亂，乾脆用照片貼上比較簡潔一些w，之後都改用這個方法好了。
- 剩餘例外處理分流 ⭐⭐⭐
    - 我建立了 `ResourcesException`，並在Handler新增了處理資源找不到的方法。
    ![image](https://hackmd.io/_uploads/BJexNcK9bl.png)
- Controller & Service層簡潔化 ⭐⭐⭐⭐⭐⭐
    - 由於Controller不知道被哪個~~阿呆~~用try-catch搞得烏煙瘴氣，導致職責分離不清。
    - 因此我今天把整個Controller優化成 **只專注回應**、Service處理 **throw exception的問題**，而exception後續動作就 **交給handler** 去用了~
    - 大概長這樣 : 
    ![image](https://hackmd.io/_uploads/BJ1mS5Fcbl.png)

## Day79
#### 學習重點 : Response業務狀態回應
- Response類別 ⭐⭐⭐⭐⭐⭐⭐⭐
    - 該類別通常會自行定義「**業務**」狀態碼，針對自己邏輯的各種可能，屬於 **業務邏輯狀態**，也可以存取user物件，讓回傳更加靈活！
    ![image](https://hackmd.io/_uploads/B1-jIkic-l.png)
    - 這邊有偷用了LOMBOOK的工具ww，簡單來說就是省去了getter&setter的部分~
- Response應用 ⭐⭐
    - 而我目前針對login的部分，若臨時密碼欄位不為空，則導向更改密碼的頁面（雖然可以再優化，但目前先這樣ww），Response則可以 **設定rc** 使得後續可以決定要跳轉到哪個頁面（首頁or更改密碼頁面）。
    ![image](https://hackmd.io/_uploads/Hy_dIkj9Wg.png)
    - 目前Exception的部分我還在修正，可能得再花個幾天時間QAQ。

## Day80
#### 學習重點 : 使用者資料管控實作（Part-1結業式🥳）
- 總結 ✨✨✨✨
    - 儘管Exception並沒有設計的很理想，但我想，這個可以留給幾十天後的我再來處理！我覺得是時候先去學習更多概念了！
    - 另外，以下是我對這次系統的功能介紹 : 
    #### 加密功能 🔑
    - **JwtUtil** : 負責 `加密、驗證、Bearer前輟新增or刪除` ➜ [**Day65 JWT研究**](https://hackmd.io/@learning-official/Java_learning#Day64)
    - **BCryptUtil** : 負責 `加密、驗證` ➜ [**Day69 BCrypt研究**](##Day69)
    #### 註冊功能 ®️
    - 要求使用者輸入 : `名稱、帳號、密碼`。
    - 搭配 `registration` 方法接收 `userRequest` 的dto格式。
    #### 登入功能 🔐
    - 要求使用者輸入 : `帳號、密碼`。
    - 搭配 `login` 方法接收 `userRequest`、`前端response結構`。
    #### 臨時密碼功能 📩 ➜ [Day71 忘記密碼研究](##Day71)
    - 要求使用者輸入 : `帳號`。
    - 以 `RandomStringGenerator` 生成隨機密碼並 **寄信至輸入帳號**。
    - 搭配 `forgetPassword` 方法接收 `路徑參數account`，寄信到指定信箱。
    - 無論資料庫是否有該帳號，一律回應ok。
    #### 使用者介面重設密碼功能 🔄
    - 要求使用者輸入 : `新密碼`、`舊密碼`。
    - 以 `JwtUtil` 做舊密碼驗證。
    #### 刪除帳號功能 🗑️
    - 簡單來說是刪帳號ww，沒什麼好說的。
- 搭配前端架構呈現 ✨✨✨✨✨✨
    - 我請AI幫我建置了一個 **static的檔案資源**（html、js、css），以介面來呈現我的成果！
    ![image](https://hackmd.io/_uploads/BkU4v739bx.png)
    ![image](https://hackmd.io/_uploads/S1-yum29be.png)
    ![image](https://hackmd.io/_uploads/BkJz_Xh9Zx.png)

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

 ## Day104
#### 學習重點 : Collection Framework - Set & HashSet
- Set是甚麼？ ⭐⭐⭐⭐ 
    - 與List跟Map一樣，Set也是Collection Framework中的一員。
    - Set是一個具「無序、唯一性」特質的容器。
    - 因此當我們想要存儲資料又希望達成以上兩個特質的話，就可以選用Set！
- 如何使用Set？ ⭐⭐
    - 與集合框架其他成員一樣，需初始型別且選用實作介面的類別（HashSet）。
    ```java=
    Set<String> names = new HashSet<>();
    ```
    - 接著可以使用add、remove等方法來操作Set，這邊就省略拉~
- Set與List的連結 ⭐⭐⭐⭐⭐
    - 由於兩者都是元素集合容器，只是一個有序且不唯一，另一個無序且唯一，因此我們可以利用Set的特性來「過濾」List的元素，使用方法如下 : 
    ```java=
    public static void main(String[] args){
        List<Integer> numberList = new ArrayList<>();
        numberList.add(1);
        numberList.add(2);
        numberList.add(3);
        numberList.add(1);

        Set<Integer> numberSet = new HashSet<>(numberList);
    }
    ```
    - 由於Set在被建構時，可以傳入集合框架的成員作為元素指派，因此這邊直接傳入numberList，當輸出時可以發現僅有 `1,2,3`，並不是 `1,2,3,1`。

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
    - 使用toMap要注意的事情 : 
        - 1️⃣ 不允許NPE的出現，因此如果偵測到null，務必做邏輯處理(像Optional那樣)
        - 2️⃣ 由於key是unique的，因此不能使用getName作為Key，應該使用唯一鍵。
