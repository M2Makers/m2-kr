.. _engine-prditem:

13장. 상품기술서 엔진
******************

상품기술서 엔진은 도큐먼트 엔진에서 파생된, E-Commerce 상품기술서에 특화된 엔진이다. 
이 엔진을 통해 Mixed Contents 이슈 등 셀 수 없이 많은 상품기술서를 즉시 개선할 수 있다.

상품기술서 엔진의 전체 설정구조와 기본 값은 다음과 같다. ::

   # /usr/local/m2/config-production.json

   {
      "m2": {
         "mixed" : {
            "traffics" : {
               "resource" : {
                  "domain" : null
               },
               "fallback": {
                  "enable" : true,
                  "method" : "redirect",
                  "conditions" : ["abort", "4xx", "5xx"]
               }
            },
            "options" : {
               "anchor" : false,
               "schemeless" : false
            },
            "upgradeHttps" : {
               "ip" : {
                  "enable" : true
               },
               "retain" : {
                  "enable" : true,
                  "list" : []
               },
               "black" : {
                  "enable" : true,
                  "list" : []
               },
               "white" : {
                  "enable" : true,
                  "list" : []
               },
               "svldb" : {
                  "url" : "https://svl.m2live.co.kr",
                  "enable" : true,
                  "report": {
                     "enable": true,
                     "schedule": "0 0 * * * *"
                  },
                  "update": {
                     "enable": true,
                     "schedule": "0 0 * * * *"
                  }
               },
               "syntax" : {
                  "enable" : true
               }
            }
         }
      }
   }


.. note::

   가상호스트 또는 엔드포인트마다 설정을 달리하고 싶다면 call chain의 첫 단계로 설정을 재정의한다.



.. toctree::
   :maxdepth: 2 


.. _engine-prditem-mixed-contents-traffic:

상품기술서 트래픽 라우팅
====================================

상품기술서를 웹 페이지에 포함시키는 패턴은 3가지가 존재한다. 

-  웹페이지 Embed
-  독립 도메인 ``추천``
-  통합 도메인

고객 환경에 알맞은 구조를 통해 속도와 안정성을 확보한다. 
아래 그림 중 빨간 점선이 M2가 처리해야 하는 상품기술서 트래픽이다.


웹페이지 Embed
---------------------

웹 페이지를 응답하기 전 서버 사이드에서 상품기술서를 포함(Embed)시킨 후 완성된 웹 페이지를 응답한다. 
브라우저는 상품기술서 서버의 존재를 알 수 없다.

.. figure:: img/prditem04.png
   :align: center

이 구조에서는 M2가 <상품기술서 서버> 를 대신하여 트래픽을 처리한다. 
<Web Server>가 보내는 요청이 <M2>로 유입되도록 “domain, ip, hosts 파일” 등을 변경한다.

.. figure:: img/prditem05.png
   :align: center

<Web Server>는 기존과 동일한 방식으로 웹 페이지에 상품기술서를 삽입한다.



독립 도메인
---------------------

상품기술서를 ``<iframe>`` 또는 AJAX 를 이용해 브라우저에서 로딩한다. 
상품기술서 서버가 완전히 독립된 도메인으로 운영된다.

.. figure:: img/prditem06.png
   :align: center

이 구조에서는 <상품기술서 서버>의 도메인을 M2로 위임하는 것만으로 구성이 가능하다. 
위 그림의 빨간 점선 트래픽이 다음과 같이 변경된다.

.. figure:: img/prditem07.png
   :align: center

상품기술서가 포함하는 ``<iframe>`` 이 내부적으로 다른 ``<iframe>`` 을 포함하여도 트래픽이 M2로 자연스럽게 유입된다.



통합 도메인
---------------------

`독립 도메인`_ 과 같은 방식이나 Top-level 페이지(메인 도메인)와 같은 도메인을 사용한다.

.. figure:: img/prditem08.png
   :align: center

이 구조에는 흔히 <프론트 캐시> 알려진 구조로 도입된다. 
M2의 URL 전처리 기능을 이용해 상품기술서 트래픽을 정확히 분리시켜야 한다. 

도입 전 반드시 다음과 같이 처리할 상품기술서 URL 패턴이 확정되어야 한다.

.. figure:: img/prditem09.png
   :align: center

또는 <Web Server>가 상품기술서 트래픽만을 분리하여 M2로 위임하는 방식도 가능하지만 매우 위험하다. 
왜냐하면 M2는 상품기술서를 찾기 위해 다시 <Web Server>로 요청하게 되어 트래픽 Loop가 발생할 수 있기 때문이다.



