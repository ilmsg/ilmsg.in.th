มาสร้างโปรเจกต์ Vite + React + TypeScript สำหรับทำ CRUD TodoList ที่เชื่อมต่อกับ Firebase Firestore กันทีละขั้นตอน

**ภาพรวมขั้นตอน:**

1.  **สร้างโปรเจกต์ Vite:** ใช้ Template React + TypeScript
2.  **ตั้งค่า Firebase:**
    *   สร้างโปรเจกต์ใน Firebase Console
    *   สร้าง Firestore Database
    *   ตั้งค่า Rules ของ Firestore (สำคัญ!)
    *   เอาค่า Configuration ของ Firebase มาใช้ในเว็บแอป
3.  **ติดตั้ง Firebase SDK:** เพิ่ม Library ของ Firebase ลงในโปรเจกต์ React
4.  **เชื่อมต่อ Firebase:** เขียนโค้ดเพื่อ Initialize Firebase ในแอป
5.  **สร้าง Interface:** กำหนดโครงสร้างข้อมูล Todo ด้วย TypeScript
6.  **เขียน Service:** สร้างฟังก์ชันสำหรับจัดการข้อมูล (CRUD) กับ Firestore
7.  **สร้าง Components:** สร้าง UI สำหรับแสดงผล, เพิ่ม, แก้ไข, ลบ Todo
8.  **จัดการ State:** ใช้ `useState`, `useEffect` เพื่อดึงและแสดงข้อมูล
9.  **เชื่อม UI กับ Service:** ทำให้ปุ่มและฟอร์มเรียกใช้ฟังก์ชัน CRUD ได้
10. **(เสริม) Styling:** ตกแต่งหน้าตาแอป

---

**ลงมือทำ:**

**1. สร้างโปรเจกต์ Vite**

เปิด Terminal หรือ Command Prompt แล้วรันคำสั่ง:

```bash
# หากใช้ npm
npm create vite@latest my-todo-app --template react-ts

# หรือ หากใช้ yarn
# yarn create vite my-todo-app --template react-ts

# หรือ หากใช้ pnpm
# pnpm create vite my-todo-app --template react-ts

# เข้าไปในโฟลเดอร์โปรเจกต์
cd my-todo-app

# ติดตั้ง Dependencies
npm install
# หรือ yarn install หรือ pnpm install
```

**2. ตั้งค่า Firebase**

