(13条消息)如何用git命令生成Patch和打Patch_苏公子的博客-CSDN博客

(13条消息)如何用git命令生成Patch和打Patch_苏公子的博客-CSDN博客

在程序员的日常开发与合作过程中，对于code的生成patch和打patch(应用patch)成为经常需要做的事情。

*   **什么是patch?简单来讲,patch中存储的是你对代码的修改**
*   **什么是生成patch?生成patch就是记录你对代码的修改并将其保存在patch文件中**
*   **什么是打patch?打patch就是将patch文件中对代码的修改，应用到源代码，从而把对代码的修改应用到code中。**

尽管本身Linux命令里有diff和patch两个命令可以生成patch和打patch。但是有两个缺点值得注意：

1. 对单个文件或者多个文件，diff和patch这两个文件比较方便。对于git这种以project为单位的修改，尤其是涉及到多个文件夹下的多个文件的改动时，就很不方便

2. 无法保存commit的信息。

**因此，推荐大家使用git的format-patch和am命令进行生成patch和打patch，用此方法获得的patch其实就是commit里提交的code修改以及commit信息。有如下好处**：

1. 对于git这种以project为单位的修改，尤其是涉及到多个文件夹下的多个文件的改动时，非常方便，能够记录所有的改动（添加，修改，删除文件等）

2. 可以保存commit信息。

3. 能够灵活的获取patch。可以获取任意两个commit之间的patch集。

使用方法（直接给一些examples）：

**git format-patch**

$ git format-patch HEAD^ 　　　　　　　　　　　　　   #生成最近的1次commit的patch

$ git format-patch HEAD^^　　　　　　　　　　　　　  #生成最近的2次commit的patch

$ git format-patch HEAD^^^ 　　　　　　　　　　　　　#生成最近的3次commit的patch

$ git format-patch HEAD^^^^ 　　　　　　　　　　　      #生成最近的4次commit的patch

$ git format-patch &lt;r1&gt;..&lt;r2&gt;                                              #生成两个commit间的修改的patch（包含两个commit. &lt;r1&gt;和&lt;r2&gt;都是具体的commit号)

$ git format-patch -1 &lt;r1&gt;                                                   #生成单个commit的patch

$ git format-patch &lt;r1&gt;                                                       #生成某commit以来的修改patch（不包含该commit）

$ git format-patch --root &lt;r1&gt;　　　　　　　　　　　　   #生成从根到r1提交的所有patch

**git am**

$ git apply --stat 0001-limit-log-function.patch   　　　　  # 查看patch的情况

$ git apply --check 0001-limit-log-function.patch   　　　  # 检查patch是否能够打上，如果没有任何输出，则说明无冲突，可以打上

(注：git apply是另外一种打patch的命令，其与git am的区别是，git apply并不会将commit message等打上去，打完patch后需要重新git add和git commit，而git am会直接将patch的所有信息打上去，而且不用重新git add和git commit,author也是patch的author而不是打patch的人)

$ git am 0001-limit-log-function.patch                                # 将名字为0001-limit-log-function.patch的patch打上

$ git am --signoff 0001-limit-log-function.patch                  # 添加-s或者--signoff，还可以把自己的名字添加为signed off by信息，作用是注明打patch的人是谁，因为有时打patch的人并不是patch的作者

$ git am ~/patch-set/*.patch　　　　　　　　　　　　　# 将路径~/patch-set/*.patch 按照先后顺序打上

$ git am --abort                                                                   # 当git am失败时，用以将已经在am过程中打上的patch废弃掉(比如有三个patch，打到第三个patch时有冲突，那么这条命令会把打上的前两个patch丢弃掉，返回没有打patch的状态)

$ git am --resolved                                                             #当git am失败，解决完冲突后，这条命令会接着打patch

**如果打Patch的过程中发生了冲突（conflicts），怎么办？**

解决patch冲突的过程是：

**如果不想打这一系列patch了，直接：git am --abort。**

如果还想打, 有两种解决方案：

**方案一（个人推荐）：**

(1) 根据git am失败的信息，找到发生冲突的具体patch文件，然后用命令git apply --reject &lt;patch_name&gt;，强行打这个patch，发生冲突的部分会保存为.rej文件（例如发生冲突的文件是a.txt，那么运行完这个命令后，发生conflict的部分会保存为a.txt.rej），未发生冲突的部分会成功打上patch

(2) 根据.rej文件，通过编辑该patch文件的方式解决冲突。

(3) 废弃上一条am命令已经打了的patch：git am --abort

(4) 重新打patch：git am ~/patch-set/*.patchpatch

**方案二：**

(1) 根据git am失败的信息，找到发生冲突的具体patch文件，然后用命令git apply --reject &lt;patch_name&gt;，强行打这个patch，发生冲突的部分会保存为.rej文件（例如发生冲突的文件是a.txt，那么运行完这个命令后，发生conflict的部分会保存为a.txt.rej），未发生冲突的部分会成功打上patch

(2) 根据.rej文件，通过编辑发生冲突的code文件的方式解决冲突。

(3) 将该patch涉及到的所有文件（不仅仅是发生冲突的文件）通过命令git add &lt;file_name&gt;添加到工作区中

(4) 告诉git冲突已经解决，继续打patch: git am --resolved (git am --resolved 和 git am --continue是一样的)

**分析**：方案一和方案二主要区别是解决冲突的方法不一样。方案一是通过编辑patch文件的方式解决冲突，方案二十通过编辑冲突code文件的方式解决冲突。这两种方案区别比较大：经过实验，核心区别在于，方案二无法验证冲突有没有切实的解决。即使你在方案二的第二步乱改一通，也能“打完”发生冲突的patch（并没有检测修改后的code文件跟patch期望的是否相同）。因此，如果采用方案二，那么再解决code文件冲突后，需要人工去确认修改的正确性。