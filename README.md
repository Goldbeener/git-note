# 概念
git本质上是一个文件系统             
做版本管理，把不同版本的仓库信息存储在`.git`目录下

> 并不是以完整版本为单位来做版本管理；而是以散列的文件版本为纬度来做版本管理；一个文件有多个版本，然后以tree + commit来索引每个版本具体包含了哪些文件的哪个版本；这样可以充分利用公共的改动少的文件，减少空间占用。同时这也是缓存区的作用。缓存区也叫索引区就是根据版本信息来索引具体的文件信息。

# Git Objects
在`.git/objects`目录下存储着所有文件的历史版本信息，objects文件夹下有三类objects：
+ blob 二进制形式的纯文件内容，不包含文件的其他信息, 该文件名称是内容的哈希值；blob对象是在`git add`之后产生的，有多少个文件变动，就会新产生多少个blob；这些blob是版本管理的基础，因为仓库的所有改动都在这里面保存着
+ tree 目录结构、对应的版本的快照，包含整个托管仓库的目录结构，每个文件的权限+类型+hash+文件名。该对象是在commit之后产生的
+ commit 一次提交的信息. 提交时对应目录结构tree + 父节点 + 作者信息 + 提交备注信息，该对象也是在`git commit`之后产生的，并且是在`tree`的基础上产生的。


这样每次的git commit其实主要都是存储了一个tree；而这个tree里面又包含着当前整个目录结构，根据目录结构中包含的文件的hash又可以找到对应的文件；像这样通过一层层的`引用`确认了整个版本的内容信息。

以上就是git版本管理存储的本质。

# index
`index`索引区，也叫暂存区，在`.git/index`文件中       
> “索引文件用识别码列出相关的blob文件以及别的数据”
维护的是工作区需要展示的内容版本，也就是工作区内容的切换是通过这个文件控制的。

# HAED & branch & tag
HEAD是本地的头指针，指向当前的最新的一次commit；         
branch分支本质上也是一个指针，指向某个分叉的commit；不同的分支本质上也是不同版本的commit        
tag 指针，也是历史的某个关键commit

本质上上述的三种指针都是指向某个特定的commit，而commit又对应着不同的仓库版本。

# 仓库区域划分
类似同类的版本管理工具，git也可以分为`远程仓库`和`本地仓库`       
不同的是在本地仓库中，git多了一个`暂存区(索引区)`
+ 远程仓库
+ 本地仓库
    + 工作区
    + 暂存区
    + 本地仓库

平时的修改编辑直接是在工作区；此时的改动不会影响`.git`内的内容，这也是常提示的`changes not staged`; 只是本机文件系统的修改       
`git add .`命令，将改动的文件（因为文件内容变更，所以hash变了）重新生成了blob，并将改动文件的索引指向到新的blob（此时已经属于暂存区的工作范畴，还未生成新的版本，因为还没有commit）, 此时`.git/objects`内会新增内容，增加的就是因为改动而产生的新的blob        
`git commit ` 命令，根据当前的文件结构生成快照`tree`，然后在此`tree`的基础上生成一个新的`commit`; 并且将本地的`HEAD`指针更新指向到新的`commit`上. 如上所述`commit`内存储有前一个`commit`的信息；以此组成一个链式的变更历史。  

# 会产生 commit Objects 的操作
+ `commit`操作
    - 常规的`git commit -am`
    - x修订备注信息的`git commit --amend` 
+ `merge`操作

产生了commit objects之后，就算是一次版本的变更了，需要注意同步操作.

## 关于修订`--amend`
`--amend`可以修改`最新的commit`的备注信息;      
+ 如果最新的commit还在本地，并没有同步到远端；那么修改之后会使用上一次的快照产生一次`新的commit`，替换掉老的commit，达到更新备注信息的目的;                 
+ 如果最新的commit已经推送到远端，那么修改备注信息就算是一次`新的commit`，并且与远端产生了分叉，需要合并更新，这时候老的备注信息还在，只是多了2次commit行为，一次是修改备注，一次是merge行为。

> 对于第二种状况的理解，意味着一旦将末次commit同步到远端，那么此次的行为将会被记录下来，就无法被更改了。

# 常见指令
## add 
add 是保存工作区内的变动到暂存区；仅仅会产生新的
## commit
commit 结合暂存区改动的内容，对当前全仓库产生一个快照tree，并生成一次commit记录，此时一个新的版本诞生了。
## checkout
用于从历史版本（历史commit）中拷贝文件同步到暂存区和工作区；       
也可用用于从暂存区同步内容同步到工作区，以覆盖工作区；        
也可用于切换分支.
### 应用场景
+ git checkout HEAD~n file 将某个文件恢复到当前版本之前的第n个版本的内容 同步在暂存区和工作区， 此时当前的分支指向信息不会改变
+ git checkout . 从暂存区拷贝文件同步到工作区，常用语覆盖工作区的改动，当前分支信息不变
+ git checkout branchName 切换到某个（本地）分支，并更新暂存区和工作区与分支信息一致，分支信息改变
+ git checkout hash/tag/远程分支/版本 没有指定file或本地分支名 这时候使用指定的这些信息对应的版本快照创建一个新的匿名分支，（分离头指针）；该分支可以正常编辑、commit不会影响其他分支信息；但是在切换到其他分支准备离开时，会提示是否保存在此分支上的改动，如果要保存的话可以使用`git branch branchName <hash|tag|版本>`或者`git checkout -b branchName <hash|tag>`; 如果不操作的话，这些改动就会被丢弃，也不会影响其他分支.

