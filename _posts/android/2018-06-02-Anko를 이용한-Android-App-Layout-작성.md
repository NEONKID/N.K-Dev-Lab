---
layout: post
title: "Anko를 이용한 Android App Layout 작성"
date: 2018-06-02 03:15:40 +0900
categories: ["android"]
tags: [android, kotlin]
---



안드로이드 개발을 하면서 Kotlin 언어를 접하게 되었고 이를 써보면서 느끼는 바는 확실히 Java보다 강력하고 안정성 있는 언어라는 것을 많이 느낍니다.

그런데 제가 정말로 안드로이드 개발을 하면서 불편하다고 생각했던 것은 바로 findViewbyId 였습니다. 이 것은 XML에 레이아웃을 작성하여 레리아웃에 있는 컴포넌트를 id 값을 이용하여 자바 코드로 가져오는 방법인데, 실제 팀 프로젝트를 진행할 때 Resource 부분에 골 때리는 면이 없지 않아 있었습니다. 일부는 컴파일이 안되거나 갑자기 다른 곳에서 잘 되는 빌드가 옮겨 타면 안되는 현상이 나타나는 등 여러가지 해괴한 현상을 많이 경험했습니다.

그런 저는 findViewById가 onCreate 메소드 밑에 귀찮게 생성하고 하는 것이 영 마음에 꺼림직하여 안드로이드 개발 초기에는 ButterKnife 라이브러리를 사용하여 그 불편함을 조금 해소하였고 나중에는 안드로이드 스튜디오에 바인딩 확장 기능이 나와 조금은 완화 되었지요.

하지만 코틀린 언어를 사용한 뒤, 코틀린 확장 기능 중 findViewbyId를 사용하지 않아도 자동으로 레이아웃의 컴포넌트를 가져와주는 편리한 기능이 있었고 번거롭게 ButterKnife를 추가해야 할 필요가 없어졌습니다. 그런데 이것보다 더 좋은 기능이 있었습니다.



## Using XML 

안드로이드는 JavaFx와 마찬가지로 레이아웃을 XML 언어로 작성하여 컴파일시 그들을 파싱하여 렌더링하여 사용자에게 보여줍니다. 그래서 안드로이드 앱을 개발하려면 당연시 XML 언어를 사용할 줄 알아야 하고 이를 이용해서 레이아웃을 짜야 했습니다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".activities.MainActivity">

    <EditText
        android:id="@+id/name"
        android:hint="Enter your name"
        android:textSize="10sp"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
    
    <Button
        android:id="@+id/clickme"
        android:text="Click me"
        android:textSize="10sp"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

</LinearLayout>
```

EditText 컴포넌트에 아무런 이름을 입력하고 Toast를 출력하는 앱을 만들어보겠다고 가정하고 Layout을 만들면 이렇게 XML로 먼저 미리 만든 다음,

```java
package xyz.neonkid.AnkoExample.activities;

import android.os.Bundle;
import android.support.annotation.Nullable;
import android.support.v7.app.AppCompatActivity;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;

import xyz.neonkid.AnkoExample.R;

/**
 * Created by neonkid on 5/28/18
 **/
public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        final EditText name = findViewById(R.id.name);
        Button btn = findViewById(R.id.clickme);
        btn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                setToastMessage(name.getText().toString());
            }
        });
    }

    public void setToastMessage(String msg) {
        Toast.makeText(this, msg, Toast.LENGTH_LONG).show();
    }
}

```

Java 코드를 예시로 하여금 설명을 드리면 이렇게 findViewbyId를 사용하여 컴포넌트를 가지고 온 다음 액션 로직을 주어 View 코드만 XML, Java 코드를 생성하여야 했습니다. 



## Using Anko

Anko는 이러한 불편함을 덜어주기 위해서 코틀린 언어를 사용하는 개발자라면 XML을 사용하지 않고도 쉽게 레이아웃을 그려줄 수 있도록 하는 라이브러리입니다. 

```kotlin
package xyz.neonkid.AnkoExample.activities

import android.os.Bundle
import android.support.v7.app.AppCompatActivity
import android.widget.Button
import android.widget.EditText
import org.jetbrains.anko.*
import org.jetbrains.anko.sdk25.coroutines.onClick

class MainActivity : AppCompatActivity() {
    private lateinit var name : EditText
    private lateinit var clickmeBtn : Button

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        createView()
    }

    fun createView() {
        verticalLayout {
            padding = dip(8)

            name = editText() {
                hint = "Enter your name"
                textSize = 10f
            }

            clickmeBtn = button("Click me") {
                textSize = 10f
                onClick {
                    clickMeClick()
                }
            }
        }
    }

    fun clickMeClick() {
        toast(name.text.toString()).show()
    }
}

