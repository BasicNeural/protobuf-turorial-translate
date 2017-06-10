# Protobuf 라이브러리

언어중립적, 플랫폼 중립적, 구조화된 데이터 직렬화로 통신 프로토콜, 데이터 저장소, 등의 확장 가능한 방법인 프로토콜 버퍼의 개발자 문서에 오신 것을 환영합니다.

## 프로토콜 버퍼란 무엇인가?

프로토콜 버퍼는 XML같지만, 더 작고, 빠르고, 간단한 구조화된 데이터 직렬화에 있어서 유연하고 효율적이며, 자동화된 매처니즘이다.
당신이 원하는 구조화된 데이터를 한번 정의하면, 생성된 소스 코드를 사용하여 다양한 데이터 스트림과 다양한 언어로 구성된 데이터를 쉽게 작성하고 읽을 수 있습니다.
심지어 이전 형식으로 컴파일하여 배포된 프로그램이 손상되지 않고 테이터 구조를 업데이트 할 수 있습니다.

## 어떻게 작동되는가?

 `.proto` 파일에서 프로토콜 버퍼 메시지 유형을 정의하여, 직렬화 할 정보를 구조화하는 방법을 지정합니다.
각 프로토콜 버퍼 메시지는 name-value pair를 포함하는 정보의 작은 논리적 레코드 입니다.
여기에 사람의 정보를 가지고 있는 메시지가 정의된 `.proto` 파일의 기본적인 예제가 있습니다:

    message Person {
      required string name = 1;
      required int32 id = 2;
      optional string email = 3;

      enum PhoneType {
        MOBILE = 0;
        HOME = 1;
        WORK = 2;
      }

      message PhoneNumber {
        required string number = 1;
        optional PhoneType type = 2 [default = HOME];
      }

      repeated PhoneNumber phone = 4;
    }

보시다싶이, 메시지 형식은 간단합니다.
각 메시지 유형에는 하나 이상의 고유한 번호가 매겨진 필드가 있습니다. 그리고 각 필드는 이름과 값 타입이 있으며, 여기서 값 타입은 숫자(정수 또는 부동소수점), 논리, 문자열, 바이트 또는 기타 프로토콜 버퍼 메시지 유형일 될 수 있으므로, 데이터를 계층적으로 구조화 할 수 있습니다.
선택적 필드, 필수 필드, 반복 필드를 지정할 수 있습니다.
`.proto.` 파일을 쓰는데 많은 정보를 [프로토콜 버퍼 가이드] 에서 찾을 수 있습니다.

메시지를 정의하고 나면, `.proto` 파일에서 사용하는 언어에 대한 프로토콜 버퍼 컴파일러를 실행하여 데이터 엑세스 클래스를 생성합니다.
이것들은 `name()`과 `set_name()`같은 각각의 필드에 대한 간단한 접근자를 제공하고 바이트에서 전체 구조를 직렬화 / 파싱하는 메서드를 제공합니다.
그래서 예를 들면, C++ 언어를 선택한다면, 위의 예제에서 컴파일하면 Person이라는 클래스가 생성됩니다.
그런 다음 응용 프로그램에서 이 클래스를 사용하여 `Person` 프로토콜 버퍼 메시지를 채우고 직렬화하여 검색 할 수 있습니다.
당신은 다음과 같은 코드를 작성할 수 있습니다:

    Person person;
    person.set_name("John Doe");
    person.set_id(1234);
    person.set_email("jdoe@example.com");
    fstream output("myfile", ios::out | ios::binary);
    person.SerializeToOstream(&output);

그러면, 나중에 메시지를 읽을 수 있습니다:

    fstream input("myfile", ios::in | ios::binary);
    Person person;
    person.ParseFromIstream(&input);
    cout << "Name: " << person.name() << endl;
    cout << "E-mail: " << person.email() << endl;

이전 버전과의 호환성을 유지하면서 새로운 메시지 형식에 새 필드를 추가할 수 있습니다. 이전 바이너리는 새로운 필드를 파싱할 때 무시됩니다.
따라서 프로토콜 버퍼를 데이터 형식으로 사용하는 통신 프로토콜을 사용한다면, 기존 코드를 깨뜨리지 않아도 프로토콜을 확장할 수 있습니다.

생성된 프로토콜 버퍼 코드 사용에 대한 참조는 [API 참조 섹션], 그리고 어떻게 프로토콜 버퍼 메시지가 인코딩 되는지 찾으려면 [프로토콜 버퍼 인코딩] 에서 찾을 수 있습니다.

## 왜 XML을 사용하지 않는가?

프로토콜 버퍼는 구조화된 데이터를 직렬화 하는 데에 XML보다 많은 장점을 가지고 있습니다.
프로토콜 버퍼:

* 간단함
* 3-10배 작음
* 20-100배 빠름
* 덜 모호
* 프로그래밍 하기 쉽게 데이터 엑세스 클래스를 생성

예를 들어, 이름과 이메일을 가진 사람을 모델링 한다고 가정 해 봅시다. XML 에서는 다음과 같이 해야 합니다:

    <person>
      <name>John Doe</name>
      <email>jdoe@example.com</email>
    </person>

해당 프로토콜의 버퍼 메시지(프로토콜 버퍼 텍스트 형식)는:

    # Textual representation of a protocol buffer.
    # This is *not* the binary format used on the wire.
    person {
      name: "John Doe"
      email: "jdoe@example.com"
    }

