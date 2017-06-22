---
layout: post
title: "Android AIDL 을 사용한 Activity 와 Service 통신"
date: 2017-06-22 02:15:40 +0900
categories: android
modified: 2017-06-22
image:
  feature: android-Banner.png
  credit: darkness
  creditlink: https://wall.alphacoders.com/big.php?i=290328
---

안녕하세요. 처음으로 Android 포스트를 쓰게 되었네요. 안드로이드 포스트를 커리큘럼별로 포스팅을 해볼까 라는 생각도 했었지만, 커리큘럼에 대한 내용은 다른 블로그에서도 많이 찾아볼 수 있는 내용이고 또 어렵지 않은 내용이기에 커리큘럼이기 보다는 자주 사용하면서도 쉽게 이해하기 어려운 부분을 정리해보고자 처음 포스트를 AIDL로 삼게 되었습니다.



## AIDL ?

AIDL은 Android Interface Definition Language의 약자로, 인터페이스를 정의한 언어입니다. 본래는 Android IDL은 Corba의 IDL 기능을 그대로 묘사한 것이며 RPC 통신할 때 많이 사용합니다. 우리가 이 포스트에서 다룰 주제도, Activity와 Service 간의 통신으로 프로세스 간의 통신을 구현하기 위한 목적으로 사용합니다.

서로 다른 프로세스, 혹은 네트워크 간의 함수를 호출하기 위해서는 인터페이스가 바뀔 때마다 언어와 구조에 맞는 코드를 생성해주어야 하는데, 실제 코드를 생성하는 부분은 AIDL Tool에 맡기고, 개발자는 IDL만 정의하도록 합니다.

그런데, Android 에서는 RPC가 아닌 IPC (Inter Process Communication, 프로세스 간 통신), 안드로이드에서는 보통 Binder라고 불리우는 부분을 사용하기 위해서 정의합니다. 보통 안드로이드에는 두 가지 서비스가 존재하는데, 그 중 Remote Service는 같은 프로세스가 아니라 다른 프로세스에서 오는 함수의 호출을 처리하는 서비스이고, 그를 연결해주는 것이 Binder죠. 그리고, 그 Binder를 사용하는 코드를 자동으로 생성해주는 것이 AIDL 입니다.



## onStart() 와 onBind()

이번 포스트에서 기본적으로 알아야 될 개념들입니다. Service 에는 onStart() 함수와 onBind() 함수가 있는데, 두 개의 차이는 무엇일까요? 둘 다 서비스를 시작할 수 있고, Lifecycle로만 본다면, Remote Service냐, Local Service냐의 차이인 것으로 알 수 있을 것입니다.

실제 onStart() 함수는 그냥 서비스를 실행시키고, Intent에 전달된 Data만 처리합니다. 그런데, onBind() 함수는 연결이 생성되고, AIDL을 통해서 서로 다른 프로세스에서 IPC를 통해 오는 함수 호출을 처리해야 하는 경우에 사용합니다. 다음 단락에서 간단한 예제를 통해 어떻게 구현되는지를 볼 것입니다.

이 개념이 이해하기 어려우시다면, Microsoft Windows의 메시지 처리 구조와 프로세스 간 IPC 처리 구조의 이해가 필요하므로 이 부분을 먼저 숙지해오신 후 다시 이 포스트를 읽어보기를 권장합니다.



## AIDL 정의 예제 (Serivce)

그럼 간단히 AIDL 을 정의해보도록 하겠습니다. 

``` java
package xyz.neonkid.aidlexample;

import android.app.Service;
import android.content.Intent;
import android.os.Binder;
import android.os.IBinder;

public class aidlService extends Service {

    // Activity에서 해당 서비스를 가져오는 Binder 생성 후 서비스 반환
    public class aidlServiceBinder extends Binder {
        public aidlService getService() { return aidlService.this; }
    }

    // Activity에서 정의해 해당 서비스와 통신할 함수를 추상 함수로 정의
    public interface Icallback {
        void recvMessage(String message);
        void recvToastMessage(String message);
    }

    // Activity랑 통신할 Callback 객체..
    private Icallback mCallback;

    // Callback 객체 등록 함수..
    public void registerCallback(Icallback callback) {
        mCallback = callback;
    }

    private final IBinder mBinder = new aidlServiceBinder();

    @Override
    public IBinder onBind(Intent intent) {
        return mBinder;
    }
}

```

먼저 Service 부터 정의합니다. 백그라운드에서 돌려하는 프로세스가 Activity에게 어떤 메시지를 전달하고자 한다면, Binder를 생성해야 합니다. 연결이 생성되고 난 후, 다른 함수 호출을 처리해야 하기 때문에 서비스 안에 Binder 클래스를 생성하여 현재 서비스를 반환하는 함수를 정의하여 구현해줍니다.

그런 다음 서비스와 Activity가 통신할 약속된 함수를 미리 정의해줍니다. 서비스에서 Activity로 데이터를 전송하여 Activity에서 처리하고자 할 때 이 방법을 사용합니다. 반대의 방법은 Activity에서 Interface를 정의하지 않고, 해당 서비스 객체에서 바로 함수를 불러와서 처리합니다. 그 예제는 Activity 부분에서 보실 수 있습니다.



## AIDL 정의 예제 (Activity)

다음은 Activity 쪽 정의입니다.

```java
package xyz.neonkid.aidlexample;

import android.content.ComponentName;
import android.content.ServiceConnection;
import android.os.IBinder;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;

public class MainActivity extends AppCompatActivity {
    private aidlService mService;

    /*
        Service와 연결할 객체입니다. 단순히 연결만 하는 고리이기 때문에,
        서비스 연결 함수에서 서비스 객체를 생성하고 정의하여야 합니다.
     */
    private ServiceConnection mConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            aidlService.aidlServiceBinder binder = (aidlService.aidlServiceBinder)service;
            mService = binder.getService();
            mService.registerCallback(mCallback);
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            mService = null;
        }
    };

    // Service에서 선언한 ICallback 객체를 생성해 추상으로 정의한 함수를 구현합니다.
    private aidlService.Icallback mCallback = new aidlService.Icallback() {
        @Override
        public void recvMessage(String message) {
            // Todo: Activity에서 처리합니다.
        }

        @Override
        public void recvToastMessage(String message) {
            // Todo: Activity에서 처리합니다.
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mService.onCreate();    // 예제로 쓴 함수입니다.
    }
}

```

서비스에서 구현한대로 객체를 생성하고, 사용하시면 됩니다. 단, 여기서 주의할 점은 Service에서 추가로 서브 쓰레드를 사용하여 Activity 함수를 사용해 처리하고자 하는 경우, UI 변경에 유의 바랍니다. 

안드로이드는 특성상 UI 변경은 메인 쓰레드에서만 변경할 수 있으므로 다른 쓰레드에서 UI 변경은 허용하지 않습니다. AIDL도 마찬가지입니다. 서비스에서 서브 쓰레드를 생성해 AIDL로 정의한 함수를 호출하는 경우, 반드시 메인 쓰레드에서 처리하도록 구현하여야 합니다. 이 부분은 다음 포스트에서 자세히 다루도록 하겠습니다.

