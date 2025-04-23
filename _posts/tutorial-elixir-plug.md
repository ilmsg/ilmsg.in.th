## บทเรียน Plug: สร้าง Web Application แบบโมดูลาร์ใน Elixir

บทเรียนนี้จะพาคุณไปทำความรู้จักกับ Plug, library ที่สำคัญสำหรับสร้าง Web Application ใน Elixir อย่างเป็นระบบและโมดูลาร์ โดยเราจะศึกษาตั้งแต่พื้นฐาน, การใช้งาน, และตัวอย่างจริง

**เป้าหมาย:**

* เข้าใจหลักการทำงานของ Plug และวิธีการสร้าง Plug Modules
* สามารถสร้าง Web Application ง่ายๆ โดยใช้ Plug
* รู้จักกับ Plug.Builder และการ pipeline Plugs
* เข้าใจความแตกต่างระหว่าง Conn และ Plug.Conn

**ความรู้พื้นฐาน:**

* ความรู้พื้นฐานเกี่ยวกับภาษา Elixir
* ความเข้าใจเบื้องต้นเกี่ยวกับ HTTP Request/Response Cycle

**บทที่ 1: Plug คืออะไร?**

Plug คือ specification สำหรับ composable modules (โมดูลที่สามารถนำมาประกอบรวมกันได้) ที่ทำงานระหว่าง Web Server (เช่น Cowboy) และ Web Application ของคุณ

**ลักษณะเด่นของ Plug:**

* **Composable:** สามารถนำมาต่อกันเป็น pipeline เพื่อจัดการ request
* **Testable:** สามารถทดสอบแต่ละ Plug module ได้อย่างอิสระ
* **Minimal:** ให้ความยืดหยุ่นสูงในการสร้าง Web Application

**Conn คืออะไร?**

Conn (มาจาก Connection) คือ data structure ที่เก็บข้อมูลทั้งหมดเกี่ยวกับ HTTP Request และ Response ในขณะที่ request เดินทางผ่าน Plug pipeline

**บทที่ 2: สร้าง Plug Module อย่างง่าย**

1. **สร้างโปรเจ็กต์ Elixir:**

   ```bash
   mix new my_plug_app --sup
   cd my_plug_app
   ```

2. **สร้าง Plug Module:**

   สร้างไฟล์ `lib/my_plug_app/hello_plug.ex` และใส่โค้ด:

   ```elixir
   defmodule MyPlugApp.HelloPlug do
     import Plug.Conn

     def init(options) do
       options # เราสามารถใช้ options ที่ส่งเข้ามาใน init() ได้
     end

     def call(conn, _opts) do
       conn
       |> put_resp_content_type("text/plain")
       |> send_resp(200, "Hello, Plug!")
       |> halt() # สำคัญ! ต้อง halt() เพื่อหยุดการทำงานของ Plug chain
     end
   end
   ```

   **อธิบาย:**

   * `import Plug.Conn`: นำ function จาก `Plug.Conn` มาใช้, เช่น `put_resp_content_type` และ `send_resp`
   * `init/1`:  เป็น callback function ที่ถูกเรียกเมื่อ Plug ถูกเริ่มต้น (ใช้สำหรับ config)
   * `call/2`: เป็น callback function ที่รับ `conn` และ `opts` และต้อง return `conn` ที่ถูกแก้ไข

3. **แก้ไข `lib/my_plug_app/application.ex` เพื่อใช้ Cowboy:**

   แก้ไข `start/2` function ใน `lib/my_plug_app/application.ex` เพื่อให้เริ่ม Web Server ด้วย Cowboy และ Plug ของเรา:

   ```elixir
   defmodule MyPlugApp.Application do
     use Application

     def start(_type, _args) do
       children = [
         # Start the Telemetry supervisor
         MyPlugAppWeb.Telemetry,
         # Start the Ecto repository
         MyPlugApp.Repo,
         # Start the PubSub system
         {Phoenix.PubSub, name: MyPlugApp.PubSub},
         # Start the Endpoint (http/https server)
         {Plug.Cowboy, scheme: :http, plug: MyPlugApp.HelloPlug, options: [port: 4000]}
       ]

       # See https://hexdocs.pm/elixir/Supervisor.html
       # for other strategies and supported options
       opts = [strategy: :one_for_one, name: MyPlugApp.Supervisor]
       Supervisor.start_link(children, opts)
     end

     # ...
   end
   ```

   **อธิบาย:**

   * เราเพิ่ม `{Plug.Cowboy, ...}` ลงใน `children` ของ Supervisor
   * `scheme: :http` บอกว่าเราใช้ HTTP
   * `plug: MyPlugApp.HelloPlug` กำหนด Plug ที่เราสร้าง
   * `options: [port: 4000]` กำหนด port ที่ server จะ listen

