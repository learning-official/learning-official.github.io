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
        private String a;
        private int b;
    
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
