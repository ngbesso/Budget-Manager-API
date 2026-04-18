# 🏗️ Architecture détaillée

Ce document explique l'architecture complète du projet et les choix de conception.

## 📐 Vue d'ensemble de l'architecture

Nous utilisons une **architecture en couches (Layered Architecture)** qui est le standard de l'industrie pour les applications Spring Boot.

```
┌──────────────────────────────────────────────────────┐
│                  CLIENT (Browser, Mobile)            │
└──────────────────────────────────────────────────────┘
                          ↓ HTTP/JSON
┌──────────────────────────────────────────────────────┐
│              PRESENTATION LAYER                      │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │
│  │ Controllers │  │    DTOs     │  │  Exception  │ │
│  │   (REST)    │  │  (Request/  │  │   Handler   │ │
│  │             │  │   Response) │  │             │ │
│  └─────────────┘  └─────────────┘  └─────────────┘ │
└──────────────────────────────────────────────────────┘
                          ↓
┌──────────────────────────────────────────────────────┐
│               SERVICE LAYER                          │
│  ┌──────────────────────────────────────────────┐   │
│  │         Business Logic                       │   │
│  │  - Validation métier                         │   │
│  │  - Orchestration                             │   │
│  │  - Transformation des données                │   │
│  └──────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────┘
                          ↓
┌──────────────────────────────────────────────────────┐
│              PERSISTENCE LAYER                       │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │
│  │Repositories │  │  Entities   │  │     JPA     │ │
│  │   (DAO)     │  │  (Models)   │  │  (Hibernate)│ │
│  └─────────────┘  └─────────────┘  └─────────────┘ │
└──────────────────────────────────────────────────────┘
                          ↓
┌──────────────────────────────────────────────────────┐
│                  DATABASE (PostgreSQL)               │
└──────────────────────────────────────────────────────┘
```

## 🔷 Couches de l'application

### 1. Presentation Layer (Couche de présentation)

**Responsabilité** : Gérer les requêtes HTTP et les réponses

#### Controllers
- Endpoints REST
- Validation des entrées (annotations `@Valid`)
- Mapping des URLs (`@GetMapping`, `@PostMapping`, etc.)
- Codes de statut HTTP appropriés

**Exemple** :
```java
@RestController
@RequestMapping("/api/v1/transactions")
@RequiredArgsConstructor
public class TransactionController {
    
    private final TransactionService transactionService;
    
    @PostMapping
    public ResponseEntity<TransactionResponseDto> create(
        @Valid @RequestBody TransactionRequestDto request
    ) {
        TransactionResponseDto response = transactionService.createTransaction(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(response);
    }
}
```

#### DTOs (Data Transfer Objects)
- Objets pour transférer les données entre couches
- Séparation entre le modèle interne et l'API publique
- Validation avec annotations (`@NotNull`, `@Size`, etc.)

**Types de DTOs** :
- `RequestDto` : Données entrantes du client
- `ResponseDto` : Données sortantes vers le client
- `CreateDto` / `UpdateDto` : Séparation création/modification

### 2. Service Layer (Couche métier)

**Responsabilité** : Logique métier et orchestration

#### Services
- Logique métier complexe
- Validation des règles métier
- Orchestration de plusieurs repositories
- Transactions (`@Transactional`)
- Transformation Entity ↔ DTO

**Exemple** :
```java
@Service
@RequiredArgsConstructor
@Transactional
public class TransactionService {
    
    private final TransactionRepository transactionRepository;
    private final AccountRepository accountRepository;
    private final TransactionMapper mapper;
    
    public TransactionResponseDto createTransaction(TransactionRequestDto request) {
        // 1. Validation métier
        Account account = accountRepository.findById(request.getAccountId())
            .orElseThrow(() -> new ResourceNotFoundException("Account not found"));
        
        // 2. Logique métier
        if (request.getType() == TransactionType.EXPENSE) {
            validateSufficientBalance(account, request.getAmount());
        }
        
        // 3. Création et sauvegarde
        Transaction transaction = mapper.toEntity(request);
        transaction.setAccount(account);
        Transaction saved = transactionRepository.save(transaction);
        
        // 4. Mise à jour du solde
        updateAccountBalance(account, request);
        
        return mapper.toDto(saved);
    }
}
```

### 3. Persistence Layer (Couche de persistance)

**Responsabilité** : Accès aux données

#### Repositories
- Interface avec la base de données
- Méthodes CRUD automatiques via Spring Data JPA
- Requêtes personnalisées avec `@Query` ou méthodes dérivées

