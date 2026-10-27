# CPU Scheduling Simulator

โปรแกรมจำลองการจัดตารางเวลา CPU (CPU Scheduling) แบบ interactive บนเว็บเบราว์เซอร์ แสดง Gantt chart ของ CPU และของแต่ละ process แบบ synchronized พร้อม playhead ที่เดินตามเวลา, ready queue แบบ real-time และหน้าเปรียบเทียบประสิทธิภาพของทุก algorithm

ทั้งโปรแกรมอยู่ในไฟล์เดียว [cpu_scheduling_simulator.html](cpu_scheduling_simulator.html) ไม่ต้อง build หรือติดตั้งอะไรเพิ่ม

## วิธีรัน

เปิดไฟล์ HTML ด้วยเบราว์เซอร์ได้เลย (ต้องต่ออินเทอร์เน็ต เพราะโหลด Tailwind CSS, Chart.js และฟอนต์จาก CDN)

```bash
xdg-open cpu_scheduling_simulator.html
```

## Algorithm ที่รองรับ

| Algorithm | ประเภท | การเลือก process | ตัดสินเสมอด้วย |
|---|---|---|---|
| **FCFS** – First-Come, First-Served | Non-preemptive | มาถึงก่อนได้ก่อน | PID |
| **SJF** – Shortest-Job-First | Non-preemptive | Burst time น้อยที่สุด | Arrival time |
| **SRTF** – Shortest-Remaining-Time-First | Preemptive | Remaining time น้อยที่สุด (เช็คทุก 1 ms) | Arrival time |
| **RR** – Round Robin | Preemptive | วนตามคิว ทีละ time quantum (q) | ลำดับในคิว |
| **Priority** | Non-preemptive | ค่า priority น้อยที่สุด (= สำคัญที่สุด) | Arrival time |
| **Priority + Aging** | Preemptive | ค่า priority น้อยที่สุด (เช็คทุก 1 ms) | Arrival time |

- **Round Robin:** process ที่มาถึงระหว่าง quantum จะเข้าคิว *ก่อน* process ที่เพิ่งถูก preempt
- **Aging:** ทุกครั้งที่ process รอครบ `Aging Step` tick ค่า priority จะลดลง 1 (สูงขึ้น 1 ระดับ) ต่ำสุดที่ 0 เพื่อป้องกัน starvation ตั้งเป็น 0 เพื่อปิด

## วิธีใช้งาน

### 1. กำหนด process (PCB Table)
- แก้ PID, Arrival Time (AT), Burst Time (BT) และ Priority ได้ในตาราง
- **+ Add** เพิ่ม process, **🎲 Random** สุ่ม 5 process, **Clear** เหลือ process เดียว
- ค่า Priority ยิ่ง **น้อย** ยิ่งสำคัญ

### 2. Preset สำหรับสาธิต
| Preset | สิ่งที่แสดง |
|---|---|
| ⚡ Case 1: Convoy | Convoy effect ของ FCFS – job ยาว (P1 = 24 ms) มาก่อนทำให้ job สั้นต้องรอนาน |
| ⏱ Case 2: RR Quantum | Round Robin ที่ q = 4 กับ process ที่มาถึงไม่พร้อมกัน |
| 🛡 Case 3: Aging | Process priority ต่ำ (`P_low`) ถูก process priority สูงแซงตลอด จนกว่า aging จะช่วย |
| 🚀 Stress Test | สุ่ม 50 process เพื่อทดสอบการแสดงผลจำนวนมาก |

### 3. ตั้งค่า Dispatcher
- **Active Scheduling Policy** – เลือก algorithm
- **Time Quantum (q)** – 1–15 ms (ใช้กับ RR)
- **Aging Step** – 0–10 tick (ใช้กับ Priority)
- **"Best Algorithm" Criterion** – เกณฑ์ที่ใช้เลือก algorithm ที่ดีที่สุดอัตโนมัติ (ดูหัวข้อด้านล่าง)

### 4. เล่น simulation
| ปุ่ม | การทำงาน |
|---|---|
| **Step (+1)** | เดินไปข้างหน้า 1 ms |
| **Play / Pause** | เล่นอัตโนมัติตามความเร็วที่เลือก (0.5x / 1x / 5x / 20x) |
| **⏭ Finish** | กระโดดไปเวลาสุดท้าย |
| **↺ Reset** | กลับไปเวลา 0 |
| **− / + / Fit** | ซูม timeline |

