# Spring Security + JWT 與策略模式授權

## Day63
#### 學習重點 : 使用者資料管控實作（基本建置、註冊架構）
- 基本架構 ⭐
    - 在前面近30天的學習，基本架構也能夠稍微了解怎麼寫了，後續我都放功能實作。
    - 今天先把架構建構起來，接下來就是研究新的method了！
- User屬性 ⭐
    - 目前我加入了包含 : 名稱、帳號、密碼三種屬性。
- **明日預計** :
    - JWT Token
    - 登入POST方法

## Day64
#### 學習重點 : JWT（JSON Web Token）
- 題外話 😭😭😭😭
    - QAQ今天棒球看了好心酸QAQ，POST明天再來做。
- 介紹 ⭐⭐⭐⭐⭐⭐
    - JWT也就是Json Web Token的簡稱，時常用在傳遞認證身分的數據，基本上會有三層，`header.payload.signature`，以點作為分隔。
    - 而根據名稱可以知道這三層其實都是利用Json格式寫入，但最後會轉成Base64格式。
- 分層
    - Header、Payload、Signature
    #### header ⭐⭐
    - header常被用來存放關於token的資訊(alg)，像是要用甚麼演算法加密token、token的type是甚麼(typ)...。
    #### payload ⭐⭐⭐⭐
    - 這層就會存放關於使用者的數據了，不過 **是公開的資訊**，像是信箱、名字之類的～一般來說會加入「使用者id」、「token的到期時間」、「簽發token時間」，因為token也是會過期的！
    #### signature ⭐⭐⭐⭐
    - 這邊就是用於驗證的中心啦~
    - 基本上在後端與資料庫的溝通上，若signature對不上，則不會做任何動作，自然也不會把DB資訊回傳給前端了！
- 疑問？（為何要用Base64這種可被Decode的編碼呢？） ⭐⭐⭐⭐⭐
    - 因為header、payload其實本來就不是隱私資訊，但卻跟signature的 **加密息息相關**！
    - signature會利用前兩部分的Base64編碼進行雜湊演算後再加上私鑰的加密，形成signature，自此形成Token！

## Day65
#### 學習重點 : JWT token生成與驗證 - 1
- Token生成 ⭐⭐⭐⭐⭐⭐
    - 透過昨天的整理，我們知道Token是透過(Header+Payload)的Base64編碼再加上secret key進行雜湊運算所得出來的。
    - 因此今天利用了JwtUtil來作為生成Token的主要區域。
    ![image](https://hackmd.io/_uploads/BJte7QnR-g.png)
- Token的回傳 ⭐⭐⭐⭐⭐⭐⭐
    - 當我們登入後，後端會藉由UserRequest的傳入，對其進行加密（像上面放入了Account在payload），而Secret則由自己定義。
    - 當我們加密完成後，會回傳給前端，並放置於一個隱密的地方（這邊放在Authorization）。
    ```java=
    @PostMapping("/login")
    public void login(@RequestBody UserRequest userRequest
                        , HttpServletResponse response){

        User user = userService.login(userRequest);
        String token = JwtUtil.sign(user, secret);
        response.setHeader("Authorization", token);
    }
    ```
    - 由於今天研究的東西很多，很幸運有大神願意指導我！我明天會再放上更多研究！

### **由於筆記限制10萬字元，因此後續部分移到part-2**
### [⭐點我看Java學習歷程Part-2⭐](https://hackmd.io/@learning-official/Java_learning_2)

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
    ![螢幕擷取畫面 2026-03-10 101819](https://hackmd.io/_uploads/HydgPGWMfg.png)
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
    ![螢幕擷取畫面 2026-03-13 161831](https://hackmd.io/_uploads/B1qzwzWfMe.png)

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
