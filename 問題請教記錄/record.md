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

## 2022年5月12日

### 問題

請教您一個問題，如果 gen.go 的程式碼用遞迴函式去處理，您覺得適合嗎？

```go
package main

import "fmt"

// 主程式
func main() {
	// 產生 demo 節點物件
	node := generateDemoNode()
	Build(node)
}

func Build(node *node) {
	node.Accept()
}

// 節點
type node struct {
	FullName string
	Parent   *node
	Child    []*node
}

func (n *node) Accept() (*node, bool) {
	v := visitor{}
	newNode, skipChildren := v.Enter(n)
	if skipChildren {
		return v.Leave(newNode)
	}

	for _, next := range n.Child {
		next.Accept()
	}

	return v.Leave(newNode)
}

// 參考者
type visitor struct {
	parent *node
}

func (v *visitor) Enter(in *node) (*node, bool) {
	for i := 0; i < len(in.Child); i++ {
		in.Child[i].Parent = in
	}
	fmt.Println("type " + in.FullName + " Completed")

	return in, false
}

func (v *visitor) Leave(in *node) (*node, bool) {
	if in != nil {
		if in.Parent != nil {
			fmt.Println(in.Parent.FullName + " -> " + in.FullName)
		}
		return in, false
	}

	return in, true
}

// 產生 demo 節點物件
func generateDemoNode() *node {
	setUserNode := &node{
		FullName: "AclSetuser",
		Parent:   nil,
		Child:    nil,
	}

	setRuleNode := &node{
		FullName: "AclSetuserRule",
		Parent:   nil,
		Child:    nil,
	}

	setUserNameNode := &node{
		FullName: "AclSetuserUsername",
		Parent:   nil,
		Child:    nil,
	}

	a := &node{FullName: "A"}
	b := &node{FullName: "B"}
	c := &node{FullName: "C"}
	setUserNode.Child = append(setUserNode.Child, setUserNameNode, a, b, c)
	setUserNameNode.Child = append(setUserNameNode.Child, setRuleNode)

	return setUserNode
}
```

後來我看了程式碼，覺得或許可以用 AST 遞迴函式去處理

有時會常看到 AST 遞迴函式，卡西哥就貼過一次

我看過二次了，AST 遞迴函式會有三個地方可以控制，彈性我覺得很夠，分別為

進入節點的 Enter 函式

中間處理函式 Accept 函式

離開節點的 Leave 函式

方向也很有彈性，可以由 Parent 節點進入 Child 節點去操作，也可以反向，由 Child 節點進入 Parent 節點

另外，如果是產生程式碼的程式，最後一定會有吐檔動作，產生程式碼

但是 AST 遞迴函式 可以把吐檔工作都分配到各個節點，每個節點會有寫檔動作

程式的執行結果
type AclSetuser Completed
type AclSetuserUsername Completed
type AclSetuserRule Completed
AclSetuserUsername -> AclSetuserRule
AclSetuser -> AclSetuserUsername
type A Completed
AclSetuser -> A
type B Completed
AclSetuser -> B
type C Completed
AclSetuser -> C
(意思為 AclSetuser 物件會有方法產生 AclSetuserUsername 物件
而 AclSetuserUsername 物件 也會有方法產生 AclSetuserRule 物件)

AST 遞迴函式對於個性比較保守的人會比較喜歡使用，因為程式碼彈性很高

而且它有自動補齊節點資料的功能，當我發現節點參數不足時，就在 Enter 進入函式時，就開始去修正或補齊節點資料

那請問如果用這方法去寫 gen.go，您覺得可能會遇到什麼問題？謝謝

<img src="../assets/image-20220901064105188.png" alt="image-20220901064105188" style="zoom:100%;" /> 

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

## 2022年6月2日

### 測試中

後來才發現用 sync 包可以寫三個 single flight 版本，版本1 為接近您的版本，我覺得您的版本比較好

