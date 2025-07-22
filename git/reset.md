push 하면 reset 불가능. 

git reset [옵션] <commit id> 

리셋은 과거로 돌아가는 것임. 

내가 abc라는 파일이 있는데 
reset을 하면 c라는 파일 내용 사라짐. 
c 내용 지울 것이냐? 아니면 c 내용 남길 것이냐. 결정할 수가 있음. 이게 reset에 주어지는 옵션들... 

--soft, --mixed, --hard

reset은 역사를 없애는 것이고 역사로 인해서 만들어진 파일들은 어떻게 할래? 임 . 


soft 
commit은 사라짐, 파일은 존재 

hard 
파일도 이력도 흔적도 없이 사라짐. 

mixed(default)
지우지는 않는데 추가하지도 않겠다. 