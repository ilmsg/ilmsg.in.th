https://www.youtube.com/watch?v=w2h54xz6Ndw&pp=ygUVbmV4dCBqcyBhdXRoIHR1dG9yaWFs

บทเรียนอย่างละเอียดเกี่ยวกับ NextAuth.js โดยอิงตามเนื้อหาวิดีโอของ Dave Gray ซึ่งเน้นการใช้งานกับ Next.js App Router

**บทเรียน NextAuth.js: การยืนยันตัวตนใน Next.js App Router (ตามวิดีโอ Dave Gray)**

**วัตถุประสงค์:**

*   ติดตั้งและกำหนดค่า NextAuth.js ในโปรเจกต์ Next.js ที่ใช้ App Router
*   ใช้งาน Authentication Providers พื้นฐาน: Credentials (ชื่อผู้ใช้/รหัสผ่าน) และ GitHub OAuth
*   จัดการข้อมูลลับ (Secrets) โดยใช้ Environment Variables
*   ป้องกันเส้นทาง (Route Protection) สำหรับ Server Components และ Client Components โดยใช้วิธีต่างๆ
*   ใช้งาน Middleware เพื่อป้องกันเส้นทาง
*   แสดงข้อมูลผู้ใช้จาก Session ใน Components

**สิ่งที่ต้องมี:**

*   ความเข้าใจพื้นฐานเกี่ยวกับ React และ Next.js (โดยเฉพาะ App Router, Server Components, Client Components, Route Handlers)
*   ความรู้พื้นฐาน JavaScript/TypeScript
*   Node.js และ npm (หรือ yarn/pnpm) ติดตั้งในเครื่อง
*   Text Editor (แนะนำ VS Code)
*   Terminal หรือ Command Prompt
*   บัญชี GitHub (สำหรับส่วน OAuth)
*   (ทางเลือก) VS Code Extension: CodiumAI (ตามที่ผู้สอนแนะนำ)

**ขั้นตอนการเรียนรู้:**

**1. การตั้งค่าโปรเจกต์เบื้องต้น (Timestamp ~1:38)**

*   **สร้างโปรเจกต์ Next.js:**
    ```bash
    npx create-next-app@latest next-auth-tut --ts --tailwind --eslint --app --src-dir --use-npm --import-alias "@/*"
    ```
    *(ผู้สอนเลือก Yes ทั้งหมดในการตั้งค่า - คำสั่งนี้จำลองการตั้งค่านั้น โดยใช้ TypeScript, Tailwind, ESLint, App Router, `src` directory, npm, และ import alias)*
*   **เข้าสู่ไดเรกทอรีโปรเจกต์:**
    ```bash
    cd next-auth-tut
    ```
*   **ติดตั้ง NextAuth.js:**
    ```bash
    npm i next-auth
    ```
*   **(เสริม) ตรวจสอบเวอร์ชัน Next.js:** ผู้สอนใช้เวอร์ชัน 13.4.7 หากต้องการผลลัพธ์เหมือนเป๊ะ อาจต้องระบุเวอร์ชันตอนติดตั้ง Next.js หรือปรับโค้ดตามเวอร์ชันล่าสุด

**2. การจัดการ Environment Variables (ข้อมูลลับ) (Timestamp ~6:16, 10:15)**

*   **ความสำคัญ:** ไม่ควรเก็บข้อมูลลับ (API Keys, Secrets) ไว้ในโค้ดโดยตรง ต้องใช้ Environment Variables
*   **สร้างไฟล์ `.env.local`:** ที่ Root ของโปรเจกต์ (ระดับเดียวกับ `package.json`)
*   **สร้าง NEXTAUTH_SECRET:** ใช้สำหรับเข้ารหัส JWT/Session Cookie เปิด Terminal แล้วรัน:
    ```bash
    openssl rand -base64 32
    ```
    คัดลอกค่าที่ได้ไปใส่ใน `.env.local` ดังนี้:
    ```
    NEXTAUTH_SECRET=ค่าที่คัดลอกมา
    ```
*   **สำคัญ:** เพิ่ม `.env.local` ลงในไฟล์ `.gitignore` เพื่อป้องกันไม่ให้ข้อมูลลับหลุดขึ้น Git Repository

**3. การตั้งค่า GitHub OAuth Provider (Timestamp ~11:51)**