ชี้เมาส์บนแถบใน Gantt chart เพื่อดู PID, สถานะ (RUNNING / WAITING / IDLE), ช่วงเวลา และระยะเวลา

### 5. เปรียบเทียบทุก algorithm
กด **Compare All Algorithms** เพื่อรันทั้ง 6 algorithm กับชุด process ปัจจุบัน แสดงเป็นกราฟแท่ง AWT/ATAT และตารางที่ไฮไลต์ตัวที่ดีที่สุดตามเกณฑ์ที่เลือก

## Metrics ที่คำนวณ

| Metric | สูตร |
|---|---|
| Turnaround Time (TAT) | `CT − AT` |
| Waiting Time (WT) | `TAT − BT` |
| Response Time (RT) | `เวลาที่ได้ CPU ครั้งแรก − AT` |
| CPU Utilization | `(เวลารวม − เวลา IDLE) / เวลารวม × 100%` |
| Context Switches | จำนวนครั้งที่ CPU สลับจาก process หนึ่งไปอีก process หนึ่งโดยตรง (ไม่นับการสลับเข้า/ออกจาก IDLE) |

## เกณฑ์เลือก "Best Algorithm"

| เกณฑ์ | วิธีเลือก |
|---|---|
| ⚖ **Balanced** | คะแนนต่ำสุดของ `0.4 × AWT + 0.4 × ART + 1.2 × Context Switches` |
| ⚡ **Response** | Avg Response Time ต่ำสุด (เสมอกันดูที่ AWT) |
| 🛡️ **Priority** | เลือก Priority Preemptive + Aging เสมอ |
| 📐 **Theoretical** | Avg Waiting Time ต่ำสุด (ส่วนใหญ่จะเป็น SRTF) |

เมื่อกด Random / Stress Test หรือเปลี่ยนเกณฑ์ โปรแกรมจะเลือก algorithm ที่ดีที่สุดให้อัตโนมัติ

## โครงสร้างโค้ด

ทุกอย่างอยู่ใน `<script>` ของไฟล์ HTML:

- `ProcessItem` – คลาสเก็บข้อมูล PCB ของแต่ละ process
- `simulateFCFS`, `simulateSJF_NonPreemptive`, `simulateSRTF`, `simulateRR`, `simulatePriority` – จำลองทีละ 1 ms แล้วสร้าง `gantt` (แถบ CPU) และ `trace` (สถานะ ready queue ณ แต่ละเวลา)
- `finalizeMetrics` – รวมแถบ Gantt ที่ติดกัน คำนวณค่าเฉลี่ย context switch และ balanced score
- `findOptimalAlgorithm` – รันทุก algorithm แล้วเลือกตามเกณฑ์
- `renderSyncedGantt`, `updateStepView` – วาด timeline และอัปเดต playhead / ready queue ตาม step ปัจจุบัน

## เทคโนโลยี

- HTML + Vanilla JavaScript
- [Tailwind CSS](https://tailwindcss.com/) (CDN)
- [Chart.js](https://www.chartjs.org/) สำหรับกราฟเปรียบเทียบ

## Known Issues

- แก้ค่า Burst Time หรือ Priority ในตารางแล้วผล simulation ไม่เปลี่ยน (`clone()` คัดลอก `remainingTime` / `currentPriority` ค่าเก่าไปด้วย)
- กด **+ Add** หลังลบ process อาจได้ PID ซ้ำ และไม่มีการเช็ค PID ซ้ำตอนแก้ชื่อ
- Aging ในโหมด Non-preemptive แทบไม่ทำงาน (เงื่อนไข `% agingInterval === 0`)
- เมื่อ Aging = 0 ส่วน Best Algorithm และหน้า Compare จะคำนวณด้วย aging = 2 แทน
- กด Step ครั้งแรกไม่เดิน, Finish หยุดก่อนเวลาสุดท้าย 1 ms และ Zoom ทำให้ playhead เลื่อนไป 1 tick
- Checkbox "Hide Finished" reset playback และไม่อัปเดตระหว่างเล่น
