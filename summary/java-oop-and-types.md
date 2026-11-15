# OOP、資料型態、Enum 與 Object 合約

## Day1
#### 學習重點 : Compile
- Terminal的編譯方法 ⭐
    - `javac 檔名.java` -> 編譯出class檔
    - `java 檔名` -> 執行main函式內的程式
- 基本class架構
    ```java=
    public class Compile{
        public static void main(String[] args){
            System.out.println("Hello World!");
            System.out.println("YEAH");
        }
    }
    ```
- 變數使用規則 ⭐⭐⭐
    - 當印整數時，default是int，因此若輸入的 **整數過大** 需要使用到Long時，需要在數字後面加上L。
    ```java=
    public class Variable {
        public static void main(String[] args){
            System.out.println(3); 
            System.out.println(300000000000L);
        }
    }
    ```
    - 當印浮點數時，有雙精度Double與單精度Float，Float只會 **取到第7位** ，因此加上F後會自動四捨五入到小數點後第7位
    ```java=
    public class Variable {
        public static void main(String[] args){
            System.out.println(3.14159268F); //輸出3.1415927
        }
    }
    ```

## Day2
#### 學習重點 : DataType
- 資料型態轉換 ⭐⭐⭐
    - 資料型態 **數字range** 比較 : double > float > long > int > short > byte
    ```java=
    // 小範圍 -> 大範圍
    byte a1 = 3;
    short a2 = a1;
    int a3 = a2;

    long a4 = 12345; //雖然指派時12345是int，但會自動轉型成宣告(long)的型態
    float a5 = a4; //這也行的通

    // 大範圍 -> 小範圍
    int b1 = 3;
    byte b2 = b1; // 錯誤!

    double b3 = 3.5;
    float b4 = b3; //錯誤!

    float b5 = 10.5; //預設10.5是double -> 因此會錯誤!
    ```
    - **強制轉換** : 有可能失真
    ```java=
    int c1 = 3;
    byte c2 = (byte) c1;

    int c3 = 128;
    byte c4 = (byte) c3; //由於128超過byte (-127~127) 因此會失真
    // 原理是將大範圍轉成二進位，去掉高位元，直到成為小範圍的二進位長度
    
    double c5 = 3.1415926536;
    float c6 = (float) c5; //失真!
    ```
    
    - **字串轉數字** : parse用法
    ```java=
    String s1 = "123";
    int d1 = Integer.parseInt(s1);
    
    String s2 = "10000000000";
    long d2 = Long.parseLong(s2);

    String s3 = "3.14";
    float d3 = Float.parseFloat(s3);
    
    String s4 = "3.1415926";
    double d4 = Double.parseDouble(s4);
    ```
    - **數字轉字串** : valueOf用法
    ```java=
    String s1 = String.valueOf(123);
    String s2 = String.valueOf(555555L);
    String s3 = String.valueOf(3.14F);
    String s4 = String.valueOf(3.1415926);    
    ```

## Day3
#### 學習重點 : Operator
- 運算子介紹 ⭐⭐
    - 有些東西很熟就不放了，放一些有趣的~
    ```java=
    public class Operator{
        public static void main(String[] args){

            int x = 5/2; // 左右皆為int型態 -> int商數(2)
            double y = 5%3.5; // 3.5為double型態 -> double餘數(1.5)

            System.out.println(x); // 印出2
            System.out.println(y); // 印出1.5

            boolean z = "Hello" == "Hello"; // String也可以比較
            System.out.println(z); // 印出true

            int p = 1;
            p++; // 後置遞增(Postincrement) 先回傳後加1
            ++p; //前置遞增(Preincrement) 先加1後回傳
        }
    }
    ```
- 判斷式 ⭐⭐
    - switch : 以switch中的**變數為基準與case比較**，像if (x == 5){} 這樣的感覺。
    - **break很重要**，若沒有break，程式會以case成功的那行開始，後面的case都會忽略條件直接執行，直到遇到break或者switch底部。
    ```java=
    int x = 5;
    swtich (x){
        case 1:
            System.out.println("Yes 1");
            break;
        case 2:
            System.out.println("Yes 2");
            break;
        case 3:  
            System.out.println("Yes 3");
            break;
        case 4:
            System.out.println("Yes 4");
            break;
        case 5:
            System.out.println("Yes 5");
            break;
        // 跟else很像
        default:
            Systen.out.println("Not in the list!");
            break;
    }
    ```
- Standard Input (標準輸入) ⭐⭐
    - Scanner用法 : 注意陷阱，close之後，程式就會停止輸入功能，因此建議整體程式只有一個Scanner物件，並在程式最後在關閉即可。
    ```java=
    import java.util.Scanner;

    public class Comparison {
        public static void main(String[] args){
            
            // new一個物件，指向Scanner記憶體位置
            Scanner s = new Scanner(System.in); 
            // 取得整數輸入
            int x = s.nextInt();
            System.out.println(x);
            // 取得字串輸入
            String y = s.next();
            System.out.println(y);
            // 關閉輸入功能
            s.close();
        }
    }

    ```

## Day4
#### 學習重點 : Loop
- 迴圈continue & break ⭐⭐
    ```java=
    for (int i=1; i<5; i++){
        if (i%2==0){
            continue; // continue回直接回到for的判斷式當中，而不進行當前判斷式的後續程式
        }
        System.out.println(i);
    }  
    
    int j = 0;
    while (true){
        System.out.println(j++); //先印出j後加1
        if (j>5){
            break; // 跳出迴圈
        } 
    }
    ```
#### Zerojudge : for + switch新手訓練
- 題目 : 利用a值判斷b&c做加減乘除，前面先輸入運算多少組數對。
    ```java=
    import java.util.Scanner;

    public class Traning_loop {
        public static void main(String[] args){

            Scanner s = new Scanner(System.in);
            int count = s.nextInt();
            for (int i=0; i<count; i++){

                int a = s.nextInt();
                long b = s.nextInt();
                long c = s.nextInt();

                switch (a) {
                    case 1:
                        System.out.printf("%d\n", b+c);
                        break;
                    case 2:
                        System.out.printf("%d\n", b-c);
                        break;
                    case 3:
                        System.out.printf("%d\n", b*c);
                        break;
                    case 4:
                        System.out.printf("%d\n", b/c);
                        break;
                    default:
                        break;
                }
            }
            s.close();
        }
    }
    ```
    - 新知識 : %d可以代表整數變數的位置，用printf來做格式化輸出，其他**還有%s、%c、%f**...。
    - 感想 : 測資一直沒過，還以為要用long型態，結果只是忘記格式化輸出ww

