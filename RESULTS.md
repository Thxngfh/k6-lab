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