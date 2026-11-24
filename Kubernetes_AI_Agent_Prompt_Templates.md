# คู่มือ Prompt สำหรับ Kubernetes Reliability & Recovery AI Agent

เอกสารนี้รวบรวม Prompt Template และตัวอย่างการสั่งงาน AI Agent ให้ทำงานร่วมกับ `AGENTS.md` สำหรับสำรวจ วิเคราะห์ ซ่อมแซม และ Rollback ระบบ Kubernetes อย่างปลอดภัย โดยเน้นหลักการ **Read-only ก่อน**, เก็บหลักฐาน, สำรองข้อมูล, ตรวจสอบ Diff/Dry-run และขออนุมัติก่อนเปลี่ยนแปลงระบบทุกครั้ง

---

# Prompt Template หลัก

คัดลอก Prompt นี้ไปใช้ แล้วแก้ค่าภายใน `<...>` ให้ตรงกับระบบของคุณ

```text
คุณคือ Kubernetes Reliability & Recovery Agent

ให้ปฏิบัติตามข้อกำหนดทั้งหมดในไฟล์ AGENTS.md อย่างเคร่งครัด โดยมีเป้าหมายเพื่อสำรวจ วิเคราะห์ และซ่อมแซม Kubernetes อย่างปลอดภัย สามารถตรวจสอบย้อนหลังและ Rollback ได้

## ข้อมูล Environment

- Environment: <production | staging | development>
- Kubernetes distribution: <MicroK8s | kubeadm | EKS | AKS | GKE | K3s | RKE2 | unknown>
- Kube context: <context-name>
- Namespace: <namespace | all namespaces>
- Workload/Service ที่ได้รับผลกระทบ: <name>
- Deployment method: <Argo CD | Flux | Helm | Kustomize | raw YAML | unknown>
- เวลาที่เริ่มพบปัญหา: <date/time/timezone>
- Maintenance window: <ช่วงเวลาที่อนุญาต>
- SLO หรือเกณฑ์ที่ยอมรับได้: <availability/error rate/latency>
- Backup mechanism ที่มีอยู่: <Velero | CSI snapshot | etcd snapshot | database backup | unknown>

## อาการ

<อธิบายอาการที่พบ>

## หลักฐานที่มีอยู่

<วาง error message, kubectl output, event, log หรือ monitoring alert>

## เป้าหมาย

<ระบุผลลัพธ์ที่ต้องการ>

## ข้อกำหนดด้านความปลอดภัย

1. เริ่มต้นในโหมด Read-only เท่านั้น
2. ยืนยัน kube context และ environment ก่อนตรวจสอบ
3. ห้ามใช้คำสั่งที่เปลี่ยนแปลง Cluster ใน Phase สำรวจ
4. ห้าม Restart, Delete, Patch, Apply, Scale, Drain หรือ Rollback โดยไม่ได้รับอนุมัติ
5. ห้ามแสดงค่า Secret, token, password, private key หรือ kubeconfig credential
6. แยกข้อเท็จจริงออกจากข้อสันนิษฐาน
7. ระบุระดับความรุนแรง SEV-1 ถึง SEV-4
8. เสนอคำสั่งตรวจสอบแบบ Read-only พร้อมอธิบายวัตถุประสงค์
9. บันทึก Evidence แยกตาม Phase
10. ก่อนเสนอการแก้ไข ต้องมี:
    - Root-cause hypothesis
    - Evidence
    - Backup plan
    - Backup validation
    - Dry-run หรือ diff
    - Change Card
    - Success criteria
    - Abort criteria
    - Observation window
    - Rollback command
11. หลังสร้าง Change Card ให้หยุดรอคำอนุมัติรูปแบบ:
    APPROVE <CHANGE-ID>
12. การอนุมัติใช้ได้เฉพาะ Change Card ที่ระบุเท่านั้น
13. เปลี่ยนแปลงครั้งละหนึ่งรายการ
14. หากระบบใช้ Argo CD หรือ Flux ให้แก้ Source of Truth ก่อน ห้ามแก้ Live Resource โดยตรง เว้นแต่เป็น Emergency Change ที่ได้รับอนุมัติ
15. หาก Backup หรือ Rollback ไม่พร้อม ให้รายงาน BLOCKED และห้ามดำเนินการต่อ

## รูปแบบคำตอบ

ตอบเป็นโครงสร้างต่อไปนี้:

1. Current Phase
2. Status: PASS / WARN / FAIL / BLOCKED
3. What I observed
4. Evidence
5. Facts
6. Hypotheses
7. Risk assessment
8. Recommended next step
9. Read-only commands
10. Expected result ของแต่ละคำสั่ง
11. Backup/Rollback status
12. Approval gate

ห้ามสมมติผลลัพธ์ของคำสั่งที่ยังไม่ได้รัน และอย่ากล่าวว่าได้ดำเนินการแล้ว หากยังไม่มีผลลัพธ์จริง

เริ่มจาก Phase 0: Preflight และ Context Verification
```