版本3 是我新寫的版本，光想到版本3 的上鎖，我就預期效能會很不好，光聽您提到用 RWMutex 我就知道問題出在那裡

版本1 使用 sync wait group ，程式碼如下

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
	wait   sync.WaitGroup
	status int32
	done   bool
	value  int
}

type demoFunc func(string)

func (d *demo) NewDemo() demoFunc {
	if exchange := atomic.CompareAndSwapInt32(&d.status, 0, 1); exchange == true {
		return d.write
	}
	return d.read
}

func (d *demo) read(name string) {
	if !d.done {
		fmt.Println(name, "waits")
		d.wait.Wait()
	}
	log.Println(name, "starts reading with lock", d.value)
}

func (d *demo) write(name string) {
	log.Println(name, "starts writing")
	d.value = 1
	d.done = true
	log.Println(name, "wakes all", d.value)
	d.wait.Done()
}

func main() {
	demo := &demo{}
	demo.wait.Add(1)

	array := [26]string{"A", "B", "C", "D", "E", "F", "G", "H", "I", "J", "K", "L", "M", "N", "O", "P", "Q", "R", "S", "T", "U", "V", "W", "X", "Y", "Z"}

	for i := 0; i < len(array); i++ {
		go func(i int) {
			demo.NewDemo()(array[i])
		}(i)
	}

	time.Sleep(time.Second * 3)
}

```

版本2 使用 sync rwmutex ，程式碼如下

````go
package main

import (
	"fmt"
	"log"
	"sync"
	"sync/atomic"
	"time"
)

type demo struct {
	rwLock sync.RWMutex
	status int32
	done   bool
	value  int
}

type demoFunc func(string)

func (d *demo) NewDemo() demoFunc {
	if exchange := atomic.CompareAndSwapInt32(&d.status, 0, 1); exchange == true {
		return d.write
	}
	return d.read
}

func (d *demo) read(name string) {
	d.rwLock.RLock()
	if !d.done {
		fmt.Println(name, "waits")
	}
	log.Println(name, "starts reading with lock", d.value)
	d.rwLock.RLock()
}

func (d *demo) write(name string) {
	log.Println(name, "starts writing")
	// d.rwLock.Lock()
	d.value = 1
	d.done = true
	log.Println(name, "wakes all")
	d.rwLock.Unlock()
}

func main() {
	demo := &demo{}
	demo.rwLock = sync.RWMutex{}
	demo.rwLock.Lock()

	array := [26]string{"A", "B", "C", "D", "E", "F", "G", "H", "I", "J", "K", "L", "M", "N", "O", "P", "Q", "R", "S", "T", "U", "V", "W", "X", "Y", "Z"}

	for i := 0; i < len(array); i++ {
		go func(i int) {
			demo.NewDemo()(array[i])
		}(i)
	}

	time.Sleep(time.Second * 3)
}
````

版本3 使用 sync cond ，程式碼如下

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
	done   bool
	value  int
}

type demoFunc func(string)

func (d *demo) NewDemo() demoFunc {
	if exchange := atomic.CompareAndSwapInt32(&d.status, 0, 1); exchange == true {
		return d.write
	}
	return d.read
}

func (d *demo) read(name string) {
	switch d.done {
	case true:
		log.Println(name, "starts reading without lock", d.value)
	case false:
		d.cond.L.Lock()
		for !d.done {
			fmt.Println(name, "waits")
			d.cond.Wait()
		}
		log.Println(name, "starts reading with lock", d.value)
		d.cond.L.Unlock()
	}
}

func (d *demo) write(name string) {
	log.Println(name, "starts writing")
	d.cond.L.Lock()
	d.value = 1
	d.done = true
	d.cond.L.Unlock()
	log.Println(name, "wakes all")
	d.cond.Broadcast()
}

func main() {
	demo := &demo{}
	demo.cond = sync.NewCond(&sync.Mutex{})

	array := [26]string{"A", "B", "C", "D", "E", "F", "G", "H", "I", "J", "K", "L", "M", "N", "O", "P", "Q", "R", "S", "T", "U", "V", "W", "X", "Y", "Z"}

	for i := 0; i < len(array); i++ {
		go func(i int) {
			demo.NewDemo()(array[i])
		}(i)
	}

	time.Sleep(time.Second * 3)
}
```

