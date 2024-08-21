# macOS Keychain

{% hint style="success" %}
Learn & practice AWS Hacking:<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">\
Learn & practice GCP Hacking: <img src="../../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="../../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Check the [**subscription plans**](https://github.com/sponsors/carlospolop)!
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Share hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) एक **डार्क-वेब** द्वारा संचालित सर्च इंजन है जो **मुफ्त** कार्यक्षमताएँ प्रदान करता है ताकि यह जांचा जा सके कि क्या कोई कंपनी या इसके ग्राहक **सुरक्षित** हैं या नहीं **चोरी करने वाले मालवेयर** द्वारा।

WhiteIntel का प्राथमिक लक्ष्य जानकारी चुराने वाले मालवेयर के परिणामस्वरूप खाता अधिग्रहण और रैनसमवेयर हमलों से लड़ना है।

आप उनकी वेबसाइट पर जा सकते हैं और **मुफ्त** में उनके इंजन का प्रयास कर सकते हैं:

{% embed url="https://whiteintel.io" %}

***

## मुख्य कीचेन

* **यूजर कीचेन** (`~/Library/Keychains/login.keycahin-db`), जिसका उपयोग **उपयोगकर्ता-विशिष्ट क्रेडेंशियल्स** जैसे एप्लिकेशन पासवर्ड, इंटरनेट पासवर्ड, उपयोगकर्ता-निर्मित प्रमाणपत्र, नेटवर्क पासवर्ड, और उपयोगकर्ता-निर्मित सार्वजनिक/निजी कुंजियों को स्टोर करने के लिए किया जाता है।
* **सिस्टम कीचेन** (`/Library/Keychains/System.keychain`), जो **सिस्टम-व्यापी क्रेडेंशियल्स** जैसे WiFi पासवर्ड, सिस्टम रूट प्रमाणपत्र, सिस्टम निजी कुंजियाँ, और सिस्टम एप्लिकेशन पासवर्ड को स्टोर करता है।

### पासवर्ड कीचेन एक्सेस

