# Best practices for writing Dockerfiles -TH

  เอกสารนี้ครอบคลุมแนวทางปฏิบัติที่ดีที่สุดและวิธีการที่แนะนำสำหรับการสร้างภาพที่มีประสิทธิภาพ
 
 Docker สร้างภาพโดยอัตโนมัติโดยการอ่านคำแนะนำจาก Dockerfile - ไฟล์ข้อความที่มีคำสั่งทั้งหมดตามลำดับที่จำเป็นในการสร้างภาพที่กำหนด Dockerfile ยึดตามรูปแบบเฉพาะและชุดคำสั่งที่คุณสามารถหาได้ในการอ้างอิง Dockerfile
  
  อิมเมจ Docker ประกอบด้วยเลเยอร์แบบอ่านอย่างเดียวแต่ละอันแสดงถึงคำสั่ง Dockerfile เลเยอร์จะซ้อนกันและแต่ละชั้นเป็นเดลต้าของการเปลี่ยนแปลงจากเลเยอร์ก่อนหน้า พิจารณา Dockerfile นี้:
```js
FROM ubuntu:18.04
COPY . /app
RUN make /app
CMD python /app/app.py
```
แต่ละคำสั่งจะสร้างหนึ่งเลเยอร์:

* FROM สร้างเลเยอร์จากอิมเมจ ubuntu: 18.04
* COPY เพิ่มไฟล์จากไดเรกทอรีปัจจุบันของไคลเอ็นต์ Docker
* RUN สร้างแอปพลิเคชันของคุณด้วย make
* CMD ระบุคำสั่งที่จะเรียกใช้ภายในคอนเทนเนอร์

เมื่อคุณเรียกใช้รูปภาพและสร้างคอนเทนเนอร์คุณเพิ่มเลเยอร์ที่เขียนใหม่ได้ (“ เลเยอร์คอนเทนเนอร์”) ที่ด้านบนของเลเยอร์ที่ขีดเส้นใต้ การเปลี่ยนแปลงทั้งหมดที่เกิดขึ้นกับคอนเทนเนอร์ที่กำลังทำงานเช่นการเขียนไฟล์ใหม่การแก้ไขไฟล์ที่มีอยู่และการลบไฟล์จะถูกเขียนไปยังเลเยอร์คอนเทนเนอร์แบบเขียนได้บางนี้

สำหรับข้อมูลเพิ่มเติมเกี่ยวกับเลเยอร์ภาพ (และวิธีที่ Docker สร้างและจัดเก็บรูปภาพ) ให้ดูที่เกี่ยวกับไดรเวอร์อุปกรณ์จัดเก็บข้อมูล
## General guidelines and recommendations
  รูปภาพที่กำหนดโดย Dockerfile ของคุณควรสร้างคอนเทนเนอร์ที่ไม่สำคัญที่สุดเท่าที่จะทำได้ โดย“ ephemeral” เราหมายถึงคอนเทนเนอร์สามารถหยุดและทำลายจากนั้นสร้างใหม่และแทนที่ด้วยการตั้งค่าและการกำหนดค่าขั้นต่ำที่แน่นอน

  อ้างถึงกระบวนการภายใต้วิธีการของแอพสิบสองปัจจัยเพื่อให้เกิดความรู้สึกถึงแรงจูงใจในการใช้ภาชนะบรรจุในแบบไร้รัฐ
### Create ephemeral containers
### Understand build context
### Pipe Dockerfile through stdin
* BUILD AN IMAGE USING A DOCKERFILE FROM STDIN, WITHOUT SENDING BUILD CONTEXT
* BUILD FROM A LOCAL BUILD CONTEXT, USING A DOCKERFILE FROM STDIN
* BUILD FROM A REMOTE BUILD CONTEXT, USING A DOCKERFILE FROM STDIN	
### Exclude with .dockerignore //Short
### Use multi-stage builds
### Don’t install unnecessary packages //Short
### Decouple applications
### Minimize the number of layers
### Sort multi-line arguments //Short
### Leverage build cache
# Dockerfile instructions //Short
### FROM //Short
### LABEL
### RUN  //Short
### APT-GET
* APT-GET
Use-case ที่น่าจะเกิดขึ้นอย่างแน่นอนสำหรับ RUN คือแอพพลิเคชันของ apt-get. เพราะคือการติดตั้ง packages คำสั่ง  RUN apt-get   มีหลากหลาย gotchas เพื่อที่จะหลีกเลี่ยง RUN apt-get upgrade และ dist-upgrade มี package ที่มีความจำเป็นหลาย package จากparent images ที่ไม่สามารถอัพเกรดข้างในunprivileged container ได้ ถ้า package contained ใน parent image คือการล้าสมัย(เลิกใช้แล้ว) ให้ติดต่อผู้ดูแล  ถ้าคุณรู้จัก

รวมกันเสมอ RUN apt-get update กับ  apt-get install ในคำสั่ง  RUN เหมือนกัน ยกตัวอย่าง:

RUN apt-get update && apt-get install -y \
    package-bar \
    package-baz \
    package-foo
ใช้คำสั่ง apt-get update เพียงอย่างเดียวใน RUN เพราะปัญหาการแคชและต่อมาคือ apt-get install คำสั่งล้มเหลว ยกตัวอย่างเชื่อ , ไปที่ Dockerfile:
FROM ubuntu:18.04
RUN apt-get update
RUN apt-get install -y curl
หลังจากสร้าง  image, layers ทั้งหมดใน Docker cache. สมมติว่าคุณแก้ไขในภายหลัง apt-get install โดดยการเพิ่ม  extra package:
FROM ubuntu:18.04
RUN apt-get update
RUN apt-get install -y curl nginx

* USING PIPES
### CMD
### EXPOSE
### ENV
### ADD or COPY
### ENTRYPOINT
### VOLUME  //Short
### USER
### WORKDIR //Short
### ONBUILD 