這三組程式會送去做基準測試，會把 runlevel 下降去壓測，去除作業系統圖形介面干擾，不過我預期版本3效能不會太好，測了就知道

做測試時，程式碼還要在小修改，慢慢把協程數量一直加大去壓測三組程式

## 2022年6月9日

我用三個 single flight 版本去進行基準測試測試環境為 

Linux 核心版本為 4.19.0-16-amd64，無 GUI 介面

Go 版本為 1.18.3

CPU 為 Intel i5-8250U ( 8 ) @ 3.400GHz

Memory 為 31870MiB

測試結果為waitGroup 版本效能最好，也是 rueidis 的版本，這符合預期，因為能少上鎖就少上鎖cond 版本效能為次好，也是我的餿主意，這不符合預期，因為我預期這版本跑出來的數據會很慘，結果表現還能看

rwMux 版本效能為最不好，這也不符合預期，沒想到是三組數據最不好的

進行測試時，程式要改寫，如下

waitGroup 版本

```go
package main

import (
	"fmt"
	"sync"
	"sync/atomic"
)

var total = 1000

var count sync.WaitGroup

type demo struct {
	wait   sync.WaitGroup
	status int32
	done   bool
	value  int
}

type demoFunc func(int)

func (d *demo) NewDemo() demoFunc {
	if exchange := atomic.CompareAndSwapInt32(&d.status, 0, 1); exchange == true {
		return d.write
	}
	return d.read
}

func (d *demo) read(num int) {
	if !d.done {
		d.wait.Wait()
	}
	if d.value != 1 {
		panic("error" + fmt.Sprintln(d.value))
	}
	count.Done()
}

func (d *demo) write(num int) {
	d.value = 1
	d.done = true
	d.wait.Done()
	count.Done()
}

func main() {
	count.Add(total)

	demo := &demo{}
	demo.wait.Add(1)

	for i := 0; i < total; i++ {
		go func(i int) {
			demo.NewDemo()(i)
		}(i)
	}

	count.Wait()
}

// >>>>> >>>>> >>>>> >>>>> >>>>>

/*
package main

import (
	"testing"
)

func Benchmark_Main(b *testing.B) {
	for i := 0; i < b.N; i++ {
		count.Add(total)

		demo := &demo{}
		demo.wait.Add(1)

		for i := 0; i < total; i++ {
			go func(i int) {
				demo.NewDemo()(i)
			}(i)
		}

		count.Wait()
	}
}
*/
```

rwMux 版本

```go
package main

import (
	"fmt"
	"sync"
	"sync/atomic"
)

var total = 1000

var count sync.WaitGroup

type demo struct {
	rwLock sync.RWMutex
	status int32
	done   bool
	value  int
}

type demoFunc func(int)

func (d *demo) NewDemo() demoFunc {
	if exchange := atomic.CompareAndSwapInt32(&d.status, 0, 1); exchange == true {
		return d.write
	}
	return d.read
}

func (d *demo) read(num int) {
	d.rwLock.RLock()
	if d.value != 1 {
		panic("error" + fmt.Sprintln(d.value))
	}
	d.rwLock.RLock()
	count.Done()
}

func (d *demo) write(num int) {
	d.value = 1
	d.done = true
	d.rwLock.Unlock()
	count.Done()
}

func main() {
	count.Add(total)

	demo := &demo{}
	demo.rwLock = sync.RWMutex{}
	demo.rwLock.Lock()

	for i := 0; i < total; i++ {
		go func(i int) {
			demo.NewDemo()(i)
		}(i)
	}

	count.Wait()
}

// >>>>> >>>>> >>>>> >>>>> >>>>> 

/*
package main

import (
	"sync"
	"testing"
)

func Benchmark_Main(b *testing.B) {
	for i := 0; i < b.N; i++ {
		count.Add(total)

		demo := &demo{}
		demo.rwLock = sync.RWMutex{}
		demo.rwLock.Lock()

		for i := 0; i < total; i++ {
			go func(i int) {
				demo.NewDemo()(i)
			}(i)
		}

		count.Wait()
	}
}
*/
```

