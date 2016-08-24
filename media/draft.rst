.. _media_draft:

Media. Draft
******************

.. warning::

   개발용 임시 문서입니다.


약어
====================================

- SES - STON Edge Server
- SMS - STON Media Server


SES 호환성 방향
====================================

- 웹과 미디어는 태생이 달라 개념이 상이한 부분이 많다.
- 서비스 단위의 표현도 웹에서는 Virtual Host, 미디어에서는 Application이 de-facto로 자리 잡았다.
- 프로토콜의 표준화방향도 웹을 중심으로 이루어지고 있으므로 웹에 무게를 둔다.
- 결과적으로 SES와 유사한 표현을 가지게 되는 것이 자연스럽다.
- 이미 SES에 친숙한 고객들에게 굳이 생소한 표현을 제시할 이유는 없다.


가상호스트 개념
====================================

가상호스트 설정은 SES와 동일하다. ::

   # vhosts.xml

   <Vhosts>
      <Vhost Status="Active" Name="www.example.com"> ... </Vhost>
   </Vhosts>

가상호스트를 만들면 아래와 같은 서비스가 기본으로 가능하다.

- HTTP Pseudo Streaming (feat. bandwidth-throttling)
- HLS
- RTMP


서비스 포트/프로토콜
====================================

포트와 프로토콜은 1:1 관계이다.
SES처럼 가상호스트끼리 같은 포트를 공유할 수 있다.
단, A가상호스트가 HTTP로 80을 열었다면 B가상호스트는 RTMP로 80을 열 수 없다. ::

    # vhosts.xml - <Vhosts>

    <Vhost Name="www.example.com">
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

    <Vhost Name="foo.com">
        <Listen>*:80, *:1935</Listen> // 가능
    </Vhost>

    <Vhost Name="bar.com">
        <Listen>*:80, *:1935</Listen> // 가능
    </Vhost>

    <Vhost Name="wine.com">
        <Listen>*:8080, *:1935</Listen> // 가능
    </Vhost>

    <Vhost Name="soft.com">
        <Listen>*:80, *:8080</Listen> // 불가능
    </Vhost>

    <Vhost Name="ston.com">
        <Listen>*:1935</Listen> // 불가능
    </Vhost>


URL - 기본
====================================

독자적인 URL표현을 기본으로 한다. ::

    # vhosts.xml - <Vhosts>

    <Vhost Name="www.example.com">
        <Listen>*:80, *:1935</Listen>
    </Vhost>

원본파일이 /video/iu.mp4이라면 다음과 같이 표현이 가능한다. ::

   // HTTP - Pseudo Streaming
   http://www.example.com/video/iu.mp4

   // HLS
   http://www.example.com/video/iu.mp4/mp4hls/index.m3u8

   // RTMP
   rtmp://www.example.com/video/iu.mp4


URL - Application 호환
====================================

기존 미디어서버는 Domain(=Virtual Host)개념이 아니라 Application으로 구성되어 있다.
Application은 주소(IP or Domain)뒤의 첫 번째 디렉토리에 배치된다. ::

    // Application = baseball
    rtmp://sports.com/baseball/highlight.mp4
    rtmp://1.1.1.1/baseball/highlight.mp4

    // Application = football
    rtmp://sports.com/football/highlight.mp4
    rtmp://1.1.1.1/football/highlight.mp4

    // Application = photo
    rtmp://sports.com/photo/highlight.mp4
    rtmp://1.1.1.1/photo/highlight.mp4

SMS에서는 Application개념이 없기 때문애 Sub-Path기능으로 호환한다. ::

   <Vhost Name="baseball.com" />
   <Vhost Name="football.com" />
   <Vhost Name="photo.com" />

   <Vhost Name="sports.com">
      <Sub Status="Active">
         <Path Vhost="baseball.com">/baseball/<Path>
         <Path Vhost="football.com">/football/<Path>
         <Path Vhost="photo.com">/photo<Path>
      </Sub>
   </Vhost>

   <Default>sports.com</Default>

각각의 가상호스트에 직접 접근도 가능하다. ::

   rtmp://baseball.com/highlight.mp4
   rtmp://football.com/highlight.mp4
   rtmp://photo.com/highlight.mp4
