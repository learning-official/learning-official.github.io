# 集合框架、泛型與比較器

## Day21
#### 學習重點 : ArrayList (Collection Framework)
- Array複習 ⭐⭐
    - Array是一個 **固定容器**，裝著相同型態的物品，而我們宣告它的方法就是利用 `型態[]`表示容器，在初始化的時候，就必須對其賦值，因此大小不可改變。
    ```java=
    String[] names1 = new String[5]; // 固定容量為5
    String[] names2 = { // 固定容量為3
        "Alice",
        "Bob",
        "Carol"
    };
    ```
- ArrayList概念 ⭐⭐⭐⭐⭐⭐
     - ArrayList表示的是一個 **動態陣列**，不需在初始化的時候給予容量大小，而是根據使用者新增or刪除 **動態調整容量**。
    - 無法裝 **primitive type**(原始型別 : int、double...)，可以裝Objects、**wrapper types**(Integer、Character、Boolean)。
    ```java=
    public class Test{
        public static void main(String[] args){
            
            // 型態宣告於前面，後面可省略不寫
            ArrayList<String> names = new ArrayList<>(
                Arrays.asList("Alice", "Bob", "Carol")
            );
            // asList方法給予陣列
            System.out.println(names.get(0)); // Alice

            names.set(0, "Ariel"); // 替換陣列元素
            System.out.println(names.get(0)); // Ariel

            names.add("Daniel"); // 加入元素在尾巴
            System.out.println(names.get(3)); // Daniel
            
            // 也可以直接選擇物件刪除，不一定要是index
            names.remove("Daniel");
        }
    }
    ```
- toString差異 ⭐⭐⭐⭐⭐⭐
    - 在Object類別當中，有一個toString用法，當我們在 `System.out.println(物件)` 時，編譯器會使用 `.toString()` 的method將其印出來。
    - Array
        - 由於Array物件 **並沒有覆寫** Object.toString()方法，因此編譯器會直接執行父類別Object的toString，導致印出來的是物件的記憶體位置。
    - ArrayList
        - ArrayList **有確實覆寫** Object.toString()，因此可以印出完美的陣列長相，ArrayList同時給予很多方便的method，如上面所展示，這都是 **Collection Framework** 的功勞(雖然我還沒學ww)。

- 實際應用層面的差異 ⭐⭐⭐⭐⭐⭐
    - 儘管ArrayList提供很多method，但是由於其 **內部儲存的是object**，因此會比一般Array存prmitive type還要慢上很多，因此在開發ML or 矩陣運算 or 定量資料時，還是會選擇使用Array。

