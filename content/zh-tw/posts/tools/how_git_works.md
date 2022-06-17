+++
title = "Git 是如何運作的"
author = "Allen Hsieh"
description = "使用 Git 當作 Soruce Control 也有一段時間了，但以前都只是基本的了解 Git 基本使用，並沒有深入了解 Git 後面實作原理。最近在 Pluralsight 上看到 Paolo Perrotta 講解 Git 是如何運作的，覺得講得不錯~這邊也算是將這次吸收到的知識做個筆記。"
featured = true
categories = ["TOOLS"]
tags = [
    "Git"
]
date = "2022-03-02"
aliases = ["how_git_works"]
images = ["images/git.png"]
+++

使用 Git 當作 Soruce Control 也有一段時間了，但以前都只是基本的了解 Git 基本使用，並沒有深入了解 Git 後面實作原理。最近在 Pluralsight 上看到 Paolo Perrotta 講解 Git 是如何運作的，覺得講得不錯~ 這邊也算是將這次吸收到的知識做個筆記。


## Git Databases
---
Git init 是初始化 Git 的 Databases 或者又稱 Repository ，而 Git 的所有資料都存取在 .git 的目錄下。當執行 git init 後，可以看到 .git 資料夾產生

```Bash
.git
├── HEAD
├── config
├── description
├── hooks
│   ├── applypatch-msg.sample
│   ├── commit-msg.sample
│   ├── fsmonitor-watchman.sample
│   ├── post-update.sample
│   ├── pre-applypatch.sample
│   ├── pre-commit.sample
│   ├── pre-merge-commit.sample
│   ├── pre-push.sample
│   ├── pre-rebase.sample
│   ├── pre-receive.sample
│   ├── prepare-commit-msg.sample
│   ├── push-to-checkout.sample
│   └── update.sample
├── info
│   └── exclude
├── objects
│   ├── info
│   └── pack
└── refs
    ├── heads
        └── tags
```

## 內容追蹤
---
有執行過 "man git" 的話，可以看到 Name 那邊寫著 "git - the stupid content tracker"。這也是 Git 最核心的價值，可以讓我們知道所有檔案的修改紀錄，但這是怎麼做到的呢？

### Hash-Obejct 
---
Git 使用 SHA1 來 hash 檔案內容，並透過雜湊值的結果，來建立起 Database，以下面為例子

```Bash
$ echo "hello world" | git hash-object --stdin
3b18e512dba79e4c8300dd08aeb37f8e728b8dad
```

### Commit
---
我們可以看到 "hello world" 的雜湊值是 3b18e512dba79e4c8300dd08aeb37f8e728b8dad。

這時我們在 local 產生一個 helloWorld.txt ，並 commit "first file" 進去。

這時我們看看 .git/objects，這時會發現多了一些 Folder

```Bash
tree .git/objects
.git/objects
├── 3b
│   └── 18e512dba79e4c8300dd08aeb37f8e728b8dad
├── 48
│   └── 971f01848a55ac80698bd8c8b999b431f14723
├── a6
│   └── 10f3aeab960b6dd657d6f371419954e923bcdf
├── info
└── pack
```

### Log
---
這時執行 Git Log 可以看到我們的 Commit 紀錄

```bash
$ git log
commit 48971f01848a55ac80698bd8c8b999b431f14723 (HEAD -> master)
Author: Chang Lin Hsieh <allen@ChangdeMacBook-Pro.local>
Date:   Tue Feb 22 22:00:22 2022 +0800

    first file
```

在 Commit 後面有一個雜湊值是 "48971f01848a55ac80698bd8c8b999b431f14723" ，不知道大家有沒有發現這個數字在哪裡見過？

答案就是在剛剛的 .git/objects 裡面，前兩位數 "48" 是資料夾的名字，剩餘的是檔案名稱。

這時我們來看看檔案裡面有什麼吧

### Cat-File
---
如果大家直接去讀 objects 下面的資料，會發現裡面是一堆不可讀亂碼！檔案的內容是有經過特殊處理過的，但這時可以使用 cat-file 來讀取裡面的資料

```Bash
# 使用方式為  git cat-file options <雜湊值>
# optins 這次這邊只會 Demo -t 顯示物件類型 -p 物件內容。更多資訊可以直接執行 git cat-file，會有使用教學

$ git cat-file -t 48971f01848a55ac80698bd8c8b999b431f14723
commit

$ git cat-file -p 48971f01848a55ac80698bd8c8b999b431f14723
tree a610f3aeab960b6dd657d6f371419954e923bcdf
author Chang Lin Hsieh <allen@ChangdeMacBook-Pro.local> 1645538422 +0800
committer Chang Lin Hsieh <allen@ChangdeMacBook-Pro.local> 1645538422 +0800

first file
```

