## Git ? GitHub?

<img src="https://blog.kakaocdn.net/dn/pbPzJ/btqDuqUUNBt/2nQHXRRCgz7qDUpt7K8fv1/img.png" style=width:30%></img>
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FeuksHz%2Fbtrkikzsvnn%2Fp5AdNUCmxGuZbnu4Vq98Y0%2Fimg.webp" style=width:30%></img>

### Git 

 형상 관리 도구 중 하나!    
 형상관리 도구란 소프트웨어 프로젝트에서 나오는 결과물을 관리하는 소프트웨어라고 한다..? ~~응? 뭐라고?~~  
자 다시 간단하게 말해보자면 우리가 작성한 코드들의 변화를 기록하고 관리해 누가, 어디를, 언제, 어떻게, 수정했는지 쉽게 파악하고 버전 별로 관리할 수 있다.
_한마디로 협업 필수 툴!_

### GitHub

원격 저장소를 이용해 언제 어디서든 git을 사용하는 협업 프로젝트를 진행할 수 있게 도와주는 웹 호스팅 플랫폼이다. _git과 github는 엄연히 다른것!_


### 그렇다면 Git은 어떻게 사용할까?
 
- git을 사용하기 위해선 당연하게 git을 설치돼있어야 한다. <a href="https://git-scm.com/downloads">Git Download</a> 
git을 사용하기 위해 터미널에서 해당 경로에 git init으로 git을 초기화 시켜주고 git config --global user.name "", git config --global user.email ""으로 사용자 정보를 입력한다. (git config --list로 확인할 수 있음)
 
- working directory에 로컬 저장소가 초기화 됐다면 이후 부터 작성하는 코드들은 전부 git이 추적하게 된다. 변경, 수정된 코드들이 있다면 해당 파일을 git이 변경됐다고 표시를 해주고 이를 git add로 Staging Area에 올리고 git commit을 통해 로컬 저장소에 스냅샷이 된다.
- 다음으론 git push를 통해서 원격 저장소인 gitHub에 올릴 수 있고 스냅샷된 지점마다 되돌릴 수 있다. (git add, git commit한 부분들도 되돌릴 수 있음)
- git이 추척하는 파일들의 상태는 4가지로 나뉘는데
  - Untracked = 맨 처음 생성하는 파일의 상태
  - Tracked = 한번이라도 Stage, Commit에 올라간 파일 중 아래 3가지 상태가 있다.
    - Modified = 내용이 한 글자라도 바뀐 파일
    - Staged = Modified 중에서 Staging Area에 저장된 상태
    - Unmodified = 이전 Commit 이후 내용이 변경되지 않은 파일
  
<img src="https://s1.md5.ltd/image/09a037f13ac30a80b64099e3945a3610.jpg" style="width:80%"></img>
- git을 통해 이렇게 코드들을 추적 관리 하고 협업을 진행할 때에는 보통 원격 저장소에서 clone을 받아와 로컬 저장소에서 개인단위 개발을 진행하게 된다.


### Git에서 자주 사용하는 명령어

- `git init` : git 초기화(생성)
- `git add 스테이징에 올릴 파일` : 파일을 스테이지에 올림, 수정한 파일을 추가할 땐 -u 하면 됌
- `git reset HEAD file` : 스테이징에 add된 파일을 삭제, 파일명이 없을 경우 전체 취소
- `git remote -v` : 연결된 원격 저장소를 확인
- `git remote add origin 원격저장소URL` : 원격 저장소 추가 
- `git clone path` : 저장소를 복제, 복사해 온다. path에 가져올 저장소의 경로를 입력해줘야함
- `git checkout -t remotePath/branchName` : 원격 브랜치 선택
- `git branch name` : 브랜치 생성, name에 생성할 이름 입력
- `git branch -r` : 원격 브랜치 목록 보기
- `git branch -a` : 로컬 브랜치 목록 보기
- `git checkout branchName` : 브랜치 전환
- `git branch -d name` : 로컬 브랜치 삭제하기, name에 삭제 할 브랜치 이름 입력
- `git add filename` : Staging에 수정된 파일 추가하기, filename에 수정된 파일 이름 입력
- `git commit -m "comment"` : Commit 시 해당 commit에 대한 설명 추가, comment에 설명 입력
- `git push remoteName branchName` : add->commit 한 코드, 파일 원격 저장소로 보내기
- `git pull remoteName branchName` : 원격 저장소의 해당 branch의 내용을 fetch로 가져와 merge까지 실행(원격 저장소의 소스를 가져온 뒤 로컬 소스와 합침) 
- `git fetch remoteName` : 원격저장소의 소스를 가져옴

### GitHub를 사용하는 방법은?

- 먼저 Git과 GitHub가 서로 접근할 수 있도록 SSH Key를 발급해줘야 한다. (Https로도 접근할 수 있으나 보안상 좋지 않음!)
  - Window 기준 gitbash를 열고 ssh-keygen -t rsa 입력 (-t rsa 암호화 타입 지정) 입력해서 ssh key 생성
  - 내 기준으로 C:\Users\USER\.ssh 에 ssh key가 생성됐고 `cat ~/.ssh/id_rsa.pub` 입력 후 나오는 해쉬코드 복사
  - GitHub의 settings의 SSH and GPG keys 탭으로 이동 후 SSH keys에 붙여넣고 추가하면 끗!
- GitHub repository를 만들고 ssh 주소를 git과 연결한다음 git에서 push할 때 GitHub에 있는 repository로 push하면 올라가 있다! 두둥!!


GitHub로 협업을 제대로 해본적이 없기 때문에.. 아주아주 잘 설명이 돼있는 0seo8님의 글을 참고하길 바란다. <a href="https://velog.io/@0seo8/Git-GitHbub%EB%A1%9C-%ED%98%91%EC%97%85%ED%95%98%EA%B8%B0">0seo8님의 [Git]GitHbub로 협업하기!</a>