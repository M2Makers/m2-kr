.. _op:

5장. 운영
******************

이 장에서는 M2운영에 필요한 다양한 도구들에 대해 설명한다.


.. _op-purge:

Caching 무효화
====================================

M2도 STON 캐싱엔진을 사용한다. 따라서 STON과 동일하게 캐싱 무효화가 가능하다. ::

   # wine (model) 변경
   http://{m2-ip}:10040/command/purge?url=example.com/myapi?model=wine&view=*

   # catalog (view) 변경
   http://{m2-ip}:10040/command/purge?url=example.com/myapi?model=*&view=catalog


모델이나 뷰가 변경되면 TTL(Time To Live) 이후 자동반영된다. 즉시 변경을 원할 경우 아래와 같이 Purge API 호출로 즉시 반영이 가능하다.



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

가상호스트에 M2가 설정되어 있다면 실시간/5분 평균 통계를 제공한다. ::

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
