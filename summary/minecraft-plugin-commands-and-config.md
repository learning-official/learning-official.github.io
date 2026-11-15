# Brigadier 指令樹、TextComponent 與 Config

## Day309
#### 學習重點 : Minecraft Plugin第四課 : 指令設計
- 比對過往指令系統 ⭐⭐
    - 過往在使用spigot時，是去實作 `CommandExecutor`，同時於onEnable註冊指令，最後還要在 `plugin.yml` 定義指令。
    - 而使用Paper，則有兩種方式。
        - 新版的paper提供了 `Brigadier` 的方式來建立指令，簡單來說就是結合Builder+lambda的方式來快速實作
        - 另外一個是較簡單的、與Bukkit/Spigot類似的方式，但「**省去了plugin.yml定義**的步驟」，其餘幾乎一模一樣。
    - paperAPI說明是這樣的 : 「For very simple commands Paper has a way to declare **Bukkit-style** commands by implementing the BasicCommand interface.」，我想說先從與spigot類似的風格來設計會比較熟悉！
- 指令設計 ⭐⭐⭐⭐
    - 首先要先做的就是實作 `BasicCommand`，基本上只需要覆寫 `execute` 即可，而內容與spigot大同小異，可以直接看我設計的成果！
    ![image](https://hackmd.io/_uploads/HytyYoKTMg.png)
    - 由於我將開啟menu的部分獨立出來，因此可以同時在 `RED_BED` 與 `指令系統` 觸發menu的開啟。
- 抽離MenuManager ⭐⭐⭐⭐
    - 原本MenuManager是在Listener被new出來的，但我的宗旨是「所有人共用同一個Manager」，因此我將其改到啟動類別中實例化，所有事件/menu/指令都用同一物件（**不過這似乎會有資源競爭的問題**...emmm）
    - 透過抽離manager，我就可以在指令類別的建構式傳進manager參數去取得啦~

## Day310
#### 學習重點 : Minecraft Plugin第四課 : 指令樹
- 何謂指令樹？ ⭐⭐⭐⭐
    - 我們知道在Minecraft中指令後面可以帶有參數，且每個參數可以對應到更多的參數，因此一個指令可以分出去多個分支，且每個分支還會有分支，如此以來就形成了指令樹。
    - 在過往的指令設計當中，我們一般會偵測 `args[0].equals("...")`，然後做出相對應的動作，但參數一多，等於是要有很多 `if-else` 做包裝，顯得冗餘且複雜。
    - 因此paper提供了一個輕鬆建立指令樹的方式，且對每個參數做包裝，這樣一來就有 **規範、包裝** 的概念，讓每個參數得到一個家的感覺ww（不然原本放在if-else）很容易就可以被更動，不是那麼嚴謹。
- 指令樹如何建立？ ⭐⭐⭐
    - 參考 [PaperMC - Command trees](https://docs.papermc.io/paper/dev/command-api/basics/command-tree/) 的作法，使用LiteralArgumentBuilder先建立一個指令的根結點，也就是斜線後的指令開頭，接著再接上不同的參數後綴。
    ```java=
    LiteralArgumentBuilder<CommandSourceStack> root = Commands.literal("customplugin");
    root.then(Commands.literal("reload"));
    root.then(Commands.literal("tphere"));
    root.then(Commands.literal("killall"));
    ```
    - 接著在其中就可以使用execute執行lambda，因此實作與參數本體綁在一起，最後在一起build上去，整套流程明顯清晰許多啦～
    - 至於實作，明天再來！最近有點忙.w.

## Day311
#### 學習重點 : Minecraft Plugin第四課 : 何謂CommandSourceStack？
- 前言 ⭐⭐
    - 我覺得在實作指令樹及參數前，可以先來了解何謂CommandSourceStack！
    - 這樣對之後寫參數節點時感覺比較清晰一些！
- 何謂CommandSourceStack ⭐⭐⭐⭐⭐
    - 在過去的spigot設計當中，我們會使用CommandSender來設計指令，但這有些問題，也就是我們只能「**取得發送指令的來源**」，但沒辦法知道，來源是否等於執行指令的實體，有可能是「伺服器以玩家身分執行指令」，此時來源就是伺服器，但執行者是玩家。
    - 因此CommandSourceStack作為CommandSender的擴大版，除了內部定義了getSender（指令的根來源是誰），還定義了getExecutor（實際執行實體），甚至是Location、Rotation等定義。
    - 這樣當伺服器使用 `/execute as xiaoba run ...` 時，sender就是console，而executor就是小八我拉～
    - 可以發現，CommandSourceStack就是圍繞著execute這個指令在轉嘛！
- 實際的意義？ ⭐⭐⭐⭐⭐⭐
    - 既然知道CommandSourceStack會將Executor、Sender、Location全部解耦，表示每次執行指令，這些內容物所代表的涵義都會不同。
    - 我實際寫了個 `/test hi`，觸發時會發送hi給 `Sender`，且在終端機印出位置。
    - 當我以終端機發送 `/execute as ... run test hi` 時，會看到以下訊息 : 
    ![image](https://hackmd.io/_uploads/SkWniYh6Ge.png)
    - 若是玩家發送 `/test hi`，則會變成 : 
    ![image](https://hackmd.io/_uploads/rk4hoKhpGx.png)
    - 因此可以發現Location若沒有指定（`at ...`），那就會是整個Stack當初執行的sender位置。
    - 這邊就可以看到其實 `execute` 的指令後綴每一步都是在「設定」，無論是 `as ...`（設定Executor），`at ...`（設定Location）都是這樣的！

## Day312
#### 學習重點 : Minecraft Plugin第四課 : 指令樹實作
- 要怎麼實作？ ⭐⭐⭐
    - 首先要先分清楚 `LiteralArgumentBuilder` 跟 `LiteralCommandNode` 的協作關係。
        - 前者是建立樹的過程，而後者是樹本人。
    - 因此我們會利用 `LiteralArgumentBuilder` 的鍊式寫法，定義每個指令的 `名稱`、`後綴`、`執行內容`。
    - 接著使用 `.build()` 取得回傳好的樹 `LiteralCommandNode`，並註冊到插件中。
- 實作過程 ⭐⭐⭐⭐⭐
    - 先創建一個指令類別，我以 `/spawn <entity>` 做測試。
    ```java=
    public class SpawnCommand {
        public static LiteralCommandNode<CommandSourceStack> create(){
            return Commands.literal("spawn").then(
                    Commands.literal("creeper").executes(
                            // ...執行內容
                    )
            ).then(
                    Commands.literal("zombie").executes(
                            // ...執行內容
                    )
            ).build();
        }
    }
    ```
    - 接著設計生成生物的函式 : 
    ```java=
    private static int spawn(CommandSourceStack source, EntityType type){
        // 直接以Executor寫，可以避免掉sender是否為實體的問題
        if (!(source.getExecutor() instanceof Player player)) return 0;
        Location loc = player.getLocation(); 
        System.out.println(loc);
        player.getWorld().spawnEntity(loc, type);
        return 1;
    }
    ```
    - 我發現我很不會用Early return的設計模式ww，似乎得習慣一下ww。
    - 最後要到插件啟動點做註冊 : 
    ![image](https://hackmd.io/_uploads/ryK6DZA6fx.png)
    - 最後就可以得到 : 
    ![image](https://hackmd.io/_uploads/S1BkF-0Tfg.png)

## Day313
#### 學習重點 : Minecraft Plugin第四課 : 指令參數
- 關於參數輸入 ⭐⭐⭐
    - 在Commands中有分成 : `literal`、`argument`，而literal就是所謂文字後綴，而argument是可供使用者自由輸入的「值」。
    - 一般來說跟literal的寫法相同，但在存取值的部分則需要加個東西 : 
    ```java=
    Commands.literal("heal")
        .then(Commands.argument("amount", DoubleArgumentType.doubleArg(1.0, 20.0))
              .executes(
                ctx -> healByAmount(ctx.getSource(), DoubleArgumentType.getDouble(ctx, "amount")
                )
        ))
        .build();
    ```
    - 簡單來說就是在當前context設下一個變數來存使用者輸入的值，而這些type是由ArgumentType所規範，當然也可以自己設計，只是這個偏複雜，之後再來研究！
    - 且參數可以設置大小值，這樣在伺服器當中輸入不符合範圍之值會顯示紅字，十分方便。
    - 這個感覺跟PDC存的key-value pair很像！感覺Mojang的設計方式都長得很像JSON格式ww

## Day314
#### 學習重點 : Minecraft Plugin第四課 : 特殊指令參數
- ArgumentTypes的參數 ⭐⭐⭐⭐⭐
    - 在進入客製化參數之前，我們可以先來看看官方提供的特殊Arguments。
    - 常見的像是 : `minecraft:...` 後面接上物品的名稱的ItemStack、`players` 提供目標選擇器（也就是所謂的 `@p、@a...`）。
    - 今天我利用目標選擇器設計了一個可以對特定玩家s發訊息的指令 `/admin send <target> <message>`，剛好結合了特殊argument以及一般String type。
- 實作內容 ⭐⭐⭐⭐
    - 以下是我的CommandNode : 
    ```java=
    public static LiteralCommandNode<CommandSourceStack> create(){
        return Commands.literal("admin")
                .then(Commands.literal("send")
                .then(Commands.argument("target", ArgumentTypes.players())
                        .then(Commands.argument("message", StringArgumentType.greedyString())
                            .executes(
                                ctx -> {
                                    var resolver = ctx.getArgument("target", PlayerSelectorArgumentResolver.class);
                                    var players = resolver.resolve(ctx.getSource());
                                    var msg = StringArgumentType.getString(ctx, "message");
                                    players.forEach(p->p.sendMessage(msg));
                                    return Command.SINGLE_SUCCESS;
                                })
                        )
                ))
                .build();
    }
    ```
    - 我在實作的時候確實有體會到context的概念，他就像是一個節點儲存庫，我可以針對當前節點存值取值，但僅限於當前分支，下文可以使用上文提供的arguments！
- 最近剛好在考試，所以沒什麼時間寫其他東西，等考完再報復性學一波ww。

## Day315
#### 學習重點 : Minecraft Plugin第四課 : 指令權限設計
- 關於指令權限 ⭐⭐⭐⭐
    - 在插件當中，我們通常會在指令設計中加入權限驗證，而好處在於只要沒有權限，甚至連指令都 **無法找到**（而不是找到指令送出是紅字）
    - 而在原本實作BasicCommand的設計中，我們會需要實作 `permission` method並回傳這指令類別的權限要求。
        - 但這就沒有辦法做細分，像是主指令可以，但後綴限制等等...。
    - 而在新版的鍊式寫法，可以在每個Commands後加上 `.requires`，並且提供Predicate，通常是以 `hasPermission` 的內建寫法去尋找sender有沒有該權限。
    - 接上昨天的寫法，就會長這樣 : 
    ```java=
    public static LiteralCommandNode<CommandSourceStack> create(){
        return Commands.literal("admin")
                .requires(commandSourceStack -> commandSourceStack.getSender().hasPermission("commandPerm.send"))
                .then(Commands.literal("send").then( // ...
    ```
- 給予權限 ⭐⭐⭐
    - 而一般會在 `plugin.yml` 設計權限對應的角色（是否op、全面開啟...）
    - 應該會長這樣 : 
    ```yaml=    
    permissions:
      commandPerm.send:
        description: send command only for operator to use
        default: op
    ```

## Day316
#### 學習重點 : Minecraft Plugin第五課 : TextComponent.1
- 關於TextCompenent的設計 - 偷懶部分
    - 今天沒什麼時間，先簡單帶一個Component的設計
    - 我們知道在Minecraft中，有三種顯示字的形式 : `chat、actionbar、title`
    - 而字體本身也可以觸發clickEvent，因此這次主軸應該會著重於這些Component的鍊式寫法。
    - 話說有一個網站似乎可以直接提供模板生成程式！[Component Studio](https://component-studio-six.vercel.app/)
    ![image](https://hackmd.io/_uploads/rJyM-x7Czx.png)
    - 基本上就是給一個ComponentFactory，只需要取得回傳值即可。

## Day317
#### 學習重點 : Minecraft Plugin第五課 : TextComponent.2
- 關於Audience ⭐⭐⭐
    - 在sender的介面中，可以發現其繼承了Audience的介面，而Audience就用來統一所有訊息的事件，包括chat、actionbar、title等等，再藉由不同實體去實作Audience（Sender就是其中一個。
    - 基本上只要跟「通訊」相關的實體，都會實作Audience，因此我們才可以使用sendMessage...。
- 關於actionbar、title的設計 ⭐⭐⭐
    - 而如同Audience所設計的，有包括actionbar跟title的傳送，且title可以加上Duration這種漸入漸出的字幕設計。
    - 以下是我的設計 : 
    ![image](https://hackmd.io/_uploads/BJXmpo4Rfg.png)

## Day318
#### 學習重點 : Minecraft Plugin第六課 : Configuration
- 關於Configuration ⭐⭐⭐
    - 我們知道在插件有個 `config.yml` 可以被設計後讀取，使插件使用者可以於後台藉由該檔案來設定數值。
    - 像是最常見的Essential就提供了像是更改暱稱前輟、home點數量、權限設計等configuration。
    - 然而在Minecraft究竟要如何讀取與設計呢？
- 設計config.yml ⭐⭐⭐⭐
    - 首先要先在 `resources/` 下建立 `config.yml`，而yml架構就不說了，應該已經很了解了w。
    - 我們即可在其中寫下要提供給使用者的配置變數。
    - 而在啟動函式中，需寫下 `saveDefaultConfig` 來將程式設計的config儲存到輸出資料夾中（注意！若目標資料夾已有config，則不會覆蓋）
- 讀取config.yml ⭐⭐⭐
    - 假設我要在原本announceCommand被呼叫時，讀取 `config.yml` 的變數路徑，可以這樣寫 : 
    ```java=
    //為避免該路徑意外不存在，可以加入預設值
    plugin.getConfig().getInt("testing", 10) 
    ```
    - plugin由啟動函式create該指令時傳入，接著就可以取值拉～

## Day319
#### 學習重點 : Minecraft Plugin第六課 : 客製化Configuration與讀取
- 關於YamlConfiguration讀取這回事 ⭐⭐⭐⭐
    - 首先要先釐清兩件事 : 
        - `.jar` 內部通常會在 `resources/..` 放預設配置檔，無論是官方提供的config，抑或者是自己的檔案。
        - 執行插件後，`resources/..` 的配置檔會輸出到 `plugins/<插件名稱>/..` 之中，也就是所謂的硬碟存取區。
        - 接著我先透過讀取配置檔時，會先將其讀到RAM中，直到save被呼叫才會更新到硬碟中，因此未save前只要伺服器關閉，配置檔也不會更新。
- 該如何客製化檔案 ⭐⭐⭐⭐⭐
    - 簡單來說，需要幾行程式
    ```java=    
    File itemYml = new File(getDataFolder(), "item.yml");
    if (!itemYml.exists()) saveResource("item.yml", false);
    YamlConfiguration config = YamlConfiguration.loadConfiguration(itemYml);
    ```
    - 當啟動 `.jar` 時，會先確認item.yml是否存於硬碟中，不存在才會做新增，存在的話也不會做覆蓋（因為設為false）。
    - 接著我們就可以透過 `loadConfiguration` 將檔案讀到RAM中，進行後續getString、getInt等動作。
- 關於讀寫占用執行緒問題 ⭐⭐⭐⭐⭐
    - 一般來說，我們可以先設計POJO並在啟動時反序列化yml檔，將其映射到POJO上，這樣當要讀取配置時，直接使用物件即可。
    - 但若是遇到需要時常更新硬碟配置的檔案時，可能就得使用非同步寫法來優化效能。
