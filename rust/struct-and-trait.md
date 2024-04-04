# Struct And Trait

ความแตกต่างข้อง Struct และ Trait

### **Struct**

* เป็นการกำหนดโครงสร้างข้อมูลแบบผสม (Composite Data Type) ซึ่งประกอบไปด้วยฟิลด์ (Fields) ต่างๆ
* สามารถกำหนดค่าเริ่มต้นให้กับฟิลด์ได้
* มี `Methods` เป็นของตัวเอง โดยใช้ `impl` ในการกำหนดเพิ่มเติม
* ใช้สำหรับการจัดเก็บข้อมูลที่เกี่ยวข้องกันไว้ด้วยกัน

ตัวอย่าง

```rust
struct Person {
    name: String,
    age: u32,
}

impl Person {
    fn new(name: &str, age: u32) -> Person {
        Person {
            name: name.to_string(),
            age,
        }
    }

    fn say_hello(&self) {
        println!("Hello, my name is {} and I'm {} years old.", self.name, self.age);
    }
}

fn main() {
    let john = Person::new("John", 30);
    john.say_hello(); // Hello, my name is John and I'm 30 years old.
}
```

### **Trait**

* เป็นการกำหนดพฤติกรรม (Behavior) ที่ต้องการให้มีในโครงสร้างข้อมูลนั้นๆ
* ประกอบไปด้วยเมธอดต่างๆ ที่ถูกกำหนดไว้ล่วงหน้า
* ไม่มีการกำหนดฟิลด์หรือเก็บข้อมูลโดยตรง
* สามารถนำไปใช้กับโครงสร้างข้อมูลต่างๆ ได้ โดยใช้คำสั่ง `impl Trait for Struct`
* ทำให้เกิดการแบ่งปันพฤติกรรมที่ถูกกำหนดไว้ระหว่างโครงสร้างข้อมูลต่างๆ ได้

ตัวอย่าง

```rust
trait Printable {
    fn print(&self);
}

struct Circle {
    radius: f64,
}

impl Printable for Circle {
    fn print(&self) {
        println!("This is a circle with radius {}", self.radius);
    }
}

struct Rectangle {
    width: f64,
    height: f64,
}

impl Printable for Rectangle {
    fn print(&self) {
        println!("This is a rectangle with width {} and height {}", self.width, self.height);
    }
}

fn main() {
    let c = Circle { radius: 5.0 };
    let r = Rectangle { width: 3.0, height: 4.0 };

    c.print(); // This is a circle with radius 5
    r.print(); // This is a rectangle with width 3 and height 4
}
```
