---
layout: post
title: "Android 에서 MQTT를 사용하는 방법"
date: 2017-10-01T13:27:19+09:00
categories: ["android"]
tags: [Kotlin, MQTT, android]
---

안녕하세요. 요즘 취업 시즌이 한창이다보니, 블로그에 글쓰는게 또 게을러지게 되었네요. ㅜㅜ 

오늘은 지난 포스트에 이어서, 프로그래밍 코드를 이용한 MQTT 메시지 교환 - 안드로이드 편을 포스트하고자 합니다.

혹시 아직 MQTT에 대한 개념이나, Broker를 설치하지 않으신 분들은 아래 링크를 통해, 이전 글을 반드시 구독해주신 후, 이 포스트를 참조하시기 바랍니다.

<div markdown="0"><a href="http://blog.neonkid.xyz/127" class="button">MQTT 프로토콜 개념과 이해</a></div>

<br />

## Eclipse Paho

MQTT 프로토콜을 사용해 메시지 교환을 하기 위해서는 MQTT 통신이 가능한 라이브러리가 필요합니다. 그 중에서도 저희는 Java와 아주 궁합이 좋은 Eclipse Paho를 사용하고자 합니다.

```
implementation 'org.eclipse.paho:org.eclipse.paho.client.mqttv3:1.1.1'
```

```
compile 'org.eclipse.paho:org.eclipse.paho.client.mqttv3:1.1.1'
```

Gradle 3.0 버전을 사용하신다면, implementation을, Gradle 2.x 버전을 사용하신다면, compile 코드를 build.gradle에 삽입합니다. 

<br />

## Using Paho

자 이제 그럼 Paho를 사용해서 MQTT를 사용해보도록 하겠습니다. MQTT가 기본적으로 하는 일에는 Publish, Subscribe가 있는데요. 그 전에 우리는 MQTT Broker와 연결해줘야 합니다.

```kotlin
val client = MqttClient(ip_addr, clientID, MemoryPersistece())
client.connect()
```

```java
private MqttClient client = new MqttClient(ip_addr, clientID, new MemoryPersistence());
client.connect();
```

MqttClient 라는 객체를 생성하여, 생성자에 브로커의 IP 주소와 해당 브로커에서 사용할 ID 그리고 채팅 내용이 들어갈 여유의 MemoryPersistence 객체를 MqttClient 생성자에게 넘겨줍니다.

마지막으로 connect 메소드를 사용하여 연결을 시도합니다.

위 형태가 기본형입니다. 통신 서비스이기 때문에 연결이 실패할 경우도 있을 수 있으므로, 반드시 예외 처리를 해주도록 합시다. 연결이 끝나면, disconnect 메소드를 이용해 연결을 끊으시면 됩니다.

<br />

### Publish

이제 연결을 하였으니, 메시지를 보내보도록 하겠습니다. 

```kotlin
try {
  client.connect()
  client.publish(topic, MqttMessage("Hello MQTT !".toByteArray()))
} catch (ex: MqttException) {
  ex.printStackTrace()
}
```

```java
try {
  client.connect();
  client.publish(topic, new MqttMessage(new String("Hello MQTT !").getBytes()));
} catch (MqttException ex) {
  ex.printStackTrace();
}
```

Message를 보내는 것도 어렵지 않습니다. 연결을 할 떄, 반드시 위와 같이 오류 처리를 해주시면 됩니다. 다만 publish를 할 때 보낼 메시지는 String이 아닌 ByteArray 형태로 전송되어야 합니다. 이 부분은 Java/Kotlin 언어에서 지원하는 String 객체의 메소드를 사용하여 쉽게 변환할 수 있습니다.

<br />

### Subscribe

메시지를 발행하는 방법을 알았으니 다음은 이제 메시지를 구독하는 방법을 알아보겠습니다.

```kotlin
try {
  client.connect()
  client.subscribeWithResponse(topic, 2, IMqttMessageListener)
} catch (ex: MqttException) {
  ex.printStrackTrace()
}
```