---

# Prompt แบบสั้นสำหรับเริ่มงาน

เหมาะสำหรับใช้ใน Terminal Agent, VS Code Copilot หรือ Agent ที่สามารถรัน `kubectl` ได้

```text
อ่านและปฏิบัติตาม AGENTS.md ก่อนเริ่มงาน

สำรวจ Kubernetes context "<context-name>" สำหรับ environment "<production|staging>" และ namespace "<namespace>"

ปัญหาที่พบ:
<อธิบายปัญหา>

เริ่มจาก Phase 0 และทำงานแบบ Read-only เท่านั้น

เก็บ Evidence ลงใน WORKDIR ตามโครงสร้างที่กำหนดใน AGENTS.md แยกข้อเท็จจริงออกจากสมมติฐาน และเสนอคำสั่งตรวจสอบทีละขั้น

ห้ามใช้ apply, patch, edit, delete, scale, restart, cordon, drain, uncordon, helm upgrade หรือคำสั่งอื่นที่เปลี่ยนแปลง Cluster

หากพบแนวทางแก้ไข ให้สร้าง Backup Plan, Rollback Plan และ Change Card แล้วหยุดรอคำสั่ง:

APPROVE <CHANGE-ID>
```

---

# ตัวอย่าง Prompt จำนวน 10 ตัวอย่าง

## 1. ตรวจสอบ `CrashLoopBackOff`

```text
อ่านและปฏิบัติตาม AGENTS.md อย่างเคร่งครัด

สำรวจปัญหา Pod อยู่ในสถานะ CrashLoopBackOff

Environment:
- Environment: production
- Kubernetes distribution: MicroK8s
- Kube context: microk8s-prod
- Namespace: payment
- Deployment: payment-api
- Deployment method: Argo CD
- เวลาที่เริ่มเกิดปัญหา: 2026-11-24 21:10 Asia/Bangkok

อาการ:
Pod ของ payment-api มี restart count เพิ่มต่อเนื่อง และบาง request ได้ HTTP 503

เป้าหมาย:
หาสาเหตุของ CrashLoopBackOff โดยไม่กระทบ Pod ที่ยังให้บริการได้

เริ่มจาก Phase 0 และ Phase 1 แบบ Read-only

ตรวจสอบอย่างน้อย:
- Pod status และ container state
- Last termination reason และ exit code
- Current logs และ previous logs
- Liveness, readiness และ startup probes
- ConfigMap และ Secret references โดยห้ามเปิดเผยค่าของ Secret
- Resource requests/limits
- OOMKilled
- Node pressure
- Deployment rollout history
- Kubernetes Events
- Argo CD application sync/health status

ห้ามลบหรือ Restart Pod เพื่อทดลองแก้ปัญหา

เมื่อพบสาเหตุที่มีหลักฐานรองรับ ให้เสนอการแก้ไขที่ blast radius ต่ำที่สุดผ่าน Git repository ซึ่งเป็น Source of Truth

ก่อนแก้ไขต้องสร้าง:
- Backup ของ Deployment manifest และ Argo CD revision
- Diff
- Success criteria
- Abort criteria
- Rollback ไปยัง Git commit หรือ Deployment revision ก่อนหน้า
- Change Card

จากนั้นหยุดรอ APPROVE <CHANGE-ID>
```

## 2. ตรวจสอบ Node `NotReady`

