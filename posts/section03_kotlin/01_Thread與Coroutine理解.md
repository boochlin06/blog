# Thread 和 Coroutine 的理解

> 原文網址：https://boochlin.com/?p=667
> 發布日期：2025-05-17
> 文章編號：667

---

[參考文章](https://impateljay.medium.com/coroutine-vs-thread-understanding-the-differences-4b98a5a22a3a)

簡單來說，**Thread (執行緒) 是由作業系統 (OS) 來管理和調度的**，而 **Coroutine (協程) 的控制權則在我們開發者手上**，由程式語言的執行環境 (Runtime) 在使用者層級進行調度。這個根本性的不同，導致了它們在資源消耗和效能上有著天壤之別。

## **成本**和**適用場景**

Thread 是『重量級』。Preemptive multitasking managed by the OS.

每建立一個 Thread，OS 都需要為它分配獨立的、MB 等級的記憶體堆疊。更致命的是它的『上下文切換』成本。當 OS 決定從 Thread A 切換到 Thread B 時，需要深入核心模式 (Kernel Mode)，保存 A 的所有現場（CPU 暫存器、完整堆疊等），再載入 B 的現場。這整個過程非常耗時，是微秒 (µs) 等級的。

* **重量級 (Heavyweight)：** 建立和管理Thread可能相當耗費資源。

* **阻塞 (Blocking)：** Thread可能會互相阻塞，特別是當它們在等待資源或 I/O 操作時。

* **上下文切換 (Context Switching)：** 在Thread之間切換需要儲存和載入它們的狀態，這可能會很耗費成本。

* **由作業系統管理 (OS Managed)：** Thread 由作業系統管理，這可能會增加額外開銷。

Coroutine 是輕量級 Cooperative multitasking

它活在 Thread 內部，像是一種合作關係。建立一個 Coroutine 的成本極低，只佔用 KB 等級的記憶體。它的切換，是程式自己決定的，通常在await這種等待 I/O 的地方，主動讓出執行權。這個切換完全在使用者模式 (User Mode) 內部完成，就像一次函式呼叫，成本是奈秒 (ns) 等級的，比 Thread 切換快了上千倍。

* **輕量級 (Lightweight)：** Coroutine比Thread輕得多；你可以建立數千個Coroutine而不會有顯著的開銷。

* **非阻塞 (Non-Blocking)：** Coroutine使用暫停函式 (suspending functions) 來讓出控制權而不阻塞執行緒，從而有效利用資源。

* **較少開銷 (Less Overhead)：** 與Thread相比，Coroutine之間的上下文切換更快且資源消耗更少。

* **由開發者控制 (Controlled by Developer)：** Coroutine提供了對任務執行的更精細控制。

**適用場景**。基於上面的成本差異，我的選擇會非常看重應用場景。

* 「對於 **CPU 密集型**任務，比如影片轉檔、複雜運算，我會毫不猶豫地使用 Thread。因為我需要真正利用好多核心 CPU 的平行處理能力，把運算能力榨乾。這時候切換成本相對次要。」

* 「但對於**高併發 I/O 密集型**任務，比如 Web 伺服器、資料庫連接池，我一定會選擇 Coroutine。在這種場景下，程式有成千上萬的任務，但絕大部分都在等待網路或磁碟回應。如果用 Thread，那光是切換的成本就會把系統拖垮。而 Coroutine 能用極低的代價在這些等待的任務間快速切換，最大化 CPU 的利用率。」
