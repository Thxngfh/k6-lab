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

