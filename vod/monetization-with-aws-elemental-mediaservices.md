---
description: 'Last Updated : 2021-5-20'
---

# Monetization with AWS Elemental MediaServices

이번 모듈의 첫번째 파트에서는 VMAP을 이용하여 VOD에 광고를 넣는 MediaTailor 설정을 생성합니다. 이 실습에서 사용되는 VMAP XML은 Preroll \(방영전\), Midroll \(방영중\), 그리고 Postroll \(방영후\) 광고를 제공합니다. 이 모듈의 두번째 파트에서는 VAST를 이용하여 VOD 콘텐트 \(SCTE-35 가 있는 Manifest\)에 실시간으로 광고를 넣는 실습을 진행합니다.

{% hint style="warning" %}
현재 MediaTailor가 서울 리전에 없기 때문에 다른 리전 \(예, 도쿄 리전\) 을 선택하여 실습을 진행합니다.
{% endhint %}

![](../.gitbook/assets/0%20%281%29.png)

## **VOD 실습**

**이 실습의 목표는 VOD에 방영전, 방영중, 그리고 방영후 광고를 넣는 것입니다.**

1. MediaTailor서비스 콘솔로 들어갑니다.
2. **New configuration Configuration** **name**에 “ImmersionDayVOD”를 입력합니다.

![](../.gitbook/assets/1%20%283%29.png)

1. **Video content source**에 다음 주소를 입력합니다. “https://s3.amazonaws.com/mediaimmersion/mediatailor/vod/”
2. **Ad Decision Server에 다음 주소를 입력합니다.** “https://s3.amazonaws.com/mediaimmersion/mediatailor/content/id\_vmap.xml”

![](../.gitbook/assets/2%20%282%29.png)

1. 마지막으로 **Create configuration**을 클릭합니다.
2. MediaTailor 대쉬보드로 돌아갑니다.
3. 화면의 표에서 방금 생성한 설정과**Playback endpoints**를 확인합니다 \(Propagation에 수분 정도 소요될 수 있습니다\).
4. 페이지 가운데 **HLS Playback Prefix**를 복사합니다.

![](../.gitbook/assets/3%20%281%29.png)

* 콘텐트를 재생하려면 playback URL을 붙여넣기 한 후 “vanlife.m3u8”를 Safari 브라우저 또는[JW Player Stream Tester](https://developer.jwplayer.com/tools/stream-tester/)에 추가합니다 방금 만든 링크는 아래 링크와 유사할 것입니다.

```text
https://85c4d9e190c642b8a55cde0fb024b5bd.mediatailor.us-west-2.amazonaws.com/v1/master/427049e32d97a07c182e16e273df0bfbc6b09413/ImmersionDayVOD/vanlife.m3u8
```

* 최초로 영상을 재생하였을 때에는 “On-the-Fly” 트랜스코딩 작업으로 인해 광고가 나오지 않을 수도 있습니다. 일단 광고가 트랜스코딩 되고 캐쉬에 저장되면, 방송 전, 방송 중, 그리고 방송 후에 광고가 추가된 영상을 확인하실 수 있습니다.

### **라이브 광고 실습**

**이 실습의 목적은 방송 중 정해진 광고 시간 대에 광고를 넣는 것입니다.**

1. [JW Player Stream Tester](https://developer.jwplayer.com/tools/stream-tester/)를 이용하여 Live 워크플로우를 구성에 이용되는 소스 파일을 재생합니다.

```text
 https://s3.amazonaws.com/mediaimmersion/mediatailor/live/vanlife.m3u8
```

* :35 초와 :95초 마크 사이에 광고를 위한 슬레이트가 있는 것을 확인할 수 있습니다. 이번 실습에서는 이 섹션을 광고 콘텐트로 대체할 것입니다.

![](../.gitbook/assets/4%20%283%29.png)

1. MediaTailor service에서, **Create Configuration**을 클릭합니다.
2. **Configuration name**에 “ImmersionDayLive”를 입력합니다.
3. **Video content** source input에 아래 주소를 입력합니다. “https://s3.amazonaws.com/mediaimmersion/mediatailor/live/”
4. **Ad Decision Server**에 아래 주소를 입력합니다. “https://s3.amazonaws.com/mediaimmersion/mediatailor/content/id\_vast.xml”
5. 마지막으로**Create configuration**을 클릭합니다.
6. MediaTailor 대쉬보드로 돌아갑니다.
   1. 화면의 테이블에서 새로 생성한 설정과 그 옆의 **Playback endpoint**를 확인할 수 있습니다 \(Propagation에 수 분 정도 소요될 수 있습니다\).
7. MediaTailor Configuration의 링크를 클릭합니다.
8. 페이지 하단을 내려간 후**HLS Playback Prefix**를 복사합니다.
9. 컨텐트를 재생하려면, playback URL을 붙여넣기한 후 “vanlife.m3u8”을 추가하여Safari 브라우저나[JW Player Stream Tester](https://developer.jwplayer.com/tools/stream-tester/)에 입력합니다.

방금 만든 링크는 아래 링크와 유사할 것입니다. 

> _https://2a10d1a1a4174f349b9887cddd844ddf.mediatailor.us-west-2.amazonaws.com/v1/master/427049e32d97a07c182e16e273df0bfbc6b09413/ImmersionDayLive/vanlife.m3u8_

* 최초로 영상을 재생하였을 때에는 “On-the-Fly” 트랜스코딩 작업으로 인해 광고가 나오지 않을 수도 있습니다. 일단 광고가 트랜스코딩 되고 캐쉬에 저장되면, 방송 전, 방송 중, 그리고 방송 후에 광고가 추가된 영상을 확인하실 수 있습니다.