독립 도메인 추천 이유
---------------------

`독립 도메인`_ 방식을 속도와 안정성 면에서 추천한다.

-  `웹페이지 Embed`_ 방식과 비교

   - 상품기술서를 웹 페이지에 삽입하는 시간만큼 ``TTFB(Time To First Byte)`` 가 줄어든다.
   - <브라우저>는 전달받은 Top-level 페이지를 먼저 화면에 표기한다. 체감 속도가 빨라진다.
   - 만약의 상품기술서 장애상황에도 핵심정보인 가격, 결제 등이 영향받지 않는다.

-  `통합 도메인`_ 방식과 비교
   - 모든 트래픽이 M2를 거친다. 1-hop의 증가로 인한 속도저하는 미비하나 프론트 캐시용도로 사용하지 않는다면 효용성이 높다고 보기 어렵다.
   - 웹 페이지 장애시 점검 범위가 M2까지로 확대된다.
   - 상품기술서 패턴이 늘어날 때마다 M2에 추가해 주어야 한다.


운영 편의성 측면에서도 <2. 독립 도메인> 방식이 강점을 가진다.

-  상품기술서와 부가 트래픽( ``<iframe>`` , http 이미지 등)을 분리해 정확히 모니터링/관리할 수 있다. 분리되어 있지 않다면 로그를 분석해야 한다.
-  상품기술서 트래픽을 손쉽게 CDN으로 위임할 수 있다.
-  상품기술서 정책이 수정되더라도 다른 백엔드 자원에 영향을 주지 않는다.
-  상품기술서 도메인을 <Web Server>로 위임하여 기존 구조로 쉽게 롤백할 수 있다.



.. _engine-prditem-mixed-traffic:

Mixed Contents 트래픽 상세
====================================


메인 트래픽 ``/m2x/mixed/main``
---------------------

상품기술서 최상단에 있는 트래픽이다. 
상품기술서에 대한 접근이 발생하는 위치에 따라 흐름이 달라진다.
호출 방식에 따라 앞서 언급한 3가지로 구분된다.


-  `웹페이지 Embed`_
-  `독립 도메인`_
-  `통합 도메인`_



리바운드 트래픽 ``/m2x/mixed/rebound``
---------------------
웹 페이지는 구조적으로 다른 문서를 포함(Embed)할 수 있다. 

.. figure:: img/prditem10.png
   :align: center

따라서 메인 트래픽으로 기존 상품기술서를 처리하였어도, 다음처럼 ``<iframe>`` 으로 참조되는 페이지까지는 처리할 수 없다. ::

   <iframe src="http://foo.com/..."></iframe>

예제의 ``http://foo.com`` 에 Mixed-Contents 문제여부는 태그만으로 알 수 없다. 
따라서 클라이언트에게 직접 노출시키는 것은 위험하다. 

M2는 메인 트래픽이 ``https://example.com/product/100`` 에 의해 접근되었다면 다음과 같이 소스를 수정한다. ::

   <iframe src="https://example.com/product/100/m2x/mixed/rebound/http://foo.com/..."></iframe>

Browser에서 상품기술서가 로딩 된 이후에 발생하는 리바운드 트래픽은 다음처럼 진행된다.

.. figure:: img/prditem11.png
   :align: center


.. note::

   리바운드 트래픽을 위한 URL변조 규칙이 Suffix인 이유는 반드시 메인 트래픽과 같은 흐름을 이용해야 하기 때문이다.

      1. Browser에서 ``https://example.com/product/100`` 를 요청한다.
      2. ``/product/100`` 에 대한 요청이 M2로 도달하고 상품기술서가 제공된다. (메인 트래픽)
      3. Browser는 상품기술서 안의 <iframe>의 src를 호출한다.

   
   여기서 발생하는 요청이 M2 로 도달하기 까지 네트워크 중간에 여러 장비들이 존재할 수 있다.

      *  L7
      *  API Gateway
      *  Web Server

   M2가 ``<iframe>`` 주소를 수정하는 시점에 알 수 있는 정보는 ``/product/100`` 요청이 자신에게 온전히 도달했다는 것 뿐이다. 
   만약 이 시점에 ``/_m2_/_iframe_/?http://foo.com/...`` 처럼 임의의 Path를 사용할 경우 리바운드 트래픽에 대한 URL 라우팅이 M2로 유입되리라 보장할 수 없다. 
   따라서 검증된 메인 트래픽 URL인 ``/product/100`` 에 참조 소스를 덧 붙이는 규칙을 사용한다.


리소스 트래픽 ``/m2x/mixed/resource``
---------------------

