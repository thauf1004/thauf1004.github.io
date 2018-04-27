---
layout: post
title:  "TooManyOpenFiles Exception"
date:   2018-04-26 10:01:59 +0900
categories: forMyFuture
---

Exceiption :
Too many open files
열린 파일 이 너무 많음

증상 :
 실행하는 어플리케이션에서 더이상 파일 또는 소켓을 읽기가 불가능 하다.

문제 원인 :
 근본적인 원인은 Linux 나 Unix 에서 허용하는 **File Description** 의 갯수가 허용치를 초과했기때문에 발생한다.

해결 방안 :
 우선적으로 Linux 나 Unix 의 **File Description** 의 갯수를 확인한다.
 ulimit -a
{% highlight ruby %}
ulimit -n 2560
matamel-MacBookPro:etc matamel$ ulimit -a
core file size (blocks, -c) 0
data seg size (kbytes, -d) 6144
file size (blocks, -f) unlimited
max locked memory (kbytes, -l) unlimited
max memory size (kbytes, -m) unlimited
open files (-n) 2560
pipe size (512 bytes, -p) 1
stack size (kbytes, -s) 8192
cpu time (seconds, -t) unlimited
max user processes (-u) 266
virtual memory (kbytes, -v) unlimited
{% endhighlight %}

그리고 file open 갯수를 늘린다.
일단 root 계정이 필요하다. **/etc/security/limits.conf** 에 다음내용을 추가한다.
{% highlight ruby %}
- soft nofile 10240
- hard nofile 20480
{% endhighlight %}
>/etc/security/limits.conf

하지만 근본적인 문제가 해결되지 않았다. 시스템에 OpenFile 갯수는 무한대로 증가하는상황. 운용중인 KILL만이 답이엇다.

현 프로젝트에서 발생한 문제는 itext 라는 라이브러리가 extern font 를 가져오기위해 File Open 을 한 이후, Close 를 대놓고 하지 않는다.
{% highlight ruby %}
//TODO: For embedded fonts, the underlying data source for the font will be left open until this TrueTypeFont object is collected by the Garbage Collector.  That may not be optimal.
        if (!embedded) {
            rf.close();
            rf = null;
        }
{% endhighlight %}
>해당 소스에 대한 주석 항상 embeded = true 이다.

결국 이 라이브러리를 사용하는 API가 호출 될 때마다 File을 Copy 한 후, File Open 하는 구조로 되어있다.
그리고 이 문제에 대해 근본적인 해결책은 아직 마련하지 못했다.

라이브러리를 수정해서 build








[아이디인큐]: https://www.jobplanet.co.kr/companies/68173/info/%EC%95%84%EC%9D%B4%EB%94%94%EC%9D%B8%ED%81%90
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
