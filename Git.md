### `git remote`
1. `git remote [-v | --verbose]`
2. `git remote add <name> <URL>`
3. `git remote remove <name>`
4. `git remote set-url <name> <remote-url>`
  

### `git add`

1. `git add .`：

2. `git add *`：

3. `git add <filename>`：

  

### `git merge`和`git rebase`

  

### `git commit --amend`撤销提交

  

### `git log`查看提交日志

  

### `git diff`

1. `git diff`查看`工作区`与`暂存区`的区别

2. `git diff --staged`查看`暂存区`与上一次提交的区别

3. `git diff <filename>`显示指定文件的差异

4. `git diff <commit1> <commit2>`

5. `git diff <branch1> <branch2>`

6. `...`

  

### `git rebase`

1. `git rebase <branch>`：将`branch`的新内容拉到`当前分支`

2. `git rebase <from> <to>`：将`from`的新内容拉到`to`

  

### `git branch`

1. `git branch`：查看本地分支

2. `git branch -r`：查看远程分支

3. `git branch -a`：查看本地和远程分支

4. `git branch <branch>`：切换到`branch`分支

5. `git branch -d <localBranch>`：删除本地分支`localBranch`

6. `git push origin --delete <remoteBranch>`；删除远程`origin`的`remoteBranch`分支

  

---