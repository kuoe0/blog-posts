---
layout:  post
title:   "第一次面試 Google 就不上手"
date:    2014-04-15
tags:    ["Google", "algorithm | 演算法"]
feature:
    photo:       true
    creator:     "Marcin Wichary"
    url:         "https://www.flickr.com/photos/mwichary/3374813066"
    license:     "CC 2.0"
    license_url: "https://creativecommons.org/licenses/by/2.0/"
---

由於沒有簽 NDA，所以應該可以分享吧？！若有不妥的部分，煩請告知。

在我大三的時候，曾經收到 Google 美國總部的面試邀約。不過，想說可能找錯人還怎樣，Google 那邊竟然不知道我還在就讀大學，在信件的多次往返後，才知道我還只是個學生。最後，就請我先完成學業囉。然而卻在我碩二快畢業這年竟然又接到 Google 台灣的面試邀約！雖然已經接受了 Mozilla 的 offer，但還是想去嘗試一下，不管最後如何，假設錄取了再來頭痛就好，如果沒錄取，就當經驗磨練一下也不錯。

Google 一開始會先請 HR 打電話來確認一下意願，也會詢問一些近期的生涯規劃，以及公司內部的大略方向等等。若確定有意願就會開始安排面試時間了，此外，Google 的 HR 也會透過 email 寄送一些面試相關的參考資料。

然後看標題就知道沒被錄取了...而且我一關都沒過，在電話面試就慘遭敗北。所以我也只能分享該次的電話面試。被淘汰後，我也再次詢問了一下是否能告知面試中是否有什麼沒注意的地方，得到的答覆是「**面試官希望看到更好的程式編寫表現**」。我想，意思就是我程式寫的有待加強。

在面試前，會先收到一封確認信，裡頭會有日期及時間，以及一個 Google 文件的連結。在開始面試前，要記得打開那個連結，所有的問題基本上就是在該文件上進行 coding 解決。整個面試過程都是以中文進行，當初整個很擔心是英文。接下來就是我在這次面試中被測驗的兩個題目。

**設計一個樹的序列化與解序列化的演算法**

剛好前幾天在寫題目時看到一種二元樹的序列化結果，該結果是利用 BFS 而得到的，當下也想了一下該怎麼解序列化，所以這題沒有特別多想就直接講出了該方法。

假設有一棵樹長得像下面這樣：

```
  1
 / \
2   3
   / 
  7  
```

其序列化結果為 `1,2,3,#,#,7,#,#,#`。

序列化流程如下：

1. 從 root 放進 queue 中
2. 若 queue 不為空，從 queue 中取出一個節點並從第 4 步繼續
3. 若 queue 為空，該演算法結束
4. 若該節點不為空節點，輸出該節點內容並從第 5 步繼續
5. 若該節點為空節點，輸出 `#` 並從第 8 步繼續
6. 將其左節點放進 queue 中
7. 將其右節點放進 queue 中
8. 回到第 2 步

序列化的部分我僅僅用口頭的方式陳述，並沒有實際將程式碼寫出來，因為面試官說他大概知道我的做法，於是我就從解序列化開始寫起。

解序列化流程如下（錯誤版）：

1. 先利用 parser 將每個元素提取出來依序放入一個 queue (q1) 中
2. 宣告一個內容未知的節點作為 root，並配上一個空節點作為其父節點放入 queue (q2) 中3. 
3. 若是 queue (q2) 不為空，取出一個節點對，並從第 5 步繼續
4. 若是 queue (q2) 為空，結束演算法
5. 若是 queue (q1) 為空或是下一個元素為 `#`，從第 7 步繼續
6. 若是 queue (q1) 不為空，從第 9 步繼續
7. 將該節點對中的父節點指向該節點對中的節點的連結改指向空指標
8. 將該節點對中的節點空間釋放，並從第 3 步繼續
9. 從 queue (q1) 中取出一個元素
10. 將該節點對中的節點內容設定為該元素
11. 宣告一個新節點，並將當前節點對中的節點的左節點指向該節點
12. 將該新節點與當前節點對中的節點組合成新的節點對放入 queue (q2) 中
13. 再宣告一個新節點，並將當前節點對中的節點的右節點指向該節點
14. 將該新節點與當前節點對中的節點組合成新的節點對放入 queue (q2) 中
15. 回到第 3 步

程式碼如下：


```c++
str = "1,2,3,#,#,7,#,#,#";
deque<int> deq = parse(str);
queue<int> que;
// first = now, second = parent;
Node *root = new Node();
que.push(pair<Node*, Node*>(root, NULL));

while (!que.empty()) {

	now = que.front().first;
	parent = que.front().second;
	que.pop();
		

	if (deque.front() == ‘#’ || deq.empty()) {
		if (parent->left == now) {
			delete now;
			parent->left = NULL;
		}
		if (parent->right == now) {
			delete now;
		parent->right = NULL;
		}
		continue;
	}

	now->val = deq.front();
	deq.pop_front();

	now->left = new Node();
	now->right = new Node();
	que.push(now->left, now);
	que.push(now->right, now);
}
```

也難怪會被說程式有待加強，上面的程式碼根本漏洞一堆。一堆變數的宣告本身就有問題，像是 `queue<int>` 明明就應該宣告成 `queue< pair<Node*, Node*> >` 才對。還有 `deque` 是資料型別，`deq` 才是變數名稱...。但不知道這邊扣分了多少，畢竟面試官說不用寫的太詳細沒關係，大致能了解就好。