이 메시지가 프로토콜 버퍼의 바이너리 형식(위의 텍스트 형식은 디버깅과 수정을 위한 사람이 읽을 수 있는 편리한 표현)으로 인코딩 될 때, 아마 28 바이트 길이에 100-200ns 걸릴 것입니다.
XML 버전은 만약 공백을 자우면 최소 69 바이트고 5,000-10,000ns 걸릴 것입니다.
또한, 프로토콜 버퍼는 더 쉽게 조작할 수 있습니다.

    cout << "Name: " << person.name() << endl;
    cout << "E-mail: " << person.email() << endl;

반면 XML은 다음과 같은 작업을 수행 해야합니다:

    cout << "Name: "
         << person.getElementsByTagName("name")->item(0)->innerText()
         << endl;
    cout << "E-mail: "
         << person.getElementsByTagName("email")->item(0)->innerText()
         << endl;

그러나, 프로토콜 버퍼는 언제나 XML보다 좋은 방법이 아닐 수도 있습니다.
예를들어, 프로토콜 버퍼는 텍스트 기반의 마크업 문서(e.g. HTML)에 좋은 방법이 아닐 수도 있고, 구조체를 쉽게 끼어 넣을 수 없습니다.
게다가 XML은 사람이 읽고 쓰기 좋고, 네이티브 형식의 프로토콜 버퍼는 그렇지 않습니다.
XML은 또한 어느정도 자기설명적입니다.
프로토콜 버퍼는 메시지를 정의할 때(`.proto` 파일)만 의미가 있습니다.

## 저를 위한 솔루션 같아요! 어떻게 시작하죠?

[패키지 다운로드] - 이것은 자바, 파이썬, C++ 프로토콜 버퍼 컴파일러 뿐만 아니라, I/O 및 테스트에 필요한 클래스를 위한 소스 코드를 포함합니다.
컴파일러를 빌드 및 설치할 것이면, README 파일의 지시를 따르세요.

일단 모든 설정을 하고 나면, 원하는 언어로 된 [튜토리얼]을 따르세요.
이것은 프로토콜 버퍼를 사용하는 간단한 응용 프로그램을 만드는 과정을 안내합니다.

## proto3 소개

최신 버전 3 릴리즈에는 새로운 언어가 추가되었습니다.
프로토콜 버퍼 버전 3 (일명 proto3)뿐만이 아니라 기존 버전 (일명 proto2)에 몇 가지 새로운 기능이 있습니다.
Proto3는 사용하기 쉽고 광범위한 프로그래밍 언어에서 사용할 수 있도록 프로토콜 버퍼 언어를 단순화합니다:
현제 릴리즈 에서는 Java, C++, Python, Java Lite, Ruby, JavaScript, Objective-C, C# 코드로 생성할 수 있습니다.
게다가 golang/protobuf Github 저장소에서 사용할 수 있는 Go porto3 플러그인을 사용하여 Go용 proto3 코드를 생성 할 수 있습니다.
더 많은 언어가 파이프 라인에 있습니다.

Note that the two language version APIs are not completely compatible. To avoid inconvenience to existing users, we will continue to support the previous language version in new protocol buffers releases.

You can see the major differences from the current default version in the release notes and learn about proto3 syntax in the Proto3 Language Guide. Full documentation for proto3 is coming soon!

(If the names proto2 and proto3 seem a little confusing, it's because when we originally open-sourced protocol buffers it was actually Google's second version of the language – also known as proto2. This is also why our open source version number started from v2.0.0).

## 약간의 역사

Protocol buffers were initially developed at Google to deal with an index server request/response protocol. Prior to protocol buffers, there was a format for requests and responses that used hand marshalling/unmarshalling of requests and responses, and that supported a number of versions of the protocol. This resulted in some very ugly code, like:

    if (version == 3) {
      ...
    } else if (version > 4) {
      if (version == 5) {
        ...
      }
      ...
    }
Explicitly formatted protocols also complicated the rollout of new protocol versions, because developers had to make sure that all servers between the originator of the request and the actual server handling the request understood the new protocol before they could flip a switch to start using the new protocol.

Protocol buffers were designed to solve many of these problems:

New fields could be easily introduced, and intermediate servers that didn't need to inspect the data could simply parse it and pass through the data without needing to know about all the fields.
Formats were more self-describing, and could be dealt with from a variety of languages (C++, Java, etc.)
However, users still needed to hand-write their own parsing code.

As the system evolved, it acquired a number of other features and uses:

* Automatically-generated serialization and deserialization code avoided the need for hand parsing.
* In addition to being used for short-lived RPC (Remote Procedure Call) requests, people started to use protocol buffers as a handy self-describing format for storing data persistently (for example, in Bigtable).
* Server RPC interfaces started to be declared as part of protocol files, with the protocol compiler generating stub classes that users could override with actual implementations of the server's interface.

Protocol buffers are now Google's lingua franca for data – at time of writing, there are 48,162 different message types defined in the Google code tree across 12,183 .proto files. They're used both in RPC systems and for persistent storage of data in a variety of storage systems.

[프로토콜 버퍼 가이드]: http://localhost

[API 참조 섹션]: https://developers.google.com/protocol-buffers/docs/reference/overview

[프로토콜 버퍼 인코딩]: https://developers.google.com/protocol-buffers/docs/encoding

[패키지 다운로드]: https://developers.google.com/protocol-buffers/docs/downloads.html

[튜토리얼]: https://developers.google.com/protocol-buffers/docs/tutorials