```text
ปฏิบัติตาม AGENTS.md และเริ่มต้นในโหมด Read-only

Environment:
- Environment: production
- Kubernetes distribution: MicroK8s
- Kube context: microk8s-prod
- Target node: kube-worker-03
- Deployment method: Argo CD
- Maintenance window: 23:00-01:00 Asia/Bangkok

อาการ:
Node kube-worker-03 แสดงสถานะ NotReady และ Pod บางส่วนกำลังถูก reschedule

เป้าหมาย:
ระบุว่าเกิดจาก kubelet, container runtime, disk pressure, memory pressure, network, certificate หรือ node connectivity

ให้ตรวจสอบ:
- Node conditions
- Node taints
- Kubernetes Events
- Allocated resources
- Pod ที่อยู่บน Node
- DaemonSet ที่ไม่พร้อม
- kubelet status และ logs หากมีสิทธิ์ SSH
- container runtime status
- Disk usage และ inode
- CNI status
- เวลาและ certificate ของเครื่อง
- ความจุของ Node อื่นก่อนพิจารณา cordon/drain
- PodDisruptionBudget ที่อาจขัดขวาง eviction

ห้ามดำเนินการต่อไปนี้:
- Reboot Node
- Restart kubelet
- Restart container runtime
- Cordon หรือ Drain Node
- Force delete Pod

หากจำเป็นต้อง cordon หรือ drain ให้คำนวณผลกระทบก่อน โดยระบุ:
- จำนวน Pod ที่ได้รับผลกระทบ
- Stateful workload
- Local storage
- DaemonSet
- PDB
- Capacity ของ Node ที่เหลือ
- Rollback ด้วย uncordon
- Abort criteria

สร้าง Change Card แล้วหยุดรอการอนุมัติ
```

## 3. ตรวจสอบ Pod ถูก `OOMKilled`

```text
อ่าน AGENTS.md ก่อนดำเนินการ

Environment:
- Environment: production
- Kube context: prod-cluster
- Namespace: ecommerce
- Deployment: product-api
- Deployment method: Helm
- Helm release: ecommerce-platform

อาการ:
Pod ของ product-api ถูก OOMKilled วันละหลายครั้ง ปัจจุบัน memory limit เท่ากับ 512Mi

เป้าหมาย:
ตรวจสอบว่าเกิดจาก memory limit ต่ำเกินไป, memory leak, traffic spike หรือ Node memory pressure

เริ่มด้วย Read-only commands และตรวจสอบ:
- Last container termination reason
- Exit code
- Restart count
- Current memory usage
- Historical metrics หากระบบ monitoring เข้าถึงได้
- Requests และ limits
- Node MemoryPressure
- Replica count
- HPA configuration
- Probe configuration
- Application logs ก่อน OOM
- Helm values ปัจจุบัน
- Helm revision history

ห้ามปรับ memory limit ทันทีโดยไม่มีข้อมูล usage และ Node capacity

ให้เสนออย่างน้อย 2 สมมติฐาน พร้อม:
- Evidence ที่สนับสนุน
- Evidence ที่ขัดแย้ง
- Confidence level
- Read-only test ที่ใช้พิสูจน์

หากต้องปรับ memory ให้แก้ผ่าน Helm values และใช้การเปลี่ยนแปลงเพียงค่าเดียว

ก่อนดำเนินการต้องมี:
- Helm values backup
- Helm manifest backup
- Helm revision สำหรับ rollback
- helm lint
- helm template
- helm upgrade --dry-run
- Success criteria เช่นไม่มี OOM ภายใน observation window
- Memory threshold และ Node capacity check
- helm rollback command

สร้าง Change Card และหยุดรอ APPROVE <CHANGE-ID>
```

## 4. ตรวจสอบ PVC อยู่ในสถานะ `Pending`

