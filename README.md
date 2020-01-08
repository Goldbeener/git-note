# git-note
git note

# 概念
git本质上是一个文件系统

做版本管理，把不同版本的仓库信息存储在`.git`目录下

# Git Objects
+ blob 二进制文件内容，不包含文件的其他信息, 该文件名称是内容的哈希值；blob对象是在`git add`之后产生的，有多少个文件变动，就会新产生多少个blob；
+ tree 目录结构、当前版本快照，包含整个仓库的目录结构，每个文件的权限+类型+hash+文件名。该对象是在commit之后产生的
+ commit 一次提交信息. 提交时对应目录结构tree + 父节点 + 作者信息 + 提交备注信息，该对象也是在`git commit`之后产生的，并且是在`tree`的基础上产生的。


这样每次的git提交其实主要都是存储了一个tree；而这个tree里面又包含着当前整个目录结构，根据目录结构中包含的文件的hash又可以找到对应的文件；像这样通过一层层的`引用`确认了整个版本的内容信息。

以上就是git版本管理存储的本质。

# HAED & branch & tag
HEAD是本地的头指针，

branch分支本质上也是一个指针，指向不同的commit；不同的分支本质上也是不同版本的commit

tag 指针，比较好理解

# 工作区划分
+ 远程仓库
+ 本地仓库
    + 工作区
    + 暂存区
    + 本地仓库

平时的修改直接是在工作区；此时的改动不会影响`.git`内的内容，这也是常提示的`changes not staged`                 
`git add .`命令，将改动的文件（因为文件内容变更，所以hash变了）重新生成了blob，并将改动文件的索引指向到新的blob, 此时`.git/objects`内回新增内容，增加的就是因为改动而产生的新的blob        
`git commit ` 命令，根据当前的文件结构生成快照`tree`，然后在此`tree`的基础上生成一个新的`commit`; 并且将本地的`HEAD`指针更新指向到新的`commit`上. 如上所述`commit`内存储有前一个`commit`的信息；以此组成一个链式的变更历史。  

# 会产生 commit Objects 的操作
+ `commit`操作
    - 常规的`git commit -am`
    - x修订备注信息的`git commit --amend` 
+ `merge`操作

产生了commit objects之后，就算是一次版本的变更了，需要注意更新操作.

## 关于修订`--amend`
`--amend`可以修改`最新的commit`的备注信息;      
如果最新的commit还在本地，并没有同步到远端；那么修改之后会使用上一次的快照产生一次新的commit，替换掉老的commit，达到更新备注信息的目的;                 
如果最新的commit已经推送到远端，那么修改备注信息就算是一次新的commit，并且与远端产生了分叉，需要合并更新，这时候老的备注信息还在，只是多了2次commit行为，一次是修改备注，一次是merge行为。

# checkout 
## 应用场景
+ git checkout HEAD~n file 将某个文件恢复到当前版本之前的第n个版本的内容 同步在暂存区
+ git checkout branchName 切换到某个分支，并更新暂存区和工作区与分支信息一致
+ git checkout hash/tag/远程分支/版本 没有指定file和本地分支名 这时候使用指定的这些信息对应的版本快照创建一个新的匿名分支，（分离头指针）；对分支可以正常编辑、commit不会影响其他分支信息；但是在切换到其他分支准备离开时，会提示是否保存在此分支上的改动，如果要保存的话可以使用`git checkout -b branchName`; 如果不操作的话，这些改动就会被丢弃，也不会影响其他分支.

# 思考
## 关于暂存区
暂存区，我理解是一个中间态的tree；如果没有暂存区的话，每次的改动add之后git都需要在本地仓库内生成一个新的tree，应该是比较混乱和高成本的。有了这个暂存区，每次改动并add之后，只更新这个中间态的tree，然后等你确认好了之后commit，这时候才真正的产生一个tree，并在此基础上产生commit。有点类似虚拟DOM与真实DOM的感觉。个人理解，欢迎讨论。
## 为什么要把文件名维护在tree里，文件的内容单独产生一个blob
这样是为了避免因为修改文件名字导致的文件hash改变，确保每次文件的变更都仅仅是因为确实是文件内容变更了      
并且每个文件改动产生一个新的blob，然后更新暂存区和仓库内的索引，指向新的版本; 同时保留着老得版本; 这样达到了版本管理的目的.     
这样带来的副作用就是空间的占用，但是git有垃圾回收机制（GC）；并且对于相似的Objects会打包压缩。
## 每次commit，Git存储的是全新的文件快照还是增量的变更部分
全新的快照，空间换时间
## Git历史不可篡改
由上可知git的变更必然是由于文件内容的改动，这样每次文件变更都会产生新的commit。并且是分布式的，相关的人都有一份完成的历史的git仓库

