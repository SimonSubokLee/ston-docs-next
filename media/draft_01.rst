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

서비스 범위
====================================

프로토콜에 따라 지원되는 파일규격은 아래와 같다.

======================== =============== =============== ===============
프로토콜                   확장자            Video Codec     Audio Codec
======================== =============== =============== ===============
RTMP                     .mp4            H.264           AAC
Apple HLS                .mp4            H.264           AAC
HTTP Pseudo-Streaming    .mp4            H.264           AAC
======================== =============== =============== ===============

지원되지 않거나 분석할 수 없는 파일에 대한 요청은 다음과 같이 예외처리된다.

====================== ===============================
프로토콜                 예외처리 응답
====================== ===============================
RTMP                   NetStream.Play.StreamNotFound
Apple HLS              406 Not Acceptable
HTTP Pseudo-Streaming  406 Not Acceptable
====================== ===============================


기본설정
====================================

- Bandwidth-Throttling (Active)
- MP4HLS (Active)

가상호스트
====================================

가상호스트는 서비스의 기본단위이다.
클라이언트가 접속하는 URL을 가상호스트 이름과 매칭하여 서비스할 가상호스트를 결정한다. ::

   # vhosts.xml

   <Vhosts>
      <Vhost Name="www.example.com"> ... </Vhost>
   </Vhosts>

   <Vhosts>
      <Vhost Name="/vod"> ... </Vhost>
   </Vhosts>

   <Vhosts>
      <Vhost Name="www.example.com/vod"> ... </Vhost>
   </Vhosts>

가상호스트 이름(Name)은 3가지 형태로 구성할 수 있다.

- 도메인 (www.example.com) + 디렉토리(/vod)
- 도메인 (www.example.com)
- 디렉토리(/vod)

디렉토리는 1-depth만 가능하다.
가상호스트를 검색할 때는 가장 명확한 표현을 우선적으로 선택한다.
URL에 따라 가상호스트가 선택되는 예제는 아래와 같다. ::

   http://www.example.com/vod/video.mp4
   // "www.example.com/vod" 선택

   http://www.example.com/sports/highlight.mp4
   // "www.example.com" 선택

   http://www.foobar.com/vod/vidoe.mp4
   // "/vod" 선택

   http://www.foobar.com/sports/highlight.mp4
   // 404 Not Found

``<Alias>`` 도 다음과 같이 표현이 가능하다. ::

   # vhosts.xml - <Vhosts>

   <Vhost Name="example.com">
      <Alias>another.com</Alias>
      <Alias>*.sub.example.com</Alias>
      <Alias>sports.com/highlight</Alias>
      <Alias>/myvideo</Alias>
   </Vhost>

패턴표현(*)은 도메인에만 사용할 수 있으며 가상호스트 이름과 ``<Alias>`` 는 유일해야 한다.


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
Prefix가 추가된 주소는 아래와 같다. ::

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


가상호스트 매칭정책
====================================

HTTP/RTMP 클라이언트 요청을 처리할 가상호스트 선택은 다음 우선순위를 따른다.

1. Application명과 일치하는 가상호스트를 찾는다.
2. (HTTP인 경우) Host헤더와 일치하는 가상호스트를 찾는다.
3. 기본 가상호스트를 찾는다.

이상의 순서에서도 가상호스트를 선택할 수 없다면 각 프로토콜에 맞도록 예외처리 한다.


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

v0.2에서 지원됩니다.
