# 📏 Standards de code et bonnes pratiques

Ce document définit les conventions de code et les meilleures pratiques à suivre.

## 🎨 Conventions de nommage

### Classes

```java
// Entities : Nom singulier, PascalCase
public class User { }
public class Transaction { }

// Controllers : Suffixe "Controller"
@RestController
public class UserController { }

// Services : Suffixe "Service"
@Service
public class UserService { }

// Repositories : Suffixe "Repository"
public interface UserRepository extends JpaRepository<User, Long> { }

// DTOs : Suffixe "Dto"
public class UserRequestDto { }
public class UserResponseDto { }

// Exceptions : Suffixe "Exception"
public class ResourceNotFoundException extends RuntimeException { }

// Mappers : Suffixe "Mapper"
@Component
public class UserMapper { }
```

### Méthodes et variables

```java
// Méthodes : camelCase, verbes d'action
public void createUser() { }
public User findById(Long id) { }
public boolean isExpenseAllowed() { }

// Variables : camelCase, noms descriptifs
private String username;
private BigDecimal accountBalance;
private LocalDate transactionDate;

// Constantes : UPPER_SNAKE_CASE
public static final String DEFAULT_CURRENCY = "EUR";
public static final int MAX_TRANSACTION_AMOUNT = 10000;

// Booleans : préfixes "is", "has", "can"
private boolean isActive;
private boolean hasPermission;
```

### Packages

```java
com.budget.api
├── config          // Configuration classes
├── controller      // REST controllers
├── dto             // Data Transfer Objects
├── entity          // JPA entities
├── repository      // Spring Data repositories
├── service         // Business logic
├── security        // Security related classes
├── exception       // Custom exceptions
├── mapper          // Entity-DTO mappers
└── util            // Utility classes
```

---

## 🏗️ Structure d'une classe Controller

```java
@RestController
@RequestMapping("/api/v1/users")
@RequiredArgsConstructor
@Validated
public class UserController {
    
    // 1. Dépendances injectées
    private final UserService userService;
    
    // 2. Endpoints POST (création)
    @PostMapping
    public ResponseEntity<UserResponseDto> create(
        @Valid @RequestBody UserRequestDto request
    ) {
        UserResponseDto response = userService.createUser(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(response);
    }
    
    // 3. Endpoints GET (lecture)
    @GetMapping
    public ResponseEntity<List<UserResponseDto>> getAll() {
        List<UserResponseDto> users = userService.getAllUsers();
        return ResponseEntity.ok(users);
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<UserResponseDto> getById(@PathVariable Long id) {
        UserResponseDto user = userService.getUserById(id);
        return ResponseEntity.ok(user);
    }
    
    // 4. Endpoints PUT (mise à jour)
    @PutMapping("/{id}")
    public ResponseEntity<UserResponseDto> update(
        @PathVariable Long id,
        @Valid @RequestBody UserRequestDto request
    ) {
        UserResponseDto updated = userService.updateUser(id, request);
        return ResponseEntity.ok(updated);
    }
    
    // 5. Endpoints DELETE (suppression)
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        userService.deleteUser(id);
        return ResponseEntity.noContent().build();
    }
}
```

---

## 🏗️ Structure d'une classe Service

```java
@Service
@RequiredArgsConstructor
@Transactional
@Slf4j
public class UserService {
    
    // 1. Dépendances
    private final UserRepository userRepository;
    private final UserMapper userMapper;
    
    // 2. Méthodes publiques
    public UserResponseDto createUser(UserRequestDto request) {
        log.debug("Creating user: {}", request.getUsername());
        
        // Validation métier
        validateUserDoesNotExist(request.getUsername());
        
        // Transformation et création
        User user = userMapper.toEntity(request);
        User saved = userRepository.save(user);
        
        log.info("User created: {}", saved.getId());
        return userMapper.toDto(saved);
    }
    
    @Transactional(readOnly = true)
    public UserResponseDto getUserById(Long id) {
        User user = findUserOrThrow(id);
        return userMapper.toDto(user);
    }
    
    // 3. Méthodes privées
    private void validateUserDoesNotExist(String username) {
        if (userRepository.existsByUsername(username)) {
            throw new DuplicateResourceException("Username exists");
        }
    }
    
    private User findUserOrThrow(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("User not found: " + id));
    }
}
```

---

## 🏗️ Structure d'une Entity

```java
@Entity
@Table(name = "users")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, unique = true, length = 50)
    private String username;
    
    @Column(nullable = false, unique = true, length = 100)
    private String email;
    
    @Column(nullable = false)
    @JsonIgnore  // Ne jamais exposer le mot de passe
    private String password;
    
    @Enumerated(EnumType.STRING)
    private Role role = Role.USER;
    
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL)
    private List<Account> accounts = new ArrayList<>();
    
    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdAt;
    
    // Méthodes utilitaires
    public void addAccount(Account account) {
        accounts.add(account);
        account.setUser(this);
    }
}
```

---

## 🏗️ Structure d'un DTO

