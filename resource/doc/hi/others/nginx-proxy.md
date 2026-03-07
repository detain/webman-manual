# Nginx प्रॉक्सी

जब webman को बाहरी नेटवर्क से सीधी पहुँच प्रदान करने की आवश्यकता होती है, तो webman के सामने Nginx प्रॉक्सी जोड़ने की सिफारिश की जाती है। इसके निम्नलिखित लाभ हैं:

- स्थिर संसाधनों को Nginx द्वारा संसाधित किया जाता है, webman व्यावसायिक तर्क पर ध्यान केंद्रित कर सकता है
- कई webman उदाहरण 80 और 443 पोर्ट साझा कर सकते हैं, डोमेन नाम से साइटों को अलग कर सकते हैं, एक सर्वर पर कई साइटें चलाने की अनुमति देते हैं
- php-fpm और webman आर्किटेक्चर को साथ काम करने की अनुमति देता है
- HTTPS के लिए Nginx प्रॉक्सी SSL सरल और अधिक कुशल है
- बाहरी नेटवर्क से अवैध अनुरोधों को सख्ती से फ़िल्टर कर सकता है

## Nginx प्रॉक्सी उदाहरण
```
upstream webman {
    server 127.0.0.1:8787;
    keepalive 10240;
}

server {
  server_name site_domain;
  listen 80;
  access_log off;
  # महत्वपूर्ण: root webman के नीचे public निर्देशिका को इंगित करना चाहिए, webman रूट निर्देशिका नहीं
  root /your/webman/public;

  location / {
    try_files $uri @proxy;
  }

  location @proxy {
    proxy_set_header Host $http_host;
    proxy_set_header X-Forwarded-For $remote_addr;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_http_version 1.1;
    proxy_set_header Connection "";
    proxy_pass http://webman;
  }

  # .php से समाप्त होने वाली सभी फ़ाइलों तक पहुँच अस्वीकृत
  location ~ \.php$ {
      return 404;
  }

  # .well-known निर्देशिका तक पहुँच की अनुमति
  location ~ ^/\.well-known/ {
    allow all;
  }

  # . से शुरू होने वाली सभी फ़ाइलों या निर्देशिकाओं तक पहुँच अस्वीकृत
  location ~ /\. {
      return 404;
  }

}
```

आम तौर पर, डेवलपर्स को केवल server_name और root को वास्तविक मानों के साथ कॉन्फ़िगर करने की आवश्यकता होती है; अन्य फ़ील्ड को कॉन्फ़िगर करने की आवश्यकता नहीं है।

> **ध्यान दें**
> विशेष रूप से महत्वपूर्ण: root विकल्प को webman के नीचे public निर्देशिका को इंगित करना चाहिए। इसे कभी भी webman रूट निर्देशिका पर सेट न करें, अन्यथा डेटाबेस कॉन्फ़िगरेशन जैसी संवेदनशील फ़ाइलों सहित आपकी सभी फ़ाइलें इंटरनेट से डाउनलोड और पहुँच योग्य हो सकती हैं।
