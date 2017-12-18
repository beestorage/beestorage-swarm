# BeeStorage
### ระบบคุณเคยเจอปัญหาแบบนี้หรือไม่
* **ระบบฝากภาพ ยากจังไม่เอาแล้ว**  
* **เซงไม่มี api ไม่มี Library พัฒนาเองทำเองทุกอย่าง**  
* **Server เก็บภาพเต็มอีกแล้วซื้อ HDD เพิ่ม**  

#### ปัญหาเหล่านี้จะหมดไปด้วย BeeStorage

# BeeStorage ฝากรูป *ขจัดปัญหา*  

### สร้างขึ้นมาเพื่อตอบโจทย์สำหรับ นักพัฒนา  
- **BeeStorage** ได้เลือก **[thumbor][thumbor-url]** เป็น front end มีความสามารถในการ **[process filter][thumbor-filter-url]** ผ่านการเรียก function ใน **[URL][thumbor-filter-url]** ได้ในตัว และยังมี **[library support หลายภาษา][thumbor-library-url]**  
- ใช้ **[nginx][nginx-url]** ที่มีความยืดหยุ่นในการจัดการระบบแคช
- ระบบการจัดเก็บไฟล์ หลังบ้านเป็น **[mongodb GridFS][mongo-gridfs-url]** ที่มีความสามารถเด่นเรื่องการ **สเกล** และความคง **ทนต่อระบบล่มได้สูง**
สามารถ เพิ่มขนาดเพิ่มเครื่อง เวลาระบบโต ภายหลังได้
- ระบบทั้งหมดอยู่บน **Docker Swarm** ติดตั้งจัดการได้ง่ายเพียงไม่กี่บรรทัด
---

### การติดตั้งแบ่งเป็นข้อๆ ตามลำดับดังนี้
1. **Clone sorce code จาก github**
2. **กำหนดสภาวะแวดล้อมของระบบ**
3. **ติดตั้ง BeeStorage**

---

### 1. Clone sorce code จาก github
#### ทุกเครื่องที่จะทำงานในระบบ BeeStorage จำเป็นต้อง setup ระบบก่อน โดยทำการ clone sorce code จาก github และสั่งการทำงานของ setup_env.sh

```shell
$ sudo git clone https://github.com/beestorage/beestorage.git






sudo sh /beestorage/setup_env.sh
```

---

### 2. กำหนดสภาวะแวดล้อมของระบบ เราจะแบ่งการตั้งค่า ออกเป็นข้อๆ ดังนี้  
 **2.1 การตั้งค่า swarm**  
 **2.2 การแป๊ะชื่ออ้างอิง หรือ label ให้กับ swarm node**

---

### 2.1 การตั้งค่า swarm


#### ที่เครื่อง **ubuntu_org1** ให้ใช้คำสั่ง `$ sudo docker swarm init`

```shell
ubuntu_org1>$ sudo docker swarm init
-0-0-0-=0=-0=-0
```

#### ให้ copy คำสั่งที่ใช้สำหรับการเชื่อมต่อ มาใส่ที่เครื่อง coreosex3

```shell
coreosex3>$ sudo docker swarm join .. . .. .
0131030130154
```
---

### 2.2 การแป๊ะ label ให้ทำที่เครื่อง ubuntu_org1
> ชื่อที่กำหนดเป็นการอ้างถึงโดย docker มีเพื่อให้ง่ายต่อการ ตั้งค่าระบบ

|  hostname     | ให้ใช้ชื่อ     | คำอธิบาย |
|:---:|:---:|:---|
| ubuntu_org1       | **frontend**       | **Swarm manager node ทำงานเป็น BeeStorage ** |
| coreosex3   |  **data1** | **Swarm worker node ทำงานเป็นเครื่อง Database** |

> | Swarm node คืออะไร     | คำอธิบาย     |
| :------------- | :------------- |
| **manager node**       | คือเครื่องที่ทำหน้าที่ในการแบ่งงานให้กับเครื่องที่เป็น worker node โดยปรกติ manager node จะทำหน้าที่เป็นทั้ง mananger และ worker ในเวลาเดียวกัน       |
| **worker node** | คอยรับงานจาก manager node |

