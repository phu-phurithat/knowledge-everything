# Netdata child on k8s stream to standalone parent

การติดตั้ง **Netdata Agent แบบ Child บน Kubernetes** เพื่อ **stream metrics ไปยัง Netdata Parent (standalone)** เป็นแนวทางที่ดีในการทำ **centralized monitoring** โดยใช้ทรัพยากรน้อยกว่าการใช้ Netdata Cloud หรือ Prometheus Stack เต็มรูปแบบ

***

### 🎯 เป้าหมาย

* ติดตั้ง Netdata Agent (Child) บน **แต่ละ Node** ของ Kubernetes (ผ่าน `DaemonSet`)
* Agent แต่ละตัวจะ **stream metrics ไปยังเครื่อง Parent Netdata** ที่รันอยู่นอก Kubernetes (หรือใน Kubernetes แบบ standalone mode)
* ใช้ **dashboard ที่ Parent** ดูรวม metrics ของทุก Node ได้

***

### 🖼️ สถาปัตยกรรม

```
[Node1] ─┐
[Node2] ─┤  (Child Agents on K8s)
[Node3] ─┘
    │
    ▼
[Parent Netdata (Standalone)]  ← All metrics streamed here
```

***

### ✅ ขั้นตอนแบบ Step-by-Step

#### 1. 🔧 เตรียมเครื่อง Parent Netdata

บนเครื่อง Parent (Standalone – นอก k8s หรือใน k8s ก็ได้):

```bash
bash <(curl -Ss https://my-netdata.io/kickstart.sh)
```

* หรือโหลด script จาก [official](https://learn.netdata.cloud/docs/netdata-agent/installation/linux)
* ตรวจสอบว่า service รันที่ `:19999`
* เปิดรับ stream จาก child:

```bash
sudo nano /etc/netdata/netdata.conf
```

```ini
[web]
    mode = static-threaded
[stream]
    enable = yes
    default memory mode = ram
    allow from = *
```

> เปลี่ยน `allow from` เป็น IP Range ของ Cluster หรือ CIDR เช่น `10.0.0.0/8`

**Restart:**

```bash
sudo systemctl restart netdata
```

***

#### 2. 🧪 ทดสอบ Agent Stream จากเครื่องภายนอก (optional)

บน client node อื่น:

```bash
bash <(curl -Ss https://my-netdata.io/kickstart.sh)

# ตั้งค่า streaming:
sudo nano /etc/netdata/stream.conf
```

```ini
[stream]
    enabled = yes
    destination = <PARENT_IP>:19999
    api key = my_k8s_stream_key
```

> ต้องตรงกับค่า `api key` ที่ parent ยอมรับ หรือ default ถ้าไม่ได้เปิด `require authentication`
>
> ถ้าอยากใช้ `api key` ต้องเพิ่ม config เข้าไปที่ `etc/netdata/stream.conf` ของ parent

```ini
[my_k8s_stream_key]
    enable = yes
    default memory mode = ram
    allow from = *
```

#### 3. ☸️ ติดตั้ง Netdata Agent บน Kubernetes via Helm

เพิ่ม Helm repo ของ Netdata

```bash
helm repo add netdata https://netdata.github.io/helmchart/
helm repo update
```

สร้างค่า config: `values.yaml`

```yaml
child:
  enabled: true
  configs:
    stream:
      enabled: true
      data: |
          [stream]
            enabled = yes
            destination = <PARENT_IP>:19999
            api key = my_k8s_stream_key
            timeout seconds = 60
            buffer size bytes = 1048576
            reconnect delay seconds = 5
            initial clock resync iterations = 60
      path: /etc/netdata/stream.conf

k8sState:
  enabled: true
  configs:
    stream:
      enabled: true
      data: |
          [stream]
            enabled = yes
            destination = <PARENT_IP>:19999
            api key = my_k8s_stream_key
            timeout seconds = 60
            buffer size bytes = 1048576
            reconnect delay seconds = 5
            initial clock resync iterations = 60
      path: /etc/netdata/stream.conf

# แก้ปัญหา nginx ingress annotation deprecated
ingress:
  enabled: true
  annotations:
    kubernetes.io/tls-acme: "true"
  path: /
  pathType: Prefix
  hosts:
    - netdata.k8s.local
  spec: 
    ingressClassName: nginx
```

ติดตั้ง Netdata ด้วย Helm

```bash
helm install netdata-agent netdata/netdata -f values.yaml
```

ตรวจสอบการทำงาน

1. เข้าหน้า dashboard ของ Parent ที่ `http://<PARENT_IP>:19999`
2. เมนูด้านซ้ายควรแสดง node ของ K8s ที่กำลัง stream metrics เข้ามา
3. ใช้คำสั่งด้านล่างเพื่อตรวจสอบ logs:

```bash
kubectl logs -l app.kubernetes.io/name=netdata -n default
```

***

### 🧠 คำแนะนำเพิ่มเติม

| สิ่งที่ควรทำ                   | วิธี                                                      |
| ------------------------------ | --------------------------------------------------------- |
| ⚙️ ตรวจสอบ firewall            | ให้แน่ใจว่า Parent เปิดพอร์ต `19999` สำหรับรับ stream     |
| 🔐 ใช้ VPN หรือ firewall rules | ถ้าไม่ต้องการเปิด wide access                             |
| 🧪 ทดสอบเชื่อมต่อจาก Pod       | `kubectl exec -it <pod> -- curl http://<PARENT_IP>:19999` |
| 📦 ดูค่า metrics บน parent     | จากเมนูซ้าย ควรเห็น `k8s-<node-name>` แต่ละตัวแยกกัน      |
| 🔧 ปรับ config                 | ผ่าน Helm `--set` หรือ `values.yaml`                      |
