---
author: boonyachart
layout: post
featimg: https://cloud.githubusercontent.com/assets/3109773/9676885/623ed0b2-52fc-11e5-96e7-c71ecfaff390.png
title: AWS Mobile Push Notifications
date: 2015-09-03 18:00:00
tags: aws push-notification
draft: false
---

การส่ง Push notification ช่วยให้สามารถส่งข่าวสารต่างๆ ไปยังลูกค้าได้อย่างรวดเร็ว ปัจจุบัน PrimeTime มีแอพมือถืออยู่ 2 แพลทฟอร์ม iOS และ Android การส่งข้อความไปทั้ง 2 แพลทฟอร์มก็ต้องจัดการต่างกัน และเนื่องจากเราใช้บริการของ AWS อยู่แล้ว ตัว AWS นั้นก็มีบริการที่เข้ามาช่วยในการส่ง Push ให้เราจึงเลือกใช้บริการนี้มาจัดการให้

### Amazon SNS (Amazon Simple Notification Service)

เป็นบริการส่งข้อความของทาง AWS ซึ่งไม่ใช่ส่งแค่ push notification แต่ยังสามารถส่งไปยังช่องทางอื่นๆ เช่น email, SMS เป็นต้น หลักการของ SNS นั้นคือให้เราสร้าง Topic แล้ว subscribe แต่ละ device ไปที่ Topic นั้น จากนั้นเมื่อเราจะส่งค่าไปยัง device ต่างๆ ก็เพียงแค่ส่ง message ไปยัง Topic เดียว แล้วทุก device ที่ subscribe ไว้ก็จะได้รับข้อความกันทั้งหมดโดยไม่ต้องเสียพลังงานบน server ของเราเอง วน loop หา device เลย

![](https://cloud.githubusercontent.com/assets/3109773/9676885/623ed0b2-52fc-11e5-96e7-c71ecfaff390.png)

### How we use it

ในฝั่งของ SNS นั้นเราต้องเข้าไปสร้างสิ่งเหล่านี้เตรียมไว้

1. Application แยกออกเป็น 2 ตัว คือ Android โดยใช้ API Key ของแอพ และ iOS โดยใช้ P12 ไฟล์
1. Topic สำหรับให้ device มา subscribe ไว้ หากจะแยกกรุ๊ปก็สามารถสร้างหลายๆ Topic ได้

![](https://cloud.githubusercontent.com/assets/3109773/9657066/0c6e79b0-5268-11e5-990f-d6d2c6a7be46.png)

เมื่อ client ลงทะเบียนกับ Apple Push Notification Service (APNS) หรือ Google Cloud Messaging for Android (GCM)  สำเร็จแล้วก็ส่งค่า Device token จาก APNS หรือ Registration ID จาก GCM มายัง server จากนั้น server จะนำ token/id มา register กับ Application บน SNS จะได้เป็น Endpoint และก็นำ Endpoint ไป subscribe กับ Topic ของ SNS

จากนั้นหากจะส่งข้อความไปยัง Device ก็สามารถส่งไปยัง Endpoint บน SNS ได้เลย ส่วนถ้าจะส่งแบบ Broadcast ไปหลายเครื่องก็ส่งไปที่ Topic ซึ่งทุก Endpoint ที่ subscribe ไว้กับ Topic นี้จะได้รับ message เหมือนกันทั้งหมด การส่ง Push notification ผ่าน SNS ช่วยให้เราประหยัดทรัพยากรไปได้มาก โดยไม่ต้องกังวลกับจำนวน Device ที่เพิ่มขึ้น แต่อย่างไรก็ตาม เราก็ยังพบปัญหาบางประการอยู่..

### The problem

เมื่อ SNS ไม่สามารถส่งข้อความไปหา Endpoint ได้ SNS จะทำการจำไว้ว่า Endpoint นี้ disable ไปแล้ว ซึ่งดูเหมือนจะเหมาะสมดีแล้ว แต่..มันไม่เป็นเช่นนั้น เมื่อบางครั้ง Endpoint ที่ disable นั้น ไม่ได้หายไปจริงๆ และที่ทำให้ปัญหานี้ดูวุ่นวายเพิ่มขึ้นอีกก็คือ SNS จะไม่ทำการตรวจซ้ำอีกเลยหลังจาก disable

สำหรับสาเหตุที่ Endpoint กลายเป็น disable นั้น อาจเป็น ไฟล์P12/API Key ไม่ถูกต้อง, ลูกค้าลบแอพไปแล้ว หรือ แอพถูกบล๊อค notification บนเครื่องนั้นๆ แต่บางครั้งก็ไม่พบสาเหตุใดๆ 

### The solution (more like workaround)

สิ่งที่ทำได้คือ ต้องทำการ enable เอง ซึ่งก็ต้องทำทีละ Endpoint ไป สำหรับการส่ง Direct message ไปยัง endpoint เดียวนั้น การ enable ก่อนส่งนั้นไม่วุ่นวายนัก แต่สำหรับ Broadcast message ที่เดิมมีประโยชน์ตรงเราไม่ต้อง loop ไปยังแต่ละ Device หากเราต้องมา loop ทุก Endpoint แล้ว enable ทีละตัวก่อนส่งคงไม่ใช่วิธีที่ดีนัก การทำเป็น Schedule แล้ว enable น่าจะเป็นวิธีที่เหมาะสมกว่า

### Future improvement

จาก [Document](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/sns/model/SetEndpointAttributesRequest.html) ของ AWS พูดถึง Enable ไว้ว่า 
> Enabled -- flag that enables/disables delivery to the endpoint. Message Processor will set this to false when a notification service indicates to SNS that the endpoint is invalid. Users can set it back to true, typically after updating Token.

ซึ่ง notification service ในที่นี้ก็คือ APNS/GCM ซึ่งก็ต้องไปหาสาเหตุกันอีกทีว่าทำไมถึงบอก SNS ว่า endpoint invalid ไปแล้ว

การไป enable ตัว Endpoint เองทุกๆตัวนั้นโดยไม่รู้ว่า Endpoint นั้นเป็นตัวที่ disable อย่างถาวรไปแล้วจริงหรือไม่ ค่อนข้างเปลืองพลังมาก หากสามารถรู้ได้ก็จะดีกว่านี้ ซึ่ง AWS มี feature ใหม่คือ [Delivery status](https://mobile.awsblog.com/post/TxHTXGC8711JNF/Using-the-Delivery-Status-feature-of-Amazon-SNS) ที่น่าจะบอกสาเหตุได้มากกว่านี้ว่าทำไมไม่ส่งข้อความไปยังแอพได้

### Conclusion

การเลือกใช้ AWS SNS ช่วยให้สามารถส่ง Push notification ไปยังแอพนั้นง่ายขึ้นมากและยัง scale ไปได้อย่างดีเมื่อมี device เพิ่มขึ้น แต่ก็ยังพบปัญหาตามที่กล่าวไปข้างต้นซึ่งก็ต้องทำการปรับปรุงกันต่อไป