## Day5
#### 學習重點 : Array
- 陣列 ⭐⭐⭐⭐
    - 初始化 : 宣告array的資料型態，並給予長度，利用**new語法**
    - 初始化並沒有給予值時，自動填入0。
    ```java=
    int[] x = new int[4]; // 沒有給予值狀況
    double[] y = new double[5];    

    int[] z = new int[]{1,2,3} // 給予值時，中括號不要寫長度
    ```
    - 取得長度 **.length**
    ```java=
    int[] a = new int[3];
    System.out.println(a.length);
    ```

#### Zerojudge : Array矩陣翻轉
- 題目 : 給予一個矩陣，求其翻轉後的形式
    ```java=
    import java.util.Scanner;
    public class Traning_array {
        public static void main(String[] args){

            Scanner s = new Scanner(System.in);

            for (int round=0; round<4; round++){
                int row = s.nextInt();
                int col = s.nextInt();

                int[][] origin = new int[row][col];
                int[][] trans = new int[col][row];

                for (int i=0; i<row; i++){
                    for (int j=0; j<col; j++){
                        origin[i][j] = s.nextInt(); 
                    }
                }

                for (int j=0; j<col; j++){
                    for (int i=0; i<row; i++){
                        trans[j][i] = origin[i][j];
                    }
                }

                for (int a=0; a<col; a++){
                    for (int b=0; b<row; b++){
                        System.out.printf("%d ", trans[a][b]);
                    }
                    System.out.println();
                }
            }
            s.close();
        }
    }
    ```
    - 解題思路 : 簡單來說我先用一個origin存原本的矩陣，然後把row跟col倒反，接者再用 **倒反的迴圈** 跑origin，再存到trans裡面之後再印出來。
    - 感想 : 程式是可以正常運作了，但是總感覺能再精進，而且還被測資搞🤣，後來看到是總共有**四組矩陣**，所以我就寫for in 4跑四次才通過測試。

## Day6
#### 學習重點 : Class的attribute

- 編譯 ⭐⭐⭐
    - 程式在編譯時，找尋的是.java檔案當中的class，可以有很多class
    ```java=
    // 假設在Test.java當中有兩個class
    class Test1{

    }

    class Test2{

    }
    ```
    - 若使用javac Test.java -> 會編譯出Test1.class 跟 Test2.class
- 執行
    - 當編譯完後，利用 `java Test1` 執行時，程式會去Test1尋找 **main方法** 作為 **入口** 開始執行，因此若要讓class能夠執行，必須在其中加入main。
- static : 靜態語句 ⭐⭐⭐⭐⭐
    - 當我加入static後，static成員會在 **載入時(執行前)** 與程式碼一起被放進記憶體空間當中，而不像一般成員要等到執行時，才會去跟記憶體要空間。
    - 簡單來說 : static就是**未雨綢繆**，先放起來等著被呼叫。
    ```java=
    class Test1{
        
        static int a = 0;
        
        public static void main(String[] args){
            System.out.println(a); // 存取本身類別的成員
            Test2.text = "HAEY"; // 修改其他類別的static成員
            Ssytem.out.println(Text2.text); // 存取其他類別內的static成員
        }
    }

    class Test2{
        static String text = "YEAH";
    }
    ```

## Day7
#### 學習重點 : Class的void、return value
- void基本形式 ⭐⭐⭐⭐
    - 這邊再補充一下，static加入後，可以讓成員本身變成 **class的屬性**，因此不需要instance就可以直接呼叫。
    ```java=
    class Test{
        
        public static void main(String[] args){
            Test.print("Hello World!"); // 直接呼叫
        }
        
        static void print(String msg){
            System.out.println(msg);
        }
        
    }
    ```
- return ⭐⭐⭐
    - 當我定義一個函式是 `static int add(){}` 類似這種形式的時候，就會需要回傳值，因為方法的型態不是void(無)，而是int(整數)。
    ```java=
    class Test{
        static int add(int a, int b);
            return a+b; // 回傳a+b的值(int)
        }
    }
    ```
    - 特別的是 : void雖然不須回傳就能運作，但也可以加入return作為 **中斷程式** 的一種方式。
    ```java=
    class Test{
        static void print(String msg){
            if (msg == ""){
                return; // 若參數的msg為空字串，就直接中斷
            }else{
                System.out.println(msg); // 不然就印出msg
            }
        }
    }
    ```

## Day8
#### 學習重點 : Package & public用法、class分類
- class分類 ⭐⭐⭐
    - 當有許多class在同一檔案中做不同事情，可以將其拆分成許多.java檔案
    - 分類後，**class與檔名須相同**，才能於主程式調用

- 封包package概念 ⭐⭐⭐
    - 如果檔案的階層長這樣 -> 
    
        > 主程式.java
          math
        > >  Add.java
        > >  Multiply.java
    - 那麼在 `Add.java` 以及 `Multiply.java` 中就要加入 `package math;` 這樣的宣告
    ```java=
    package math;
    ```
    -  這樣表示 add 跟 multiply 是**屬於** math 這個 **封包** 當中
    -  如果封包當中還有封包，則需要利用 `package math.geometry;` 像這樣子的宣告。

- public關鍵字 ⭐⭐⭐⭐
    - 若我單純寫一般形式的函式，這樣是無法被 **封包外的檔案** 調用
    ```java=
    class Add{
        static int add(int a, int b){
            return a+b;
        }
    }
    ```
    - 加入public -> 使其能夠被封包外部檔案取用
    ```java=
    public class Add{
        public static int add(int a, int b){
            return a+b;
        }
    }
    ```
- import關鍵字 ⭐⭐
    - import可以方便我們在調用封包程式時，不用重複打封包名稱
    - 利用剛剛的math封包，如果要在主程式調用，原本要這樣 ->
    ```java=
    public static void main(String[] args){
        System.out.println(math.Add.add(1,2));
    }
    ```
    - 修改後變成 ->
    ```java=
    import math.Add;
    public static void main(String[] args){
        System.out.println(Add.add(1,2));
    }
    ```
    - 看起來雖然沒什麼變🤣，但如果封包有很多層可能就會變成 `app.arithmetic.basic......`，這樣重複打超級麻煩!
    - 若一個封包有多個 `.java` 檔案，可以使用 `import math.*`，直接全部引入。

## Day9
#### 學習重點 : (non-static) 物件、Constructor
- 物件(object) v.s 類別(class) ⭐⭐⭐⭐⭐⭐
    - 簡單來說，類別是 **藍圖**，物件是 **實體**，藍圖只有一張，但實體**可以有很多個**。
    - 而藍圖中有固定的設定，像是手機藍圖 - 長寬高是固定的，但**顏色**有很多種，這樣我們可以設定 **長寬高為類別屬性(static)**，而 **顏色可以設定為物件屬性(non-static)**

