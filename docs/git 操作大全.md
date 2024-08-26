# checkout

`git checkout feature_01` 命令的含义是 **切换** 到名为 `feature_01` 的分支。如果本地不存在`feature_01`这个分支，那么执行命令会出错：`error: pathspec 'feature_01' did not match any file(s) known to git。
如果远程仓库已经有这个分支，那么执行上述命令hu
`

**例子:**

假设你当前在 `main` 分支上，并且你想要切换到 `feature_01` 分支:

```bash
git checkout feature_01
```

执行完该命令后，你将位于 `feature_01` 分支上，可以开始在该分支上进行开发工作。

- `git checkout -b` 与 `git checkout -t` 的区别
- `git checkout` 在切换分支的时候，与 `git switch`、`git branch` 的区别
- `git merge` 与 `git rebase` 的区别
