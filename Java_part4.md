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