M2 도입 전 후 트래픽 흐름은 다음과 같이 바뀐다.

.. figure:: img/prditem12.png
   :align: center

*  초록 실선 - 상품기술서 트래픽
*  빨간 점선 - SSL-onLoading 되어야 하는 트래픽 (백엔드로 인바운드 되는 트래픽)
*  빨간 실선 - SSL-onLoading 된 트래픽 (백엔드에서 아웃바운드 되는 트래픽)


리소스 트래픽의 대부분은 이미지이다. 
이미지 서비스는 CDN 서비스를 이용하는 경우가 많아 다음과 같이 별도의 도메인 지정이 가능하다. ::

   # m2.productDesc

   "traffics" : {
      "resource" : {
         "domain" : null
      }
   }


메인, 리바운드 트래픽은 가상호스트 이름을 따른다. 하지만 리소스의 경우 설정에 따라 변경된다. ::

   // 메인, 리바운드 트래픽
   https://example.com/products/100/m2x/mixed/main
   https://example.com/products/100/m2x/mixed/rebound/...

   // { "domain" : null } 이라면 가상호스트 이름과 같다.
   https://example.com/products/100/m2x/mixed/resource/...

   // { "domain" : "cdn.example.com" } 이라면 해당 도메인명을 사용한다.
   https://cdn.example.com/products/100/m2x/mixed/resource/...
   


트래픽 Fallback
---------------------

정상적인 경우 M2는 클라이언트와 외부 서비스를 사이의 중계를 담당한다.

.. figure:: img/prditem13.png
   :align: center

여러 이유로 연계는 실패할 수 있다.

-  일부 네트워크 장애
-  의도적인 IP 차단
-  공격으로 오인
-  403 forbidden

이런 ``Unavailable`` 한 상황을 단순히 실패로 처리하는 것은 문제가 있다. 
왜냐하면 B2B에서 실패를 한 것이지 B2C에서 실패한 것이 아니기 때문이다.

M2는 이런 상황에서 클라이언트가 직접 외부 서비스를 호출할 대안(fallback)을 제공하여 서비스 가용성을 높인다. ::

   # m2.productDesc.mixed

   "traffics" : {
      "fallback": {
         "enable" : true,
         "method" : "redirect",
         "conditions" : ["abort", "4xx", "5xx"]
      }
   }


-  ``fallback`` 원본과 정상적인 통신이 불가능할 경우 동작정책을 설정한다.

   -  ``enable``

      -  ``true (기본)`` fallback 정책을 수행한다.
      -  ``fallback`` 500 internal error로 응답한다.


   -  ``method (기본: redirect)`` fallback 동작방식을 설정한다.
   -  ``conditions`` fallback이 동작할 조건목록을 설정한다.

      -  ``abort`` - HTTP 트랜잭션이 중단된 경우
      -  ``HTTP 응답코드`` - 구체적인 HTTP 응답코드
      -  ``3xx`` , ``4xx`` , ``5xx``- 해당 계열의 응답코드

.. figure:: img/prditem14.png
   :align: center


가장 효율적인 fallback은 ``302 Redirect`` 이다. ::

   https://example.com/.../m2x/mixed/resource/http://foo.com/1.jpg


위 예제에서 foo.com과 정상통신이 불가능하다면 M2는 다음과 같이 응답하여 클라이언트가 직접 통신하도록 한다. ::

   HTTP/1.1 302 Found
   Location: http://foo.com/1.jpg



.. _engine-prditem-mixed-contents:

Mixed Contents 처리
====================================

Mixed Contents 엔진의 목적은 최소한의 ``URL`` 에 대해 SSL Onloading 을 적용하는 것이다.
아래와 같이 6단계의 정책 우선순위로 SSL Onloading 이 결정된다.

.. figure:: img/prditem01.png
   :align: center


*  ``IP`` IP 주소는 인증서를 탑재할 수 없다. SSL Onloading 한다.
*  ``Retain List`` 등록된 도메인은 처리하지 않는다.
*  ``Black List`` 등록된 도메인은 강제로 SSL Onloading 시킨다.
*  ``White List`` 등록된 도메인은 ``https://`` 프로토콜만 명시한다.
*  ``SVL (SSL/TLS Validation List)`` `m2live 서비스 <https://svl.m2live.co.kr>`_ 데이터베이스를 참조한다.
*  ``Syntax`` HTML 문법만으로 판단한다. ``http://`` 프로토콜 Scheme이 명시된 경우에만 SSL Onloading 한다.