* line5 : 我們可以看到 "48971f01848a55ac80698bd8c8b999b431f14723" 這個物件代表的是 commit 
* line8 : 我們可以看到雜湊值前面有個 Tree，是代表這個類型。 Tree 記錄該目錄下有哪些檔案(blob) 和 Tree
* line9-10: 是作者和 Commit 人的資訊
* line12: 是 commit 的訊息

那接著來看看 line8 tree 的內容吧
```Bash
$ git cat-file -p a610f3aeab960b6dd657d6f371419954e923bcdf
100644 blob 3b18e512dba79e4c8300dd08aeb37f8e728b8dad	helloWorld.txt
```

這邊可以看到 blob 這個類型，代表紀錄檔案內容，而最後面是檔案名稱。

大家可以仔細看，helloWorld的雜湊值會跟前面 Hash-Obejct 使用的範例是一樣的，因為 Git 是拿檔案裡面的內容去做 Hash。同樣的值 Hash 出來的雜湊值需要是一樣的。

```Bash
$ git cat-file -p 3b18e512dba79e4c8300dd08aeb37f8e728b8dad
hello world
```

### 版本實現
---
我們現在在 Local 建立一個 Folder 叫 I_AM_FOLDER 並在裡面建立一個 helloWorld2.txt 但內容跟 helloWorld.txt 一樣，並 commit 看看會發生什麼事情。

目前資料結構
```Bash
$ tree
.
├── I_AM_FOLDER
│   └── helloWorld2.txt
└── helloWorld.txt
```

新的 File Commit 之後的 Log 
```Bash
$ git log
commit 9c15c7d8323c7f9cd2cb18164e0c8d69c575a15a (HEAD -> master)
Author: Chang Lin Hsieh <allen@ChangdeMacBook-Pro.local>
Date:   Tue Feb 22 22:50:02 2022 +0800

    add helloWorld2.txt

commit 48971f01848a55ac80698bd8c8b999b431f14723
Author: Chang Lin Hsieh <allen@ChangdeMacBook-Pro.local>
Date:   Tue Feb 22 22:00:22 2022 +0800

    first file
```

接下來我們來看看新的 commit 內容吧

```Bash
$ git cat-file -p 9c15c7d8323c7f9cd2cb18164e0c8d69c575a15a
tree bbdf0c43f735049c51bca17ea7543621c38efdd3
parent 48971f01848a55ac80698bd8c8b999b431f14723
author Chang Lin Hsieh <allen@ChangdeMacBook-Pro.local> 1645541402 +0800
committer Chang Lin Hsieh <allen@ChangdeMacBook-Pro.local> 1645541402 +0800

add helloWorld2.txt
```
* Line3: 我們可以看到 parent， parent 代表這個 commit 的父是誰，所以透過這樣的方式，可以知道之前的 commit 內容，這也是 Git 實現版本的方式。

接著我們來看看 tree 裡面的內容

```Bash
$ git cat-file -p bbdf0c43f735049c51bca17ea7543621c38efdd3
040000 tree 41758064447cf1624a474c3732dc2aead98e701e	I_AM_FOLDER
100644 blob 3b18e512dba79e4c8300dd08aeb37f8e728b8dad	helloWorld.txt
```

我們可以看到 line2 ， I_AM_FOLDER 的屬性也是 tree。接著我們來看看I_AM_FOLDER 裡面有什麼吧。

```Bash
$ git cat-file -p 41758064447cf1624a474c3732dc2aead98e701e 
100644 blob 3b18e512dba79e4c8300dd08aeb37f8e728b8dad	helloWorld2.txt
```

我們可以看到， I_AM_FOLDER 裡面有 helloWorld2.txt ，而 helloWorld2.txt 和 helloWorld.txt 的內容是一樣的，所以不會花額外的空間存取。


## 分支實現
---

當我們使用 git branch 時，可以看到現在所有的分支。而有一個會是當前的分支，前面會備註星號。那 Git 怎麼知道我們現在在哪個分支呢，是透過 .git/HEAD 這個檔案