*   ไปที่ GitHub -> Settings -> Developer settings -> OAuth Apps -> New OAuth App
*   **Application name:** ตั้งชื่อ (เช่น `nextauth-tut`)
*   **Homepage URL:** `http://localhost:3000` (สำหรับตอนพัฒนา)
*   **Authorization callback URL:** `http://localhost:3000/api/auth/callback/github` (สำคัญมาก ต้องตรงตามนี้)
*   คลิก Register application
*   **คัดลอก Client ID:** นำไปใส่ใน `.env.local`:
    ```
    GITHUB_ID=ค่าClient IDที่คัดลอกมา
    ```
*   **สร้าง Client Secret:** คลิก "Generate a new client secret" -> คัดลอกค่า Secret ที่ได้ *ทันที* (จะแสดงแค่ครั้งเดียว) -> นำไปใส่ใน `.env.local`:
    ```
    GITHUB_SECRET=ค่าClient Secretที่คัดลอกมา
    ```

**4. สร้าง NextAuth API Route Handler (Timestamp ~3:28, 4:55)**

*   ใน Next.js App Router เราใช้ Route Handlers แทน API Routes แบบเดิม
*   **สร้างโครงสร้างโฟลเดอร์:** ใน `src/app/` สร้าง `api/auth/[...nextauth]/`
*   **สร้างไฟล์ `route.ts`:** ภายในโฟลเดอร์ `[...nextauth]` (`src/app/api/auth/[...nextauth]/route.ts`)
*   **โค้ดเบื้องต้นใน `route.ts`:**
    ```typescript
    // src/app/api/auth/[...nextauth]/route.ts
    import NextAuth from "next-auth"
    import { options } from "./options" // เดี๋ยวเราจะสร้างไฟล์นี้

    const handler = NextAuth(options)

    export { handler as GET, handler as POST }
    ```

**5. สร้างไฟล์กำหนดค่า NextAuth (`options.ts`) (Timestamp ~4:55, 9:02)**

*   เพื่อความเป็นระเบียบ สร้างไฟล์ `options.ts` ไว้ที่เดียวกับ `route.ts` (`src/app/api/auth/[...nextauth]/options.ts`)
*   **โค้ดเบื้องต้นใน `options.ts`:**
    ```typescript
    // src/app/api/auth/[...nextauth]/options.ts
    import type { NextAuthOptions } from 'next-auth'
    import GitHubProvider from 'next-auth/providers/github'
    import CredentialsProvider from 'next-auth/providers/credentials'

    export const options: NextAuthOptions = {
        providers: [
            GitHubProvider({
                clientId: process.env.GITHUB_ID as string,
                clientSecret: process.env.GITHUB_SECRET as string,
            }),
            CredentialsProvider({
                name: "Credentials",
                credentials: {
                    username: {
                        label: "Username:",
                        type: "text",
                        placeholder: "your-cool-username"
                    },
                    password: {
                        label: "Password:",
                        type: "password",
                        placeholder: "your-awesome-password"
                    }
                },
                async authorize(credentials) {
                    // นี่คือส่วนที่ต้องดึงข้อมูลผู้ใช้จาก Database มาตรวจสอบจริง
                    // ในตัวอย่างนี้ ใช้การ hardcode เพื่อการสาธิต
                    const user = { id: "42", name: "Dave", password: "password1" } // ห้ามใช้ password จริงแบบนี้!

                    if (credentials?.username === user.name && credentials?.password === user.password) {
                        return user // ถ้าข้อมูลถูกต้อง คืนค่า object ผู้ใช้
                    } else {
                        return null // ถ้าข้อมูลไม่ถูกต้อง คืนค่า null
                    }
                }
            })
        ],
        // (ทางเลือก) สามารถกำหนดหน้า custom sign-in/error/etc. ได้ที่นี่
        // pages: {
        //    signIn: '/signin',
        // }
    }
    ```
*   **อธิบายโค้ด:**
    *   Import `NextAuthOptions` และ Providers ที่ต้องการ (GitHub, Credentials)
    *   สร้าง `options` object ที่มี type เป็น `NextAuthOptions`
    *   กำหนด `providers` เป็น array:
        *   **GitHubProvider:** ใส่ `clientId` และ `clientSecret` จาก `.env` (ใช้ `as string` เพื่อบอก TypeScript)
        *   **CredentialsProvider:**
            *   `name`: ชื่อที่จะแสดงบนหน้า sign in
            *   `credentials`: กำหนดฟิลด์สำหรับฟอร์ม (username, password) พร้อม label, type, placeholder
            *   `authorize`: ฟังก์ชัน async ที่รับ `credentials` ที่ผู้ใช้กรอกเข้ามา -> **ส่วนนี้สำคัญมาก** ในแอปจริง คุณต้องเขียน logic เพื่อ:
                1.  ดึงข้อมูลผู้ใช้จากฐานข้อมูลของคุณโดยใช้ `credentials.username`
                2.  เปรียบเทียบรหัสผ่านที่ผู้ใช้กรอก (จาก `credentials.password`) กับรหัสผ่านที่ Hashed ไว้ในฐานข้อมูล (ใช้ library อย่าง bcrypt)
                3.  ถ้าข้อมูลถูกต้อง ให้ return object ที่มีข้อมูลผู้ใช้ (อย่างน้อยต้องมี `id` และ `name` หรือ `email`)
                4.  ถ้าไม่ถูกต้อง ให้ return `null` หรือ throw Error

