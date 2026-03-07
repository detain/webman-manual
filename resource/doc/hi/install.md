# webman कैसे इंस्टॉल करें

* PHP >= 8.1
* [Composer](https://getcomposer.org/) >= 2.0


## Linux: PHP + Composer वातावरण इंस्टॉल करें (पहले से है तो छोड़ें)
```
curl -sO https://www.workerman.net/install-php-and-composer && sudo bash install-php-and-composer
```
> **ध्यान दें**
> उपरोक्त कमांड Linux और macOS के लिए है। Windows उपयोगकर्ताओं को अलग से PHP इंस्टॉल करना होगा।

आप webman द्वारा प्रदान किए गए [स्टैटिक PHP](https://www.workerman.net/download) को मैन्युअल रूप से डाउनलोड करके एक्सट्रैक्ट करके भी उपयोग कर सकते हैं।

## 1. परियोजना बनाएं

```php
composer create-project workerman/webman:~2.0
```

> **सुझाव**
> एरर आने पर आप खराब Composer मिरर उपयोग कर रहे होंगे। प्रॉक्सी हटाने के लिए `composer config -g --unset repos.packagist` चलाएं।

## 2. चलाना

webman निर्देशिका में जाएं

#### Windows उपयोगकर्ता
शुरू करने के लिए `windows.bat` पर दो बार क्लिक करें या `php windows.php` चलाएं

> **सुझाव**
> एरर आने पर फ़ंक्शन निष्क्रिय हो सकते हैं। निष्क्रियता हटाने के लिए [फ़ंक्शन अवरुद्ध जांच](others/disable-function-check.md) देखें।

#### Linux उपयोगकर्ता
**डिबग मोड** (विकास के लिए: आउटपुट टर्मिनल में दिखता है; टर्मिनल बंद होने पर webman सेवा रुक जाती है)

```php
php start.php start
```

**डेमॉन मोड** (प्रोडक्शन के लिए: टर्मिनल में आउटपुट नहीं दिखता; टर्मिनल बंद के बाद भी webman सेवा चलती रहती है)

```php
php start.php start -d
```

#### Docker उपयोगकर्ता

सभी सेवाएं शुरू करें और कंसोल से जोड़ें
```php
docker-compose up
```

बैकग्राउंड मोड में सेवाएं चलाएं
```php
docker-compose up -d
```

> **सुझाव**
> एरर आने पर फ़ंक्शन निष्क्रिय हो सकते हैं। निष्क्रियता हटाने के लिए [फ़ंक्शन अवरुद्ध जांच](others/disable-function-check.md) देखें।

## 3. पहुंच

ब्राउज़र में `http://ip-पता:8787` खोलें।