#### Software ที่จำเป็น

| Software     | หมายเหตุ     |
| :-------------: | :-------------: |
| docker       | version 13.1 ขึ้นไป       |
| git   | version ล่าสุด  |

  - **การแป๊ะ label ในส่วนของ hostname ubuntu_org1 และ hostname coreosex3 ปรับเปลี่ยนตามชื่อเครื่องจริง นอกจากนั้นก๊อปแป๊ะได้เลย**

  ```shell
  ubuntu_org1>$ sudo docker node update --label-add mongo.role=frontend ubuntu_org1
  ubuntu_org1>$ sudo docker node update --label-add mongo.role=data1 coreosex3
  ```

[**อ่านเพิ่มเติม คลิก**][mongo-label-add-url]

---
### 3. ติดตั้ง BeeStorage
**คำสั่ง Deploy จะสั่งการทำงานที่เครื่อง ubuntu_org1 ที่เป็น manager node ด้วคำสั่ง**

```shell
ubuntu_org1>$ sudo docker stack deploy -c compose-swarm-beestorage.yml beestorage
```

**สำหรับการใช้งานหลัง Deploy เสร็จต้องรอ 1 นาที โดย 1 นาทีนี้ระบบจะทำการตรวจสอบ database โดยอัตโนมัติ**

---

### ลองใช้งานผ่าน curl และ wget

- ทดสอบ upload รูปผ่าน curl ด้วยคำสั่ง

  ```shell
  curl -i -H "Content-Type: image/jpeg" -XPOST http://11.0.0.164:9000/image --data-binary "@/mnt/img.jpg"                                               HTTP/1.1 100 (Continue)

  HTTP/1.1 201 Created
  Date: Thu, 07 Dec 2017 04:39:18 GMT
  Content-Length: 0
  Content-Type: text/html; charset=UTF-8
  Location: /image/c4ba387492864969b1e46591698cbc1b/image.jpg
  Server: TornadoServer/4.5.2
  ```

- สามารถทดสอบ download ด้วย wget

  ```shell
  wget http://11.0.0.164:9000/image/c4ba387492864969b1e46591698cbc1b -O testdownload.jpg
  --2017-12-07 11:40:11--  http://11.0.0.164:9000/image/c4ba387492864969b1e46591698cbc1b
  Connecting to 11.0.0.164:9000... connected.
  HTTP request sent, awaiting response... 200 OK
  Length: 265284 (259K) [image/jpeg]
  Saving to: ‘testdownload.jpg’

  testdownload.jpg                             100%[==============================================================================================>] 259.07K  --.-KB/s    in 0.004s

  2017-12-07 11:40:11 (69.0 MB/s) - ‘testdownload.jpg’ saved [265284/265284]
  ```
หรือสามารถทดสอบด้วย [**postman**][postman-url] ก็ได้

## [การใช้งานด้วย Library][thumbor-library-url]
---
### การ Scale เพิ่มเครื่อง เพิ่มความจุ เพิ่มความเร็ว  
บทความส่วนี้ จะดีมากถ้าผู้ใช้ เป็น docker อยู่แล้ว
1. เพิ่ม worker node ใหม่ อย่าลืมแป๊ะ role ด้วยนะครับ ในที่นี้เราจะใช้ Role `data2`
  - หลังจากเพิ่ม worker node ใหม่ทำการแป๊ะ Role ให้เรียบร้อย  ในที่นี้ใช้ชื่อว่า `data2`
  ```
  $ sudo docker node update --label-add mongo.role=data2 beedb2
  ```
