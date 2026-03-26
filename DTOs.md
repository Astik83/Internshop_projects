## Role Enum
```java
public enum RoleType {
    ADMIN,
    RECEPTIONIST,
    SECURITY
}
```

## ROle DTO

```java
import jakarta.validation.constraints.NotNull;
import jakarta.validation.constraints.Size;

public class RoleDTO {

    private Integer roleId;

    @NotNull(message = "Role name is required")
    private RoleType roleName;

    @Size(max = 255, message = "Description cannot exceed 255 characters")
    private String description;

    // getters & setters
}


```
## user Dto

```java
import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;
import jakarta.validation.constraints.Size;

public class UserDTO {

    private Integer userId;

    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 100, message = "Name must be between 2 and 100 characters")
    private String name;

    @NotBlank(message = "Email is required")
    @Email(message = "Invalid email format")
    private String email;

    @NotBlank(message = "Password is required")
    @Size(min = 6, message = "Password must be at least 6 characters")
    private String password;

    @NotNull(message = "Role ID is required")
    private Integer roleId;

    private RoleType roleName;

    // getters & setters
}

```
## 🔷 1. VisitorRequestDto (Create / Update Visitor)

**Used for:**
- Create visitor (inside wrapper)
- Update visitor

```java
public class VisitorRequestDto {

    @NotBlank(message = "Name is required")
    private String name;

    @NotBlank(message = "Company is required")
    private String company;

    @NotBlank(message = "Contact number is required")
    @Pattern(regexp = "\\d{10}", message = "Invalid phone number")
    private String contactNumber;

    @NotBlank(message = "Email is required")
    @Email(message = "Invalid email format")
    private String email;

    private String notes;

    // getters & setters
}
```

---

## 🔷 2. VisitorResponseDto (Full Visitor Response)

**Used for:**
- GET all visitors
- GET visitor by ID

```java
public class VisitorResponseDto {

    private Integer visitorId;
    private String uniqueId;
    private String name;
    private String company;
    private String contactNumber;
    private String email;
    private String notes;
    private String status;
    private LocalDateTime createdAt;

    // All visits (for detailed API)
    private List<VisitResponseDto> visitRecords;

    // getters & setters
}
```

---

## 🔷 3. VisitRequestDto (Create / Update Visit)

**Used for:**
- Add visit
- Update visit

```java
public class VisitRequestDto {

    @NotBlank(message = "Reason for visit is required")
    private String reasonForVisit;

    private String visitNotes;

    @Min(value = 1, message = "Duration must be at least 1 hour")
    private Integer gatePassDuration;

    private String gatePassTemplate;

    // getters & setters
}
```

---

## 🔷 4. VisitResponseDto (Visit Output)

**Used for:**
- Returning visit details

```java
public class VisitResponseDto {

    private Integer recordId;
    private String reasonForVisit;
    private String visitNotes;
    private Integer gatePassDuration;
    private String gatePassTemplate;
    private LocalDateTime entryTime;
    private LocalDateTime exitTime;
    private LocalDateTime gatePassExpiryTime;
    private String statusAtTime;

    // getters & setters
}
```

---

## 🔷 5. VisitorWithVisitRequestDto ⭐ (IMPORTANT)

**Used ONLY for:**
- `POST /visitors` (Register visitor + first visit)

```java
public class VisitorWithVisitRequestDto {

    @Valid
    @NotNull(message = "Visitor details are required")
    private VisitorRequestDto visitor;

    @Valid
    @NotNull(message = "Visit details are required")
    private VisitRequestDto visit;

    // getters & setters
}
```

---

## 🔷 ✅ How JSON Maps to DTO (Your Main Doubt SOLVED)

### Request:

```json
{
  "visitor": {
    "name": "Astik",
    "company": "TCS",
    "contactNumber": "9876543210",
    "email": "astik@gmail.com"
  },
  "visit": {
    "reasonForVisit": "Meeting",
    "gatePassDuration": 2
  }
}
```

### Mapping:

```
VisitorWithVisitRequestDto
 ├── VisitorRequestDto
 └── VisitRequestDto
```

✔ No mismatch  
✔ Clean separation  
✔ Easy to maintain

---

## 🔷 DTO Usage per API

| API                                  | DTO Used                   |
| ------------------------------------ | -------------------------- |
| POST /visitors                       | VisitorWithVisitRequestDto |
| POST /visitors/{id}/visits           | VisitRequestDto            |
| GET /visitors                        | VisitorResponseDto         |
| GET /visitors/{id}                   | VisitorResponseDto         |
| PUT /visitors/{id}                   | VisitorRequestDto          |
| PUT /visitors/{id}/visits/{recordId} | VisitRequestDto            |

---

## 🔥 Important Design Principles (Interview Ready)

### ✅ 1. Separation of Concern
- Visitor → Permanent data
- Visit → Transaction data

### ✅ 2. No Nested Lists in Request DTO

❌ Don’t do this:
```java
List<VisitRequestDto> visits;
```

✔ Because:
- You only create ONE visit at a time
- Keeps API simple

### ✅ 3. DTO ≠ Entity
```
Entity → DB structure  
DTO → API structure
```

### ✅ 4. Keep DTOs Lightweight
✔ Only fields  
✔ Only validation  
❌ No business logic

---

## 🔷 Final Folder Structure

```
dto/
 ├── VisitorRequestDto
 ├── VisitorResponseDto
 ├── VisitRequestDto
 ├── VisitResponseDto
 └── VisitorWithVisitRequestDto
```

---

