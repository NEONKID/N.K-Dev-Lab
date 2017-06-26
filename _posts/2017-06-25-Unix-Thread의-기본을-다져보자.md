---
layout: post
title: "Unix Thread의 기본을 다져보자"
date: 2017-06-25 00:15:20 +0900
categories: unix
modified: 2017-06-25
tags: [POSIX thread, pthread]
---

개발 글을 써온지 벌써 4개월이 지났지만, 학교 생활과 성적 관리 등으로 인해 컨텐츠가 많이 부족하여 이리저리 생각해 본 결과, 이 개발 블로그에 제가 알고 있는 모든 개발 지식들을 올리려고 합니다. 물론 용도는 여러 가지가 있겠지만, 주 용도는 사람의 기본적인 특성인 망각 특성 때문이겠죠? ㅎㅎ

오늘은 Unix Programming에 대한 카테고리를 새로 추가하여, pthread부터 시작해보려 합니다. 왜 쓰레드부터 시작하냐고요? 그렇군요. 올해는 유난히 제 프로젝트, 과제 등에서 모두 쓰레드를 사용했기 떄문이라고 이야기 하고 싶네요..

이 글을 읽어보시기 전에, Thread에 대한 기초적인 개념이 숙지된 상황에서 읽기를 권장합니다.



## Typical Thread

Typical Thread는 Unix, System V 의 쓰레드로, POSIX Thread의 모태가 된 쓰레드입니다. 아마 지금은 pthread라고 불리우는 것이 제가 보기엔 이 쓰레드이지 않나 생각합니다. 



## POSIX Thread

쓰레드에는 여러 가지 쓰레드가 있습니다. Windows 운영체제에서 사용하는 WindowThread, Mac OS X 에서 사용하는 NSThread 등 여러 가지가 있는데요. 오늘은 그 중에서 POSIX Thread에 대한 이야기를 해보고자 합니다.

POSIX Thread는 병렬 프로그래밍을 하기 위한 C 의 표준 API 입니다. 실제 Unix 계열 운영체제에서 쓰던 쓰레드 함수이며 이를 표준 쓰레드 API로 삼은 것이죠. 이는 Mac OS X도 마찬가지입니다. Mac OS X 에는 NSThread가 존재하지만, Objective C 언어를 사용해 pthread와 병행해서 사용할 수 있으며 Unix 시스템과 전혀 다른 Windows 에서도 pthread를 사용할 수 있습니다.

자세한 레퍼런스 문서는 아래 버튼을 눌러서 참고하시기 바랍니다.

<div markdown="0"><a href="https://www.joinc.co.kr/w/Site/Thread/Beginning/PthreadApiReference" class="btn btn-info">Pthread API Reference </a></div>



## pthread 기본 사용법

레퍼런스 문서를 충분히 읽어보셨다면, 바로 기본 사용법으로 넘어가도록 하겠습니다. (워낙 레퍼런스가 잘 되어 있는지라...)

```c
#include <cstdio>
#include <pthread.h>

typedef struct User
{
    char *id;
    char *pwd;
} User;

void *thread_func(void *arg)
{
    // Todo: 여기에 쓰레드가 해야할 처리를 코딩합니다.
}

int main(int argc, char **argv)
{
    pthread_t thread;
    User user;

    int thread_check = 0;   // Thread 생성 시, 결과를 반환할 변수입니다.

    // Thread 생성 결과가 양수면 정상, 음수이면, 오류가 발생한 것입니다.
    if((thread_check = pthread_create(&thread, NULL, thread_func, (void*)&user) < 0))
    {
        perror("Thread create error !");
        return -1;
    }

    return 0;
}
```

아마 Java, C# 언어에서 Thread 코딩을 하셨던 분들이라면, 조금 생소할 수도 있습니다. Java의 경우, Looper가 존재하는데, 여기는 Looper가 별도로 존재하지 않습니다. Thread를 계속 실행하기 원한다면, while 문으로 코딩하셔야 합니다.

그런데, 위 방식으로 코딩을 하면, scanf, 심지어 커널 함수인 read() 함수를 호출해도 프로그램이 Blocking 되지 않고, 바로 종료가 되어버립니다. 이유는 쓰레드로 분리 시켰기 때문이죠. 

![pthread-2]({{ site.url }}/images/unix/pthread-2.png)

위 그림을 보면, 실제 메인 프로세스에서 pthread_create() 함수를 사용해, Thread를 생성하면, Main Process와 분리된 형태의 Sub Thread의 모양이 나타나게 됩니다. 그러면 Thread 함수에서 생성된 read() 함수 등의 호출은 사실상 Main Process가 return 0; 을 만나기 때문에 부모 프로세스가 소멸함으로써 Sub Thread 또한 종료되고, read() 함수는 무시되는 것입니다.

