---
layout: post
comments: true
title:  "iOS URLSession 이해하기"
excerpt: "iOS에서 네트워킹을 도와주는 URLSession API에 대해 알아봅니다."
categories: iOS URLSession Networking
date:   2017-07-29 00:30:00
tags: [iOS, URLSession, Networking]
image:
  feature: iOS10.png
---

iOS 앱에서 서버와 통신하기 위해 애플은 `URLSession`이라는 API를 제공하고 있습니다. `URLSession`은 iOS 앱 통신에서 유명한 라이브러리 중 하나인 `Alamofire`, `SDWebImage` 등의 기반이 되는 API로 서버와의 데이터 교류를 위해서는 필수적으로 알아야 하는 API입니다. `URLSession`은 HTTP를 포함한 몇 가지 프로토콜을 지원하고, 인증, 쿠키 관리, 캐시 관리 등을 지원합니다.

## URLSession의 Request와 Response

`URLSession`은 다른 웹 통신과 마찬가지로 `Request`와 `Response`를 기본 구조로 가지고 있습니다. 먼저 `Request`는 `URL` 객체를 통해 직접 통신하는 형태와, `URLRequest` 객체를 만들어서 옵션을 설정하여 통신하는 형태가 있습니다. 다음으로 `Response`는 설정된 `Task`의 `Completion Handler` 형태로 `response`를 받거나, `URLSessionDelgate`를 통해 지정된 메소드를 호출하는 형태로 `response`를 받는 형태가 있습니다.

일반적으로 간단한 response를 작성할 때에는 `Completion Handler`를 사용하지만, 앱이 background 상태로 들어갈 때에도 파일 다운로드를 지원하도록 설정하거나, 인증과 캐싱을 default 옵션으로 사용하지 않는 상황과 같은 경우에는 `Delegate` 패턴을 사용해야 합니다.

## URLSession의 기본 컨셉

`URLSession`은 기본적으로 다음과 같은 순서(Life Cycle)로 진행됩니다.

1. Session configuration을 결정한다.
2. Session을 생성한다.
2. 통신할 URL과 Request 객체를 설정한다.
3. 사용할 Task를 결정하고, 그에 맞는 Completion Handler나 Delegate 메소드들을 작성한다.
4. 해당 Task를 실행한다.
5. Task 완료 후 Completion Handler가 실행된다.

#### Session

`URLSession`은 크게 3가지 종류의 Session을 지원합니다.

1. Default Session: 기본적인 Session으로 디스크 기반 캐싱을 지원합니다.
2. Ephemeral Session: 어떠한 데이터도 저장하지 않는 형태의 세션입니다.
3. Background Session: 앱이 종료된 이후에도 통신이 이뤄지는 것을 지원하는 세션입니다.

#### Request

`URLRequest`를 통해서는 서버로 요청을 보낼 때 어떻게 데이터를 캐싱할 것인지, 어떤 HTTP 메소드를 사용할 것인지(Get, Post 등), 어떤 내용을 전송할 것인지 등을 설정할 수 있습니다.

#### Task

Task 객체는 일반적으로 Session 객체가 서버로 요청을 보낸 후, 응답을 받을 때 URL 기반의 내용들을 받는(retrieve) 역할을 합니다. 3가지 종류의 Task가 지원됩니다.

1. Data Task - Data 객체를 통해 데이터 주고받는 Task입니다.
2. Download Task - data를 파일의 형태로 전환 후 다운 받는 Task입니다. 백그라운드 다운로드 지원
3. Upload Task - data를 파일의 형태로 전환 후 업로드하는 Task입니다.


## URLSession 사용하기

URLSession과 같은 네트워킹용 API는 일반적으로 앱 전역에서 사용됩니다. 그렇기 때문에 ViewController에 메소드를 작성하기보다는 하나의 모듈(class)을 만들고 그 안에 static 함수들을 만들어 사용하는 것이 좋습니다.

{% highlight swift %}
class NetworkHandler {
  class func getData(resource: String) {
    // 세션 생성, 환경설정
      let defaultSession = URLSession(configuration: .default)

      guard let url = URL(string: "\(resourceURL)") else {
            print("URL is nil")
            return
      }

      // Request
      let request = URLRequest(url: url)

      // dataTask
      let dataTask = defaultSession.dataTask(with: request) { data, response, error in
        // getting Data Error
          guard error == nil else {
              print("Error occur: \(String(describing: error))")
              return
          }

          if let data = data, let response = response as? HTTPURLResponse, response.statusCode == 200 {
              // 통신에 성공한 경우 data에 Data 객체가 전달됩니다.

              // 받아오는 데이터가 json 형태일 경우,
              // json을 serialize하여 json 데이터를 swift 데이터 타입으로 변환
              // json serialize란 json 데이터를 String 형태로 변환하여 Swift에서 사용할 수 있도록 하는 것을 말합니다.
              guard let jsonToArray = try? JSONSerialization.jsonObject(with: data, options: []) else {
                    print("json to Any Error")
                    return
              }

              // 원하는 작업
          }

      }

      dataTask.resume()
  }
}

NetworkHandler.getData(resource: "http://www.example.com")
{% endhighlight %}