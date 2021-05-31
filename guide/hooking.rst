.. _hooking:

Appendix B: Hooking 함수
***********************

M2 후킹(Hooking) 함수를 이용하면 자유롭게 HTTP 트랜잭션을 제어할 수 있다. 

-  클라이언트 HTTP 요청에 대한 재정의
-  캐싱키 정의
-  원본 HTTP 요청에 대한 재정의


.. figure:: img/m2_34.png
   :align: center



1.  클라이언트 요청을 M2의 Hooking 함수로 보낸다.

2.  Hooking 함수는 캐싱키와 ``MISS`` 시 원본에 보낼 요청을 정의하여 응답한다. ``200 OK`` 가 아니라면 M2RT는 ``403 Forbidden`` 을 응답한다.

3.  재정의된 캐싱키를 사용해 캐싱엔진에서 찾는다.

4.  원본 요청 시 Hooking 함수가 정의한 요청 모델을 참조해 원본서버에 요청한다.


.. note::

   Purge의 경우 2, 3 항에서 사용하는 캐싱 키를 사용한다.


.. _hooking-conf:

설정
====================================

아래와 같이 가상호스트 설정파일 ``/usr/local/ston/vhosts.xml`` 의 확장 커스터마이징으로 구성한다.  ::

   # vhosts.xml - <Vhosts><Vhost><Options>

   <Customize Name="M2_HOOK1">
      <OriginOptions>
         <ConnectTimeout>3</ConnectTimeout>
         <ReceiveTimeout>3</ReceiveTimeout>
      </OriginOptions>
      <Log Dir="/cache_log">
         <Origin Type="time" Unit="1440" Retention="10">ON</Origin>
      </Log>
   </Customize>


-  ``Customize`` 의 ``Name`` 속성이 ``M2_HOOK1`` 일 때만 활성화된다.
-  M2는 ``/usr/local/ston/svc/{가상호스트}/hook1.js`` 를 로딩한다.
-  Hooking 함수는 비동기 Loopback 통신으로 구성된다. 따라서 ``OriginOptions`` 의 ``ConnectTimeout`` , ``ReceiveTimeout`` 으로 최대 대기시간을 설정한다.
-  Hooking 함수에 대한 접근 로그를 origin.log 형식으로 구성한다.


.. _hooking-client:

클라이언트 요청후킹
====================================

클라이언트가 보낸 HTTP 요청을 M2-Core가 구동하는 Hooking 함수로 위임한다. ::

   {
      "host": "foo.com",
      "clientIp": "181.43.1.33",
      "sessionId": 2,
      "protocol": "http",
      "message": "POST /itemimage/LO/12/37/50/02/80/vdo/LO1237500280_1.mpx3123 HTTP/1.1\r\nHost: foo.com\r\nContent-Type: text/plain\r\nUser-Agent: PostmanRuntime/7.26.8\r\nAccept: */*\r\nPostman-Token: 0bce4527-7d8b-4974-9c2c-742efb8a549c\r\nAccept-Encoding: gzip, deflate, br\r\nConnection: keep-alive\r\nContent-Length: 519\r\nX-Forwarded-For: 181.43.1.33\r\n\r\n",
      "body": "<s:Envelope xmlns:s=\"http://schemas.xmlsoap.org/soap/envelope/\">\r\n  <s:Body>\r\n    <serviceCall xmlns=\"http://webservice.B2BOnline.com\">\r\n      <AvailRQ>\r\n        <AgencyId>JJSEL13157</AgencyId>\r\n        <CarrierCode>7C</CarrierCode>\r\n        <DepApoCode>CJU</DepApoCode>\r\n        <DepApoName></DepApoName>\r\n        <ArrApoCode>PUS</ArrApoCode>\r\n        <ArrApoName></ArrApoName>\r\n        <FlightDate>20171228</FlightDate>\r\n        <PaxCount>1</PaxCount>\r\n      </AvailRQ>\r\n    </serviceCall>\r\n  </s:Body>\r\n</s:Envelope>"
   }


-  ``host`` 요청을 처리하는 가상호스트 이름

-  ``clientIp`` 소켓 리모트 주소

-  ``sessionId`` 세션 고유번호 (access.log의 session-id 필드와 동일)

-  ``protocol`` 프로토콜 ( http, https, http2 )

-  ``message`` 클라이언트 요청의 HTTP 메시지

-  ``body`` 클라이언트 POST 요청의 HTTP Body. POST가 아니라면 해당 필드 없음


개발 호환성을 위해 요청 헤더에 ``Content-Type: application/json`` 를 명시한다.


.. _hooking-error:

에러 처리
====================================

M2-Core가 ``200 OK`` 를 보내지 않는다면 ``510 Not Extended`` 응답과 함께 오류 메시지를 보낸다. ::

   {
      "name": "m2hook1", 
      "code": 403, 
      "body": "<HTML><TITLE>403 Forbidden</TITLE><BODY><H1>Forbidden</H1></BODY></HTML>"
   }

-  ``name`` m2hook1 (고정)

-  ``code`` M2-Core의 응답코드

-  ``body`` M2-Core가 보낸 Body


.. _hooking-continue:

요청 재정의
====================================

Hooking 함수의 응답에 클라이언트 HTTP 요청을 재정의한다. ::

   {
      "sessionId": 2,
      "response": {
         "code": 100,
         "body": "blah blah"
      },
      "cacheKey": "/availity?key={a,b,1}",
      "vhost": "bar.com",
      "originRequest": {
         "method": "POST",
         "url": "/itemimage/LO/12/37/50/02/80/vdo/LO1237500280_1.mpx3123",
         "headers": [
            { "key": "host", "value": "baq.com" },
            { "key": "x-custom-header", "value": "abcdefg" },
            { "key": "x-custom-header2", "value": "baq.com" },
            { "key": "cookie", "value": "NNB=LS3KUV63E5RV6; NRTK=ag#all_gr#1_ma#-2_si#0_en#0_sp#0;" }
         ],
         "body": "<s:Envelope xmlns:s=\"http://schemas.xmlsoap.org/soap/envelope/\">\r\n  <s:Body>\r\n    <serviceCall xmlns=\"http://webservice.B2BOnline.com\">\r\n      <AvailRQ>\r\n        <AgencyId>JJSEL13157</AgencyId>\r\n        <CarrierCode>7C</CarrierCode>\r\n        <DepApoCode>CJU</DepApoCode>\r\n        <DepApoName></DepApoName>\r\n        <ArrApoCode>PUS</ArrApoCode>\r\n        <ArrApoName></ArrApoName>\r\n        <FlightDate>20171228</FlightDate>\r\n        <PaxCount>1</PaxCount>\r\n      </AvailRQ>\r\n    </serviceCall>\r\n  </s:Body>\r\n</s:Envelope>"
      }
   }


-  ``sessionId`` 세션 고유번호 (디버그 용, USERDATA 개념)

-  ``resCode`` 응답코드

   -  ``100`` - Continue (흐름 지속)

   -  그 외에는 트랜잭션을 더 진행하지 않고 ``code`` 와 ``body`` 를 즉시 응답한다. 이는 디버깅에 용이하다.

-  ``cacheKey`` 캐싱엔진에서 사용할 키

-  ``vhost`` 변경될 가상호스트. 이 값이 NULL 또는 빈문자열 이라면 가상호스트를 변경하지 않는다.

-  ``originRequest`` 원본에 요청해야 하는 경우 HTTP 요청 구조체