---
layout: post
title: "자바스크립트 프로젝트북 #5"
tags: [javascript, jQuery]
---

2018.02.04  
자바스크립트 프로젝트북 스터디(#5) 내용을 정리한 글입니다.

  
  
  
### jQuery.getJSON 분석
> [jQuery JavaScript Library v1.12.3](https://code.jquery.com/jquery-1.12.3.js)

**jQuery.getJSON**
```js
getJSON: function( url, data, callback ) {
  return jQuery.get( url, data, callback, "json" );
},
```

**jQuery.get**
```js
jQuery.each( [ "get", "post" ], function( i, method ) {
  jQuery[ method ] = function( url, data, callback, type ) {
     // shift arguments if data argument was omitted
      if ( jQuery.isFunction( data ) ) {
      type = type || callback;
      callback = data;
      data = undefined;
    }
  
    // The url can be an options object (which then must have .url)
    return jQuery.ajax( jQuery.extend( {
      url: url,
      type: method,
      dataType: type,
      data: data,
      success: callback
    }, jQuery.isPlainObject( url ) && url ) );
  };
});
```

**jQuery.ajax**
```js
ajax: function( url, options ) {
  
  // If url is an object, simulate pre-1.5 signature
  if ( typeof url === "object" ) {
    options = url;
    url = undefined;
  }
  
  // Force options to be an object
  options = options || {};
  
  ...
  // Create the final options object
  s = jQuery.ajaxSetup( {}, options ),
  
  ...
}
```

**jQuery.ajaxSetup**
```js
ajaxSetup: function( target, settings ) {
  return settings ?
    // Building a settings object
    ajaxExtend( ajaxExtend( target, jQuery.ajaxSettings ), settings ) :
    
    // Extending ajaxSettings
    ajaxExtend( jQuery.ajaxSettings, target );
},
```

**jQuery.ajaxSettings**
```js
// Create the request object
// (This is still attached to ajaxSettings for backward compatibility)
jQuery.ajaxSettings.xhr = window.ActiveXObject !== undefined ?
	// Support: IE6-IE8
  function() {
    // XHR cannot access local files, always use ActiveX for that case
    if ( this.isLocal ) {
    	return createActiveXHR();
    }
        // Support: IE 9-11
    // IE seems to error on cross-domain PATCH requests when ActiveX XHR
    // is used. In IE 9+ always use the native XHR.
    // Note: this condition won't catch Edge as it doesn't define
    // document.documentMode but it also doesn't support ActiveX so it won't
    // reach this code.
    if ( document.documentMode > 8 ) {
    	return createStandardXHR();
    }
        // Support: IE<9
    // oldIE XHR does not support non-RFC2616 methods (#13240)
    // See http://msdn.microsoft.com/en-us/library/ie/ms536648(v=vs.85).aspx
    // and http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html#sec9
    // Although this check for six methods instead of eight
    // since IE also does not support "trace" and "connect"
    return /^(get|post|head|put|delete|options)$/i.test( this.type ) &&
    	createStandardXHR() || createActiveXHR(); 
  } :
	// For all other browsers, use the standard XMLHttpRequest object
  createStandardXHR;
```

jQuery.getJSON function은 browser에 따라서 XMLHttpRequest object를 사용한다는것을 알 수 있다.

### JSON(JavaScript Object Notation)
비동기 브라우저/서버 통신 (AJAX)을 위해, 넓게는 XML(AJAX가 사용)을 대체하는 주요 데이터 포맷.  
본래는 자바스크립트 언어로부터 파생되어 자바스크립트의 구문 형식을 따르지만 언어 독립형 데이터 포맷.
> [Wiki JSON](https://ko.wikipedia.org/wiki/JSON)

### AJAX(Asynchronous JavaScript and XML)
비동기적인 웹 애플리케이션의 제작을 위해 아래와 같은 조합을 이용하는 웹 개발 기법.  
Ajax 애플리케이션은 필요한 데이터만을 웹서버에 요청해서 받은 후 클라이언트에서 데이터에 대한 처리를 할 수 있다.  
> [Wiki AJAX](https://ko.wikipedia.org/wiki/Ajax)

### XMLHttpRequest Object, over HTTP (file and FTP)
XMLHttpRequest는 Microsoft가 만든 JavaScript 개체(object).  
XMLHttpRequest는 HTTP를 통해서 쉽게 데이터를 받을 수 있게 해준다.  
> [MDN XMLHttpRequest](https://developer.mozilla.org/ko/docs/XMLHttpRequest)

### HTTP(HyperText Transfer Protocol)
WWW 상에서 정보를 주고받을 수 있는 프로토콜.  
주로 HTML 문서를 주고받는 데에 쓰인다. TCP와 UDP를 사용하며, 80번 포트를 사용한다.  
> [Wiki HTTP](https://ko.wikipedia.org/wiki/HTTP)

### HTTP Message
HTTP 메시지는 서버와 클라이언트 간에 데이터가 교환되는 방식.  
메시지에는 두 가지 타입이 존재함. request는 클라이언트에 의해 전달되어 서버의 동작을 일으키는 것이고, response는 그에 대한 서버의 회신  
> [MDN HTTP Messages](https://developer.mozilla.org/ko/docs/Web/HTTP/Messages)

### TCP/IP Layer
![TCP/IP Layer](/assets/img/tcp_layer.png)

### Chromium XMLHttpRequest implement
> [Chromium XMLHttpRequest](https://chromium.googlesource.com/chromium/blink.git/+/99b8c9800ac123eddc3e199088d22569c5294b22/Source/core/xml/XMLHttpRequest.h)

> [Chromium SocketStreamHandleClient](https://chromium.googlesource.com/chromium/blink.git/+/99b8c9800ac123eddc3e199088d22569c5294b22/Source/core/platform/network/SocketStreamHandleClient.h)
    
### SPA(Single Page Application) & SSR (Server Side Rendering)
SPA는 기존의 전통적인 새로고침 방식의 웹과는 달리 필요한 정적파일을 한번에(나눠서도 가능하다) 모두 다운로드 받고, 이후 사용자와의 상호작용 가운데 필요한 데이터만 서버로부터 (비동기로) 동적으로 받게하여 트래픽의 총량을 줄이고, 네이티브 앱과 유사한 사용자 경험을 제공할 수 있는 어플리케이션 형태.

SSR은 페이지 로딩시마다 서버로부터 리소스를 전달받아 해석하고 화면에 렌더링하는 방식.
> [SPA 실습](http://poiemaweb.com/js-spa)
> [SPA(Sigle Page Applications) 란 무엇인가](http://devstory.ibksplatform.com/2017/08/spasigle-page-applications.html)

### SEO (search engine optimization)
검색 엔진 최적화는 웹 페이지 검색엔진이 자료를 수집하고 순위를 매기는 방식에 맞게 웹 페이지를 구성해서 검색 결과의 상위에 나올 수 있도록 하는 작업을 말한다.
> [How Search Works(YouTube)](https://youtu.be/BNHR6IQJGZs)
  
---
- [RESTFul이란 무엇인가?](http://blog.remotty.com/blog/2014/01/28/lets-study-rest/)
- [REST API](http://poiemaweb.com/js-rest-api)
- [SPA & Routing](http://poiemaweb.com/js-spa)
- [CSR vs SSR](https://jongmin92.github.io/2017/06/06/JavaScript/client-side-rendering-vs-server-side-rendering/)
- [SPA 단점에 대한 단상](http://m.mkexdev.net/374)
- [웹의 Native 化, PWA(Progressive Web Apps)](http://m.mkexdev.net/356)
- [서버 사이드 렌더링 그리고 클라이언트 사이드 렌더링](http://asfirstalways.tistory.com/244)
  
  