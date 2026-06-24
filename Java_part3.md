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
