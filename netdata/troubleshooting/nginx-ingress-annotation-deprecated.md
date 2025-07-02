# ปัญหา nginx ingress annotation deprecated

สมมุติว่า install via helm ให้แก้ไขไม่ให้ใช้ annotation deprecated&#x20;

```
# เพิ่มลงใน values.yaml
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