- new關鍵字 ⭐⭐⭐⭐⭐
    - 藍圖到製作成實體的過程稱為 **建構**。
    - 而new這個字就是 **建構**，因此我們要將藍圖做成物件，必須透過new
    - new出一個物件後，要找地方存放，那當然就是像int、double一樣，**類別本身也是一種資料型態**，因此可以宣告該類別的變數。
    ```java=
    Cellphone c = new Cellphone();
    // 類別宣告 變數名稱 = 建構 藍圖();
    ```
- this關鍵字 ⭐⭐⭐
    - 在指稱物件時，可以利用this來作為物件的代名詞
    ```java=
    public class Player{
        public String name;
        public void setName(String name){
            this.name = name;
        }
    }
    ```
    - 也就是當我new一個新的物件出來後，在使用setName這個方法時，我會取得 **該物件** 的name名稱，替換成 **傳進來的name**。
    - 雖然一般來說會把參數的變數跟成員變數的名稱故意錯開，但this的出現可以保證電腦認得 **左邊是物件的name而右邊是參數的name**。
    
- Constructor ⭐⭐⭐
    - 它本質上是一種方法，在 **建立物件** 的時候會自動呼叫。
    - 若沒有定義，Java會自動建立**deafult constructor(無參數無內容的建構式)**
    - 名稱必須跟class相同
    - 可以有參數，若有參數建構式，則不會自動建立deafult constructor
    ```java=
    public class Cellphone{
        
        // 藍圖屬性(static) -> 賦值
        public static double length = 11.5;
        public static double width = 5.5;
        public static double height = 1.2;
        
        // 物件屬性(non-static) -> 可以先有default或者在建構式當中再default
        public String color;
        
        // 初始建構式 > 自動呼叫
        public Cellphone(){}
        
        // 建構式(有參數版本)
        public Cellphone(String color){
            this.color = color;            
        }
        
        // 類別的方法
        public static void getSize(){
            System.out.println("Length: " + length);
            System.out.println("Width: " + width);
            System.out.println("Height: " + height);
        }
        
        // 物件的方法
        public void getColor(){
            System.out.println(this.color);
        }
    }
    ```

## Day10
#### 學習重點 : Access Modifiers
- Modifiers  ⭐⭐⭐⭐⭐⭐
    - protected Modifier
        - 子類權限 : 用於 **繼承** 中，只能供子類繼承以及封包內使用。
    - private Modifier
        - 私人權限 : 只能在 **類別內** 被調用
    - default Modifier
        - 預設權限 : 只能給 **封包內** 的的類別調用
        - 一般來說，通常會將 **同類型** 的類別放在同封包中，在封包當中創造一個 **對外接口 (使用public宣告) 檔案**，其餘可設定成default來提供對外接口檔案做調用。
    - public Modifier
        - 公開權限 : 可以被 **封包外** 的類別調用
        - **建構式** 通常會設成public，才能在外部建構物件。
    
    ```java=
    class Test{
        // 私人
        private int x = 5;
        // 預設
        int y = 6;
        // 公開
        public z = 7;
    }
    ```
- Getter / Setter ⭐⭐⭐⭐
    - 利用getter & setter來代替單純 `object.variable` 這種寫法
    - 這樣才能達到class **封裝** 的效果 (管理哪些成員能夠被外部取得or更改)
    ```java=
    class Test{
        
        private int x = 5;
        private int y = 6;
        
        // getter
        public int getX(){
            return this.x; // x可利用getX取得
        }
        
        public int getY(){
            return this.y; // y也一樣
        }
        
        // setter
        public void setX(int x){
            if (x>0){
                this.x = x; // 當x>0時(條件)，x可被更改
            }
        }
    }
    ```

## Day11
#### 學習重點 : Inheritance
- 繼承意義 ⭐⭐⭐
    - 繼承通常被用在 **事物有相同性質** 時的場景，比如說，手機跟平板都有 **螢幕、產品型號...等等**，我們可以將這些相同性質設成一個類別-> Properties，再由Phone、Tablet去 **繼承**，這樣就不用在Phone跟Tablet寫重複的基本性質宣告了。
- extends關鍵字 ⭐⭐⭐⭐⭐
    - 藉由剛剛的Properties以及Phone跟Tablet，我們說Properties是父類，而Phone跟Tablet是子類，子類繼承父類 ->
    ```java=
    public class Properties{ // 父類定義
        public int size = 0;
        public String model = "WHAT";
    }
    ```
    ```java=
    public class Phone extends Properties{ // 子類 extends 父類
        
        public int number; // 子類獨有性質
        public void getSize(){
            System.out.println(this.size); // 調用父類性質
        }
    }    
    ```
#### Zerojudge : 空間切割
- 這題是給定切割次數 -> 找出可切出幾個空間
    - 這題利用 f(0) = 1、f(1) = 2、f(2) = 4、f(3) = 8 再利用牛頓差值法找出均差來。
    ```java=
    import java.util.Scanner;

    public class Space{
        public static void main(String[] args){
            Scanner s = new Scanner(System.in);

            while(s.hasNextInt()){
                int n = s.nextInt();
                int result = (n*n*n+5*n+6)/6;
                System.out.println(result);
            }
            s.close();
        }
    }

    ```

## Day12
#### 學習重點 : 繼承中的建構式、super調用
- super 關鍵字 ⭐⭐⭐⭐
    - 在繼承中，子類物件形成時會**先呼叫父類的建構式** (很合理嘛!要先有父親才有兒子)，此時Java會在子類建構式中 **自動插入** `super()`(無參數) 來做為 **呼叫父類建構式** 的橋樑。
    - 當父類的建構式朋友們中有**帶參數的建構式時**，我們就得 **自己寫super(參數)**，在括號中加入父類建構式需要的參數。
    ```java=
    // 父類
    public class Father {
        public String name;
        public Father(String name){
            this.name = name;
        }

        public void print(){
            System.out.println(this.name);
        }
    }
    ```
    -  在子類建構式中，記得要 **帶有父類要求的name**，後面甚麼int num之類的才是子類自己本身的!
    ```java=
    // 子類
    public class Son extends Father {
        public int number; 
        public Son(String name, int num){ 
            super(name); // 利用super作為呼叫父類參數建構式
            this.number = num;
        }
    }
    ```
    ```java=
    // 主程式
    public class Inheritance {
        public static void main(String[] args){
            Son s = new Son("Bob", 15);
            s.print();
        }
    }
    ```
    - 結語 : 總結來說，super讓子類在建構時，能夠遵守父類前，子類後的特性啦~

## Day13
#### 學習重點 : 繼承中的物件轉換 -> Up/Down Casting
- Private成員可以被繼承嗎? : ⭐⭐⭐⭐
    - 我今天在學轉換時有個疑問 : 子類繼承時會包含父類的private成員嗎? 
    - 答案是 : **會的**，儘管無法調用，但繼承這個動作本質上就是把父類的東西copy給子類，就像我爸給我一個寶箱，但不給我鑰匙，我雖然無法打開，但內容物還是我的
    