## Day22
#### 學習重點 : LinkedList & 淺淺的資料結構
- LinkedList與ArrayList的差別(part 1) ⭐⭐⭐⭐⭐⭐⭐⭐⭐
    - 兩者雖然都是List介面下的實作，架構相同，但內容卻大相逕庭，兩者對資料處理的方式有很大的不同。
    - 這邊稍微跳槽到資料結構的部分來看看LinkedList，淺淺的碰一下就好🤣，畢竟主軸在學Java
    #### LinkedList的指向，參考 [GeeksforGeeks : Types of Linked List](https://www.geeksforgeeks.org/dsa/types-of-linked-list/)
    - 單指向(Singly LinkedList, SLL)
    ![SLL](https://hackmd.io/_uploads/B13rOMbGfg.png)
        - 所謂單指向就是指每個node(節點)存有下個Data的address pointer
    - 雙指向(Doubly LinkedList, DLL)
    ![DLL](https://hackmd.io/_uploads/B1bruGWGfx.png)
        - 雙指向亦即每個node都存有previous以及next的Data address pointer
        #### 與單指向(SLL)比較 : 查找數據
        - 儘管記憶體必須 **比SLL多出一些空間來存prev**，但查找數據時，**可比SLL少一半時間**(因為可根據index位置來決定要從Tail開始還是從Head)
        #### 與單指向(SLL)比較 : 刪除數據
        - 若在單指向中要刪除node G，我們必須知道G前面是誰，因此要從頭開始跑，跑到F之後再把F的next pointer接到H身上，這樣超級麻煩。
        - 但在雙指向中，node G本身有存prev跟next位置，可以直接利用
        ```java=
        G.prev.next = G.next
        G.next.prev = G.prev
        // 這樣替換即可刪除G數據
        ```
    - 迴圈指向(Circularly LinkedList, CLL)
    ![CLL](https://hackmd.io/_uploads/BJzN_fWGMg.png)
        - 迴圈指向中tail的next指向 **並不是null** 而是head的位置，因此可以達到循環。
        - 像是聽歌時，歌單是一個List，當我開啟循環播放後，最後一首歌所指向的就是第一首歌。
            ```java=
            // 原本(in SLL or DLL)
            tail.next -> null
            
            // 開啟循環播放(in CLL)
            tail.next = head // 指派head位置給tail.next
            ```
    - Java中的LinkedList類型
        - 在Java中，通常採用Doubly LinkedList，也蠻好理解的？ 明天來嘗試實作看看這些類型的LinkedList好了，順便跟ArrayList比較看看。

## Day23
#### 學習重點 : LinkedList-DLL實作 + remove功能實現(整合Class觀念)
- 原本我只是想單純把Geeks那邊的code拿過來看一看抄一抄，但抄著抄著就來勁了🤣，直接加了個remove功能，順便整合之前學的Class建構跟一些關鍵字用法，腦袋打結了不知道多少次，但最後解開超開心，[完整程式碼 in github](https://github.com/learning-official/JavaLearning/tree/main/LinkedListExercise)。
    #### 製作LinkedList架構思路 : 🔥🔥🔥🔥🔥🔥🔥
    - DLL.Node型態 
        - data : **int**、prev : **Node**、next : **Node**) 
        - 做一個Node class
        ```java=
        private static class Node {
            Node next, prev;
            int data;

            Node(int data){
                this.next = null;
                this.prev = null;
                this.data = data;
            }
        }
        ```
    - LinkedList物件 : 
        - 數組輸入
        - 依序new出Node物件並放進其data成員中，形成一組Node陣列
        - 將.next以及.prev串起來
        - Node[0] 存入 this.head、Node[length-1] 存入 this.tail
        ```java=
        LinkedListT(int[] array){
            if (array.length == 0) {
                System.out.println("陣列是空的!");
                return;
            }

            Node[] nodeG = new Node[array.length];
            this.length = array.length;

            for (int i = 0; i<array.length; i++){
                nodeG[i] = new Node(array[i]);
            }

            for (int i = 0; i<array.length-1; i++){
                nodeG[i+1].prev = nodeG[i];
                nodeG[i].next = nodeG[i+1];
            }
            // 讓head作為nodeG[0]的別名，當建構完成後，nodeG消失，head留著
            // 並存有next的位置 -> Reachability，不會被GC回收
            this.head = nodeG[0];
            this.tail = nodeG[array.length - 1];
        }
        ```
    - remove功能 : 
        - 利用基本的 `node.next.prev = node.prev` 以及 `node.prev.next = node.next` 串接
        - 使node.next 以及prev = null 進行斷鍵，讓node不被指也不指別人 -> **Unreachable** -> 被刪除(GC)
        ```java=
        public void remove(int index){
            if (this.length == 0) return;
            Node node = this.head;
            // 讓node跟head指向同一塊記憶體位置，node是head的reference
            // node本身是要被刪除的那個，因此整體都是由node在跑

            if (index <0 || index >= length){
                System.out.println("超出陣列範圍!");
                return;
            }

            for (int i = 0; i < index; i++) node = node.next;

            // 顧好Head位置
            if (node.prev == null){
                this.head = node.next;   
                if (this.head != null) this.head.prev = null;
            }else node.prev .next = node.next;

            // 顧好Tail位置
            if (node.next == null){
                this.tail = node.prev;
                if (this.tail != null) this.tail.next = null;
            } else node.next.prev = node.prev;

            // 最後要讓node本身跟指向斷開
            node.next = null;
            node.prev = null;
            this.length -= 1;
        }
        ```
    - 老實說，我突然想到可以用多建構子的方式，可以針對不同型態做儲存，不過太麻煩了，算了ww，明天再來跟ArrayList做比較🔥

## Day24
#### 學習重點 : ArrayList與LinkedList的詳細比較
- 關於Access的差別 ⭐⭐⭐⭐⭐⭐
    #### LinkedList : 
    - 由前兩天所學的，可以猜到它是 **Sequential access**，也就是**搜尋時**要從head or tail開始搜尋，**時間複雜度O(n)**。
    #### ArrayList :
    - 利用 **Random access**，相較於LinkedList每個node位置是分散的，ArrayList的**數據位置是相連的**，因此在搜尋時，只需利用 `陣列起始位置 + (index * 型態的byte)` 即可存取，**時間複雜度O(1)**。
- 關於Insert & Remove的差別 ⭐⭐⭐⭐⭐⭐
    #### LinkedList : 
    - 在新增或刪除只需要針對該index的node.next跟prev動手腳就好，斷開再接起來就這麼簡單，**理論上 : 時間複雜度O(1)**...
    不過在實際處理中，若不知道該節點位置，就必須利用**for迴圈**跑到該index進行動作，因此 **時間複雜度又變成O(n)**
    #### ArrayList : 
    - Array在新增跟刪除資料就整個超搞剛，要先copy整坨資料再一個一個放進 **新的Array** 當中，再把要刪除或新增的資料根據index隨著copy過程放進去，因此**時間複雜度O(n)**。

- 關於實際執行的反差 ⭐⭐⭐⭐⭐⭐⭐⭐⭐
    - 雖然理論上LinkedList有其厲害之處，但是在實際計算上由於它的**記憶體不連貫性** 導致在增刪當中除了要for迴圈找index(**O(n)**)之外還要讓在不同位置跳來跳去，讓整個讀寫速度慢的有剩。
        - 不過如果是 **已知node位置 or 只對head動手腳** 的情況 **O(1)**，那麼還是用LinkedList比較好。
    - 而在ArrayList當中，由於它**記憶體是連貫的**，CPU剛好對整塊記憶體的copy很有效率，也**符合cache的邏輯**，因此儘管同樣是O(n)但實際還是會比LinkedList快。
        #### 總結來說，ArrayList還是主要動態陣列用法。

## Day25
#### 學習重點 : HashMap、Hash原理
- HashMap介紹 ⭐⭐⭐⭐⭐⭐
    - 實作Map介面，與Python中的Dictionary相同，都是儲存key、value。
    - 與List兩個實作相同，**只能存Wrapper Classes**，不能存primitive type
    ```java=
    import java.util.HashMap;
    public class Test{
        public static void main(String[] args){
            HashMap<String, Integer> dict = new HashMap<>();
            
            // 新增資料 : {Alice=100}
            dict.put("Alice", 100); 
            // 刪除資料 : {}
            dict.remove("Alice"); 
            
            // 替換資料，若key不存在，就不動作 : {}
            dict.replace("Bob", 80); 
            
            // 若該key不存在就新增，反之則不動作 : {Carol = 70}
            dict.putIfabsent("Carol", 70); 
            
            dict.get("Alice"); // 用key查value : null(因為找不到)
            dict.containsKey("Apple") // false
            dict.containsValue(100) // false
        }
    }
    ```
- HashMap存資料的原理 ⭐⭐⭐⭐⭐⭐
    - 首先，Map本身是以key作為index加上 **以array儲存value** 建立起來的。
    - 因為是用Array，因此在搜尋時，**時間複雜度為O(1)**。
    - 假設我們要求使用者建立帳號，會建立User物件，其中有username以及password，我們以 **username作為key**，User物件作為value，對username做hashing，而實際hashing跟存取方式請看下方 ->
    #### HashFunction : 👉key轉化為index
    - 其存在的意義就是讓單純的key被hash成一個 **難以被反推** 的數字(**Hash code/key**)，並以它做為Array的index。
    #### Bucket : 👉裝User的地方
    - 由於Hash code通常較大，若將所有hash code作為index存取，整個陣列會超級龐大。
    - 因此會利用mod，`"hash(key)" mod "length"`，縮減成0~length-1的大小。
    - 而每個index又稱Bucket，因為可能有多個key經hashing跟mod後，index還是一樣，此時即為Collision。
    #### Collision : 👉一堆User住同樣的Bucket
    - 當多個hash code經模運算後得出相同的index就稱為Collision。
    - 若發生衝突，在搜尋時就變成多個key對上同一User，此時有兩種解決方案 : **Open addressing、Chaining**。
    #### Open Addressing : 👉處理多User住同Bucket的問題
    - 簡單來說，就是發生衝突時就往下找直到找到空位(其他bucket) 放進去，這個動作叫做**Linear probing(線性探測)**。
    - 但這衍伸一個問題 - 當過多hash code被分配到同一Bucket，由於線性探測作用，會造成附近的Bucket都塞滿，此時稱為**Clustering(很擁擠)**。
    - 此時可以用Quadratic或者Double probing來產生較亂的probing順序以減少Clustering發生。
    #### Load Factor : 👉負載因子
    - 用以說明 `目前資料量 / 該array的長度`，Load Factor越接近1，Clustering機率越高，此時就得resize array(擴容) & rehash data，因此我們得盡可能將Load Factor控制在75%以內，**以保持O(1)**。
    #### Chaining - Closed Addressing : 👉傳統上處理多User住同Bucket的辦法
    - 傳統上，發生Collision時，會利用LinkedList來存與Bucket衝突的所有User，並在User類別中存入該node的nextIndex。
    - 因為是用LinkedList，所以如果Hash Function設計不好導致Collision，而進入到LinkedList這個地步，**時間複雜度就回到O(n)**。

## Day26
#### 學習重點 : Generics泛型 - 1
- 泛型是甚麼？ ⭐⭐⭐
    - 在List跟Map當中都可以看到 `< >` 這個東西的存在，我們知道它是在宣告陣列 & Map的 **型別**，也難怪叫做泛型，因此泛型可以說是在我們在型別使用上的 **變數**
    - 當我們需要**對多個型別做同樣的事情**，一般來說要寫N個同樣的檔案，只是型別不一樣，這時就可以利用泛型來整合成一個檔案。
    - 由於泛型的實作是**透過Object來做的**，因此在使用泛型時，其實是以 "它是extends自Object" 來實作的，因此必須是封裝型態，所以才**不能使用int、double**等原始資料型態，而是用Integer、Double...，這樣才能拿來宣告 **物件**。
- 泛型怎麼用？ ⭐⭐⭐⭐
    - 在泛型當中，有三種用法 `Generic Class / Interface / Method`
    - 今天卡忙，僅討論Class的用法，我們會在Class名稱後方加入 `< >`，中間就是放入 **型別變數名稱**，因此可以自己取名，但是一般都用T (Type)。
    ```java=
    public class Test <T> {
        
        // 底下就可以利用T，以是一個型態的用法下去用。
        private T Athing;
        public Test(T Athing){
            this.Athing = Athing;
        }
        
        public void print(){
            System.out.println(Athing);
        }
    }
    ```
    - 測試時就用跟List & Map相同的方式下去做
    ```java=
    public static void main(String[] args){
        Test<Integer> testInt = new Test<>(55);
        testInt.print(); // 印出55
    }
    ```

## Day27
#### 學習重點 : Generics泛型 - 2
- Generics class的繼承 ⭐⭐⭐⭐⭐⭐
    - 由於T本身也是代表一個類別，因此我們也可以利用extends來 **限縮**，T的範圍 由**大範圍Object -> 小範圍Properties** [點我看Properties在幹嘛](#Day15)
    ```java=
    //-------------泛型檔-------------
    public class Generics <T extends Properties>{
        T Athing;
        public Test(T Athing){
            this.Athing = Athing;
        }
        
        public void close(){
            Athing.close();
        }
    }
    //-------------主程式-------------
    public class Test{
        public static void main(String[] args){
            Generics<Phone> phone = new Generics<>(
                    new Phone(5, "A1", "0800092000")
                );
            
            phone.close(); // 使用phone物件帶著Phone型態
            System.out.println(phone.getNumber()); 
            // 調用獨有形態，這是多型做不到的
        }
    }
    ```
    - 當然，有繼承就有實作，因此在Generics當中也可以使用Implements，不過在<>當中一樣是用extends來當作 **實作**，因此我們會把 **繼承** 放在最前面，實作放在後面，以 `&` 做分隔
    ```java=
    public class Generics <T extends Properties & aInterface>
    ```
- 泛型跟多型到底差在哪？ ⭐⭐⭐⭐⭐⭐⭐⭐⭐
    - 簡單來說，泛型主要注重 **對內型別的精確度**，因此注重**型別本身** 的 **獨有method**。
    - 而多型注重 **對外一致性**，因此使用的是 **子類** 的 **共同method** (也就是父類的protected method)。
- Generics method ⭐⭐⭐⭐⭐⭐⭐⭐
    - 它有點多型的影子，但就如同上面所說，它還是注重在型別本身。
    ```java=
    public static <T, K> void print(T Athing, K Bthing){
        Athing.print();
        Bthing.print();
    } 
    ```
    - Wildcard關鍵字 : `?`
        - 使用List來理解，當我們想print出List時，需要利用 `?` 來作為 **未知型別** 的符號。
        - 同時我們也可以對wildcard做 **限縮** 範圍的動作。
        ```java=
        public static void main(String[] args){
            List<Integer> intList = new ArrayList<>();
            intList.add(5);
            printA(intList);
            
            List<Phone> phoneList = new ArrayList<>();
            phoneList.add(new Phone(5, "A1", "000"));
            printB(phoneList);
        }
        
        // 原本長這樣
        private static void print(List<Object> o){
            System.out.println(o);
        }

        // 雖然Integer是繼承自Object
        // 但如果是List<Object> & List<Integer>就沒有繼承關係了！
        // 因此要用 ? 來作為未知型別
        private static void printA(List<?> o){
            System.out.println(o);
        }
    
        // wildcard繼承型別
        private static void printB(List<? extends Properties> p){
            System.out.println(p);
        }
        ```

## Day28
#### 學習重點 : Generics泛型 - 3、Varargs用法、Type Erasure
- LinkedList泛型實作 ⭐⭐⭐⭐
    - 我把前幾天一直在做的LinkedList加上了泛型用法，這樣就更還原原版LinkedList了！ [看我的LinkedList實作](##Day23)
    - 在其中我還學到了利用varargs(可變參數)的用法！
    #### 原本我是這樣做
    ```java=
    public class LinkedListT<T>{
        
        private static class Node<T>{
            Node<T> next, prev;
            T data;
            Node(T data){
                this.next = this.prev = null;
                this.data = data;
            }
        }
        
        public LinkedListT(T[] array){
            // ...省略
            for (T data : array){
                // ...省略
            }
            // ...省略
        }
    }
    ```
    - 這樣在主程式中只能這樣宣告
    ```java=
    public static void main(String[] args){
        Integer[] intList = {1,2,3,4};
        LinkedListT<Integer> test = new LinkedListT<>(intList);
    }
    ```
    #### 加上varargs之後
    ```java=
    public LinkedListT(T... array){
        // ...省略
    }
    ```
    ```java=
     public static void main(String[] args){
        LinkedListT<Integer> test = new LinkedListT<>(1,2,3,4);
    }
    ```

- Varargs到底是甚麼？ ⭐⭐⭐⭐
    - varargs = **Var**iable **Arg**uments，允許多個 **相同型態** 的參數以陣列形式被打包作為單一參數
    - 這樣我們就不用同一種類型要寫一堆同樣的method了！
    ```java=
    // 原本要這樣寫
    int add(int a, int b){
        return a+b;
    }
    int add(int a, int b, int c){
        return a+b+c;
    }
    int add(int a, int b, int c, int d{
        return a+b+c+d;
    }
            
    // 利用varargs可以這樣寫
    int add(int... a){
        int sum = 0;
        for (int i : a){
            sum += i;
        }
        return sum;
    }
    ```

- **Type Erasure** ⭐⭐⭐⭐⭐⭐⭐⭐⭐
    - 由於舊版java沒有泛型的功能，為了讓後期的程式能在舊版JVM當中執行，因此編譯後的class檔就不會有泛型出現，而是一堆Object。
    - **Compile-time**時，泛型會檢查型態是否符合(Type checking)，若我在<>中放入String，但是參數卻寫入Integer，此時就會報錯。
    - **Runtime前(Compile後)**，所謂的Integer、String等等都會被Type Erasure的機制而變成Object，若編譯沒有報錯，成功轉成Object後，java就會在每個利用泛型T的地方自動casting。
    - **Runtime**時，我們就不會看到任何泛型的影子了。

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

## Day167
#### 學習重點 : iterator簡介
- 甚麼是iterator？ ⭐⭐⭐⭐⭐⭐⭐⭐
    - iterator(迭代器)，用於遍歷collection的元素，是 **一個Interface**，開發者會藉由collection實作的Iterator來建立Iterator物件，使用Iterator時 **無需知道集合的結構型態**，只須關注「**元素本身**」及「**下一元素**」。
    - 注意！Iterator遍歷collection時，不是利用offset去找下一index，而是藉由collection實作封裝的Iterator去搜尋下一個node，這也是為甚麼迭代器 **可以** 用在Tree、LinkedList等 **非連續記憶體** 身上。
    - 因此在Java集合框架中，每個集合都有自己實作Iterator的邏輯(有自己的演算邏輯 --> 如 : 該如何找到下一node)。
- Why Iterator？ ⭐⭐⭐⭐
    - 如前一點的第三小點，每個集合都有自己實作Iterator的演算邏輯，因此當我們在遍歷集合時，也可以透過itr.remove()來刪除元素，**不像** 原本用for loop遍歷想嘗試時，會出現 `ConcurrentModificationException` 這個Exception。
    - 原因 : 仔細看集合實作Iterator的邏輯，會發現邏輯都會 **確保遍歷時的集合一致性**，使元素被刪時，可以確保下一指向位置正確。

## Day168
#### 學習重點 : iterator實際用法及語法糖
- 實際用法 ⭐⭐⭐⭐⭐
    - 簡單來說，我們會宣告Iterator物件並指派集合的實例給它，如下 : 
    ```java=
    List<Integer> list = new ArrayList<>();
    //...元素新增ing...
    Iterator<Integer> it = list.iterator();
    while (it.hashNext()){
        if (it.next() > 5) it.remove();
    }
    ```
    - 上述是一個簡單的迭代器實際範例，實際上Iterator還有許多用法，我稍微統整了一下 : 
    ```java=
    // 字面上意思 : 自it指向的位置的「下一node」執行Consumer動作
    it.forEachRemaining(System.out::println);

    // 除了基本Iterator介面，還有ListIterator介面(繼承自Iterator)，支援雙向遍歷
    ListIterator<Integer> it = list.listIterator();
    while (it.hashPrevious()) //...
    ```
- for-each語法糖 ⭐⭐⭐⭐⭐
    - 當我們在寫 `for (var s : list)` 時，其實底層原理就是使用了iterator的概念，底層會建立一個localIterator物件去跑每個元素。
    - 但由於我們無法操控底層的localIterator物件，因此也無法做到it.remove這種事，因此for-each的語法糖只適合用於 「**唯讀**」，若要remove，就必須自行建立iterator去跑loop。

## Day169
#### 學習重點 : iterator及Map.Entry
- Map.Entry是甚麼？ ⭐⭐⭐⭐⭐⭐
    - 當我們在建立「HashMap」時，其實是利用一個 **table陣列存Node**，`Node<K,V>` 作為存取鍵值對的物件存進table。
    - 而Entry是一個interface，Node就是實作了Entry的真實物件！因此在HashMap中，實際K、V即存於Node並存於 `Node<K,V>[]` 的陣列當中。
    - 由於Map有多個實作class，因此Map.Entry也有多個實作方式，HashMap選擇用 **Node** 存K、V，TreeMap選擇用 **TreeMap.Entry** 實作 **Map.Entry**。
- Iterator怎麼跟Entry扯上關係？ ⭐⭐⭐⭐⭐⭐
    - 以HashMap為研究對象，在其中，有個類別叫做 `EntrySet`，裡面有個method叫做iterator()，呼叫時，回傳EntryIterator，而這個EntryIterator即**實作了Iterator**，因此透過呼叫iterator()，即可以取得指向table的iterator。
    - 實務上，我們藉由entrySet()取得EntrySet物件（`Set<Map.Entry<K,V>> entrySet`成員）。
    - 實際code展示 : 
    ![螢幕擷取畫面 2026-06-18 180628](https://hackmd.io/_uploads/rkMKmKZfzl.png)

## Day170
#### 學習重點 : Iterable與Collection的關係
- 繼承Iterable ⭐⭐⭐⭐
    - 在Day168時，我有提到for-each語法糖底層是使用了iterator的概念。
    - 但更具體一些，其實for-each在編譯時會分成傳統陣列、Iterable obj派。
        - 若為傳統陣列 --> 則使用一般for loop的語法。
        - 若obj繼承了Iterable --> 則使用Iterator指針去遍歷。
- 那麼到底甚麼是Iterable呢？ ⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐
    - Iterable，如名字所寫，**可迭代**，仔細翻看Collection的介面，其實Collection介面上層就是Iterable介面！
    - 之所以可迭代，是因為Iterable裡面規定了iterator()的method，因此當我們實作了iterable，也就代表必須實作iterator。
    #### 為甚麼不把Iterable跟Iterator合併？
    - 由於我們可以在同一集合類別內建立多個iterator（重複遍歷），若合併了兩者，代表我必須僅能實作一次iterator。
    - 若兩者分離，則能夠先實作iterator，再透過iterator()的抽象method中 `return new ...`，這樣子才能達到解耦、單一職責原則、多路走訪的動作。
    - 用敘述的方式很難解釋ww，還是直接看原始碼好了。
    ![image](https://hackmd.io/_uploads/Hk2bODGGGx.png)
    - 當我每次呼叫iterator()，都會new一個新的Iterator，因此有不同的cursor，使得可多路走訪！

## Day171
#### 學習重點 : Iterable實作
- Range implements iterable ⭐⭐⭐⭐
    - 參考 [Java Iterator](https://www.fooish.com/java/collections/iterator.html) 教學，我跟著實作了一次Iterable。
    ![image](https://hackmd.io/_uploads/BJNv8-VGfe.png)
    - 首先是建立Range被迭代的範圍(start、end)
    ![image](https://hackmd.io/_uploads/Hyf9PZEGGg.png)
    - 再來則是覆寫iterator()方法，這邊直接使用anonymous inner class的寫法！
    - 實際成果 : 
    ![image](https://hackmd.io/_uploads/BJX_iZEMGx.png)
    ![image](https://hackmd.io/_uploads/B1_uoWEGGx.png)

## Day191
#### 學習重點 : Callable搭配Void
- 常見例子
    - 在執行緒當中，我們有時候會搭配Void object來完成一些task。
    ```java=
    import java.util.concurrent.Callable;

    public class VoidT implements Callable<Void> {
        @Override
        public Void call() throws Exception {
            System.out.println("執行中...");
            return null;
        }
    }
    ```
    - 我們知道Void object可以有null Reference，且不能被實例化，因此唯一能回傳的就是null，但其指向的形態還是Void物件。

## Day192
#### 學習重點 : Comparable
- 甚麼是Comparable？ ⭐⭐⭐
    - 在集合框架中，我們時常會對物件進行排序，但排序的規則為何？
    - 這時我們就會實作Comparable介面來定義sort時，是按照什麼樣的邏輯下去做的！
- Comparable實作 ⭐⭐⭐⭐
    - 我們會覆寫一個叫做compareTo的method。
    ```java=
    public class Person implements Comparable<Person> {
        private String name;
        private int age;

        public Person(String name, int age) {
            this.name = name;
            this.age = age;
        }

        @Override
        public int compareTo(Person other) {
            return this.age - other.age;  // 依年齡排序
        }
    }
    ```
    - 會發現其回傳的是int，也就是根據正數負數決定是否要交換兩者順序，按照**自然排序** --> 小的往前排，大的往後。
    - 因此若this大於other則this往後，反之則往前。

## Day193
#### 學習重點 : Comparator
- 甚麼是Comparator？ ⭐⭐⭐⭐
    - 針對一個類別，像是Person，通常會實作compareTo，因此我們可以利用Collection的sort來比較。
    - 但如果我們不想使用類別原生的自然排序呢？
    - 這時我們可以按照我們自己的邏輯來實作比較器！而這個可以讓我們自己設計比較邏輯的工具就是Comparator。
- Comparator怎麼被運用？ ⭐⭐⭐⭐⭐⭐⭐
    - 在使用 `List<String> list` 存放字串時，String內建的compareTo方法是按照ASCII字典順序來排的。
    - 但當我們今天想要將list依照 **字串長度** 來排序呢？
    - 這時就可以使用Comparator介面！
    ```java=
    public static void main(String args[]){
        List<String> list = new ArrayList<>();
        list.sort((s1, s2) -> s1.length() - s2.length())
    }
    ```
    - 上述的 `(s1, s2)...` 就是在實作Comparator介面！

## Day194
#### 學習重點 : Comparator工具
- comparing ⭐⭐⭐⭐
    - 簡單來說，我們可以打包一個排序邏輯工具物件，以Comparator.comparing來生成一個排序物件，可以放進Collections的sort功能中！
    ![image](https://hackmd.io/_uploads/BkKmJdG4zx.png)
- naturalOrder ⭐⭐
    - 當然，Comparator也有naturalOrder的排序方法
    ![image](https://hackmd.io/_uploads/H1fE-dfEMg.png)

## Day195
#### 學習重點 : Collections的功能
- Collections的sort ⭐⭐⭐⭐⭐
    - 在集合框架中，可以使用Collections的sort，其中可以放入單純list（有實作過Comparable介面），或者list搭配comparator的比較器！
    - 像是Person自己有設計過compareTo，但我還是可以塞比較器！
    ![image](https://hackmd.io/_uploads/H1B46oQVfg.png)
    ![image](https://hackmd.io/_uploads/B1Q_6iX4Ge.png)
    - 我順便覆寫了toString讓輸出比較好看！

## Day196
#### 學習重點 : Deque
- 甚麼是Deque？ ⭐⭐⭐⭐⭐
    - 別於一般的queue，deque指的是雙向佇列，他擁有了FIFO及LIFO的特性，亦即push、pop都可以作用於front跟tail。
    #### 實際例子？
    - 一般常用的ctrl+z回溯功能，都是使用stack概念，push操作，pop出top形成回溯，但若一直操作而不回溯，stack就會overflow！
    - 此時若採用deque的概念，一樣push操作，但當操作堆疊達到一定數量後，pop掉tail的操作，此時結合stack可以pop出top(或者說tail)以及queue可以pop出front(或者說bottom)的功能達到最佳回溯功能！
- Java中實際使用deque ⭐⭐⭐⭐⭐⭐⭐
    - Java官方明確表示 : `A more complete and consistent set of LIFO stack operations is provided by the Deque interface and its implementations, which should be used in preference to this class(Stack).`，這也是因為Stack繼承了Vector，使得其Stack嚴謹度降低，且Vector是Thread-safe，亦即在單一執行緒下執行vector會有無用的鎖在拉低效能！
    - 而deque是一個介面，開發者可以選擇無鎖的ArrayDeque或者有鎖的LinkedBlockingDeque，這樣的兼容性使得deque優於stack！
    - 而實際的實作如下 : 
    ![image](https://hackmd.io/_uploads/By8qlxrEzx.png)

## Day197
#### 學習重點 : Deque的結構
- 深入Deque ⭐⭐⭐⭐
    - 昨天只有稍微帶到Deque的意義以及使用方法，今天則是來看看Deque在Java的繼承&實作架構！
    ![image](https://hackmd.io/_uploads/ByWrC88NMe.png)
    - 簡單來說，Deque建立在Queue之下，只是其內部在多定義了addFirst(針對front進行push)、pollLast(針對tail進行poll)，這種只有雙向Queue才會出現的功能！
    - 而在實作當中，則是由ArrayDeque與LinkedList為主。
    ![image](https://hackmd.io/_uploads/B1buGwUVfe.png)
- Deque的特色 ⭐⭐⭐⭐
    - ArrayDeque實作不接受null元素，因此若在ArrayDeque當中放入null會拋出NPE。但在LinkedList當中是可以的！但官方還是不建議在Deque中加入null。
    - 再來，其Array長度是可伸縮的，因此沒有固定大小。
    - 最後，Deque與Vector不同在於其不是thread-safe，因此不允許多執行緒的同時取用，也因此ArrayDeque這種實作只適用於單執行緒！

## Day198
#### 學習重點 : Deque作為Stack的應用
- Stack應用 ⭐⭐⭐⭐⭐⭐⭐
    - 在Java官方的建議中，通常會使用Deque作為代替Stack的選項，因此我找了個十分常見的範例 --> 瀏覽歷史紀錄，來做為Deque(Stack)的一種實作途徑。
    - 首先是定義好三個功能 : 訪問、上一頁、下一頁，上一頁與下一頁是一條路，而訪問代表按照過去的路往前開闢另一條路，因此當使用訪問時，下一頁的stack會被清空。
    - 以下是我的實作 : 
    ![image](https://hackmd.io/_uploads/H1hyDcDVGl.png)
    - 然後我寫了個簡單的測試 : 
    ![image](https://hackmd.io/_uploads/Bk6mt9wEze.png)
    - 透過上述實作可以達到以下成果 : 
    ![image](https://hackmd.io/_uploads/r1YDK5P4Ml.png)
