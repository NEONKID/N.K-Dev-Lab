---
layout: post
title: "OpenCV Trackbar 주무르기"
date: 2017-03-01 09:15:40 +0900
categories: ["opencv"]
tags: [OpenCV, C++]
---

안녕하세요. 새로운 블로그에 저도 모르게 상쾌한 기분이군요. 

오늘은 Tistory 블로그에 이어서, 계속 OpenCV 코드 카테고리 글 작성을 계속하려 합니다.



## Trackbar

OpenCV에는 Trackbar라는 컴포넌트가 존재합니다. (사실 컴포넌트라고 하기에는 조금 흠이 있지만...) 여러분이 원하는 영상을 마우스의 드래그만으로 형상을 변화시킬 수 있도록 하는 것이지요. 말씀만으로는 설명이 어렵기 때문에 간단한 예시를 보며 설명을 드리겠습니다.

![Trackbar_example-1](/media/images/opencv/trackbar-1.png)

위 사진은 Trackbar를 움직여서 가운데에 원을 그려놓고 그 크기를 점점 늘리고 줄이고 할 수 있는 이미지입니다. 간단한 소스 코드를 공개해보도록 하겠습니다.

```c++
#include <opencv2/opencv.hpp>

#define WINDOW_NAME	"circledImage"

using namespace std;
using namespace cv;

void onCircleSizeChange(int pos, void *param)
{
  Mat dstImage = *(Mat*)param;
  Mat newImage = dstImage.clone();
  Point center = Point(dstImage.size().width / 2, dstImage.size().height / 2);

  dstImage.copyTo(newImage);
  if(pos == 0) { return; }
  circle(newImage, center, pos, Scalar(255, 128, 64), 2);

  imshow(WINDOW_NAME, newImage);
}

int main(int argc, char **argv)
{
  Mat srcImage = imread("/home/neonkid/Pictures/bigban.png");
  int size = 1;
  if(srcImage.empty()) { return -1; }
  imshow(WINDOW_NAME, srcImage);
  createTrackbar("Circle size", WINDOW_NAME, &size, srcImage.size().width / 2, onCircleSizeChange, (void*)&srcImage);
  waitKey();

  return 0;
}
```

음 쓰다보니, 가독성이 조금 별로네요.. (다음에는 조금 더 가독성에 고려를 해보도록 하겠습니다.)

코드를 작성하는 것은 그다지 어렵지 않습니다. 여러분이 원하는 이미지나 영상을 한 개 준비하시고, Mat 클래스를 사용해서 영상을 하나 띄웁니다.

그리고, createTrackbar 함수를 사용해 Trackbar를 만들어줍니다. createTrackbar의 인자값은 다음과 같습니다.

```c++
createTrackbar(trackbarName, windowsName, *min_size, max_size, func, image)
```

- trackbarName: trackbar 왼쪽에 있는 이름입니다.
- windowName: 여러분이 다루고자 하는 영상의 윈도우 이름입니다. (이름이 다르면, 다른 창에 trackbar가 입혀집니다.)
- min_size: 최소 Trackbar 크기입니다.
- max_size: 최대 Trackbar 크기입니다. (최대 trackbar 크기를 잘못 지정할 경우, 프로그램이 느려질 수 있습니다.)
- func: 영상을 처리할 함수입니다.
- image: 처리할 이미지입니다.



## Multiple Trackbar

이번에는 Trackbar를 여러개 사용해서, 영상을 다뤄보겠습니다.

![Trackbar_example-2](/media/images/opencv/trackbar-2.png)

위 Trackbar는 색상 Red, Green, Blue를 놓고, 각각의 Trackbar를 움직이면 각 색상이 입혀지는 코드입니다. 소스 코드를 한 번 보겠습니다.

{% highlight cpp %}

#include <opencv2/opencv.hpp>

#define	WINDOW_NAME	"image"

using namespace cv;
using namespace std;

int rgb[3] = {0,0,0};

