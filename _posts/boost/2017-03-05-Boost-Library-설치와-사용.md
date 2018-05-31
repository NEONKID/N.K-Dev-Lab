---
layout: post
title: "Boost Library 설치와 사용"
date: 2017-03-05 02:15:40 +0900
categories: ["boost"]
tags: [Boost, C++]
---

음 생각해보니, CMake에 관한 포스트를 Tistory에 게시를 해버렸네요.  이 개발자 페이지에 신경을 썼음에도 불구하고, 아무래도 아직 Tistory 블로그는 잊혀지지 않았나 봅니다. (다음에는 개발 포스트를 반드시 여기에 게시하도록 할게요 ㅠㅠ)

최근 저는 C++ 언어에 다시 몰두하기 시작했습니다. 작년까지는 학부 수업에서 두 개 이상 Java 언어로 수업을 받았었(?)... 사실 OpenCV는 C++ 언어로 진행하긴 했습니다만 Java 언어로 수업받았던 두 과목이 프로젝트로 진행되는 과목이어서 어쩌다보니 Java에 몰두하게 되어버렸습니다.

다시 C++ 언어를 시작하려니 너무 어려운게 익숙하지가 않아 블로그에 조금 정리해보려 합니다.



## Boost Library

Boost 라이브러리는 현재 C++ 언어에서도 표준 라이브러리로 채택이 검토될 정도로 굉장히 우수한 라이브러리 중 하나입니다. 실제 게임 서버 개발이나 Data Structure 등에도 많이 사용되고 있으며 Linux, OS X, Windows 할 것없이 어디서든 사용할 수 있어 굉장한 메리트를 지니고 있습니다.

(실제 제 Tistory 블로그에도 Boost 라이브러리에 대한 몇 개의 글이 존재합니다. 4년 전 제가 군 입대하기 전에 잠깐 만지작거리고 놀던 글들인데, 이제와서 다시 시작하려하니 가물가물한 느낌이네요.)

> Atomic, Coroutine, Lockfree, Multiprecision, Odeint. 
> Updated Libraries: Algorithm, Array, Asio, Bimap, Chrono, Container, Context, Geometry, GIL, Graph, Hash, Interprocess, Intrusive, Lexical Cast, Locale, Math, MinMax, Move, Polygon, Random, Range, Ratio, Regex, Smart Pointers, StringAlgo, Thread, Utility, Unordered, Variant, Wave and xpressive.

잠깐 소개를 드리기 위해, 저의 Tistory 블로그에 있던 몇 가지 내용을 가져왔습니다. Boost 라이브러리에는 위와 같은 각종 Data Structure와 Algorithm이 존재합니다. 이들 자료구조들 중에는 여러분들이 알고 계시는 Array(배열)이나 Map이 포함되어 있습니다. 사실 여러분들이 자주 사용하는 C++ STL에 들어있는 자료구조는 대부분 Boost 라이브러리에 있던 것들입니다.



## Install Boost

그럼 이제 Boost 라이브러리를 설치해봅시다. 설치하는 것은 예전이나 지금이나 다를 것이 없기 때문에, 간략하게 설명드리고, 바로 사용하는 방법으로 넘어가겠습니다.

### Linux

> $ sudo apt install libboost-1.60-all-dev	

리눅스에서는 레포지터리에서 설치 바이너리 파일을 제공하기 때문에, 간단한 명령어 한 개로도 설치가 가능합니다.



### Windows

<div markdown="0"><a href="https://sourceforge.net/projects/boost/files/boost/" class="button">Download Boost</a></div>

1. 먼저 Download Boost 버튼을 클릭하여, 라이브러리를 다운로드 받습니다.

2. 압축을 풀게 되면, bootstrap.bat 이라는 이름의 배치 파일을 실행합니다.

3. 명령 프롬포트 창을 열고, 아래의 명령어를 입력합니다.

   > b2 toolset=msvc-14.0 variant=debug,release link=static threading=multi address-model=32 runtime-link=shared

   이 명령어를 입력하실 때 주의하셔야할 부분은 아래와 같습니다.

   - toolset: 여러분이 사용하는 Visual C++ 컴파일러의 버전입니다. (14.0은 2015 버전입니다.)
   - address-model: 이 부분은 여러분이 32비트 프로그램을 만들 것이냐, 64비트 프로그램을 만들 것이냐가 나뉩니다.

   (OpenCV도 그러하지만, Boost도 소스 설치하면 이렇게 복잡한 과정을 지나야 하네요..ㅠㅠ)

   더 정확한 빌드 옵션에 대해서는 아래의 Build Documentation 버튼을 눌러주시면 됩니다.

   <div markdown="0"><a href="http://www.boost.org/build/doc/html/bbv2/overview/invocation.html" class="button">Build Documentation</a></div>

