## Git 기본. 

중앙 : like Wiki
버전은 중앙 서버에 저장되고 중앙 서버에서 파일을 가져와 다시 중앙에 업로드

분산 : 버전을 여러 개의 복제된 저장소에 저장 및 관리 

- Working Directory : 폴더. 프로젝트의 루트.. 물리적 저장 루트. 
- Staging Area : 작업 공간, 추가를 하겠다. add . <-- 
- Repository : 최종적으로 프로젝트에 저장하면 Repo에 들어감. 현재 버전이 기록되어 있는 Repo임. 

 버전 이력과 파일들이 영구적으로 저장되는 영역 . 모든 버전과 변경 이력이 기록됨. commit,
 snapshot -> 최종 버전. 

git push -u orign master 
-u : setUpStream 얘가 항상 원본일 것이야. 
지금 push랑 pull을 할 때는 항상 그 내용을 지정해주는데 
-u default 값 지정. 

gitignore : Git에서 특정 파일이나 디렉토리 추적하지 않도록 설정하는 데 사용되는 텍스트 파일 

나중에 gitignore 만들면 트래킹 해제 안됨. 
