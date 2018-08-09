---
title: "程序化购买专题广告需求方平台DSP竞价流程"
date: 2017-08-21T17:21:49-07:00
draft: false
slug: Programatic-Buying-Advertising-2
gitment: true
---
## 概要
作为 DSP 角色，当接受到由广告交易平台(ADX)发送来的竞价请求时，通常会根据一些业务需求、广告主要求、用户特征等，对请求进行过滤，从而决策竞价与否。如果决定参与竞价，则会对请求特征连同自己的广告数据库、用户数据库等进行一系列的计算匹配，同时根据广告(Ads)、竞选(Campaign)、广告位(Placement)等的时长、已展示量等特征计算出竞价。以此逐步筛选，最终选出最优的广告以及竞选价格返回给广告交易平台。整个流程根据业界标准，SLA 通常需要在 200ms 内完成，同时考虑到网络通信的损耗，真正数据在 DSP 内部的用时不能超过 100ms。因为极高的 SLA 要求，所以对系统布局和算法提出了挑战。

## 具体流程
1. 接收竞价请求(Bid Request)
我们以 OpenRTB v2.4 的竞价请求为例，如下图。

```json
{
    "id": "DxU0032U8a",
    "at": 2,
    "allimps": 0,
    "imp": [
        {
            "id": "1",
            "banner": {
                "w": 320,
                "h": 50,
                "format": [
                    {
                        "w": 320,
                        "h": 50
                    }
                ],
                "btype": [
                    1,
                    3
                ],
                "battr": [
                    1,
                    3,
                    5,
                    6,
                    8,
                    9,
                    10,
                    11
                ],
                "pos": 3,
                "mimes": [
                    "image/jpeg",
                    "image/png",
                    "image/gif"
                ],
                "api": []
            },
            "ext": {
                "strictbannersize": 0
            },
            "instl": 0,
            "displaymanager": "SOMA",
            "tagid": "101000415",
            "secure": 0
        }
    ],
    "device": {
        "geo": {
            "lat": 53.550003,
            "lon": 10,
            "ipservice": 3,
            "country": "DEU",
            "region": "04",
            "zip": "20099",
            "metro": "0",
            "city": "Hamburg",
            "type": 2
        },
        "make": "Apple",
        "model": "iPhone",
        "os": "iOS",
        "osv": "4.3.2",
        "ua": "Mozilla/5.0 (iPhone; U; CPU iPhone OS 4_3_2 like Mac OS X; en-us) AppleWebKit/533.17.9 (KHTML, like Gecko) Version/5.0.2 Mobile/8H7 Safari/6533.18.5",
        "ip": "95.90.255.47",
        "js": 0,
        "connectiontype": 0,
        "ifa": "e4273e31-97a9-4b29-93a8-8a99f0cea068",
        "devicetype": 1
    },
    "app": {
        "id": "101000415",
        "name": "OpenRTB_2_4_UATest_openRtb_2_4_iOS_XXLARGE_320x50_IAB1",
        "domain": "example.com",
        "cat": [
            "IAB1"
        ],
        "storeurl": "http://example.com",
        "keywords": "",
        "publisher": {
            "id": "1001028764",
            "name": "OpenRTB_2_4_UATest_openRtb_2_4_iOS_XXLARGE_320x50_IAB1"
        }
    },
    "user": {
        "keywords": ""
    },
    "bcat": [
        "IAB17-18",
        "IAB7-42",
        "IAB23",
        "IAB7-28",
        "IAB26",
        "IAB25",
        "IAB9-9",
        "IAB24"
    ],
    "badv": [],
    "ext": {
        "udi": {
        },
        "operaminibrowser": 0,
        "carriername": "unknown - probably WLAN"
    },
    "regs": {
        "coppa": 0
    }
}
```
这个竞价请求来自于一个 iOS App，当我们深入这个请求结构可以发现，这是一个横幅广告(Banner Ad)，宽度 320px，高度 50px，来自一台 iOS 设备，操作系统是 4.3.2，地理坐标为经度 53.550003，纬度 10，邮政编码是20099，城市为Hamburg，用户的 IP 地址，以及这个 App 的信息例如名字、编号、域名、App Store 的地址、发行商等，同时竞价请求也包含了这个广告需要满足的类别，比如年龄的要求等。

2. 过滤(Filter)竞价请求
再接收到竞价请求后，DSP 是否参与这次竞价一般会基于几个方面的考虑：

1) 该请求是否在黑名单内？
当作弊检测(Fraud Detection)发现，该请求属于有害请求或者无效请求，则需要选择性的过滤掉该竞价请求。作弊检测通常基于 IP 地址、用户设备 ID、地理位置以及 App 本身属性等特征进行判断。

2) 是否参与该 ADX 的竞价？
此选项往往基于业务的考虑，例如某些 ADX 平台的流量质量不佳或者利润抽成过高，都可能导致不参与竞价。