**6. สร้าง Session Provider (React Context) (Timestamp ~34:03)**

*   เพื่อให้ Client Components เข้าถึงข้อมูล Session ได้ ต้องใช้ `SessionProvider`
*   **สร้างไฟล์ `AuthProvider.tsx`:** แนะนำให้สร้างในโฟลเดอร์ `context` (เช่น `src/context/AuthProvider.tsx`)
*   **โค้ดใน `AuthProvider.tsx`:**
    ```typescript
    // src/context/AuthProvider.tsx
    'use client' // ★★★ สำคัญ: ต้องเป็น Client Component ★★★

    import { SessionProvider } from "next-auth/react"

    type Props = {
        children?: React.ReactNode
    }

    export default function AuthProvider({ children }: Props) {
        return (
            <SessionProvider>
                {children}
            </SessionProvider>
        )
    }
    ```
*   **นำ `AuthProvider` ไปใช้งานใน `layout.tsx`:**
    ```typescript
    // src/app/layout.tsx
    import './globals.css'
    import type { Metadata } from 'next'
    import { Inter } from 'next/font/google'
    import AuthProvider from '../context/AuthProvider' // Import path อาจต่างกัน
    import Navbar from '../components/Navbar' // สมมติว่ามี Navbar

    const inter = Inter({ subsets: ['latin'] })

    export const metadata: Metadata = {
        title: 'NextAuth Tutorial',
        description: 'Learn NextAuth.js by Dave Gray',
    }

    export default function RootLayout({
        children,
    }: {
        children: React.ReactNode
    }) {
        return (
            <html lang="en">
                <body className={inter.className}>
                    <AuthProvider> {/* ★★★ หุ้มด้วย AuthProvider ★★★ */}
                        <Navbar />
                        <main className="flex justify-center items-start p-6 min-h-screen">
                            {children}
                        </main>
                    </AuthProvider>
                </body>
            </html>
        )
    }
    ```

**7. การป้องกันเส้นทาง (Route Protection)**

*   **วิธีที่ 1: Server Component - Conditional Rendering (หน้า Home) (Timestamp ~29:18)**
    *   ใน `src/app/page.tsx`:
    ```typescript
    // src/app/page.tsx
    import { options } from "./api/auth/[...nextauth]/options"
    import { getServerSession } from "next-auth/next"
    import UserCard from "../components/UserCard" // Component แสดงข้อมูลผู้ใช้ (ตัวอย่าง)

    export default async function Home() {
        const session = await getServerSession(options) // ดึง session ฝั่ง server

        return (
            <>
                {session ? ( // ★★★ ตรวจสอบว่ามี session หรือไม่ ★★★
                    <UserCard user={session?.user} pagetype={"Home"} />
                ) : (
                    <h1 className="text-5xl">You Shall Not Pass!</h1>
                )}
            </>
        )
    }
    ```
*   **วิธีที่ 2: Server Component - Redirect (หน้า Server) (Timestamp ~30:12)**
    *   สร้างโฟลเดอร์ `src/app/server/` และไฟล์ `page.tsx` ข้างใน
    *   ใน `src/app/server/page.tsx`:
    ```typescript
    // src/app/server/page.tsx
    import { options } from "../api/auth/[...nextauth]/options"
    import { getServerSession } from "next-auth/next"
    import UserCard from "@/components/UserCard" // ใช้ alias ถ้าตั้งไว้
    import { redirect } from "next/navigation" // Import redirect

    export default async function ServerPage() {
        const session = await getServerSession(options)

        if (!session) { // ★★★ ถ้าไม่มี session ★★★
            redirect('/api/auth/signin?callbackUrl=/server') // ★★★ Redirect ไปหน้า signin พร้อม callback ★★★
        }

        return (
            <section className="flex flex-col gap-6">
                <UserCard user={session?.user} pagetype={"Server"} />
            </section>
        )
    }
    ```
