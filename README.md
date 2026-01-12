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

#### 💻실습 자료
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

💡 정리
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

### ⚠️충돌 해결 연습
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

>> 편집기에서 직접 conflict.txt 수정 후 
$ git add conflict.txt & git commit
```
<img width="565" height="235" alt="3-1" src="https://github.com/user-attachments/assets/dfff51cc-1145-49be-a572-5d25c00a59eb" />

### Git Flow 실습
```bash
$ git checkout -b develop
$ git checkout -b feature/login develop
$ echo "로그인 기능" > login.txt
$ git add . && git commit -m "feat: 로그인 추가"

# Develop으로 merge
$ git checkout develop
$ git merge feature/login --no-ff

# Release 브랜치
$ git checkout -b release/1.0.0 develop
$ echo "1.0.0" > version.txt
$ git add . && git commit -m "chore: 버전 1.0.0"

# Main으로 merge
$ git checkout main
$ git merge release/1.0.0 --no-ff
$ git tag v1.0.0

# Hotfix
$ git checkout -b hotfix/critical-bug main
$ echo "버그 수정" > bugfix.txt
$ git add . && git commit -m "fix: 긴급 버그 수정"

$ git checkout main
$ git merge hotfix/critical-bug
$ git checkout develop
$ git merge hotfix/critical-bug

$ git branch -d feature/login
$ git branch -d release/1.0.0
$ git branch -d hotfix/critical-bug
```
### 원격 저장소 (Remote)
```bash
$ git remote add origin https://github.com/jowoohyeong/git-practice.git
$ git remote -v

# Push
$ git push origin main
$ git push origin develop

>>  github 웹사이트에서 README.md 수정후

# Fetch: 다른 사람 작업 가져오기 
$ git fetch origin
$ git log origin/main 		# 가져온 커밋 기록보기
$ git diff main origin/main 	# 변경된 코드 비교

# Pull: Fetch + Merge
$ git pull origin main
```
#### 💻실습 과제: Fork와 PR 연습
```bash
# 현재 작업프로젝트 git-practice-second으로 복사하기
$ git clone https://github.com/jowoohyeong/git-practice.git
```
<img width="559" height="118" alt="image" src="https://github.com/user-attachments/assets/64aca65a-958b-48ce-997c-46e9b4124796" />

```bash
# Upstream: Fork 한 원래의 소스 코드 저장소
$ git remote add upstream https://github.com/jowoohyeong/git-practice.git
$ git remote -v   # remote 확인

# git-practice에서 작업 후 push
$ git checkout -b feature/contribution
$ echo "Fork & PR 실습 중" >> contribution.txt
$ git add .
$ git commit -m "feat: 새로운 기여 내용 추가"
$ git push origin feature/contirbution
```
- Step 1. GitHub 웹사이트에서 Pull requests(PR) 탭
- Step 2. Compare & pull request
- Step 3. 변경 내용 확인 후 Create pull request
- Step 4. Merge pull request
- Step 5. 기존 작업소, git-practice에 동기화
<img width="488" height="213" alt="image" src="https://github.com/user-attachments/assets/baaf9c16-f469-4d32-bc7b-fc4d7d7de96c" />

### 히스토리 관리
```bash
$ git log --graph --oneline --all  
$ git log --author="jwh"       # jwh 변경 기록
$ git log --since="1 week ago" # 기간
$ git log --grep="fix"         # 커밋 내용 검색
$ git log -p contribution.txt  # 특정 파일 변경 이력
```
- Line 5: 오류 커밋 생성
```bash
# 파일 하나 생성 (복붙하자..)
touch app.py
# 1~4번째 정상 커밋
for i in {1..4}; do echo "Line $i: Good" >> app.py; git add app.py; git commit -m "Commit $i"; done
# 5번째 버그 커밋 (일부러 잘못된 코드 삽입)
echo "Line 5: BUG HERE!" >> app.py; git add app.py; git commit -m "Commit 5 (Buggy)"
# 6~10번째 커밋 (버그가 있는 상태로 계속 진행)
for i in {6..10}; do echo "Line $i: Working..." >> app.py; git add app.py; git commit -m "Commit $i"; done
```
<img width="846" height="223" alt="image" src="https://github.com/user-attachments/assets/31955ff8-ffcf-4083-b2c8-389aa24f16df" />

```bash bisect 탐색 과정
$ git bisect start
$ git bisect good 9ff9781     # 9ff9781 Commit 1 해쉬 값
$ git bisect bad              # (Buugy) 가 현재 시점에 있으면 실행
$ git bisect reset            # 탐색 종료

# 오류 시점 수정 및 커밋
$ vi app.py
$ git add app.py
$ git commit -m "Fix: modify bug found by bisect"     
```
## 📂실전 & 고급
### Reset vs Revert vs Restore
```bash
# 테스트용 커밋 생성
$ echo "learning reset" > test.txt
$ git add test.txt
$ git commit -m "Test: Reset and Restore"

# Restore 테스트!
$ echo "add line" >> test.txt   # 내용 추가하기!
$ git store test.txt            # stage에 올라와 있는 test.txt으로 되돌리기
$ cat test.txt

# Reset 테스트!
# 3가지 모드 테스트 후 backup 진행, reset 명령 되돌리기
backup : git reset --hard HEAD@{1}

# --soft: 최근 commit 되돌리기
$ git reset --soft HEAD~1
# --mixed(기본값): 최근 commit 되돌리기 및 해당 파일 add 되돌리기
$ git reset HEAD~1
# --hard: 모두 제거, 파일 자체 제거
$ git reset -hard HEAD~1

# Revert 테스트
# test.txt 에 내용 추가, 커밋
$ echo "This is a BUG!" >> test.txt
$ git add test.txt
$ git commit -m "Commit with BUG"

$ git revert HEAD          # vi 편집기 수정하지 않고(커밋 메시지 수정해도 상관없음),  :wq
$ git log oneline -n 5     # 5줄만 보고싶어서 그냥 추가했고, 취소 커밋메시지 확인
$ cat test.txt

# 커밋 기록 반복적으로 확인하자!
$ git log --oneline -n 5
$ git diff --staged --name-only  
$ git diff --cached --name-only
```
💡 간편 정리
- reset: 커밋 내용 지우기!
- revert: 커밋한 내용이 오류가 있어 되돌린다는 커밋을 추가하고 해당 커밋전으로 파일을 되돌리기!
- restore: 커밋한 내용으로 되돌리기!


## 💡 유용한 설정 및 팁
```bash
$ git branch -m master main 						# 브랜치명 변경
$ git branch --set-upstream-to=origin/main main 	# 자동 업스트림 연결
$ git log --oneline --graph --all 				
$ git log --oneline --graph --all --decorate		# 히스토리 확인
$ git ls-tree branch_name --name-only				# 브랜치에 이동하지 않고, 파일 구조를 확인하는 명령어
```