2. เพิ่มเครื่องใหม่ใน compose-swarm-beestorage.yml ในส่วนท้ายของไฟล์
  - เพิ่มส่วนท้ายไฟล์ตามนี้ ดูตัวอย่างไฟล์ที่แก้เสร็จแล้วได้ใน `example\scale_out\compose-swarm-beestorage.yml`
  ```
  datadb_r1s2:     #เปลี่ยนชื่อจาก datadb_r1s1 เป็น datadb_r1s2
    image: mongo:3.4
    volumes:
      - /mnt/beestorage-swarm/database/db:/data/db database
    command: mongod --noprealloc --replSet dataGroup2 --shardsvr --port 27020     #แก้ dataGroup1 เป็น dataGroup2
    deploy:
      mode: replicated
      replicas: 1
      placement:
        constraints:
          - node.labels.mongo.role == data2             #แก้ Role data1 เป็น data2
  ```
  - แล้ว deploy ซ้ำในชื่อ stack เดิมจะเป็นการ update swarm ด้วยคำสั่งด้านล่างนี้
  ```
  $ sudo docker stack deploy -c compose-swarm-beestorage.yml beestorage
  ```
3. config mongoDB ตัวใหม่
  - ที่เครื่อง manager node เข้าไปที่ shell ของ beestorage_mongo_router ให้ใช้ `sudo docker ps` ดู Name ก่อนโดยจะขึ้นต้นด้วย beestorage_mongo_router
  ```
  $ sudo docker exec -it beestorage_mongo_router.1.5li4j9iytbc6qypv63y04j3x1 mongo --host datadb_r1s2 --port 27020
  ```

  - คัดลอกข้อมูลตามนี้ลงไป
  ```
  rs.initiate(
  {
    "_id": "dataGroup2",
    "members": [
      {
        "_id": 0,
        "host": "datadb_r1s2:27020",
        "priority": 100
      }
    ]
  }
  )
  ```
  กด Enter จะขึ้นว่า OK  
  และ ตามด้วย `exit` ออกจาก shell

4. บอก mongo router ว่ามีเครื่องใหม่มาแล้วนะ ที่เครื่อง manager node เข้าไปที่ shell ของ beestorage_mongo_router ให้ใช้ `sudo docker ps` ดู
  ```
  $ sudo docker exec -it beestorage_mongo_router.1.5li4j9iytbc6qypv63y04j3x1 mongo --host mongo_router --port 27081
  ```
  ใส่คำสั่งตามนี้ลงไป
  ```
  mongo> use picture
  mongo> sh.addShard( "dataGroup2/datadb_r1s2:27020")
  ```
  กด Enter จะขึ้นว่า OK  
  และ ตามด้วย `exit` ออกจาก shell เสร็จสิ้น

---

[**สามารถอ่านเพิ่มเติมได้ใน Wiki คลิก**][wiki-url]


## License

    MIT License

    Copyright (c) 2017 NECTEC
       National Electronics and Computer Technology Center, Thailand

    Permission is hereby granted, free of charge, to any person obtaining a copy
    of this software and associated documentation files (the "Software"), to deal
    in the Software without restriction, including without limitation the rights
    to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
    copies of the Software, and to permit persons to whom the Software is
    furnished to do so, subject to the following conditions:

    The above copyright notice and this permission notice shall be included in all
    copies or substantial portions of the Software.

    THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
    IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
    FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
    AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
    LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
    OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
    SOFTWARE.


[![NECTEC](http://www.nectec.or.th/themes/nectec/img/logo.png)](https://www.nectec.or.th)




[postman-url]: https://www.getpostman.com/
[thumbor-url]: https://thumbor.readthedocs.io
[thumbor-filter-url]: https://thumbor.readthedocs.io/en/latest/filters.html
[thumbor-library-url]: https://thumbor.readthedocs.io/en/latest/libraries.html?highlight=library
[mongo-gridfs-url]: https://docs.mongodb.com/manual/core/gridfs/#when-to-use-gridfs
[mongo-url]: https://www.mongodb.com/
[mongo-shard-url]: https://docs.mongodb.com/manual/sharding/
[nginx-url]: https://www.nginx.com/
[docker-install-url]: https://docs.docker.com/engine/installation/
[docker-swarm-url]: https://docs.docker.com/engine/swarm/swarm-tutorial/
[docker-machine-url]: https://docs.docker.com/machine/get-started/#create-a-machine
[wiki-url]: https://beestorage.github.io/

[mongo-label-add-url]: https://docs.docker.com/engine/swarm/manage-nodes/#add-or-remove-label-metadata
