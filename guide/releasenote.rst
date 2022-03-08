.. _release:

Appendix A: 릴리스 노트
***********************

v22.x
====================================


.. _release-22-02-0:

v22.02.0 (2022.03.08)
----------------------------

-  상품기술서 엔진 - Mixed-Content 대상 주소 내 줄내림 문자로 인한 미동작 증상 수정
-  :ref:`op-log-backup` 기능 추가



v21.x
====================================


.. _release-21-12-0:

v21.12.0 (2021.12.31)
----------------------------

-  상품기술서 엔진

   - :ref:`engine-prditem-edit-refontrol` 기능 추가
   - SSL 온로딩 시 문자열 공백제거 처리와 HTML Comment Tag 누락버그 수정
   - ``<LINK>`` , ``<SCRIPT>`` 태그만 존재하는 HTML의 SSL 온로딩 시 Tag 누락버그 수정



.. _release-21-11-1:

v21.11.1 (2021.12.03)
----------------------------

-  상품기술서 엔진

   -  Data-URI 처리시 리소스 트래픽 도메인 미적용 증상 수정
   -  Mixed-Contents 처리를 위한 원본 요청시 ``304 Not Modified`` 응답인 경우, ``200 OK`` 으로 전송되는 버그 수정
   -  CSS style 내 ``background-image`` 의 SSL 온로딩 시 간헐적 미처리 증상 개선
   -  CSS 파일의 ``@import`` , ``font`` 등의 url 에 대한 SSL 온로드 기능 추가
   -  CSS 파일의 SSL 온로딩시 URL의 따옴표가 제거되는 증상 수정



.. _release-21-11-0:

v21.11.0 (2021.11.16)
----------------------------

-   상품기술서 엔진

    - :ref:`engine-prditem-postmain` 기능 추가
    - :ref:`engine-prditem-mixed-contents-targettags` 기능 추가
    - :ref:`engine-prditem-mixed-traffic-rebound` 도메인 위임기능 추가



.. _release-21-10-0:

v21.10.0 (2021.11.04)
----------------------------

-   상품기술서 엔진 - 리바운드 트래픽에서 SCRIPT 태그의 상대경로 미처리 증상 수정
-   상품기술서 엔진 - :ref:`engine-prditem-mixed-contents-targettags` 기능 추가



.. _release-21-09-0:

v21.09.0 (2021.10.05)
----------------------------

-  :ref:`engine-prditem-area` - ``<HTML>`` , ``<BODY>`` 태그 없이 ``<DIV>`` 만 존재하는 문서에서 ``:`` 키워드가 동작하지 않던 증상 수정



.. _release-21-08-0:

v21.08.0 (2021.08.20)
----------------------------

-  상품기술서 엔진/ :ref:`engine-prditem-mixed-options-anchor` - ``href`` 속성제거 기능 추가
-  Hooking 함수 요청시 간헐적인 Server-Close 문제 수정



.. _release-21-07-2:

v21.07.2 (2021.08.05)
----------------------------

-  Hooking 함수 – 모듈 반영 시점을 파일 변경에서 Reload API 호출 시점으로 변경


.. _release-21-07-1:

v21.07.1 (2021.07.21)
----------------------------

-  상품기술서 엔진 - ``<video>`` , ``<embed>`` 태그 SSL 온로딩 지원
-  Hooking 함수 - :ref:`hooking-smartapi` 기능 지원


.. _release-21-07-0:

21.07.0 (2021.07.13)
----------------------------

-  오토스케일 지원을 위한 설정 동기화 기능 추가


.. _release-21-06-1:

21.06.1 (2021.06.29)
----------------------------

-  상품기술서 엔진 개선
   
   -  PageRender 기능 추가
   -  모든 이미지를 SSL Onloading 처리 하는 기능 추가
   -  ``<iframe>`` 에 포함된 상대경로 이미지가 리소스가 아닌 리바운드 트래픽으로 처리되던 증상 수정



.. _release-21-06-0:


