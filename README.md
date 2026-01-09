# 🛠 Git 핵심 개념 및 실습 마스터 가이드

## 📂 Git 내부 구조 이해하기
Git이 데이터를 어떻게 관리하는지 기초를 다집니다.

- git은 스냅샷을 저장한다.
- tree: 디렉터리, blob: 객체 파일
- HEAD는 현재 브랜치를 가리키는 포인터
```bash
# 작업 디렉터리 생성 및 초기화
$ mkdir git-practice
$ cd git-practice
$ git init

# .git 폴더 및 객체 탐색
$ ls -al .git
$ cat .git/HEAD     # 현재 가리키는 브랜치 참조 확인
$ cat .git/config   # 저장소 로컬 설정 확인
``` 

#### 실습 자료
```bash
# 파일 만들고 커밋
$ echo "Hello Git" > file1.txt
$ git add file1.txt
$ git commit -m "file commit"	

# .git/objects 폴더 확인 (commit, tree, blob 객체 생성됨)
$ find .git/objects -type f 		

# 커밋 객체 및 트리 구조 내부 들여다보기
$ git cat-file -p HEAD           # 커밋 메타데이터 확인
$ git cat-file -p HEAD^{tree}    # 트리(파일 목록) 확인
```

### add와 commit 제대로 이해하기

```bash
# 부분 add 연습
$ echo "Line 1" > file2.txt
$ git add file2.txt					# Staging Area로
$ echo "Line 2" >> file2.txt

$ git diff  			# Working Directory vs Staging
$ git diff --staged		#  Staging vs Repository
``` 

* 브랜치에 생성한 파일이 다르기때문에 각 브랜치로 switch하면 파일 구조가 다르다!

```bash
git add -p file2.txt
e: 편집기 사용
s: 연속된 같은 줄이아닐때 한줄씩 추가할지 선택하기

# 2. 커밋 수정하기 (기존 마지막 커밋에 file2.txt를 추가)
#  기존 커밋에 추가하고 커밋 메시지 바꾸기
$ git commit --amend "add file2.txt and commit message modify"
# 기존 커밋에 파일만 추가하기
$ git commit --amend --no-edit

# 3. git restore !!
# 강제로 Staging area 있는 파일 내용 Working Direcotry에 update하기
$ git restore --staged file3.txt

# 강제로 Repository에 있는 파일 내용 Working Direcotry에 update하기
$ git restore file2.txt

```

정리
- add는 "스냅샷 준비", commit은 "스냅샷 저장"
- Working Direcotry: 현재 작업중인 파일들
- Staging Area: 스냅샷 준비, git add된 파일들
- Repository: 스냅샷 저장, git commit 파일들

### 브랜치 완전 정복
실습 목표: 3개의 브랜치를 만들어서 각각 다른 파일을 커밋하고, 브랜치 구조를 시각화하기
``` bash 
# 1. 브랜치 feature-a,b,c를 만들고 텍스트 파일을 만들고 add, commit 하는 단계까지 각각 진행했다.
$ git checkout -b feature-a
$ echo "A" > a.txt 
$ git add .
$ git commit -m "Add A"

# 그래프로 확인
git log --graph --oneline --all --decorate
``` 

- 브랜치는 포인터일 뿐이기에 가볍다


## 📂 협업의 기본 (merge, rebase, 충돌)

### Merge 마스터
```bash
# 3-way Merge 실습 환경 만들기
$ git checkout -b feature-x
$ echo "X" > x.txt
$ git add . && git commit -m "X"

# master 변경
$ git checkout master
$ echo "Master task" > master.txt
$ git add . && git commit -m "Master task"

$ git merge feature-x
$ git merge --no-ff feature-x		# 병합햇다는 기록 남기기
```
- Fast-forward Merge: master에서 브랜치를 딴 이후로 main에 새로운 커밋이 하나도 없을 때 발생
- 3-way Merge: master과 feature 양쪽 모두에 새로운 커밋이 있을 때 발생 >> 충돌 발생할 수 있음

### Rebase 이해하기
```bash# 여러 개 커밋 만들기
$ git checkout -b test-rebase
$ echo "1" > test.txt && git add . && git commit -m "커밋1"
$ echo "2" >> test.txt && git add . && git commit -m "커밋2"
$ echo "3" >> test.txt && git add . && git commit -m "오타 수정"
$ echo "4" >> test.txt && git add . && git commit -m "커밋3"

# 합치기
$ git rebase -i HEAD~4
- pick: 그대로 냅두기
- s(squash): 위 커밋과 합칩
- r(reword): 커밋 메시지 수정
- d(drop): 커밋 삭제
```
- 남들이 먼저 한 작업을 내 작업 아래에 밑거름으로 깔고, 내 작업은 그 위에 새롭게 쌓는다
- SVN update 작업과 비슷하다. 
- 순서가 main의 커밋이 내가 커밋한 시점보다 이후에 있어도 rebase를 하면 내가 커밋한 작업 이전에 main의 커밋한 기록이 쌓이고 그다음 내가 커밋한 내용이 위에 덧붙여진다.

### 충돌 해결 연습
```bash
$ git checkout -b conflict-branch master~1	# master~1 브랜치의 마지막 커밋 하나를 제외한 상태
# 고의로 충돌 만들기
$ git add . && git commit -m "Branch version"

$ git checkout master
$ echo "Master vesrion" > conflict.txt
$ git add . && git commit -m "master version"

# Merge 시도
$ git checkout master
$ git merge conflict-branch  # 충돌!

# 파일 확인
$ cat conflict.txt
편집기에서 직접 conflict.txt 수정 후 
$ git add conflict.txt & git commit
```

### 🔍 유용한 설정 및 팁
```bash
$ git branch -m master main 						# 브랜치명 변경
$ git branch --set-upstream-to=origin/main main 	# 자동 업스트림 연결
$ git log --oneline --graph --all 				
$ git log --oneline --graph --all --decorate		# 히스토리 확인
$git ls-tree branch_name --name-only				# 브랜치에 이동하지 않고, 파일 구조를 확인하는 명령어
```