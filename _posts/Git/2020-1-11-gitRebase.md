---
layout: post
title: Git Rebase
categories: [Git]
tags: [Git]
comments: false

---

[참고](https://backlog.com/git-tutorial/kr/stepup/stepup2_8.html)

### Case

<img src="/assets/media/Git/GitRebase1.png">

#### Develop > aFile

```
Develop : 1
Develop : 2
```

#### Feature/aa > aFile

```
Feature/aa : 1
Feature/aa : 2
```

### 순서

#### 1. git checkout feature/aa

#### 2. git rebase develop

- rebase  시작
- 충돌

```
First, rewinding head to replay your work on top of it...
Applying: aa : 1
Using index info to reconstruct a base tree...
M    aFile
Falling back to patching base and 3-way merge...
Auto-merging aFile
CONFLICT (content): Merge conflict in aFile
error: Failed to merge in the changes.
Patch failed at 0001 aa : 1
hint: Use 'git am --show-current-patch' to see the failed patch
Resolve all conflicts manually, mark them as resolved with
"git add/rm <conflicted_files>", then run "git rebase --continue".
You can instead skip this commit: run "git rebase --skip".
To abort and get back to the state before "git rebase", run "git rebase --abort".
```

#### 3. git status

```
rebase in progress; onto 6494f01
You are currently rebasing branch 'feature/aa' on '6494f01'.
  (fix conflicts and then run "git rebase --continue")
  (use "git rebase --skip" to skip this patch)
  (use "git rebase --abort" to check out the original branch)

Unmerged paths:
  (use "git reset HEAD <file>..." to unstage)
  (use "git add <file>..." to mark resolution)

    both modified:   aFile

no changes added to commit (use "git add" and/or "git commit -a")
```

#### 4. 충돌 수정

```
<<<<<<< HEAD
Develop : 1
Develop : 2

=======
Feature/aa : 1
>>>>>>> aa : 1
```

```
<<<<<<< HEAD
Develop : 1
Develop : 2
Feature/aa : 1
```

#### 5. git add aFile

- commit이 아닌 add사용

#### 6. git rebase --continue

최종형태
```
Develop : 1
Develop : 2
Feature/aa : 1
Feature/aa : 2
```
#### 6. git checkout develop

- develop 에 가서 feature/aa 머지 후 종료
- rebase된 커밋들은 날짜가 변경된다.
