# 데이터 크롤링 프로젝트
------------------

## 1.why

![123](https://user-images.githubusercontent.com/49007889/57361504-7ce75500-71b7-11e9-9008-95d0b1559cb6.PNG)

![캡처](https://user-images.githubusercontent.com/49007889/57361224-e9ae1f80-71b6-11e9-8ec8-4086b8d1575b.PNG)


알려려지 않은 불법 홍보가 얼마나 더 있을지 알 수 없다. 어떻게 해야 참된 정보를 얻을것 인가?

---------------------

## 2.what

<p>커뮤니티의 글과 댓글에서 특정 강사의 이름이 어떤 단어와 주로 결합되는지 관찰 한다.(긍정 or 부정) 
나아가 이것이 진실된 반응인지 아니면 마케팅인지 구분한다.</p>

## 3.how(R-code)

<pre><code>
library(rvest)
for (i in 1:5) {

###read url
url1="https://orbi.kr/list?page="

adr1=read_html(paste0(url1,i))

head=adr1%>%html_nodes(css = ".list-text")%>%html_nodes(css = ".title")%>%html_nodes("a")
body=adr1%>%html_nodes(css = ".notice")%>%html_nodes(css = ".list-text")%>%html_nodes("a")

###title
orbi=gsub("\n","",head%>%html_text())[-c(1:length(body))]
title=trimws(orbi)
###nickname
nic=adr1%>%html_nodes(css = ".author-text")%>%html_nodes(css = ".nickname")%>%html_nodes(css = "span")
nic0=adr1%>%html_nodes(css = ".notice")%>%html_nodes(css = ".author-text")%>%html_nodes(css = ".nickname")%>%html_nodes(css = "span")
nic1=nic[-c(1:length(nic0))]
nickname=nic1[grep("style",nic1)]%>%html_text()

###content-text
ui="https://orbi.kr"
link=head[-c(1:length(body))]%>%html_attr("href")
np=c()
txt=c()

for (k in 1:length(link)) {
  tryCatch(#예외처리함수
    {
      #expr
      script<-read_html(paste0(ui,link[k]))
      this_page<<- script#전역변수 this_page
    },
    error=function(e){#e는 오류내용
      this_page<<-NULL
      #message(e)
    },
    warning=function(e){#e 는 경고내용
      this_page<<-NA
      #message(e)
    },
    finally = {}
  )
  content=html_nodes(script,"div")%>%html_nodes(css = ".content-wrap")%>%html_text()
  txt=append(txt,trimws(gsub("\n","",content)))
  ###Unique number
  np=append(np,strsplit(link[k],"/")[[1]][2])
}
df=cbind(txt,np)
contents=data.frame(title,nickname,df)

###comment
commentf<-function(t){
  tryCatch(
    {
      #expr
      script<-read_html(paste0(ui,link[t]))
      this_page<<- script
    },
    error=function(e){
      this_page<<-NULL
      #message(e)
    },
    warning=function(e){
      this_page<<-NA
      #message(e)
    },
    finally = {}
  )
  comment=trimws(gsub("\n","",html_nodes(script,"div")%>%html_nodes(css = ".comment-content")%>%html_text()))
  xx=html_nodes(script,"div")%>%html_nodes(css = ".meta-wrap")%>%html_nodes("span")
  comment_nick=xx[grep("style",xx)]%>%html_text()
  comment_np=strsplit(link[t],"/")[[1]][2]
  if(length(xx)!=0){
    x=data.frame(comment_nick,comment,comment_np)
  }else{
    x=NA
  }
  return(x)
}
fst=commentf(1)
  for (j in 2:length(link)) {
    fst=rbind(fst,commentf(j))
  }
###save data
write.table(fst,"d:/comment.txt",sep = ",",row.names = F,append = T)
write.table(contents,"d:/title&text.txt",sep = ",",row.names = F,append = T)
}
</code></pre>

-------------------------

<p>
  코드 실행시 D드라이브에 commnet.txt 파일과 title&text.txt 파일이 생성된다.
이를 자동화 시키기위해서 taskschedulR를 활용한다.
</p>

---------------------------

<pre><code>
###taskscheduleR
library(taskscheduleR)
myscript <- system.file("extdata", "orbi.R", package = "taskscheduleR")
taskscheduler_create(taskname = "mt", rscript = myscript,
                     schedule = "MINUTE", starttime = "17:00", modifier = 300,startdate = format(Sys.Date(), "%Y/%m/%d"))

</code></pre>

-----------------------------

<p>
  커뮤니티 글이 갱신되는 속도를 관찰한 결과 약 1시간에 1페이지 정도가 갱신되었다. 그러나 오차를 고려해야 하므로 5시간마다 5페이지를 크롤링하였다.
</p>

-----------------------------

#### 시행착오
<p>
  주제를 정하고 난 후 가장 먼저 제목크롤링을 시도하였다. 비교적 쉽게 성공하였으나 '공지'가 문제였다. '공지'는 페이지마다 동일하게 존재하므로 중복을 막고 온전히 게시글의 제목만을 얻기위해 '공지'를 제거해야 했다.
  
  
  </p>