```Bash
$ git branch
* master

$ cat .git/HEAD
ref: refs/heads/master

$ cat .git/refs/heads/master
9c15c7d8323c7f9cd2cb18164e0c8d69c575a15a

$ git log
commit 9c15c7d8323c7f9cd2cb18164e0c8d69c575a15a (HEAD -> master)
Author: Chang Lin Hsieh <allen@ChangdeMacBook-Pro.local>
Date:   Tue Feb 22 22:50:02 2022 +0800

    add helloWorld2.txt

commit 48971f01848a55ac80698bd8c8b999b431f14723
Author: Chang Lin Hsieh <allen@ChangdeMacBook-Pro.local>
Date:   Tue Feb 22 22:00:22 2022 +0800

    first file
```

* line5: ref 後面跟著一個路徑，代表現在分支所在的檔案。
* line8: 我們可以看到 master 檔案裡面實際只存了一個雜湊值。
* line11: 我們可以看到後面有標記 Head 指向 Master ，而 Master 指向這個 commit 

### 建立新的分支
---
```Bash
$ git branch I_AM_GOING_TO_UPDATE_HELLO_UPDATE

$ git branch
  I_AM_GOING_TO_UPDATE_HELLO_UPDATE
  * master

$  cat .git/HEAD
ref: refs/heads/master

$ tree .git/refs
.git/refs
├── heads
│   ├── I_AM_GOING_TO_UPDATE_HELLO_UPDATE
│   └── master
└── tags

$ cat .git/refs/heads/I_AM_GOING_TO_UPDATE_HELLO_UPDAT
9c15c7d8323c7f9cd2cb18164e0c8d69c575a15a

$ cat .git/refs/heads/master
9c15c7d8323c7f9cd2cb18164e0c8d69c575a15a

$ git log
commit 9c15c7d8323c7f9cd2cb18164e0c8d69c575a15a (HEAD -> master, I_AM_GOING_TO_UPDATE_HELLO_UPDATE)
Author: Chang Lin Hsieh <allen@ChangdeMacBook-Pro.local>
Date:   Tue Feb 22 22:50:02 2022 +0800

    add helloWorld2.txt

commit 48971f01848a55ac80698bd8c8b999b431f14723
Author: Chang Lin Hsieh <allen@ChangdeMacBook-Pro.local>
Date:   Tue Feb 22 22:00:22 2022 +0800

    first file
```

* Line1: 我們這邊建立一個新的分支叫做 I_AM_GOING_TO_UPDATE_HELLO_UPDATE
* Line3 和 line 7: 我們可以看到，現在的 Head 還是指向 Master
* Line10: 我們可以看到 refs 裡面有新的檔案就是我們 Branch 的名字
* Line 17,20 和23: 我們可以看到目前 I_AM_GOING_TO_UPDATE_HELLO_UPDATE & Master 都指向同一個 Git Commit

### 切換分支
---

```Bash
$ git checkout I_AM_GOING_TO_UPDATE_HELLO_UPDATE
Switched to branch 'I_AM_GOING_TO_UPDATE_HELLO_UPDATE'

$ git branch
* I_AM_GOING_TO_UPDATE_HELLO_UPDATE
  master

$ cat .git/HEAD
ref: refs/heads/I_AM_GOING_TO_UPDATE_HELLO_UPDATE
```

* Line1 : 我們使用 checkout 將現在的 Branch 切換到 I_AM_GOING_TO_UPDATE_HELLO_UPDATE 
* Line5& Line7 : 我們可以看到現在分之在 I_AM_GOING_TO_UPDATE_HELLO_UPDATE 

### 新分支 Commit
---
現在我們嘗試修改 helloWorld.txt 來看看回如何

```Bash
$ cat helloWorld.txt
hello world2

$ git log
commit 08e7447af10f14ec4cd6bffc210fddfbae3aa64f (HEAD -> I_AM_GOING_TO_UPDATE_HELLO_UPDATE)
Author: Chang Lin Hsieh <allen@ChangdeMacBook-Pro.local>
Date:   Tue Feb 22 23:43:14 2022 +0800

    update helloWorld.txt

commit 9c15c7d8323c7f9cd2cb18164e0c8d69c575a15a (master)
Author: Chang Lin Hsieh <allen@ChangdeMacBook-Pro.local>
Date:   Tue Feb 22 22:50:02 2022 +0800

    add helloWorld2.txt

commit 48971f01848a55ac80698bd8c8b999b431f14723
Author: Chang Lin Hsieh <allen@ChangdeMacBook-Pro.local>
Date:   Tue Feb 22 22:00:22 2022 +0800

    first file
```

