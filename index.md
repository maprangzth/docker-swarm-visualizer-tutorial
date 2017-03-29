# มาเล่น Docker Swarm Visualizer กันเถอะ  

สืบเนื่องจากเมื่อวันที่ 28 ม.ค. 2560 ผมได้มีโอกาสจัดคอร์สเทรนนิ่ง Docker กันภายในบริษัทซึ่งตัวเนื้อหาก็จะเป็นการเล่าพื้นฐานและมี Workshop ย่อยๆ ตามหัวข้อดังนี้  

  - Software Container คืออะไร  
  - การใช้งาน Docker เพื่อให้เข้าใจ Concept **"Build, Ship, Run"**  
  - การจัดการ Container หลาย ๆ ตัวด้วย **Docker Compose**  
  - Provisioning Docker-Host ด้วย **Docker Machine**  
  - สุดท้ายก็จบด้วยการสร้าง Cluster ด้วย **Docker Swarm**  

จริงๆ ก็ไม่มีอะไรน่าตื่นเต้นครับ เพราะเนื้อหาเป็นแค่พื้นฐานเท่านั้น แต่ตอน Workshop ของ Docker Swarm นี่สิทำยังไงจะทำให้คนที่มาฟังเรา เข้าใจและเห็นภาพ เน้นว่า "เห็นภาพ" การทำงานของ Docker Swarm นี่มัน Thailand 4.0 แล้วนะ จะมัวมองแต่ Terminal ดำๆ ได้ไง 555+ พอดีไปเจอเจ้าตัวนี้: **Docker Swarm Visualizer** จึงเป็นที่มาของชื่อ Blog นี้ครับ  

เข้าเรื่องกันดีกว่า Docker Swarm Visualizer คืออะไรชื่อมันก็บอกอยู่แล้วครับคือการทำ visualizer ให้กับตัว Docker Swarm  จากที่คอยเคาะ Command บน Terminal เพื่อดูว่า Node ใน Cluster ของเราตอนนี้มีกี่ Node หรือ ตอนสั่ง Replicas Service เราก็อยากจะรู้ว่าแต่ล่ะ Task มันไปลงที่ Node ไหนก็ให้ภาพมันอธิบายซะเลยสิ จะได้เห็นกันชัดๆ ปั๊ดโถ๊วววววววว!!!