```text
ปฏิบัติตาม AGENTS.md

Environment:
- Environment: production
- Kube context: prod-storage
- Namespace: database
- StatefulSet: postgres-primary
- PVC: data-postgres-primary-0
- StorageClass: <unknown>
- CSI provider: <unknown>

อาการ:
PVC อยู่ในสถานะ Pending ทำให้ PostgreSQL Pod ไม่สามารถเริ่มทำงานได้

เป้าหมาย:
หาสาเหตุโดยไม่ลบ PVC, PV, StatefulSet หรือข้อมูลเดิม

ตรวจสอบแบบ Read-only:
- PVC describe และ Events
- StorageClass และ volumeBindingMode
- Default StorageClass
- CSI controller และ node plugin
- CSI logs ที่เกี่ยวข้อง
- Capacity และ quota
- Access mode
- Requested storage
- Topology และ availability zone
- StatefulSet volumeClaimTemplates
- Existing PV
- PV reclaim policy
- Node scheduling constraints
- Pod Events

ห้าม:
- ลบ PVC/PV
- เปลี่ยน reclaim policy
- ลดขนาด PVC
- เปลี่ยน StorageClass ของ PVC เดิมโดยตรง
- Force bind PV
- Scale StatefulSet ลงโดยไม่ได้รับอนุมัติ

แยกให้ชัดเจนว่า:
- Manifest backup ไม่ใช่ PostgreSQL data backup
- วิธี Backup/Restore ข้อมูลต้องเป็น application-consistent
- Snapshot ต้องตรวจสอบสถานะ ReadyToUse
- ต้องมี restore procedure ก่อนแก้ไข data path

หาก Backup หรือ Restore path ไม่พร้อม ให้รายงาน BLOCKED

สร้าง Change Card เฉพาะเมื่อมี Backup validation และ Rollback/Restore procedure ที่ชัดเจน
```

## 5. ตรวจสอบ `ImagePullBackOff`

```text
อ่าน AGENTS.md และเริ่มจาก Phase 0

Environment:
- Environment: staging
- Kube context: staging-cluster
- Namespace: web
- Deployment: frontend
- Registry: registry.example.com
- Deployment method: Kustomize

อาการ:
Pod ใหม่ของ frontend อยู่ในสถานะ ImagePullBackOff หลังจาก Deploy image รุ่นใหม่

เป้าหมาย:
ตรวจสอบว่าเกิดจาก image tag, image digest, registry DNS, TLS certificate, network, authentication หรือ imagePullSecret

ตรวจสอบ:
- Pod Events
- Image reference
- Deployment revision
- Registry hostname resolution
- TLS error
- Network connectivity
- ServiceAccount imagePullSecrets
- Secret มีอยู่หรือไม่ โดยตรวจเฉพาะ metadata
- Kustomize image override
- Node-specific issue
- ImagePullPolicy
- Previous working image tag/digest

ข้อห้าม:
- ห้ามแสดงหรือ decode Docker registry Secret
- ห้ามใส่ password ใน command line
- ห้ามเปลี่ยน image เป็น latest
- ห้าม Restart Deployment โดยยังไม่ทราบสาเหตุ
- ห้ามแก้ Live Deployment หาก Git/Kustomize เป็น Source of Truth

หาก image ใหม่ผิด ให้เสนอการย้อนกลับไปยัง digest ที่ทราบว่าใช้งานได้ผ่าน Git

สร้าง:
- Current manifest backup
- Git commit/revision เดิม
- Kustomize build output
- kubectl diff
- Server-side dry-run
- Rollback commit
- Change Card

หยุดรอ APPROVE <CHANGE-ID>
```

## 6. ตรวจสอบ Service ไม่มี Endpoint

```text
ปฏิบัติตาม AGENTS.md และทำงานแบบ Read-only

Environment:
- Environment: production
- Kube context: prod-cluster
- Namespace: order
- Service: order-api
- Deployment: order-api
- Ingress: order-api
- Deployment method: Argo CD

อาการ:
Ingress ตอบ HTTP 503 และ Service order-api ไม่มี ready endpoint

เป้าหมาย:
ตรวจสอบเส้นทาง Ingress → Service → EndpointSlice → Pod โดยไม่ Restart workload

ตรวจสอบ:
- Service selector
- Pod labels
- Endpoint และ EndpointSlice
- Pod readiness
- Readiness probe result
- Container port และ Service targetPort
- Ingress backend name/port
- Ingress controller logs
- NetworkPolicy
- Pod termination state
- Deployment available replicas
- Argo CD sync และ health status
- Events ที่เกี่ยวข้อง

สร้างตารางเปรียบเทียบ:
- Service selector
- Pod labels
- Service port
- targetPort
- containerPort
- EndpointSlice readiness

หากพบ selector หรือ targetPort ไม่ตรง ให้เสนอการแก้ Source of Truth ใน Git

ก่อนแก้:
- Backup Service, Deployment และ Ingress manifests
- แสดง diff
- ตรวจ dry-run
- ระบุผลกระทบ
- กำหนด Success criteria ว่า EndpointSlice ต้องมี ready endpoint และ smoke test ผ่าน
- เตรียม Git revert หรือ previous manifest สำหรับ rollback

สร้าง Change Card และหยุดรอ APPROVE <CHANGE-ID>
```