void onChange(int pos, void* param)
{
    Mat *pMat = (Mat*)param;
    Mat srcImage = Mat(pMat[0]);
    Mat dstImage = Mat(pMat[1]);
    
    for(int i = 0; i < srcImage.rows; i++)
    {
        for(int j = 0; j < srcImage.cols; j++)
        {
            Vec3b rgb_temp = srcImage.at<Vec3b>(i,j);
            Vec3b rgb_sum;
    
            for(int k = 0; k < 3; k++)
            {
                if(rgb_temp[k] + rgb[k] > 255) rgb_sum[k] = 255;
                else if(rgb_temp[k] + rgb[k] < 0) rgb_sum[k] = 0;
                else rgb_sum[k] = rgb_temp[k] + rgb[k];
            }
    
            rgb_temp = Vec3b(rgb_sum[0], rgb_sum[1], rgb_sum[2]);
            dstImage.at<Vec3b>(i,j) = rgb_temp;
        }
    }
    imshow(WINDOW_NAME, dstImage);
}

int main(int argc, char **argv)
{
    Mat image[2];
    image[0] = imread("/home/neonkid/Pictures/bigban.png");
    
    if(image[0].empty())
    {
        return -1;
    }
    
    image[1].create(image[0].size(),CV_8UC3);
    
    onChange(0, (void*)&image);
    
    createTrackbar("Red", WINDOW_NAME, &rgb[2], 255, onChange, (void*)&image);
    createTrackbar("Green", WINDOW_NAME, &rgb[1], 255, onChange, (void*)&image);
    createTrackbar("Blue", WINDOW_NAME, &rgb[0], 255, onChange, (void*)&image);
    
    waitKey();
    
    return 0;
}

{% endhighlight %}

Trackbar를 만드는 방법은 비슷합니다. 색상을 나누는 법에 대해서 잠깐만 얘기해보도록 하죠.

보통 저는 RGB 색상을 나눌 때, 한 이미지의 Mat 클래스에서 Trackbar에 이미지를 넣어주고, 그를 3채널로 구성한 다음 색상을 바꿨지만, 결과는 한 이미지에서 색상이 입혀지는 것이 아닌, 색상이 덮어지는 현상이 나타났습니다.

![Trackbar_example-3](/media/images/opencv/trackbar-3.png)

해당 소스 코드에 대해서는 공개하지 않겠습니다. (메모리 오버플로우 문제 등을 일으킴) 

간단히 접근한 방법에 대해서만 알려드리자면, Vec3b 자료형을 이용해, 컬러값을 저장한 후, 해당 컬러 값을 단순히 합치기만 한 것입니다. 이렇게 할 경우, 아래와 같은 문제가 발생합니다.

1. 영상 3채널 모두 색상을 덮어씌우기 때문에 기존의 형상이 보이지 않게 됨.
2. 단순히 합친 결과를 이미지에 씌우므로 255 이상의 컬러 값이 나타나게 될 경우, 오버플로우 발생.

위 2가지 문제로 인해, 저는 Mat 클래스 배열을 사용하여 각 채널에 해당하는 이미지를 create 함수를 통해 만들어준 후, 각각의 이미지를 건드리는 방법을 사용했습니다.

3채널의 이미지를 두 개 만들어, 임시로 원래 모습의 이미지를 Mat 클래스 변수의 srcImage에 저장해놓습니다. 그리고 해당 색상 값을 temp 변수로 만들어, 저장한 후, 그 위에 새로이 움직이는 taskbar의 컬러 값과 합칩니다. 그러면, 색상이 위에 덮어씌워지지 않고, 원래 이미지에 입혀질 수 있겠죠?

그리고 색상값이 255를 넘기지 않도록 해야 합니다. 이 이유는 taskbar에서 이미 255를 제한걸어놨는데, RGB 색상을 동시에 움직이게 될 경우, 색상값이 255를 넘겨버립니다. 이렇게 될 경우, taskbar에서 최소값이 0이라고 하더라도, 실제 영상의 색상 값은 0이 아닐 수도 있기 때문입니다.

합쳐진 색상 값을 dstImage에 씌우면 적용이 완료됩니다.

![Trackbar_example-4](/media/images/opencv/trackbar-4.png)

OpenCV를 최신 버전으로 패치했더니 빨라진 느낌이네요. (원래 색상 조절하는데 처리가 굉장히 느렸던 것으로 기억하는데...;;)



## 마치며...

여기까지 trackbar를 이용한 몇 가지 소스 코드를 다뤄봤습니다.

trackbar로 할 수 있는 것은 색상만이 있는 것이 아닙니다. 역투영을 할 수도 있고, 확대/축소도 가능하니, 여러분이 원하는 이미지의 색상을 바꾼다거나 역투영을 해보고 싶다면, 이런 소스 코드도 만들어보는게 어떨까요?

Jekyll 블로그의 첫 OpenCV 포스트는 여기까지였습니다. ^^;