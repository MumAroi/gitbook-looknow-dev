# Init

เริ่มต้นด้วย

```
zig init
```

zig จะทำการสร้าง file มาให้ตามนี้

```
created build.zig
created build.zig.zon
created src/main.zig
created src/root.zig
```

โดยแต่ละ file จะใช้งานในหน้าที่ต่างกัน โดยจะอธิบายคร่าวๆได้ดังนี้

* build.zig.zon: ใช้ในการจัดการ dependencies ที่อยู่ภายใน project ว่าต้องใช้อะไรบ้าง
* build.zig: ใช้ในการ config project ของเราว่า โปรเจกต์ควรถูกคอมไพล์อย่างไรและแบบไหน บน platform ใดเพื่อสร้างให้เป็น executable file&#x20;
* src/root.zig: เป็นไฟล์ root source สำหรับโมดูลไลบรารีของโปรเจกต์ หรือจะไม่เก็บในนี้ก็ได้ไม่ได้ บังคับ
* src/main.zig: เป็น main entry point สำหรับโปรแกรม executable ส่วนใหญ่จะเป็นตัว code ที่ใช้เริ่มต้น project



