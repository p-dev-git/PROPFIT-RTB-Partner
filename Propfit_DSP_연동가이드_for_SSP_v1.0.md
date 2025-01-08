# PROPFIT DSP 연동 가이드 for SSP v1.0
***
- [1. PROPFIT RTB Partner?](#1-propfit-rtb-partner)
    * [1.1. 소개](#11-소개)
    * [1.2. 연동 절차](#12-연동-절차)
    * [1.3. 지원 형식 및 요청](#13-지원-형식-및-요청)
        + [1.3.1. PROPFIT RTB Partner 지원 소재 종류](#131-propfit-rtb-partner-지원-소재-종류)
        + [1.3.2. 전송](#132-전송)
        + [1.3.3. 헤더](#133-헤더)
        + [1.3.4. 데이터 인코딩](#134-데이터-인코딩)
- [2. 입찰 요청 (Bid Request)](#2-입찰-요청-bid-request)
    * [2.1. Bid Request Object](#21-bid-request-object)
    * [2.2. Imp](#22-imp)
    * [2.3. Imp > Banner](#23-imp--banner)
    * [2.4. Imp > Video](#24-imp--video)
    * [2.5. Device](#25-device)
    * [2.6. Device > Geo](#26-device--geo)
    * [2.7. Site](#27-site)
    * [2.8. App](#28-app)
    * [2.9. User](#29-user)
    * [2.10. Banner Ad Types](#210-banner-ad-types)
    * [2.11. Ad Position](#211-ad-position)
    * [3. 입찰 요청 예시](#3-입찰-요청-예시)
        + [3.1. WEB - BANNER](#31-web---banner)
        + [3.2. APP - BANNER](#32-app---banner)
        + [3.3. APP - VIDEO](#33-app---video)
- [4. 응찰 (Bid Response)](#4-응찰-bid-response)
    * [4.1. Bid Response](#41-bid-response)
    * [4.2. SeatBid](#42-seatbid)
    * [4.3. SeatBid > Bid](#43-seatbid--bid)
- [5. 응찰 예시](#5-응찰-예시)
    * [5.1. RTB - 1](#51-rtb---1)
    * [5.2. RTB - 2](#52-rtb---2)
    * [5.3. PMP](#53-pmp)
    * [5.4. 비디오](#54-비디오)
- [6. 매크로 종류](#6-매크로-종류)


# 1. PROPFIT RTB Partner?
## 1.1. 소개
> PROPFIT RTB Partner 는 광고 구매사와 퍼블리셔 인벤토리 판매자 간 실시간 자동 입찰을 진행하는 광고 경매 시스템 입니다. OpenRTB Specification version 2.3 ~ 2.6와 Video Ad Serving Template (VAST) 3.0 을 기반으로 설계되었으며, 모든 스펙 지원이 아닌 제한된 범위의 Object만이 구현되어 있습니다.

## 1.2. 연동 절차
PROPFIT RTB Partner 연동 절차는 다음과 같습니다.
<p align="center">
  <img src="https://github.com/newdana01/campers/assets/60310887/ea3af397-b331-4e62-96a0-a26ce158c0c5">
</p>

## 1.3. 지원 형식 및 요청
### 1.3.1. PROPFIT RTB Partner 지원 소재 종류
- 디스플레이 배너: PROPFIT 에서 구현된 소재를 HTML 형태로 반환
- 비디오: VAST 3.0 규격의 XML 형태로 반환

### 1.3.2. 전송
기본 프로토콜은 HTTP 입니다. 입찰 요청에는 POST가 사용되며, No Bid 인 경우 HTTP 코드 204, 입찰인 경우 200 코드와 함께 Response Body가 반환됩니다. 입찰 요청과 응답은 JSON 형식을 따릅니다.

### 1.3.3. 헤더
HTTP header에는 mime 타입을 나타내는 Content-type을 사용합니다. JSON 임을 명시하는 application/json 을 사용하며, 응답에도 동일한 헤더가 포함되어 있습니다.
```
Content-Type: application/json
```

### 1.3.4. 데이터 인코딩
헤더에 아래 값을 포함하면 데이터를 압축된 형태로 받을 수 있습니다.
```
Accept-Encoding: gzip
```
<br><br>

# 2. 입찰 요청 (Bid Request)
## 2.1. Bid Request Object
> 입찰 요청을 위한 Object 입니다. 해당 Object 로 입찰요청을 보냄으로써 RTB가 시작됩니다. 최상위에는 입찰 고유 id 값을 포함하고 있습니다. 필수 값으로는 id, imp가 있으며 site와 app 중 하나의 attribute가 포함되어야 합니다.

| Attribute | Type          | Required | Comment                                             |
|-----------|---------------|----------|-----------------------------------------------------|
| id        | string        | O        | 입찰 요청 아이디                                           |
| imp       | array(object) | O        | [Imp Object 표 참조](#22-imp)                          |
| site      | object        | O (web)  | [Site Object 표 참조](#27-site)                        |
| app       | object        | O (app)  | [App Object 표 참조](#28-app)                          |
| device    | object        |          | [Device Object 표 참조](#25-device)                    |
| user      | object        |          | [User Object 표 참조](#29-user)                        |
| at        | integer       |          | 옥션 타입 (1:1 st price 입찰, 2:2nd price 입찰)             |
| tmax      | integer       |          | 입찰요청에 대해 응답해야 할 최대시간 (milisecond)                   |
| wseat     | array(string) |          | 입찰요청에 응찰할 수 있는 seat ID 리스트                          |
| cur       | array(string) |          | 허용 가능한 입찰가 통화 단위. ISO-4217 참조<br/>(예: 'KRW', 'USD') |
| bcat      | array(string) |          | 차단할 광고주 카테고리 리스트                                    |
| badv      | array(string) |          | 차단 할 광고주 도메인 리스트 <br/> (예: 'aaa.com', 'bbb.com')    |

## 2.2. Imp
> 입찰 대상 광고의 위치나 광고 종류 등을 나타냅니다.

| Attribute   | Type          | Required | Comment                                                             |
|-------------|---------------|----------|---------------------------------------------------------------------|
| id          | string        | O        | 광고 Impression의 고유 ID                                                |
| banner      | array(object) |          | [Banner Object 표 참조](#23-imp--banner)                               |
| video       | object        |          | [Video Object 표 참조](#24-imp--video)                                 |
| tagid       | object        |          | 특정 광고 게재위치 또는 광고 태그의 식별자                                            |
| bidfloor    | object        |          | 입찰 최저 CPM 단가                                                        |
| bidfloorcur | object        |          | bidfloor CPM의 통화 단위                                                 |
| secure      | integer       |          | 노출에 HTTPS URL이 필요한지 여부를 나타내는 플래그.<br/> 0 : 비보안(http), 1 : 보안(https) |

## 2.3. Imp > Banner
> 배너 광고 요청의 경우 해당 Object를 포함하여야 합니다.

| Attribute | Type           | Required | Comment                                                      |
|-----------|----------------|----------|--------------------------------------------------------------|
| id        | string         | O        | 배너 고유 ID                                                     |
| w         | integer        | O        | 배너의 width                                                    |
| h         | integer        | O        | 배너의 height                                                   |
| pos       | integer        |          | 광고 위치 코드 [(Ad Position 표 참조)](#211-ad-position)              |
| btype     | array(integer) |          | 차단 할 배너 타입 코드 [(Banner Ad Types 표 참조)](#210-banner-ad-types) |
| battr     | array(integer) |          | 차단할 광고물 타입 (IAB OpenRTB 2.3.x 참조)                            |


## 2.4. Imp > Video
> 비디오 광고 요청의 경우 해당 Object를 포함하여야 합니다.

| Attribute      | Type           | Required | Comment                                      |
|----------------|----------------|----------|----------------------------------------------|
| mimes          | array(string)  | O        | 콘텐츠 MIME 유형                                  |
| minduration    | integer        | O        | 최소 동영상 광고 재생 시간(초)                           |
| maxduration    | integer        | O        | 최대 동영상 광고 재생 시간(초)                           |
| protocols      | array(integer) |          | 지원되는 동영상 입찰 응답 프로토콜                          |
| w              | integer        | O        | 비디오 플레이어의 너비(픽셀)                             |
| h              | integer        | O        | 비디오 플레이어의 높이(픽셀)                             |
| linearity      | integer        |          | 노출 선형,비선형 여부                                 |
| startdelay     | integer        |          | 광고가 시작되는 시작 딜레이 초 단위                         |
| placement      | integer        |          | impression의 배치 유형                            |
| sequence       | integer        |          | 여러 개의 impression일 경우 시퀀스 숫자.                 |
| playbackmethod | array(integer) |          | IAB OpenRTB Spec 2.5 > 3.2.7 표 참조            |
| minbitrate     | integer        |          | 최소 비트 속도                                     |
| maxbitrate     | integer        |          | 최대 비트 속도                                     |
| delivery       | array(integer) |          | 지원하는 전달 방법. IAB OpenRTB Spec 2.5 > 5.11 표 참조 |


## 2.5. Device
> 사용자와 상호작용하는 장치와 관련된 정보를 담고 있습니다. 

| Attribute  | Type    | Required | Comment                         |
|------------|---------|----------|---------------------------------|
| ua         | string  |          | 브라우저 User Agent                 |
| dnt        | integer |          | 광고 추적 제한<br/>0: 추적허용, 1: 추적금지   |
| ip         | string  |          | Device의 IP Address              |
| devicetype | integer |          | Device의 종류                      |
| make       | string  |          | Device의 제조사                     |
| model      | string  |          | Device의 모델                      |
| os         | string  |          | Device의 OS                      |
| osv        | string  |          | Device의 OS Version              |
| geo        | object  |          | Device의 위치 정보                   |
| ifa        | string  |          | 광고 식별 ID (type: idfa, gaid..등)  |
| didsha1    | string  |          | SHA1로 암호화 된 Device ID           |
| didmd5     | string  |          | MD5로 암호화 된 Device ID            |
| dpidsha1   | string  |          | SHA1로 암호화 된 Platform Device ID  |
| dpidmd5    | string  |          | MD5로 암호화 된 Platform Device ID   |

## 2.6. Device > Geo
> Device의 위치 정보를 나타내는 객체입니다.

| Attribute | Type    | Required | Comment  |
|-----------|---------|----------|----------|
| lat       | float   |          | 위도       |
| lon       | float   |          | 경도       |
| type      | integer |          | 위치 정보 방식 |
| country   | string  |          | 국가 코드    |
| region    | string  |          | 지역 코드    |

## 2.7. Site
> 광고가 전송될 지면이 웹사이트인 경우 이 Object를 포함해야 합니다. 입찰 요청은 Site 및 App 객체를 모두 포함할 수 없습니다.

| Attribute | Type          | Required     | Comment                           |
|-----------|---------------|--------------|-----------------------------------|
| id        | string        | O (web 일 경우) | Exchange의 Site ID                 |
| name      | string        |              | Exchange의 Site Name               |
| domain    | string        |              | 매체 Domain. (예: ‘www.performancebytbwa.net’) |
| cat       | array(string) |              | 사이트에 대한 IAB 컨텐츠 카테고리 리스트          |
| page      | string        |              | 현재 페이지의 URL                       |

## 2.8. App
> 광고가 전송될 지면이 mobile과 같은 non-browser 형태인 경우 이 Object를 포함해야 합니다. 입찰 요청은 Site 및 App 객체를 모두 포함할 수 없습니다.

| Attribute | Type          | Required     | Comment                |
|-----------|---------------|--------------|------------------------|
| id        | string        | O (App 일 경우) | Exchange의 App ID       |
| name      | string        |              | Exchange의 App Name     |
| bundle    | string        |              | App Bundle 또는 Package명 |
| cat       | array(string) |              | App에 대한 컨텐츠 카테고리 리스트   |

## 2.9. User
> 디바이스 사용자의 정보를 나타냅니다.

| Attribute | Type          | Required | Comment             |
|-----------|---------------|----------|---------------------|
| id        | string        |          | Exchange의 사용자 고유 ID |
| buyeruid  | string        |          | Exchange의 App Name  |

## 2.10. Banner Ad Types
> 다음 표는 배너 광고 유형을 나타냅니다.

| Attribute | Comment         |
|-----------|-----------------|
| 1         | XHTML Text 광고   |
| 2         | XHTML Banner 광고 |
| 3         | Javascript 광고   |
| 4         | iframe 광고       |

## 2.11. Ad Position
> 다음 표는 광고의 위치를 지정하는 속성을 나타냅니다.

| Attribute | Comment                                          |
|-----------|--------------------------------------------------|
| 0         | Unknown                                          |
| 1         | Above the Fold                                   |
| 2         | DEPRECATED - 화면크기/해상도에 따라 처음에 표시되거나 표시되지 않을 수 있음 |
| 3         | Below the Fold                                   |
| 4         | Header                                           |
| 5         | Footer                                           |
| 6         | Sidebar                                          |
| 7         | Full screen                                      |

<br><br>

## 3. 입찰 요청 예시
### 3.1. WEB - BANNER
```json
{
  "id":"4321h3245g1234y1234iu2345k63456lkj2345j4",
  "at":1,
  "cur":["USD"],
  "imp":
  [
    {
      "id":"1",
      "bidfloor":0.03,
      "bidfloorcur":"USD",
      "banner":
      {
        "h":200,
        "w":300,
        "pos":0
      }
    }
  ],
  "site":
  {
    "id":"123456",
    "cat":["IAB5-1","IAB5-2","IAB5-3"],
    "domain":" www.test.com ",
    "page":"http://www.test.com/test.html"
  },
  "device":
  {
    "ua":"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.116 Safari/537.36",
    "ip":"172.217.31.132"
  },
  "user":
  {
    "id":"1234b2345m3456j2345g2345g2345ui2345u245a"
  },
  "source":
  {
    "tid":"1234b2345m3456j2345g2345g2345ui2345u245a"
  }
}
```

### 3.2. APP - BANNER
```json
{
    "id": "b680210a-17b4-4650-8bd9-f99c89c74e3a",
    "imp": [
        {
            "id": "1eb3fd3b-b52e-4f33-8474-3762efdcb64d",
            "instl": 0,
            "tagid": "f9278757",
            "secure": 1,
            "banner": {
                "w": 320,
                "h": 480,
                "btype": [
                ],
                "pos": 0
            },
            "bidfloor": 0.0809688576,
            "bidfloorcur": "USD"
        }
    ],
    "device": {
        "carrier": "LG Uplus",
        "ip": "218.39.94.47",
        "language": "ko",
        "make": "LG",
        "model": "LGM-G600S",
        "os": "Android",
        "osv": "7.0",
        "devicetype": 4,
        "ifa": "894dfcb3-8ac4-43c0-a69c-012345678901",
        "geo": {
            "lat": 37.5985,
            "lon": 126.9783,
            "type": 2,
            "city": "Seoul",
            "zip": "02878",
            "country": "KOR",
            "region": "11"
        }
    },
    "app": {
        "id": "1233434",
        "name": "앱명",
        "bundle": "com. app",
        "storeurl": "https://play.google.com/store/apps/details?id=com.app",
        "cat": ["IAB3"],
        "publisher": {
            "name": "(주)매체",
            "id": "a345434e-31ba-488c-939c-290c48d577e4"
        }
    },
    "cur": ["USD"],
    "tmax": 234,
    "badv": ["block.com", "adv_test.com"],
    "at": 2
}
```

### 3.3. APP - VIDEO
```json
{
  "id": "flower.req.id-b8cc05be-adec-4472-8d7a-0dd961015310",
  "imp": [
    {
      "id": "1",
      "video": {
        "mimes": ["video/mp4"],
        "minduration": 5,
        "maxduration": 1000,
        "protocols": [2, 3, 5, 6],
        "w": 1920,
        "h": 1080,
        "startdelay": 0,
        "linearity": 1
      },
      "instl": 1,
      "tagid": "apm_linear_q_skbroadband_01",
      "bidfloor": 3,
      "bidfloorcur": "USD",
      "secure": 0,
      "pmp": {
        "private_auction": 0,
        "deals": [
          {
            "id": "d227e695-3aa2-4683-ab4c-e8c392a7c70c",
            "bidfloor": 3,
            "bidfloorcur": "USD",
            "at": 3
          }
        ]
      }
    }
  ],
  "app": {
    "id": "tv.skb",
    "bundle": "tv.skb",
    "domain": "skbrodband.com",
    "publisher": {
      "id": "1"
    }
  },
  "device": {
    "ua": "SupplySide-SSP/2.0",
    "geo": {
      "lat": 37.56,
      "lon": 126.91,
      "region": "KR-11",
      "city": "Seoul",
      "country": "KOR",
      "zip": "23145"
    },
    "ip": "/127.0.0.1:37288",
    "devicetype": 3,
    "language": "ko",
    "ifa": "test_ad"
  },
  "at": 1,
  "tmax": 500
}
```

<br><br>

# 4. 응찰 (Bid Response)
## 4.1. Bid Response
> 하나의 입찰요청에 대해 하나의 응답이 반환됩니다. Object 최상위에는 Bid Request 의 id가 그대로 사용됩니다.

| Attribute | Type          | Required | Comment                          |
|-----------|---------------|----------|----------------------------------|
| id        | string        | O        | BidRequest ID                    |
| seatid    | array(object) |          | SeatBid 표 참조                     |
| bidid     | string        |          | 로깅 / 추적을 지원하기 위해 입찰자가 생성 한 응답 ID |
| cur       | string        |          | 응찰가의 통화                          |
| nbr       | integer       |          | 응찰포기 사유                          |

## 4.2. SeatBid
| Attribute | Type          | Required | Comment         |
|-----------|---------------|----------|-----------------|
| bid       | array(object) |          | Bid Object 표 참조 |
| seatid    | string        |          | 응찰자 seat ID     |

## 4.3. SeatBid > Bid
| Attribute | Type           | Required | Comment                                     |
|-----------|----------------|----------|---------------------------------------------|
| id        | string         | O        | 로깅/추적을 위해 입찰자가 생성한 유니크 ID                   |
| impid     | string         | O        | BidRequest의 Imp 항목의 ID                      |
| price     | float          | O        | CPM으로 환산된 응찰 가격                             |
| adid      | string         |          | 응찰건에 대해 낙찰 시 전송될 광고 ID                      |
| nurl      | string         |          | 응찰건에 대해 낙찰 시 응찰자에게 알릴 Postback URL          |
| lurl      | string         |          | 응찰건에 대해 패찰 시 응찰자에게 통보해 줄 알림 URL             |
| adm       | string         | O        | 낙찰시 전송될 광고 마크업(Markup)                      |
| adomain   | array(string)  | O        | 차단목록 확인을 위한 광고주 Domain 리스트                  |
| bundle    | object         |          | 광고주 앱의 Bundle ID 또는 패키지명                    |
| iurl      | string         | O        | 이미지 URL                                     |
| cid       | string         |          | 캠페인 ID                                      |
| crid      | string         |          | 광고물 ID                                      |
| cat       | array(string)  |          | 광고물에 대한 IAB 카테고리 리스트 (IAB OpenRTB 2.3.x 참조) |
| attr      | array(integer) |          | 광고물 타입 리스트 (IAB OpenRTB 2.3.x 참조)           |

<br><br>

# 5. 응찰 예시
## 5.1. RTB - 1
```json
{
    "id": "ptbwa_593c7ed4-1650-4304-aeed-02560b41e84c",
    "cur": "USD",
    "seatbid": [
        {
            "seat": "151",
            "bid": [
                {
                    "id": "b680210a-17b4-4650-8bd9-f99c89c74e3a",
                    "impid": "1eb3fd3b-b52e-4f33-8474-3762efdcb64d",
                    "price": 0.0827501724672,
                    "nurl": "https://bid.exemple.com/ptbwa/v1/rtb.nurl?tid=ptbwa_593c7ed4-1650-4304-aeed-02560b41e84c&is_cpvc=0&imp_id=1eb3fd3b-b52e-4f33-8474-3762efdcb64d&imp_idx=0",
                    "iurl": "https://creative.exemple.com/creatives/image/2024/03/19/14/20240319140509002.jpg",
                    "lurl": "https://bid.exemple.com/ptbwa/v1/rtb.lurl?tid=ptbwa_593c7ed4-1650-4304-aeed-02560b41e84c&is_cpvc=0&imp_id=1eb3fd3b-b52e-4f33-8474-3762efdcb64d&imp_idx=0",
                    "adm": "<div style=\"width:320px; height:480px; z-index:999; overflow:hidden; margin-left:auto; margin-right:auto;\" onclick=\"new Image().src='https://bid.exemple.com/ptbwa/v1/rtb.click?tid=ptbwa_593c7ed4-1650-4304-aeed-02560b41e84c&is_cpvc=0&imp_id=1eb3fd3b-b52e-4f33-8474-3762efdcb64d&imp_idx=0';\"><meta name='viewport' content='width=device-width, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0, user-scalable=no' /><style type='text/css'>body {margin: 0;padding: 0;}</style><a href=\"https://migunlife.co.kr/%EB%AF%B8%EA%B1%B4%EC%B2%B4%ED%97%98%EC%84%BC%ED%84%B0-%EB%AA%A8%EC%A7%91\" target=\"_blank\"><img width=\"320\" height=\"480\" src=\"https://creative.exemple.com/creatives/image/2024/03/19/14/20240319140509002.jpg\" /></a></div><img src=\"https://bid.exemple.com/ptbwa/v1/rtb.imp?tid=ptbwa_593c7ed4-1650-4304-aeed-02560b41e84c&auction_price=${AUCTION_PRICE}&is_cpvc=1&imp_id=1eb3fd3b-b52e-4f33-8474-3762efdcb64d&imp_idx=0\" style=\"width:0px; height:0px; border:0px;\">",
                    "adomain": ["advertiserdomain.com"],
                    "bundle": "",
                    "cat": ["IAB01"],
                    "cid": "151_272",
                    "crid": "122",
                    "attr": [],
                    "seat": "nhn"
                }
            ]
        }
    ]
}
```

## 5.2. RTB - 2
```json
{
    "id": " ptbwa_8201c812-f432-4aab-ab59-39bbd640e7d5",
    "bidid": "a1123",
    "cur": "USD",
    "seatbid": 
    [
        {
            "seat": "123",
            "bid": 
            [
                {
                    "id": "1",
                    "impid": "102",
                    "price": 9.43,
                    "nurl": "http://adserver.com/winnotice?impid=102",
                    "lurl": "http://adserver.com/lossnotice?impid=102&lossreason=${AUCTION_LOSS}",
                    "adm": "<!DOCTYPE html><html>…$(AUCTION_PRICE}</html>",
                    "iurl": "http://adserver.com/pathtosampleimage",
                    "adomain": [ "advertiserdomain.com" ],
                    "cid": "campaign1",
                    "crid": "creative1",
                    "attr": [ 4, 5, 6, 7 ]
                }
            ]
        }
    ]
}
```

## 5.3. PMP
```json
{
  "id": "ptbwa_8201c812-f432-4aab-ab59-39bbd640e7d3",
  "cur": "USD",
  "seatbid": [
    {
      "seat": "82",
      "bid": [
        {
          "id": "flower.req.id-b8cc05be-adec-4472-8d7a-0dd961015310",
          "impid": "1",
          "price": 3.0831,
          "nurl": "",
          "iurl": "https://creative.exemple.com/creatives/video/2023/12/13/15/20231213153604001.mp4",
          "lurl": "",
          "adm": "<VAST version=\"3.0\"><Ad id=\"73\">........</InLine></Ad></VAST>",
          "adomain": ["adomain.com"],
          "bundle": "",
          "cat": [],
          "dealid": "d227e695-3aa2-4683-ab4c-e8c392a7c70c",
          "dealbidfloor": 3,
          "dealbidfloorcur": "USD",
          "cid": "82_215",
          "crid": "73",
          "attr": [],
          "seat": "anypointmedia"
        }
      ]
    }
  ]
}
```

## 5.4. 비디오
> 비디오 광고 요청 시 응답 값에 VAST 3.0 규격의 XML을 반환합니다.
```xml
<VAST version="3.0">
    <Ad id="125">
        <InLine>
            <AdSystem version="1.0">
                <![CDATA[Propfit]]>
            </AdSystem>
            <AdTitle>
                <![CDATA[Campaign Title]]>
            </AdTitle>
            <Description>
                <![CDATA[Campaign Description]]>
            </Description>
            <Error>
                <![CDATA[https://bid.exemple.com/ptbwa/v1/rtb.error?tag=KLT8FS4IEW2X&req_id=uuid&code=${CODE}]]>
            </Error>
            <Impression id="0">
                <![CDATA[https://bid.exemple.com/ptbwa/v1/rtb.imp?tag=KLT8FS4IEW2X&req_id=uuid&res_id=ptbwa_cc6c5bb3-58c1-486e-a735-882e28168676&tag_id=vod&cmp=146&ag=265&creative=125&auction_price=${AUCTION_PRICE}&ad_type=3&app_id=&app_name=gima&bundle=google.ima&ifa=53bc7daf-4e0f-4274-a799-022a6e46dfa6]]>
            </Impression>
            <Creatives>
                <Creative id="125" sequence="1">
                    <Linear>
                        <Duration>
                            <![CDATA[00:00:14]]>
                        </Duration>
                        <TrackingEvents>
                            <Tracking event="start">
                                <![CDATA[https://bid.exemple.com/ptbwa/v1/rtb.vast/tracking/start?tag=KLT8FS4IEW2X&req_id=uuid&res_id=ptbwa_cc6c5bb3-58c1-486e-a735-882e28168676&tag_id=vod&cmp=146&ag=265&creative=125&auction_price=${AUCTION_PRICE}&ad_type=3&app_id=&app_name=gima&bundle=google.ima&ifa=53bc7daf-4e0f-4274-a799-022a6e46dfa6]]>
                            </Tracking>
                            <Tracking event="firstQuartile">
                                <![CDATA[https://bid.exemple.com/ptbwa/v1/rtb.vast/tracking/firstQ?tag=KLT8FS4IEW2X&req_id=uuid&res_id=ptbwa_cc6c5bb3-58c1-486e-a735-882e28168676&tag_id=vod&cmp=146&ag=265&creative=125&auction_price=${AUCTION_PRICE}&ad_type=3&app_id=&app_name=gima&bundle=google.ima&ifa=53bc7daf-4e0f-4274-a799-022a6e46dfa6]]>
                            </Tracking>
                            <Tracking event="midpoint">
                                <![CDATA[https://bid.exemple.com/ptbwa/v1/rtb.vast/tracking/mid?tag=KLT8FS4IEW2X&req_id=uuid&res_id=ptbwa_cc6c5bb3-58c1-486e-a735-882e28168676&tag_id=vod&cmp=146&ag=265&creative=125&auction_price=${AUCTION_PRICE}&ad_type=3&app_id=&app_name=gima&bundle=google.ima&ifa=53bc7daf-4e0f-4274-a799-022a6e46dfa6]]>
                            </Tracking>
                           <Tracking event="thirdQuartile">
                                <![CDATA[https://bid.exemple.com/ptbwa/v1/rtb.vast/tracking/thirdQ?tag=KLT8FS4IEW2X&req_id=uuid&res_id=ptbwa_cc6c5bb3-58c1-486e-a735-882e28168676&tag_id=vod&cmp=146&ag=265&creative=125&auction_price=0.5&ad_type=3&app_id=&app_name=gima&bundle=google.ima&ifa=53bc7daf-4e0f-4274-a799-022a6e46dfa6]]>
                            </Tracking>
                            <Tracking event="complete">
                                <![CDATA[https://bid.exemple.com/ptbwa/v1/rtb.vast/tracking/complete?tag=KLT8FS4IEW2X&req_id=uuid&res_id=ptbwa_cc6c5bb3-58c1-486e-a735-882e28168676&tag_id=vod&cmp=146&ag=265&creative=125&auction_price=0.5&ad_type=3&app_id=&app_name=gima&bundle=google.ima&ifa=53bc7daf-4e0f-4274-a799-022a6e46dfa6]]>
                            </Tracking>
                            <Tracking event="progress" offset="00:00:10">
                                <![CDATA[https://bid.exemple.com/ptbwa/v1/rtb.vast/tracking/progress?tag=KLT8FS4IEW2X&req_id=uuid&res_id=ptbwa_cc6c5bb3-58c1-486e-a735-882e28168676&tag_id=vod&cmp=146&ag=265&creative=125&auction_price=0.5&ad_type=3&app_id=&app_name=gima&bundle=google.ima&ifa=53bc7daf-4e0f-4274-a799-022a6e46dfa6]]>
                            </Tracking>
                        </TrackingEvents>
                        <VideoClicks>
                            <ClickTracking id="blog">
                                <![CDATA[https://bid.exemple.com/ptbwa/v1/rtb.click?tag=KLT8FS4IEW2X&req_id=uuid&res_id=ptbwa_cc6c5bb3-58c1-486e-a735-882e28168676&tag_id=vod&cmp=146&ag=265&creative=125&auction_price=0.5&ad_type=3&app_id=&app_name=gima&bundle=google.ima&ifa=53bc7daf-4e0f-4274-a799-022a6e46dfa6]]>
                            </ClickTracking>
                        </VideoClicks>
                        <MediaFiles>
                            <MediaFile id="125" delivery="progressive" type="video/mp4" bitrate="28351" width="1920" height="1080">
                                <![CDATA[https://creative.exemple.com/creatives/video/2024/05/16/15/20240516153526001.mp4]]>
                            </MediaFile>
                        </MediaFiles>
                    </Linear>
                </Creative>
            </Creatives>
        </InLine>
    </Ad>
</VAST>
```

<br><br>

# 6. 매크로 종류
> 낙찰통보(Win Notice) URL은 입찰자에 의해 결정됩니다. 가격 등의 정보를 입찰자에게 전달하기 위한 몇 가지 매크로가 URL에 삽입 될 수 있습니다. 낙찰 시 매체에서는 매크로를 적정한 값으로 치환합니다. 대체할 값이 없다면 URL에서 삭제합니다.

| Macro               | Comment                                              |
|---------------------|------------------------------------------------------|
| ${AUCTION_ID}       | 입찰 요청의 ID. BidRequest.id 값                           |
| ${AUCTION_BID_ID}   | 입찰 ID. BidResponse.bidid 값                           |
| ${AUCTION_IMP_ID}   | 노출 ID. BidResponse.seatbid.bid.impid 값               |
| ${AUCTION_SEAT_ID}  | Seat ID. BidResponse.seatbid.seat 값                  |
| ${AUCTION_AD_ID}    | 입찰자가 게재하려는 광고 마크업 ID. BidResponse.seatbid.bid.adid 값 |
| ${AUCTION_PRICE}    | 입찰과 동일한 통화 및 단위를 사용한 정산 가격.                          |
| ${AUCTION_CURRENCY} | 입찰에 사용된 통화. 확인 전용                                    |

<br>