```

Anko를 사용하면 위와 같이 setContentView 메소드를 호출하여 XML 파일을 불러와 파싱 작업을 거치지 않아도 되어 컴파일에 소요하는 CPU와 배터리 시간을 단축시키는 효과가 있습니다. 또한 굳이 View를 보기 위해서 XML을 같이 보는 번거로움도 덜 뿐더러 XML을 할 줄 몰라도 레이아웃을 구성할 수 있다는 장점이 있죠.



## Anko vs XML

그럼 둘을 비교해봤을 때 어떤 장점이 있고 단점이 있는지를 한 번 제 나름대로 적어보도록 하겠습니다.

* Anko 장점
  * XML 을 굳이 몰라도 레이아웃을 구성
  * Activity 한 클래스 안에 View 코드를 모두 볼 수 있어서 용이
  * 레이아웃에서 컴포넌트에 대한 이벤트(버튼 클릭, 컴포넌트 텍스트 변경 등)를 굳이 id 값을 가져오지 않아도 쉽게 변경할 수 있음
  * XML 파싱을 필요로 하지 않기 때문에 컴파일로 인한 시간과 비용을 감소시킬 수 있음.
* Anko 단점
  * XML 레이아웃에는 Preview 기능이 있어 작성할 때마다 바로 Live 기능을 통해 해당 View를 시뮬레이션 하는 기능이 있지만 Anko에는 해당 기능이 없어 처음 레이아웃을 작성하는 사람에겐 어려운 러닝 커브
  * 코틀린 언어에서만 사용할 수 있음

저는 Anko를 사용한지 몇 일 되지 않아서 간단하게 장단점이 이정도였지만 아마 레이아웃에 자신있는 개발자들이라면 Anko가 오히려 편한 점이 더 많을지도 모를 것 같네요. 역시 개발을 잘하면 그만큼 앱의 퍼포먼스가 좋게 나오고 그렇지 않으면 나쁘게 나오는 것이 정말 정답인걸까요...



## How to use

그러면 Anko를 사용하는 방법에 대해 간단히 적어보고 포스트를 마치도록 하겠습니다. Anko를 사용하려면 먼저 코틀린이 지원되도록 안드로이드 프로젝트를 생성한 다음 build.gradle 파일에서 아래와 같은 디펜던시를 추가해줍니다.

```groovy
dependencies {
    implementation "org.jetbrains.anko:anko:0.10.4"
    implementation "org.jetbrains.anko:anko-commons:0.10.4"
}
```

해당 내용은 Gradle 3.0 이상에서 사용 가능하며 Gradle 2.x 버전에서는 아래의 내용을 적으시면 됩니다.

```groovy
dependencies {
    compile "org.jetbrains.anko:anko:0.10.4"
    compile "org.jetbrains.anko:anko-commons:0.10.4"
}
```

그러고 난 후, MainActivity 에서 setContentView 메소드를 지운 다음 onCreate() 메소드에 레이아웃을 정의해주면 됩니다. 만약 번거롭다면 제가 쓴 위의 코드처럼 메소드 하나에 레이아웃 코드를 집어넣는 방법도 좋은 예가 될 수 있죠.

또한 Anko에서 사용할 수 있는 레이아웃은 com.android.support 패키지에서 디펜던시 되지 않고 아래의 패키지에서 디펜던시 됩니다.

```groovy
dependencies {
    // Appcompat-v7 (Anko Layouts)
    implementation "org.jetbrains.anko:anko-appcompat-v7:$anko_version"
    implementation "org.jetbrains.anko:anko-coroutines:$anko_version"
}
```

```kotlin
package com.midas2018mobile5.realmexample.activities

import android.os.Bundle
import android.support.v7.app.AppCompatActivity
import android.widget.Button
import android.widget.EditText
import org.jetbrains.anko.*
import org.jetbrains.anko.sdk25.coroutines.onClick

class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        MainActivityUI().setContentView(this)
    }

    class MainActivityUI : AnkoComponent<MainActivity> {
        private lateinit var name : EditText
        private lateinit var clickmeBtn : Button

        override fun createView(ui: AnkoContext<MainActivity>) = with(ui) {
            verticalLayout {
                padding = dip(8)

                name = editText {
                    hint = "Enter your name"
                    textSize = 10f
                }

                clickmeBtn = button("Click me") {
                    textSize = 10f
                    onClick {
                        toast(name.text.toString()).show()
                    }
                }
            }
        }
    }

}

```

Anko Compat을 상속한 MainActivityUI 클래스를 별도로 생성 혹은 inner class 형태로 만들어주게 되면 아래와 같이 Preview를 보실 수 있습니다.

![Anko_Preview](/media/images/android/ankoPreview.png)





더 자세한 Anko 사용법은 아래의 Github Repo에서 확인하실 수 있습니다.

<div markdown="0"><a href="https://github.com/Kotlin/anko" class="btn btn-info">Anko Github</a></div>

## 마치며...

여기까지 Anko에 대한 간단한 설명을 해봤습니다. XML로 왔다갔다 번거로움이 많았지만 이렇게 코틀린 언어를 가지고 한 클래스 파일에서 쉽게 수정할 수 있도록 되어 있는 것은 매우 좋은 것 같습니다.