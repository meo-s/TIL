# Git/submodules

## 리포지토리에 서브모듈 추가하기

`git submodule add` 명령어를 사용하면 리파지토리에 외부 프로젝트를 서브모듈로 추가할 수 있다.

```
$ git submodule add [Git repository URI]
```

만약 처음으로 서브모듈을 추가했다면 `.gitmodules` 파일이 생성된 것을 확인할 수 있다. 해당 파일에는 리파지토리가 사용하는 서브모듈들의 정보가 기록된다.

```
[submodule "TIL"]
    path = TIL
    url = https://github.com/meo-s/TIL.git
```

기본적으로 서브모듈 디렉토리 안에서 발생한 변경 사항의 세부 내용은 추적되지 않는다. 서브모듈을 추가한 리포지토리에서 `git status` 혹은 `git diff` 명령어를 사용하면 이를 확인할 수 있다. 만약 비교적 친절한 출력 결과를 얻고 싶다면 명령어를 사용할 때 `--submodule` 플래그를 지정하면 된다.

```
$ git diff
diff --git a/TIL b/TIL
--- a/TIL
+++ b/TIL
@@ -1 +1 @@
-Subproject commit 82052c5b77d41be3b485d81a3898218ecfe3acf2
+Subproject commit 82052c5b77d41be3b485d81a3898218ecfe3acf2-dirty

$ git diff --submodule
Submodule TIL contains modified content
```

## 서브모듈을 포함한 리포지토리 복제하기

`git clone` 명령어를 사용하여 서브모듈을 포함하는 리포지토리를 복제하면, 서브모듈의 디렉토리는 빈 디렉토리도 복제된다. 서브모듈의 파일까지 복제하기 위해서는 `git submodule init` 명령어를 사용하여 로컬 환경 설정 파일을 구성하고,  `git submodule update` 명령어를 사용하여 실제 프로젝트의 데이터를 불러와 프로젝트에서 사용중인 커밋을 체크 아웃하는 과정을 거쳐야 한다.

```
$ git submodule init
Submodule 'TIL' (https://github.com/meo-s/TIL.git) registered for path 'TIL'

$ git submodule update
Submodule path 'TIL': checked out '82052c5b77d41be3b485d81a3898218ecfe3acf2'

or

# git submodule update --init
```

`git clone` 명령어를 사용할 때 `--recurse-submodules` 플래그를 지정하면 위의 과정을 생략할 수 있다. 그 뿐만 아니라 `--recurse-submodules` 플래그를 지정한 경우, 중첩된 서브모듈을 포함한 모든 서브모듈을 대상으로 위의 작업을 수행한다. `git submodule update --init` 명령어를 사용할 때도 `--recursive` 플래그를 지정하여 동일한 결과를 얻을 수 있다.

```
$ git clone --recurse-submodules ~

or

$ git clone ~
$ cd ~
$ git submodule update --init --recursive
```

## 서브모듈을 포함한 리포지토리에서의 작업

### 서브모듈의 리모트 리포지토리로부터 서브모듈의 변경 사항 가져오기

서브모듈의 변경 사항을 리모트 리포지토리에서 불러오는 방법 중 하나는 일반적인 리포지토리에서와 마찬가로 서브모듈의 디렉토리로 이동해 `git fetch` 명령어를 사용해 원격 리포지토리의 데이터를 불러오고 `git merge` 명령어를 사용해 변경 사항을 반영하는 것이다.

```
$ cd PATH_TO_SUBMODULE_DIRECTORY
$ git fetch
$ git merge origin/master
```

이제 서브모듈을 사용하는 프로젝트 디렉토리로 돌아와 `git diff` 명령어를 사용하면 서브모듈에 새롭게 추가된 커밋들을 확인할 수 있다.

```
$ git diff --submodule
Submodule DbConnector c3f01dc..d0354fc:
  > more efficient db routine
  > better connection routine
```

서브모듈의 원격 리포지토리의 변경 사항을 로컬로 가져오기 위해 항상 서브모듈의 디렉토리로 이동해 `git fetch`와 `git merge` 명령어를 사용하는 것은 번거로울 수 있다. 이 과정은 `git submodule update --remote` 명령어를 사용함으로써 간소화 할 수 있다.

```
$ git submodule update --remote [SUBMODULE NAME]
```

이 명령어는 기본적으로 서브모듈의 원격 리포지토리에 있는 기본 브랜치의 변경 사항을 체크아웃한다. 만약 다른 브랜치를 사용하고 싶다면 `.gitmodules` 설정 파일에 사용할 브랜치의 이름을 명시하면 된다.

```
$ git config -f .gitmodules submodule.DbConnector.branch stable

$ git submodule update --remote
```

`-f .gitmodules` 인자를 생략할 경우 해당 설정은 로컬에서만 반영되므로, 혼자서만 작업하는 프로젝트가 아니라면 `.gitmodules` 파일에 설정을 기록하는 것이 좋다.

