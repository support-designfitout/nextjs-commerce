# 🚀 FitoutLab Pull Request

## 📍 Related Issue
Closes #____  
(Link to the corresponding [🚀 New Worker Endpoint](./ISSUE_TEMPLATE/new-worker-endpoint.md) or other issue.)

---

## 🎯 Purpose
What does this PR implement or fix?  
(e.g., "Add `/api/projects/:id/boq/snapshots` endpoint for P&L export")

---

## 🗂 Changes
- [ ] New endpoint(s) in `src/index.ts`
- [ ] Data model update(s)
- [ ] Worker binding usage (DB / R2 / KV / Queue / DO)
- [ ] Unit tests
- [ ] Documentation updated (`README.md` or API docs)

---

## 🔐 Security / Access
- [ ] Owner-only (Zero-Trust)  
- [ ] Designer role  
- [ ] Client read-only  
- [ ] Supplier restricted  
- [ ] Rate limiting added  
- [ ] Audit log event created  

---

## ✅ Checklist
- [ ] Code compiles and passes lint/typecheck (`ci.yml`)  
- [ ] Unit tests green  
- [ ] Preview build available (see sticky comment from `pr-previews.yml`)  
- [ ] Staging deployment validated  
- [ ] Documentation updated  

---

## 🧪 Testing Notes
Steps to validate this PR (include curl/Postman examples or UI flow):
