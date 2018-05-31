---
layout: post
title: "Android CustomView를 만들어보자"
date: 2017-08-01 20:30:20 +0900
categories: ["android"]
tags: [CustomEditText, CustomView, android]
---

벌써 8월이 시작됐네요. 올해 장마는 무척 길었던 것 같습니다. 

오늘은 안드로이드의 CustomView를 제작하는 간단한 방법을 적어보고자 합니다.

저는 사실 디자인에 많이 약합니다. 제가 현재 맡고 있는 졸업 작품에서도 클라이언트로 안드로이드 앱 개발을 담당하고 있지만, 앱 디자인에 코드를 여러 번 뜯어고쳤습니다. 처음에는 그다지 디자인에 많이 신경도 쓰지 않았고, Android, android-support, material design에서 기본적으로 제공해주는 View, Widget을 가져다가 쓰는 것이 전부이고, 그 이상은 생각하지 않았습니다.

하지만 제가 보기에도 제가 만든 앱의 디자인이 정말 초라하게 느껴집니다. 앱의 품질이 디자인에 좌우되는 것은 아니지만, 그렇다고 디자인이 이쁘지 않으면 저도 그 앱을 꺼려한다는 생각인 것 같아, 이번에 졸업 작품 앱을 개발하게 되면서, 제 손으로 직접 View를 Custom하여 사용해봤고, 그를 통해 얻은 방법을 간단히 적어보고자 합니다.

<br />

## CustomEditText

이 글에서 Custom 해 볼 View는 EditText 입니다. 보통 제공되는 EditText는 다음과 같이 평범하게 글자만 입력할 수 있는 Widget 입니다.

![CustomView-1](/media/images/android/CustomEditText-1.gif)

보다시피 스타일도 평범하고, 그저 글씨만 쓸 수 있어, 앱의 차별성을 두기에는 무언가가 부족합니다. 그래서 저는 이 EditText에 버튼을 추가하여, 입력 자판을 바꾸고자 합니다. 

<br />

## View 를 Custom 하는 방법

먼저 기본적으로 존재하는 View를 Custom 하는 것입니다. 그럴려면 먼저 기본적으로 존재하는 View의 코드를 가져와야 합니다. OOP 언어에는 상속이라는 것이 존재하기 때문에 굳이 코드를 그대로 쓰지 않아도 되므로, 우리는 상속을 사용할 것입니다. 

```java
public class CustomEditText extends AppCompatEditText implements View.OnTouchListener, View.OnFocusChangeListener {
  
}
```

상속과 더불어, 우리는 버튼을 클릭하였을 때, 자판이 바뀌어야 하므로, OnTouchListener, OnFocusChangeListener를 사용합니다. OnFocusChangeListener는 EditText  View에 커서가 붙었는지 붙지 않았는지를 확인해줍니다.

View에는 각 3개의 생성자가 존재합니다. (Lollipop 운영체제에서 4개로 늘었다고는 하는데, EditText에서는 3개가 존재합니다.) 기본 생성자 외에 다른 생성자가 존재하므로, 각 생성자를 생성해줘야 합니다. 



```java
public class CustomEditText extends AppCompatEditText {
  public CustomEditText(Context context) {
    
  }
  
  public CustomEditText(Context context, AttributeSet attrs) {
    
  }
  
  public CustomEditText(Context context, AttributeSet attrs, int defStyleAttr) {
    
  }
}
```

각 생성자에서 초기화 작업을 진행합니다. 우리는 EditText에 버튼을 추가할 것이므로, 버튼 이미지를 초기화 코드에 삽입하여야 합니다. 그러려면 각 코드에 초기화 코드를 삽입해야 합니다.

```java
public void init() {
  Drawable phoneNumImage = ContextCompat.getDrawable(getContext(), R.drawable.ic_dialpad_black_48dp);
  Drawable normalImage = ContextCompat.getDrawable(getContext(), R.drawable.ic_keyboard_black_48dp);

  phoneDrawable = DrawableCompat.wrap(phoneNumImage);
  normalDrawable = DrawableCompat.wrap(normalImage);

  modIcon(phoneDrawable);
  modIcon(normalDrawable);
}
```

입력 자판을 바꾸기 위해서, 번호판이 그려져 있는 Drawable 이미지와, 평범한 자판이 그려진 Drawable 이미지를 사용하였습니다. 

```java
@Override
public void setOnFocusChangeListener(OnFocusChangeListener l) {
  this.onFocusChangeListener = l;
}

@Override
public void setOnTouchListener(OnTouchListener l) {
  this.onTouchListener = l;
}

@Override
public boolean onTouch(View view, MotionEvent motionEvent) {
  final int x = (int)motionEvent.getX();
  if(phoneDrawable.isVisible() && x > getWidth() - getPaddingRight() - phoneDrawable.getIntrinsicWidth()) {
    if(motionEvent.getAction() == MotionEvent.ACTION_UP) {
      setInputType(InputType.TYPE_CLASS_PHONE);
      setText(null);
      keyboardIconVisible();
    }
    return true;
  } else if(normalDrawable.isVisible() && x > getWidth() - getPaddingRight() - normalDrawable.getIntrinsicWidth()) {
    if(motionEvent.getAction() == MotionEvent.ACTION_UP) {
      setInputType(InputType.TYPE_CLASS_TEXT);
      setText(null);
      phoneIconVisible();
    }
    return true;
  }
  return false;
}

@Override
public void onFocusChange(View view, boolean hasFocus) {
  if(hasFocus() && getInputType() == InputType.TYPE_CLASS_TEXT)
    phoneIconVisible();
  else if(hasFocus() && getInputType() == InputType.TYPE_CLASS_PHONE)
    keyboardIconVisible();

  if(onFocusChangeListener != null)
    onFocusChangeListener.onFocusChange(view, hasFocus);
}
```

