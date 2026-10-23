# k6 Lab Results

## Step 0.1, http_req_duration:

- avg: 293.76 ms
- min: 293.76 ms
- med: 293.76 ms
- max: 293.76 ms
- p(90): 293.76 ms
- p(95): 293.76 ms

## Step 0.2, More Users for Longer:

- HTTP requests: 72
- Request rate: 3.386077 requests/s
- p(95): 661.13 ms
ถึงจะมีผู้ใช้จำลอง 5 คนทำงานตลอด 20 วินาที แต่แต่ละคนต้องรอ 1 วินาทีจากคำสั่ง sleep(1) หลังส่ง request และยังต้องรอเซิร์ฟเวอร์ตอบกลับด้วย ทำให้แต่ละรอบใช้เวลามากกว่า 1 วินาที จึงส่ง request ได้ไม่ถึง 5 requests ต่อวินาที

## Step 0.3, A fixed number of iterations:

- Command: k6 run --vus 5 --iterations 20 01-first-test.js
- iterations: 20 iterations
duration คือ การกำหนดเวลาที่ต้องการทดสอบ ส่วน iterations คือ การกำหนดจำนวนรอบที่ต้องการให้ทำงาน โดยการทดสอบนี้จะหยุดเมื่อทำครบ 20 รอบ

## Step 0.6: Thresholds

- Exit code (ปกติ, P95 = 1500 ms): 0
- Exit code (เข้มงวด, P95 = 300 ms): 99


## Step 0.7, From a requirement to thresholds

- Normal requirement:
  - p(95): 476.28 ms — Passed
  - p(99): 568.06 ms — Passed
  - Request failure rate: 0.00% — Passed
  - Checks: 100.00% — Passed

- Modified requirement:
  - Changed threshold: p(95) < 300 ms
  - Measured p(95): 537.1 ms — Failed
  - Exit code: 99

## Step 0.8: Bug Hunt

- ปัญหา: คำขอเดิมใช้ค่า pepperoni และ knife เป็นตัวพิมพ์เล็ก ทำให้ผลลัพธ์ไม่ตรงกับที่ลูกค้าต้องการ
- วิธีแก้ไข: เปลี่ยนเป็น Pepperoni และ Knife ให้ตรงกับค่าที่ API ส่งกลับมา
- ผลการทดสอบ: หลังแก้ไข Check ผ่านครบ 195/195 ครั้ง คิดเป็น 100%
การทดสอบแบบ Load ช่วยให้ตรวจสอบการทำงานหลายครั้งและพบปัญหาที่อาจไม่เจอจากการทดสอบด้วยตัวเองเพียงครั้งเดียว

## Part A results

| Step | Test type | Users | Req/s | p(95) | Max | Failed | Checks |
|---|---|---:|---:|---:|---:|---:|---:|
| A1 | Smoke | 1 | - | 373.76 ms | - | 0% | 100% (20/20) |
| A2 | Average load | 3 | - | 381.13 ms | - | 0% | 100% (312/312) |
| A3 | Stress | 5 | - | 386.48 ms | - | 0% | 100% (532/532) |
| A4 | Spike | 5 | - | 377.62 ms | 635.87 ms | 0% | 100% (200/200) |
| A5 | Soak | 2 | - | 390.08 ms | 880.05 ms | 0% | 100% (844/844) |

## Part B: Reflection

**1. Fill in the results table for Steps A1 to A5.**

กรอกข้อมูลจากผลการทดสอบ k6 ของแต่ละ Step โดยดูจำนวน Virtual Users สูงสุด, Requests per Second, p(95), Max, Failed และ Checks จากผลลัพธ์จริง

**2. In Step 0.2, why was the request rate below 5 requests per second with 5 virtual users?**

เพราะ Virtual User แต่ละคนต้องรอ 1 วินาทีหลังจากส่ง Request และยังต้องใช้เวลารอผลตอบกลับด้วย ทำให้แต่ละคนส่ง Request ได้ไม่ถึง 1 ครั้งต่อวินาทีเสมอไป ดังนั้น อัตราการส่ง Request รวมจึงต่ำกว่า 5 Requests ต่อวินาที

**3. Did the reaponse time grow from the average-load test to the stress test? Explain your answer with the operational envelope.**

ค่า p(95) เพิ่มขึ้นเล็กน้อยจาก 381.13 มิลลิวินาทีใน Average-load Test เป็น 386.48 มิลลิวินาทีใน Stress Test หรือเพิ่มขึ้น 5.35 มิลลิวินาที แต่ยังอยู่ในเกณฑ์ที่กำหนด โดยค่า p(95) ของ Home ต่ำกว่า 1,000 มิลลิวินาที และของ Recommendation ต่ำกว่า 1,500 มิลลิวินาที นอกจากนี้ Check ผ่านทั้งหมดและไม่มี HTTP Request ที่ล้มเหลว แสดงว่าระบบยังทำงานได้ดีภายใต้การทดสอบนี้

