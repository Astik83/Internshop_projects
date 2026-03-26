

# 🧩 1. Helper Method (Get Logged-in User by ID)

👉 Assumption: your JWT already contains **userId**

```java
private User getCurrentUser() {
    Authentication auth = SecurityContextHolder.getContext().getAuthentication();

    // 🔥 principal should contain userId
    CustomUserPrincipal principal = (CustomUserPrincipal) auth.getPrincipal();

    Long userId = principal.getId();

    return userRepository.findById(userId)
            .orElseThrow(() -> new RuntimeException("User not found"));
}
```

---

# 🧠 If Your Principal Currently Has Only Username

Then update your **JWT filter** to store `CustomUserPrincipal` (with ID).
Otherwise this cast won’t work.

---

# 🧩 2. Updated Service Logic ✅

Here is your **final clean method**:

```java
@Override
public VisitorResponseDto registerVisitor(VisitorWithVisitRequestDto dto) {

    // 🔥 STEP 1: Get logged-in receptionist (via userId)
    User receptionist = getCurrentUser();

    // STEP 2: Map visitor
    Visitor visitor = modelMapper.map(dto.getVisitor(), Visitor.class);

    visitor.setUniqueId(generateUniqueId());
    visitor.setStatus(VisitorStatus.PENDING);
    visitor.setCreatedAt(LocalDateTime.now());

    // 🔥 STEP 3: Set createdBy
    visitor.setCreatedBy(receptionist);

    // STEP 4: Save visitor
    Visitor savedVisitor = visitorRepository.save(visitor);

    // STEP 5: Create visit
    VisitRecord visit = new VisitRecord();
    visit.setVisitor(savedVisitor);
    visit.setReasonForVisit(dto.getVisit().getReasonForVisit());
    visit.setVisitNotes(dto.getVisit().getVisitNotes());
    visit.setGatePassDuration(dto.getVisit().getGatePassDuration());
    visit.setGatePassTemplate(
        dto.getVisit().getGatePassTemplate() != null
            ? dto.getVisit().getGatePassTemplate()
            : "Standard"
    );

    visitRecordRepository.save(visit);

    savedVisitor.setVisitRecords(List.of(visit));

    return mapToVisitorResponse(savedVisitor);
}
```

---

# 🔐 3. Important Requirement (Must Have)

👉 Your `CustomUserPrincipal` must contain **userId**

```java
public class CustomUserPrincipal implements UserDetails {

    private Long id;
    private String username;

    public Long getId() {
        return id;
    }
}
```

---

# 🧩 4. JWT Filter (Important Part)

Make sure you set principal like this:

```java
Long userId = jwtService.extractUserId(token);
String username = jwtService.extractUsername(token);

CustomUserPrincipal principal =
    new CustomUserPrincipal(userId, username, null, List.of());

UsernamePasswordAuthenticationToken auth =
    new UsernamePasswordAuthenticationToken(principal, null, principal.getAuthorities());

SecurityContextHolder.getContext().setAuthentication(auth);
```

---

# 🧠 Flow (Your Final System)

```text
Login → JWT (contains userId)
        ↓
POST /visitors (with token)
        ↓
JWT Filter extracts userId
        ↓
SecurityContext stores principal
        ↓
getCurrentUser() → findById()
        ↓
visitor.setCreatedBy(receptionist) ✅
```

---

# ⚠️ Common Mistakes

❌ Principal not castable to `CustomUserPrincipal`
❌ JWT not storing userId
❌ Token not sent in header

---

# 🎯 One-Line Summary

👉 **Extract userId from JWT → use `findById()` → set `createdBy`**