3) 服务器集群是否过载？
当遇到例如双 11或者感恩节时候，广告的竞价请求往往会出现持续的洪峰，如果不进行限流可能引起集群服务雪崩，从而影响正常业务。

过滤竞价请求能够防止类似于机器人点击等作弊性质的刷量行为，有效保护了广告主的利益。同时遇到流量洪峰时，起到削峰整流的作用，保护正常业务的运转。

3. 预处理
首先声明，并不是所有 DSP 都必须做预处理。或者说，预处理的内容完全取决于每家 DSP 的自身需求。在预处理阶段，DSP 可以对请求 App 或者网页的内容进行预爬取，然后提取一些 NLP 特征处理，或者有预先缓存好的网页特征，则可以直接提取插入到请求特征中。或者某些 DSP 拥有较大的用户数据库，则会对请求用户提取出预先计算好的用户特征段(Segments)出来以便后续计算。预处理主要目的是为了更好地定位受众(Audience Targeting)。

4. 筛选广告
程序化购买的优势之一就在于精准投放。而体现了这一优势的地方就在于如何筛选广告。最基本的可以直接根据用户浏览的网页内容进行广告推荐，例如用户浏览汽车网站，则可以推荐汽车的品牌广告。如果做得精度更高，则可以对用户进行画像(Demographic)，对用户一系列的历史行为进行分析，例如最近一段时间去过哪些地方、购买过什么商品、浏览了什么网页，对用户的行为、喜好有清晰的刻画。

以精准投放为目的，训练出一系列的算法模型，对于每个用户，可以在众多的广告库中筛选出用户最有可能感兴趣的一个，从而达到将广告推荐给有效用户的目的，进而提高点击率(CTR)和转化率(ROI)。无论是最基本的图片广告，还是最新的动态创意优化(DCP)，都是以此为基础发展的。

5. 定价
实时竞价通常采用次价密封投标拍卖(Second-price sealed-bid auction)，所以如何既能中标又能以最低价格中标，成为定价阶段的主要目标，这些主要以博弈论为理论基础，我们就在此不赘述。除此之外，如何在竞选时间内平稳均匀地将广告展示(Impression)出去，而不是在竞选初期消耗掉大量展示量造成后期展示量不足，也是定价的次要目标，而业界通常以 Pacing 算法为原型，拓展出其他预算模型控制出价。

6. 返回竞价回复
在筛选出最佳广告，需要把广告素材、追踪代码(Tracking Code)等渲染成网页代码，然后把网页代码、定价等等按照格式生成竞价回复，送返给 ADX。

我们以 OpenRTB v2.4 的竞价回复为例，如下图。

```json
{
 "seatbid":[{
    "bid":[{
        "nurl":"http://reports.ubimo.com/fb?b=JdZQFdbCARgKMURHWGhvUVl0bSMBJeAhAA&c=MTo6&wp=${AUCTION_PRICE}",
        "crid":"12459",
        "adomain":["academy.com"],
        "price":2.93,
        "id":"1DGXhoQYtm",
        "adm":
            "<ad xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="smaato_ad_v0.9.xsd" modelVersion="0.9">
                <imageAd>
                    <clickUrl>http://reports.ubimo.com/fb?b=JdZQFdbCARgKMURHWGhvUVl0bSMBJeAhAA&amp;c=Mzo6&amp;t=https%3A%2F%2Fad.doubleclick.net%2Fddm%2Fclk%2F292804678%3B119963336%3Bw%3Fhttp%3A%2F%2Fwww.academy.com%2Fwebapp%2Fwcs%2Fstores%2Fservlet%2FContainer_10151_10051_-1_%3Fname%3DOfficial_Rules%26uv%3Dvanity%3Aofficialrules
                    </clickUrl>
                    <imgUrl>http://static.ubimo.com/io/603/ecd01dce
                    </imgUrl>
                    <height>50</height>
                    <width>320</width>
                    <beacons>
                        <beacon>http://reports.ubimo.com/fb?b=JdZQFdbCARgKMURHWGhvUVl0bSMBJeAhAA&amp;c=Mjo6</beacon>
                        <beacon>https://ad.doubleclick.net/ddm/ad/N5865.276855.MOBILEFUSE/B8852634.119963336;sz=1x1;ord=1436319256367</beacon>
                    </beacons>
                </imageAd>
            </ad>",
        "impid":"1",
        "cid":5163
    }]
 }],
 "id":"1DGXhoQYtm"
}
```

用请求数据流串联上上述所有的步骤，便如下图所示
![](https://res.cloudinary.com/darrenxyli/image/upload/v1532982134/E5_B9_BF_E5_91_8A_E9_9C_80_E6_B1_82_E6_96_B9_E5_B9_B3_E5_8F_B0-DSP-_E7_AB_9E_E4_BB_B7_E6_B5_81_E7_A8_8B.png)