不過，除了一些 typo 外，我演算法其實有 bug...，就是在第 5 步中，若是 queue (q1) 下一個元素是 `#` 時，忘了把 `#` 給拿出來！這樣會照成遇到一次 `#` 後，接下來就一直把後面的節點視為空指標了。

修正一下流程：

1. 先利用 parser 將每個元素提取出來依序放入一個 queue (q1) 中
2. 宣告一個內容未知的節點作為 root，並配上一個空節點作為其父節點放入 queue (q2) 中3. 
3. 若是 queue (q2) 不為空，取出一個節點對，並從第 5 步繼續
4. 若是 queue (q2) 為空，結束演算法
5. 若是 queue (q1) 為空，從第 7 步繼續
6. 若是 queue (q1) 不為空，從第 9 步繼續
7. 將該節點對中的父節點指向該節點對中的節點的連結改指向空指標
8. 將該節點對中的節點空間釋放，並從第 3 步繼續
9. 從 queue (q1) 中取出一個元素
10. 若該元素為 `#` 時，從第 7 步繼續
11. 將該節點對中的節點內容設定為該元素
12. 宣告一個新節點，並將當前節點對中的節點的左節點指向該節點
13. 將該新節點與當前節點對中的節點組合成新的節點對放入 queue (q2) 中
14. 再宣告一個新節點，並將當前節點對中的節點的右節點指向該節點
15. 將該新節點與當前節點對中的節點組合成新的節點對放入 queue (q2) 中
16. 回到第 3 步

修正後的程式碼如下：

```c++
string str = "1,2,3,#,#,7,#,#,#";
deque<int> deq = parse(str);
queue< pair<Node*, Node*> > que;
// first = now, second = parent;
Node *root = new Node();
que.push(pair<Node*, Node*>(root, NULL));

while (!que.empty()) {

	Node *now = que.front().first;
	Node *parent = que.front().second;
	que.pop();

	if (deq.front() == ‘#’ || deq.empty()) {
		deq.pop_front();
		if (parent->left == now) {
			delete now;
			parent->left = NULL;
		}
		if (parent->right == now) {
			delete now;
		parent->right = NULL;
		}
		continue;
	}

	now->val = deq.front();
	deq.pop_front();

	now->left = new Node();
	now->right = new Node();
	que.push(now->left, now);
	que.push(now->right, now);
}
```

後來又想了想還是直接尋訪存放元素的列表好了，看起來會比較簡潔些，進一步修改後的程式碼如下：

```c++
string str = "1,2,3,#,#,7,#,#,#";
deque<string> deq = parse(str);
queue< pair<Node*, Node*> > que;
// first = now, second = parent;
Node *root = new Node();
que.push(pair<Node*, Node*>(root, NULL));

for (auto elem: elements) {
	Node *now = que.front().first;
	Node *parent = que.front().second;
	que.pop();

	if (elem != "#") {
		int val = stoi(elem);
		now->val = val;
		now->left = new Node(0);
		now->right = new Node(0);
		que.push(pair<Node*, Node*>(now->left, now));
		que.push(pair<Node*, Node*>(now->right, now));
	}
	else {
		if (parent->left == now) parent->left = NULL;
		if (parent->right == now) parent->right = NULL;

		delete now;
	}
}
```

**前序運算式轉為後續運算式**

題目應該很清楚了，直接看例子吧！

```
input: + 1 2
output: 1 2 +
input: + + 1 2 + 1 2
output: 1 2 + 1 2 + +
```

這一題莫名一開始卡了很久，有點想成要計算出前序運算式的結果了，導致一開始程式碼有點混亂。後來靜下心來思考一下，想出了我個人覺得滿漂亮的遞迴解。其原理是，每個運算子都會有兩個運算元。而運算元不一定是一個數字，也可能是一個運算式，所以就在遞迴兩次來找出該運算元。當遇到數字時，可以直接回傳該運算元。但若是遇到運算子時，則表示其會是一個完整個運算式，所以可以繼續遞迴下去進行，直到完整的運算式出現再回傳。

流程如下：

1. 進入遞迴函式後，檢查剩餘元素中的第一個元素
2. 若該元素為運算子，從第 4 步開始
3. 若該元素為數字，
4. 將該元素取出，並暫存起來
5. 再一次進入遞迴函式，以取得第一個運算元
6. 再一次進入遞迴函式，以取得第二個運算元
7. 依序將第一個運算元與第二個運算元的內容放入結果中
8. 再將暫存的運算子放入結果中，並離開函式
9. 將該元素取出放入結果中，並離開函式

程式碼如下：

```c++
void pre2post(deque<element> &pre, deque<element> &post) {
	if (isOperator(pre.front())) {
		element tmp = pre.front();
		pre.pop_front();
		
		deque<element> op1, op2;
		pre2post(pre, op1);
		pre2post(pre, op2);
		
		post.insert(end(post), begin(op1), end(op1));
		post.insert(end(post), begin(op2), end(op2));
		
		post.push_back(tmp);
		return;
	}
	if (isDigit(pre.front())) {
		post.push_back(pre.front());
		pre.pop_front();
	}
}
```

我個人是覺得這方法滿漂亮的，我想可能被評為表現不佳的原因可能是一開始卡太久了，而且還想錯方向使得我的行為表現的非常混亂。這個題目可能花了我 20 分鐘有吧，說不定因為前後表現差異太大被認為是上網搜尋資料或是作弊之類的？！

以上就是這次 Google 電話面試的內容，深深覺得太久沒碰演算法的問題，腦袋變得好容易打結...。還有最近太少寫 C++，幾乎都在寫 Python，也使得自己變得不熟練了。還要多努力才有機會被 Google 肯定啊！