4. 모든 빌드 과정이 오류 없이 성공하면 설치가 완료된 것입니다.


### Visual Studio

Visual Studio에서 제공하는 NuGet 설치 도구를 이용한 사용 방법입니다. 이 방법은 한 프로젝트를  수행할지 여러 프로젝트를 수행할지를 결정할 수 있으며 Visual Studio IDE 도구가 실행 가능한 플랫폼에서는 되게 유용하게 사용될 수 있습니다.

![nuget-1](/media/images/boost/nuget-1.png)

먼저, 프로젝트를 불러와서, 솔루션 탐색기에 프로젝트 오른쪽 버튼을 누르고, Nuget 패키지 관리를 클릭합니다.

![nuget-2](/media/images/boost/nuget-2.png)

Browse 버튼을 클릭하여, NuGet 패키지를 검색합니다.

![nuget-3](/media/images/boost/nuget-3.png)

Boost를 검색하면 맨 상단에 바로, 최신의 Boost 라이브러리가 뜹니다. 선택 후, 오른쪽에 프로젝트 리스트에서 Boost 설치할 프로젝트에 체크 박스를 둔 후, Install 버튼을 클릭하면 설치가 시작됩니다.

![nuget-4](/media/images/boost/nuget-4.png)

설치하는 데 2~3분 정도 소요됩니다. 설치가 끝나면, 바로 해당 프로젝트에서 Boost 라이브러리를 사용할 수 있습니다.




## Using Boost

Boost 라이브러리의 설치가 끝났으니 이제 사용을 한 번 해보겠습니다. 

### CMake

CMake를 사용한 사용 방법은 CLion 등과 같은 크로스 플랫폼 IDE 프로그램, vim 등에서 사용할 수 있습니다.
먼저 CLion을 사용해보록 하죠.

![clion_preview-1](/media/images/boost/clion-1.png)

새 프로젝트를 만들어줍니다.

```cmake
cmake_minimum_required(VERSION 3.6)
project(Jekyll_post)

set(CMAKE_CXX_STANDARD 11)

set(SOURCE_FILES main.cc)
add_executable(Jekyll_post %{SOURCE_FILES})
```

그럼 CMakeLists.txt 파일이 보일 겁니다. [CMake 관련 글][cmake-post]을 참조하시면 좀 더 이해가 쉬울 수 있으니 미리 보는 것을 권장해봅니다. 이 파일에서 여러분이 아래의 내용 3줄만 추가하시면 됩니다.

**혹 단독으로 CMake를 사용하시는 분은 반드시 위의 [CMake 관련 글][cmake-post]을 반드시 보고나신 후 사용하시면 되겠습니다.**

> find_package(Boost 1.63 COMPONENTS system filesystem REQUIRED)
> include_directories($(Boost_INCLUDE_DIRS))
>
> target_link_libraries(Jekyll post ${Boost_LIBRARIES})

- find_package: Boost 모듈을 찾아 포함시킵니다. (버전명도 정확하게 기재해야 합니다.) 
- include_directories: 그리고, Boost 헤더 파일이 담긴 디렉터리를 include 시켜줍니다. 
- target_link_libraries: 최종 바이너리 파일을 만들기 위해 필요한 라이브러리를 명시해줍니다. (gcc -l 인자값)


## 마치며...

Boost를 사용했던 기억을 다시 되살려가며 글을 적어봤습니다. Linux에서는 쉽게 사용했던 것들. 4년 전에는 알지 못했던 CMake의 사용 방법 등 여러모로 배워가는 것들이 더 많아진다는 생각이 듭니다.

특히 Makefile에 대해서는 예전부터 알고 있었던 것이지만, 이를 매번 작성하기가 여러모로 너무 귀찮은 탓에 자동화된 IDE 프로그램에 모든 것을 맡기곤 했지만, 라즈베리파이나 아두이노 등 임베디드 프로그래밍에서는 많은 불편함이 있습니다.

이번 Boost 글 작성을 계기로 C++ 프로그래밍에 한 걸음 다가갈 수 있는 기회가 된 것 같군요.

[cmake-post]: http://blog.neonkid.xyz/112