cond 版本

```go
package main

import (
	"fmt"
	"sync"
	"sync/atomic"
)

var total = 1000

var count sync.WaitGroup

type demo struct {
	cond   *sync.Cond
	status int32
	done   bool
	value  int
}

type demoFunc func(int)

func (d *demo) NewDemo() demoFunc {
	if exchange := atomic.CompareAndSwapInt32(&d.status, 0, 1); exchange == true {
		return d.write
	}
	return d.read
}

func (d *demo) read(num int) {
	switch d.done {
	case true:
		if d.value != 1 {
			panic("error" + fmt.Sprintln(d.value))
		}
		count.Done()
	case false:
		d.cond.L.Lock()
		for !d.done {
			d.cond.Wait()
		}
		if d.value != 1 {
			panic("error" + fmt.Sprintln(d.value))
		}
		d.cond.L.Unlock()
		count.Done()
	}
}

func (d *demo) write(num int) {
	d.cond.L.Lock()
	d.value = 1
	d.done = true
	d.cond.L.Unlock()
	d.cond.Broadcast()
	count.Done()
}

/*
func main() {
	count.Add(total)

	demo := &demo{}
	demo.cond = sync.NewCond(&sync.Mutex{})

	for i := 0; i < total; i++ {
		go func(i int) {
			demo.NewDemo()(i)
		}(i)
	}

	count.Wait()
}
*/
```

結果我是覺得不合預期，您覺得呢？

<img src="../assets/image-20220714053507956.png" alt="image-20220714053507956" style="zoom:100%;" />

### 解答

waitGroup 版本中第 30 行讀取 d.done 是會 race 的，要直接 d.wait.Wait()

單純用 rwMux 的版本因為可能很久才會輪到 write goroutine 執行，所以比較不好應該是合理

cond 版本第 30 行讀取 d.done 一樣是會 race 的。你可以把 d.cond.L.Lock() 底下接到 rwMux.RLock

## 2022年6月16日和23日

1 這次跑基準測試，比上星期小心了，注意機器的散熱，不然數據會跳動

2 要測 write 協程何時結束工作，只要修改二行程式碼，當 write 協程結束工作時，主程式也就跟著結束工作，是可以測出

3 我是用上鎖的程度去判斷那一個版本的效能最好，我認為上鎖的程度為 cond 版本 \> RWMux 版本 > waitGroup 版本，上愈少鎖效能愈好，我預期 RWMux 版本效能應不是最差的

最後上星期您說

單純用 rwMux 的版本因為可能很久才會輪到 write goroutine 執行，所以比較不好應該是合理

根據基準測試的結果，協程數量為 100000，和您說的相符合，不過我不是很了解原因，您能說明更多細節嗎？謝謝

<img src="../assets/image-20220714054414384.png" alt="image-20220714054414384" style="zoom:100%;" />

我假設以下原因會有這些現象，您看看有沒有道理？

問題一，數據為何會跳動？

假設一：和機器散熱有關，每次執行基準測試時，要先停15秒，等機器散熱完畢

假設二：和調度器 scheduler 有關，要看調度器何時把 write 協調排進 queue 裡，還是一開始就被處理掉了

問題二，數據為何 cond 的效能會比 rwMux 還要好？

