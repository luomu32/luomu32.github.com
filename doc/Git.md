# Git

merge与rebase

两者都是合并分支。造成的分支历史是不同的。merge会成为一个新的commit，很多分支，多次merge操作之后，会导致分支历史杂乱，分支演进图看起来很不直观。rebase合并的分支，历史会成一条直线，看起来比较好看。

reflog与log



cherry-pick

简单将，就是允许将当前分支的个别commit合并的其他分支上。如果使用merge或rebase会将目标分支的所有commit合并到当前分支上。最好按照commit的提交顺序合并。

reset与checkout与revert

reset是将HEAD指针指向一个指定的commit，所以一般先用reflog查询commit历史。reset撤销本的分支的更改。如果已经将更改的内容push到了远程分支上，那么需要使用revert操作，生成一个新的反操作的commit，提交到远程分支达到撤销的效果。而checkout是将本地未commit的文件还原。

pull与pull --rebase

