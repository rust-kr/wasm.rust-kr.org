# Rust로 작성한 WebAssembly 코드를 프로덕션 환경에 배포하기

> **⚡ 사실 Rust, WebAssembly로 웹 앱을 작성했더라도 배포 과정이 다른 웹 앱을 배포하는 과정과 거의 동일합니다!**

Rust로 생성한 WebAssembly를 실행하는 웹 앱을 배포해보겠습니다. 빌드된 웹 앱을 프로덕션 서버의 파일 시스템으로 복사하고, 복사한 파일에 접근할수 있도록 HTTP 서버를 설정해주세요.

## HTTP 서버가 `application/wasm` MIME 타입을 지원하는지 확인해주세요

페이지가 빠르게 로드될수 있도록, 네트워크 전송을 통해 [`WebAssembly.instantiateStreaming` 함수][instantiateStreaming]를 사용하여 wasm 컴파일과 인스턴스화 과정을 파이프라인 처리해야 합니다. (번들러가 이 함수를 사용할수 있는지도 확인해주세요.) 하지만 `instantiateStreaming`를 실행할 때 HTTP 응답이 `application/wasm` [MIME 타입][MIME type] 유형을 가지고 있지 않다면 오류가 발생하게 되니 이 부분도 잘 확인해주세요.

* [Apache HTTP 서버에서 MIME 타입 설정하기][apache-mime]
* [NGINX HTTP 서버에서 MIME 타입 설정하기][nginx-mime]

[instantiateStreaming]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WebAssembly/instantiateStreaming
[MIME type]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types
[apache-mime]: https://httpd.apache.org/docs/2.4/mod/mod_mime.html#addtype
[nginx-mime]: https://nginx.org/en/docs/http/ngx_http_core_module.html#types

## 추가 자료

* [프로덕션 개발 환경에서 Webpack를 사용하는 모범 사례.][webpack-prod] 많은 Rust와 WebAssembly 프로젝트들은 Rust로 생성한 WebAssembly와 JavaScript, CSS, HTML 코드를 번들링 하기 위해 Webpack을 사용합니다. 이 가이드는 프로덕션 환경에 배포할 때 어떻게 Webpack을 가장 잘 활용할수 있는지 알려주는 여러 유용한 정보들를 포함합니다.
* [Apache 문서.][apache] Apache는 프로덕션 환경에서 많이 사용되는 HTTP 서버입니다.
* [NGINX 문서.][nginx] NGNIX는 프로덕션 환경에서 많이 사용되는 HTTP 서버입니다.

[webpack-prod]: https://webpack.js.org/guides/production/
[apache]: https://httpd.apache.org/docs/
[nginx]: https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/
