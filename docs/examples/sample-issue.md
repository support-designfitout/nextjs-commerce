# 🚀 New Worker Endpoint: BOQ Snapshots

## 📍 Endpoint Request
**Path:**  
`/api/projects/:id/boq/snapshots`

**Method:**  
- [x] POST (save snapshot)  
- [x] GET (list snapshots)  

---

## 🎯 Purpose
Enable project teams to **save and retrieve BOQ (Bill of Quantities) snapshots** for cost tracking and export.  
Supports milestone **M2: Calculations & BOQ Engine**.

---

## 🗂 Data Model(s)
Touches the following tables:  
- [x] `Project`  
- [x] `BoqItem`  
- [ ] `Material` (indirect reference)  
- [x] `AuditLog`  

New model:  
- `BoqSnapshot { id, projectId, totals_json, createdAt }`

---

## ⚙️ Logic & Flow
1. **POST** `/api/projects/:id/boq/snapshot`  
   - Validate `projectId` exists and user has access.  
   - Save snapshot with items + totals into `BoqSnapshot` table.  
   - Log action in `AuditLog`.  

2. **GET** `/api/projects/:id/boq/snapshots`  
   - Validate `projectId` and user role.  
   - Return list of snapshots with timestamps and totals.  

---

## 🔐 Security Considerations
- [x] Owner/Admin role only (for saving snapshots)  
- [x] Designers can view snapshots  
- [ ] Clients read-only access (future)  
- [ ] Suppliers not applicable  
- [x] Audit log entry required for each snapshot  

---

## ✅ Acceptance Criteria
- [x] Snapshot save API accepts JSON payload of items and totals.  
- [x] Saved snapshots appear when listing snapshots for a project.  
- [x] Unauthorized users cannot access project snapshots.  
- [x] Automated tests cover both endpoints.  
- [x] Staging deployment shows working endpoints.  

---

✍️ Notes for implementation:
- Use Hyperdrive → PlanetScale to store snapshots.  
- Future: integrate Looker Studio export on snapshot creation.