# daily

<h2>'간결하고 매혹적인' 제목을 찾아서</h2>

## 목표

<p>특정 강사의 이름이 어떤 단어와 주로 결합되는지 관찰한다면, 어떤 평가를 받고있는지 알 수 있을 것이다.</p>

## R-code

<pre><code>
library(rvest)
for (i in 1:5) {


url1="https://orbi.kr/list?page="

adr1=read_html(paste0(url1,i))

head=adr1%>%html_nodes(css = ".list-text")%>%html_nodes(css = ".title")%>%html_nodes("a")
body=adr1%>%html_nodes(css = ".notice")%>%html_nodes(css = ".list-text")%>%html_nodes("a")

###제목
orbi=gsub("\n","",head%>%html_text())[-c(1:length(body))]

###닉네임
nic=adr1%>%html_nodes(css = ".author-text")%>%html_nodes(css = ".nickname")%>%html_nodes(css = "span")
nic0=adr1%>%html_nodes(css = ".notice")%>%html_nodes(css = ".author-text")%>%html_nodes(css = ".nickname")%>%html_nodes(css = "span")
nic1=nic[-c(1:length(nic0))]
nickname=nic1[grep("style",nic1)]%>%html_text()

###게시글
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
      message(e)
    },
    warning=function(e){#e 는 경고내용
      this_page<<-NA
      message(e)
    },
    finally = {}
  )
  content=html_nodes(script,"div")%>%html_nodes(css = ".content-wrap")%>%html_text()
  txt=append(txt,trimws(gsub("\n","",content)))
  ###고유번호
  np=append(np,strsplit(link[k],"/")[[1]][2])
}
df=cbind(txt,np)
contents=data.frame(trimws(orbi),nickname,df)

### 댓글
commentf<-function(t){
  tryCatch(#예외처리함수
    {
      #expr
      script<-read_html(paste0(ui,link[t]))
      this_page<<- script#전역변수 this_page
    },
    error=function(e){#e는 오류내용
      this_page<<-NULL
      message(e)
    },
    warning=function(e){#e 는 경고내용
      this_page<<-NA
      message(e)
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

write.csv(contents,paste0("d:/title&text_page_", i, ".csv"))
write.csv(fst,paste0("d:/comment_page_", i, ".csv"))
}

</code></pre>

-------------------------------------

하루치 정보로는 턱없이 부족하다. 이제 위 코드를 매일 특정시간에 실행시켜서 데이터를 축적해야한다. <br>
자동화를 위해 taskscheduleR 패키지를 사용한다. 그전에 system.file(package= "taskschedulR")를 사용해서 패키지가 저장된 주소를 찾는다.

![캡처](https://user-images.githubusercontent.com/49007889/56306682-105fd400-617e-11e9-9696-e86cd6b33faa.PNG)

script를 taskschduelR의 extdata 폴더에 저장한 다음 아래 코드를 실행한다.

--------------------------

<pre><code>
if(!require(taskscheduleR))install.packages("taskscheduleR")
library(taskscheduleR)
myscript <- system.file("extdata", "ytb_title.R", package = "taskscheduleR")
taskscheduler_create(taskname = "dailyload", rscript = myscript, 
                     schedule = "DAILY", starttime = "10:30", startdate = format(Sys.Date()+1, "%Y/%m/%d"))

#mylog <- system.file("extdata", "ytb_title.log", package = "taskscheduleR")
#cat(readLines(mylog), sep = "\n")
</code></pre>

-----------------------------
