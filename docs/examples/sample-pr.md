# 🚀 FitoutLab Pull Request

## 📍 Related Issue
Closes #42  
(Request: Add `/api/projects/:id/boq/snapshots` endpoint)

---

## 🎯 Purpose
Implements an endpoint to save and retrieve BOQ (Bill of Quantities) snapshots for a project.  
This supports project P&L exports and milestone M2: **Calculations & BOQ Engine**.

---

## 🗂 Changes
- [x] Added `POST /api/projects/:id/boq/snapshot` to save snapshot  
- [x] Added `GET /api/projects/:id/boq/snapshots` to list all snapshots  
- [x] Created `BoqSnapshot` table in DB schema (Drizzle/Prisma model)  
- [x] Wrote unit tests (`tests/boq-snapshots.test.ts`)  
- [x] Updated API docs in `README.md`

---

## 🔐 Security / Access
- [x] Owner and Designer roles only  
- [ ] Client read-only access (future)  
- [ ] Supplier access not applicable  
- [x] Audit log created on snapshot save  

---

## ✅ Checklist
- [x] Code compiles and passes lint/typecheck (`ci.yml`)  
- [x] Unit tests green (`npm test`)  
- [x] Preview build available (see sticky comment from `pr-previews.yml`)  
- [x] Staging deployment validated  
- [x] Documentation updated  

---

## 🧪 Testing Notes
To test locally and in staging:

1. **Save snapshot**
   ```bash
   curl -X POST https://staging.fitoutlab.app/api/projects/123/boq/snapshot \
     -H "Authorization: Bearer <token>" \
     -H "Content-Type: application/json" \
     -d '{
       "items": [
         { "code": "M-001", "desc": "Tiles", "qty": 150, "unit": "m2", "rate": 55 }
       ],
       "totals": { "material": 8250, "labor": 2000, "equipment": 500 }
     }'
   ```