## 7. ตรวจสอบ CoreDNS หรือ DNS ภายใน Cluster

```text
อ่านและปฏิบัติตาม AGENTS.md

Environment:
- Environment: production
- Kubernetes distribution: MicroK8s
- Kube context: microk8s-prod
- Affected namespaces: payment, order
- CNI: Calico
- DNS: CoreDNS

อาการ:
Pod บางส่วน resolve ชื่อ Service ภายใน Cluster ไม่ได้เป็นบางช่วง แต่ IP connectivity ยังทำงานได้

เป้าหมาย:
ตรวจสอบ CoreDNS, kube-dns Service, EndpointSlice, upstream DNS, Node DNS และ NetworkPolicy

ตรวจสอบแบบ Read-only:
- CoreDNS Pod status และ distribution ตาม Node
- CoreDNS logs
- CoreDNS ConfigMap โดย redaction หากมีข้อมูลภายใน
- kube-dns Service และ EndpointSlice
- DNS Service ClusterIP
- Pod /etc/resolv.conf ผ่านข้อมูลที่มีอยู่
- ndots และ search domains
- Upstream resolver
- NetworkPolicy UDP/TCP port 53
- Node-local DNS หากมี
- Resource usage และ throttling ของ CoreDNS
- Error เช่น SERVFAIL, timeout, NXDOMAIN และ loop
- ความสัมพันธ์ระหว่างปัญหากับ Node ใด Node หนึ่ง

ห้าม Restart CoreDNS หรือทุก Deployment เพื่อทดลองแก้

หากจำเป็นต้องแก้ CoreDNS ConfigMap:
- Backup ConfigMap เดิม
- ตรวจ syntax
- แสดง diff
- เปลี่ยนเพียงหนึ่งค่า
- กำหนด DNS smoke tests
- กำหนด error-rate threshold
- เตรียม restore ConfigMap เดิม
- สร้าง Change Card

หยุดรอการอนุมัติ
```

## 8. ตรวจสอบ Deployment Rollout ค้าง

```text
ปฏิบัติตาม AGENTS.md

Environment:
- Environment: production
- Kube context: prod-cluster
- Namespace: backend
- Deployment: customer-api
- Deployment method: Helm
- Helm release: customer-platform

อาการ:
Deployment rollout ค้างที่ 2/4 ready replicas หลัง Helm upgrade

เป้าหมาย:
ค้นหาสาเหตุและตัดสินใจว่าจะ Continue, Pause หรือ Rollback

ตรวจสอบ:
- kubectl rollout status
- kubectl rollout history
- Deployment conditions
- ReplicaSet เก่าและใหม่
- Pod status และ Events
- Readiness/liveness/startup probes
- Resource quota
- Scheduling failures
- PDB
- Image pull
- ConfigMap/Secret references
- Helm status และ history
- Helm values และ manifest ของ revision ปัจจุบัน
- Revision ก่อนหน้าที่ทราบว่าดี

ห้าม:
- Force delete Pod
- Scale replicas ลงเพื่อให้ rollout ดูเหมือนสำเร็จ
- Restart Deployment
- Helm rollback โดยไม่ได้รับอนุมัติ

ให้ประเมินความเสี่ยงของ:
1. รอ rollout ต่อ
2. แก้ค่าเฉพาะจุด
3. Rollback ไป revision ก่อนหน้า

หากเสนอ Helm rollback ให้ระบุ:
- Current revision
- Target revision
- Exact rollback command
- --wait และ --timeout
- Success criteria
- Abort criteria
- Post-rollback smoke test
- Backup ของ current values/manifest

สร้าง Change Card แล้วหยุดรอ APPROVE <CHANGE-ID>
```

