---
title: "OnePlus OxygenOS built-in analytics"
layout: page
permalink: /post/oneplus-analytics/
excerpt: We take a look at the analytics built into the OxygenOS, the flavour of Android built by phone manufacturer OnePlus.
---

---

#### Update 2017-10-10
_After gaining some traction online, Twitter user [@JaCzekanski](https://twitter.com/JaCzekanski) pointed out that there is a way to remove the OnePlus Device Manager app via adb, without requiring root (substitute `net.oneplus.odm` for `pkg`)_

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr"><a href="https://twitter.com/chrisdcmoore?ref_src=twsrc%5Etfw">@chrisdcmoore</a> I&#39;ve read your article about OnePlus Analytics. Actually, you can disable it permanently: pm uninstall -k --user 0 pkg</p>&mdash; Jakub Czekański (@JaCzekanski) <a href="https://twitter.com/JaCzekanski/status/917691128807395328?ref_src=twsrc%5Etfw">October 10, 2017</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

---


Whilst completing the [SANS Holiday Hack Challenge 2016](/post/sans-holiday-hack-challenge-2016/), I had cause to proxy the internet traffic from my phone, a [OnePlus 2](https://oneplus.net/2), through [OWASP ZAP](https://www.owasp.org/index.php/OWASP_Zed_Attack_Proxy_Project), a security tool for attacking web applications.

Amidst the traffic, I noticed requests to a domain which I'd not seen before, `open.oneplus.net`, and decided to examine them a little closer.

[![open.oneplus.net proxied traffic](/images/zap-oneplus-1.png)](/images/zap-oneplus-1.png)

Our first question is what am I connecting to at `open.oneplus.net`.
Obviously the top level domain `oneplus.net` belongs to the manufacturer of the device, but what's with the `open` bit?
Doing a DNS lookup, we can find that this points to an [Amazon AWS](https://aws.amazon.com) instance with mention of [Apache Hadoop](https://hadoor.apache.org/) in the record, located in the `us-east-1` region.

So the next question is what is being sent here?
From the example screenshot, we see two requests being sent over HTTPS; the first (not pictured) sending authentication information to `/oauth/token` and the second, more interesting request, to `/cloud/pushdata/` with two parameters; `access_token` which was the OAuth token returned from the first request, and `data` which appears to be Base64 encoded.

Decoding the Base64 parameter gives us some JSON, show below (formatting mine)

{% highlight json %}
{
    "ty": 3,
    "dl": [
        {
            "id": "258cfeb1",
            "en": "screen_off",
            "ts": 1484177517017,
            "oed": [],
            "it": 0,
            "rv": "OnePlus2Oxygen_14.A.27_GLO_027_1612271635"
        }, {
            "id": "258cfeb1",
            "en": "screen_on",
            "ts": 1484177826984,
            "oed": [],
            "it": 0,
            "rv": "OnePlus2Oxygen_14.A.27_GLO_027_1612271635"
        }, {
            "id": "258cfeb1",
            "en": "unlock",
            "ts": 1484177827961,
            "oed": [],
            "it": 0,
            "rv": "OnePlus2Oxygen_14.A.27_GLO_027_1612271635"
        }, {
            "id": "258cfeb1",
            "en": "abnormal_reboot",
            "ts": 1484178427035,
            "oed": [],
            "it": 0,
            "rv": "OnePlus2Oxygen_14.A.27_GLO_027_1612271635"
        }, ...
    ]
}
{% endhighlight %}

OK, so it looks like they're collecting timestamped (the `ts` field is the event time in milliseconds since unix epoch, which we'll be seeing more of) metrics on certain events, some of which I understand - from a development point of view, wanting to know about abnormal reboots seems legitimate - but the screen on/off and unlock activities feel excessive.
At least these are anonymised, right? Well, not really - taking a closer look at the ID field, it seems familiar; this is my phone's serial number.
This I'm less enthusiastic about, as this can be used by OnePlus to tie these events back to me personally (but only because I bought the handset directly from them, I suppose).

I leave the traffic proxied for some time, to see what other information is collected, and boy am I in for a shock...

{% highlight json %}
{
    "ty": 1,
    "dl": [
        {
            "ac": "",
            "av": "6.0.1",
            "bl": 82,
            "br": "OnePlus",
            "bs": "CHARGING",
            "co": "GB",
            "ga": 11511,
            "gc": 234,
            "ge": 6759424,
            "gn": 30,
            "iac": 1,
            "id": "258cfeb1",
            "im": "123456789012345,987654321098765",
            "imei1": "123456789012345",
            "it": 0,
            "la": "en",
            "log": "",
            "ma": "aa:bb:cc:dd:ee:ff",
            "mdmv": "1.06.160427",
            "mn": "ONE A2003",
            "nci": "23430,",
            "ncn": ",",
            "noi": "23430,",
            "non": "EE,",
            "not": "LTE,",
            "npc": "gb,",
            "npn": "07123456789,07987654321",
            "nwa": "aa:bb:cc:dd:ee:ff",
            "nwb": "ff:ee:dd:cc:bb:aa",
            "nwh": false,
            "nwl": 0,
            "nws": "\"CHRISDCMOORE\"",
            "ov": "Oxygen ONE A2003_24_161227",
            "pcba": "",
            "rh": 1920,
            "ro": false,
            "romv": "3.5.6",
            "rw": 1080,
            "sov": "A.27",
            "ts": 1484487017633,
            "tz": "GMT+0000"
        }
    ]
}
{% endhighlight %}

Amongst other things, this time we have the phone's IMEI(s), phone numbers, MAC addresses, mobile network(s) names and IMSI prefixes, as well as my wireless network ESSID and BSSID and, of course, the phone's serial number.
Wow, that's quite a bit of information about my device, even more of which can be tied directly back to me by OnePlus and other entities.

It gets worse.

{% highlight json %}
{
    "ty": 4,
    "dl": [{
            "id": "258cfeb1",
            "pn": "com.Slack20003701",
            "pvc": "20003701",
            "tk": [
                [1484079940460, 1484079952177],
                [1484081525486, 1484081603191],
                [1484081603424, 1484081619211],
                ...
            ],
            "it": 0
        }, {
            "id": "258cfeb1",
            "pn": "com.microsoft.office.outlook170",
            "pvc": "170",
            "tk": [
                [1484084321735, 1484084333336],
                [1484084682578, 1484084683668],
                [1484084685843, 1484084688985],
                ...
            ],
            "it": 0
        }, ...
    ]
}
{% endhighlight %}

Those are timestamp ranges (again, unix epoch in milliseconds) of the when I opened and closed applications on my phone.
From this data we can see that on Tuesday, 10th Jan 2017, I had [Slack](https://slack.com/) open between `20:25:40 UTC` and `20:25:52 UTC`, and the Microsoft Outlook app open between `21:38:41 UTC` and `21:38:53 UTC`, to take just two examples, again stamped with my phone's serial number.

It gets _even worse_.

{% highlight json %}
{
    "ty": 2,
    "dl": [{
            "id": "258cfeb1",
            "pi": 12795,
            "si": "127951484342058637",
            "ts": 1484342058637,
            "pn": "com.android.chrome",
            "pvn": "55.0.2883.91",
            "pvc": 288309101,
            "cn": "ChromeTabbedActivity",
            "en": "start",
            "aed": [],
            "sa": true,
            "it": 0,
            "rv": "OnePlus2Oxygen_14.A.27_GLO_027_1612271635"
        }, ... {
            "id": "258cfeb1",
            "pi": 4143,
            "si": "41431484342115589",
            "ts": 1484342115589,
            "pn": "com.android.systemui",
            "pvn": "1.1.0",
            "pvc": 0,
            "cn": "RecentsActivity",
            "en": "stop",
            "aed": [],
            "sa": true,
            "it": 0,
            "rv": "OnePlus2Oxygen_14.A.27_GLO_027_1612271635"
        }, {
            "id": "258cfeb1",
            "pi": 26449,
            "si": "264491484342115620",
            "ts": 1484342115620,
            "pn": "com.android.settings",
            "pvn": "6.0.1",
            "pvc": 23,
            "cn": "WifiSettingsActivity",
            "en": "start",
            "aed": [],
            "sa": true,
            "it": 0,
            "rv": "OnePlus2Oxygen_14.A.27_GLO_027_1612271635"
        }, ... {
            "id": "258cfeb1",
            "pi": 2608,
            "si": "26081484346421908",
            "ts": 1484346421908,
            "pn": "com.android.settings",
            "pvn": "6.0.1",
            "pvc": 23,
            "cn": "Settings",
            "en": "start",
            "aed": [],
            "sa": true,
            "it": 0,
            "rv": "OnePlus2Oxygen_14.A.27_GLO_027_1612271635"
        }, ...
    ]
}
{% endhighlight %}

These event data contain timestamps of which [activities](https://developer.android.com/reference/android/app/Activity.html) were fired up in which in applications, again stamped with the phone's serial number.

I took to Twitter to ask OnePlus on Twitter how this could be turned off, which disappointingly led down the usual path of "troubleshooting" suggestions, before being met with radio silence:

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Hey <a href="https://twitter.com/OnePlus_Support">@OnePlus_Support</a>, it&#39;s none of your business when I turn my screen on/off or unlock my phone - how do I turn this off? /cc:<a href="https://twitter.com/troyhunt">@troyhunt</a> <a href="https://t.co/VihaIDI6wP">pic.twitter.com/VihaIDI6wP</a></p>&mdash; Christopher Moore (@chrisdcmoore) <a href="https://twitter.com/chrisdcmoore/status/819708963633541121">January 13, 2017</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr"><a href="https://twitter.com/chrisdcmoore">@chrisdcmoore</a> Try wiping out the cache.Turn off your device&gt;Power key + volume down&gt;English&gt;Wipe and cache&gt;Wipe Cache&gt;Confirm wipe&gt;Reboot.</p>&mdash; OnePlus Support (@OnePlus_Support) <a href="https://twitter.com/OnePlus_Support/status/819951791827611650">January 13, 2017</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr"><a href="https://twitter.com/chrisdcmoore">@chrisdcmoore</a> Alright. Please try doing a hard reset <a href="https://t.co/1qyq9XajiJ">https://t.co/1qyq9XajiJ</a> and see if there are improvements.</p>&mdash; OnePlus Support (@OnePlus_Support) <a href="https://twitter.com/OnePlus_Support/status/820033728596451329">January 13, 2017</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

A member of the community, [@VenomSarad](https://twitter.com/VenomSarad), who had noticed my tweets suggested that, even if they wanted to, OnePlus support were not allowed to suggest disabling applications, and that my time might be better spent looking on their forums:

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr"><a href="https://twitter.com/chrisdcmoore">@chrisdcmoore</a> <a href="https://twitter.com/OnePlus_Support">@OnePlus_Support</a> The support team isn&#39;t allowed tell people to disable apps. You may want to go to OnePlus forum for this</p>&mdash; Sarad (@VenomSarad) <a href="https://twitter.com/VenomSarad/status/820070636647317504">January 14, 2017</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

I did some searching for any other mentions of this analytics data collection, and came across a few forum posts of varying relevance, the closest being [this one](https://forums.oneplus.net/threads/android-uid-system-does-requests-to-open-oneplus-net.472803/), as well as a [thread on Reddit](https://www.reddit.com/r/oneplus/comments/4t20ri/oxygenos_reports_back_tons_of_data_with/) based off of a tweet from July 2016 rather closely mirroring my own sentiments:

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr"><a href="https://twitter.com/oneplus">@oneplus</a> Why are you collecting timestamps of when I unlock my phone, and when the screen turns on/off? <a href="https://twitter.com/hashtag/caught?src=hash">#caught</a><a href="https://t.co/ejt4p9uPFn">https://t.co/ejt4p9uPFn</a></p>&mdash; Tux (@__Tux) <a href="https://twitter.com/__Tux/status/754085708843786240">July 15, 2016</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

Reading through the Reddit thread, we learn that the code responsible for this data collection is part of the OnePlus Device Manager and the OnePlus Device Manager Provider, which run the `OneplusAnalyticsJobService` under the `OnePlus System Service`.
In my case, these services had sent 16MB of data in approximately 10 hours.

[![oneplus system service running services](/images/oneplus-services.png)](/images/oneplus-services-orig.png)

Using `pm` to locate the application package files, we find that it is located at `/system/priv-app/OPDeviceManager/OPDeviceManager.apk`.
Grabbing the APK and extracting it using `apktool` gives us the manifest and some resources, but no bytecode - this is because, as a system application, it has been optimised, so the `classes.dex` file has been removed from the APK archive, optimised into an architecture-specific `.odex` file and placed at, in my case, `/system/priv-app/OPDeviceManager/oat/arm64/OPDeviceManager.pdex`.

Running this, in combination with `boot.oat`, through [baksmali](https://github.com/JesusFreke/smali) gives us the bytecode for further analysis.
The OnePlus Device Manager (OPDM) which drives the Oneplus System Service, utilises a bunch of libraries - some expected, given the data we've seen, and others less so - such as `com.google.gson` and `com.squareup.okhttp` for serialisation and making requests, but also namespaces which imply geolocation functionality such as `com.amap.api`, `com.autonavi.aps.amapapi`.

Here's a list of the public methods in `net/oneplus/odm/common/Utils.smali`, just to give us a good idea for some of the breadth of this functionality, and an indication of some of the kinds of data it might collate:

```
.method public static encodeToBase64(Ljava/lang/String;)Ljava/lang/String;
.method public static getAndroidVersion()Ljava/lang/String;
.method public static getBSSID(Landroid/content/Context;)Ljava/lang/String;
.method public static getBatteryLevel(Landroid/content/Context;)F
.method public static getBatteryStatus(Landroid/content/Context;)Ljava/lang/String;
.method public static getBrandName()Ljava/lang/String;
.method public static getCellSignalLevel(Landroid/content/Context;)Ljava/lang/String;
.method public static getDeviceId()Ljava/lang/String;
.method public static getIMEI(Landroid/content/Context;)Ljava/lang/String;
.method public static getIMEI1(Landroid/content/Context;)Ljava/lang/String;
.method public static getIsHiddenSSID(Landroid/content/Context;)Z
.method public static getLocale(Landroid/content/Context;)Ljava/util/Locale;
.method public static getMacAddr(Landroid/content/Context;)Ljava/lang/String;
.method public static getModelName()Ljava/lang/String;
.method public static getOSVersion()Ljava/lang/String;
.method public static getPCBA()Ljava/lang/String;
.method public static getResolutionHeight(Landroid/content/Context;)I
.method public static getResolutionWidth(Landroid/content/Context;)I
.method public static getRomVersion()Ljava/lang/String;
.method public static getSimCountryCode(Landroid/content/Context;)Ljava/lang/String;
.method public static getSoftVersion()Ljava/lang/String;
.method public static getTimezone()Ljava/lang/String;
.method public static getWifiMacAddress(Landroid/content/Context;)Ljava/lang/String;
.method public static getWifiSSID(Landroid/content/Context;)Ljava/lang/String;
.method public static getWifiSignalLevel(Landroid/content/Context;)I
.method public static isH2()Z
.method public static isO2()Z
.method public static isRooted()Z
```

Unfortunately, as a system service, there doesn't appear to be any way of permanently disabling this data collection or removing this functionality without rooting the phone.
One alternative would be to stop the service every time you boot your phone (assuming it doesn't get periodically restarted) or using an app to achieve the same effect, or perhaps prevent communication with `open.oneplus.net` somehow.

This kind of data collection, especially one containing information that can be directly tied back to me as an individual, should really be opt-in and/or have an easily accessible off switch...