假設：因為 cond 底層的 lock 可以用 mutex 或 rwMux 去實作，而我的例子是用 mutex

所以先直接比較 mutex 和 rwMux 的性能就可以

在 write 協程很早完成工作時，mutex 效能會比 rwMux 好

在 write 協程很晚很晚才完成工作時，rwMux 效能會比 mutex 好

因為 mutex 的 lock 可以用 channel 去模擬

如果 write 協程很早完成工作時，channel 就被關閉了，這時 所有 read 協程得知 channel 關閉時，就很快的結束工作，這時效能會很好

### 解答

建議要先把上次程式中的 race 修掉再重新測試一下

### 回應

好的,謝謝您的建議,我再仔細檢查,不過能通過十萬協程的壓測,也有可能不是 race ，不然早就 panic 了,反正我再仔細檢查,謝謝您

## 2022年6月30日

### 測試中

您說的沒錯，第三十行的 d.done 是 race

用 golang 的 race 偵測去檢查，這是 race

我有另一個觀點，如果就算這一行發生 race，也沒有關係

因為 d.done 預設值為 false，所以 !d.done 的值預設為 true

所以這整段來看，預設就是進行等待

```go
if !d.done {
	d.wait.Wait()
}
```

當 race 發生時，d.done 突然被改成 true，但這一段讀取 d.done 的值為 false，這個值是錯的，沒有差，因為協程還是在等待的狀態

也就是這個原因，golang 的 race 偵測指出程式有問題，但是去跑基準測試時反而沒有問題

這是因為 golang 的 race 偵測只檢查 d.done 是不是同時寫入讀取，但是它不管整個程式邏輯

<img src="../assets/image-20220714055628083.png" alt="image-20220714055628083" style="zoom:100%;" />

再來，這次實驗其實有個盲點，在跑測試時會有一個問題，編譯器會進行優化，編譯器在優化時，會影響實驗數據所以執行基準測試時，不只機器溫度要控制，連編譯器優化的功能也要關閉，因為這些都會影響跑出來的數據

以這次例子來說，編譯器有沒有進行優化我還不知道，我就急著幫整個程式進行優化，

```go
if !d.done {
  d.wait.Wait()
}
```

這一段其實是我為了性能考量所寫的，因為能減少等待就減少等待，但是這會影響跑出來的數據

總而言之，我同意您的說法，第三十行的 d.done 要去掉，為了數據的客觀性，謝謝

## 2022年7月7日

### 問題

1 謝謝您，您說的是對的，當把所有 第 30 行讀取 d.done 都移除後，cond 的版本效能會下降

就跟一開始預期的結果一樣，wait 效能 > rwmux > cond

2 前一個版本效能會不符合預期是因為我都在用 第 30 行讀取 d.done 去躲開上鎖

3 最後結論是能像 wait 版本都不上鎖的話，效能是最好的，其他的話就要看寫法了，有的寫法是 rwmux 效能較好，有的寫法是 cond 效能較好

4 前一個版本就算有 race ，結果也是正確的，因為只要答案是錯的，程式會直接 panicif d.value != 1 {    panic("error" + fmt.Sprintln(d.value))}

5 圖中的數據結果單位為 ns/op

<img src="../assets/image-20220714060606116.png" alt="image-20220714060606116" style="zoom:80%;" /> 

### 解答

