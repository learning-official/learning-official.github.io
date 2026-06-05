---
title: Java學習歷程（Part-1）
tags: [Java]

---

預計學習細項 : 
- [ ] Spring Boot
    - [ ] Tomcat與Servlet
    - [x] Spring Data JPA實作
- [ ] Collection Framework
- [ ] Gradle
- [ ] JavaFX or Swing？
- [ ] return `<Void>`？
- 小專案Part-3預計 :
    - [ ] 高併發Stock超賣問題
    - [ ] Async執行緒優化
    - [ ] Join Fetch解N+1問題
    

**由於筆記限制10萬字元，因此後續部分移到part-2**
### [⭐點我看Java學習歷程Part-2⭐](https://hackmd.io/@learning-official/Java_learning_2)

## 總目錄
- [資料型態 - Day 1~5](##Day1)
- [Class - Day 6~10](##Day5)
- [繼承與多型 - Day 11~15](##Day11)
- [抽象與介面 - Day 16~20](##Day16)
- [List概念與實作 - Day 21~24](##Day21)
- [HashMap - Day 25](##Day25)
- [泛型Generics - Day 26~28](##Day26)
- [Inner Class - Day 29](##Day29)
- [Garbage Collection - Day 30](##Day30)
- [Annotation - Day 31~32](##Day31)
- [Spring Boot - Day 33~62](##Day33)
- [Spring 「使用者資料管控專案Part1」 - Day 63~80](##Day63)
---
- [Multi-Thread - Day 81~90](https://hackmd.io/@learning-official/Java_learning_2#Day81)
- [Serializeable - Day91](https://hackmd.io/@learning-official/Java_learning_2#Day91)
- [Java IO - Day92~100](https://hackmd.io/@learning-official/Java_learning_2#Day92)
- [Exception Handling - Day101~103](https://hackmd.io/@learning-official/Java_learning_2#Day101)
- [HashSet - Day104](https://hackmd.io/@learning-official/Java_learning_2#Day104)
- [Lambda（Method Reference） - Day105~107](https://hackmd.io/@learning-official/Java_learning_2#Day105)
- [Optional淺入淺出 - Day108~110](https://hackmd.io/@learning-official/Java_learning_2#Day108)
- [Spring 「使用者資料管控專案Part2」 - Day111~156](https://hackmd.io/@learning-official/Java_learning_2#Day111)

## 學習雜談
- Day20 : 
    > 老實說，我從來沒有想過可以這麼認真學一樣東西，雖然對大家來說可能很簡單，但對3分鐘熱度的我來說，真的是一大挑戰，不管最後是否有辦法精通Java，我都很滿意了！
- Day33 : 
    > 突然發現，最累人的不是學習，是寫筆記🤣，不過也正是如此我才可以把學習內容真正刻在腦中吧ww 不然通常是學了又忘，忘了又學。
- Day40 : 
    > 逐漸感受到每天學習的痛苦了ww，撐下去啊!!
- Day80 : 
    > 距離上次打心得竟然過去40天了OMG，我覺得我最近變得有點迷惘，倒不是說熱忱不見，而是不知道該怎麼學習了QAQ
- Day144 : 
    > 最近在做小專案Part-2的時候，總有一種體悟 : 為甚麼怎麼做都做不完🤧，做好一部份，就有下一個概念要學，做個系統到底要學多少技術rrrrr。
    
**阿對了！如果你是用手機版看的話，版面會亂掉ww，請見諒**
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
    ![image](https://hackmd.io/_uploads/HJYML_CSbl.png)
             
        - 所謂單指向就是指每個node(節點)存有下個Data的address pointer
    - 雙指向(Doubly LinkedList, DLL)
    ![image](https://hackmd.io/_uploads/Hy2PwO0H-g.png)
            
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
    ![image](https://hackmd.io/_uploads/BySG3_0BZx.png)
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
    ![image](https://hackmd.io/_uploads/B1byc8lwWe.png)

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
    ![image](https://hackmd.io/_uploads/B1vMS5gwZl.png)

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
    ![image](https://hackmd.io/_uploads/HJpQTFmvbg.png)
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
    ![image](https://hackmd.io/_uploads/HyHpKEsvWx.png)
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
         ![image](https://hackmd.io/_uploads/Byo5vmJdbx.png)
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
    ![image](https://hackmd.io/_uploads/rybdNiOdbl.png)
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
    ![image](https://hackmd.io/_uploads/rysyWZnuZl.png)
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
### [⭐點我看Java學習歷程Part-2⭐](https://hackmd.io/@learning-official/Java_learning_2)預計學習細項 : 
- [ ] Spring Boot
    - [ ] Tomcat與Servlet
    - [x] Spring Data JPA實作
- [ ] Collection Framework
- [ ] Gradle
- [ ] JavaFX or Swing？
- [ ] return `<Void>`？
- 小專案Part-3預計 :
    - [ ] 高併發Stock超賣問題
    - [ ] Async執行緒優化
    - [ ] Join Fetch解N+1問題
    

**由於筆記限制10萬字元，因此後續部分移到part-2**
### [⭐點我看Java學習歷程Part-2⭐](https://hackmd.io/@learning-official/Java_learning_2)

## 總目錄
- [資料型態 - Day 1~5](##Day1)
- [Class - Day 6~10](##Day5)
- [繼承與多型 - Day 11~15](##Day11)
- [抽象與介面 - Day 16~20](##Day16)
- [List概念與實作 - Day 21~24](##Day21)
- [HashMap - Day 25](##Day25)
- [泛型Generics - Day 26~28](##Day26)
- [Inner Class - Day 29](##Day29)
- [Garbage Collection - Day 30](##Day30)
- [Annotation - Day 31~32](##Day31)
- [Spring Boot - Day 33~62](##Day33)
- [Spring 「使用者資料管控專案Part1」 - Day 63~80](##Day63)
---
- [Multi-Thread - Day 81~90](https://hackmd.io/@learning-official/Java_learning_2#Day81)
- [Serializeable - Day91](https://hackmd.io/@learning-official/Java_learning_2#Day91)
- [Java IO - Day92~100](https://hackmd.io/@learning-official/Java_learning_2#Day92)
- [Exception Handling - Day101~103](https://hackmd.io/@learning-official/Java_learning_2#Day101)
- [HashSet - Day104](https://hackmd.io/@learning-official/Java_learning_2#Day104)
- [Lambda（Method Reference） - Day105~107](https://hackmd.io/@learning-official/Java_learning_2#Day105)
- [Optional淺入淺出 - Day108~110](https://hackmd.io/@learning-official/Java_learning_2#Day108)
- [Spring 「使用者資料管控專案Part2」 - Day111~156](https://hackmd.io/@learning-official/Java_learning_2#Day111)

## 學習雜談
- Day20 : 
    > 老實說，我從來沒有想過可以這麼認真學一樣東西，雖然對大家來說可能很簡單，但對3分鐘熱度的我來說，真的是一大挑戰，不管最後是否有辦法精通Java，我都很滿意了！
- Day33 : 
    > 突然發現，最累人的不是學習，是寫筆記🤣，不過也正是如此我才可以把學習內容真正刻在腦中吧ww 不然通常是學了又忘，忘了又學。
- Day40 : 
    > 逐漸感受到每天學習的痛苦了ww，撐下去啊!!
- Day80 : 
    > 距離上次打心得竟然過去40天了OMG，我覺得我最近變得有點迷惘，倒不是說熱忱不見，而是不知道該怎麼學習了QAQ
- Day144 : 
    > 最近在做小專案Part-2的時候，總有一種體悟 : 為甚麼怎麼做都做不完🤧，做好一部份，就有下一個概念要學，做個系統到底要學多少技術rrrrr。
    
**阿對了！如果你是用手機版看的話，版面會亂掉ww，請見諒**
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
    ![image](https://hackmd.io/_uploads/HJYML_CSbl.png)
             
        - 所謂單指向就是指每個node(節點)存有下個Data的address pointer
    - 雙指向(Doubly LinkedList, DLL)
    ![image](https://hackmd.io/_uploads/Hy2PwO0H-g.png)
            
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
    ![image](https://hackmd.io/_uploads/BySG3_0BZx.png)
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
    ![image](https://hackmd.io/_uploads/B1byc8lwWe.png)

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
    ![image](https://hackmd.io/_uploads/B1vMS5gwZl.png)

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
    ![image](https://hackmd.io/_uploads/HJpQTFmvbg.png)
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
    ![image](https://hackmd.io/_uploads/HyHpKEsvWx.png)
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
         ![image](https://hackmd.io/_uploads/Byo5vmJdbx.png)
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
    ![image](https://hackmd.io/_uploads/rybdNiOdbl.png)
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
    ![image](https://hackmd.io/_uploads/rysyWZnuZl.png)
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