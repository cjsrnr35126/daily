# daily
데이터 크롤링 프로젝트

### index

1. why
2. what
3. how




---------------------

## 2.what

<p>특정 강사의 이름이 어떤 단어와 주로 결합되는지 관찰 (긍정 or 부정) 
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
<pre><code>
###taskscheduleR
library(taskscheduleR)
myscript <- system.file("extdata", "orbi.R", package = "taskscheduleR")
taskscheduler_create(taskname = "mt", rscript = myscript,
                     schedule = "MINUTE", starttime = "17:00", modifier = 300,startdate = format(Sys.Date(), "%Y/%m/%d"))

</code></pre>

-----------------------------
