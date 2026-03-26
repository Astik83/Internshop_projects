

```text
com.vms
│
├── controller
│   └── VisitorController.java
│
├── service
│   ├── VisitorService.java
│   └── JwtService.java
│
├── repository
│   ├── UserRepository.java
│   └── VisitorRepository.java
│
├── entity
│   ├── User.java
│   └── Visitor.java
```

---

# 🧩 1. JwtService (📁 service/JwtService.java)

👉 Responsible for token logic

```java
@Service
public class JwtService {

    private final String SECRET = "my-secret-key-my-secret-key";

    public String generateToken(User user) {
        return Jwts.builder()
                .setSubject(user.getEmail())
                .claim("userId", user.getId()) // 🔥 IMPORTANT
                .setIssuedAt(new Date())
                .setExpiration(new Date(System.currentTimeMillis() + 1000 * 60 * 60))
                .signWith(Keys.hmacShaKeyFor(SECRET.getBytes()))
                .compact();
    }

    public Long extractUserId(String token) {
        return Jwts.parserBuilder()
                .setSigningKey(SECRET.getBytes())
                .build()
                .parseClaimsJws(token)
                .getBody()
                .get("userId", Long.class);
    }
}
```

---

# 🧩 2. VisitorService (📁 service/VisitorService.java)

👉 Main business logic (MOST IMPORTANT)

```java
@Service
public class VisitorService {

    @Autowired
    private JwtService jwtService;

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private VisitorRepository visitorRepository;

    // 🔥 Helper method
    private Long getCurrentUserId(HttpServletRequest request) {

        String authHeader = request.getHeader("Authorization");

        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            throw new RuntimeException("No token found");
        }

        String token = authHeader.substring(7);

        return jwtService.extractUserId(token);
    }

    // 🔥 Your main method
    public Visitor registerVisitor(Visitor visitor, HttpServletRequest request) {

        // 1. Get logged-in userId
        Long userId = getCurrentUserId(request);

        // 2. Fetch receptionist
        User receptionist = userRepository.findById(userId)
                .orElseThrow(() -> new RuntimeException("User not found"));

        // 3. Set createdBy
        visitor.setCreatedBy(receptionist);

        // 4. Save visitor
        return visitorRepository.save(visitor);
    }
}
```

---

# 🧩 3. VisitorController (📁 controller/VisitorController.java)

👉 Handles API request

```java
@RestController
@RequestMapping("/visitors")
public class VisitorController {

    @Autowired
    private VisitorService visitorService;

    @PostMapping
    public Visitor createVisitor(@RequestBody Visitor visitor,
                                 HttpServletRequest request) {

        return visitorService.registerVisitor(visitor, request);
    }
}
```

---

# 🧩 4. UserRepository (📁 repository/UserRepository.java)

```java
public interface UserRepository extends JpaRepository<User, Long> {
}
```

---

# 🧩 5. VisitorRepository (📁 repository/VisitorRepository.java)

```java
public interface VisitorRepository extends JpaRepository<Visitor, Long> {
}
```

---

# 🧩 6. Visitor Entity (📁 entity/Visitor.java)

```java
@Entity
public class Visitor {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @ManyToOne
    @JoinColumn(name = "created_by")
    private User createdBy;

    // getters & setters
}
```

---

# 🧩 7. User Entity (📁 entity/User.java)

```java
@Entity
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String email;

    // getters & setters
}
```

---

# 🔄 FINAL FLOW (VERY CLEAR)

```text
1. Login → JWT generated (contains userId)
2. Frontend stores token
3. Frontend calls POST /visitors with token

   Header:
   Authorization: Bearer eyJhbGciOiJIUzI1Ni...

4. Controller receives request
5. Service extracts userId from token
6. Fetch user using findById
7. visitor.setCreatedBy(receptionist)
8. Save in DB ✅
```

---

# 🧠 What You Achieved

✔ You know which receptionist created visitor
✔ Clean and simple logic
✔ No complex security setup
✔ Easy to debug

---

# ⚠️ Important Notes

❌ Don’t send userId from frontend
✔ Always extract from token

---

# 🎯 One-Line Summary

👉 **Controller → Service → extract userId → find user → set createdBy → save**

---
