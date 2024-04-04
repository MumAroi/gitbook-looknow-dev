# Macros

Macros ใน Rust เป็นการขยายหรือแทนที่โค้ดที่เกิดขึ้นในช่วงการคอมไพล์ (Compile time) โดยอาศัยการทำงานของ Rust Compiler เจ้าตัว Macros ถูกออกแบบมาเพื่อช่วยลดความซ้ำซ้อนของโค้ด (Code duplication) และเพิ่มความสามารถในการเขียนโค้ดที่มีประสิทธิภาพและง่ายต่อการอ่านและบำรุงรักษา

ถูกแบ่งออกเป็น   3 ประเภทหลักดังนี้

1. **Declarative Macros หรือ `macro_rules!` macros**
   * &#x20;เป็น Macros แบบง่ายๆ ที่ใช้สำหรับการขยายหรือแทนที่โค้ด
   * นิยมใช้กับการสร้าง data structures หรือการทำ pattern matching

```rust
macro_rules! calculate {
    (add $x:expr, $y:expr) => {
        $x + $y
    };
    (sub $x:expr, $y:expr) => {
        $x - $y
    };
}
```

2. **Procedural Macros หรือ Compiler Plugins**
   * เป็น Macros ที่ซับซ้อนกว่า สามารถแปลงและสร้างโค้ดใหม่ได้ในขั้นตอนการคอมไพล์
   * ถูกเขียนเป็น Function หรือ Crate ภายนอก
   * ใช้สำหรับ Code generation, Syntax extension, Optimization

```rust
use derive_builder::Builder;
#[derive(Builder)]
struct Person { ... }
```

3. **Built-in Macros**
   * เป็น Macros ที่ถูกสร้างและใช้งานโดยตัว Rust Compiler
   * ใช้สำหรับจัดการกับการประมวลผลในขั้นตอนการคอมไพล์
   * ตัวอย่าง: `println!`, `vec!`, `assert_eq!`, `concat!`

> Macros ช่วยให้การเขียนโค้ดใน Rust มีความยืดหยุ่นและสามารถปรับแต่งได้มากยิ่งขึ้น โดยเฉพาะอย่างยิ่ง Procedural Macros ที่สามารถขยายขอบเขตของภาษาได้อย่างกว้างขวาง ทำให้ Rust เป็นภาษาที่มีประสิทธิภาพและแข็งแกร่งอย่างมาก
