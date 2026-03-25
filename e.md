

## Updated Entities

### 1. `Role.java` – unchanged

```java
package com.example.visitor.entity;

import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.AllArgsConstructor;
import javax.persistence.*;
import javax.validation.constraints.*;
import java.util.List;

@Data
@NoArgsConstructor
@AllArgsConstructor
@Entity
@Table(name = "roles")
public class Role {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long roleId;

    @NotBlank(message = "Role name is required")
    @Size(max = 50, message = "Role name cannot exceed 50 characters")
    @Column(unique = true, nullable = false)
    private String roleName;

    @Size(max = 255, message = "Description cannot exceed 255 characters")
    private String description;

    @Column(columnDefinition = "TEXT")
    private String permissions;

    @OneToMany(mappedBy = "role", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<User> users;
}
```

### 2. `User.java` – unchanged

```java
package com.example.visitor.entity;

import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.AllArgsConstructor;
import javax.persistence.*;
import javax.validation.constraints.*;
import java.time.LocalDateTime;
import java.util.List;

@Data
@NoArgsConstructor
@AllArgsConstructor
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long userId;

    @NotBlank(message = "Name is required")
    @Size(max = 100, message = "Name cannot exceed 100 characters")
    private String name;

    @NotBlank(message = "Email is required")
    @Email(message = "Please provide a valid email address")
    @Column(unique = true, nullable = false)
    private String email;

    @NotBlank(message = "Password is required")
    private String passwordHash;

    @NotNull(message = "Role is required")
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "role_id", nullable = false)
    private Role role;

    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt = LocalDateTime.now();

    @Column(name = "last_login")
    private LocalDateTime lastLogin;

    @OneToMany(mappedBy = "createdBy", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<Visitor> visitors;
}
```

### 3. `Visitor.java` – **reasonForVisit removed**, added optional general notes

```java
package com.example.visitor.entity;

import com.example.visitor.enums.VisitorStatus;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.AllArgsConstructor;
import javax.persistence.*;
import javax.validation.constraints.*;
import java.time.LocalDateTime;
import java.util.List;

@Data
@NoArgsConstructor
@AllArgsConstructor
@Entity
@Table(name = "visitors")
public class Visitor {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long visitorId;

    @NotBlank(message = "Visitor name is required")
    @Size(max = 100, message = "Name cannot exceed 100 characters")
    private String name;

    @NotBlank(message = "Company name is required")
    @Size(max = 100, message = "Company name cannot exceed 100 characters")
    private String company;

    @NotBlank(message = "Contact number is required")
    @Pattern(regexp = "^[0-9+\\-\\s]{10,20}$", message = "Phone number must contain at least 10 digits and may include +, -, space")
    private String contactNumber;

    @NotBlank(message = "Email is required")
    @Email(message = "Please provide a valid email address")
    private String email;

    // General notes about the visitor (e.g., "VIP", "frequent visitor")
    @Size(max = 500, message = "Notes cannot exceed 500 characters")
    private String notes;

    @Column(name = "unique_id", unique = true, nullable = false)
    private String uniqueId;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private VisitorStatus status = VisitorStatus.PENDING;

    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt = LocalDateTime.now();

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "created_by", nullable = false)
    private User createdBy;

    @OneToMany(mappedBy = "visitor", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<VisitRecord> visitRecords;
}
```

### 4. `VisitRecord.java` – **added reasonForVisit and visitNotes**

```java
package com.example.visitor.entity;

import com.example.visitor.enums.VisitorStatus;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.AllArgsConstructor;
import javax.persistence.*;
import javax.validation.constraints.*;
import java.time.LocalDateTime;

@Data
@NoArgsConstructor
@AllArgsConstructor
@Entity
@Table(name = "visit_records")
public class VisitRecord {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long recordId;

    @NotNull(message = "Visitor must be specified")
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "visitor_id", nullable = false)
    private Visitor visitor;

    // Reason for this specific visit
    @NotBlank(message = "Reason for visit is required")
    @Size(max = 255, message = "Reason for visit cannot exceed 255 characters")
    private String reasonForVisit;

    // Optional notes specific to this visit
    @Size(max = 500, message = "Visit notes cannot exceed 500 characters")
    private String visitNotes;

    @NotNull(message = "Gate pass duration is required")
    @Min(value = 1, message = "Gate pass duration must be at least 1 hour")
    @Max(value = 24, message = "Gate pass duration cannot exceed 24 hours")
    private Integer gatePassDuration;

    @NotBlank(message = "Gate pass template is required")
    private String gatePassTemplate;

    @Column(name = "entry_time")
    private LocalDateTime entryTime;

    @Column(name = "exit_time")
    private LocalDateTime exitTime;

    @Column(name = "gate_pass_expiry_time")
    private LocalDateTime gatePassExpiryTime;

    @Enumerated(EnumType.STRING)
    @Column(name = "status_at_time")
    private VisitorStatus statusAtTime;
}
```

### 5. Enums and Configuration – unchanged

```java
package com.example.visitor.enums;

public enum VisitorStatus {
    PENDING,
    CHECKED_IN,
    CHECKED_OUT,
    EXPIRED
}
```

```java
package com.example.visitor.entity;

import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.AllArgsConstructor;
import javax.persistence.*;
import javax.validation.constraints.*;

@Data
@NoArgsConstructor
@AllArgsConstructor
@Entity
@Table(name = "system_config")
public class SystemConfig {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long configId;

    @NotBlank(message = "Config key is required")
    @Column(unique = true, nullable = false)
    private String configKey;

    @NotBlank(message = "Config value is required")
    @Column(nullable = false)
    private String configValue;
}
```



