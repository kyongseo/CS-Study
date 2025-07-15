### 🧨 문제 상황
- 문제
  - GitHub에 Push 하려고 했는데 다음과 같은 오류 발생
  ```angular2html
    error: File gradle-8.12.1-bin.zip.1 is 130.23 MB; this exceeds GitHub's file size limit of 100.00 MB
    GH001: Large files detected.
    ```
- 원인
  - `gradle-8.12.1-bin.zip.1` 파일이 Git history 상에 존재하며 commit 용량이 100MB 이상이기 때문에 GitHub에서 push를 거부함

### ✅ 해결 과정
#### 1. 해당 zip 파일 Git 기록에서 완전 삭제
```angular2html
# git-filter-repo 설치 필요 (최초 1회)
# pip install git-filter-repo

# 1. 기존 작업 디렉토리 백업 or 새로 clone
cd ~/work
rm -rf toy-project-02-clean
git clone https://github.com/kyongseo/toy-project-02.git toy-project-02-clean
cd toy-project-02-clean

# 2. 대용량 zip 파일 Git history에서 제거
git filter-repo --path gradle-8.12.1-bin.zip.1 --invert-paths
```
#### 2. GitHub remote 재설정 (origin 오류 방지)
```angular2html
# 원격 origin 재설정
git remote remove origin
git remote add origin https://github.com/kyongseo/toy-project-02.git
```
#### 3. 강제 push로 업데이트된 Git history 반영
```angular2html
git push --force origin develop
```
### 참고
1. GitHub는 100MB 넘는 파일을 업로드할 수 없어요. 
2. 만약 대용량 파일이 필요하다면 Git LFS 를 사용하는 걸 고려하세요. 
3. 앞으로는 .gitignore에 다음과 같이 추가해두면 안전
```angular2html
*.zip
*.tar.gz
*.bin
*.exe
```