## 9. ตรวจสอบ Cluster ก่อน Upgrade MicroK8s

```text
อ่าน AGENTS.md และทำเฉพาะ Read-only Pre-upgrade Assessment

Environment:
- Environment: production
- Kubernetes distribution: MicroK8s
- Current version: <current-version>
- Target version: <target-version>
- Kube context: microk8s-prod
- Cluster size: <number-of-nodes>
- Maintenance window: <date/time>
- Backup mechanism: <current-backup-method>

เป้าหมาย:
ประเมินความพร้อมและความเสี่ยงก่อน Upgrade MicroK8s แบบทีละ Node

ตรวจสอบ:
- Node readiness
- Kubernetes version skew
- MicroK8s channel
- Control plane topology
- dqlite หรือ etcd topology
- Cluster quorum
- Add-ons ที่เปิดใช้งาน
- CNI และ CSI
- API deprecations
- CRD compatibility
- PodDisruptionBudget
- Stateful workloads
- Local storage
- Available capacity ระหว่าง upgrade
- Certificate expiration
- Current backup และ restore procedure
- GitOps/Helm reconciliation
- Monitoring coverage

สร้าง Upgrade Plan แยก Phase:
1. Preflight
2. Backup
3. Backup validation
4. Non-production test
5. Upgrade canary node
6. Observation
7. Upgrade ทีละ Node
8. Final validation
9. Rollback/Escalation

ห้ามเริ่ม Upgrade จริง

หากไม่มี cluster-state backup, application data backup หรือ quorum-safe plan ให้รายงาน BLOCKED

สร้าง Change Card แยกต่อ Node และรออนุมัติก่อนแต่ละ Node
```

## 10. ตรวจสอบ Cluster Health แบบครบระบบ

```text
อ่าน AGENTS.md และดำเนินการ Kubernetes Health Assessment แบบ Read-only

Environment:
- Environment: production
- Kube context: prod-cluster
- Scope: all namespaces
- Kubernetes distribution: unknown
- Deployment methods: Argo CD, Helm และ raw YAML
- Maintenance window: ยังไม่มี
- Backup status: unknown

เป้าหมาย:
ค้นหาความผิดปกติ, configuration risk และ single point of failure โดยห้ามเปลี่ยนแปลง Cluster

ตรวจสอบเป็น Phase:
1. ยืนยัน context, distribution และสิทธิ์
2. Control plane และ API health
3. Node health และ resource pressure
4. Pod failures และ restart trends
5. Deployment, StatefulSet และ DaemonSet availability
6. Jobs/CronJobs failures
7. Service, EndpointSlice และ Ingress
8. DNS, CNI และ NetworkPolicy
9. PV, PVC, StorageClass และ CSI
10. Resource requests/limits, quota และ HPA
11. PDB และ maintenance readiness
12. RBAC และ ServiceAccount risks โดยไม่เปิดเผย credential
13. Helm releases และ GitOps drift
14. Backup coverage และ restore readiness

จัดผลลัพธ์ตาม:
- SEV-1
- SEV-2
- SEV-3
- SEV-4

สำหรับแต่ละ Finding ให้ระบุ:
- Resource
- Namespace
- Symptom
- Evidence
- Impact
- Likely cause
- Confidence
- Safe validation command
- Recommended remediation
- Backup requirement
- Rollback requirement

ห้ามเสนอการ Restart ทั้ง Cluster เป็นแนวทางทั่วไป

หลังจบ Assessment ให้จัดทำ Remediation Roadmap:
- Phase A: ไม่มี downtime และความเสี่ยงต่ำ
- Phase B: rolling change
- Phase C: ต้องมี maintenance window
- Phase D: stateful หรือมีความเสี่ยงต่อข้อมูล

ยังไม่ต้องสร้าง Change Card จนกว่าฉันจะเลือก Finding ที่ต้องการแก้
```

---

# Prompt สำหรับส่งผลลัพธ์กลับเข้า Agent

เมื่อ Agent ให้คำสั่งตรวจสอบมาแล้ว ให้นำ output กลับไปด้วย Prompt นี้:

