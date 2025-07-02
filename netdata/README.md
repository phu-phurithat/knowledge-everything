# Netdata

### 🔧 คุณสมบัติเด่นของ Netdata

| คุณสมบัติ                             | รายละเอียด                                                                          |
| ------------------------------------- | ----------------------------------------------------------------------------------- |
| 📊 **Real-time monitoring**           | อัปเดตข้อมูลทุกวินาที ดูได้แบบกราฟเคลื่อนไหว                                        |
| ⚙️ **Auto-detect services**           | ตรวจเจอบริการต่างๆ โดยอัตโนมัติ เช่น nginx, MySQL, docker                           |
| 📦 **รองรับ plugins หลากหลาย**        | รองรับ Python, Go, bash สำหรับเขียน plugin เอง                                      |
| 📉 **Low-resource usage**             | ออกแบบมาให้เบาและเร็ว ไม่กิน CPU หรือ RAM มาก                                       |
| 🌐 **Web Dashboard**                  | มี dashboard แบบ web-based สำหรับดูทุก metric                                       |
| 🔒 **Security และ role-based access** | ควบคุมสิทธิ์การเข้าถึงข้อมูลและ dashboard ได้                                       |
| 🔄 **Distributed Monitoring**         | รองรับการรวมข้อมูลจากหลาย node (ผ่าน Parent-Agent)                                  |
| 💾 **Data Storage**                   | ใช้ `dbengine` เป็น TSDB ในตัว (ไม่ต้องพึ่ง InfluxDB หรือ Prometheus ถ้าไม่ต้องการ) |

***

### 🏗️ สถาปัตยกรรมของ Netdata

Netdata มี 2 mode:

1. **Standalone Agent**: ติดตั้งบนแต่ละเครื่อง ใช้ดู performance เฉพาะเครื่องนั้น
2. **Parent + Child Agents (Streaming)**:
   * Agent ที่เป็น "child" stream metrics ไปยัง "parent"
   * parent ทำหน้าที่รวมข้อมูลจากหลายเครื่อง (central monitoring)
   * ใช้สำหรับ monitoring multi-node แบบ distributed

***

### 📚 สิ่งที่สามารถ monitor ได้

* **System**: CPU, memory, disk, I/O, network
* **Web servers**: nginx, apache
* **Databases**: MySQL, PostgreSQL, Redis, MongoDB
* **Containers**: Docker, Kubernetes (ผ่าน plugins หรือ `go.d.plugin`)
* **Apps & Services**: Python scripts, bash scripts, custom exporters

***

### 🧠 การจัดเก็บข้อมูล (Database Engine)

* Netdata ใช้ฐานข้อมูลของตัวเองชื่อ `dbengine`
* มีการเก็บข้อมูลแบบ tiered (recent: ความละเอียดสูง, old: ความละเอียดต่ำ)
* สามารถตั้ง retention เองได้ เช่น 1 วัน, 1 สัปดาห์, 1 เดือน
* ไม่เหมาะกับ long-term storage เป็นปีๆ (Netdata Cloud ใช้เก็บข้อมูลระยะยาวแทน)

***

### ☁️ Netdata Cloud

* เป็นบริการ SaaS ที่ให้สามารถเชื่อมต่อกับ agents หลายเครื่อง แล้วดู metrics ได้จาก web เดียว
* **ฟรี** สำหรับ use-case ทั่วไป (มี plan เสียเงินสำหรับองค์กรใหญ่)
* ใช้เพื่อรวมการ monitor หลายๆ เครื่องในองค์กรแบบรวมศูนย์

***

### 🖥️ ตัวอย่าง UI Dashboard

Netdata แสดงผล metrics แบบ interactive ด้วย HTML5:

* กราฟแบบ real-time
* ซูมดูช่วงเวลาได้
* แจ้งเตือน (alarms) ได้

***

### 🚀 วิธีติดตั้ง (Linux)

```bash
bashCopyEditbash <(curl -Ss https://my-netdata.io/kickstart.sh)
```

> จะติดตั้งแบบ agent โดยอัตโนมัติ พร้อมเริ่มบริการทันทีที่พอร์ต `19999`

***

### 🧪 ใช้กับ Kubernetes ได้ไหม?

ได้แน่นอน:

* **ติดตั้ง Netdata Agent เป็น DaemonSet** บนแต่ละ node
* หรือ **ติดตั้งแค่บน node เดียวเพื่อ monitor Node นั้น**
* ใช้ร่วมกับ Prometheus, Grafana หรือ Netdata Cloud ได้

***

### 🔍 เปรียบเทียบกับ Prometheus

| ประเด็น           | Netdata                | Prometheus                     |
| ----------------- | ---------------------- | ------------------------------ |
| Focus             | Real-time monitoring   | Time-series metrics collection |
| Install           | ง่ายมาก (script เดียว) | ต้อง config หลายส่วน           |
| Dashboard         | Built-in               | ต้องใช้ Grafana                |
| Alerting          | มีในตัว                | ต้องใช้ AlertManager           |
| Long-term storage | ไม่เหมาะ               | เหมาะ                          |
| ใช้งานง่าย        | ⭐⭐⭐⭐⭐                  | ⭐⭐                             |

***

### ✅ เหมาะกับใคร?

* DevOps ที่อยากได้ระบบดู performance แบบเร็วๆ ง่ายๆ
* คนที่ต้องการ monitor ระบบขนาดเล็ก-กลาง
* นักพัฒนาอยาก debug performance แบบเรียลไทม์