상품기술서 처리에 앞서 대상을 지정한다. ::

   # m2.productDesc.mixed

   "options" : {
      "anchor" : false,
      "schemeless" : false
   }


-  ``options`` 태그내에서 특별하게 다루어야 하는 요소들에 대한 동작방식을 설정한다.

   -  ``anchor`` 앵커태그 ``<a href="http://...">`` 에 대한 처리정책을 설정한다.

      -  ``false (기본)`` 수정하지 않는다.

      -  ``true`` Mixed Contents 정책에 따라 https로 업그레이드만 진행하며 proxying 하지 않는다. ::
      
            // AS-IS
            <a href="http://foo.com/index.html">

            // TO-BE
            <a href="https://foo.com/index.html">



   -  ``schemeless`` scheme이 생략된 URL에  대한 동작방식을 설정한다.

      -  ``false (기본)`` 수정하지 않는다.

      -  ``true`` 상품기술서내의 다른 리소스와 동일하게 처리한다. scheme을 명확히 지정한다. ::

            // AS-IS
            <script src="//foo.com/common.js">

            // TO-BE
            <script src="http://foo.com/common.js">



.. _engine-prditem-mixed-contents-ip:

IP
---------------------

상품기술서 내에 IP를 포함하는 ``URL`` 을 처리한다. ::

   <img src="http://182.162.143.217/cob/20FW/20FW_main2_kid780.jpg">
   <img src="http://182.162.143.217:8080/cob/20FW/20FW_main2_kid781.jpg">

위 태그는 다음과 같이 변환된다. ::

   <img src="https://example.com/.../m2x/mixed/resource/http://182.162.143.217/cob/20FW/20FW_main2_kid780.jpg">
   <img src="https://example.com/.../m2x/mixed/resource/"http://182.162.143.217:8080/cob/20FW/20FW_main2_kid781.jpg">">


SSL Onloading 여부를 설정할 수 있다. ::

   # m2.productDesc.mixed

   "upgradeHttps" : {
      "ip" : {
         "enable" : true
      }
   }

-  ``enable``

   -  ``true (기본)`` IP를 SSL Onloading한다.
   
   -  ``false`` IP를 처리하지 않는다.


.. _engine-prditem-mixed-contents-retainlist:

Retain List
---------------------

Retain List(유지목록)에 등록된 도메인에 대해서는 어떠한 처리도 수행하지 않는다. ::

   # m2.productDesc.mixed

   "upgradeHttps" : {
      "retain" : {
         "enable" : true,
         "list" : []
      }
   }


-  ``retain`` 유지목록을 설정한다.

   -  ``enable``
      -  ``true (기본)`` 유지목록을 활성화한다.
      -  ``false`` 유지목록을 비활성화한다.

   -  ``list`` 수정하지 않을 도메인 리스트


HTTPS를 지원하지 않는 foo.com을 예로 들어보자. ::

   <img src="http://foo.com/logo.jpg">


정상적인 경우 위 URL은 다음과 같이 SSL Onloading된다. ::

   <img src="https://example.com/.../m2x/mixed/resource/http://foo.com/logo.jpg">


하지만 유지목록이 다음과 같이 활성화되어 있다면 원본 태그인 ``<img src="http://foo.com/logo.jpg">`` 를 그대로 유지한다. ::

   "upgradeHttps" : {
      "retain" : {
         "enable" : true,
         "list" : ["foo.com"]
      }
   }


.. note::

   유지목록은 백엔드에서 통제되는 도메인에 한정하여, 개발 또는 디버깅 용도로 사용하는 것이 바람직하다.   



Black List
---------------------

``Black List`` 에 등록되어 있다면 무조건 SSL Onloading 시킨다. ::

   "upgradeHttps" : {
      "black" : {
         "enable" : true,
         "list" : []
      }
   }

이 기능은 보다 명확하게 동작한다는 이유에서 ``White List`` 보다 우선한다. ::

   // 원본 태그
   <img src="https://foo.com/logo.jpg">

   // "list" : [] 인 경우라면 수정하지 않는다.
   <img src="https://foo.com/logo.jpg">

   // "list" : ["foo.com"] 인 경우라면 SSL onloading 시킨다.
   <img src="https://example.com/.../m2x/mixed/resource/https://foo.com/logo.jpg">



White List
---------------------

``White List`` 에는 명확히 https를 지원하는 도메인을 등록한다. ::

   "upgradeHttps" : {
      "white" : {
         "enable" : true,
         "list" : []
      }
   }