이 시점에서 `git status` 명령어를 사용하면 서브모듈에 새로운 커밋들이 존재하는 것을 확인할 수 있다. `git status`에서 서브모듈에 대한 더욱 자세한 정보를 얻고 싶다면 `status.submodulesummary` 값을 설정해주면 된다.

```
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.

Changes no staged for commit:
  (use "git add <file>...") to update what will be commited)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:    .gitmodules
    modified:    DbConnector (new commits)

no changes added to commit (use "git add" and/or "git commit -a")
```

```
$ git config status.submodulesummary 1

$ git status
On branch master
Your branch is up-to-date with 'origin/master'

Changes not staged for commit:
  (use "git add <file>...") to update what will be commited
  (use "git checkout -- <file>...") to discard changes in working directory

    modified:    .gitmodules
    modified:    DbConnector (new commits)

Submodules changed but no updated:

* DbConnector c3f01dc...c87d55d (4):
  > catch non-null terminated lines
```

### 서브모듈을 사용하는 프로젝트의 변경 사항 가져오기

서브모듈을 사용하는 프로젝트는 단순히 `git pull` 명령어를 사용하는 것만으로는 모든 변경 사항을 가져올 수 없다.
```
$ git pull
From https://github.com/chaconinc/MainProject
   fb9093c..0a24cfc  master     -> origin/master
Fetching submodule DbConnector
From https://github.com/chaconinc/DbConnector
   c3f01dc..c87d55d  stable     -> origin/stable
Updating fb9093c..0a24cfc
Fast-forward
 .gitmodules         | 2 +-
 DbConnector         | 2 +-
 2 files changed, 2 insertions(+), 2 deletions(-)

$ git status
 On branch master
Your branch is up-to-date with 'origin/master'.
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   DbConnector (new commits)

Submodules changed but not updated:

* DbConnector c87d55d...c3f01dc (4):
  < catch non-null terminated lines
  < more robust error handling
  < more efficient db routine
  < better connection routine

no changes added to commit (use "git add" and/or "git commit -a")
```

`git status` 명령어를 통해 알 수 있듯이 `git pull` 명령어는 서브모듈들의 변경 사항은 불러오지만, 서브모듈들을 업데이트하지는 않는다. 이는 `git status`의 출력에 "modified"와 "new commits" 문구를 통해 알 수 있다. 추가적으로 서브모듈에서 발생한 커밋에서 "<" 문자는 커밋이 MainProject에는 기록되어 있지만 DbConnector의 로컬 체크아웃에는 존재하지 않는다는 것을 나타낸다. 프로젝트의 업데이트를 마무리하기 위해서는 `git submodule update` 명령어를 사용해 서브모듈들을 직접 업데이트해주어야 한다.

```
$ git submodule update --init --recursive
Submodule path 'vendor/plugins/demo': checked out '48679c6302815f6c76f1fe30625d795d9e55fc56'

$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
nothing to commit, working tree clean
```

로컬에는 존재하지 않았던 서브모듈이 새롭게 추가되는 경우가 있을 수 있기에, `git submodule update` 명령어를 사용할 때, `--init` 플래그를 지정해주는 것이 안전하다. 그리고 중첩된 서브모듈이 있을 수 있기 때문에 `--recursive` 플래그 또한 지정해주었다.

위의 과정은 `git pull` 명령어를 사용할 때 `--recurse-submodules` 플래그를 지정해줌으로써 간소화 할 수 있다. 해당 플래그를 지정하면 `git pull` 명령어의 작업이 끝나면 자동적으로 `git submodule update` 명령어를 실행해준다. 매번 `--recurse-submodules` 플래그를 지정하는 것이 번거롭다면 `submodule.recurse` 설정을 true로 설정하면 된다 (`git clone`에는 적용되지 않는다).

프로젝트가 업데이트되면서 사용하던 서브모듈의 원격 리포지토리의 URI가 변경되는 상황이 생길 수 있다. 이때는 `git submodule sync` 명령어를 사용해 변경된 서보 모듈의 원격 리포지토리의 URI가 로컬 환경 설정 파일에 반영되도록 해주어야 한다.

```
$ git submodule sync --recursive
$ git submodule update --init --recursive
```

### 서브모듈에서 작업하기

지금까지는 `git submodule update` 명령어를 사용해 서브모듈의 리모트 리포지토리에서의 변경 사항을 불러왔다. 이 명령어를 사용하면 Git은 서브모듈의 변경 사항을 얻어오고 업데이트한다. 하지만 서브모듈의 로컬 저장소는 "detached HEAD" 상태가 된다. 이는 서브모듈의 변경 사항을 추척하는 로컬 작업 브랜치가 존재하지 않는다는 것을 의미한다. 이 상태에서 서브모듈에서 발생한 커밋들은 프로젝트 디렉토리에서  `git submodule update` 명령어를 사용하면 소실된다. 이와같은 불상사를 방지하기 위해서는 몇가지 추가적인 작업을 해야 한다.