4. **รัน Application:**

   ```bash
   mix run --no-halt
   ```

   จากนั้นเปิด browser ไปที่ `http://localhost:4000` คุณควรเห็น "Hello, Plug!"

**บทที่ 3: Plug.Builder และ Plug Pipeline**

`Plug.Builder` ช่วยให้เราสร้าง Plug pipelines ได้ง่ายขึ้น โดยการต่อ Plug หลายตัวเข้าด้วยกัน

1. **สร้าง Plug Modules เพิ่มเติม:**

   สร้างไฟล์ `lib/my_plug_app/log_plug.ex`:

   ```elixir
   defmodule MyPlugApp.LogPlug do
     import Plug.Conn

     def init(options) do
       options
     end

     def call(conn, _opts) do
       IO.puts "Request received at: #{DateTime.utc_now()}"
       conn
     end
   end
   ```

   สร้างไฟล์ `lib/my_plug_app/params_plug.ex`:

   ```elixir
   defmodule MyPlugApp.ParamsPlug do
     import Plug.Conn

     def init(options) do
       options
     end

     def call(conn, _opts) do
       IO.puts "Params: #{inspect conn.params}"
       conn
     end
   end
   ```

2. **แก้ไข `lib/my_plug_app/endpoint.ex` (สร้างถ้ายังไม่มี):**

   ```elixir
   defmodule MyPlugApp.Endpoint do
     use Plug.Builder

     plug(MyPlugApp.LogPlug)
     plug(MyPlugApp.ParamsPlug)
     plug(MyPlugApp.HelloPlug) # HelloPlug สุดท้ายเพื่อให้ส่ง response

     def init(_opts) do
       :ok
     end
   end
   ```

   **อธิบาย:**

   * `use Plug.Builder` บอกว่าเราจะใช้ `Plug.Builder` ในการสร้าง pipeline
   * `plug(MyPlugApp.LogPlug)` เพิ่ม `LogPlug` เข้าไปใน pipeline
   * `plug(MyPlugApp.ParamsPlug)` เพิ่ม `ParamsPlug` เข้าไปใน pipeline
   * `plug(MyPlugApp.HelloPlug)` เพิ่ม `HelloPlug` เข้าไปใน pipeline (response)

3. **แก้ไข `lib/my_plug_app/application.ex` เพื่อใช้ Endpoint:**

   ```elixir
   # ... (เหมือนเดิมจากบทที่ 2)

         # Start the Endpoint (http/https server)
         {Plug.Cowboy, scheme: :http, plug: MyPlugApp.Endpoint, options: [port: 4000]}

   # ...
   ```

   เปลี่ยน `plug: MyPlugApp.HelloPlug` เป็น `plug: MyPlugApp.Endpoint`

4. **รัน Application:**

   ```bash
   mix run --no-halt
   ```

   ลองเปิด browser ไปที่ `http://localhost:4000?name=John&age=30`

   คุณจะเห็น:

   * "Hello, Plug!" ใน browser
   * ข้อความ log ใน console ที่แสดงเวลา request และ parameters ที่ส่งมา

**บทที่ 4: การจัดการ Route ด้วย Plug.Router**

`Plug.Router` ช่วยให้เราจัดการ routes ของ Web Application ได้อย่างมีประสิทธิภาพ

1. **ติดตั้ง `plug_cowboy` และ `plug_router`:**

   แก้ไข `mix.exs` และเพิ่ม dependencies:

   ```elixir
   def deps do
     [
       {:plug_cowboy, "~> 2.0"},
       {:plug_router, "~> 3.0"}
     ]
   end
   ```

   จากนั้นรัน:

   ```bash
   mix deps.get
   ```

