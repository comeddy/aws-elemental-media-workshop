---
description: 이 솔루션은 다음과 같은 기능을 제공합니다.
---

# AWS 미디어서비스를 통한 라이브 워크플로 구성 및 CloudFront로 배포하기

이 실습에서는 **AWS Elemental MediaLive**와 **MediaPackage**를 사용하여 라이브 스트리밍 채널을 **Amazon CloudFront** 통한 배포생성하는 것을 진행합니다. 그리고, 추가적으로 OBS를 이용하여 통해 라이브 채널을 최종 단계서 VLC 플레이어로 확인해봅니다.

![ 미디어서비스가 CloudFront 를 통한 라이브 영상 스트리밍 과정 ](../.gitbook/assets/picture1.png)

**AWS Elemental MediaLive**는 브로드캐스트 및 스트리밍 전송을 위한 라이브 출력을 생성할 수 있는 실시간 비디오 서비스로, 실시간 비디오 콘텐츠의 형식 및 패키지를 다른 형식 및 패키지로 변환할 수 있습니다.

**AWS Elemental MediaPackage**는 AWS 클라우드에서 실행되는 JIT(Just-In-Time) 비디오 패키징 및 제작 서비스로, MediaPackage를 사용하면 매우 안전하고 확장 가능하며 신뢰할 수 있는 비디오 스트림을 다양한 재생 디바이스 및 CDN(콘텐츠 전송 네트워크)에 전달할 수 있습니다.

**Amazon CloudFront**는 MediaPackage 사용자 지정 엔드포인트를 오리진으로 사용하도록 구성되어 있으며 요청 인증을 위해 CDN 식별자 사용자 지정 HTTP 헤더를 포함합니다. CloudFront 배포판은 짧은 지연 시간과 빠른 전송 속도로 시청자에게 라이브 스트림을 전달합니다.

### ****

### **1. AWS Elemental MediaPackage Channel 생성**

![미디어 패키지 게시 및 JIP 패키징](../.gitbook/assets/mp.png)

노트북에서 전송하는 라이브 스트림을 방송하기 위해, 별도의 MediaPackage 채널을 생성하겠습니다.

1. MediaPackage 서비스 콘솔 페이지로 이동합니다.
2. 우측 상단 ‘Create’를 클릭합니다.

![](<../.gitbook/assets/image (56).png>)

1. **ID**에 ‘builders-live-channel’를 넣고,
2. Input type이 Apple HLS로 되어있는 것을 확인합니다.
3. **Create a CloudFront distrubtion for this channel** 을 선택합니다.
4. **Create** 버튼을 클릭합니다.

![](../.gitbook/assets/screen-shot-2021-05-16-at-6.53.11-pm.png)

* **Add endpoints** 버튼을 클릭합니다.

![](<../.gitbook/assets/image (85).png>)

* **ID**에 ‘builders-live-hlsendpoint’를 입력한 후 **Save** 버튼을 클릭합니다.

![MediaPackage endpoints ](../.gitbook/assets/screen-shot-2021-05-16-at-6.56.10-pm.png)

* MediaPackage 채널의 생성이 완료 되었습니다. MediaLive 채널을 생성하는 다음 섹션으로 이동합니다.

### 2. AWS MediaLive Channel 생성

![미디어 비디오 프로세싱](../.gitbook/assets/ml.png)

1. **AWS Elemental MediaLive Channel 생성하기**
   1. MediaLive 서비스 콘솔 페이지로 이동합니다.
   2. **Create channel** 버튼을 클릭합니다.

![MediaLive Channel 생성](<../.gitbook/assets/image (118).png>)

* MediaLive channel에 ingest point 역할을 할 Input을 생성합니다. 채널 메뉴에서 **Input attachments > Add** 버튼을 클릭합니다.&#x20;
  1. **Attach input** 에서 **Create input**버튼을 클릭합니다.

![](<../.gitbook/assets/image (5).png>)

![](<../.gitbook/assets/image (98).png>)

* **Inputs**를 선택하고**Create input**을 클릭합니다. **Input name** 입력박스에 ‘live-camera’ 입력하고, RTMP(push)를 **Input type**으로 선택합니다.

![](<../.gitbook/assets/image (29).png>)

* **Input security group** 섹션에서**Create** 옵션을 선택하고 CIDR block 텍스트 필드에 ‘0.0.0.0/0’을 입력한 후, **Create input security group**을 클릭하여 security group을 생성합니다.

![Input Security Group 화면](<../.gitbook/assets/image (63).png>)

* **Input destinations**로 이동합니다.  Destination A의 application name으로 ‘builders’를 입력하고 ‘a’를 instance name으로 입력합니다. Destination B의 application name으로 ‘builders’를 입력하고 ‘b’를 instance name으로 입력합니다.

![MediaLive 입력대상 A/B 설정](../.gitbook/assets/screen-shot-2021-05-16-at-6.59.03-pm.png)

* **Create(생성)**를 클릭하여 **Input(입력)**을 생성합니다.
* Input이 생성된 후, Input 드롭다운 메뉴를 누른 후 ‘live-camera’ 을 선택한 후에 **Confirm (확인)** 버튼을 누릅니다. 여기서 아래 나오는 Input destinations 정보를 기록해 둡니다. 이는 추후 OBS 스트리밍 설정 시 사용하게 됩니다.

![](../.gitbook/assets/screen-shot-2021-05-16-at-7.00.56-pm.png)

