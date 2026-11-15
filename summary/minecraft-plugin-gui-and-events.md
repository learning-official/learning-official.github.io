# Paper 插件：事件、GUI 與 PDC

## Day302
#### 學習重點 : 重回Minecraft Java Plugin的懷抱
- 重回Minecraft Java Plugin的懷抱 ⭐⭐⭐⭐
    - 其實當初促使我學習Java的其中一個原因就是寫麥塊插件ww，而歷經基石的奠定以及框架的設計，我認為我現在對於Java已經算有一定的了解了！
    - 前幾年我在探索插件的時候，是使用Spigot作為核心，然而現今使用Paper好像是個更好的選擇owo，因此我這次會選擇使用Paper來開發！
    - 但在進入Plugin之前我想先好好認識何謂Paper？
    - 早在Paper成為主流之前，Minecraft伺服器經歷了幾個時期 : 
        - **原生Vanilla Server** : 原版的伺服器是不支援plugins的，也就是原味生存伺服器拉～
        - **Bukkit的誕生** : 後來玩家訂定了Bukkit API，定義了伺服器上的功能，後續則有 `CraftBukkit` 作為實作。
        - **Spigot** : 後來Spigot針對CraftBukkit進行了優化，同時結合了BungeeCord奠定了伺服器的興盛。
        - **Paper** : 針對Spigot單一執行緒加載的問題進行優化，改成非同步，優化伺服器效能！
    - 話說我今天光是搞Minecraft Plugin的Gradle build就花了一段時間，所以明天寫完再統整好ㄌ～

## Day303
#### 學習重點 : Minecraft Plugin第一課 : 伺服器啟動與JavaPlugin
- 伺服器啟動 ⭐⭐⭐⭐⭐
    - 記得之前使用spigot時，需要自行建立伺服器，但gradle似乎有提供直接runServer的選項，大幅提升了開發速度！
    - 而基本的伺服器啟動要素則是需要有一個Class繼承 `JavaPlugin`，並覆寫onEnable與onDisable的method
        - 而這其中的緣由就如同main作為程式進入點一樣。**作為一個插件的啟動點**，在 `plugin.yml` 中設定了以下東西 : 
        ```yml=
        main: me.xiaoba.firstPlugin.FirstPlugin
        ```
        - 我們設定了main是指向FirstPlugin這個類別，因此在伺服器啟動時，就會去利用 **反射檢查** 該類別是否繼承JavaPlugin，若是則向上轉型成JavaPlugin，交由Paper去掌控該插件的後續動作。
        - 而原本我們是應該自行去實作 `Plugin介面`，但JavaPlugin幫我們實作好，並建立起插件的骨架，只需專注在啟動與關閉的事件即可。
        - 仔細看可以發現JavaPlugin是個 **抽象類別**，其中留了啟動與關閉尚未實作！
    - 可以發現以上的Plugin類別向上轉型也是一種 **反轉控制** 的經典例子！由伺服器去掌控物件的生命週期owo。
