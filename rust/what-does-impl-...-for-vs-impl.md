# What does \`impl ... for\`  vs \`impl\` ?

### ข้อแตกต่างระหว่าง impl และ impk ... for



การใช้คำสั่ง `impl` ใน Rust นั้นมีความแตกต่างกันอยู่นิดหน่อยดังนี้

1. `impl Struct ...`
   * เป็นการเพิ่มเติม method ใหม่ๆ เข้าไปใน struct `Struct`
   * เมธอดเหล่านี้จะสามารถใช้งานได้กับ struct `Struct` เท่านั้น และไม่สามารถใช้งานกับโครงสร้างข้อมูลประเภทอื่นๆ หรือ trait อื่นๆ ได้
2. `impl Trait for Struct ...`
   * เป็นการนำ `method` ที่อยู่ใน `Trait` มาใช้งานกับ `Struct`
   * หลังจากทำการ implement แล้ว  `Struct` จะสามารถใช้งาน `method` ต่างๆ ที่อยู่ใน  `Trait` ได้

> สรุปสั้นง่ายๆ `impl` เป็นการเพิ่ม `method` ใหม่เข้าไปใน `Struct` ส่วน `impl ... for` เป็นการนำ `method` ที่มีอยู่แล้วมาแก้ใขแล้วเพิ่มเข้าไปใน method อีกที
