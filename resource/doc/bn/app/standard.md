# অ্যাপ্লিকেশন প্লাগইন ডেভেলপমেন্ট গাইড

## অ্যাপ্লিকেশন প্লাগইন প্রয়োজনীয়তা
* প্লাগইন কোনো অধিকারও বা চিত্রও ভূল কোড বা চিত্র, ছবি ইত্যাদি থাকতে পারবে না
* প্লাগইন সোর্স কোডটি পূর্ণভাবে অপেক্ষা করে, এবং সংকেতে হতে পারবে না
* প্লাগইনটি পূর্ণভাবে কার্যকর হতে হবে, এটি সাধারণ কার্য নয়
* পূর্ণভাবে প্রাপ্ত ফাংশনালিটি এবং নথিটি অবশ্যই সরবরাহ করতে হবে
* প্লাগইনে উপনগর বা উপনগর অন্তর্ভুক্ত থাকতে পারবে না
* প্লাগইনে কোনো পাঠ বা প্রচারের লিংক থাকবে না

## অ্যাপ্লিকেশন প্লাগইন পরিচিতি
প্রতিটি অ্যাপ্লিকেশন প্লাগইনের একটি অদ্যাবধিক পরিচিতি আছে, এই পরিচিতিটি আলফাবেটিক অক্ষর থেকে বিচার করে। এই পরিচিতির আশরু হওয়া অনুরোধ করা হয়, এবং আপনার প্রয়োজনে, একটি জিজ্ঞাসা করুন [অ্যাপলিকেশন অভিযান নিশ্চিতকরণ](https://www.workerman.net/app/check)।

## ডাটাবেস
* টেবিল নাম ডানিক অক্ষর "a-z" এবং আন্ডারস্কোর "_" দিয়ে তৈরি হওয়া উচিত
* প্লাগইন ডেটাবেস টেবিলের প্রিফিক্স প্লাগইন পরিচ্ছেয়ে হতে হবে, উদাহরণস্বরূপ, ফু প্লাগইনের অ্যার্টিকেল টেবিল হল `foo_article`
* টেবিলের প্রাথমিক কী হেব ইডি হবে
* সংরক্ষণ যন্ত্র সাধারণ ভাবে ইন্নোদ্ব ইঞ্জিন ব্যবহার করা উচিত
* অক্ষরণ খনন সাধারণভাবে utf8mb4 জনারাল_ci ব্যবহার করা উচিত
* ডাটাবেস ORM হারাভেল বা থিং-ওআরএম ব্যবহার করা যেতে পারে
* সময় ক্ষেত্রে DateTime ব্যবহারকারীগণকে পরামর্শ দেওয়া হয়

## কোড পরিষ্কার
#### PSR পরিষ্কার
কোডটি পিএসআর 4 লোডিং পরিষ্কার করা উচিত

#### ক্লাস নাম বিশেষভাবে বড় হাতের কম্পাসিট নাম বুঝা উচিত
```php
<?php

namespace plugin\foo\app\controller;

class ArticleController
{
    
}
```

#### ক্লাস গুলোর সংপত্তি এবং পদক্ষেপ ছোট হাতের কম্পাসিট নাম বুঝাউচিত
```php
<?php

namespace plugin\foo\app\controller;

class ArticleController
{
    /**
     * অনুমতি প্রয়োজন নেই
     * @var array
     */
    protected $noNeedAuth = ['getComments'];
    
    /**
     * মন্তব্য পেতে
     * @param অনুরোধ $রিকোয়েস্ট
     * @return জবাব
     * @throws বিজনেস এক্সিপশ্যান
     */
    public function getComments(Request $request): Response
    {
        
    }
}
```

#### মন্তব্য
ক্লাস এর অ্যাট্রিবিউট এবং ফাংশন এক প্রতিটি মন্তব্য অ্বশ্যই থাকতে আবশ্যক, যেমন বিবরণ, প্যারামিটার, প্রাপ্য প্রকার

#### অবন্ধন
কোডটি 4 টি স্পেস ব্যবহার করে ইন্ডেন্টিং করা উচিত এবং ট্যাবের ব্যবহার না করা উচিত

#### প্রক্রিয়া নিয়ন্ত্রণ
প্রক্রিয়া নিয়ন্ত্রন কীওয়ার্ড (যদি জন্য হোয়াইল ছাড়া if for while foreach ইত্যাদি) পরে একটি স্পেস অনুসরণ করবে, এবং প্রক্রিয়া নিয়ন্ত্রন কোডে ব্র্যাকেট শুরু হবে এবং শেষ ব্র্যাকেট একই লাইনে থাকবে।
```php
foreach ($users as $uid => $user) {

}
```

#### অস্থায়ী ভ্্ার নাম
প্রস্থায়ী ভ্্ার অবশ্যই ছোট হাতে কম্পাসিট নাম বড় হাতে শুরু হতে শুরু করতে পারে (প্রয়োজন নেই)
```php
$articleCount = 100;
```