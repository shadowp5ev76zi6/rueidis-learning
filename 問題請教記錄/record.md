# Rueidis 問題記錄

> 以下為所有請教問題的記錄

## 2022年1月27日

### 問題

當我看到 Redis 6 時，我在想一個問題當一個資料在 Redis 被改變時，是 Redis 會主動馬上通知我呢？還是我還要再去問 Redis ？看起來 Redis 不會主動通知我去更新 Cache，還要自已跑去問再去更新 Cache

 <img src="../assets/image-20220309215214038.png" alt="image-20220309215214038" style="zoom:100%;" />

### 解答

如果有用到server-assisted client side caching 的話，Redis 會主動通知你喔圖片圈起來的部分是想問什麼問題呢？

### 問題

紅色圈起來的地方，像是程式碼主動去問 Redis，那如果不是，能否指出 Redis 會主動通知 Client 的程式碼位置？謝謝

### 解答

紅色圈起來的部分是程式在被動的讀取 redis 傳過來的訊息處理 redis invalidation notification 的地方是在裡面一點 [https://github.com/....../f957b83a911fc6....../pipe.go......](https://github.com/rueian/rueidis/blob/f957b83a911fc695a0a6137171d1a54e8a4724ce/pipe.go?fbclid=IwAR1uyznrRrjcTeuJiAaRJr7dEL_efS3U6hKj-G3r9lnOKh7ieqp6IbjRTRE#L271)

## 2022年2月3日

### 問題

pipe 的優點在程式碼裡可以看得到，DoMulti 函式可以處理向 Redis 傳送多條訊息，再一次讀取多條回應的結果

請教一下，我有看到您把 pipe 的連線用 Close() 函式關閉，pipe 的連線不就無法再繼續接收到 Redis 的新訊息了

那您是如何恢復 pipe 的連線的？

如果能的話，能否配合程式碼說明，謝謝

<img src="../assets/image-20220309220454983.png" alt="image-20220309220454983" style="zoom:100%;" />

### 解答

走到那邊的話，的確是沒有要讀取之後的訊息了重新連線的部分可以看這裡 [https://github.com/....../02cd0ba16fe89a4....../mux.go......](https://github.com/rueian/rueidis/blob/02cd0ba16fe89a4635d93a7a1f7b05e4d3fb6be7/mux.go?fbclid=IwAR0dG2Jhzb9PT2cdhgZA1IW5ICAFm2V3GLsRmtgXl4fv8zZObUztz4drXKU#L89)

## 2022年2月10日

### 問題

這一段程式碼就可以看出是環狀結構環狀的數量為 2 的 n 次方，

比如 2^3 = 88 的二進制為 1000

環狀的數量減一的值為 77 的二進制為 0111

1000 和 0111 用 & 操作為 0

今天值只要一直增加，加到 8 時，用 & 操作，就直接跳回原點 0 就為環狀結構

請教一下，PutOne 和 PutMulti 兩個函式中，是否可以用 PutMulti 函式去取代 PutOne 函式的功能

可能您為了能節省效能，所以另外在多寫出 PutOne 函式

還是說 PutOne 和 PutMulti 有不同的意義和用途？謝謝

<img src="../assets/image-20220309220829597.png" alt="image-20220309220829597" style="zoom:100%;" /> 

### 解答

我也想用 PutMulti 取代 PutOne，不過還沒找到好方法另外你看的這份有點舊了，建議更新一下

## 2022年2月16日

### 問題

請教一個問題？

大部份 sync.WaitGroup 的 Done 會在 Wait 的前面請問為何這裡程式碼，為何 Wait 會在 Done 的前面，我是認為 _pipe() 會在不同的地方被執行，但細節我目前不清楚？

是否能說明運作流程？謝謝

<img src="../assets/image-20220309221250899.png" alt="image-20220309221250899" style="zoom:100%;" /> 

### 解答

是的，_pipe 同時間會被很多地方呼叫，但我希望同時只有一個能呼叫到 wireFn。因此有拿到 WaitGroup 的就等待結果即可可以參考官方的 singleflight pkg

### 版務人員提供參考資料

可以參考這個解釋比較詳細[https://xnum.github.io/2018/11/syncmap-loadorstore](https://xnum.github.io/2018/11/syncmap-loadorstore?fbclid=IwAR0uCsXXgFhLKd4CZ4ohf_QOakmTVmcntztmPHG2T_l8f3OqpIuE3HKKPRk)

## 2022年2月23日

請教一個問題，我去看程式的流程，當使用 Pipe 時，最後要寫入 EOF 的訊息作為結束，代表沒有資料要傳送了p.background() 會傳入 Pipe EOF 訊息，但在比較後面，而且有 if 判斷式，代表不一定會執行

如果 p.background() 不及時執行的話，那不就代表 writeCmd(p.w, cmd.Commands()) 和syncRead(p.r) 會發生互相等待，那整個 syncDo 函式不會停止

我主要是想要請教 EOF 在程式碼裡是如何傳送和處理的？謝謝

<img src="../assets/image-20220309221926308.png" alt="image-20220309221926308" style="zoom:100%;" />

我預期 EOF 的寫入位置在這裡

<img src="../assets/image-20220309222216001.png" alt="image-20220309222216001" style="zoom:100%;" /> 

### 解答

可能你有什麼誤會，應該沒有 EOF 要寫入

### 問題

我一開始還覺得奇怪，把 mockClient.Close() 註解起來，整個程式永遠不會停止，進行互相等待的死結，後來想想這是合理的因為要告知 Pipe 沒有更多的資料要傳了，執行 Close() 函式時，(應該同時也傳入 EOF)

我是想要請教您，這方面的問題，您是如何處理的？

如果防止 mockClient 和 mockServer 互相等待的死結？

沒有告知對方EOF的訊息，程式不會停的

<img src="../assets/image-20220309222356943.png" alt="image-20220309222356943" style="zoom:100%;" /> 

### 解答

喔，原來你說是 Go 的 io.EOF，這只是 Golang 用來表示一個 read stream 已經讀完了的訊息。

而一個 tcp stream 什麼時候才會讀完？就是讀到對方送給你的 FIN 包的時候，這時候 Golang 會返回 io.EOF 給你。

而你用的 io.ReadAll 的行為就是要把整個 stream 讀完，也就是讀到 io.EOF 才結束。

在 Redis client 中 tcp stream 是重複使用的，不會用 io.ReadAll 來讀取

## 2022年3月2日

### 問題

我這樣理解，您看對不對？

如果要在圖上找 pipe 物件的位置，我認為pipe 類實現 wire 接口，wire 接口可以讀寫 client side cache，因為這接口有 DoCache 函式再上一層的 conn 接口可以重置連線，因為這接口有 Overwrite 函式最後 mux 類為所有功能的大集合，就把它當成主要的控制端Client 接口為用戶介面，Client 接口馬上直接連接到 mux 控制端，就可以控制整個程式了這樣子理解對嗎？謝謝

<img src="../assets/image-20220309222617175.png" alt="image-20220309222617175" style="zoom:100%;" />

### 解答

是的，大致上就是這樣。我的區分方式是 pipe 負責一個 tcp connection。mux 則是負責多個 pipe。 

## 2022年3月17日

### 問題

請教您有關 Cache 的問題，我是覺得看您的程式碼，每看一次會有不同的啟發當 Linked List 用在 Cache 時，我是覺得 hit 數量最多的資料會往前移動但為何在下圖中，反而是往後移動？謝謝

<img src="../assets/image-20220329015537478.png" alt="image-20220329015537478" style="zoom:100%;" /> 

這是另一個版本的 Cache，是往前移動的，和您的程式碼不同

<img src="../assets/image-20220329020753735.png" alt="image-20220329020753735" style="zoom:80%;" /> 

### 解答

單純只是我選擇 list 後面放要留比較久的。要改成放前面也是可以

### 回應

了解，謝謝您，後來再去看程式碼，發現程式清除數據是從頭開始清除了，所以往後移動數據就可以留久一點

<img src="../assets/image-20220329021737097.png" alt="image-20220329021737097" style="zoom:100%;" /> 

## 2022年3月24日

### 問題

這是一個比較沒有變化的 LRU cache 的組件圖

您的 cache 很有趣，變化比較多，之後再畫成另一張組件圖作比較

想請教您，兩個問題

1 您的 cache 有使用 channel 作等待，cache 已經有 sync.RWMutex 多讀單寫的上鎖，

再加一個 channel 做更進一步的控制，讓有些讀取可以進行必要的等待，

我是覺得很有趣，我這樣理解正確嗎？

2 moveThreshold 臨界值的用途為？我是想不透

謝謝

<img src="../assets/image-20220414004906895.png" alt="image-20220414004906895" style="zoom:100%;" />

補上程式碼的圖片

<img src="../assets/image-20220414005013211.png" alt="image-20220414005013211" style="zoom:80%;" /> 

### 解答

1. 對，RWMutex 是為了取出 entry。取出後 entry 本身就用 channel 做 singleflight
2. moveThreshold 只是為了增加使用 RLock 的機會而已

### 回應

關於 single flight 的部份我還要在更仔細的看程式碼了，看看能不能再學習到更多技巧。

後來我再去查，程式不會把 "快取的擊中次數" 重置為零

代表只是單純想讓 "擊中次數超過臨界值的資料" 將更有機會留在快取裡。

謝謝您的引導和解答

<img src="../assets/image-20220414005258672.png" alt="image-20220414005258672" style="zoom:80%;" /> 

## 2022年3月31日

### 問題

我想您說的 Single Flight 和 LRU 快取的關係如下圖

很多 pipe 會執行，但是大多數 pipe 會等待，因為 channel 沒訊號送來，所以會先等待，為下圖的 "Single Flight 的被通知方" 的部份

少數的 pipe ，會更新 cache 和 利用 channel 去通知其他 pipe 快取已更新了，大家可以去讀資料了，為下圖的 "Single Flight 的主動通知方" 的部份

我覺得這張圖還不是很清楚，有錯誤請指正，謝謝

另外我在聽到有人在 meetup 問在本地的快取，如何克服 GC 效能的問題？您在提到一些專案，請問 這些專案的 github 在那？謝謝您

<img src="../assets/image-20220421011851575.png" alt="image-20220421011851575" style="zoom:100%;" /> 

### 解答

我是這樣想：

當使用者用 DoCache() 的時候，先查找 LRU Cache。如果 Cache Miss 的話，會在 Cache 中創建一個 singleflight placeholder，同時也會往下層 pipe 送 redis request。當 response 回來的時候就可以更新 placeholder 並通知其他在等待的 goroutine。

我在 Meetup 中提到的是 bigcache 跟 fastcache：

https://github.com/VictoriaMetrics/fastcache

https://github.com/allegro/bigcache

## 2022年4月7日

### 問題

我也把您上星期所說的改正如下圖，並用紫色來標記

也找到您提到 cache missing 的部份，但還是看不懂，請教您一下，關於這段程式碼
} else if entry != nil {return newResult(entry.Wait(), nil)

我對這小段程式碼的理解為 當 LRU cache 有資料，再進行 single fligt 的等待

我想請教的是，LRU cache 都己經有資料了，為何還要等待，就直接回傳不就好了？

謝謝您

<img src="../assets/image-20220414003928521.png" alt="image-20220414003928521" style="zoom:100%;" />

### 解答

第一個分支的 v.typ != 0 才是真的 cache hit

那個 entry 就是之前提到的 singleflight placeholder，有拿到這個 placeholder 的就要等待

沒拿到 placeholder 的就創建一個並且往下走真的發出 request

## 2022年4月14日

### 問題

我發現之前 Sigle Flight 運作圖，我畫的錯誤太多了，所以大修改

先直接說我認為的結論，您 Single Flight 做兩次，但是這兩次的 Single Flight 不同

第一次的 Single Flight 是在建立和 Redis 的連線

第二次的 Single Flight 才是真的去跟 Redis 進行查詢

雖然程式會先去查詢 LRU 快取，但是和 Redis 的連線會先準備好

您所說的 Place Holder 就是為了要做 Single Flight 的關鍵，所以 LRU 快取才要做 Channel 進行等待

如圖中所示

wire := m.pipe() 為建立連線

resp := wire.DoCache(ctx, cmd, ttl) 為向 LRU Cache 查詢

而 wire := m.pipe() 在 resp := wire.DoCache(ctx, cmd, ttl) 的前面

從這裡可以知道和 Redis 的連線會先準備好

如果還有錯誤請指正，謝謝您

另外，能否提示一下，之前請教的問題，PutOne 函式不能被 PutMulti 取代的原因為何？
謝謝

<img src="../assets/image-20220428033006289.png" alt="image-20220428033006289" style="zoom:100%;" />

附上修正的運作圖

橘色的區塊為第一次的 Single Flight，主要是先準備和 Redis 的連線

紅色的區塊為第二次的 Single Flight，會對 Redis 進行查詢

<img src="../assets/image-20220428033321506.png" alt="image-20220428033321506" style="zoom:100%;" />

### 解答

- 由 mux 去查找 cache 應該是不太對，Cache 是在pipe 物件內部的，圖上改成由 pipe 負責比較接近實際的程式碼
- 是可以被 PutMulti 取代的，只是我覺得分開會對 GC 的影響比較小

## 2022年4月21日

### 問題

我參考之前畫的類圖，把活動圖進行修正，改成
* 產生 mux 物件，pipe 物件實現 wire 接口
* pipe 物件先去查詢 LRU Cache



另外我想請教，您上星期提到，考量的 GC 問題是否為以下內容？

```go
type queue interface {
    PutOne(m cmds.Completed) chan RedisResult
    PutMulti(m []cmds.Completed) chan RedisResult
    NextWriteCmd() (cmds.Completed, []cmds.Completed, chan RedisResult)
    NextResultCh() (cmds.Completed, []cmds.Completed, chan RedisResult, *sync.Cond)
}
```

在 PutMulti(m []cmds.Completed) chan RedisResult 這一行
[]cmds.Completed 為切片內含指標，cmds.Completed 資料內有指標資料，如下

```go
type Completed struct {
    cs *CommandSlice
    cf uint16
    ks uint16
}
```

切片內含指標會給 GC 造成壓力，我以前有想過這問題，後來想想，覺得問題可能不大

因為如果切片內含指標，到時程式會想辨法限制切片的大小，避免 GC 壓力過大

我是覺得，如果把 PutOne 合拼並拼入 PutMulti，雖然會有參數 []cmds.Completed 為 切片內含指標，但是切片大小為1，資料量太小，對於 GC 壓力應不大

我是比較支持 PutOne 拼入 PutMulti，還是您有其他的考量？謝謝

<img src="../assets/image-20220428034315921.png" alt="image-20220428034315921" style="zoom:100%;" />

### 解答

是沒其他考量了，我想這邊還需要再研究一下。
如果對 GC 沒影響，那合併起來是最好，code 會更簡單一些。但有影響的話我覺得分開還是值得的。

## 2022年4月28日

### 問題

以前在看您的程式，會很明顯看到會先執行 Build 函式，一開始會看不太懂

比如

c.Do(ctx, c.B().Set().Key("key").Value("val").Nx().Build()).Error() 中
B() 函式為 Build 函式

之後再把類圖繼續往下畫，圖中紅色部份為最新新增的部份。

queue 接口會使用 Completed 物件做為參數。

而 Completed 物件是由 Arbitrary 物件 提供的方法產生出來的。

Arbitrary 物件會使用 sync.Pool 去提升效能，減少產生和回收物件的動作，減少 GC 的壓力。

所以我認為，會常看到 B() 函式進行 Build 的動作，原因是在於為了要提升程式性能，減少 GC 壓力，不知道這樣的說法是否正確？

另外請教一個問題，在程式碼檔案裡 rueidis/internal/cmds/gen.go ，有一行 Code generated DO NOT EDIT。

這代表整個程式碼是自動產生的，但是在 gen.go 旁沒有看到自動產生程式的腳本，不太清楚這些程式碼是怎麼來的？

也不太清楚 gen.go 的程式碼和 整個專案 rueidis 是如何連接的？

謝謝

<img src="../assets/image-20220516025842683.png" alt="image-20220516025842683" style="zoom:100%;" />

### 解答

B() 是 command builder 的入口，呼叫後就會從 Pool 中取出 []string 準備塞入 command 用，的確是為了減少 GC。
internal/cmds/gen.go 是由 /hack/cmds/gen.go 生出來的

## 2022年5月5日

### 問題

我閱讀了 rueidis/hack/cmds/gen.go 程式碼後，這裡所產生的程式碼 應為最後要傳送到 redis 指令的地方

看到這份自動產生程式碼的程式，第一個直覺在想兩個問題，想要請教您
1 這是否為 AST 節點？
2 在 hack 資料夾裡是在想要突破什麼效能議題？

( 題外話，我目前參與的專案的廣度和深度都比較大，把專案當課本
同一個專案，用到很多技巧，其中一個技巧就是 AST 節點
也有 hack 資料夾，hack 資夾內可能是處理效能的相關議題，我一直覺得 Go 語言是有難度的
所以難免我會有這些聯想 )

關於第一點，您的 rueidis/hack/cmds/gen.go 程式碼，我覺得有點像 Ast 節點，但好像又少了一些東西，

像 Ast 節點的原因為有樹狀結構產生，比如 AclDeluser 的子結點為 AclDeluserUsername
但是結點的類型從頭到尾只有一種類型為
type node struct {
Parent *node
Child *node
Next *node
Cmd command
Arg argument
Root bool
}

如果是 AST 結點的話，應會有各種不同類型的節點可以進行操作
可能您也認為這不是 AST 節點，所以沒有把 package 名種命名為 ast

第二點，放在 hack 資料夾的程式可能會和效能有關，可能您一直在處理 GC 的效能議題，所以才把程式碼放在 hack 資料夾內

之後，自動產生的程式碼也和 sync.Pool 連接在一起
我對 AST 節點不熟，如果觀念錯誤，麻煩指正了，謝謝

<img src="../assets/image-20220516030420284.png" alt="image-20220516030420284" style="zoom:100%;" /> 

### 解答

這段程式碼跟 AST 還有效能都無關，它也不在 rueidis 裡面運行。

它單純是個 code generator 用來生產 rueidis 的 command builder

## 2022年5月19日

### 問題

之後有考慮要使用 single flight，看您操作兩次後來有想，為何 single flight 不跟 sync cond 一起使用？

所以自己就來試試看，好像可行，程式碼如下[https://go.dev/play/p/eV490rghJor](https://go.dev/play/p/eV490rghJor?fbclid=IwAR39To0eb7c0x_lhKQtnbuqUMehMeY-ZIX00gWB3uxn5udNZhFHsEXRnTyI)請教一下，如果我這樣做，會有什麼問題？

```go
package main

import (
	"fmt"
	"log"
	"sync"
	"sync/atomic"
	"time"
)

type demo struct {
	status atomic.Value
	cond   *sync.Cond
}

var done = false

type demoFunc func(string)

func (d *demo) NewDemo() demoFunc {
	var returnfunc demoFunc
	if d.status.Load().(int) != 0 {
		if d.cond != nil {
			returnfunc = d.read
		}
	}
	if d.status.Load().(int) == 0 {
		d.status.Store(1)
		d.cond = sync.NewCond(&sync.Mutex{})
		returnfunc = d.write
	}
	return returnfunc
}

func (d *demo) read(name string) {
	d.cond.L.Lock()
	for !done {
		fmt.Println(name, "waits")
		d.cond.Wait()
	}
	log.Println(name, "starts reading")
	d.cond.L.Unlock()
}

func (d *demo) write(name string) {
	log.Println(name, "starts writing")
	d.cond.L.Lock()
	done = true
	d.cond.L.Unlock()
	log.Println(name, "wakes all")
	d.cond.Broadcast()
}

func main() {
	demo := &demo{}
	demo.status.Store(0)

	array := [5]string{"A", "B", "C", "D", "E"}

	for i := 0; i < len(array); i++ {
		go func(i int) {
			demo.NewDemo()(array[i])
		}(i)
	}

	time.Sleep(time.Second * 3)
}
```

### 解答

你的 NewDemo() 裡面會有 race

atomic.Value 沒辦法這樣當作 Lock 用除此之外，用 sync cond 只要知道到 Wait() 離開前會自動呼叫 L.Lock() 就沒問題了

## 2022年5月26日

### 問題

您上次提到我程式有 race 的問題，您說的沒錯，是有這個問題如下圖所示，如果紅色區塊為蛋殼的話，上週程式蛋殼有縫，會被協程衝破

改良的程式碼如下[https://go.dev/play/p/bqbPzdMM3kz](https://go.dev/play/p/bqbPzdMM3kz?fbclid=IwAR2VgqKTK-TKKp8z47_ji3LIfGUxjn8g_yd5WRLpQ-vzyoi95CI4j6ueIvM)

```go
package main

import (
	"fmt"
	"log"
	"sync"
	"sync/atomic"
	"time"
)

type demo struct {
	cond   *sync.Cond
	status int32
	count  uint64
}

var done = false

type demoFunc func(string)

func (d *demo) NewDemo() demoFunc {
	atomic.AddUint64(&d.count, 1)
	var returnfunc demoFunc
LOOP:
	exchange := atomic.CompareAndSwapInt32(&d.status, 0, 1)
	if exchange == false {
		if d.cond != nil {
			returnfunc = d.read
		}
		if d.count == 1 {
			goto LOOP
		}
	}
	if exchange == true {
		returnfunc = d.write
	}
	return returnfunc
}

func (d *demo) read(name string) {
	d.cond.L.Lock()
	for !done {
		fmt.Println(name, "waits")
		d.cond.Wait()
	}
	log.Println(name, "starts reading")
	d.cond.L.Unlock()
}

func (d *demo) write(name string) {
	log.Println(name, "starts writing")
	d.cond.L.Lock()
	done = true
	d.cond.L.Unlock()
	log.Println(name, "wakes all")
	d.cond.Broadcast()
}

func main() {
	demo := &demo{}
	demo.cond = sync.NewCond(&sync.Mutex{})

	array := [7]string{"A", "B", "C", "D", "E", "F", "G"}

	for i := 0; i < len(array); i++ {
		go func(i int) {
			demo.NewDemo()(array[i])
		}(i)
	}

	time.Sleep(time.Second * 3)
}
```

改良的方式就是把蛋殼的縫補起來，二行程式碼合成一行

改良效果也不錯，我覺得這個 Single Flight 比較好讀，效能也不錯

請教您一個問題，這個版本的 Slight Flight 和您版本的 Slight Flight 那個比較好？

還有這個 Slight Flight 您覺得還有那些漏洞，謝謝您

<img src="../assets/image-20220602023102601.png" alt="image-20220602023102601" style="zoom:100%;" />

這是執行結果，多個協程程式再執行過程中，只有部份協程等待，所以我才會比較喜歡這個版本的 Single Flight

<img src="../assets/image-20220602023147461.png" alt="image-20220602023147461" style="zoom:100%;" /> 

目前基準測試也壓的過去，上週程式直接 panic

<img src="../assets/image-20220602023253385.png" alt="image-20220602023253385" style="zoom:100%;" />

### 解答

30 行的 d.count 要用 atomic load31 行的 busy wait 最好避免，而且看起來有可能無窮迴圈singleflight 的 cond Locker 可以試試看 RWMutex
