# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## โปรเจกต์นี้คืออะไร

เว็บหน้าเดียว "วันนี้กินอะไรดี" แนะนำอาหารไทย 30 เมนู มีสุ่มเมนู รายการโปรด (เก็บใน LocalStorage) ตัวกรอง และหน้าต่างรายละเอียดส่วนประกอบ ไม่มี build step, ไม่มี package manager, ไม่มี test framework

## คำสั่ง

- รัน: เปิด `index.html` ในเบราว์เซอร์ตรงๆ (หรือ `python -m http.server` แล้วเปิด `localhost:8000`) ไม่มี lint/build/test
- ฟอนต์ (Prompt, Sarabun) โหลดจาก Google Fonts ต้องมีอินเทอร์เน็ต ถ้าไม่มีจะ fallback เป็นฟอนต์ระบบ
- ขั้นตอนทดสอบด้วยมือที่ยังค้างอยู่ อยู่ในหัวข้อ 6 ของ `task.md`

## สถาปัตยกรรม

ทุกอย่างอยู่ใน `index.html` (`<style>` แล้วตามด้วย `<script>` เดียว) ไฟล์รูปอยู่ใน `images/`

- **ข้อมูล**: `DISHES` (id, name, desc, cat, spice 0–3, emoji, `img` ไม่บังคับ) และ `INGREDIENTS` (คีย์เป็น `id` เดียวกัน: `main`, `season`, `allergens`) แยกเป็นสองตาราง เพิ่มเมนูใหม่ต้องเพิ่มทั้งสองที่ และเพิ่ม slug ใน `IMG_SLUGS` ถ้ามีรูป
- **รูปภาพ**: `imgSrc()` เลือก `d.img` ก่อน ไม่งั้นใช้ `images/<slug>.jpg` จาก `IMG_SLUGS` (ผัดไทยเป็น `.png` ผ่าน `img`) `iconHTML()` ใส่ `onerror` ให้กลับไปแสดง emoji ถ้าไฟล์หาย
- **สถานะ**: ตัวแปรระดับโมดูล `favorites` (array ของ id), `lastPickId`, `activeCat`, `shuffling` ไม่มี framework การแสดงผลทำผ่านฟังก์ชัน `render*` ที่เขียน `innerHTML` ใหม่ทั้งส่วน
- **การ render**: `render()` เรียก `renderPick` + `renderFavs` + `renderAll` (`renderAll` วาดรายการเมนูทั้งหมดตามตัวกรอง) ตัวกรอง (หมวด, ความเผ็ด, ค้นหา, เฉพาะโปรด) อ่านค่าจาก DOM ใน `filteredDishes()` ระหว่างอนิเมชันสุ่ม (`shuffling`) `render()` จะไม่วาดการ์ดผลสุ่มทับ
- **Event**: ใช้ event delegation ตัวเดียวที่ `document` ผ่าน attribute `data-fav`, `data-detail`, `data-cat` และ id `#clearBtn`, `#detailClose` ปุ่มใหม่ควรใช้รูปแบบเดียวกัน
- **รายการโปรด**: บันทึกใน LocalStorage คีย์ `thaiFoodFavorites` โหลดแล้วกรอง id ที่ไม่มีใน `DISHES` ทิ้ง และครอบ try/catch ไว้ทุกจุด
- **สุ่มเมนู**: `randomPick()` ตัด `lastPickId` ออกจาก pool เมื่อ pool มีมากกว่า 1 เมนู เพื่อไม่ให้ซ้ำติดกัน ถ้า `prefers-reduced-motion` จะข้ามอนิเมชัน
- **หน้าต่างรายละเอียด**: ใช้ `<dialog>` (`showModal`) และคืน focus ให้ปุ่มที่เปิดเมื่อปิด

## ธีมและการเข้าถึง

สีกำหนดเป็น CSS variables ที่ `:root` พร้อมชุด dark ใน `prefers-color-scheme` ปุ่มต้องมี `aria-label`/`aria-pressed` และความเผ็ดใช้ `role="img"` + `aria-label` (`spiceLabel`)

## คำศัพท์โดเมน

ใช้คำตาม `CONTEXT.md` ทั้งใน UI และโค้ด: เมนู (Dish), หมวด (Category), ระดับความเผ็ด (Spice Level), สุ่มเมนู (Random Pick), รายการโปรด (Favorite) และหลีกเลี่ยงคำที่ระบุใน _Avoid_ (เช่น "ไลก์", "บุ๊กมาร์ก", "แท็ก") เมื่อแก้คำศัพท์หรือเพิ่มแนวคิดใหม่ ให้ใช้ skill `domain-modeling` ที่ `.agents/skills/`