* Line1: 新的 helloWorld.txt 內容
* Line5: 我們可以看到 HEAD 指向 I_AM_GOING_TO_UPDATE_HELLO_UPDATE 和 I_AM_GOING_TO_UPDATE_HELLO_UPDATE 指向最新的 Commit 08e7447af10f14ec4cd6bffc210fddfbae3aa64f
* Line11: 我們可以看到 Master 現在還是指向上一個 Commit 9c15c7d8323c7f9cd2cb18164e0c8d69c575a15a

### 舊分支 Commit
---
我們現在切回 master branch 來看看

```Bash
$ git checkout master
Switched to branch 'master'

$ git log 
commit 9c15c7d8323c7f9cd2cb18164e0c8d69c575a15a (HEAD -> master)
Author: Chang Lin Hsieh <allen@ChangdeMacBook-Pro.local>
Date:   Tue Feb 22 22:50:02 2022 +0800

    add helloWorld2.txt

commit 48971f01848a55ac80698bd8c8b999b431f14723
Author: Chang Lin Hsieh <allen@ChangdeMacBook-Pro.local>
Date:   Tue Feb 22 22:00:22 2022 +0800

    first file

$ git log --oneline --graph --decorate --all
* 08e7447 (I_AM_GOING_TO_UPDATE_HELLO_UPDATE) update helloWorld.txt
* 9c15c7d (HEAD -> master) add helloWorld2.txt
* 48971f0 first file

$ cat helloWorld.txt
hello world
```

* Line1: 切換回 master branch
* Line4: 大家會發現，Log 只顯示當下 Master 相關的 Log ，會看不到 I_AM_GOING_TO_UPDATE_HELLO_UPDATE Branch 相關的
* Line17: 我們可以更清楚看到現在的 Commit 結構
* Line22: 我們會發現 helloWolrd.txt 恢復成原本舊的還在 Master Branch 的版本。

我們現在一樣來修改 helloWorld.txt 並 commit 看看會發生什麼事情。

```Bash
$ cat helloWorld.txt
hello world3

$ git log --oneline --graph --decorate --all
* f7c9f13 (HEAD -> master) master update helloWorld.txt
| * 08e7447 (I_AM_GOING_TO_UPDATE_HELLO_UPDATE) update helloWorld.txt
|/
* 9c15c7d add helloWorld2.txt
* 48971f0 first file
```

line4 : 大家應該會發現 9c15c7d 這邊分支成兩個 f7c9f13 和 08e7447。

### 合併兩個分支
---

我們現在來將 I_AM_GOING_TO_UPDATE_HELLO_UPDATE 合併回 master 看看會發生什麼事情

```Bash
$ git checkout master
Switched to branch 'master'

$ git merge I_AM_GOING_TO_UPDATE_HELLO_UPDATE
Auto-merging helloWorld.txt
CONFLICT (content): Merge conflict in helloWorld.txt
Automatic merge failed; fix conflicts and then commit the result.

$ git diff
diff --cc helloWorld.txt
index 923e989,1142904..0000000
--- a/helloWorld.txt
+++ b/helloWorld.txt
@@@ -1,1 -1,1 +1,5 @@@
++<<<<<<< HEAD
 +hello world3
++=======
+ hello world2
++>>>>>>> I_AM_GOING_TO_UPDATE_HELLO_UPDATE
```

* Line4: 由於 I_AM_GOING_TO_UPDATE_HELLO_UPDATE 和 master 都修改過 helloWorld.txt。所以 Git 會不知道要保留哪個版本，而這時就出現了 CONFLICT。
* Line9: 從 Git Diff 中我們可以明顯看到， Git 已經幫我們修改檔案，並 Highlight 出兩邊的不同。


現在假設我覺得 master 寫得比較好，所以我保留 master 的內容。

```Bash
$ cat helloWorld.txt
hello world3

commit 063d5d5307dd178d11bed8ee67a1ed5c614276bf (HEAD -> master)
Merge: f7c9f13 08e7447
Author: Chang Lin Hsieh <allen@ChangdeMacBook-Pro.local>
Date:   Wed Feb 23 09:20:10 2022 +0800

    fix CONFLICT

commit f7c9f13b17ba1fa6641e7c6abb90935321e4f33c
Author: Chang Lin Hsieh <allen@ChangdeMacBook-Pro.local>
Date:   Tue Feb 22 23:57:00 2022 +0800

    master update helloWorld.txt

commit 08e7447af10f14ec4cd6bffc210fddfbae3aa64f (I_AM_GOING_TO_UPDATE_HELLO_UPDATE)
Author: Chang Lin Hsieh <allen@ChangdeMacBook-Pro.local>
Date:   Tue Feb 22 23:43:14 2022 +0800

    update helloWorld.txt

commit 9c15c7d8323c7f9cd2cb18164e0c8d69c575a15a
Author: Chang Lin Hsieh <allen@ChangdeMacBook-Pro.local>
Date:   Tue Feb 22 22:50:02 2022 +0800

    add helloWorld2.txt

commit 48971f01848a55ac80698bd8c8b999b431f14723
Author: Chang Lin Hsieh <allen@ChangdeMacBook-Pro.local>
Date:   Tue Feb 22 22:00:22 2022 +0800

    first file
```