ये फ़ाइलें, जबकि इनमें अंतर्निहित सुरक्षा नहीं है और इन्हें **डाउनलोड** किया जा सकता है, एन्क्रिप्टेड हैं और इन्हें **डिक्रिप्ट** करने के लिए **उपयोगकर्ता का प्लेनटेक्स्ट पासवर्ड** की आवश्यकता होती है। डिक्रिप्शन के लिए [**Chainbreaker**](https://github.com/n0fate/chainbreaker) जैसे टूल का उपयोग किया जा सकता है।

## कीचेन प्रविष्टियाँ सुरक्षा

### ACLs

कीचेन में प्रत्येक प्रविष्टि **एक्सेस कंट्रोल सूचियों (ACLs)** द्वारा शासित होती है जो यह निर्धारित करती है कि कौन कीचेन प्रविष्टि पर विभिन्न क्रियाएँ कर सकता है, जिसमें शामिल हैं:

* **ACLAuhtorizationExportClear**: धारक को रहस्य का स्पष्ट पाठ प्राप्त करने की अनुमति देता है।
* **ACLAuhtorizationExportWrapped**: धारक को दूसरे प्रदान किए गए पासवर्ड के साथ एन्क्रिप्टेड स्पष्ट पाठ प्राप्त करने की अनुमति देता है।
* **ACLAuhtorizationAny**: धारक को कोई भी क्रिया करने की अनुमति देता है।

ACLs के साथ एक **विश्वसनीय अनुप्रयोगों की सूची** होती है जो बिना प्रॉम्प्ट के ये क्रियाएँ कर सकती हैं। यह हो सकता है:

* **N`il`** (कोई प्राधिकरण आवश्यक नहीं, **सभी विश्वसनीय हैं**)
* एक **खाली** सूची (**कोई भी** विश्वसनीय नहीं है)
* **विशिष्ट** **अनुप्रयोगों** की **सूची**।

इसके अलावा, प्रविष्टि में कुंजी **`ACLAuthorizationPartitionID`** हो सकती है, जिसका उपयोग **teamid, apple,** और **cdhash** की पहचान के लिए किया जाता है।

* यदि **teamid** निर्दिष्ट है, तो **प्रविष्टि** मान **बिना** **प्रॉम्प्ट** के **एक्सेस** करने के लिए उपयोग किए जाने वाले अनुप्रयोग का **समान teamid** होना चाहिए।
* यदि **apple** निर्दिष्ट है, तो ऐप को **Apple** द्वारा **साइन** किया जाना चाहिए।
* यदि **cdhash** निर्दिष्ट है, तो **ऐप** को विशिष्ट **cdhash** होना चाहिए।

### कीचेन प्रविष्टि बनाना

जब **`Keychain Access.app`** का उपयोग करके एक **नया** **प्रविष्टि** बनाई जाती है, तो निम्नलिखित नियम लागू होते हैं:

* सभी ऐप्स एन्क्रिप्ट कर सकते हैं।
* **कोई ऐप्स** निर्यात/डिक्रिप्ट नहीं कर सकते (उपयोगकर्ता को प्रॉम्प्ट किए बिना)।
* सभी ऐप्स इंटीग्रिटी चेक देख सकते हैं।
* कोई ऐप्स ACLs को नहीं बदल सकते।
* **partitionID** को **`apple`** पर सेट किया गया है।

जब एक **अनुप्रयोग कीचेन में एक प्रविष्टि बनाता है**, तो नियम थोड़े अलग होते हैं:

* सभी ऐप्स एन्क्रिप्ट कर सकते हैं।
* केवल **बनाने वाला अनुप्रयोग** (या कोई अन्य ऐप्स जो स्पष्ट रूप से जोड़े गए हैं) निर्यात/डिक्रिप्ट कर सकते हैं (उपयोगकर्ता को प्रॉम्प्ट किए बिना)।
* सभी ऐप्स इंटीग्रिटी चेक देख सकते हैं।
* कोई ऐप्स ACLs को नहीं बदल सकते।
* **partitionID** को **`teamid:[teamID here]`** पर सेट किया गया है।

## कीचेन तक पहुँच

### `security`
```bash
# List keychains
security list-keychains

# Dump all metadata and decrypted secrets (a lot of pop-ups)
security dump-keychain -a -d

# Find generic password for the "Slack" account and print the secrets
security find-generic-password -a "Slack" -g

# Change the specified entrys PartitionID entry
security set-generic-password-parition-list -s "test service" -a "test acount" -S

# Dump specifically the user keychain
security dump-keychain ~/Library/Keychains/login.keychain-db
```
### APIs

{% hint style="success" %}
**कीचेन एन्यूमरेशन और सीक्रेट्स का डंपिंग** जो **प्रॉम्प्ट नहीं जनरेट करेगा** उसे [**LockSmith**](https://github.com/its-a-feature/LockSmith) टूल से किया जा सकता है।
{% endhint %}

प्रत्येक कीचेन एंट्री के बारे में **जानकारी** सूचीबद्ध करें और प्राप्त करें:

* API **`SecItemCopyMatching`** प्रत्येक एंट्री के बारे में जानकारी देता है और इसका उपयोग करते समय कुछ विशेषताएँ सेट की जा सकती हैं:
* **`kSecReturnData`**: यदि सत्य है, तो यह डेटा को डिक्रिप्ट करने की कोशिश करेगा (संभावित पॉप-अप से बचने के लिए इसे गलत सेट करें)
* **`kSecReturnRef`**: कीचेन आइटम का संदर्भ भी प्राप्त करें (यदि बाद में आप देखते हैं कि आप बिना पॉप-अप के डिक्रिप्ट कर सकते हैं तो इसे सत्य पर सेट करें)
* **`kSecReturnAttributes`**: एंट्रीज़ के बारे में मेटाडेटा प्राप्त करें
* **`kSecMatchLimit`**: कितने परिणाम लौटाने हैं
* **`kSecClass`**: किस प्रकार की कीचेन एंट्री

प्रत्येक एंट्री के **ACLs** प्राप्त करें:

* API **`SecAccessCopyACLList`** का उपयोग करके आप **कीचेन आइटम के लिए ACL** प्राप्त कर सकते हैं, और यह ACLs की एक सूची लौटाएगा (जैसे `ACLAuhtorizationExportClear` और अन्य पहले उल्लेखित) जहाँ प्रत्येक सूची में:
* विवरण
* **विश्वसनीय एप्लिकेशन सूची**। यह हो सकता है:
* एक ऐप: /Applications/Slack.app
* एक बाइनरी: /usr/libexec/airportd
* एक समूह: group://AirPort

डेटा निर्यात करें:

* API **`SecKeychainItemCopyContent`** प्लेनटेक्स्ट प्राप्त करता है
* API **`SecItemExport`** कुंजी और प्रमाणपत्रों को निर्यात करता है लेकिन सामग्री को एन्क्रिप्टेड निर्यात करने के लिए पासवर्ड सेट करने की आवश्यकता हो सकती है

और ये हैं **आवश्यकताएँ** ताकि आप **बिना प्रॉम्प्ट के एक सीक्रेट निर्यात कर सकें**:

* यदि **1+ विश्वसनीय** ऐप्स सूचीबद्ध हैं:
* उचित **अधिकार** की आवश्यकता है (**`Nil`**, या सीक्रेट जानकारी तक पहुँचने के लिए अनुमति सूची में **भाग** होना)
* **PartitionID** से मेल खाने के लिए कोड सिग्नेचर की आवश्यकता है
* एक **विश्वसनीय ऐप** का कोड सिग्नेचर मेल खाने की आवश्यकता है (या सही KeychainAccessGroup का सदस्य होना)
* यदि **सभी एप्लिकेशन विश्वसनीय** हैं:
* उचित **अधिकार** की आवश्यकता है
* **PartitionID** से मेल खाने के लिए कोड सिग्नेचर की आवश्यकता है
* यदि **कोई PartitionID नहीं है**, तो यह आवश्यक नहीं है

{% hint style="danger" %}
इसलिए, यदि **1 एप्लिकेशन सूचीबद्ध है**, तो आपको **उस एप्लिकेशन में कोड इंजेक्ट करने** की आवश्यकता है।

यदि **apple** **partitionID** में इंगित है, तो आप इसे **`osascript`** के साथ एक्सेस कर सकते हैं इसलिए कुछ भी जो partitionID में apple के साथ सभी एप्लिकेशनों पर भरोसा कर रहा है। इसके लिए **`Python`** का भी उपयोग किया जा सकता है।
{% endhint %}

### Two additional attributes

* **अदृश्य**: यह **UI** कीचेन ऐप से एंट्री को **छिपाने** के लिए एक बूलियन फ्लैग है
* **सामान्य**: यह **मेटाडेटा** को स्टोर करने के लिए है (इसलिए यह एन्क्रिप्टेड नहीं है)
* Microsoft सभी रिफ्रेश टोकन को संवेदनशील एंडपॉइंट तक पहुँचने के लिए प्लेन टेक्स्ट में स्टोर कर रहा था।

## References

* [**#OBTS v5.0: "Lock Picking the macOS Keychain" - Cody Thomas**](https://www.youtube.com/watch?v=jKE1ZW33JpY)

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) एक **डार्क-वेब** द्वारा संचालित सर्च इंजन है जो यह जांचने के लिए **मुफ्त** कार्यक्षमताएँ प्रदान करता है कि क्या कोई कंपनी या उसके ग्राहक **स्टीलर मैलवेयर** द्वारा **समझौता** किए गए हैं।

WhiteIntel का प्राथमिक लक्ष्य जानकारी-चोरी करने वाले मैलवेयर के परिणामस्वरूप अकाउंट टेकओवर और रैनसमवेयर हमलों से लड़ना है।

आप उनकी वेबसाइट देख सकते हैं और **मुफ्त** में उनके इंजन का प्रयास कर सकते हैं:

{% embed url="https://whiteintel.io" %}

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="../../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="../../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* **💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में शामिल हों या **Twitter** पर हमें **फॉलो** करें 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स साझा करें और** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PRs सबमिट करें।

</details>
{% endhint %}