**4. In the spike test, how long did the response time take to return to normal after the spike?**

จากกราฟในรายงาน Response Time กลับมาอยู่ในระดับใกล้เคียงเดิมภายในช่วงรายงานถัดไปประมาณ 10 วินาทีหลังจากลดจำนวน Virtual Users กลับเหลือ 1 คน อย่างไรก็ตาม ยังระบุเวลาฟื้นตัวที่แน่นอนไม่ได้จากข้อมูลสรุปเพียงอย่างเดียว

**5. In the soak test,did the response time or the error rate get worse over time? Name one kind of defect that a soak test of several hours could reveal.**

ค่า p(95) เฉลี่ยในช่วง 3 นาทีแรกอยู่ที่ประมาณ 383.74 มิลลิวินาที และเพิ่มเป็น 425.24 มิลลิวินาทีในช่วง 3 นาทีสุดท้าย แต่ Response Time ไม่ได้เพิ่มขึ้นอย่างต่อเนื่อง และอัตรา HTTP Request ที่ล้มเหลวยังคงเป็น 0%

ตัวอย่างปัญหาที่ Soak Test อาจตรวจพบได้คือ Memory Leak ซึ่งเกิดจากโปรแกรมใช้หน่วยความจำเพิ่มขึ้นเรื่อย ๆ โดยไม่คืนหน่วยความจำที่ไม่จำเป็น หากปล่อยให้ระบบทำงานนานหลายชั่วโมง อาจทำให้ระบบช้าลงหรือหยุดทำงานได้

**6. Why do the thresholds in this lab use p(95) rather than the average?**

เพราะ p(95) ช่วยให้เห็นว่า Request ส่วนใหญ่ใช้เวลาตอบกลับนานแค่ไหน โดย 95% ของ Request จะตอบกลับภายในเวลาที่กำหนด ทำให้ตรวจสอบปัญหา Request ที่ตอบสนองช้าได้ดีกว่าการดูค่าเฉลี่ยเพียงอย่างเดียว

**7. Your results come from a shared public site reached over the internet. Give two reasons why the same tests against a system in a lab could give different numbers.**

1. ความล่าช้าของเครือข่าย: ความเร็วอินเทอร์เน็ตและ Network Latency อาจแตกต่างกันในแต่ละช่วงเวลา ทำให้ Response Time เปลี่ยนแปลง
2. จำนวนผู้ใช้งาน: เว็บไซต์สาธารณะอาจมีผู้ใช้หลายคนเข้าใช้งานพร้อมกัน ทำให้ Server มีภาระงานเพิ่มขึ้นและตอบสนองช้าลง

**8. Why must you never run a stress, spike or breakpoint test with hundreds of users against a site that you do not own?**

เพราะการจำลองผู้ใช้จำนวนมากอาจทำให้ Server ทำงานหนักเกินไป เว็บไซต์อาจช้าลงหรือหยุดให้บริการ และส่งผลกระทบต่อผู้ใช้งานคนอื่นได้ นอกจากนี้ การทดสอบโดยไม่ได้รับอนุญาตอาจผิดเงื่อนไขการใช้งานหรือกฎหมาย จึงควรทดสอบเฉพาะระบบที่ตนเองเป็นเจ้าของหรือได้รับอนุญาตเท่านั้น

**9. Write one vague performance requirement for an app you use every day. Rewrite it as a testable requirement, and show the K6 thresholds that check it.**

ข้อกำหนดแบบกว้าง ๆ: แอปสั่งอาหารต้องตอบสนองรวดเร็วและใช้งานได้ดี

ข้อกำหนดที่ทดสอบได้: เมื่อมีผู้ใช้จำลอง 50 คน ระบบต้องตอบกลับภายใน 1,000 มิลลิวินาทีสำหรับอย่างน้อย 95% ของ Request และมีอัตรา Request ที่ล้มเหลวต่ำกว่า 1%

ตัวอย่าง k6 Thresholds:

```javascript
export const options = {
  vus: 50,
  duration: '2m',
  thresholds: {
    http_req_duration: ['p(95)<1000'],
    http_req_failed: ['rate<0.01'],
  },
};
```
http_req_duration ใช้ตรวจสอบว่า p(95) ต่ำกว่า 1,000 มิลลิวินาที และ http_req_failed ใช้ตรวจสอบว่าอัตรา Request ที่ล้มเหลวต่ำกว่า 1% โดยตัวอย่างนี้เป็นข้อกำหนดสมมติสำหรับการทดสอบระบบ