* Line5： 多了 Merge的提示

這時我們來看看最新的 commit 內容吧

```Bash
$ git cat-file -p 063d5d5307dd178d11bed8ee67a1ed5c614276bf
tree e65ced472fc129e23c9074c2cfd3649d4c43ff29
parent f7c9f13b17ba1fa6641e7c6abb90935321e4f33c
parent 08e7447af10f14ec4cd6bffc210fddfbae3aa64f
author Chang Lin Hsieh <allen@ChangdeMacBook-Pro.local> 1645579210 +0800
committer Chang Lin Hsieh <allen@ChangdeMacBook-Pro.local> 1645579210 +0800

fix CONFLICT

$ git cat-file -p f7c9f13b17ba1fa6641e7c6abb90935321e4f33c
tree e65ced472fc129e23c9074c2cfd3649d4c43ff29
parent 9c15c7d8323c7f9cd2cb18164e0c8d69c575a15a
author Chang Lin Hsieh <allen@ChangdeMacBook-Pro.local> 1645545420 +0800
committer Chang Lin Hsieh <allen@ChangdeMacBook-Pro.local> 1645545420 +0800

master update helloWorld.txt
```

* Line1: 這時可以清楚看到裡面有兩個parent，第一個 parent (line3) 是 master 而第二個 parent (line4) 是 I_AM_GOING_TO_UPDATE_HELLO_UPDATE。所以從這裡這裡知道，我們是從第二個 parent merge 到第一個 parent
* Line10: 我們再回頭看看原本 master parent 的內容，因為我們保留了 master ，所以 Line11 和 Line2 的雜湊值是一樣的

接下來我們來看一下最新的 Tree 的樣子

```
$ git log --oneline --graph --decorate --all
*   063d5d5 (HEAD -> master) fix CONFLICT
|\
| * 08e7447 (I_AM_GOING_TO_UPDATE_HELLO_UPDATE) update helloWorld.txt
* | f7c9f13 master update helloWorld.txt
|/
* 9c15c7d add helloWorld2.txt
* 48971f0 first file
```

可以看到最上面又將兩個 Tree 合併了


## Rebase 原理
---

我將之前的 Merge reset 回去

```Bash
$ git reset f7c9f13

$ git log --oneline --graph --decorate --all
* f7c9f13 (HEAD -> master) master update helloWorld.txt
| * 08e7447 (I_AM_GOING_TO_UPDATE_HELLO_UPDATE) update helloWorld.txt
|/
* 9c15c7d add helloWorld2.txt
* 48971f0 first file
```

rebase 會先找到兩個 branch 開始分差的 commit ，以上面的例子是 9c15c7d。如果要將 I_AM_GOING_TO_UPDATE_HELLO_UPDATE rebase 到 master，會將 9c15c7d 到 I_AM_GOING_TO_UPDATE_HELLO_UPDATE 之間的所有commit 修改，從 Master 之後嘗試做一次，最後在將 master 指向最新的 commit

```Bash
$ git rebase I_AM_GOING_TO_UPDATE_HELLO_UPDATE

Auto-merging helloWorld.txt
CONFLICT (content): Merge conflict in helloWorld.txt
error: could not apply f7c9f13... master update helloWorld.txt
Resolve all conflicts manually, mark them as resolved with
"git add/rm <conflicted_files>", then run "git rebase --continue".
You can instead skip this commit: run "git rebase --skip".
To abort and get back to the state before "git rebase", run "git rebase --abort".
Could not apply f7c9f13... master update helloWorld.txt
```

由於兩個分支之前都修改過 helloWorld.txt ，所以會有 conflict 。如果沒有 conflict， master 就會 fast forward 到最新的 commit。

修改完 helloWorld.txt 我們繼續執行  git rebase --continue 