**Exemple** :
```java
@Repository
public interface TransactionRepository extends JpaRepository<Transaction, Long> {
    
    // Méthode dérivée (Spring Data génère la requête)
    List<Transaction> findByAccountIdAndDateBetween(
        Long accountId, 
        LocalDate startDate, 
        LocalDate endDate
    );
    
    // Requête personnalisée
    @Query("SELECT t FROM Transaction t WHERE t.account.user.id = :userId " +
           "AND t.date >= :startDate ORDER BY t.date DESC")
    List<Transaction> findUserTransactionsSince(
        @Param("userId") Long userId, 
        @Param("startDate") LocalDate startDate
    );
}
```

#### Entities
- Représentation objet des tables de base de données
- Annotations JPA (`@Entity`, `@Id`, `@ManyToOne`, etc.)
- Relations entre entités

## 📊 Modèle de données

### Diagramme ER (Entity-Relationship)

```
┌─────────────────┐
│      User       │
│─────────────────│
│ id (PK)         │
│ username        │◄──────────┐
│ email           │           │
│ password        │           │
│ createdAt       │           │
└─────────────────┘           │
        │                     │
        │ 1:N                 │
        ▼                     │
┌─────────────────┐           │
│    Account      │           │
│─────────────────│           │
│ id (PK)         │           │ N:1
│ name            │           │
│ type            │           │
│ balance         │           │
│ currency        │           │
│ userId (FK)     │───────────┘
└─────────────────┘
        │
        │ 1:N
        ▼
┌─────────────────┐       ┌─────────────────┐
│  Transaction    │       │    Category     │
│─────────────────│       │─────────────────│
│ id (PK)         │   N:1 │ id (PK)         │
│ description     │──────►│ name            │
│ amount          │       │ type            │
│ type            │       │ icon            │
│ date            │       │ color           │
│ accountId (FK)  │       │ userId (FK)     │
│ categoryId (FK) │       └─────────────────┘
└─────────────────┘
        │
        │ 1:N
        ▼
┌─────────────────┐
│     Budget      │
│─────────────────│
│ id (PK)         │
│ amount          │
│ month           │
│ year            │
│ categoryId (FK) │
│ userId (FK)     │
└─────────────────┘
```

### Entités principales

#### 1. User (Utilisateur)
```java
@Entity
@Table(name = "users")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(unique = true, nullable = false)
    private String username;
    
    @Column(unique = true, nullable = false)
    private String email;
    
    @Column(nullable = false)
    private String password;  // Hash BCrypt
    
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL)
    private List<Account> accounts;
    
    @CreatedDate
    private LocalDateTime createdAt;
    
    @Enumerated(EnumType.STRING)
    private Role role = Role.USER;
}
```

#### 2. Account (Compte bancaire)
```java
@Entity
@Table(name = "accounts")
@Data
public class Account {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String name;
    
    @Enumerated(EnumType.STRING)
    private AccountType type;  // CHECKING, SAVINGS, CREDIT_CARD
    
    @Column(precision = 19, scale = 4)
    private BigDecimal balance = BigDecimal.ZERO;
    
    private String currency = "EUR";
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", nullable = false)
    private User user;
    
    @OneToMany(mappedBy = "account", cascade = CascadeType.ALL)
    private List<Transaction> transactions;
}
```

#### 3. Transaction
```java
@Entity
@Table(name = "transactions")
@Data
public class Transaction {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String description;
    
    @Column(nullable = false, precision = 19, scale = 4)
    private BigDecimal amount;
    
    @Enumerated(EnumType.STRING)
    private TransactionType type;  // INCOME, EXPENSE
    
    @Column(nullable = false)
    private LocalDate date;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "account_id", nullable = false)
    private Account account;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "category_id")
    private Category category;
    
    @CreatedDate
    private LocalDateTime createdAt;
}
```

## 🔐 Architecture de sécurité

### Flux d'authentification JWT

```
┌────────┐                                    ┌────────┐
│ Client │                                    │ Server │
└────────┘                                    └────────┘
    │                                              │
    │  POST /api/auth/login                       │
    │  { username, password }                     │
    ├──────────────────────────────────────────►  │
    │                                              │
    │                                      ┌───────┴────────┐
    │                                      │ 1. Valider     │
    │                                      │    credentials │
    │                                      └───────┬────────┘
    │                                              │
    │                                      ┌───────┴────────┐
    │                                      │ 2. Générer JWT │
    │                                      └───────┬────────┘
    │                                              │
    │  200 OK                                      │
    │  { token: "eyJhbGc..." }                     │
    │  ◄──────────────────────────────────────────┤
    │                                              │
    │  GET /api/transactions                       │
    │  Authorization: Bearer eyJhbGc...            │
    ├──────────────────────────────────────────►  │
    │                                              │
    │                                      ┌───────┴────────┐
    │                                      │ 3. Valider JWT │
    │                                      └───────┬────────┘
    │                                              │
    │                                      ┌───────┴────────┐
    │                                      │ 4. Extraire    │
    │                                      │    user info   │
    │                                      └───────┬────────┘
    │                                              │
    │  200 OK                                      │
    │  { transactions: [...] }                     │
    │  ◄──────────────────────────────────────────┤
```