21.06.0 (2021.06.02)
----------------------------

-  설정 Commit/Rollback 기능 추가
-  상품기술서 엔진 :ref:`engine-prditem-edit-delete` 기능 추가



.. _release-21-05-1:


21.05.1 (2021.05.28)
----------------------------

-  :ref:`engine-prditem-mixed-options-sizeLimit` 기능 추가
-  상품기술서 엔진 :ref:`engine-prditem-edit-prereplacesource` 기능 추가

**버그수정**

-  상품기술서 엔진 - ``<input type="image" src="...">`` 미처리 증상 수정


.. _release-21-05-0:


21.05.0 (2021.05.14)
----------------------------

-  `SVL(SSL/TLS Validation List) 서비스 <https://svl.m2live.co.kr/>`_ 리뉴얼
-  :ref:`engine-prditem-mixed-log` 추가
-  Hooking 함수 추가


**버그수정**

-  상품기술서 엔진
   
   -  ``Style Sheet`` 로 삽입 된 이미지가 SSL Onloading 되지 않던 증상 수정
   -  ``Transfer-Encoding`` 으로 전송된 이미지를 분할로딩시 여러 장으로 노출되던 증상 수정
   -  이미지 분할로딩시 ``SVL`` 처리된 호스트의 이미지에 대해 최적화가 적용되지 않던 증상 수정

-  Access로그에 ``referer`` , ``user-agent`` 필드가 ``undefined`` 로 기록되던 증상 수정
-  솔루션 업데이트시 기존 설정이 간헐적으로 초기화 되는 증상 수정


.. _release-21-04-0:

21.04.0 (2021.04.7)
----------------------------

-  puppeteer 기반 오토메이션 기능 추가

**버그수정**

-  설치스크립트 실행시마다 path export가 추가되는 버그 수정


.. _release-21-03-0:

21.03.0 (2021.03.10)
----------------------------

-  `M2 데모사이트 <https://demo.winesoft.co.kr>`_ 리뉴얼
-  :ref:`engine-prditem-mixed-options-sizeLimit` 추가
-  이미지 분할시 :ref:`engine-prditem-mixed-options-schemeless` 에 대해 명시적 scheme 추가



.. _release-21-02-0:

21.02.0 (2021.02.10)
----------------------------

-  트래픽 변조가 가능한 커스터마이징 기능 추가
-  View 다운로드 요청시 헤더 변조 기능 추가



v20.x
====================================

20.11.0 (2020.11.25)
----------------------------

- 프로세스 메모리 사용량 제한 기능 추가
- AWS S3 업로드 기능 추가


20.10.0 (2020.10.28)
----------------------------

- Video Transcoding 피처 추가
- Progressive Video Transcoding 피처 추가
- <iframe> 내 리소스들의 SSL onloading시 간헐적인 Content-Type 미설정 버그 수정


20.09.0 (2020.9.23)
----------------------------

- Videoshot (동영상 내 썸네일 추출) 피처 추가
- 프로세스 모니터링 기능 추가


20.08.0 (2020.8.26)
----------------------------

- :ref:`endpoint-control-error` 기능 추가
- :ref:`getting-started-command-saferestart` 명령어 추가


20.07.0 (2020.7.29)
----------------------------

- m2.log 압축 지원
- 세션 재사용 정책개선


20.06.0 (2020.6.26)
----------------------------

- 장애 URL 디버깅 헤더 추가


20.05.0 (2020.5.27)
----------------------------

- :ref:`endpoint-default` 추가
- :ref:`endpoint-post-method` 에서 :ref:`endpoint-control-module` 지원
- ``m2-render-gif`` 렌더링 성능개선



20.04.0 (2020.4.28)
----------------------------

- 정식 릴리스


20.03.0 (2020.3.25)
----------------------------

- 베타2 릴리스


20.02.0 (2020.2.13)
----------------------------

- 베타1 릴리스


20.01.0 (2020.1.3)
----------------------------

- 알파 릴리스