이제 터치 이벤트와 포커스 이벤트의 코드를 추가합니다. EditText에 커서를 줬을 경우, 먼저 InputType을 확인합니다. InputType이 일반 자판인 경우, 다이얼 모양의 아이콘을 표시합니다. 다이얼 버튼을 터치했을 경우, InputType을 전화기 방식으로 바꾸고, 일반 자판 아이콘이 나타나도록 하는 것입니다.

아래의 화면처럼 구현됩니다.

![CustomView-2](/media/images/android/CustomEditText-2.gif)

실제로 이 EditText는 주소록에서 전화번호로 검색을 할 것이냐, 이름으로 검색할 것이냐에 사용할 수 있습니다. 저의 작품 앱에 사용하진 않았지만, 간단하게 생각이 나서 한 번 구현해봤습니다.

```java
public void init() {
  ...
  super.setOnTouchListener(this);
  super.setOnFocusChangeListener(this);
}
```

TouchListener와 FocusChangeListener를 추가해줍니다. 이 부분을 추가하지 않으면, 동작하지 않습니다.



```java
private void setPhoneIconVisible(boolean visible) {
  phoneDrawable.setVisible(visible, false);
  setCompoundDrawables(null, null, visible ? phoneDrawable : null, null);
}

private void setKeyboardIconVisible(boolean visible) {
  normalDrawable.setVisible(visible, false);
  setCompoundDrawables(null, null, visible ? normalDrawable : null, null);
}

private void modIcon(Drawable iconDrawable) {
  DrawableCompat.setTintList(iconDrawable, getHintTextColors());
  iconDrawable.setBounds(0, 0, iconDrawable.getIntrinsicWidth(), iconDrawable.getIntrinsicHeight());
}

private void phoneIconVisible() {
  setKeyboardIconVisible(false);
  setPhoneIconVisible(true);
}

private void keyboardIconVisible() {
  setPhoneIconVisible(false);
  setKeyboardIconVisible(true);
}
```

메소드는 간단합니다. EditText 우측에 아이콘을 두도록 하는 메소드, 나머지는 아이콘의 표시 유무입니다. 전화 키패드 아이콘의 모양이 나타날 경우, 자판 아이콘이 나타나지 않고, 반대도 그렇게 구현하였습니다.

<br />

## 생성자 이슈

그런데, 제가 생성자를 작성하고 나서 보니, 이런 글을 발견했습니다.

https://kunny.github.io/tip/ui/2016/02/03/custom_view_init/

커니님께서 올린, init() 메소드에 대한 이슈입니다. 물론 init 메소드를 쓰게 되면, 각 생성자마다 함수를 호출하게 되므로 반복 코드를 사용하게 되어 가독성이 조금 흐려집니다. 그래서, 사용한 위 블로그에서는 아래와 같은 방법을 제안하였습니다.

```java
/**
 * 첫 번째 생성자에서, 두 번째 생성자를 호출하고, 그 생성자는 세 번쨰 생성자를
 * 호출 하는 방법입니다.
 *
 * 하지만 Samsung Galaxy S6 Android Nugat 버전에서 이 방법이 동작하지 않습니다.
 */
public CustomEditText(Context context) {
  this(context, null);    // Call Second Constructor
}

public CustomEditText(Context context, AttributeSet attrs) {
  this(context, attrs, 0);    // Call Third Constructor
}

public CustomEditText(Context context, AttributeSet attrs, int defStyleAttr) {
  super(context, attrs, defStyleAttr);

  Drawable phoneNumImage = ContextCompat.getDrawable(getContext(), R.drawable.ic_dialpad_black_48dp);
  Drawable normalImage = ContextCompat.getDrawable(getContext(), R.drawable.ic_keyboard_black_48dp);

  phoneDrawable = DrawableCompat.wrap(phoneNumImage);
  normalDrawable = DrawableCompat.wrap(normalImage);

  modIcon(phoneDrawable);
  modIcon(normalDrawable);

  super.setOnTouchListener(this);
  super.setOnFocusChangeListener(this);
}
```

분명히 중복 코드를 제거할 수 있는 좋은 방법이다. 라고 생각하여 적용을 해봤지만, 결과는 좋지 않았습니다.

![CustomView-3](/media/images/android/CustomEditText-3.gif)

아무리 터치를 해도 반응이 없었고, 심지어는 자판 조차 나오지 않았습니다. 원칙적으로 본다면, 1번째 생성자, 2번째 생성자가 그저 3번째 생성자로 연결되어, 부모 클래스에게 인자를 넘기는 방식이라, 작동에 별 문제가 없다 생각했지만, 디바이스 별로 되는 장치와 안 되는 장치가 있었다는 이슈 등이 있어, 아직까지 모든 디바이스가 사용하기에는 좋은 방법은 아닌 것 같습니다. 

이번 Custom View의 자세한 코드는 Github에 조만간 공개할 예정입니다.

<br />

## 마치며..

안드로이드 프로그래밍 처음으로, 커스텀 뷰라는 것을 만들어봤습니다. GUI 프로그래밍을 해보면서, View를 직접 손으로 만들어보니, 처음부터 만든다고 생각하면 꽤 손이 많이 가지만, 기존에 있는 것을 커스텀 하는 것 만으로도 다른 느낌을 줄 수 있다는 것에 재미를 느낄 수 있었습니다.

여러분들도 즐겁게 코딩하세요 ~ ^^;