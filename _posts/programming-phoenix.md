**บทที่ 1: การแนะนำ Phoenix**

*   Phoenix เป็น Framework ที่เน้นด้านประสิทธิภาพ, การทำงานพร้อมกัน, ความสวยงาม, การโต้ตอบ, และความน่าเชื่อถือในการสร้างเว็บแอปพลิเคชัน
*   มีเครื่องมือและโครงสร้างพื้นฐานที่ครบถ้วนเพื่อช่วยให้นักพัฒนาทำงานได้อย่างมีประสิทธิภาพ
*   เน้นความสมดุลระหว่าง Productivity (ประสิทธิภาพ) และ Maintainability (การบำรุงรักษา)
*   ใช้ประโยชน์จากคุณสมบัติของ Elixir เช่น Immutability (การไม่เปลี่ยนแปลงค่า) เพื่อให้โค้ดอ่านง่ายและบำรุงรักษาง่าย
*   รองรับการทำงานแบบ Concurrent (พร้อมกัน) ได้อย่างมีประสิทธิภาพโดยใช้ Erlang VM
*   มีระบบ Routing ที่ดี, ขยายขีดความสามารถได้ง่าย, และสร้างสถาปัตยกรรมที่ยืดหยุ่น
*   มีการพัฒนาอย่างต่อเนื่องเพื่อรองรับ Interactive applications เช่น Phoenix LiveView
*   มีระบบที่เชื่อถือได้ (Reliable) โดยใช้ประโยชน์จาก Erlang/OTP เพื่อให้แอปพลิเคชันมีความทนทาน

**บทที่ 2: การทำความเข้าใจโครงสร้าง Phoenix**

*   เว็บเซิร์ฟเวอร์สามารถมองเป็นการเรียกใช้ฟังก์ชันจากระยะไกล
*   การแสดงโครงสร้าง Phoenix โดยใช้ Pipes (การส่งต่อข้อมูล) เพื่อให้เข้าใจการทำงานของแต่ละ Layer
*   แต่ละ Request จะผ่าน Endpoint > Router > Pipelines > Controller
*   Controller ทำหน้าที่จัดการ Request และประสานงานระหว่าง Model และ View
*   เน้นการติดตั้ง Development Environment ที่จำเป็น เช่น Erlang, Elixir, PostgreSQL และ Node.js
*   การสร้าง Project ใหม่และทำความเข้าใจโครงสร้าง

**บทที่ 3: Controllers**

*   ขยายความเข้าใจ Controller และส่วนประกอบที่เกี่ยวข้อง (View, Template, Context)
*   การสร้าง Context เพื่อแยก Business Logic ออกจาก Controller
*   การสร้าง Module และ Struct ที่จำเป็นสำหรับการจัดการข้อมูล
*   การสร้าง Controller ใหม่และเชื่อมต่อกับ Context
*   การ Coding View และ Template
*   การใช้ Helpers
*   ทำความเข้าใจเรื่อง Atom Keys vs. String Keys

**บทที่ 4: Ecto และ Changesets**

*   การใช้ Ecto เพื่อเชื่อมต่อกับ Database
*   การ Defining Schema และ Migration
*   การใช้ Repository เพื่อเพิ่ม Data
*   การสร้าง Forms และการจัดการ Input จากผู้ใช้
*   การ Creating Resources
*   การ Authentication Users และจัดการ Password อย่างปลอดภัย
*   การ Validate ข้อมูล

**บทที่ 5: Authenticating Users**

*   เตรียมการสำหรับ Authentication โดยการวางแผนและติดตั้ง Dependencies ที่จำเป็น
*   การจัดการ Registration Changesets
*   การสร้าง Custom Authentication Plug
*   การ Implementing Login และ Logout
*   การสร้าง User Account Links
*   ความแตกต่างระหว่าง Atom Keys กับ String Keys

**บทที่ 6: Generators และ Relationships**