- 子類轉父類 ⭐⭐
    - 本質上，子類擁有父類所有東西，因此在轉換時，不需過多於語法
    ```java=
    class Inheritance{
        public static void main(String[] args){
            Son s = new Son("Bob", 5);
            Father f = s; // 直接把Son型態變數s 指派給 Father型態f
            
            // 或者直接寫
            Father f = new Son("Bob", 5);
        }
    }
    ```
- 父類轉子類 ⭐⭐⭐⭐⭐
    - 需要 **明確標示** 子類名稱 (因為一個父類可能有很多個子類)
    - 情境一 : **原本** 就是**父類**的物件直接轉子類物件 ❌
    ```java=
    Father f = new Father("Bob");
    Son s = (Son) f; // 失敗!
    ```
    - 情境二 : **原本** 就是**子類**的物件，只是先轉成父類物件後，再轉回子類物件 ✅
        - 注意! 當子類轉成父類後，子類獨有的東西都還在，只是到父類後不能存取。
    ```java=
    Father f = new Son("Bob", 5); // 原本是Son物件 -> 轉成Father
    Son s = (Son) f; // 成功! 這邊把Father f 轉回 Son s
    ```

- instanceof 關鍵字　⭐⭐⭐⭐
    - 它用於判斷物件是否屬於某類別或者其子類，會傳布林值
    ```java=
    Father f = new Son("Bob", 5); // 轉型成父類

    System.out.println(f instanceof Son); // 回傳true!
    // 雖然f是Father型態，但它本身還是帶有Son獨有成員(只是不能被存取而已)

    System.out.println(f instanceof Father); //　回傳true!
    ```

## Day14
#### 學習重點 : Protected Modifiers、Overriding
- protected 關鍵字 ⭐⭐⭐⭐
    - protected在Class繼承封裝當中扮演很重要的角色。
        - 回顧Day11的Properties，單靠其屬性及建構式做出來的物件 **並沒有意義**
        - **需要** Phone跟Tablet等 **實際類別** 去繼承並做出物件，此時Properties就可使用protected僅開放給子類調用。

- Overriding子類覆寫 ⭐⭐⭐⭐⭐
    - Overriding出現在子類跟父類有 **相同method且參數一樣** 時。
    - 可見性 : 當父類的 `A method` 是 **private** 時，因子類不可見 -> 喪失覆寫權。
    - 加入 `@Override` 在方法上面，像保險絲，編譯時會幫你找有沒有覆寫上的錯誤。
    ```java=
    // 父類
    public class Father{
        protected String name; // 使用protected僅供子類調用
        protected Father(String name){ // 使用protected僅供子類"建構"
            this.name = name;
        }
        public void sayHi(){
            System.out.println("HI" + this.name);
        }
    }
    ```
    ```java=
    // 子類
    public class Son extends Father{
        private int num;
        public Son(String name, int num){
            super(name);
            this.num = num;
        }
        @Override // 標註該void為Overriding
        public void sayHi(){
            System.out.println("HI " + this.name, + this.num);
        }
    }
    ```
    ```java=
    // 主程式
    public class Test{
        public static void main(String[] args){
            Father f = new Son("Bob ", 5);
            // 儘管轉型成父類(本質上還是子類)，因此還是執行子類的覆寫方法
            f.sayHi(); // out -> Hi Bob 5
        }
    }
    ```

## Day15
#### 學習重點 : Polymorphism多型 (Overriding具象化)
- 多型的概念 : ⭐⭐⭐⭐⭐⭐⭐
    - 在Overriding當中，我們知道子類可以對父類的同種方法進行覆寫，但這有甚麼意義呢？
        - 再次回顧Day11的Properties，如果今天Properties中有個 `按關機鍵事件`，在Phone當中可以 **覆寫** 按下去會進入省電畫面，而在Tablet當中會 **覆寫** 進入睡眠畫面。
        - 這時利用 **不同子類型**，有不同樣的動作就稱為 **多型**。
    - 在多型當中，通常會設定method **有父類物件參數**，但傳入子類物件，其運用到的就是 **覆寫+向上轉型**。
    ```java=
    public class Properties{
        public int size;
        public String model;
        protected Properties(int size, String model){
            this.size = size;
            this.model = model;
        }
        
        public void close(){
            System.out.println("已關閉螢幕.");
        }
    }
    ```
    ```java=
    public class Phone extends Properties{
        private String number; // Phone專有屬性
        public Phone(int size, String model, String number){
            super(size, model);
            this.number = number;
        }
        @Override
        public void close(){
            System.out.println("進入省電模式!"); // Phone覆寫
        }
        
        // getter
        public String getNumber(){
            return this.number;
        }
    }
    ```
    ```java=
    public class Tablet extends Properties{
        public Tablet(int size, String model){
            super(size, model);
        }
        @Override
        public void close(){
            System.out.println("進入睡眠模式!"); // Tablet覆寫
        }
    }
    ```
    ```java=
    public class Test{
        public static void main(String[] args){
            Phone phone = new Phone(5, "A","0800092000");
            Tablet tablet = new Tablet(15, "B");
            
            Test.print(phone);
            Test.print(tablet);
        }
        public static void print(Properties p){ // 運用多型
            p.close();
        }
    }
    ```
- 我對多型的疑問 : ⭐⭐⭐⭐
    - Q : 若我在子類添加一個 `private` A方法，在父類添加一個 `public/protected` A方法，在多型試驗當中，會不會因為轉型到父類後，而無法調用子類所覆寫的A呢? (注意! 父類若是private則不能被覆寫)
    - A : 答案是不會！當我呼叫A方法時，其機制就是去尋找這個方法原本所在的類別，就如同我 ==Day13中的private可以被繼承嗎?== 裡面想同的概念，編譯器會去尋找A方法的所在的 **初始記憶體位置(在子類內部，可以調用)，不因轉型而改變！**

## Day16
#### 學習重點 : Abstract抽象
- 抽象化的概念 : ⭐⭐
    - 簡單來說就是父類搞抽象、子類去實踐！爸爸的夢想小孩來實現！
    - 有時候父類別本身不具物件意義，需要被實踐才完整，此時就可以在父類別加上 `public abstract class`。
- 抽象化的**強制力** : ⭐⭐⭐⭐⭐
    - 如果method **必須** 被override才能運作的話，就在父類別的void前面加上abstract，讓子類別 **必須** 寫一個override函式。
    - 這提供程式開發者能夠確實覆寫類別的抽象方法 **(不能被忽略)**
    ```java=
    // 利用之前的Properties來做示範    
    public abstract class Properties{ // 抽象化 -> 本身不能建物件
        public int size;
        public String model;
        protected Properties(int size, String model){
            this.size = size;
            this.model = model;
        }
        
        public abstract void close();
        // 抽象化 -> 不能有{}主體 (因為我們期待它是在子類被實現而不是父類)
    }
    ```