```java
// Request DTO
@Data
@NoArgsConstructor
@AllArgsConstructor
public class TransactionRequestDto {
    
    @NotBlank(message = "Description is required")
    @Size(max = 200)
    private String description;
    
    @NotNull(message = "Amount is required")
    @DecimalMin(value = "0.01")
    private BigDecimal amount;
    
    @NotNull
    private TransactionType type;
    
    @NotNull
    @PastOrPresent
    private LocalDate date;
    
    @NotNull
    private Long accountId;
    
    private Long categoryId;
}

// Response DTO
@Data
@Builder
public class TransactionResponseDto {
    private Long id;
    private String description;
    private BigDecimal amount;
    private TransactionType type;
    private LocalDate date;
    private AccountSummaryDto account;
    private CategoryDto category;
    private LocalDateTime createdAt;
}
```

---

## 🎯 Gestion des exceptions

### Exceptions personnalisées

```java
public class BudgetApiException extends RuntimeException {
    public BudgetApiException(String message) {
        super(message);
    }
}

public class ResourceNotFoundException extends BudgetApiException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}

public class DuplicateResourceException extends BudgetApiException {
    public DuplicateResourceException(String message) {
        super(message);
    }
}
```

### Handler global

```java
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(
        ResourceNotFoundException ex
    ) {
        log.error("Resource not found: {}", ex.getMessage());
        ErrorResponse error = ErrorResponse.builder()
            .status(HttpStatus.NOT_FOUND.value())
            .message(ex.getMessage())
            .timestamp(LocalDateTime.now())
            .build();
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(
        MethodArgumentNotValidException ex
    ) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(error ->
            errors.put(error.getField(), error.getDefaultMessage())
        );
        
        ErrorResponse error = ErrorResponse.builder()
            .status(HttpStatus.BAD_REQUEST.value())
            .message("Validation failed")
            .errors(errors)
            .timestamp(LocalDateTime.now())
            .build();
            
        return ResponseEntity.badRequest().body(error);
    }
}
```

---

## 🎯 Validation

### Annotations de validation

```java
public class UserRequestDto {
    
    @NotBlank
    @Size(min = 3, max = 50)
    @Pattern(regexp = "^[a-zA-Z0-9_]+$")
    private String username;
    
    @NotBlank
    @Email
    private String email;
    
    @NotBlank
    @Size(min = 8)
    private String password;
}
```

---

## 🎯 Logging

```java
@Service
@Slf4j
public class TransactionService {
    
    public TransactionResponseDto createTransaction(TransactionRequestDto request) {
        // DEBUG : Informations détaillées
        log.debug("Creating transaction: {}", request);
        
        try {
            Transaction transaction = processTransaction(request);
            
            // INFO : Événements importants
            log.info("Transaction created: id={}", transaction.getId());
            
            return mapper.toDto(transaction);
            
        } catch (InsufficientBalanceException e) {
            // WARN : Problèmes non critiques
            log.warn("Insufficient balance: {}", e.getMessage());
            throw e;
        } catch (Exception e) {
            // ERROR : Erreurs graves
            log.error("Failed to create transaction", e);
            throw new BudgetApiException("Transaction failed", e);
        }
    }
}
```

**Niveaux de log** :
- `DEBUG` : Informations de débogage
- `INFO` : Événements normaux importants
- `WARN` : Situations anormales non critiques
- `ERROR` : Erreurs nécessitant attention

---

## 🎯 Tests

### Test unitaire d'un Service

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    
    @Mock
    private UserRepository userRepository;
    
    @Mock
    private UserMapper userMapper;
    
    @InjectMocks
    private UserService userService;
    
    @Test
    void createUser_WithValidData_ShouldReturnCreatedUser() {
        // Given
        UserRequestDto request = new UserRequestDto("test", "test@example.com", "pass");
        User user = User.builder().id(1L).username("test").build();
        UserResponseDto expected = new UserResponseDto(1L, "test", "test@example.com");
        
        when(userRepository.existsByUsername(anyString())).thenReturn(false);
        when(userMapper.toEntity(request)).thenReturn(user);
        when(userRepository.save(any(User.class))).thenReturn(user);
        when(userMapper.toDto(user)).thenReturn(expected);
        
        // When
        UserResponseDto result = userService.createUser(request);
        
        // Then
        assertNotNull(result);
        assertEquals(expected.getId(), result.getId());
        verify(userRepository).save(any(User.class));
    }
    
    @Test
    void createUser_WithDuplicateUsername_ShouldThrowException() {
        // Given
        UserRequestDto request = new UserRequestDto("existing", "new@example.com", "pass");
        when(userRepository.existsByUsername("existing")).thenReturn(true);
        
        // When & Then
        assertThrows(DuplicateResourceException.class, () -> {
            userService.createUser(request);
        });
        
        verify(userRepository, never()).save(any());
    }
}
```

---

## ✅ Checklist qualité du code

Avant chaque commit :

- [ ] Code compilé sans warnings
- [ ] Conventions de nommage respectées
- [ ] Pas de code commenté inutile
- [ ] Pas de System.out.println() (utiliser logger)
- [ ] Exceptions gérées proprement
- [ ] Validation des entrées utilisateur
- [ ] Tests unitaires pour logique métier
- [ ] Pas de secrets en dur
- [ ] DTOs pour communication externe
- [ ] Documentation pour méthodes publiques complexes

---

**Prochaine étape** → [Guide API](API_GUIDE.md)

