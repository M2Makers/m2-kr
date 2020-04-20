.. _op:

7장. 운영
******************

이 장에서는 운영에 필요한 다양한 도구들에 대해 설명한다.


.. _op-purge:

Caching 무효화
====================================

M2도 STON 캐싱엔진을 사용한다. 따라서 STON과 동일하게 캐싱 무효화가 가능하다. ::

   # wine (model) 변경
   http://{m2-ip}:10040/command/purge?url=example.com/myapi?model=wine&view=*

   # catalog (view) 변경
   http://{m2-ip}:10040/command/purge?url=example.com/myapi?model=*&view=catalog


모델이나 뷰가 변경되어도 TTL(Time To Live) 이후 반영된다. 
즉시 변경을 원할 경우 `Purge <https://ston.readthedocs.io/ko/latest/admin/caching_purge.html#purge>`_ 를 호출한다.



.. _op-log:

로그
====================================

STON의 로그에 더해 각 엔드포인트의 호출 기록은 m2.log에 기록된다. 
STON 로그 설정방식과 동일하며 가상호스트별로 설정한다. ::

   # server.xml - <Server><VHostDefault><Log>
   # vhosts.xml - <Vhosts><Vhost><Log>

   <M2 Type="time" Unit="1440" Retention="10">OFF</M2>

기본 값은 ``OFF`` 이다. ::

   #Fields: date time ...

모든 필드는 공백으로 구분되며 각 필드의 의미는 다음과 같다.

-  ``date`` M2 트랜잭션 완료된 날짜
-  ``time`` M2 트랜잭션이 완료된 시간
-  ``cs-sid`` M2 트랜잭션을 요청한 클라이언트 세션 ID
-  ``cs-uri`` 클라이언트가 요청한 URI
-  ``x-result`` M2 트랜잭션 결과 코드. ``200`` = 성공
-  ``x-result-size`` M2 출력물 크기
-  ``x-error`` M2 에러 상세내용
-  ``x-resource-count`` 참조한 리소스 개수
-  ``x-resource-size (단위: Bytes)`` 참조한 리소스 크기의 합
-  ``x-render`` 렌더링 설정변수
-  ``x-transaction-status`` M2 트랜잭션 정상처리 여부. (c=완료, x=중지)
-  ``time-firstbyte (단위: ms)`` 요청 후 첫 응답이 올때까지의 소요시간
-  ``time-taken (단위: ms)`` M2 트랜잭션이 완료될 때까지의 전체 소요시간
-  ``time-taken-rendering (단위: ms)`` View 렌더링 소요시간


상세 에러코드는 다음과 같다.

-  ``200`` - 성공
-  ``400`` - 파라미터가 불충분
-  ``404`` - Endpoint 없음
-  ``500`` - 초기화 실패 (문법, 결합모델의 파라미터 오류 등)
-  ``501`` - 모델, 뷰 참조 실패
-  ``503`` - 렌더링 실패


.. _op-monitoring:

통계
====================================

`STON 가상호스트 통계 <https://ston.readthedocs.io/ko/latest/admin/monitoring_stats.html#id4>`_ 의 하위에 위치한다. ::

   "M2":                                     <M2
   {                                           Requests="30"
     "Requests": 30,                           Converted="29"
     "Converted": 29,                          Failed="1"
     "Failed": 1,                              AvgRscsSize="1457969"
     "AvgRscsSize": 1457969,                   AvgRscsCount="67"
     "AvgRscsCount": 67,                       AvgRenderTime="124" />
     "AvgRenderTime": 124,                     AvgTime="34" />
     "AvgTime": 34
   },

-  ``Requests`` M2로 요청된 횟수
-  ``Converted`` M2로 생성된 응답 (성공)
-  ``Failed`` M2에서 실패한 응답
-  ``AvgRscsSize (단위: Bytes)`` 엔드포인트에서 참조한 평균 리소스 크기
-  ``AvgRscsCount`` 엔드포인트에서 참조한 평균 리소스 카운트
-  ``AvgRenderTime (단위: ms)`` 렌더링 소요시간
-  ``AvgTime (단위: ms)`` 요청 처리시간



.. _op-vhost-volatile:

휘발성 엔드포인트
====================================

엔드포인트는 이미지/동영상에 비해 짧은 TTL(Time To Live)를 가진다. 
짧은 TTL 콘텐츠는 디스크에 캐싱해도 재사용성이 떨어져 비효율적이다. 
따라서 엔드포인트를 다루는 가상호스트는 ``Volatile (기본: OFF)`` 속성을 활성화시킨다. ::
   
   # vhosts.xml

   <Vhosts>
      <Vhost Name="www.example.com" Volatile="ON">
        ... (생략) ...
      </Vhost>

      <Vhost Name="image.example.com">
        ... (생략) ...
      </Vhost>
   </Vhosts>


`캐시 Storage <https://ston.readthedocs.io/ko/latest/admin/environment.html#storage>`_ 가 구성되어 있더라도 ``Volatile`` 가상호스트는 메모리만 사용한다.



.. _op-vhost-multi:

가상호스트 분리
====================================

엔드포인트가 이미지/동영상을 처리해야 한다면 각각 독립된 가상호스트로 구성하는 것을 권장한다. 
캐싱정책, 통계, 로그등을 분리시켜 다룰 수 있어 높은 유연성을 가지기 때문이다. ::

   # vhosts.xml

   <Vhosts>
      <Vhost Name="www.example.com" Volatile="ON">
         ... (생략) ...
         <M2>
            <Endpoints>
               ... (생략) ...
            </Endpoints>
        </M2>
      </Vhost>

      <Vhost Name="image.example.com">
        ... (생략) ...
        <Options>
           <Dims Status="Active" Keyword="dims">
              ... (생략) ...
           </Dims>
        <Options>
      </Vhost>

      <Vhost Name="video.example.com">
        ... (생략) ...
        <Media>
           ... (생략) ...
        </Media>
      </Vhost>
   </Vhosts>


만약 멀티인증서를 사용할 수 없고, 1개의 도메인으로만 통합하여 서비스해야 한다면, `URL전처리 <https://ston.readthedocs.io/ko/latest/admin/adv_vhost.html#url>`_ 를 추가한다. ::

   # vhosts.xml

   <Vhosts>
      ... (생략) ...

      <URLRewrite AccessLog="Replace">
         <Pattern><![CDATA[^www.example.com/m2/([^/]+)/(.*)]]></Pattern>
         <Replace><![CDATA[#1.example.com/#2]]></Replace>
      </URLRewrite>
   </Vhosts>


``/m2/.../{{ 원본-url }}`` 패턴을 이용해 손쉽게 구성이 가능하다.