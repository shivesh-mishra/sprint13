# Prompts.md — AI Architectural Queries & Iterations

Below is the record of my prompt engineering process while designing the VitalSync architecture. I used AI as a sounding board for scoping, database schema design, and concurrency logic, but iterated heavily to meet the specific constraints of this capstone.

## 1. Project Scoping & Scope Creep Prevention
**My Prompt:** 
"I'm building VitalSync for a 4-week capstone. It's a healthcare EHR with RBAC (Doctor/Patient), appointment booking, medical history, and prescriptions. I have to build an MVP by Sprint 14 and final deployment by Sprint 17. Suggest a PRD structure with prioritized features (P0, P1, P2) and explicitly list what I should NOT build to avoid scope creep."

**What I kept:** 
The P0/P1/P2 framework. I moved the "AI summarizer" to P2 (Sprint 16) and kept the basic Auth + Booking as P0.

**What I changed/rejected:** 
The AI suggested adding a "Video Calling" feature for telemedicine. I rejected this because implementing WebRTC within 4 weeks for an individual contributor would guarantee failure. I added it to the "Explicitly Out of Scope" list.

## 2. Database Schema Design (MongoDB)
**My Prompt:** 
"I have a strict limit of 5 MongoDB collections for my healthcare app. I need users (doctors/patients), appointments, and medical records. Should I create a separate `doctors` and `patients` collection, or embed them in `users`? Also, how do I structure the slot availability to prevent double booking?"

**Outcome:** 
The AI initially suggested 6 collections (Users, Doctors, Patients, Slots, Appointments, Records). I had to push back: *"I can only use 5. How do I optimize this?"* 
**Final Decision:** I embedded the doctor's specialization and the patient's blood group/allergies directly into the `users` collection to save space. This is the logic behind my ERD.

## 3. Concurrency & Double-Booking Logic
**My Prompt:** 
"What is the best way to prevent two patients from booking the same appointment slot simultaneously in Node.js and Mongoose? I want to avoid race conditions."

**Outcome:** 
The AI suggested using MongoDB Transactions (Sessions). However, I realized that adding transactions adds boilerplate code. Instead, I asked: *"Can I do this with a single atomic query?"* 
**Final Decision:** I implemented a `findOneAndUpdate` on the `availability_slots` collection, filtering by `status: 'available'`. If it returns a document, I create the appointment. If it returns null, someone else booked it. This is documented in my ERD notes.

## 4. UI/UX & Figma Layouts
**My Prompt:** 
"I have 6 core screens for a healthcare app (Auth, Patient Dashboard, Doctor Dashboard, Booking, Timeline, Patient Detail). I am using Tailwind CSS and Shadcn UI. Can you suggest a modern dashboard layout that handles a sidebar, a top navbar, and a data-heavy main content area? Keep mobile responsiveness in mind."

**What I changed:** 
The AI suggested a heavy sidebar with nested menus. I simplified it to a flat sidebar (Dashboard, Availability, Patients, Appointments) because doctors need to access things quickly without clicking through folders. I also ensured the "Book Appointment" button had a primary blue color (`#2563EB`) to stand out.

## 5. Real-Time Architecture (Socket.IO)
**My Prompt:** 
"I want to use Socket.IO to push real-time slot updates to patients booking appointments. How should I structure the rooms so I don't send updates to everyone? Also, how do I secure the socket connection?"

**Outcome:** 
**Final Decision:** I learned from the AI that I should authenticate the socket connection using the JWT token during the handshake. I also decided to create dynamic rooms based on the doctor's ID (e.g., `room:doctor_123`). When a slot is booked, I emit `slot:updated` only to that specific room.

---
*Note: This document reflects my actual process of constraining AI outputs to fit the 4-week timeline and the 5-collection database limit.*