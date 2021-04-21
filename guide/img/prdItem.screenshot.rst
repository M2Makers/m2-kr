

.. _engine-prditem-screenshot:

스크린샷
====================================

스크린샷은 상품기술서 안의 리소스를 이미지로 렌더링하여 제공한다. 

.. note::

   상품기술서 ``<HTML>`` 을 ``JPG`` 로 변환하는 것이 아니다. 
   상품기술서 ``<HTML>`` 은 유지하며 내부 리소스를 ``JPG`` 로 바꾸는 것이다.

이미지로 렌더링된 리소스는 리소스 트래픽으로 제공된다. ::

   https://example.com/products/100/m2x/mixed/screenshot
   
   https://example.com/products/100/m2x/mixed/screenshot/split/400

   https://example.com/products/100/m2x/mixed/screenshot/split/400/optimize


단순히 상품기술서 페이지를 이미지로 렌더링하는 것이 아닌 다음 요소들이 연동된다.

-  YouTube 및 <iframe> 태그 인식
-  Animated GIF 인식
-  동적 Width
-  이미지 분할 및 최적화

::

   # /usr/local/m2/config-production.json

   {
      "m2": {
         "mixed" : {
            "screenshot" : {
               "mode" : "{ optimize | exclude-images | minimum | single }",
            }
         }
      }
   }


-  ``screenshot``

   -  ``optimize (기본)`` 태그를 기준으로영역을 분할하여 이미지로 생성한다.

   -  ``exclude-images`` 이미지는 URL 그대로 유지하며, 텍스트 요소만 이미지로 생성한다.
   
   -  ``minimum`` 동적 리소스(<iframe>, Animated GIF)를 제외하고 이미지로 생성한다.
   
   -  ``single`` 화면 그대로 캡쳐한다.
   

.. note::

   ``JPG`` 포맷의 가로, 세로 최대 길이는 65,535 pixel이다.
   따라서 ``single`` 로 생성하는 경우 스크린샷이 실패할 수 있다.

