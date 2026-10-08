---
title: 雜七雜八的知識

---

## 何謂菱形繼承？
```
  A (基類)
 / \
B   C (中間類別)
 \ /
  D (衍生類別)
```
- 如上圖，由於c++允許多重繼承，因此像D類別就會有 `兩份` 關於A類的資料（B副本 or C副本），這樣當我在D類呼叫A類的成員時，編譯器會不知道要透過B還是C去找！
- 以上這種情況就是菱形繼承！

## 何謂Proxy（代理）？
- 總結來說，可以將Proxy看作是「中繼站」，負責分配、暫存的工作，但究竟為何要有Proxy的存在呢？
- 這可以從 `Server & Client` 的交流來解釋 : 
    #### Forward Porxy
    - 當Server與Client在溝通時，Server通常會存取Client端的資訊 ➞ 所謂的資訊有分為 「連線層資訊 : IP、Port...」、「應用驗證層資訊 : Header、Body...」。
    - 若我們在Client端發送請求前加上Proxy，這樣Server存取的 **連線層資訊** 就會是Proxy的，而不是Client。
        - 這種Proxy稱為「**正向Proxy**」。
    #### Reverse Porxy
    - 當Client發請求到Server時，Server通常要確認Client端是否為「惡意請求」，也要避免自己的IP位置洩漏，這時候就會設一個Proxy在Server端前面，讓請求先抵達Server前的Proxy，使Client認為Proxy就是Server端點，但實質上該Proxy只是代理Server，幫忙過濾請求及保護Server資訊。
    - 同時，當接收過多請求，Server就會超過負載上限，但透過Proxy則可以分配請求至不同Server，以減少負荷。
    - 像是市面上常見的Cloudflare，就是很經典的Proxy。
        - 這種Proxy稱為「**反向Proxy**」。
    - 而無論是正向還是反向Proxy，他們都有共通點，也就是「快取」，快取存在的意義就是讓使用者更快取得靜態資料，同時減少伺服器的負荷。
    - 正向Proxy以Client視角來看，快取就是當使用者多次存取某網頁時，Proxy會將該網頁快取至自身的儲存空間中，以加入下次使用者請求時載入的速度。
    - 反向Proxy以Server視角來看，它可以將靜態資源分散（CDN...）至各地伺服器，以離Client最近的伺服器去存取，降低延遲與負荷。