*   การใช้ Generators เพื่อสร้าง Schema, Context, และ Migration
*   การ Building Relationships ระหว่าง Model (เช่น User กับ Video)
*   การ Managing Related Data
*   การ In-context Relationships
*   การ Connecting Categories with Videos

**บทที่ 7: Ecto Queries และ Constraints**

*   การ Seeding Data และการ Associating Categories
*   การใช้ Ecto Queries เพื่อดึงข้อมูลจาก Database
*   การ Seeding และการ Associating Data
*   การ Diving Deeper into Ecto Queries (Syntax, Operators, etc.)
*   การ Understanding Ecto Constraints (Uniqueness, Foreign Key, etc.)
*   การ Validating Data

**บทที่ 8: การ Testing MVC**

*   การทำความเข้าใจ ExUnit Framework สำหรับ Testing
*   หลักการของการ Testing ที่ดี (Fast, Isolated, DRY, Repeatable)
*   การสร้าง Test Fixtures
*   การ Testing Contexts
*   การใช้ Ecto Sandbox สำหรับ Test Isolation และ Concurrency
*   การ Integration Tests
*   การ Unit-Testing Plugs
*   การ Testing Views และ Templates

**บทที่ 9: Watching Videos**

*   การเตรียมพร้อมสำหรับเพิ่ม Feature การ Watching Videos
*   การใช้ JavaScript เพื่อเชื่อมต่อกับ YouTube API
*   การ Adding CSS สำหรับ UI
*   การ Implement Slug URLs
*   การ Working with Protocols ใน Elixir

**บทที่ 10: Using Channels**

*   แนะนำ Channels: การสื่อสาร Real-Time แบบใหม่
*   Channels Overview: State, Topics, and Concurrency
*   Phoenix Clients with ES6: Setting Up the Frontend
*   Preparing the Server: Sockets and Topics
*   Sending and Receiving Events: Real-Time Communication
*   Socket Authentication: Securing the Connection

**บทที่ 11: Observer และ Umbrellas**

*   Introduction to Observer: Getting Insights into a Running System
*   Deep Dive into Applications and Processes with Observer
*   Overview of Umbrellas: Modularizing Large Applications
*   Building Maintainable Codebases with Subprojects
*   Steps for Refactoring to the New Project
*   Managing the Umbrella Project Configuration

**บทที่ 12: OTP**

*   Functional programs and statelessness
*   Concurrent processes and recursion
*   Exploring Erlang and Erlang Term Storage (ETS)
*   GenServers for OTP : 
การเรียนรู้วิธีสร้าง GenServers เพื่อจัดการสถานะและการทำงานแบบพร้อมกัน
* Building a Cache Server Without Bottlenecks
การนำเสนอหลักการและขั้นตอนการสร้าง Cache Server เพื่อเพิ่มประสิทธิภาพ
* The Wolfram Info System: Combining multiple OTP: การสร้างระบบที่รองรับการทำงานหลายอย่างพร้อมกัน

**บทที่ 13: Testing Channels and OTP**
* การใช้ Phoenix Helper เพื่อการทดสอบ Channels
*Testing the Information System
 การทดสอบระบบข้อมูลและการโต้ตอบกับบริบทภายนอก
* การสร้างความมั่นใจในความปลอดภัยของซ็อกเก็ต (Test Socket) และการเชื่อมต่อ
*การจำลองและการตรวจสอบการสื่อสารโดยใช้ Channel ต่างๆ

**บทที่ 14: What's Next?**
* การพิจารณาและสำรวจคุณลักษณะที่น่าสนใจอื่น ๆ ที่อาจมีใน Phoenix Framework 
* การนำเสนอ Phoenix LiveView และการพัฒนา Web ด้วยเทคนิคใหม่ๆ
*การใช้ PubSub 2.0 และ Telemetry เพื่อประสิทธิภาพและการวัดผลลัพธ์ของระบบ
*การให้กำลังใจและแนะนำแนวทางสำหรับการเรียนรู้และพัฒนาต่อ