## reset
把当前分支HEAD头指向另一个位置(历史commit)，理解是版本回退的感觉；    
默认是只把变动同步到暂存区（`--soft`）而不改变工作区，也可以通过参数控制把变动同步到工作区(`--hard`)

+ git reset commitId <--soft | --hard>
+ git reset commitId -- file 仅针对指定文件，此时不能够加参数soft或hard

## merge
不同分支间的合并，       
合并之前，当前所在的分支必须是干净的，即当前所在的分支工作区、暂存区都是干净的，与仓库是一致的;     
```bash
# 当前所在分支假定是featA
git merge master
```
合并可能出现以下三种情况:     
1. master ---> commitX... ---> featA        

merge目标分支master是当前分支的祖先节点；那么本次分支什么都不做

2. featA ----> commitX... ---> master

merge 的目标分支是当前分支的 后辈分支， 那么这是一次`fast-forward`合并，只是简单的指针移动，生成一次新的提交

3. commitA ----> commitX.. ---> master

    |____  ----> commitY.. ---> featA

两个分支是分叉的，那么是一次比较复杂的合并

公共祖先节点commitA + master + featA 先进行一次三方合并 
    
如果有冲突，解决冲突，然后生成一次新的commit完成合并  


# cherry-pick 
`cherry-pick`是复制某一次`提交节点`，并且在当前的分支上做一次完全一样的提交。
```
git cherrt-pick commitID # 复制commitID指定的节点状态（仓库tree），在当前分支上做一次相同的提交
```
> 关于提交节点的理解，最初我理解为当时的仓库快照，经过实验发现，提交节点并不是一个完整的仓库快照；而是此次提交中改变的地方；这样延伸开来，每次提交是`增量的`; 有说法是commit提交的是暂存区的内容；那意味着暂存区内也是增量的；只保存文件改动的地方；

![WechatIMG11.jpeg](https://i.loli.net/2020/01/10/ngWGTzcfHEBdmYq.jpg)

如上图，本次提交改动了2个地方；那个在使用本次的commitId做cherry-pick的时候，只会把这2处改动与当前分支合并。

> `cherry-pick`可以理解为是一种特殊形式的merge

# rebase
rebase 是合并的另一种选择；      
在当前分支上重演另一个分支的历史;       
```js
// 当前分支在某功能分支上
git rebase master
```
首先，当前分支是从master某一节点开出来的新的分支节点；进行某一功能的开发；     
开发过程中，其他同学从别的功能分支上合并了master；导致master分支发生了更新；     
当你在你的功能分支上开发完成的时候，往master合并的时候，因为master已经发生了改变，可能会发生冲突;    
使用rebase，git会做3件事儿:
1. 将功能分支的改变先保存
2. 从共同的父节点开始，把master的演变历史应用在功能分支上，这时候就会把别人的功能合并到你的分支上;
3. 保证功能分支上master部分的功能与当前最新的master一致时；再将第一步保存的信息，在此基础上演变

> 注意，使用rebase时，rebase命令后面跟的分支，是需要同步的标准参考，会将该分支从与当前分支的分叉之后的改变同步到当前分支；该分支是不会变化的；只会改变当前分支。

## flow
分支间协同合作方式     

### 功能分支向主干分支合并的方式
github中从功能分支向主干分支合并的时候，有三种可选的方式:
+ merge commits  功能分支通过合并的方式，在主干分支上产生一个新的commit，会有功能分支向master合并的记录，不是线性的
+ squash merging  合并压缩功能分支的commit记录为一个整体的commit，然后在master分支的基础上，产生一个与这个整体commit一样的commit；master分支是线性的；功能分支无任何改变
+ rebase merging  可以理解为多个cherry-pick；以功能分支的每个commit节点为原型，在master上创建一个相同的cherry-pick；最终master分支也是线性的；功能分支也没有任何改变


# .gitignore
添加`.gitignore`文件可以用来忽略某些不希望被托管的文件; 但是必须注意只能忽略的是，没有被track的文件；如果某些文件已被track，那么修改`.gitignore`是无效的.

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

## checkout VS reset
checkout感觉主要是处理不同分支之间的问题；相互切换的是并行的版本线; 是一个分叉的切换;            
reset主要处理的是同一个分支的不同版本之间的问题；是一个线性的切换；