2. **สร้าง Router:**

   สร้างไฟล์ `lib/my_plug_app/router.ex`:

   ```elixir
   defmodule MyPlugApp.Router do
     use Plug.Router

     plug(:match)
     plug(:dispatch)

     get "/" do
       send_resp(conn, 200, "Hello, World!")
     end

     get "/hello/:name" do
       name = conn.params["name"]
       send_resp(conn, 200, "Hello, #{name}!")
     end

     match _ do
       send_resp(conn, 404, "Not Found")
     end
   end
   ```

   **อธิบาย:**

   * `use Plug.Router` บอกว่าเราจะใช้ `Plug.Router`
   * `plug(:match)` ทำหน้าที่จับคู่ request กับ route
   * `plug(:dispatch)` ทำหน้าที่เรียก handler ของ route ที่จับคู่ได้
   * `get "/"` กำหนด route สำหรับ GET request ที่ `/`
   * `get "/hello/:name"` กำหนด route สำหรับ GET request ที่ `/hello/:name` (`:name` คือ parameter)
   * `match _` เป็น catch-all route สำหรับ routes ที่ไม่ match

3. **แก้ไข `lib/my_plug_app/application.ex` เพื่อใช้ Router:**

   ```elixir
   # ... (เหมือนเดิมจากบทที่ 2)

         # Start the Endpoint (http/https server)
         {Plug.Cowboy, scheme: :http, plug: MyPlugApp.Router, options: [port: 4000]}

   # ...
   ```

   เปลี่ยน `plug: MyPlugApp.Endpoint` เป็น `plug: MyPlugApp.Router`

4. **รัน Application:**

   ```bash
   mix run --no-halt
   ```

   ลองเปิด browser ไปที่:

   * `http://localhost:4000/` จะเห็น "Hello, World!"
   * `http://localhost:4000/hello/John` จะเห็น "Hello, John!"
   * `http://localhost:4000/something_else` จะเห็น "Not Found"

**บทที่ 5:  Conn vs Plug.Conn**

* **Conn:** คือ type ที่ phoenix ใช้ ซึ่งพัฒนาต่อจาก Plug.Conn เพื่อให้ครอบคลุม feature ที่ phoenix ต้องการ
* **Plug.Conn:** คือโครงสร้างข้อมูลพื้นฐานที่ Plug กำหนด และมีอยู่ทั่วไปใน ecosystem ของ Plug

ดังนั้นถ้าคุณใช้ Plug เพียงอย่างเดียวโดยไม่ได้ใช้ Phoenix คุณก็จะใช้แค่ `Plug.Conn`

**สรุป:**

Plug เป็นเครื่องมือที่ทรงพลังในการสร้าง Web Application ที่โมดูลาร์และ testable ใน Elixir ด้วยการใช้ Plug.Builder และ Plug.Router คุณสามารถสร้าง application ที่ซับซ้อนได้อย่างมีโครงสร้าง

**แบบฝึกหัดเพิ่มเติม:**

* สร้าง Plug ที่ authentication requests โดยตรวจสอบ header หรือ cookies
* สร้าง Plug ที่ compress response body ด้วย gzip
* สร้าง API endpoint ที่ return JSON response

**แหล่งข้อมูลเพิ่มเติม:**

* Plug Documentation: [https://hexdocs.pm/plug/readme.html](https://hexdocs.pm/plug/readme.html)
* Plug.Builder: [https://hexdocs.pm/plug/Plug.Builder.html](https://hexdocs.pm/plug/Plug.Builder.html)
* Plug.Router: [https://hexdocs.pm/plug_router/Plug.Router.html](https://hexdocs.pm/plug_router/Plug.Router.html)
* Understanding Plugs in Elixir: [https://medium.com/@sean_keane/understanding-plugs-in-elixir-5d6e1604d887](https://medium.com/@sean_keane/understanding-plugs-in-elixir-5d6e1604d887)

ขอให้สนุกกับการเรียนรู้ Plug!  และอย่าลืมทดลองทำแบบฝึกหัดเพื่อเสริมสร้างความเข้าใจให้มากยิ่งขึ้น
