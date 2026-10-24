# K6 lab results

## Part 0

Step 0.1, http_req_duration: 260.21ms
Step 0.2, requests, rate and p(95) and why the rate stays below 5: 80 requests, rate 3.78 req/s, p(95) = 278.22 ms. The request rate stays below 5 req/s because each iteration takes ~1.27s - 1.31s (HTTP request duration ~0.27s + sleep(1s)). Each VU completes ~0.78 iterations per second, so 5 VUs yield a total rate of ~3.78 req/s.
Step 0.6, exit code of the normal run and of the run with P95=300: Normal run exit code = 0, P95=300 run exit code = 99
Step 0.7, rules that passed, p(95), p(99), exit code after your change: All rules passed (p(95) = 381.81ms, p(99) = 386.54ms). Exit code after changing requirement = 99
Step 0.8, what was wrong, and how the checks showed it: The original script used lowercase 'pepperoni' and 'knife' instead of capitalized 'Pepperoni' and 'Knife', causing the API to ignore exclusions. Testing under load revealed this because repeated requests eventually generated unwanted ingredients/tools, causing checks to fail.

## Part A results

| Step | Test type    | Users | Req/s | p(95)    | Max      | Failed | Checks  |
| ---- | ------------ | ----- | ----- | -------- | -------- | ------ | ------- |
| A1   | Smoke        | 1     | 0.54  | 335.91ms | 346.67ms | 0.00%  | 100.00% |
| A2   | Average load | 3     | 1.33  | 371.77ms | 387.61ms | 0.00%  | 100.00% |
| A3   | Stress       | 5     | 2.17  | 373.24ms | 396.90ms | 0.00%  | 100.00% |
| A4   | Spike        | 5     |       |          |          |        |         |
| A5   | Soak         | 2     |       |          |          |        |         |

## Part B

1.
2.
3.
4.
5.
6.
7.
8.
9.

---

### ขั้นตอนที่ 3: ทำโจทย์ Part 0 ต่อทีละ Step

ทุกครั้งที่ทำเสร็จแต่ละ Step อย่าลืมรันคำสั่ง push งานขึ้น GitHub:

```powershell
git add .
git commit -m "Step X.X"
git push
```

- **Step 0.1:** นำค่า `http_req_duration` จากการรัน `01-first-test.js` ไปกรอกลง `RESULTS.md`
- **Step 0.2:** รันคำสั่ง `k6 run --vus 5 --duration 20s 01-first-test.js` บันทึกค่าที่ได้ลง `RESULTS.md` พร้อมอธิบายว่าทำไม Request rate ถึงไม่เกิน 5 req/s (วิเคราะห์จากเวลา `sleep(1)`)
- **Step 0.3:** รัน `k6 run --vus 5 --iterations 20 01-first-test.js`
- **Step 0.4:** สร้างไฟล์ `04-pizza-api.js` (ก็อปโค้ดจาก PDF ข้อ 0.4) แล้วรัน `k6 run 04-pizza-api.js`
- **Step 0.5:** สร้างไฟล์ `05-no-token.js` (ก็อปโค้ดจาก PDF ข้อ 0.5) แล้วรัน `k6 run 05-no-token.js`
- **Step 0.6:** สร้างไฟล์ `06-thresholds.js` (ก็อปโค้ดจาก PDF ข้อ 0.6)
  - รัน `k6 run 06-thresholds.js`
  - รัน `k6 run -e P95=300 06-thresholds.js`
  - พิมพ์คำสั่ง `$LASTEXITCODE` ใน Terminal เพื่อดู Exit code แล้วจดลง `RESULTS.md`
- **Step 0.7:** สร้างไฟล์ `07-requirement.js` กำหนด thresholds ตามโจทย์ (p95<1000, p99<1500, failed<0.01)
  - รันแล้วจดค่า p95/p99 ที่วัดได้ลง `RESULTS.md`
  - ปรับตัวเลขให้การทดสอบไม่ผ่าน รันใหม่ แล้วดูค่า `$LASTEXITCODE` บันทึกลง `RESULTS.md`
- **Step 0.8:**
  - สร้าง `08-bug-hunt.js` และ `explore.js` ตามโค้ดข้อ 0.8
  - รัน `k6 run explore.js` เพื่อดูว่า API ใช้คำว่าอะไร
  - แก้ไขคำใน `08-bug-hunt.js` ให้ถูกต้อง รัน `k6 run --vus 5 --duration 20s 08-bug-hunt.js` ให้ผ่าน 100% แล้วจดอธิบายลง `RESULTS.md`

---

### ขั้นตอนที่ 4: ทำโจทย์ Part A (User Journey)

1. สร้างไฟล์ `journey.js` (ก็อปโค้ดจาก PDF ข้อ A1)[cite: 9]
2. **Step A1:** สร้าง `a1-smoke.js` แล้วรัน `k6 run a1-smoke.js`
3. **Step A2:** สร้าง `a2-average.js` แล้วรันคำสั่งส่งออกรายงาน HTML:
   ```powershell
   k6 run --out web-dashboard=export=reports/a2-average.html a2-average.js
   ```
4. **Step A3:** สร้าง `a3-stress.js` ปรับ stages เป็น 5 users รันคำสั่ง:
   ```powershell
   k6 run --out web-dashboard=export=reports/a3-stress.html a3-stress.js
   ```
5. **Step A4:** สร้าง `a4-spike.js` กำหนด stages แบบ spike รันคำสั่ง:
   ```powershell
   k6 run --out web-dashboard=export=reports/a4-spike.html a4-spike.js
   ```
6. **Step A5:** สร้าง `a5-soak.js` (2 users ค้างไว้ 10 นาที) รันคำสั่ง:
   ```powershell
   k6 run --out web-dashboard=export=reports/a5-soak.html a5-soak.js
   ```
7. นำผลลัพธ์จากรายงาน HTML/Terminal ของข้อ A1–A5 ไปกรอกลงตารางใน `RESULTS.md`

---

### ขั้นตอนที่ 5: ตอบคำถาม Part B และส่งงาน

1. ตอบคำถามข้อ 1–9 ในไฟล์ `RESULTS.md` ให้ครบถ้วน
2. ตรวจสอบว่าในโฟลเดอร์มีไฟล์ครบทั้งหมดตามที่กำหนด (สคริปต์ `.js`, รายงาน `.html` ในโฟลเดอร์ `reports/`, และไฟล์ `RESULTS.md`)
3. พิมพ์คำสั่ง Push โค้ดทั้งหมดขึ้น GitHub:
   ```powershell
   git add .
   git commit -m "Complete all lab assignments"
   git push
   ```
4. คัดลอก URL ของ GitHub Repository (เช่น `[https://github.com/](https://github.com/)<your-username>/k6-lab`) ไปวางส่งในระบบ **MyCourses**