```java
try {
  client.connect();
  client.subscribeWithResponse(topic, 2, new IMqttMessageListener);
} catch (MqttException ex) {
  ex.printStaceTrace();
}
```

메시지를 구독을 할 떄는 단순히 메소드 호출로만 이루어지기 힘들겠죠? 단순 subscribe 메소드도 있긴 하지만, 이는 진짜 메시지를 구독만 할 것일 뿐 이를 프로그램에서 소화하려면 콜백 함수 인터페이스를 사용해야 합니다. 그래서, IMqttMessageListener를 사용합니다.

코틀린 언어에서는 다음과 같은 코드로도 사용하실 수 있습니다.

```kotlin
try {
  client.connect()
  client.subscribeWithResponse(topic, 2, { topic, message -> ... })
} catch (ex: MqttException) {
  ex.printStrackTrace()
}
```

Lambda 방법이 생소하신 분들을 위해 조금 설명을 드리자면, 매우 쉽습니다. No-Lambda에서는 함수 이름을 붙여야 했지만, Lambda 방법을 사용하면 메소드의 인자값을 주고, 화살표만 넣어주신 다음 메소드의 액션을 넣으시면 됩니다. 얼핏보면 Java 8 에서 지원하는 Lambda 사용법이랑 비슷합니다.



```kotlin
override fun messageArrived(topic: String?, message: MqttMessage?) {
  // Todo: 여기에서 메시지를 처리합니다. 메시지는 MqttMessage 객체로 받습니다.
  Log.i(topic + ": ", String(message!!.payload))	// String 값으로 받으려면 이렇게....
}
```

```java
@Override
public void messageArrived(String topic, MqttMessage message) {
  // Todo: 여기에서 메시지를 처리합니다. 메시지는 MqttMessage 객체로 받습니다.
  Log.i(topic + ": ", new String(message.getPayload()));	// Java 에서...
}
```

콜백 함수에 들어있는 인자 값을 이용해 String 생성자를 이용하여 간단히 String 형태로 바꾸어 사용하실 수 있습니다

그리고, 가운데 인자인 2는 QoS를 의미합니다. MQTT의 QoS는 다음과 같습니다.

- QoS = 0 : 수신 측 확인 없이 1회 전송합니다.
- QoS = 1 : 수신 측의 ACK를 확인하고 ACK가 오지 않으면 재전송합니다. (여가서 재전송할 때, 수신측에서는 중복 메시지를 받을 수 없습니다)
- QoS = 2 : 송신/수신 측 모두 ACK 를 확인합니다

QoS는 여러분이 원하시는 번호로 하시면 됩니다. 사실 테스트 전송에는 많은 영향을 받지 않습니다. 실제 QoS 동작을 확인해보시려면 직접 패킷 캡처링을 해보시기 바랍니다.

<br />

## Android / Kotlin Example

그럼 이제 안드로이드 앱으로 만들어보도록 하겠습니다. 중간에 Broker를 놓고, 클라이언트 두개가 서로 메시지를 주고 받는 것에 대해 간단한 예제를 담아봤습니다.

이 예제 프로그램은 안드로이드 앱에서 특정 메시지를 전송하면, 더 이상 그들 둘 사이의 메시지 교환이 이루어지지 않는 프로그램으로 메시지를 통한 인증 프로그램과 유사한 앱을 만들어봤습니다.

![MQTT-1](/media/images/android/MQTT-1.gif)

겉보기에는 MQTT를 이용해 통신하고 있다는 모습처럽 보이지는 않죠? 포스트를 하면서 여러모로 많은 고민을 해봤지만 가장 간단한 예제로 접할 수 있는 것이 이것밖에 생각나지 않았습니다.. ㅠㅠ

![MQTT-2](/media/images/android/MQTT-2.png)

