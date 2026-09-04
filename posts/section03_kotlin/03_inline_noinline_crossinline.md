# inline, noinline, crossinline：存在即合理

> 原文網址：https://boochlin.com/?p=655
> 發布日期：2025-10-14
> 文章編號：655

---

老樣子，存在必合理，有這樣的 Keyword ，那肯定是前人有過不去的坎，所以才設計出這個要想一想的概念

#### 問題：優雅的 Lambda，不堪重負的 GC

Kotlin 的高階函數與 Lambda 是偉大的發明，它讓我們的程式碼變得簡潔、可讀性更高。但這在資源有限的 Android 裝置上，是有著血淋淋的代價的。

**痛點：Lambda 會產生隱性的物件開銷。**

每一段 Lambda `{...}`，在底層都會被編譯器變成一個新的匿名類別實例。這意味著，每一次呼叫使用 Lambda 的高階函數，都會在記憶體中 `new` 一個新物件。

如果只是按鈕點擊，無傷大雅。但想像一下，若這段邏輯發生在 RecyclerView 的 onBindViewHolder，或是一個每秒更新 60 次的動畫迴圈裡？這將瞬間產生海量的、稍縱即逝的短命物件，迫使垃圾回收（GC）機制頻繁啟動，瘋狂加班。而 GC 一旦工作，

#### inline 那就別 new 啦，直接把程式碼貼過去

* **問題：** 高階函數與 Lambda 造成的「效能開銷」與「記憶體壓力」。

* **運作：** 當你將一個函數標記為 inline，你等於是命令編譯器：「停止創建物件！直接把這個函數的程式碼，連同傳入的 Lambda 程式碼，像『**複製貼上**』一樣，完整地貼到呼叫它的地方。」

* **結果：** 沒有了物件的建立，沒有了函數的呼叫成本。用增加一點點 APK 體積的代價，換來了執行時的極致效能。

然而，inline在解決問題的同時，也創造了兩個棘手的副作用，這就需要另外兩把更精密的工具來輔助。

#### 三、noinline：在「複製貼上」中保留一個「超連結」

* **問題：** 當函數被 inline 後，其 Lambda 參數「失去物件特性」的問題。

* ****運作**：** inline 的「複製貼上」特性，意味著 Lambda 不再是一個獨立的物件，它被分解成了程式碼片段。這導致你無法將這個 Lambda 賦值給一個變數，也無法再將它傳遞給其他需要物件的函數。

* **解決：** noinline 就像一個例外標籤。它告訴編譯器：「這個 inline 函數中的所有東西你照常貼上，但請唯獨放過這個標記了 noinline 的 Lambda，讓它**維持物件的原貌**，以便我後續操作。」

#### 四、crossinline：為「跨部門任務」簽下一份「切結書」

* **問題：** inline 所賦予的「非局部控制權」，在不安全情境下可能造成的「邏輯混亂」。

* **說明：** inline 賦予了 Lambda 一個強大的超能力——「非局部控制」（non-local control）。最典型的就是 return，在 Lambda 內部 return 可以直接結束外層的整個函數。這個超能力在 Lambda 被立即執行的情況下很方便。但如果你把這個 Lambda 傳遞到另一個執行上下文（比如另一個執行緒 Runnable，或另一個物件）中稍後執行呢？這就好像你派了一個助手去「隔壁部門」辦事，結果他卻在隔壁部門直接按下了按鈕，把你「本部門」的整個專案給終止 (return) 或暫停 (break) 了。這是絕對不允許的邏輯混亂。

* **解決：** crossinline 就是你與這個 Lambda 簽訂的「切結書」。它向編譯器保證，這個 Lambda **不會使用任何非局部控制權**。它依然會被內聯以獲取效能，但它內部的 return, break, continue 等指令，都不能影響到外部函數的流程。這就確保了程式的穩定性與可預測性。

[Kotlin 源码里成吨的 noinline 和 crossinline 是干嘛的？看完这个视频你转头也写了一吨](https://rengwuxian.com/kotlin-source-noinline-crossinline/#:~:text=%E6%80%BB%E7%BB%93%20%20inline%20%E5%8F%AF%E4%BB%A5%E8%AE%A9%E4%BD%A0%E7%94%A8%E5%86%85%E8%81%94%E2%80%94%E2%80%94%E4%B9%9F%E5%B0%B1%E6%98%AF%E5%87%BD%E6%95%B0%E5%86%85%E5%AE%B9%E7%9B%B4%E6%8F%92%E5%88%B0%E8%B0%83%E7%94%A8%E5%A4%84%E2%80%94%E2%80%94%E7%9A%84%E6%96%B9%E5%BC%8F%E6%9D%A5%E4%BC%98%E5%8C%96%E4%BB%A3%E7%A0%81%E7%BB%93%E6%9E%84%EF%BC%8C%E4%BB%8E%E8%80%8C%E5%87%8F%E5%B0%91%E5%87%BD%E6%95%B0%E7%B1%BB%E5%9E%8B%E7%9A%84%E5%AF%B9%E8%B1%A1%E7%9A%84%E5%88%9B%E5%BB%BA%EF%BC%9B%20%20noinline%20%E6%98%AF%E5%B1%80%E9%83%A8%E5%85%B3%E6%8E%89%E8%BF%99%E4%B8%AA%E4%BC%98%E5%8C%96%EF%BC%8C%E6%9D%A5%E6%91%86%E8%84%B1inline%20%E5%B8%A6%E6%9D%A5%E7%9A%84%E3%80%8C%E4%B8%8D%E8%83%BD%E6%8A%8A%E5%87%BD%E6%95%B0%E7%B1%BB%E5%9E%8B%E7%9A%84%E5%8F%82%E6%95%B0%E5%BD%93%E5%AF%B9%E8%B1%A1%E4%BD%BF%E7%94%A8%E3%80%8D%E7%9A%84%E9%99%90%E5%88%B6%EF%BC%9B%20*%20crossinline%20%E6%98%AF%E5%B1%80%E9%83%A8%E5%8A%A0%E5%BC%BA%E8%BF%99%E4%B8%AA%E4%BC%98%E5%8C%96%EF%BC%8C%E8%AE%A9%E5%86%85%E8%81%94%E5%87%BD%E6%95%B0%E9%87%8C%E7%9A%84%E5%87%BD%E6%95%B0%E7%B1%BB%E5%9E%8B%E7%9A%84%E5%8F%82%E6%95%B0%E5%8F%AF%E4%BB%A5%E8%A2%AB%E5%BD%93%E5%81%9A%E5%AF%B9%E8%B1%A1%E4%BD%BF%E7%94%A8%E3%80%82)

https://medium.com/evan-android-note/kotlin-%E7%9A%84-inline-noinline-crossline-function-8deceacc893d