``Retain List`` 는 수정이 없는 반면 ``White List`` 는 해당 도메인이 ``https`` 를 지원할 것으로 기대한다는 차이점이 있다. ::

   // 원본 태그
   <img src="http://foo.com/logo.jpg">

   // "list" : [] 인 경우라면 SSL onloading 시킨다.
   <img src="https://example.com/.../m2x/mixed/resource/https://foo.com/logo.jpg">

   // "list" : ["foo.com"] 인 경우라면 foo.com이 https를 지원한다고 간주하여 프로토콜만 변경한다.
   <img src="https://foo.com/logo.jpg">
  



SVL-DB
---------------------

.. note::

   개념과 동작상세에 대해서는 `SVL 연동 시나리오`_ 를 참고한다.


SVL-DB를 연동하는 방식에 대해 설정한다. ::

   "upgradeHttps" : {
      "svldb" : {
         "url" : "https://svl.m2live.co.kr",
         "enable" : true,
         "report": {
            "enable": true,
            "schedule": "0 0 * * * *"
         },
         "update": {
            "enable": true,
            "schedule": "0 0 * * * *"
         }
      }
   }


-  ``svldb`` SVL-DB 사용여부를 설정한다.

   -  ``url (기본: "https://svl.m2live.co.kr")`` SVL 서비스와 통신할 URL을 설정한다. M2에서 외부 통신이 불가능할 경우 Proxy주소를 통해 통신이 가능하다.
   
   -  ``enable``

      -  ``true (기본)`` SVL-DB를 사용하여 URL을 처리한다.
      -  ``false`` SVL-DB를 사용하지 않는다.

   -  ``report`` 상품기술서 내 불확실한 도메인 목록을 SVL 서비스로 보고한다.
   -  ``update`` SVL-DB를 업데이트한다.


``report`` , ``update`` 의 하위 설정인 ``schedule`` 은 `cron <https://ko.wikipedia.org/wiki/Cron>`_ 표현을 사용한다. 
1시간마다 도메인 목록을 보고하며 SVL-DB를 업데이트한다. 


다음과 같이 도메인에 대한 http, https 서비스가 불안정한 상태를 예로 들어보자.

.. figure:: img/prditem02.png
   :align: center

엔진은 SVL-DB를 참고하여 다음 태그를 수정한다. ::

   <img src="http://foo.com/1.jpg">
   <img src="https://foo.com/2.jpg">
   <img src="http://bar.com/3.jpg">
   <img src="https://bar.com/4.jpg">


위 태그는 순서대로 다음과 같이 수정된다. ::

   <img src="https://foo.com/1.jpg">   // upgrade
   <img src="https://foo.com/2.jpg">   // do nothing
   <img src="https://example.com/.../m2x/mixed/resource/http://bar.com/3.jpg">  // proxy + downgrade
   <img src="https://example.com/.../m2x/mixed/resource/http://bar.com/4.jpg">  // proxy

         



Syntax
---------------------

URL 형식만 보고 문법적으로 판단한다. ::

   "upgradeHttps" : {
      "syntax" : {
         "enable" : true
      }
   }

-  ``syntax`` 프로토콜만으로 판단한다.

   -  ``enable``
      
      -  ``true (기본)`` http:// 로 시작되는 URL만 SSL Onloading한다.

      -  ``false`` 아무 것도 하지 않는다.


.. note::

   scheme이 생략된 URL(예. src="//foo.com/")은 HTTP 트랜잭션이 진행된 프로토콜을 따른다는 의미이기 때문에 처리하지 않는다.



.. _engine-prditem-image:

이미지 로딩개선
====================================

상품기술서를 구성하는 이미지는 매우 긴 경우가 많다. 

.. figure:: img/prditem15.png
   :align: center

   2만 pixel이 넘는 이미지


특히 모바일 환경이라면 보이지 않는 영역까지 로딩하거나, 너무 큰 이미지를 로딩하는 것은 고객 경험을 헤친다.


M2는 서비스 품질을 개선하기 위해 상품기술서 내 이미지를 분석하여 분할, 최적화가 가능하다. ::

   // 원본
   https://example.com/product/100

   // Mixed Contents 처리
   https://example.com/product/100/m2x/mixed/main

   // Mixed Contents 처리 + 이미지 분할로딩
   https://example.com/product/100/m2x/mixed/main/image/split/400

   // Mixed Contents 처리 + 이미지 분할로딩 + 이미지 최적화
   https://example.com/product/100/m2x/mixed/main/image/split/400/optimize



.. _engine-prditem-svl-details:

SVL 연동 시나리오
====================================

SVL(SSL/TLS Validation List) 의 개념과 시나리오에 대해 설명한다.
이번 주안에 완성예정