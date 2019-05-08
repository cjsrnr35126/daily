# 데이터 크롤링 프로젝트
------------------

## 1.why(왜 이 주제를 선택하였는가?)

![123](https://user-images.githubusercontent.com/49007889/57361504-7ce75500-71b7-11e9-9008-95d0b1559cb6.PNG)

![캡처](https://user-images.githubusercontent.com/49007889/57361224-e9ae1f80-71b6-11e9-8ec8-4086b8d1575b.PNG)


알려려지 않은 불법 홍보가 얼마나 더 있을지 알 수 없다. 어떻게 해야 참된 정보를 얻을것 인가?

---------------------

## 2.what(무엇을 할 것인가?)

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
  커뮤니티 글이 갱신되는 속도를 관찰한 결과 1시간에 1~2 페이지 정도의 속도로 갱신되었다. 오차를 고려하여 5시간마다 5페이지를 크롤링하였다.
</p>

-----------------------------

#### 시행착오/ 4. when(언제 시작했는가?) -> since 04-29 
<p>
  주제를 정하고 난 후 가장 먼저 제목크롤링을 시도하였다. 비교적 쉽게 성공하였으나 '공지'가 문제였다. '공지'는 페이지마다 동일하게 존재하므로 중복을 막고 온전히 게시글의 제목만을 얻기위해 '공지'를 제거해야 했다.
</p>

![제목 없음1](https://user-images.githubusercontent.com/49007889/57380639-5390ee00-71e4-11e9-8d67-e5f757bd4539.png)

![제목 없음2](https://user-images.githubusercontent.com/49007889/57380977-05301f00-71e5-11e9-95d3-ceefcd1058fc.png)

<p>위 사진에서 공지와 일반글의 차이점을 알 수 있다. 모든 제목을 읽은다음 공지 제목 수 만큼 데이터를 삭제하여 문제를 해결 하였다. <br>
  게시글 제목 다음에는 닉네임에 도전하였다. 글쓴이에 대한 정보는 목표를 찾아가는데 중요한 이정표 역할을 할 것이다. 
</p>

-------------

![제목 없음3](https://user-images.githubusercontent.com/49007889/57384050-d3ba5200-71ea-11e9-817d-b17c40743128.png)

<p>
  닉네임에 해당하는 부분을 <span> 태그가 감싸고 있는 모습이다. 특이점이 있다면 속성이 class가 아닌 style이라는 점이다.<br> grep함수를 이용하여 style 속성에 해당하는 부분을 선별하고, 제목과 마찬가지로 전체 글의 닉네임을 읽은뒤, 공지 글의 닉네임에 해당하는 데이터수 만큼 제외시켰다.
</p>
  
--------------

![txt](https://user-images.githubusercontent.com/49007889/57383876-863de500-71ea-11e9-91e0-92b21db816d6.png)
![http error 404](https://user-images.githubusercontent.com/49007889/57383880-8807a880-71ea-11e9-9cbe-ee2240481be3.png)

<p> 비교적 쉬운 과제를 해결하고 난 후 큰 복병이 기다리고 있었다. 이제 게시글 내용을 가져와야한다. 게시글의 경우 제목에 링크된 주소로 넘어가서 읽어야한다. 하지만 '회원에 의해 삭제된 글'의 링크의 경우 존재하지 않는 주소 취급이기에 HTTP error 404가 발생한다. 때문에 이러한 링크들은 무시하고서 온전한 것들만 가져와야하는데 그렇게 되면 데이터가 일대일 대응이 되지않아 문제가 발생 한다. 고민 끝에 각각의 데이터에 고유값을 부여할 수 있다면, 일대일 대응이 아니더라도 특정 데이터에 접근할 수 있다는 결론에 이르게 되었다. 문제는 어떤값을 고유값으로 부여할 지 인데 게시글 주소에 해답이 있었다.
404 에러는 예외처리 함수 tryCatch를 이용하여 해결하였다.</p>

![content](https://user-images.githubusercontent.com/49007889/57387797-f6039e00-71f1-11e9-98d7-daf1bef84ee6.png)

---------------

![comment](https://user-images.githubusercontent.com/49007889/57387800-f734cb00-71f1-11e9-9804-2c09083bc294.png)

<p> 마지막으로 댓글이 남았다. 댓글에는 댓글 닉네임과 댓글내용이 함께 존재한다. 그러면서 게시글에 종속되는 정보 이므로 게시글의 고유값을 가져야한다. 댓글을 읽어오는데 발생하는 문제는 두가지가 있었다. 첫 번째는 앞서 게시글에서 본 것 처럼 '회원에 의해 삭제된 글'일 경우 HTTP error 404가 발생했다. 두 번째는 댓글이 없는경우, 즉 게시글에 대한 댓글 수가 0일때 발생 했다. 첫 번째 문제는 게시글과 마찬가지로 tryCatch 함수를 사용하였고, 두 번째 문제는 댓글이 없는 경우 NA 처리 하였다.</p>

--------------

<p>이제</p>