간단히 다이어그램으로 알아보자면, 위와 같습니다. 서버에서도 동일한 MQTT 통신 코드를 가지고 있습니다. 해당 서버에서 메시지를 받았을 때, 올바른 키가 전송되었으면 그에 해당하는 응답을 주고, 그렇지 않으면 실패 코드를 안드로이드 앱에게 부여합니다. 

코드를 통해 살펴보도록 하겠습니다.

<br />

### Server Code

```kotlin
client.subscribeWithResponse("mqttKotlin_send", 2, { topic, message ->
  // Java 의 Switch - case 문..
  when(String(message.payload).trim()) {
    "IMNKID" -> {
      client.publish("mqttKotlin_recv", MqttMessage("Congratulations Key !!!".toByteArray()))
      client.disconnect()
      exitProcess(0)
    }
    else -> {
      println(String(message.payload))
      client.publish("mqttKotlin_recv", MqttMessage("FAILED.....".toByteArray()))
    }
  }
})
```

서버 코드는 간단합니다. 올바른 키를 받았을 떄의 상황을 Switch - case 문을 사용하였고, 서로가 주고 받는 메시지의 Topic을 다르게 주었습니다. 이 부분은 다르게 주는 것을 권장합니다. 메시지가 섞이는 경우, 통계나 관리가 힘들어지게 됩니다.


<br />

### Client Code

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
  try {
      client.subscribeWithResponse(topic_recv, 2, { topic, msg ->
        Log.i("[RECV]", String(msg.payload))
        when(String(msg.payload).contains("Congratulations")) {
          true -> {
            runOnUiThread({
              action_image.setImageDrawable(getDrawable("DRAWABLE_SRC"))
              message.setHint("Congratulations, This is correct key ! ")
              setToastMessage("Wow.............")
              sendBtn.isEnabled = false
            })
            client.disconnect()
          }
          false -> setToastMessage("Cheer up ! Please Try again......")
        }
      })
  } catch (ex: MqttException) {
      setToastMessage(ex.message)
      ex.printStackTrace()
  }
}
```

맨 처음 Activity가 만들어지는 부분에 Broker 연결과 Subscribe 를 추가해줍니다. Subscribe를 Activity가 생성되는 부분에 추가하여 메시지가 구독될 경우, 콜백/쓰레드를 통해 실시간으로 받을 수 있도록 합니다.

(어? 여기서 Thread 따로 생성안했는데? 아, MQTT 라이브러리는 별도로 Thread 객체를 사용안하셔도 됩니다...)

```kotlin
sendBtn.setOnClickListener(this)

override fun onClick(p0: View?) {
    try {
      client.publish(topic_send, MqttMessage(message.text.toString().toByteArray()))
      message.setText("")
    } catch (ex: MqttException) {
      setToastMessage(ex.message)
      ex.printStrackTrace()
    }
}

fun setToastMessage(msg: String?) {
    runOnUiThread({
        Toast.makeText(this, msg, Toast.LENGTH_SHORT).show()
    })
}
```

키를 서버에게 주는 방법 또한 위에서 얘기한 Publish의 간단한 예제를 조금만 응용하면 됩니다. 

<br />

## 마치며....

여기까지 Java, Kotlin 언어를 사용하여 간단한 MQTT 앱을 만들어보는 것과 예제를 다뤄봤습니다.

저도 처음 다뤄보는 Kotlin이라 그런지 많이 헷갈립니다. ㅋㅋ.. Kotlin 에 대해 짧게나마 얘기 드리자면, 수다를 엄청 좋아하는 Java에 비하면 정말 10배는 좋은 것 같습니다. 코드의 줄 수도 많이 줄고, 그만큼 독해(?) 하는 데 있어서 많이 완화된 모습을 보여줍니다.

다만 지금 Kotlin이 안드로이드 공식 언어가 되었다고 하여, 무작정 Kotlin으로 나가는 것은 개인적으로 추천드리진 않습니다. Java를 확실하게 익혀두신 후, Kotlin에 접해보시기 바랍니다.