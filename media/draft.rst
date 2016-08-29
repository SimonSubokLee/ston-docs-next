.. _media_draft:

Media. v0.1 Draft
******************

.. warning::

   개발용 임시 문서입니다. 설정 중 기본 값에 대해서는 정식버전 릴리스 전까지 변경될 수 있습니다.


약어
====================================

- SES - STON Edge Server
- SMS - STON Media Server


호환성
====================================

- AMS(구 FMS), WOWZA 등과 URL호환성을 가진다.
- SES 경험에 익숙한 고객들에게 되도록 친숙한 표현을 제공한다.


기본값 - 설정 활성화
====================================

- Bandwidth-Throttling
- MP4HLS

가상호스트 개념
====================================

서비스의 기본 단위는 가상호스트이다.
가상호스트 설정은 SES와 동일하다. ::

   # vhosts.xml

   <Vhosts>
      <Vhost Name="www.example.com" Application="exam_vod"> ... </Vhost>
   </Vhosts>

가상호스트를 만들면 HTTP(80)/RTMP(1935) 서비스가 기본으로 활성화된다.
멀티 프로토콜 서비스를 위해 가상호스트 이름과 Application이름을 반드시 같이 넣어 주어야 한다.

- 가상호스트 Name 속성은 유일해야 한다.
- Application 값은 유일해야 한다.


URL 표현
====================================

URL 표현은 Adobe Media Server(구 FMS)를 따르며
파생된 미디어 서버(i.e. WOWZA)들과 호환성을 가진다.
앞서 설정한 가상호스트(www.example.com)의
원본서버 경로가 /subdir/iu.mp4 라면 서비스 주소는 아래와 같다. ::

    // Adobe Flash Player (RTMP)
    Server: rtmp://www.example.com/exam_vod
    Stream: mp4:subdir/iu.mp4

    // Apple iOS device (Cupertino/Apple HTTP Live Streaming)
    http://www.example.com/exam_vod/mp4:subdir/iu.mp4/playlist.m3u8

    // HTTP Pseudo-Streaming (+ Bandwidth-Throttling)
    http://www.example.com/exam_vod/mp4:subdir/iu.mp4

가상호스트의 Prefix 속성을 설정하면 URL 호환성을 더 강화할 수 있다. ::

   # vhosts.xml

   <Vhosts>
      <Vhost Name="www.example.com"
             Application="exam_vod"
             Prefix="http/"> ... </Vhost>
   </Vhosts>

Prefix는 URL에만 추가될 뿐 아무런 역할을 수행하지 않는다.
Prefix가 추가된 상태 주소는 아래와 같다. ::

    // Adobe Flash Player (RTMP)
    Server: rtmp://www.example.com/exam_vod
    Stream: mp4:http/subdir/iu.mp4

    // Apple iOS device (Cupertino/Apple HTTP Live Streaming)
    http://www.example.com/exam_vod/mp4:http/subdir/iu.mp4/playlist.m3u8

    // HTTP Pseudo-Streaming (+ Bandwidth-Throttling)
    http://www.example.com/exam_vod/mp4:http/subdir/iu.mp4

WOWZA의 경우 Application이름 뒤에 application-instance명을 함께 명시하고 있다.
(이 값은 대부분 _definst_ 이다.)
다음 주소에서 대해서도 정상적인 서비스가 가능하다. ::

    // Adobe Flash Player (RTMP) - 동일
    Server: rtmp://www.example.com/exam_vod
    Stream: mp4:http/subdir/iu.mp4

    // Apple iOS device (Cupertino/Apple HTTP Live Streaming)
    http://www.example.com/exam_vod/_definst_/mp4:http/subdir/iu.mp4/playlist.m3u8

    // HTTP Pseudo-Streaming (+ Bandwidth-Throttling)
    http://www.example.com/exam_vod/_definst_/mp4:http/subdir/iu.mp4


멀티 프로토콜에서 가상호스트 찾기
====================================

프로토콜에 따라 클라이언트 요청에 해당하는 가상호스트를 찾는 방식이 다르다.

HTTP
-----------------------------------------------
HTTP 프로토콜에서는 클라이언트가 보낸 Host헤더를 통해 가상호스트를 찾는 것이 가장 표준적인 방식이다. ::

   GET /exam_vod/mp4:http/subdir/iou.mp4 HTTP/1.1
   Host: www.example.com

Host헤더와 일치하는 가상호스트/Alias가 있다면 Application이름이 달라도 서비스는 이루어진다.

다음의 경우 가상호스트를 찾을 수 없다.

- 클라이언트가 IP로 접근한 경우
- Host헤더를 의도적으로 생략한 경우
- 존재하지 않는 가상호스트를 Host헤더로 설정한 경우

이상의 경우에는 Application이름이 일치하는 가상호스트를 찾는다.


RTMP
-----------------------------------------------

RTMP에서는 Application이름만으로 가상호스트를 찾는다.


서비스 포트/프로토콜
====================================

포트와 프로토콜은 1:1 관계이다.
SES처럼 가상호스트끼리 같은 포트를 공유할 수 있다.
단, A가상호스트가 HTTP로 80을 열었다면 B가상호스트는 RTMP로 80을 열 수 없다. ::

    # vhosts.xml - <Vhosts>

    <Vhost Name="www.example.com" Application="exam_vod">
        <Listen>*:80, *:1935</Listen>
    </Vhost>

SMS는 콤마를 구분자로 HTTP, RTMP순서로 포트를 명시한다.
기본 포트로 HTTP는 80, RTMP는 1935를 사용한다.
다음과 같은 표현이 가능하다. ::

    // HTTP=80, RTMP=1935
    <Listen></Listen>

    // HTTP=90, RTMP=1935
    <Listen>*:90</Listen>

    // HTTP=80, RTMP=2222
    <Listen>, *:2222</Listen>

    // HTTP=90, RTMP=2222
    <Listen>*:90, *:2222</Listen>

멀티 가상호스트 예제는 아래와 같다. ::

    # vhosts.xml - <Vhosts>

    <Vhost Name="foo.com" Application="foo">
        <Listen>*:80, *:1935</Listen> // 가능
    </Vhost>

    <Vhost Name="bar.com" Application="bar">
        <Listen>*:80, *:1935</Listen> // 가능
    </Vhost>

    <Vhost Name="wine.com" Application="wine">
        <Listen>*:8080, *:1935</Listen> // 가능
    </Vhost>

    <Vhost Name="soft.com" Application="soft">
        <Listen>*:80, *:8080</Listen> // 불가능
    </Vhost>

    <Vhost Name="ston.com" Application="ston">
        <Listen>*:1935</Listen> // 불가능
    </Vhost>



통계/로그
====================================

준비 중입니다.