```Bash
$  git rebase --continue
Successfully rebased and updated refs/heads/master.

$ git log --oneline --graph --decorate --all
* 08e7447 (HEAD -> master, I_AM_GOING_TO_UPDATE_HELLO_UPDATE) update helloWorld.txt
* 9c15c7d add helloWorld2.txt
* 48971f0 first file
```


### Merge v.s.  Rebase
---
Merge 的好處就是會紀錄所有的修改，包含兩個分支合併時的修改。但是當開發者太多時，同時太多 Merge 會很難追中，這時候反而 rebase 會讓整個 Histroy 看起來比較直觀，但是代價就是不能像 merge 一樣，清楚地知道所有的修改。


## Tag 原理
---

```Bash
$ git checkout 08e7447

$ git tag tag1 

$ git log --oneline --graph --decorate --all
* ac766f3 (master) master2
* 19f9d5c (I_AM_GOING_TO_UPDATE_HELLO_UPDATE) helloWorld
* 08e7447 (HEAD, tag: tag1) update helloWorld.txt
* 9c15c7d add helloWorld2.txt
* 48971f0 first file

$ cat .git/refs/tags/tag1
08e7447af10f14ec4cd6bffc210fddfbae3aa64f
```

* line3: 我們在 08e7447 建立一個 tag 叫 tag1 
* line8: 我們可以看到 08e7447 後面多了一個 tag
* line12: 在 .git/refs/tags 檔案夾裡面，可以找到一個檔案叫做 tag1 ，而裡面的雜湊值是我們 tags 的 commit 

如果我們想要在 tag 上加上一些有用的描述呢

```Bash
$ git tag tag2 -a -m "i am tag2"
# -a Make an unsigned, annotated tag object 

$ cat .git/refs/tags/tag2
9a9cf462e7960ddbe7a926dbf46c3ffc8afdb363

$ git cat-file -t 9a9cf462e7960ddbe7a926dbf46c3ffc8afdb363
tag

$ git cat-file -p 9a9cf462e7960ddbe7a926dbf46c3ffc8afdb363
object 08e7447af10f14ec4cd6bffc210fddfbae3aa64f
type commit
tag tag2
tagger Chang Lin Hsieh <allen@ChangdeMacBook-Pro.local> 1645628590 +0800

i am tag2
```

* line1: 我們建立一個新的 tag object ，並包含了 "i am tag2" 的資訊
* line4: 我們可以看到 tag2 裡面的雜湊值不是我們的 commit 的雜湊值
* line9: 我們可以知道 line5 的這個雜湊值是一個 tag 物件
* line11,12: 我們可以知道這個 tag 對應到一個 commit 而這個 commit 的雜湊值為 08e7447af10f14ec4cd6bffc210fddfbae3aa64f

### Tag v.s Branch
---
當有新的commit 時， Tag 不會指向新的 commit ，而 Branch 會


## Remote v.s Local

當我們在執行 git clone 時，我們是將 Remote 的 .git folder 特定的 Branch 複製下來 (default master)

```Bash
$  cat .git/config
[core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
	ignorecase = true
	precomposeunicode = true
[remote "origin"]
	url = git@github.com:allen-hsieh1992/blog.git
	fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
	remote = origin
	merge = refs/heads/master

$ tree .git/refs/remotes
.git/refs/remotes
└── origin
    ├── HEAD
    └── master
```

我以我的 github 為例子
* line9: 我們可以看到，有一個 remote 就 origin 並且他的 url 在下面。
* line16: 我們可以看到 refs 資料夾下面多了 remotes 的資料夾
 
現在我們嘗試在local 加一些commit 

```Bash
## before commit
$ git show-ref master
10275a905f41267b2b2a12f5cd3e3cbee984203a refs/heads/master
10275a905f41267b2b2a12f5cd3e3cbee984203a refs/remotes/origin/master

## after commit
$ git show-ref master
1251ab4bdd68fdf8a3175719f47565b6c8af44d2 refs/heads/master
10275a905f41267b2b2a12f5cd3e3cbee984203a refs/remotes/origin/master

## after push
$ git show-ref master
1251ab4bdd68fdf8a3175719f47565b6c8af44d2 refs/heads/master
1251ab4bdd68fdf8a3175719f47565b6c8af44d2 refs/remotes/origin/master
```

* line2: line2 我們可以看到在 commit 之前 local & remote 的 master 都是同一個物件。
* line7: 當我們 local commit 以後，可以看到 local 的物件已經更新了，但 remote 還是舊的。
* line12