*   **วิธีที่ 3: Client Component - `useSession` Hook (หน้า Client) (Timestamp ~35:25)**
    *   สร้างโฟลเดอร์ `src/app/client/` และไฟล์ `page.tsx` ข้างใน
    *   ใน `src/app/client/page.tsx`:
    ```typescript
    // src/app/client/page.tsx
    'use client' // ★★★ สำคัญ: ต้องเป็น Client Component ★★★
    // Remember you must use an AuthProvider for client components to useSession
    import { useSession } from 'next-auth/react'
    import { redirect } from 'next/navigation' // Redirect ยังใช้ได้
    import UserCard from '@/components/UserCard'

    export default function ClientPage() {
        const { data: session } = useSession({ // ★★★ ใช้ useSession ★★★
            required: true, // ★★★ กำหนดว่าต้องมี session ★★★
            onUnauthenticated() { // ★★★ Callback เมื่อไม่มี session ★★★
                redirect('/api/auth/signin?callbackUrl=/client')
            }
        })

        // ถ้า required: true โค้ดส่วนล่างนี้จะรันเมื่อมี session เท่านั้น
        return (
            <section className="flex flex-col gap-6">
                <UserCard user={session?.user} pagetype={"Client"} />
            </section>
        )
    }
    ```
*   **วิธีที่ 4: Middleware (หน้า Extra และทุกหน้า/หน้าที่ระบุ) (Timestamp ~26:27, 27:41)**
    *   สร้างไฟล์ `middleware.ts` ที่ **Root ของโปรเจกต์** (ระดับเดียวกับ `src` และ `app`) หรือ *ภายใน `src`* (ถ้าใช้ `src` directory) -> **ต้องอยู่ระดับเดียวกับ `app` directory**
    *   **โค้ดใน `middleware.ts` (ป้องกันทุกหน้า):**
        ```typescript
        // middleware.ts (หรือ src/middleware.ts)
        // แบบนี้จะป้องกันทุกหน้าในโปรเจกต์ (ยกเว้น /api/auth/* โดยอัตโนมัติ)
        export { default } from "next-auth/middleware"
        ```
    *   **โค้ดใน `middleware.ts` (ป้องกันเฉพาะหน้าที่ระบุ):**
        ```typescript
        // middleware.ts (หรือ src/middleware.ts)
        import { withAuth } from "next-auth/middleware"

        export default withAuth({
            // สามารถเพิ่ม logic การ redirect หรือเช็ค role ได้ที่นี่
            // pages: { signIn: "/signin" } // ถ้าใช้หน้า signin แบบ custom
        })


        // ★★★ กำหนด matcher เพื่อระบุว่าจะให้ middleware ทำงานกับ path ไหนบ้าง ★★★
        export const config = { matcher: ["/extra", "/dashboard"] } // ป้องกันเฉพาะ /extra และ /dashboard
        ```
    *   **อธิบาย:**
        *   วิธีแรก (export default) คือวิธีที่ง่ายที่สุดในการป้องกัน *ทุกหน้า* ด้วย NextAuth
        *   วิธีที่สอง (ใช้ `config` และ `matcher`) ช่วยให้ป้องกัน *เฉพาะบางเส้นทาง* ที่ต้องการได้ (`/extra` และ `/dashboard` ในตัวอย่าง) `matcher` รองรับ Regex ด้วย

**8. การแสดงข้อมูลผู้ใช้ (Timestamp ~33:31, 38:02)**

*   เมื่อมี `session` object (จาก `getServerSession` หรือ `useSession`) สามารถเข้าถึงข้อมูลผู้ใช้ได้ผ่าน `session.user`
*   ตัวอย่าง Property: `session.user?.name`, `session.user?.email`, `session.user?.image` (ใช้ Optional Chaining `?.` เพราะค่าเหล่านี้อาจไม่มี)
*   **แสดงรูปภาพจาก GitHub:** ต้องกำหนดค่าใน `next.config.js` เพื่ออนุญาต Domain ของรูปภาพ:
    ```javascript
    // next.config.js
    /** @type {import('next').NextConfig} */
    const nextConfig = {
        images: {
            remotePatterns: [
                {
                    protocol: 'https',
                    hostname: 'avatars.githubusercontent.com', // domain ของรูป GitHub
                    port: '',
                    pathname: '/u/**', // รูปแบบ path ของรูป
                },
            ]
        }
    }

    module.exports = nextConfig
    ```
*   สร้าง Component `UserCard.tsx` (ใน `src/components`) เพื่อแสดงข้อมูล (ดูโค้ดตัวอย่างจาก GitHub ของ Dave Gray)