[![Visualizer](https://raw.githubusercontent.com/maprangzth/docker-swarm-visualizer-tutorial/gh-pages/visualizer-05.png)](https://github.com/maprangzth/docker-swarm-visualizer-tutorial/blob/gh-pages/visualizer-05.png)  

อ่านรายละเอียดเพิ่มเติม: [https://github.com/ManoMarks/docker-swarm-visualizer](https://github.com/ManoMarks/docker-swarm-visualizer/)  

> **Note:** This only works with Docker Swarm Mode in Docker Engine 1.12.0 and later. It does not work with the separate Docker Swarm project.  

เริ่มด้วยการ Provisioning Docker Host ด้วย Docker Machine ผมใช้เป็น **1 Manager 3 Worker** รวมเป็น 4 Nodes  
(-d หรือ --driver มีหลายตัวนะครับเข้าไปดูเพิ่มเติมจากลิงค์นี้ครับ https://docs.docker.com/machine/drivers/ ในตัวอย่างผมใช้เป็น virtualbox)

<pre>
$ docker-machine create -d virtualbox manager01  
$ docker-machine create -d virtualbox worker01  
$ docker-machine create -d virtualbox worker02 
$ docker-machine create -d virtualbox worker03 
</pre>

ตรวจสอบให้แน่ใจว่าได้ 4 Nodes อย่างที่สร้างไว้หรือเปล่า:  

<pre>
$ docker-machine ls
NAME        ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
manager01   *        virtualbox   Running   tcp://192.168.99.100:2376           v17.03.0-ce
worker01    -        virtualbox   Running   tcp://192.168.99.101:2376           v17.03.0-ce
worker02    -        virtualbox   Running   tcp://192.168.99.102:2376           v17.03.0-ce
worker03    -        virtualbox   Running   tcp://192.168.99.103:2376           v17.03.0-ce
</pre> 

ต่อด้วยสร้าง Cluster:  

<pre>
$ docker-machine ssh manager01 "docker swarm init --advertise-addr 192.168.99.100"  
</pre>

หรือ  

<pre>
$ eval $(docker-machine env manager01)  
$ docker swarm init --advertise-addr 192.168.99.100
</pre>  

หลังจากที่เราใช้คำสั่ง **docker swarm init** เราจะได้ Token สำหรับให้ node อื่นมา join cluster ประมาณนี้:  

<pre>
Swarm initialized: current node (r2tn81j7z9c619v5wm0df6u94) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-16c9xpw9m9f2o2tl5gn3qpdk8iss80ebj2zmha7tnajjhk3hgk-aegjp1uk82vzt6tw3rf8mky4s \
    192.168.99.100:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
</pre>  

แต่ยังไม่ต้องเอา Node อื่นมา Join Cluster นะ ให้รันคำสั่งเพื่อสร้าง Service ไอ้ Visualizer ที่ว่ามานั่นแหละก่อน

<pre>
$ docker service create \
  --name=viz \
  --publish=8080:8080/tcp \
  --constraint=node.role==manager \
  --mount=type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock \
  manomarks/visualizer
</pre>

จะได้เลข ID ของ Service ที่ชื่อว่า **viz** ใครทำตามแล้วได้เลขอื่น ไม่ต้องเครียดนะ ก็แล้วแต่ว่ามันจะ Generate อะไรให้ ตัวนี้เราไม่สนใจแค่อยากบอกเฉยๆ 555

<pre>
0ajbpkqsujk6i6qcf2lw9l51b
</pre>

เช็ค Service ดูซักหน่อยว่าพร้อมใช้งานแล้วหรือยัง

<pre>
$ docker service ls
ID            NAME  MODE        REPLICAS  IMAGE
0ajbpkqsujk6  viz   replicated  0/1       manomarks/visualizer:latest
</pre>

ถ้าลองเปิด Web Browser แล้วเข้าไปที่ IP ที่เป็นของ Manager ที่นี้ 192.168.99.100 ตามด้วย Port 8080 ตามที่ Map ไว้ในตอนสร้าง Service ก็ยังยังใช้งานไม่ได้ เพราะ REPLICAS ยังเป็น 0/1 ต้องรอให้เป็น 1/1 ก่อนเน้อ

<pre>
$ docker service ls
ID            NAME  MODE        REPLICAS  IMAGE
0ajbpkqsujk6  viz   replicated  1/1       manomarks/visualizer:latest
</pre>

พอ REPLICAS เป็น 1/1 แล้วก็เปิด Web Browser แล้วพิมพ์ **192.168.99.100:8080** (IP ท่านไม่จำเป็นต้องเหมือนผมนะครับ Manager ของท่านเป็น IP อะไรก็เอาตามนั้นอย่าเลียนแบบผม) กระแทก Enter ดัง ๆ ก็จะได้เจ้าตัว Visualizer ที่โม้มาแล้วจ้า สำหรับ Web Browser เปิดค้างไว้เลยเพราะมันแสดงผล (เกือบจะ) Real time

[![Visualizer](https://raw.githubusercontent.com/maprangzth/docker-swarm-visualizer-tutorial/gh-pages/visualizer-00.png)](https://github.com/maprangzth/docker-swarm-visualizer-tutorial/blob/gh-pages/visualizer-00.png) 

หลังจากนั้นเราก็เอา token ที่ได้จากการรัน **docker swarm init** ไปรันที่ worker01, worker02, worker03 ถ้าลืมไปแล้วก็รัน **docker swarm join-token worker** ที่ Manager (อยาก join ในส่วนของ Manager ก็เปลี่ยนจาก worker เป็น manager จอบอ):

<pre>
$ docker swarm join-token worker
To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-16c9xpw9m9f2o2tl5gn3qpdk8iss80ebj2zmha7tnajjhk3hgk-aegjp1uk82vzt6tw3rf8mky4s \
    192.168.99.100:2377
</pre>

แล้ว Copy Command ที่มันบอกมาไปวางที่ Node: worker01, worker02, worker03 

<pre>
$ eval $(docker-machine env worker01)  
$ docker swarm join \
    --token SWMTKN-1-16c9xpw9m9f2o2tl5gn3qpdk8iss80ebj2zmha7tnajjhk3hgk-aegjp1uk82vzt6tw3rf8mky4s \
    192.168.99.100:2377
</pre>

จะได้ผลลัพธ์แบบนี้:

<pre>
This node joined a swarm as a worker.
</pre>

จากนั้นก็ไปเช็คที่ Node Manager ดูว่ามี Node Join เข้ามาหรือยัง

<pre>
$ docker node ls
ID                           HOSTNAME   STATUS  AVAILABILITY  MANAGER STATUS
ol23e9qoopikf941f2eihqu6l    worker01   Ready   Active
r2tn81j7z9c619v5wm0df6u94 *  manager01  Ready   Active        Leader
</pre>

กลับไปที่หน้าเว็บ Visualizer เราก็จะพบว่ามี Node ที่เป็น Worker Join เข้ามาแล้นนนนนนนนนนน 

[![Visualizer](https://raw.githubusercontent.com/maprangzth/docker-swarm-visualizer-tutorial/gh-pages/visualizer-01.png)](https://github.com/maprangzth/docker-swarm-visualizer-tutorial/blob/gh-pages/visualizer-01.png) 

Node worker02, worker03 ก็ join แบบเดียวกับ worker01 นั่นแหละ ขอละไว้ในฐานที่เข้าใจ ถ้าเรา Join ครบแล้วจะได้หน้าตาประมาณนี้

[![Visualizer](https://raw.githubusercontent.com/maprangzth/docker-swarm-visualizer-tutorial/gh-pages/visualizer-02.png)](https://github.com/maprangzth/docker-swarm-visualizer-tutorial/blob/gh-pages/visualizer-02.png) 

ต่อไปเราจะสร้าง Service เพิ่มอีกซักตัวเพื่อดูว่ามันกระจาย Task กันยังไง (ต้องสร้างบน Node ที่เป็น Manager นะ)

<pre>
$ docker service create --name web --replicas 1 -p 80:80 nginx
xgm7z5zdmh8kba3sfx444njre
</pre>

จะเห็นได้ว่ามี Service ที่ชื่อว่า web โพล่มาแล้ว

[![Visualizer](https://raw.githubusercontent.com/maprangzth/docker-swarm-visualizer-tutorial/gh-pages/visualizer-03.png)](https://github.com/maprangzth/docker-swarm-visualizer-tutorial/blob/gh-pages/visualizer-03.png) 

แค่นี้ถ้ายังไม่แจ่ม ลอง Scale Service web เป็น 5 ดู

<pre>
$ docker service scale web=5
web scaled to 5
</pre>

จากภาพที่เห็นมันแดงๆ เพราะมันกำลังไป pull image ลงมานะครับ รอครับรอ อย่ารีบ **ทำเป็นวัยรุ่นใจร้อนไปได้ !!!**

[![Visualizer](https://raw.githubusercontent.com/maprangzth/docker-swarm-visualizer-tutorial/gh-pages/visualizer-04.png)](https://github.com/maprangzth/docker-swarm-visualizer-tutorial/blob/gh-pages/visualizer-04.png) 

ลอง Scale Service web กลับมาเป็น 1 (ก่อนหน้านี้ Scale ไป 5 ทำม๊ายยยยยย!!!)

<pre>
$ docker service scale web=1
web scaled to 1
</pre>

[![Visualizer](https://raw.githubusercontent.com/maprangzth/docker-swarm-visualizer-tutorial/gh-pages/visualizer-06.png)](https://github.com/maprangzth/docker-swarm-visualizer-tutorial/blob/gh-pages/visualizer-06.png) 


สุดท้ายลบ Node ออกจาก Cluster (อยากลบ Node ก็แล้วแต่ท่านนะครับ)

<pre>
$ eval $(docker-machine env worker03)
$ docker swarm leave
Node left the swarm.

$docker node rm worker03
worker03
</pre>

<pre>
$ eval $(docker-machine env worker02)
$ docker swarm leave
Node left the swarm.

$docker node rm worker02
worker02
</pre>

[![Visualizer](https://raw.githubusercontent.com/maprangzth/docker-swarm-visualizer-tutorial/gh-pages/visualizer-07.png)](https://github.com/maprangzth/docker-swarm-visualizer-tutorial/blob/gh-pages/visualizer-07.png) 


คงจะเห็นภาพการทำงาน ของ Docker Swarm (แบบโคตรจะพื้นฐาน) กันนะครับเพราะผมแคปมาให้ดูแล้วตั้งหลายภาพ อิอิ  

บทความนี้เป็นบทความแรกในชีวิตนะครับ ผิดพลาดตรงไหนต้องขออภัยไว้ ณ ที่นี้ด้วย เจอกันบทความหน้าครับ ^_^