일반적인 프로젝트와 같이 서브모듈에서 작업하기 위해서는 우선 작업할 브랜치를 체크아웃해야 한다.

```
$ cd DbConnector/
$ git checkout stable
Switched to branch 'stable'
```

"merge" 옵션을 사용해 서브모듈을 업데이트해보자. 이는 `git submodule update` 명령어를 사용할 때 `--merge` 플래그를 지정하면 된다. 이 명령어를 사용하면 리모트 리포지토리에 변경 사항이 존재한다면 해당 변경 사항을 받아와 병합하는 것을 확인할 수 있다.

```
$ cd ..
$ git submodule update --remote --merge
remote: Counting objects: 4, done
remote: Compressing objects: 100% (2/2), done
remote: Total 4 (delta 2), reused 4 (delta 2)
Unpacking objects: 100% (4/4), done.
From https://github.com/chaconinc/DbConnector
   c87d55d..92c7337  stable     -> origin/stable
Updating c87d55d..92c7337
Fast-forward
 src/main.c | 1 +
 1 file changed, 1 insertion(+)
Submodule path 'DbConnector': merged in '92c7337b30ef9e0893e758dac2459d07362ab5ea'
```

이제 로컬과 원격 리포지토리에 동시에 변경 사항이 있는 경우, `git submodule update`가 어떻게 동작하는지 살펴보자.

```
$ cd DbConnector/
$ vim src/db.c
$ git commit -am 'Unicode support'
[stable f906e16] Unicode support
 1 file changed, 1 insertion(+)

$ cd ..
$ git submodule update --remote --rebase
First, rewinding head to replay your work on top of it...
Applying: Unicode support
Submodule path 'DbConnector': rebased into '5d60ef9bbebf5a0c1c1050f242ceeb54ad58da94'
```

만약 `git submodule update` 명령어에 `--rebase`나 `--merge` 플래그를 지정하지 않는다면, Git은 리모트 리포지토리의 내용을 불러오고 서브모듈을 "detached HEAD" 상태로 초기화한다. 하지만 서브모듈의 디렉토리에서 작업할 브랜치를 체크아웃했다면 초기화되기 전의 변경 사항은 단순히 작업한 브랜치를 다시 체크아웃함으로써 복구할 수 있다.

### 서브모듈의 변경 사항 Push하기

```
$ git diff
Submodule DbConnector c87d55d..82d2ad3:
  > Merge from origin/stable
  > Update setup script
  > Unicode support
  > Remove unnecessary method
  > Add new option for conn pooling
```

만약 서브모듈의 변경 사항을 push 하지 않고 메인 프로젝트의 변경 사항만을 push 하면 문제가 발생할 수 있다. 서브모듈의 변경 사항은 로컬 리포지토리에만 존재하기 때문에, 메인 프로젝트가 의존하는 서브 모듈의 변경 사항을 다른 사람들은 받아 올 수 없기 때문이다.

이런 사황을 미연에 방지하기 위해, Git으로 하여금 push 작업을 진행할 때 서브모듈들의 변경 사항이 모두 push 되었는 지 확인하도록 할 수 있다. 이는 `git push` 명령어에 `--recurse-submodules=check|on-demand` 플래그를 지정하면 된다. `check` 옵션을 사용하면 서브모듈의 변경 사항이 push 되지 않은 경우 메인 프로젝트의 push 작업을 취소한다.

```
$ git push --recurse-submodules=check
The following submodule paths contain changes that can not be found on any remote:
  DbConnector

Please try

        git push --recurse-submodules=on-demand

or cd to the path and use

        git push

to push them to a remote
```

`check` 옵션의 대안으로는 `on-demand` 옵션이 있다. `on-demand` 옵션을 사용하면 Git은 서브모듈의 디렉토리로 이동해 서브모의 push 작업을 자동으로 수행한다. 만약 push 작업 중 에러가 발생하면 메인 프로젝트의 push 작업을 취소한다.

```
$ git push --recurse-submodules=on-demand
Pushing submodule 'DbConnector'
Counting objects: 9, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (8/8), done.
Writing objects: 100% (9/9), 917 bytes | 0 bytes/s, done.
Total 9 (delta 3), reused 0 (delta 0)
To https://github.com/chaconinc/DbConnector
   c75e92a..82d2ad3  stable -> stable
Counting objects: 2, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (2/2), 266 bytes | 0 bytes/s, done.
Total 2 (delta 1), reused 0 (delta 0)
To https://github.com/chaconinc/MainProject
   3d6d338..9a377d1  master -> master
```

`git push` 명령어를 실행할 때마다 `--recurse-submodules=check|on-demand` 플래그를 지정하는 것이 번거롭다면 `push.recurseSubmodules` 설정을 `check` 또는 `on-demand` 값으로 지정하면 된다.

```
$ git config push.recurseSubmodules check|on-demand
```

## References

[7.11 Git Tools - Submodules, git](https://git-scm.com/book/en/v2/Git-Tools-Submodules)
