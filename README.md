# daily
데이터 크롤링 프로젝트

오르비 게시글정보에서 특이점을 찾아보자
특정 강사의 이름이 자주 언급된다면 어떤 단어와 결합되는가?


library(rvest)

#for (i in 1:5) {
  
  
  url1="https://orbi.kr/list/tag/group/%EC%9E%85%EC%8B%9C?page="
  #url2="https://orbi.kr/list/tag/group/%ED%95%99%EC%8A%B5?page="
  
  #paste0(url,i)
  #adr1=read_html(paste0(url1,i))
  adr1=read_html(paste0(url1,1))
  #adr2=read_html(paste0(url2,i))
  
  head=adr1%>%html_nodes("div")%>%html_nodes("li")%>%html_nodes(css = ".title")%>%html_nodes("a")%>%html_text()
  body=adr1%>%html_nodes("div")%>%html_nodes("li")%>%html_nodes(css = ".title")%>%html_nodes("a")#%>%html_attr('href')
  #body
  #board=gsub("\n","",body[-c(1:18)])
  ###제목
  orbi=gsub("\n","",head)[-c(1:21)];np1=c()
  for (y in 1:length(orbi)) {
    np1=append(np1,strsplit(body[-c(1:21)]%>%html_attr("href"),"/")[[y]][2])#고유번호
  }
  p1=cbind(trimws(orbi[-c(1:21)]),np1)
  ###닉네임
  nic=adr1%>%html_nodes("div")%>%html_nodes("li")%>%html_nodes("div")%>%html_nodes(css = ".nickname")%>%html_nodes(css = "span")#%>%html_text()
  nic0=nic[grep("style",nic)]%>%html_text()
  nickname=nic0[-c(1:5)]
  
  np0=c()
  for (j in 1:length(nickname)) {
    np0=append(np0,strsplit(body[-c(1:21)]%>%html_attr("href"),"/")[[j]][2])#고유번호
    }
  p=cbind(nickname,np0)
  
  #####게시글
  ui="https://orbi.kr"
  board0=body[-grep("style",body)]%>%html_attr("href")
  board1=board0[-c(1:21)]
   np=c()
  txt=c()
  #cmt=c()
  for (k in 1:length(board1)) {
    script=read_html(paste0(ui,board1[k]))
    content=html_nodes(script,"div")%>%html_nodes(css = ".content-wrap")%>%html_text()
    txt=append(txt,trimws(gsub("\n","",content)))
    ###고유번호
    np=append(np,strsplit(board1[k],"/")[[1]][2])
    #comment=trimws(gsub("\n","",html_nodes(script,"div")%>%html_nodes(css = ".comment-content")%>%html_text()))
    #p1=cbind(comment,strsplit(board1[k],"/")[[1]][2])
    #print(p1)
    #cmt=rbind(cmt,p1)
  }
  df=cbind(txt,np)
  
  #### 댓글
  #html_nodes(script,"div")%>%html_nodes(css = ".comment-content")%>%html_text()
  comment=trimws(gsub("\n","",html_nodes(script,"div")%>%html_nodes(css = ".comment-content")%>%html_text()))
  #comment
  
  ob=list(title=p1,id=p,text=df)
  
#}

현재 얻을수 있는 정보
제목/내용/닉네임/댓글

하지만 회원이 삭제한 글이 남아있어서 url로 접근불가
셀레늄 이용해야하는데 왜인지 안돌아감 