**9. ลิงก์ Sign In / Sign Out (Timestamp ~32:59)**

*   NextAuth จัดการ Endpoint พื้นฐานให้:
    *   Sign In: `/api/auth/signin`
    *   Sign Out: `/api/auth/signout`
*   สร้าง Component `Navbar.tsx` (ใน `src/components`) และเพิ่มลิงก์ง่ายๆ:
    ```typescript
    // src/components/Navbar.tsx
    import Link from "next/link"

    export default function Navbar() {
        return (
            <nav className="bg-blue-800 p-4">
                <ul className="flex justify-evenly text-2xl font-bold">
                    <li><Link href="/">Home</Link></li>
                    <li><Link href="/api/auth/signin">Sign In</Link></li>
                    <li><Link href="/api/auth/signout">Sign Out</Link></li>
                    <li><Link href="/server">Server</Link></li>
                    <li><Link href="/client">Client</Link></li>
                    <li><Link href="/extra">Extra</Link></li>
                </ul>
            </nav>
        )
    }
    ```
*   **(ปรับปรุง)** ในแอปจริง ควรแสดง "Sign In" หรือ "Sign Out" อย่างใดอย่างหนึ่ง โดยเช็คสถานะ Session ก่อน (ทำได้ทั้งใน Server Component หรือ Client Component)

**10. ทดสอบการทำงาน (Timestamp ~23:50, 25:06, 31:14, etc.)**

*   รัน Development Server:
    ```bash
    npm run dev
    ```
*   เปิด Browser ไปที่ `http://localhost:3000`
*   ทดลองเข้าหน้าต่างๆ ที่ป้องกันไว้ (Home, Server, Client, Extra) จะถูก Redirect ไปหน้า Sign In
*   ทดลอง Sign In ด้วย Credentials (ที่ hardcode ไว้ใน `options.ts`)
*   ทดลอง Sign Out
*   ทดลอง Sign In ด้วย GitHub (ต้อง Authorize ครั้งแรก)
*   ตรวจสอบว่าเข้าถึงหน้าต่างๆ ได้หลัง Sign In และข้อมูลผู้ใช้ (ชื่อ, รูปภาพ) แสดงถูกต้อง
*   ทดลอง Sign Out อีกครั้ง

**สรุปและประเด็นสำคัญ:**

*   NextAuth.js ทำให้การเพิ่ม Authentication ใน Next.js ง่ายขึ้นมาก
*   การใช้ App Router ต้องสร้าง API Handler ใน `app/api/auth/[...nextauth]/route.ts`
*   ใช้ไฟล์ `options.ts` แยกต่างหากเพื่อกำหนดค่า NextAuth (Providers, etc.)
*   จัดการ Secrets (เช่น `NEXTAUTH_SECRET`, `GITHUB_ID`, `GITHUB_SECRET`) ผ่านไฟล์ `.env.local` (และห้าม commit ไฟล์นี้)
*   ต้องตั้งค่า OAuth App บน Platform นั้นๆ (เช่น GitHub) ให้ถูกต้อง (Callback URL สำคัญมาก)
*   ใช้ `SessionProvider` (Client Component) หุ้ม Layout หลัก เพื่อให้ Client Components อื่นๆ ใช้ `useSession` hook ได้
*   ป้องกัน Server Components โดยใช้ `getServerSession` แล้วเช็คค่า หรือใช้ `redirect`
*   ป้องกัน Client Components โดยใช้ `useSession` hook และตั้งค่า `required: true`
*   ใช้ `middleware.ts` เพื่อป้องกันทุกหน้า หรือใช้ `matcher` เพื่อป้องกันเฉพาะบางหน้า (เป็นวิธีที่ง่ายและครอบคลุม)
*   ต้องกำหนดค่า `images.remotePatterns` ใน `next.config.js` หากต้องการแสดงรูปภาพจาก External Domains

**การเรียนรู้เพิ่มเติม:**

*   การใช้งาน Providers อื่นๆ (Google, Facebook, etc.)
*   การปรับแต่งหน้า Sign In/Sign Out/Error
*   การใช้งาน Database Adapter เพื่อเก็บข้อมูล Session/User ในฐานข้อมูล
*   การจัดการ Role-Based Access Control (RBAC)
*   การใช้งาน Callbacks เพื่อปรับแต่งข้อมูล Session/JWT

บทเรียนนี้ครอบคลุมเนื้อหาหลักจากวิดีโอของ Dave Gray ซึ่งเป็นจุดเริ่มต้นที่ดีมากสำหรับการใช้งาน NextAuth.js กับ Next.js App Router ครับ!