- Q : 非抽象類別可以有抽象方法嗎？如果是抽象類別可以有一般方法嗎？ ⭐⭐⭐⭐⭐
    - A : 不可以，可以。
    - 前者不可以的原因是因為當非抽象方法被建構出來後，編譯器發現 : ㄟ？怎麼有個 **方法是抽象的，阿我要怎麼調用...** -> 很明顯有錯吧，所以行不通。
    - 後者可以的原因是，抽象類別被子類繼承後，不見得每個method都要被覆寫！**因此有些method還是可以交給父類統一做就好**，所以抽象類別有一般方法是可以的！

## Day17
#### 學習重點 : Interface介面型態
- Interface : 子類的實作集合 ⭐⭐⭐⭐⭐
    - Interface提供我們在 **不同類別、不同繼承結構** 中實作相同的method。
    - Abstract則是作用在 **不同類別、同一繼承結構下** 的實作。
    - 舉例來說，我繼承自「人」、小黑繼承自「狗」，我們明顯在 **不同繼承結構** 當中 (先不要說甚麼 : 阿都是哺乳類的w，我只是舉例🫠)，但我們有共通動作「吃」，因此吃這個動作就可以放入Interface作為人類跟狗狗類別去實作。
- Interface 介面寫法 ⭐⭐⭐
    - 它其實跟一般類別很像，只是其 **內部method不需要主體** (跟Abstract很像吧)
    - 不過Interface中也是可以有主體的(加上default或static)，這裡的default不是modifiers，而是 **實作**
    ```java=
    public interface Eat{
        void eat(); // 這裡不是default權限，而是public
        
        default void print(){ // 實作
            System.out.println("Yummy");
        }
        static void say(){ // 實作
            System.out.println("WHAT");
        }
    } 
    ```
- implement 關鍵字 ⭐⭐⭐⭐⭐
    - implement就是實作的意思，它被放在類別名稱後面
    ```java=
    public class Me extends Human implements Eat {
        @Override
        // 必須得使用public，因為interface中都是public
        public void eat(){ // Override
            System.out.println("我是人，我吃了食物");
        }
    }
    ```
- Interface casting ⭐⭐⭐⭐⭐⭐
    - 由於Interface跟一般類別很像，因此 **它也有多型的寫法**，當我跟小黑都 **確實** 實作了Eat介面(亦即 **全部都有覆寫** Eat中的method)，那麼就可以利用多型向上casting成Eat型態 (跟父類的感覺很像吧！)。
    ```java=
    public class Test{
        public static void main(String[] args{
            Me m = new Me();
            Test.getEatStatus(m); // 屬於Human子類、Eat介面型態
        }
        private static void getEatStatus(Eat e){
            e.eat();
        }
    }
    ```
- 總結 : 
    - 介面提供 **不同繼承結構** 中的類別可以實作同樣的動作，因此可以說 -> **繼承提供屬性連結，介面提供方法連結**。
    - 介面讓編譯器只需要認得介面動作，不必要知道子類到底是誰即可運作。

## Day18
#### 學習重點 : final用法
- 概念 : ⭐⭐⭐⭐⭐⭐
    - final又可以說 **唯一性，不能被更動**，很類似於c++中的const，很酷的是它final可以用在class身上。
    - **final class** :
        - class加入final亦即它 **不能被繼承**，這對於開發者來說很有幫助，畢竟有些類別是單獨運作的，不希望該class被拿去亂繼承。
        ```java=
        // 加入final使其無法被繼承
        public final class Test{
            public static void main(String[] args){
                ...略;
            }
        }
        ```
    - **final method**
        - method加入final後，即不能在子類中Override該method，以保持父類class的獨特性。
        ```java=
        public class Father{
            public final void print(){
                System.out.println("不要覆寫我w");
            }
        }
        ```
    - **final variable**
        - variable加入final，最關鍵的就是限制該變數只能被 **指派一次**-> 亦即賦值初始化後即不能再更動！常用在constant中。
        - 不過final物件變數本身還是 **能夠更改其型態中非final的成員**。
        ```java=
        public class Test{
            public static final int PI = 3.1415;
            final Son s = new Son();
            s = new Son(); // 錯誤! 不能再指派
            s.x = 5 // 但是可以更改Son形態中的非final變數
        }
        ```

## Day19
#### 學習重點 : Lambda
- Interface前導 ⭐⭐⭐⭐⭐⭐
    - 我們知道在介面型態中，只需要宣告而不需要實作，若今天Interface中 **只有一個抽象方法** 我們即可利用lambda形式來節省「**創class、implements、overriding**」等過程。
    - 我們稱只含一個抽象方法的Interface為 **functional Interface**，同樣可加上Annotation來幫助編譯器理解。
    ```java=
    //------Interface檔案------- 
    @FunctionalInterface
    public interface InterfaceLambda{
        void print(); // 抽象方法
        default sayHi(){ // 預先實作方法不影響其成為functional interface
            System.out.println("嗨");
        }
    }
    ```
- Lambda基本形式 ⭐⭐⭐⭐⭐⭐⭐⭐
    - 原本創物件寫法
        ```java=
        //--------實作檔案-----------
        public class Lambda implements InterfaceLambda{
            @Override
            public void print(){
                System.out.println("I love Lambda!");
            }
        }
        //---------主檔案------------
        public class Test{
            static void lambdaTest(InterfaceLambda l){
                l.print();
            }
            public static void main(String[] args){
                Lambda l = new Lambda();
                lambdaTest(l);
            }
        }
        ```
    - 利用 `(參數) -> {主體}` 形式
    
        ```java=
        //---------主檔案------------
        public class Test{
            static void lambdaTest(InterfaceLambda l){
                l.print();
            }
            
            public static void main(String[] args){
                
                // 寫法一
                InterfaceLambda l = 
                    () -> System.out.println("I love lambda!"); // 直接實作
                lambdaTest(l);
                
                // 寫法二(最簡短)
                lambdaTest(
                    () -> System.out.println("I love lambda!") // 直接實作
                );
            }
        }
        ```
- Lambda參數形式 ⭐⭐⭐⭐⭐⭐⭐⭐
    - 當今天Interface中的抽象方法帶有參數，或者是非void方法需要這樣寫。
    
        ```java=
        //------Interface檔案------- 
        @FunctionalInterface
        public interface InterfaceLambda{
            String print(String s); // 參數且回傳非void抽象方法
        }
        ```
        ```java=
        //---------主檔案------------
        public class Test{
            static void lambdaTest(InterfaceLambda l, String s){
                l.print(s);
            }
            public static void(String[] args){
                lambdaTest(
                    (s) -> {
                        System.out.print("HI" + s);
                        return s;}
                    , "I love lambda!")
            }
        }
        ```