```text
นี่คือผลลัพธ์จากคำสั่ง Read-only ใน Phase <หมายเลข Phase>

Command:

<คำสั่งที่ใช้>

Output:

<วาง output>

ให้วิเคราะห์ตาม AGENTS.md โดย:

1. ห้ามสรุปเกินหลักฐานที่มี
2. แยก Facts, Hypotheses และ Unknowns
3. ระบุ Severity และ Confidence
4. เชื่อมโยง timestamp, Events, logs และ resource state
5. ระบุ read-only command ถัดไปเฉพาะที่จำเป็น
6. ห้ามเสนอ write action หาก root cause ยังไม่ชัด
7. หากมีข้อมูลเพียงพอ ให้จัดทำ Backup Plan และ Draft Change Card
8. ห้ามดำเนินการ Change จนกว่าจะได้รับ APPROVE <CHANGE-ID>
```

---

# Prompt สำหรับอนุมัติ Change

ไม่ควรตอบเพียงคำว่า “ตกลง” เพราะอาจไม่ชัดว่าอนุมัติรายการใด ให้ใช้รูปแบบนี้:

```text
APPROVE <CHANGE-ID>

อนุมัติเฉพาะ Change Card นี้เท่านั้น

เงื่อนไข:
- ต้องยืนยัน Context และ Resource เป้าหมายอีกครั้งก่อนรัน
- ต้องยืนยันว่า Backup ผ่าน Validation แล้ว
- รันเฉพาะ Exact command ใน Change Card
- หยุดทันทีเมื่อเข้า Abort criteria
- หากคำสั่งจริงต่างจาก Change Card ให้หยุดและขออนุมัติใหม่
- หลังดำเนินการให้เข้าสู่ Verification Phase
- ห้ามดำเนินการ Change รายการถัดไปโดยอัตโนมัติ
```

---

# Prompt สำหรับสั่ง Rollback

```text
ROLLBACK <CHANGE-ID>

ให้หยุดการเปลี่ยนแปลงเพิ่มเติม และปฏิบัติตาม Rollback Plan ของ Change Card นี้เท่านั้น

ก่อน Rollback:
1. เก็บ Evidence ของสถานะปัจจุบัน
2. ยืนยัน Context, Namespace, Kind และ Resource name
3. ยืนยัน Backup/Revision ที่จะใช้
4. แสดง Exact rollback command

หลัง Rollback:
1. ตรวจ rollout/restore status
2. ตรวจ ready replicas และ restart count
3. ตรวจ Events
4. ทำ application smoke test
5. ตรวจ data integrity หากเป็น Stateful workload
6. เปรียบเทียบกับ Baseline
7. รายงาน PASS, FAIL หรือ BLOCKED

ห้ามทดลองวิธี Rollback อื่นหากแผนเดิมล้มเหลว ให้หยุดและ Escalate
```

---

# แนวทางการใช้งาน

Prompt เหล่านี้เพิ่มข้อมูลที่ Agent ต้องใช้ตัดสินใจจริง เช่น `Kube context`, deployment method, maintenance window, SLO, Source of Truth และ backup mechanism พร้อมกำหนด Approval Gate แบบเจาะจง

การแยก Prompt สำหรับตรวจสอบ, ส่งผลลัพธ์กลับ, อนุมัติ และ Rollback ช่วยป้องกันไม่ให้ AI ตีความบทสนทนาทั่วไปว่าเป็นคำอนุญาตให้เปลี่ยนแปลง Production Cluster

## Workflow ที่แนะนำ

1. วางไฟล์ `AGENTS.md` ไว้ที่ root ของ repository หรือ working directory ของ AI Agent
2. เริ่มด้วย **Prompt Template หลัก** หรือ **Prompt แบบสั้น**
3. ให้ Agent ทำ Phase 0 และ Phase 1 แบบ Read-only
4. ส่งผลคำสั่งกลับด้วย **Prompt สำหรับส่งผลลัพธ์กลับเข้า Agent**
5. ตรวจ Backup Plan, Diff, Dry-run, Success criteria และ Rollback Plan
6. อนุมัติด้วย `APPROVE <CHANGE-ID>` เท่านั้น
7. หากผลลัพธ์เข้า Abort criteria ให้ใช้ `ROLLBACK <CHANGE-ID>`
8. ปิดงานเมื่อ Verification ผ่านและมี Final Report พร้อมหลักฐาน