- 成功載入插件與運行Server！ ⭐
    - 示意圖 :
    ![image](https://hackmd.io/_uploads/rkBrsVGaMg.png)

## Day304
#### 學習重點 : Minecraft Plugin第二課 : 事件監聽與註冊
- 何謂事件監聽？ ⭐⭐⭐⭐⭐⭐⭐
    - 所謂的事件係指「遊戲內部的任何動態事件」，與昨天所提及的「啟動、關閉」並非同一種類型，因此啟動關閉也是事件，但不屬於監聽範圍。
    - 而Paper的事件監聽會使用 `Listener` 介面來統一所有事件監聽的實作類別。
    - 當我們建立一個事件的監聽行為，必須有底下步驟 : 
        - **建立事件類別，實作 `Listener` 介面**（可以發現該介面沒有任何抽象方法，因其僅作為標記介面Tagging Interface）
        - 於事件類別內部設計監聽的函式，並掛上 `@EventHandler` 標記為監聽函式。
        - 最後在 `onEnable` 中註冊監聽類別的物件。
    - 了解完上述步驟後可以繼續深入挖掘 : 
        - 對於伺服器啟動時註冊的監聽器，最終都會被PluginManager放進相對應類型的 `HandlerList`，可以注意到，每一個事件的類別開頭都是像底下這樣 : 
        ![image](https://hackmd.io/_uploads/ryF85tXaMe.png)
        - 代表每次註冊的Listener，Manager會根據內部監聽器所監聽的事件，放進相對應的List。
    - 不過其實我很好奇為何監聽類別要自行new給Manager，而不是使用DI直接注入給IoC容器來管理呢？
- 事件實作 ⭐⭐⭐
    - 由於我以前很喜歡玩Hypixel的遊戲，其返回大廳的方式是對紅色床點右鍵，所以我就寫了一個監聽 `PlayerInteractEvent` 的事件，並偵測 `是否是右鍵` 及 `Material是否為RED_BED` 來觸發！
    ![image](https://hackmd.io/_uploads/BJXthF7TGx.png)
    - 接著再到onEnable註冊！
    ![image](https://hackmd.io/_uploads/SJO3ht76Gg.png)
- 話說我今天筆記寫得好嚴肅ww，寫完才發現ww

## Day305
#### 學習重點 : Minecraft Plugin第三課 : GUI介面及點擊事件 - 1
- GUI介面 ⭐⭐⭐⭐
    - 此次學習重點多出自於 : [PaperMC - Menu Type API](https://docs.papermc.io/paper/dev/menu-type-api/)，如果要參考請前往
    - 常玩伺服器的玩家應該知道，伺服器通常都會有選單，而早期在寫選單時，會使用 `Bukkit.createInventory` 來建立容器，並使 `player.openInventory` 來打開Menu。
    - 但自從 `1.21` 之後，Paper加上了 `MenuType` 及更新了 `InventoryView` 的功能，來更直覺的建立Menu。
    - 而在此得先看看 `Inventory` 跟 `InventoryView` 的差別 : 
        - `Inventory` : 單純裝ItemStack的地方，也就是很單純的容器。
        - `InventoryView` : 當玩家打開Menu時所產生，分為 `TopInventory` 跟 `BottomInventory`，而Top就是menu本人，Bottom則是玩家的背包，兩者都是容器，但都在同一視窗，因此整合稱為View。
    - View除了存取兩個Inventory，還會存取開啟的玩家、視窗標題，並掌控視窗的開啟關閉。
- MenuType建構View ⭐⭐⭐⭐
    - 我們一般會使用MenuType來建構一個InventoryView，比起早期的 `Bukkit.createInventory` 需要自行填入slots，新型的MenuType採用的是Enum形式，規定了slots的格式，而且還可以設置成特殊介面（熔爐、工作台等等）
    - 我就不秀出我怎麼建立的，反正大同小異w。
- 點擊事件與偵測 ⭐⭐⭐⭐⭐⭐
    - 我想要設計一個功能，當玩家開啟menu點擊紅色床後可以回大廳，但這產生出幾個問題
        1. 點擊後要怎麼知道我是點擊「menu內的紅色床」，而不是玩家背包的紅色床？
        2. menu內的紅色床跟其他的紅色床有甚麼差別？
    - 第一個問題 : 可以透過 `inventoryView.getTopInventory() == event.getClickedInventory()` 確認玩家是在menu操作。
    - 第二個問題 : 需要使用物品特殊的PDC（PersistentDataContainer）屬性，它也存在於NBT當中，屬於插件級別的東西，因此不能單純用指令、鐵砧能夠改動，這給我們一個方便之處去賦予紅色床標籤！
- 今天好像寫得有點太多了，明天再繼續！

## Day306
#### 學習重點 : Minecraft Plugin第三課支線 : ItemStack與ItemMeta
- 何謂ItemMeta？
    - 在理解ItemMeta之前，得先看看ItemStack。
    - 何謂ItemStack？ ⭐⭐⭐
        - ItemStack簡單理解就是 : 物品的骨幹，包含「數量、材質、ItemMeta」
        - 可以發現ItemStack似乎把屬性、名字...都包給了ItemMeta，而正是如此！
    - ItemMeta作為一個物品的詳細資訊，卻獨立成了一個介面，為何呢？ ⭐⭐⭐⭐
        - 深入Bukkit邏輯可以發現很多物品本身資訊架構就不一樣，像是Book會存取title、author、content...，而旗幟則有pattern，因此不可能所有物品的資訊都塞進ItemMeta作為成員。
        - 獨立出介面後，ItemMeta本身就只關注所有物品都可以做的「自訂名稱、自訂Lore、附魔...」屬性，而特殊的屬性則由BookMeta、BannerMeta...介面再去繼承，加上函式。
    - 既然所有物品都有「自訂名稱、自訂Lore...」，幹嘛不直接放在ItemStack？ ⭐⭐⭐⭐⭐⭐⭐⭐⭐
        - 首先，Minecraft世界的任何掉落物、箱子內的物品都是ItemStack，**預設** 生成這些東西是 **不需要自訂的屬性**，畢竟量大很耗記憶體，因此才會獨立出來給ItemMeta。
        - 這樣當生成這些物品、或者我建立一個ItemStack，就不會特地去生成ItemMeta，除非等我去設定才會建立。
        - 這邊很有趣，在ItemStack類別是找不到ItemMeta的成員，只能找到底下這行 : 
        ```java=
        public ItemMeta getItemMeta() {
            return this.craftDelegate.getItemMeta(); // Paper - delegate
        }
        ```
        - 其使用了「委託模式+動態工廠轉換」 : 
            - `craftDelegate` 本人是 `ItemStackServer` 介面，在執行期預設實作類別是 `CraftItemStack`，實際去抓取原始的物品資料。
            - 當 `CraftItemStack` 抓到底層的物品meta時，會進到工廠，根據物品回傳相對應的特殊ItemMeta（繼承了 `CraftMetaItem`）。
        - 綜上所述可以發現，「**ItemMeta就是一種Lazy Evaluation**」！
        - 我請AI給我一個流程比較好理解owo : 
        ```
          │ 1. 呼叫 item.getItemMeta()
          ▼
        ItemStack
          │ 委託：this.delegate.getItemMeta()
          ▼
        CraftItemStack (delegate 的具體實作)
          │ 工廠：CraftItemFactory.getItemMeta(...)
          ▼
        CraftMetaItem (動態new出來ItemMeta骨架，實作了ItemMeta)
          │
          │ 2. 拿到後進行修改：meta.displayName("大廳")
          │ 3. 呼叫 item.setItemMeta(meta)
          ▼
        CraftItemStack
          │ 接收 CraftMetaItem，提取內部資料並序列化
          ▼
        net.minecraft.world.item.ItemStack (Mojang 原版底層物品)
        ```
        ```
              ItemMeta (介面)
                   ▲
                   │ implements
              CraftMetaItem (基礎實作類別)
                   ▲
       ┌──────────┴──────────┬─────────────────────┐
       │ extends             │ extends             │ extends
        CraftMetaBook         CraftMetaPotion        CraftMetaSkull ...      
      │ 
      ➤ implements BookMeta   
        ```
- 我發現我這樣寫下去，demo會一直都找不到時間展示ww，明天稍微消停一下好了。

## Day307
#### 學習重點 : Minecraft Plugin第三課 : GUI介面及點擊事件 - 2
- 接續ItemMeta與點擊事件 ⭐
    - 昨天稍微認識了下ItemMeta的特點，今天可以結合昨天所學設計一個特殊的 `RED_BED`，帶有屬性，這樣就可以解決前天的第二個問題 : 「menu內的紅色床跟其他的紅色床有甚麼差別？」。
- ItemMeta的自訂資料屬性PDC ⭐⭐⭐⭐⭐
    - 在ItemMeta中，有規定一種 `PersistentDataContainer（PDC）`，雖然他隸屬於NBT內部，但如它的名字所說，是 **持久的資料容器**，因此不會根據遊戲進行而改變（除非自行去設定）。
    - 且PDC普遍來說是由插件去修改（儘管其隸屬於NBT，但無法輕易透過指令進行修改，除非知道PDC詳細結構）。
- 應用PDC ⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐
    - 我們可以透過生成一個ItemStack，材質是RED_BED，接著設定ItemMeta中的PDC，這樣就可以去比對我點這個紅色床是否是正確的紅色床啦～
    - 底下是我的實作code（參考 [PaperMC : PersistentDataContainer](https://docs.papermc.io/paper/dev/pdc/)）
    ```java=
    private void setUpItems(ItemStack item, int pdcValue, int slot){
        item.editPersistentDataContainer(
                pdc -> {
                    pdc.set(
                        this.namespacedKey,
                        PersistentDataType.INTEGER,
                        pdcValue
                    );
                }
        );
        this.inventoryView.getTopInventory().setItem(slot, item);
    }
    ```
    - 這個NamespacedKey的意義是「**設定命名空間**」。因為可能有多個插件對同一物件加PDC屬性，因此在NBT結構中為了分辨，要先設定命名空間，**作為子空間的概念**。
        - NBT結構如下（這邊把namespace跟key分成兩層，但實際是namespace : key） 
        ```
        {
            // 物件原本的原生 NBT (物品的 id, Count 等)
            ... 
            PublicBukkitValues: {

                // 「命名空間 (Namespace)」
                "my_plugin": {

                    // 第二層：設定key-value pairs
                    "my_key": 1b,                         // Byte 
                    "player_score": 1500,                 // TAG_Int
                    "custom_description": "這是一行自訂文字" // TAG_String
                },

                // 另一個插件的命名空間
                "other_plugin": {
                    "cooldown_time": 17182938292l         // TAG_Long
                }
            }
        }
        ```
        - 而宣告一個NamespaedKey會給定命名空間跟第二層Key : `NamespacedKey actionItem = new NamespacedKey(this, "action");` 像這樣。
        - 至於value，則會先給定Type，再來是實際的值。
- GUI View的註冊 ⭐⭐⭐⭐⭐
    - 由於我要確認「玩家是在menu操作，而不是一般容器操作」，儘管我前天的想法是 `inventoryView.getTopInventory() == event.getClickedInventory()`，但「只能確認玩家是在容器操作而不是背包操作」，我似乎誤解了。
    - 因此我先設計了一個　`MenuManager`，當玩家使用「**插件給定**」的紅色床or指令or...的方式開啟View時，先把這個View註冊到Manager當中。
    - 接著玩家點擊事件發生時，我就先去確認該事件的View是否有被註冊，而玩家關閉View時，就自動去呼叫Manager註銷。
    - 實際作法如下 : 
    ![image](https://hackmd.io/_uploads/rJZPHT8pMx.png)
    ![image](https://hackmd.io/_uploads/BJNKH6ITze.png)
    ![image](https://hackmd.io/_uploads/ByNoS6IpMl.png)
    ![image](https://hackmd.io/_uploads/rk32Sa86fg.png)

## Day308
#### 學習重點 : Minecraft Plugin第三課 : GUI介面及點擊事件 - 3
- PDC取得及判別 ⭐⭐⭐⭐
    - 透過昨天的 `item.editPersistenDataContainer` 可以去取得ItemStack的ItemMeta並修改PDC。
    - 而今天則是要將PDC取出來，並判斷物品PDC是否有正確的標籤，才能執行後續程式。
    - 取得PDC : 
    ```java=
    Integer value = event.getCurrentItem().getItemMeta().getPersistentDataContainer().get(
            this.namespacedKey, 
            PersistentDataType.INTEGER
    );
    ```
    - 要注意的是，與一般Container一樣，記得要做null check。
- Menu解耦與Manager註冊Menu介面代替View ⭐⭐⭐⭐⭐⭐
    - 我先抽離出了Menu介面，讓其他Menu（如LobbyMenu）去實作它。
        - Menu介面規範了getView的功能，讓menu與view綁定。
    - 接著將原本MenuManager存取InventoryView的部分改成了存取Menu介面。
        - 再藉由「介面規範的getView」與「sessionCheck的InventoryVeiw參數」做比對！
- addButtons替換setItems，加入Consumer之Callback接收觸發動作 ⭐⭐⭐⭐⭐
    - **原本** setItems單純接收Material、slot、pdcValue去建立一個帶有資訊的ItemStack，但實際動作卻要我再寫一個對應函式，有點麻煩。
    - 因此我改成除了接收上述三者，還會接收 `Consumer<InventoryClickEvent>` ，在同一函式 `addButtons` 完成所有設定。
        - 前三者一樣放進inventoryView去setItem。
        - 而後者則放進我自製的menuMap，以pdcValue對應Consumer，這樣當我取出pdcValue後就可以找到相對應的動作啦~~
    - 不過這邊留個小疑問 : 「我似乎可以直接將slot作為key對上Consumer就好，這樣可以省去PDC的存取問題emmmm」。