* **Channel and input details**을 클릭하여 나머지 MediaLive channel 설정을 수행합니다.
* **Channel name**에 ‘builders-live’을 입력합니다.&#x20;

![](../.gitbook/assets/screen-shot-2021-05-16-at-7.02.58-pm.png)

{% hint style="success" %}
**알고계신가요?** 입력대상 주소를 A와B 이중화가 아닌 단일 파이프라인으로 설정하고 싶을 경우, Channel class를 SINGLE\_PIPELINE으로 선택합니다.
{% endhint %}

* 아래 IAM role 란에는 앞서 만든 MediaLiveAccessRole이 보일 겁니다. Channel template 드롭다운 박스에서 ‘HTTP live streaming(MediaPackage)’을 선택합니다. 팝업 창이 나오면 Confirm을 눌러 넘어갑니다.

![](<../.gitbook/assets/image (19).png>)

1. **왼쪽 메뉴** 바의 **Output groups(출력 그룹)** 섹션에서 **MediaPackage group**을 선택합니다.
2. MediaPackage Destination 칸에서 MediaPackage channel ID 란에 앞서 생성한 MediaPackage의 ID(builders-live-channel)를 입력합니다. 아래 MediaPackage settings의 name에도 같은 값을 입력해 줍니다.

![](../.gitbook/assets/screen-shot-2021-05-16-at-7.05.17-pm.png)

* 아래 MediaPackage outputs에서 Output 10: WebVTT란 오른쪽의 X 버튼을 클릭하여 해당 Output을 삭제합니다.

![HLS 출력화면](<../.gitbook/assets/image (40).png>)

1. **Create channel(채널 생성)**을 클릭하여 채널을 생성합니다.
2. **Start** 버튼을 클릭하고 잠시 기다리면 채널이 시작됩니다.

![](../.gitbook/assets/screen-shot-2021-05-16-at-7.26.04-pm.png)

### **3. OBS Live streaming 생성**

![영상채널 소스](../.gitbook/assets/camera.png)

* [https://obsproject.com](https://obsproject.com/) 에서 OBS Studio를 다운로드한 후 실행합니다.

![](<../.gitbook/assets/image (106).png>)

* OBS의 Sources에서 +를 클릭한 후 Video Capture Device를 선택하면 팝업 설정창이 열립니다.

![](../.gitbook/assets/43.png)

* 팝업 설정창에서 이름을 입력하신 후 OK를 클릭하면 방금 생성한 입력에 대한 Property를 설정하는 화면이 나옵니다.

![](../.gitbook/assets/44.png)

* Property 설정창에서 Device로 노트북의 카메라를 선택하신 후 OK 버튼을 클릭합니다.

![](<../.gitbook/assets/image (8).png>)

* OBS studio 메인화면의 우측 하단의 Settings를 클릭하여 스트리밍 설정을 진행합니다.

![](<../.gitbook/assets/image (32).png>)

* Service값은 Custom으로 선택하고, Server 값으로 앞서 MediaLive Input생성 시 생성되었던 두개의 Endpoint 의 URL 중 Application Name 까지의 URL을 입력하고 Stream Key 값으로는 Instance Name에 입력한 값을 입력합니다. 예를 들어 endpoint의 URL 값이 rtmp://xx.xx.xx.xx:1935/builders/a인 경우 rtmp://xx.xx.xx.xx:1935/builders/를 Server 값으로 입력하고, a를 Stream Key 값으로 입력한 후 OK를 눌러 설정을 완료합니다.

![](../.gitbook/assets/screen-shot-2021-05-16-at-7.07.03-pm.png)

* MediaLive 채널이 Running 상태이고 MediaPackage와 연결된 상태에서 Start Streaming 버튼을 클릭하여 스트리밍을 시작합니다. (스트리밍이 정상적으로 시작되면 OBS 우측 하단에 정사작형의 녹색 표시가 나타답니다.)

![](<../.gitbook/assets/image (45).png>)

![](<../.gitbook/assets/image (97).png>)

\*\*\*\*

### **4. MedaiPackage 의 Endpoint 에서 동영상 재생 확인 및 CloudFront URL 확인**

![](../.gitbook/assets/cloudfront.png)

1. 스트리밍이 시작되면 MediaPackage의 Endpoints에서 Play를 눌러 영상을 확인할 수 있습니다.

![](<../.gitbook/assets/screen-shot-2021-05-16-at-7.14.47-pm (1) (1).png>)

1. PlayBack preview 에서 미리 보기확인과 QR code로 미리보기 할 수 있습니다.

![](../.gitbook/assets/screen-shot-2021-05-16-at-7.16.57-pm.png)

1. [VLC player를 다운로드](https://www.videolan.org/vlc/) 설치 후 네트워크주소에 CloudFront URL을 입력합니다.

![멀티플 OTT Device ](../.gitbook/assets/device.png)

![](<../.gitbook/assets/image (112).png>)

또는 링크를 클릭하여 Safari 브라우저에서 스트림을 재생하거나JW Player 스트림 테스터를 이용하여 재생합니다.

* [https://www.jwplayer.com/developers/stream-tester/](https://www.jwplayer.com/developers/stream-tester/)

![](../.gitbook/assets/screen-shot-2021-05-16-at-9.14.14-pm.png)

1. CloudFront 를 통해 배포되는 라이브 영상 확인이 완료되었습니다.

![](../.gitbook/assets/screen-shot-2021-05-16-at-7.21.45-pm.png)

### **완료**

축하합니다! AWS Elemental MediaLive Service를 이용한 HoL을 완료하셨습니다.