## 🔄 Flux de données typique

### Exemple : Création d'une transaction

```
Client                Controller              Service                Repository          Database
  │                       │                      │                        │                  │
  │ POST /transactions    │                      │                        │                  │
  │ + TransactionRequest  │                      │                        │                  │
  ├──────────────────────►│                      │                        │                  │
  │                       │                      │                        │                  │
  │                       │  1. Valider DTO      │                        │                  │
  │                       │     (@Valid)         │                        │                  │
  │                       │                      │                        │                  │
  │                       │  2. Appeler service  │                        │                  │
  │                       ├─────────────────────►│                        │                  │
  │                       │                      │                        │                  │
  │                       │                      │  3. Valider règles     │                  │
  │                       │                      │     métier             │                  │
  │                       │                      │                        │                  │
  │                       │                      │  4. Mapper DTO → Entity│                  │
  │                       │                      │                        │                  │
  │                       │                      │  5. Sauvegarder        │                  │
  │                       │                      ├───────────────────────►│                  │
  │                       │                      │                        │  INSERT          │
  │                       │                      │                        ├─────────────────►│
  │                       │                      │                        │                  │
  │                       │                      │                        │  Entity + ID     │
  │                       │                      │                        │◄─────────────────┤
  │                       │                      │  Entity                │                  │
  │                       │                      │◄───────────────────────┤                  │
  │                       │                      │                        │                  │
  │                       │                      │  6. Mapper Entity → DTO│                  │
  │                       │                      │                        │                  │
  │                       │  ResponseDto         │                        │                  │
  │                       │◄─────────────────────┤                        │                  │
  │                       │                      │                        │                  │
  │  201 Created          │                      │                        │                  │
  │  + TransactionResponse│                      │                        │                  │
  │◄──────────────────────┤                      │                        │                  │
```

## 🎯 Patterns de conception utilisés

### 1. Repository Pattern
- Abstraction de l'accès aux données
- Facilite les tests (mocking)

### 2. DTO Pattern
- Séparation API publique / modèle interne
- Sécurité (évite d'exposer des champs sensibles)

### 3. Service Layer Pattern
- Centralisation de la logique métier
- Réutilisabilité

### 4. Dependency Injection
- Couplage faible
- Testabilité

### 5. Builder Pattern (via Lombok)
- Construction d'objets complexes
- Code plus lisible

## 📝 Conventions de nommage

### Classes
- **Controllers** : `*Controller` (ex: `TransactionController`)
- **Services** : `*Service` (ex: `TransactionService`)
- **Repositories** : `*Repository` (ex: `TransactionRepository`)
- **Entities** : Nom singulier (ex: `Transaction`, `User`)
- **DTOs** : `*RequestDto`, `*ResponseDto` (ex: `TransactionRequestDto`)
- **Exceptions** : `*Exception` (ex: `ResourceNotFoundException`)

### Méthodes
- **Controllers** : verbes HTTP (ex: `create()`, `getAll()`, `update()`)
- **Services** : verbes métier (ex: `createTransaction()`, `calculateBalance()`)
- **Repositories** : préfixes Spring Data (ex: `findBy...`, `existsBy...`)

### URLs
- Toujours au pluriel : `/api/v1/transactions`
- Ressources imbriquées : `/api/v1/accounts/{id}/transactions`
- Actions : `/api/v1/transactions/{id}/categorize`

## 🧪 Stratégie de tests

### Pyramide des tests

```
           /\
          /  \         E2E Tests (peu nombreux)
         /────\        
        /      \       Integration Tests (quelques-uns)
       /────────\      
      /          \     Unit Tests (beaucoup)
     /____________\    
```

### Types de tests

1. **Tests unitaires** : Services et méthodes isolées
2. **Tests d'intégration** : Repositories + base de données
3. **Tests de contrôleurs** : MockMvc pour tester les endpoints
4. **Tests E2E** : Scénarios complets utilisateur

## 🚀 Prochaines étapes

Maintenant que tu comprends l'architecture, consulte :

➡️ **[Roadmap de développement](ROADMAP.md)** - Plan d'implémentation par phases