*   ไปที่ [Firebase Console](https://console.firebase.google.com/)
*   **สร้างโปรเจกต์ใหม่** (Add project) หรือเลือกโปรเจกต์ที่มีอยู่
*   ในหน้า Project Overview, คลิกที่ไอคอน **เว็บ** (`</>`) เพื่อเพิ่มแอปพลิเคชันเว็บ
*   ตั้งชื่อแอป (เช่น "My Todo App") แล้วคลิก "Register app"
*   **คัดลอก** ค่า `firebaseConfig` ที่แสดงขึ้นมา เก็บไว้ก่อน (สำคัญมาก!) คุณไม่จำเป็นต้องทำตามขั้นตอน "Add Firebase SDK" ในหน้านั้น เพราะเราจะติดตั้งเองผ่าน npm/yarn
*   คลิก "Continue to console"
*   จากเมนูด้านซ้าย ไปที่ **Build > Firestore Database**
*   คลิก **Create database**
*   เลือก **Start in production mode** (แนะนำ) หรือ test mode (ง่ายกว่าตอนเริ่ม แต่ไม่ปลอดภัย) แล้วคลิก Next
*   เลือกตำแหน่ง Cloud Firestore ที่ใกล้คุณที่สุด แล้วคลิก **Enable**
*   **ตั้งค่า Rules (สำคัญมาก):**
    *   ไปที่แท็บ **Rules** ในหน้า Firestore Database
    *   **สำหรับช่วงพัฒนา (ไม่ปลอดภัยสำหรับ Production):** คุณอาจจะตั้งค่าให้อ่านเขียนได้ทั้งหมดก่อน เพื่อให้ทดสอบง่าย (เปลี่ยน `false` เป็น `true`)
    
        ```rules
        rules_version = '2';
        service cloud.firestore {
          match /databases/{database}/documents {
            // อนุญาตให้อ่านเขียนได้ทั้งหมด **ชั่วคราว** สำหรับการพัฒนาเท่านั้น
            match /{document=**} {
              allow read, write: if true;
            }
          }
        }
        ```

    *   **สำหรับ Production (แนะนำ):** ควรตั้งค่าให้เฉพาะผู้ใช้ที่ล็อกอินแล้วเท่านั้นที่เขียนได้ และอาจจะจำกัดการอ่านเขียนเฉพาะข้อมูลของตัวเอง (ต้องใช้ Firebase Authentication ร่วมด้วย) ตัวอย่างเบื้องต้นสำหรับผู้ใช้ที่ล็อกอินแล้ว:
        ```rules
        rules_version = '2';
        service cloud.firestore {
          match /databases/{database}/documents {
            // อนุญาตให้ผู้ใช้ที่ล็อกอินแล้วอ่านเขียนได้
            match /todos/{todoId} {
              allow read, write: if request.auth != null;
              // หากต้องการจำกัดให้เจ้าของข้อมูลเท่านั้น:
              // allow read, write: if request.auth != null && request.auth.uid == resource.data.userId;
            }
          }
        }
        ```    *   คลิก **Publish** เพื่อบันทึก Rules

**3. ติดตั้ง Firebase SDK**

ใน Terminal ที่อยู่ในโฟลเดอร์โปรเจกต์ (`my-todo-app`) รันคำสั่ง:

```bash
npm install firebase
# หรือ yarn add firebase หรือ pnpm add firebase
```

**4. เชื่อมต่อ Firebase**

*   สร้างไฟล์ใหม่ใน `src` ชื่อ `firebase.ts`
*   นำค่า `firebaseConfig` ที่คัดลอกไว้มาใส่ในไฟล์นี้ และ initialize Firebase:

```typescript
// src/firebase.ts
import { initializeApp } from "firebase/app";
import { getFirestore } from "firebase/firestore";

// Your web app's Firebase configuration
// ใส่ค่าที่คุณคัดลอกมาจาก Firebase Console ตรงนี้
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_AUTH_DOMAIN",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_STORAGE_BUCKET",
  messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
  appId: "YOUR_APP_ID"
};

// Initialize Firebase
const app = initializeApp(firebaseConfig);

// Initialize Cloud Firestore and get a reference to the service
const db = getFirestore(app);

export { db }; // Export ตัวแปร db เพื่อนำไปใช้ในส่วนอื่น
```

**5. สร้าง Interface**

*   สร้างโฟลเดอร์ `src/types`
*   สร้างไฟล์ `src/types/Todo.ts` เพื่อกำหนดโครงสร้างข้อมูล:

```typescript
// src/types/Todo.ts
import { Timestamp } from "firebase/firestore";

export interface Todo {
  id: string; // Firestore document ID
  text: string;
  completed: boolean;
  createdAt: Timestamp; // ใช้ Timestamp ของ Firestore เพื่อการเรียงลำดับ
}
```

**6. เขียน Service จัดการข้อมูล (CRUD)**

*   สร้างโฟลเดอร์ `src/services`
*   สร้างไฟล์ `src/services/todoService.ts`

```typescript
// src/services/todoService.ts
import { db } from '../firebase'; // Import db จากไฟล์ firebase.ts
import {
  collection,
  addDoc,
  getDocs,
  doc,
  updateDoc,
  deleteDoc,
  Timestamp,
  query,
  orderBy // Import เพิ่มเติม
} from "firebase/firestore";
import { Todo } from '../types/Todo'; // Import Interface

const todoCollectionRef = collection(db, "todos"); // อ้างอิงไปยัง Collection ชื่อ "todos"

// CREATE
export const addTodo = async (text: string): Promise<void> => {
  try {
    await addDoc(todoCollectionRef, {
      text: text,
      completed: false,
      createdAt: Timestamp.now() // เพิ่ม timestamp ตอนสร้าง
    });
    console.log("Todo added successfully!");
  } catch (error) {
    console.error("Error adding document: ", error);
    throw error; // โยน error ต่อเพื่อให้ Component จัดการได้
  }
};

// READ
export const getTodos = async (): Promise<Todo[]> => {
  try {
    // สร้าง query เพื่อเรียงข้อมูลตาม createdAt จากใหม่ไปเก่า
    const q = query(todoCollectionRef, orderBy("createdAt", "desc"));
    const querySnapshot = await getDocs(q); // ใช้ query ที่สร้างขึ้น
    const todos = querySnapshot.docs.map(doc => ({
      id: doc.id, // ดึง ID ของ Document มาด้วย
      text: doc.data().text,
      completed: doc.data().completed,
      createdAt: doc.data().createdAt
    } as Todo)); // กำหนด Type เป็น Todo
    return todos;
  } catch (error) {
    console.error("Error getting documents: ", error);
    throw error;
  }
};

// UPDATE (Toggle Completed Status)
export const toggleTodoComplete = async (id: string, currentStatus: boolean): Promise<void> => {
  try {
    const todoDocRef = doc(db, "todos", id); // อ้างอิง Document ด้วย ID
    await updateDoc(todoDocRef, {
      completed: !currentStatus // สลับค่า completed
    });
    console.log("Todo status updated!");
  } catch (error) {
    console.error("Error updating document: ", error);
    throw error;
  }
};

// UPDATE (Edit Text - เสริม)
export const updateTodoText = async (id: string, newText: string): Promise<void> => {
    try {
      const todoDocRef = doc(db, "todos", id);
      await updateDoc(todoDocRef, {
        text: newText
      });
      console.log("Todo text updated!");
    } catch (error) {
      console.error("Error updating document text: ", error);
      throw error;
    }
  };


// DELETE
export const deleteTodo = async (id: string): Promise<void> => {
  try {
    const todoDocRef = doc(db, "todos", id); // อ้างอิง Document ด้วย ID
    await deleteDoc(todoDocRef);
    console.log("Todo deleted successfully!");
  } catch (error) {
    console.error("Error deleting document: ", error);
    throw error;
  }
};
```

**7. สร้าง Components และจัดการ State ใน `App.tsx`**

*   แก้ไขไฟล์ `src/App.tsx`:

```typescript
// src/App.tsx
import React, { useState, useEffect, useCallback } from 'react';
import { Todo } from './types/Todo';
import { addTodo, getTodos, toggleTodoComplete, deleteTodo } from './services/todoService';
import './App.css'; // Import CSS สำหรับ Styling

function App() {
  const [todos, setTodos] = useState<Todo[]>([]);
  const [newTodoText, setNewTodoText] = useState('');
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  // --- Fetch Todos ---
  const fetchTodos = useCallback(async () => {
    setIsLoading(true);
    setError(null);
    try {
      const fetchedTodos = await getTodos();
      setTodos(fetchedTodos);
    } catch (err) {
      console.error("Failed to fetch todos:", err);
      setError("ไม่สามารถโหลดรายการ Todo ได้");
    } finally {
      setIsLoading(false);
    }
  }, []);

  // --- Load todos on component mount ---
  useEffect(() => {
    fetchTodos();
  }, [fetchTodos]); // ใส่ fetchTodos ใน dependency array

  // --- Handle Add Todo ---
  const handleAddTodo = async (event: React.FormEvent) => {
    event.preventDefault(); // ป้องกันการรีโหลดหน้า
    if (!newTodoText.trim()) return; // ไม่เพิ่มถ้า input ว่าง

    setIsLoading(true); // อาจจะแสดง loading เฉพาะส่วน add
    try {
      await addTodo(newTodoText);
      setNewTodoText(''); // เคลียร์ input
      await fetchTodos(); // โหลดข้อมูลใหม่หลังเพิ่มสำเร็จ
    } catch (err) {
      console.error("Failed to add todo:", err);
      setError("เกิดข้อผิดพลาดในการเพิ่ม Todo");
    } finally {
        setIsLoading(false); // สิ้นสุด loading ไม่ว่าจะสำเร็จหรือไม่
    }
  };

  // --- Handle Toggle Complete ---
  const handleToggleComplete = async (id: string, currentStatus: boolean) => {
    try {
      await toggleTodoComplete(id, currentStatus);
      // อัพเดท state ทันทีเพื่อ UX ที่ดีขึ้น หรือจะ fetch ใหม่ก็ได้
      setTodos(prevTodos =>
        prevTodos.map(todo =>
          todo.id === id ? { ...todo, completed: !currentStatus } : todo
        )
      );
      // หรือ await fetchTodos();
    } catch (err) {
      console.error("Failed to toggle todo:", err);
      setError("เกิดข้อผิดพลาดในการอัพเดทสถานะ");
       // หากต้องการ rollback state เมื่อเกิด error อาจจะต้อง fetch ใหม่
       // await fetchTodos();
    }
  };

  // --- Handle Delete Todo ---
  const handleDeleteTodo = async (id: string) => {
    // Optional: เพิ่ม confirmation dialog
    // if (!window.confirm("คุณแน่ใจหรือไม่ว่าต้องการลบรายการนี้?")) {
    //   return;
    // }

    try {
      await deleteTodo(id);
       // อัพเดท state ทันที หรือจะ fetch ใหม่
       setTodos(prevTodos => prevTodos.filter(todo => todo.id !== id));
      // หรือ await fetchTodos();
    } catch (err) {
      console.error("Failed to delete todo:", err);
      setError("เกิดข้อผิดพลาดในการลบ Todo");
       // await fetchTodos(); // Fetch ใหม่ถ้าลบไม่สำเร็จเพื่อ sync state
    }
  };

  return (
    <div className="App">
      <h1>My Todo List (React + TS + Vite + Firebase)</h1>

      {/* --- Add Todo Form --- */}
      <form onSubmit={handleAddTodo} className="add-todo-form">
        <input
          type="text"
          value={newTodoText}
          onChange={(e) => setNewTodoText(e.target.value)}
          placeholder="เพิ่มรายการใหม่..."
          disabled={isLoading} // Disable input ตอนกำลังโหลด/เพิ่ม
        />
        <button type="submit" disabled={isLoading || !newTodoText.trim()}>
          {isLoading ? 'กำลังเพิ่ม...' : 'เพิ่ม'}
        </button>
      </form>

      {/* --- Display Loading/Error --- */}
      {isLoading && <p>กำลังโหลด...</p>}
      {error && <p className="error-message">{error}</p>}

      {/* --- Todo List --- */}
      <ul className="todo-list">
        {todos.map((todo) => (
          <li key={todo.id} className={`todo-item ${todo.completed ? 'completed' : ''}`}>
            <span onClick={() => handleToggleComplete(todo.id, todo.completed)}>
              {todo.text}
            </span>
            <div className="todo-actions">
              {/* ปุ่ม Toggle อาจใช้ Checkbox แทนได้ */}
              <button onClick={() => handleToggleComplete(todo.id, todo.completed)}>
                {todo.completed ? 'ยกเลิก' : 'เสร็จ'}
              </button>
              <button onClick={() => handleDeleteTodo(todo.id)} className="delete-button">
                ลบ
              </button>
              {/* สามารถเพิ่มปุ่ม Edit ได้ตรงนี้ */}
            </div>
          </li>
        ))}
      </ul>
       {/* แสดงข้อความเมื่อไม่มีรายการ */}
       {!isLoading && !error && todos.length === 0 && (
            <p>ยังไม่มีรายการ Todo</p>
        )}
    </div>
  );
}

export default App;
```

**8. (เสริม) Styling**

*   เปิดไฟล์ `src/App.css` (หรือ `src/index.css` ถ้า import ไว้ที่ `main.tsx`) แล้วเพิ่ม CSS ตามต้องการ ตัวอย่างเบื้องต้น:

```css
/* src/App.css (ตัวอย่าง) */
body {
  font-family: sans-serif;
  background-color: #f4f4f4;
  margin: 0;
  padding: 20px;
  display: flex;
  justify-content: center;
}

.App {
  background-color: #fff;
  padding: 30px;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  width: 100%;
  max-width: 600px;
}

h1 {
  text-align: center;
  color: #333;
}

.add-todo-form {
  display: flex;
  margin-bottom: 20px;
}

.add-todo-form input[type="text"] {
  flex-grow: 1;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px 0 0 4px;
}

.add-todo-form button {
  padding: 10px 15px;
  border: none;
  background-color: #5cb85c;
  color: white;
  cursor: pointer;
  border-radius: 0 4px 4px 0;
  transition: background-color 0.2s ease;
}

.add-todo-form button:hover {
  background-color: #4cae4c;
}

.add-todo-form button:disabled {
    background-color: #cccccc;
    cursor: not-allowed;
}


.todo-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

.todo-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px;
  border-bottom: 1px solid #eee;
  transition: background-color 0.2s ease;
}

.todo-item:last-child {
  border-bottom: none;
}

.todo-item span {
  cursor: pointer;
  flex-grow: 1;
  margin-right: 10px;
}

.todo-item.completed span {
  text-decoration: line-through;
  color: #888;
}

.todo-actions button {
  margin-left: 5px;
  padding: 5px 10px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 0.9em;
}

.todo-actions .delete-button {
  background-color: #d9534f;
  color: white;
}
.todo-actions .delete-button:hover {
    background-color: #c9302c;
}

.error-message {
    color: #d9534f;
    text-align: center;
    margin-bottom: 15px;
    font-weight: bold;
}
```

**9. รันโปรเจกต์**

กลับไปที่ Terminal แล้วรันคำสั่ง:

```bash
npm run dev
# หรือ yarn dev หรือ pnpm dev
```

เปิด Browser ไปที่ URL ที่แสดงใน Terminal (ปกติคือ `http://localhost:5173` หรือใกล้เคียง) คุณควรจะเห็น Todo List ของคุณ และสามารถ เพิ่ม, ติ๊กถูก (เปลี่ยนสถานะ), และลบรายการได้ โดยข้อมูลจะถูกบันทึกและดึงมาจาก Firebase Firestore!

**ข้อควรจำและปรับปรุงเพิ่มเติม:**

*   **Security Rules:** ย้ำอีกครั้งว่า Rules ที่ตั้งค่าแบบ `allow read, write: if true;` **ไม่ปลอดภัย**สำหรับแอปจริง ควรใช้ Firebase Authentication และกำหนด Rules ให้เหมาะสม
*   **Error Handling:** โค้ดตัวอย่างมีการจัดการ Error เบื้องต้น ควรปรับปรุงให้แสดงผลหรือจัดการ Error ได้ดีขึ้นตามความต้องการของแอป
*   **Loading States:** แสดงสถานะกำลังโหลด (Loading indicator) ให้ชัดเจนระหว่างรอข้อมูลจาก Firebase เพื่อ UX ที่ดี
*   **Real-time Updates:** หากต้องการให้รายการ Todo อัพเดททันทีเมื่อมีการเปลี่ยนแปลงจากผู้ใช้อื่น (หรือจาก Firebase Console โดยตรง) ให้ศึกษาการใช้ `onSnapshot` ของ Firestore แทน `getDocs`
*   **Component Breakdown:** สำหรับแอปที่ซับซ้อนขึ้น อาจจะแยก List และ Item ออกเป็น Component ย่อย (`TodoList.tsx`, `TodoItem.tsx`)
*   **Edit Functionality:** เพิ่มความสามารถในการแก้ไขข้อความของ Todo ที่มีอยู่
*   **Authentication:** หากต้องการให้ Todo เป็นของผู้ใช้แต่ละคน ต้อง Implement ระบบ Firebase Authentication เพิ่มเติม

ขอให้สนุกกับการพัฒนาครับ! หากมีคำถามเพิ่มเติม ถามได้เลย