可以看一下文件中的 Incorrect synchronization 範例 [https://tip.golang.org/ref/mem](https://tip.golang.org/ref/mem?fbclid=IwAR2jbGDfSNp4kfSvDA5qTOIYL1R1V0UWp4BQFkxp45Dv7cRRAiLwi9I4wQE)

## 2022年8月4日

### 問題

我為了了解 Redis 的叢集，把整個 Redis Cluster 的封包倒出來看

(1) Redis Cluser 是基於 udp 和 tcp，我先過濾 udp 封包，只剩 tcp 的封包

(2) 但是會發現一個很不正常的現在，節點之間會不停的用 tcp 去進行連線，問題是節點的資料都已經收斂了，一直用 tcp 連線不是在浪費連線

這現象不是很合理？

(3) 直接去查 Gossip 協定的模型，有提供兩個公式，全部是以病毒傳播進行理解

(4) 第一個公式 Pi+1 = pi (1-1/n) ^ n(1-pi)，是指在病毒傳播下的存活率

有多少次方代表有多少人 n(1-p) 中毒，成為傳播者

(1-1/n) 為和人接觸的機率，除了自己以外

p1 為現在的存活率

整個公式看起來很合理，這是真實世界的公式，這公式允計病毒傳播可以失誤，而且收斂速度慢很多

和大家想像的病毒傳播速度差很多，因為這個公式是以 RT 值為1 時，去進行計算模型應以這個公式訂為下界

(5) 第二個公式 pi+1 = pi * pi ，這個公式我不能理解，但可以在紙上計算，這個公式收斂的速度很快

有可能是作者想像中的病毒傳播的存活機率公式

上一個公式病毒還可以傳播失敗，要達到這個公式的收斂速度，病毒傳播不能失敗但現實的狀況是不可能，因為可以對方早就中毒了

(6) 當初在實作 gossip 協定的人，並無法達到第二個公式的收斂速度，那怎麼辨？就用大量的連線去接進第二個公式

所以今天或者是未來，Redis Cluser 一定會有節點數的限制，很難在突破，如果要突破，就是跟模型進行挑戰

(7) 請教您有沒有什麼方法可以擴展節點數，而且能維持一樣的收斂速度？我是偏向不挑戰模型，謝謝

## 2022年8月18日

### 問題

Redis Cluster 的 Gossip 協定做的很精簡，因為 Redis 不能跨區跨分區，最好 Cluster 就在一個區網內，不然我找不到合理的解釋了

<img src="../assets/image-20220825041631496.png" alt="image-20220825041631496" style="zoom:100%;" /> 

[參考文件](https://l.facebook.com/l.php?u=http%3A%2F%2Fbitsavers.trailing-edge.com%2Fpdf%2Fxerox%2Fparc%2FtechReports%2FCSL-89-1_Epidemic_Algorithms_for_Replicated_Database_Maintenance.pdf%3Ffbclid%3DIwAR1QfLQPOEYalvpKlYDs5_tX6RlU6iQnLWKe8WbfotXejF4iDawEfzRsf1w&h=AT3WA3i2fsNPHtw0pkJaziojdDr7_53nsbGMlpDx_2FezIFcGQnf83fe-cn8AtC2VoU9baomcp4BA2EpqtYlkqgkr7yNscMteu38E7virWMw8cl3gCxgaFYbLc2c&__tn__=R]-R&c[0]=AT2PH43SMmfg_cya5iCMIKM4Gki_BKm9Td1jIBZtes8YOGknnDRtoeoSZMYUJdPheF5zXgZsQx-lSsOh7sZFY9hmmt0juQNqdB9Rsq8wGUcHndXXvDFoNmDAxuamuCS-QOZ44miRQ0jRpWH4j6HKr-NUlKQfsed16PFNZQ_YMfPqRTXVXBfg3RK2R8trGq_DcauFIvWcLSF4mXj_lQGyvIKrKkMNFse3zTjSqc0iremXbG9evaYD)

Gossip 協定有二種模式，一為用 tcp 去實現 Anti-entropy，用 udp 去實現 Rumor，今天 Redis 的文件沒特別說明 Anti-entropy ，Rumor 又做在心跳裡，Redis 的 Gossip 機制太精簡了

<img src="../assets/image-20220825042152440.png" alt="image-20220825042152440" style="zoom:100%;" /> 

我的之前的假設應該算是合理的

<img src="../assets/image-20220825042306681.png" alt="image-20220825042306681" style="zoom:100%;" /> 
