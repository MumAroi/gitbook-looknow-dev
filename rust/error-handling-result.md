# Error Handling Result

ใน Rust จะมีเมธอด  `.unwrap()` `.expect()` `.ok()`  และ  `?` ทั้ งสามนี้ใช้จัดการกับ `Result` หรือใช้ในการตรวจสอบข้อผิดพลาด (Error Handling) โดยมีความแตกต่างกันดังนี้

1.  `.unwrap()`

    * จะแปลงค่า `Result` เป็น `T` (ค่าที่ต้องการ) หากผลลัพธ์เป็น `Ok(value)`
    * แต่หากผลลัพธ์เป็น `Err(err)` มันจะทำให้โปรแกรมหยุดการทำงานและแสดงข้อความของ `err` ที่เกิดขึ้น
    * ใช้เมื่อมั่นใจว่าจะไม่เกิด error หรือใช้ในขณะที่พัฒนาเท่านั้น ไม่ควรใช้ใน production code

    Example

```rust
// function to find a user by their username which return an Option enum
fn get_user(username: &str) -> Option<&str> {
    if username.is_empty() {
        return None;
    }

    return Some(username);
}

fn main() {
    // use of unwrap method to get the result of Option enum from get_user function
    let result = get_user("Hari").unwrap();


    // print the result
    println!("user = {:?}", result);
}
```

1. `expect()`&#x20;
   * มีความคล้ายคลึงกับ `unwrap()` แต่มีการเพิ่มข้อความ `panic` แบบกำหนดเองเป็นอาร์กิวเมนต์ได้
2. `.ok()`
   * จะแปลงค่า `Result` เป็น `Option<T>` โดยที่
     * `Ok(value)` จะกลายเป็น `Some(value)`
     * `Err(_)` จะกลายเป็น `None`
   * มีประโยชน์เมื่อต้องการตรวจสอบค่า error โดยไม่ให้โปรแกรมหยุดทำงาน
3. `?` (оperator)
   * &#x20;จะขยายผลลัพธ์ของ `Result` โดย
     * `Ok(value)` จะคืนค่า `value`
     * `Err(err)` จะหยุดการทำงานของฟังก์ชันปัจจุบันและส่งค่า `err` ไปยังจุดเรียกใช้ฟังก์ชัน
   * ใช้เพื่อผนวกการจัดการ error ให้อยู่ในจุดเดียวกัน ทำให้รหัสสะอาดและอ่านง่ายขึ้น
   * `?` สามารถใช้ได้ในฟังก์ชันที่คืนค่าเป็น `Result`

โดยสรุปแล้ว `.unwrap()` ใช้เมื่อมั่นใจว่าจะไม่เกิด error, `.ok()` ใช้เมื่อต้องการตรวจสอบค่า error โดยไม่ต้องการให้โปรแกรมหยุดการทำงาน และ `?` ใช้สำหรับจัดการ error ให้อยู่ในจุดเดียวกัน
