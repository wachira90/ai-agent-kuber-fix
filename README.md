# ai-agent-kuber-fix
ai-agent-kuber-fix

สร้างไฟล์ AGENTS.md สำหรับให้ AI สำรวจและแก้ไข Kubernetes แบบปลอดภัยเรียบร้อยแล้ว โดยออกแบบให้เริ่มต้นในโหมด Read-only และต้องผ่านจุดอนุมัติก่อนเปลี่ยนแปลงระบบทุกครั้ง


เนื้อหาสำคัญภายในไฟล์
```txt
กระบวนการแก้ปัญหาแบบ 9 Phases ตั้งแต่ Preflight จนถึง Closure
ตรวจสอบ context อย่างชัดเจน ป้องกันการทำงานผิด cluster
สำรวจ Nodes, Pods, Workloads, Storage, Network, DNS, CNI, RBAC และ Events
แยกข้อเท็จจริงออกจากสมมติฐานของ root cause
เริ่มต้นด้วยคำสั่ง Read-only เท่านั้น
Backup resource ก่อนทุก write action
แยก manifest backup ออกจาก application data backup
รองรับ Helm, Kustomize, Argo CD, Flux และ raw YAML
ใช้ kubectl diff และ --dry-run=server ก่อน Apply
มี Change Card สำหรับขออนุมัติเป็นราย Change
กำหนด Success criteria, Abort criteria และ Observation window
มี Rollback command ที่เฉพาะเจาะจงสำหรับแต่ละ Change
ป้องกันการเปิดเผย Kubernetes Secrets
ห้ามลบ PVC, PV, CRD, Namespace หรือแก้ etcd โดยอัตโนมัติ
มี Decision Matrix สำหรับ CrashLoopBackOff, OOMKilled, Pending, ImagePullBackOff, Node NotReady, PVC, DNS และ rollout ค้าง
สร้าง Evidence, Backup, Diff, Logs และ Final report พร้อม SHA-256 checksum
```

แนวทางหลักที่เพิ่มเข้าไปคือ Observe → Diagnose → Plan → Backup → Validate → Change → Verify → Rollback/Close เพื่อไม่ให้ AI ข้ามจากการพบอาการไปสู่การแก้ไขทันที และลดความเสี่ยงจากการสั่ง restart หรือลบ resource โดยยังไม่ทราบ root cause.