```c
#include <cstdio>
#include <pthread.h>

typedef struct User
{
    char *id;
    char *pwd;
} User;

void *thread_func(void *arg)
{
    // Todo: 여기에 쓰레드가 해야할 처리를 코딩합니다.
}

int main(int argc, char **argv)
{
    pthread_t thread;
    User user;

    int thread_check = 0;   // Thread 생성 시, 결과를 반환할 변수입니다.
    int status; // Thread 종료 시, 결과를 반환할 변수입니다.

    // Thread 생성 결과가 양수면 정상, 음수이면, 오류가 발생한 것입니다.
    if((thread_check = pthread_create(&thread, NULL, thread_func, (void*)&user) < 0))
    {
        perror("Thread create error !");
        return -1;
    }

    // Thread 가 종료될 때 까지 기다리다가, 그 반환 값을 status에 저장한다.
    pthread_join(thread, (void**)&status);

    return 0;
}
```

만약, 쓰레드가 종료될 때 까지 Main Thread를 종료하고 싶지 않은 경우, (즉, Thread에서 read() 함수 등을 실행시켰을 떄, 종료하지 않도록 하고 싶은 경우.) pthread_join() 함수를 사용해서 해당 쓰레드가 종료될 때 까지 대기하도록 할 수 있습니다.

다만 이 함수를 사용하게 되면, 비동기 처리가 되지 않습니다. pthread_join() 함수 호출 후, 정지하기 때문에 해당 코드 밑으로 정의된 코드들은 이 쓰레드가 종료될 때 까지 처리되지 않으므로 반드시 필요한 경우에만 사용하시기 바랍니다.

```c
void *thread_func(void *arg)
{
    // 처리해야 할 값을 받아오지 못했다면, 쓰레드 종료.
    if(arg == NULL)
    {
        pthread_exit(0);
    }

    // Todo: 여기에 쓰레드가 해야할 처리를 코딩합니다.
}
```

pthread_exit() 함수는 현재 쓰레드를 종료시키는 함수입니다. 쓰레드 함수 내에서 사용하는 함수이며 return을 사용해도 되지만, 공식적으로는 pthreax_exit() 함수를 사용하는 것을 권장하고 있는 듯합니다. 



```c
#include <cstdio>
#include <pthread.h>
#include <unistd.h>

typedef struct User
{
    char *id;
    char *pwd;
} User;

void *thread_func(void *arg)
{
    // 처리해야 할 값을 받아오지 못했다면, 쓰레드 종료.
    if(arg == NULL)
    {
        pthread_exit(0);
    }

    // Todo: 여기에 쓰레드가 해야할 처리를 코딩합니다.
}

int main(int argc, char **argv)
{
    pthread_t thread;
    User user;

    int thread_check = 0;   // Thread 생성 시, 결과를 반환할 변수입니다.
    int status; // Thread 종료 시, 결과를 반환할 변수입니다.

    // Thread 생성 결과가 양수면 정상, 음수이면, 오류가 발생한 것입니다.
    if((thread_check = pthread_create(&thread, NULL, thread_func, (void*)&user) < 0))
    {
        perror("Thread create error !");
        return -1;
    }
  
  	sleep(1);

    // Thread가 종료되었을 때, 사용했던 자원을 모두 반납한다.
    pthread_detach(thread);

    return 0;
}
```

pthread_detach() 함수는 pthread_join() 과 비슷합니다. pthread_join()은 쓰레드가 종료될 때까지 기다리다가 반환 값을 받고, 쓰레드에서 사용했던 모든 자원을 해제합니다. 하지만 detach() 함수는 비동기 처리를 함과 동시에 쓰레드가 종료되면, 자원을 반납하도록 하는 함수입니다. 차이점은 join() 함수는 쓰레드가 종료될 때 까지 기다렸다가 자원을 해제하지만, pthread_detach() 함수는 비동기 처리를 계속하다가 쓰레드가 종료되면, 쓰레드 자원을 반납해주는 함수입니다.

정확히는 Main Thread와 Sub Thread를 분리시키는 함수인데, 그림을 통해 알아보도록 하겠습니다.

![pthread-2]({{ site.url }}/images/unix/pthread-2.png)

최초에 Main Thread에서 pthread_create() 함수를 이용해 Thread를 생성하면, 위 그림과 같은 관계를 가지게 됩니다. Sub Thread의 부모 프로세스는 Main Thread이고, 그들은 연결 고리를 물고 있어 자식 쓰레드인 Sub Thread의 자원을 Main Thread가 안고 갑니다. 따라서 Sub Thread가 썼던 자원은 Main Thread가 소멸될 때 까지 메모리에 계속 남게 됩니다.

![pthread-1]({{ site.url }}/images/unix/pthread-1.png)

하지만 pthread_detach() 함수를 사용하면, 이 둘은 서로 분리됩니다. 그래서, Sub Thread가 소멸하면, Sub Thread에서 사용했던 메모리의 자원은 모두 해제되고, Main Thread에서는 이를 1도 관여하지 않습니다.



## 마치며...

여기까지 pthread에 대해 간단한 몇 가지 예제만을 다뤄봤습니다. 요즘 Thread를 활용한 프로그래밍을 많이 하다보니, 생각나는게 쓰레드 뿐이라서 막상 글을 쓰게 되어서 자연스럽지 못한 부분이 많을 수 있습니다. 

다음 포스트는 Thread Programming 에 대한 중점적인 부분에 대해 적어보도록 하겠습니다.