- 總結 : 關於參數形式還有問題，但我需要休息一下，一天肝完有點難ww，明天繼續！

## Day20
#### 學習重點 : lambda簡化
- 單一簡化 ⭐⭐⭐⭐
    - 參數 & return形式簡寫
    - 單參數 -> **小括號省略**、單行return -> **大括號 & return關鍵字省略**
        ```java=
        //------Interface檔案------- 
        @FunctionalInterface
        public interface InterfaceLambda{
            String str(String s); // 帶有一參數
        }
        ```
        ```java=
        //---------主檔案------------
        public class Test{
            static void lambdaTest(InterfaceLambda l, String s){
                System.out.println(l.str(s));
            }
            public static void(String[] args){
                lambdaTest(
                    s -> "HI " + s, "I love lambda!"
                );
                // (s) 省略為 s
                // {return "HI " + s;} 省略為 "HI " + s
            }
        }
        ```

## Day29
#### 學習重點 : Inner Class
- Inner class用在哪？ ⭐⭐⭐⭐⭐
    - 在前幾天我做Node節點時，就突然想到為甚麼有Inner Class這種東西，(雖然我寫得很理所當然🤣
    - 簡單來說，Inner class就像機器的零組件，獨立出來沒什麼意義，但又是機器的一部分，因此我們會把這種零組件的class寫進去機器class當中。
- Inner class怎麼寫？ ⭐⭐⭐⭐⭐⭐⭐
    - 首先有分`non-static` & `static`，而non-static又有分member、local、anonymous三種。
    #### **non-static** : member inner class
    ```java=
    public class OuterClass {

        public void printOuter(){
            System.out.println("HI this is outer class");
        }
        
        // inner class
        public class InnerClass{
            public void pirntInner(){
                System.out.println("HI this is inner class");
            }
        }
    }
    ```
    - 在外部建立 **non-static** inner class物件時，必須**先**建立outer class物件，再利用 **outer物件** new出 **inner物件**。
    ```java=
    public class InnerTest {
    
        public static void main(String[] args){
            OuterClass outer = new OuterClass();
            // 超神奇的寫法吧 outer.new建構出inner物件
            OuterClass.InnerClass inner = outer.new InnerClass();

            inner.pirntInner();
        }
    }    
    ```
    - 不過一般來說，**non-static**的inner class通常會設成private，因為只是要給內部成員使用，就像Node class一樣。
    #### **non-static** : Local inner class
    - 雖然說這幾乎不會用到，但是還是寫一下好了w
    ```java=
    public class OuterClass{
        public void sayHi(){
            System.out.println("HI");
            
            class LocalInnerClass{
                
                String name = "小八";
                public void sayName(){
                    System.out.println(name);
                }
            }
            
            LocalInnerClass what = new LocalInnerClass();
            waht.sayName();
        }
    }
    ```
    - 簡單來說就是在method當中建立class，這個class生命週期只會在runtime的時候出現而已。
    
    #### **non-static** : anonymous inner class
    - 先說，它就像extneds class或implements interface一樣。
    - 這個還蠻有趣的，它就像是 **單次使用** subClass，我可以建立一個A類別的物件但同時覆寫A類別的方法，有點神奇，讓我們看code！
    ```java=
    public static void main(String[] args){
        
        OuterClass outerAnony = new OuterClass(){
            @Override
            public void sayHi(){
                System.out.println("Hi");
            }
        }
        outerAnony.sayHi(); // 印出Hi
    }
    ```
    - 這就是**單次覆寫**，outerAnony型態上雖是OuterClass，但 `sayHi` method卻有覆寫。
    - 更簡短也可以這樣寫 ->
    ```java=
    new OuterClass(){
        @Override
        public void sayHi(){
            System.out.println("Hi");
        }
    }.sayHi();
    ```
    #### 關於inner class調用local variable
    - 若在method當中宣告的區域變數要被 `local inner class` 調用，需將該變數設final形式。
    - 原因是當我們在local class當中 **return了物件** 且有關local variable的東西，當method結束，變數就消失了，但物件卻留著，就會導致物件找不到變數，發生錯誤，因此 **Java會自動copy** variable到inner class中。
    - 但當我們在method當中又改了該變數，Java會很問號？？要不要再copy一次，因此乾脆 **統一設定成final**。
    
    #### static : member inner class
    - 基本上就是在class前面加上static即可
    ```java=
    public class OuterClass {
        // ...省略
        // static inner class
        public static class InnerClass{
            // ...省略
        }
    }
    ```
    ```java=
    public class InnerTest {
        public static void main(String[] args){
            // 這邊就改成利用Outer.Inner這樣做
           OuterClass.InnerClass inner = new OuterClass.InnerClass();

           inner.pirntInner();
        }
    }    
    ```
- 為何不用Lambda就好？ ⭐⭐⭐⭐⭐⭐⭐
    - 仔細看Lambda跟Anoymous會發現真的好像喔！不過還是有部分差異 : 
        #### lambda
        - 用在**functional interface**中
        - **Only** 覆寫一個method
        - 優點在於語法簡潔且專一性高
        #### Anonymous
        - 用於**class**以及interface中
        - 可以覆寫**一堆method**
        - 優點在於可以實作**複雜的interface**(多個抽象method)

## Day30
#### 學習重點 : Garbage Collection (GC)
- 甚麼是Garbage Collection？ ⭐⭐⭐⭐⭐
    - 簡單來說，它像是程式清潔員，把一些沒在用的實體阿物件阿全部從記憶中釋放，以保持記憶體是乾淨的空間。
    - 它本質上就是一種演算法，利用標記(Mark)、清除(Sweap)在做事。
    - 一般來說GC運作時會**stop-the-world pause**，也就是暫停程式，因此好的GC就是讓STW的時長越短越好。
- 它實際怎麼運作的？ ⭐⭐⭐⭐⭐⭐⭐
    - 當我們在new物件時，都會去跟記憶體要空間，久而久之記憶體就理所當然的被塞爆了嘛！
    - 基於許多物件生命週期都不長的原則，Java採用Generational GC，分出**Young generation**、**Old generation**，還有一個不歸GC管的 **Metaspace**。
    #### Young Generation : 
    - 此區域是**最頻繁**被GC偵測的部分，因為 **物件的出生點** 都在這！ 當GC掃過整個Young Gen的區塊後，沒有reference的物件，或者說沒有被指向的物件，都會被清理掉(Sweap)，而有reference的物件則會被標記起來(Mark)
    - 當物件被偵測過幾次都保有reference，此時GC就會將其**打包送往Old Gen**，讓Young Gen變得更乾淨。
    #### Old Generation : 
    - 這個區域不常被偵測，因為這裡存放的都是從Young Gen存活下來的物件，**始終有reference**，它的出現讓Young Gen不用每次都要掃整個Heap，可以專心看那些新的物件，省下許多時間。
    #### Metaspace : 
    - 它本身是存放固定靜態的東西，像是Class架構、method名稱等不可能變動的東西。
    - 在Java7之前，**它叫做PermGen**，跟著Generation一起在Heap堆中玩樂w，但在Java8之後被踢出Heap區，因為基本上Metaspace的東西**都是固定的**，沒有必要跟Young/Old Gen這種動態區塊放在一起。
    - 它存儲於Native memory區塊，因此能存多少，就看我們電腦RAM有多大(不過通常會給它一個MaxSize，不然等等電腦爆掉ww)

- GC參數設定 ⭐⭐⭐⭐
    - 由於Heap是動態的，因此在執行之前就必須給定參數如 -> Xms初始記憶體、Xmx最大記憶體、Young/Old gen的比例(Ratio)、GC的種類 -XX:+UseG1GC。
    - 當然Metaspace也需要給定MaxSize以防止它占用太多Native memory。
    - Heap如果設的太大，GC清理頻率⬇️，清理時長⬆️，設的太小就是相反。

- GC的種類 ⭐⭐⭐
    - 自Java9後，預設都是採用 **G1GC**，因為它能夠切分Heap區塊，有效清理垃圾。
    - 另外還有Parallel GC、ZGC、CMS GC等，不過等有興趣再來看吧！

## Day31
#### 學習重點 : Annotation
- Annotation是甚麼？ ⭐⭐⭐⭐
    - 在前幾周的學習中，`@Override`、`@FunctionalInterface` 這種東西常常出現，它們都是一種Annotation。
    - 跟一般註解不一樣的是，它們可以對class、method、variable產生作用。
- 基本架構 : ⭐⭐⭐⭐⭐⭐
    - 跟interface很像，只是多加一個 `@` 而已，所以本質上操作跟interface差不多。
    ```java=
    @Retention(RetentionPolicy.RUNTIME) // 作用時機
    @Target(ElementType.TYPE) // 作用對象
    public @interface Annoatation{
    }
    ```
- 作用時機與作用對象 : ⭐⭐⭐⭐⭐
    - annotation可以設定其作用的時機，也同時決定了它的生命週期
    #### 作用時機 : 
    分為三種 `Runtime`、`Class`、`Source`
    - 利用Retention來設定。
    - **Runtime** 就是在執行時還保持作用。
    - **Class** 就是編譯時供編譯器作用，運行時被丟棄。
    - **Source** 在原程式碼中作用，編譯後被丟棄，不被.class保留，像 `@Override`。
    #### 作用對象
    主要有三種 `Class`、`Method`、`Field`
    - 利用ElementType來設定，就單純設定作用對象，應該很好理解w

- 怎麼找annotation ⭐⭐⭐⭐⭐⭐⭐⭐
    - 基本上在做annotation時，我們會想要針對有annotation的class、method、variable做事情，就像 `@Override` 一樣是針對覆寫method去check。
    - 這邊利用class來舉例 : 
        - 要確認class是否有加註annotation，就是在判斷式當中利用
        `物件.getClass().isAnnotationPresent(annotation.class)`
        來取得boolean回傳並做後續動作。
        ```java=
        public static void main(String[] args){
            AnnotationTest cat = new AnnotationTest("Alice");

            if (cat.getClass()
                .isAnnotationPresent(ClassAnnotation.class)
               ){
                System.out.println("This is an annotation class!");
           }
        }
        ```
    - method與invoke ⭐⭐⭐⭐⭐⭐⭐
        - 我們可以針對有annotation的method經判斷來呼叫method。
        ```java=
        try{
            // 利用try catch確保存取的是非私有method
            // 利用Method型別先跑過整個class中的method
            for (Method method : 
                 cat.getClass().getDeclaredMethods()){
                if (method.
                    isAnnotationPresent(MethodAnnotation.class)){    
                        // 放入物件並以該物件呼叫
                        method.invoke(cat);
                    }
                }
        } catch (IllegalAccessException e){
            System.out.println(e.getCause());
        } catch (InvocationTargetException e){
            System.out.println(e.getCause());
        }
        ```
    - Field特殊找法 ⭐⭐⭐⭐⭐
        - 基本上對有Annotation的變數找法跟method一樣，但是在最後會多一個找型別的動作 ->
        ```java=
        try{
            for (Field field : cat.getClass().getDeclaredFields()){
                // 由於field存取的是value
                // 因此利用該value是否instanceof某個type
                // 並存到str中進行動作
                Object obj = field.get(cat);
                if (obj instanceof String str){
                    System.out.println(str);
                }
            }
        } catch (IllegalAccessException e){
            System.out.println(e);
        }
        ```
- Annotation參數設置 ⭐⭐⭐⭐⭐⭐⭐⭐
    - 我們可以在 **annotation設置參數**，但我們知道在**interface當中不能設變數**
    - 因此把它弄成method -> 需要被實作(指派、賦值)，這種method不會有參數，因此可以把它當作一種變數的變形？
    - 當然型態都是primitive或者String等。
    - 當然也能設預設值，利用[default實作 -> 注意不是modifiers](##Day17)，跟interface一樣。
    ```java=
    public @interface Test{
        int times() default 5;
    }
    ```
    - 要加上參數就像 `@annotation(times = 3)` 這樣。
    - 而要取得參數值就是利用(以Method舉例) ->
    ```java=
    for (Method method : cat.getClass().getDeclaredMethods()){
        if (method.
            isAnnotationPresent(MethodAnnotation.class)){
            // 以interface物件來存method上annotation的實作
            MethodAnnotation ma = method.
                getAnnotation(MethodAnnotation.class);
            // 利用time取得來觸發method
            for (int i = 0; i<ma.time(); i++){
                method.invoke(cat);
            }
        }
    }
    ```

## Day32
#### 學習重點 : Annotation實作、reflection反射
- 反射是甚麼？ ⭐⭐⭐⭐⭐
    - 昨天在看Annotation的時候，一直很不理解反射是甚麼神奇的東西，今天決定好好來研究下。
    - 簡單來說，反射就是在 **Runtime** 時還能 **取得Class資訊** 的一個動作。
    - 當我們寫 `物件.getClass().get...` 其實就是以物件所屬類別視角來做一些動作。
    ```java=
    AnnotationTest at = new AnnotationTest();

    // 以物件.getClass() = class視角來看本身
    Method[] methods = at.getClass().getDelaredMethods();

    for (int i=0; i<methods.length; i++){
        System.out.print(methods[i].getName() + " ");
    }
    ```
- Annotation實作 ⭐⭐⭐⭐⭐⭐⭐
    - 由於我實在搞不懂annotation真正的用處在哪，所以請AI給我個例子，我再抓下來實作 ( 可能真的得等我自己開發東西的時候才會很有感吧w
    - 這邊實作一個annotation用來 **確保變數不是null** 的功能！　
    - 想法是這樣的 : 
        - 1️⃣ 建立fieldCheck註解 : 意義 - 使必須被賦值的variable都**不能為null**。
        - 2️⃣ 於fieldCheck中加入**message** : 偵測時可在terminal給予error。
        - 3️⃣ 利用for-loop在checkNull中找到**有annotation**的變數。
        - 4️⃣ 確認變數是否是null，若否則pass，若是則噴出message。
    - **checkNull** 長這樣(好亂ww) : 
        ```java=
        public static void checkNull(Object obj) throws IllegalAccessException{
            for (Field field : obj.getClass().getDeclaredFields()){
                if (field.isAnnotationPresent(FieldCheck.class)){
                    field.setAccessible(true);
                    Object value = field.get(obj);
                    if (value == null){
                        System.out.println("error : " + field.getAnnotation(FieldCheck.class).message());
                    }else{
                        System.out.println("pass : " + field.getName());
                    }
                }
            }
        }
        ```

- Override復刻 - Runtime ⭐⭐⭐⭐⭐⭐⭐
    - 由於Override本身是在**Source時期偵測**，要用到Processor，以我目前的能力應該不太可能w
    - 所以我改成**在Runtime檢查**，實際code太多就不放上來了，我丟到[github](https://github.com/learning-official/JavaLearning/tree/main/Annotation)！

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

## Day176
#### 學習重點 : java.lang的Object之deep copy
- 續看clone ⭐⭐⭐⭐⭐⭐
    - 接續昨天的clone，今天來討論將clone實作成deep copy的形式。
    - 一般使用super.clone()，意即去呼叫父類的clone，此時其實是以子類的身分 : `this指向子類`，到父類去做事，但由於super的關係，使多型暫時失效，轉以父類執行clone。
    - 但由於Object定義clone是以 `native` 定義，亦即使用原生語言(C++)，此時 C++ **找的是this所在的記憶體位置**，因此super多型遮罩在 C++ 不管用，這也是為甚麼明明寫super.clone，卻是去複製自己，而不是複製父類(**原因就在this的歸屬**)。
    - 這邊有找到幾個詞彙，之後可以研究研究 : `Klass、invokevirtual/invokespecial、Field Hiding`
- Deep copy的實作 ⭐⭐⭐
    - 其原理很簡單，使用super.clone()後，還需要針對本身的Wrapper成員再進行一次clone。
    ![image](https://hackmd.io/_uploads/S1TKpjqffl.png)
    - 這邊加了個ObjectCloneTest的Wrapper成員，因此要再針對它再進行一次clone。

## Day177
#### 學習重點 : java.lang的Object之執行緒
- wait/notify/notifyAll ⭐⭐⭐⭐⭐⭐⭐⭐⭐
    - 簡單來說，這三者專門處理Object這個資源的控管！
    - 以搶票系統來看 : 若一坨人要搶A區的票，A區作為一個Object紀錄剩餘票數，若發現票已售完，後續排隊用戶的進入購票環節時會因 `synchronized` 緣故，依序取用A區的資源，若發現沒有票則執行 `(A區)this.wait()`，意即進到A區資源的等待區，並交出自身的對A區的「**資源鎖**」給下一個執行緒。
    - 等有使用者釋票後，就會發出 `A區(this).notify()` 去「喚醒」其中一個在等待區的執行緒。
    - 而notifyAll就只是一次喚醒「所有」等待的執行緒而已。
    - 不過這產生一個問題 : 如果排頭檢查票數發現是0，並到等待區等待，結果第二個人進來時，剛好有票了，第二個人就順勢購票了。
        - 因此synchronized這種monitor lock又被稱為「**Non-fair Lock**」
        - 此時改用ReentrantLock會更加FAIR！！
- 為何wait/notify要在Object中？ ⭐⭐⭐
    - 一般來說，在討論執行緒時，都會以Thread/Runnable等為主，但是為甚麼wait/notify這些東西會出現在Object中呢？
    - 主要原因 : Object凌駕於所有class之上，且任何class都可以是「資源」，若以執行緒(使用者)去呼叫wait，會導致JVM `不知道這個執行緒是要進到哪個資源的等待區？`，這也是為何不會直接以Thread呼叫wait！

## Day178
#### 學習重點 : java.lang的Record
- 基本概念 ⭐⭐⭐⭐
    - 基本上Record在Java中是一個很方便建立model等的關鍵字。
    - 於Java 16引入，非常方便用於設計一些「不可變」的類別。
    - 其簡化了我們寫參數建構子、Getter、覆寫Object功能的過程。
    - 因為是不可變，因此當我們給定成員後，會自動變成final形式(class本身也是final)，且沒有Setter。
    - 由於record類別是去繼承java.lang的Record抽象類別，因此寫record不能再繼承其他類別，也不能被其他類別繼承！
- 實際範例 ⭐⭐⭐
    - 十分簡單易懂ww
    ![image](https://hackmd.io/_uploads/rJZtYWafMe.png)
    - 當然，也可以實作 : 
    ![image](https://hackmd.io/_uploads/SJqYKWpzfx.png)
    - 這邊就用前幾天碰到的Cloneable來示範實作ouo。
    - 但其實簡化成這樣，底層還是會還原成 : 
    ![image](https://hackmd.io/_uploads/Sk45F-Tffx.png)

## Day179
#### 學習重點 : java.lang的Record複製
- record的copy ⭐⭐⭐⭐
    - 今天看個簡單的主題，Record的複製！
    - 一般在class當中，我們可能會寫下一些成員，他們不是被參數指派，隨機產生、根據時間不同而有不同數值。像是下面這個 : 
    ```java=
    public class User {
        private String name;
        private long createTime;

        public User(String name) {
            this.name = name;
            this.createTime = System.currentTimeMillis();
        }    
    }
    ```
    - 此時若我們建構一個新的user，但copy了另一個User的name，此時用equals判斷會不一樣。
    - 但如果我們使用record，由於定義不可變及可預測性，我們只要透過`R newR = new R(oldR.name(), oldR.age()...)` 建構完畢後，newR與oldR做equals必是True。

## Day180
#### 學習重點 : java.lang的Record簡便用法
- 函式內宣告 ⭐⭐⭐
    - 就像匿名類別一樣，Record也可以被簡單實現在函式內部中被「單次」使用。
    - 以下是範例 : 
    ```java=
    public void calculate(List<Merchandise> list) {
        // ...建立List的過程
        record TempTotal(Long id, double finalPrice) {}

        List<TempTotal> result = list.stream()
            .map(m -> new TempTotal(m.getId(), m.getPrice() * 0.8))
            .toList();
    }
    ```
