# 🗺️ Roadmap de développement

Ce document détaille le plan d'implémentation phase par phase avec des objectifs concrets et progressifs.

## 🎯 Vue d'ensemble

Le projet est divisé en **5 phases** qui te permettent d'apprendre progressivement tout en construisant une application fonctionnelle.

```
Phase 1: Fondations      → 2-3 semaines
Phase 2: CRUD Basique    → 2-3 semaines  
Phase 3: Sécurité        → 1-2 semaines
Phase 4: Features        → 2-3 semaines
Phase 5: Polish          → 1-2 semaines
──────────────────────────────────────
Total estimé: 8-13 semaines
```

---

## 📅 PHASE 1 : Fondations du projet

**Durée estimée** : 2-3 semaines  
**Objectif** : Configuration complète et première entité fonctionnelle

### ✅ Semaine 1 : Configuration de l'environnement

- [ ] Installer Java 17, Maven, PostgreSQL, IDE
- [ ] Créer le projet Spring Boot avec Spring Initializr
- [ ] Configurer pom.xml avec toutes les dépendances
- [ ] Créer application.yml avec configuration DB
- [ ] Tester la connexion à la base de données
- [ ] Configurer Git et créer le repository
- [ ] Créer la structure de dossiers
- [ ] Écrire le fichier .gitignore

**Livrables** :
- Application Spring Boot qui démarre sans erreur
- Connexion PostgreSQL fonctionnelle
- Repository Git initialisé

### ✅ Semaine 2 : Première entité User

- [ ] Créer l'entité User avec annotations JPA
- [ ] Créer UserRepository extends JpaRepository
- [ ] Créer les DTOs (UserRequestDto, UserResponseDto)
- [ ] Créer UserMapper pour convertir Entity ↔ DTO
- [ ] Créer UserService avec méthode createUser()
- [ ] Créer UserController avec endpoint POST /api/v1/users

**Tests** :
- [ ] Tester avec Postman : créer un utilisateur
- [ ] Vérifier dans PostgreSQL que l'utilisateur est créé

### ✅ Semaine 3 : Compléter CRUD User

- [ ] Ajouter méthode getAll() dans le service
- [ ] Ajouter endpoint GET /api/v1/users
- [ ] Ajouter méthode getById(Long id)
- [ ] Ajouter endpoint GET /api/v1/users/{id}
- [ ] Ajouter méthode update()
- [ ] Ajouter endpoint PUT /api/v1/users/{id}
- [ ] Ajouter méthode delete()
- [ ] Ajouter endpoint DELETE /api/v1/users/{id}
- [ ] Gérer les exceptions (ResourceNotFoundException)
- [ ] Ajouter validation avec @Valid et annotations

**Tests** :
- [ ] Tester tous les endpoints CRUD avec Postman
- [ ] Tester les cas d'erreur

### 🎓 Concepts appris en Phase 1

- Structure projet Spring Boot
- Annotations JPA (@Entity, @Id, @Column)
- Repository pattern avec Spring Data JPA
- DTO pattern et séparation des responsabilités
- Architecture en couches
- Validation des données
- Gestion des exceptions
- Requêtes HTTP (GET, POST, PUT, DELETE)

---

## 📅 PHASE 2 : Entités liées et relations

**Durée estimée** : 2-3 semaines  
**Objectif** : Implémenter Account, Transaction et Category avec leurs relations

### ✅ Semaine 4 : Entity Account

- [ ] Créer l'entité Account
- [ ] Créer enum AccountType
- [ ] Ajouter relation dans User : @OneToMany List<Account> accounts
- [ ] Créer AccountRepository, DTOs, Service, Controller
- [ ] Implémenter CRUD complet pour Account
- [ ] Endpoint : GET /api/v1/users/{userId}/accounts

**Tests** :
- [ ] Créer un compte pour un utilisateur
- [ ] Récupérer tous les comptes d'un utilisateur

### ✅ Semaine 5 : Entity Category

- [ ] Créer l'entité Category
- [ ] Créer des catégories système (prédéfinies)
- [ ] Implémenter CRUD pour Category
- [ ] Permettre aux utilisateurs de créer leurs propres catégories

### ✅ Semaine 6 : Entity Transaction

- [ ] Créer l'entité Transaction
- [ ] Créer enum TransactionType
- [ ] Implémenter CRUD pour Transaction
- [ ] Endpoint : GET /api/v1/accounts/{accountId}/transactions
- [ ] Implémenter filtrage par date
- [ ] Implémenter filtrage par type (income/expense)
- [ ] Mettre à jour le solde du compte lors d'une transaction
- [ ] Valider que le solde ne devient pas négatif

### 🎓 Concepts appris en Phase 2

- Relations JPA (@ManyToOne, @OneToMany)
- Fetch types (LAZY vs EAGER)
- Enum avec @Enumerated
- Requêtes dérivées Spring Data
- Logique métier dans les services
- Transactions base de données avec @Transactional
- BigDecimal pour les montants financiers
- Gestion des dates avec LocalDate

---

## 📅 PHASE 3 : Sécurité et authentification

**Durée estimée** : 1-2 semaines  
**Objectif** : Implémenter JWT et Spring Security

### ✅ Semaine 7 : Configuration Spring Security

- [ ] Ajouter dépendances JWT dans pom.xml
- [ ] Créer JwtService pour générer/valider tokens
- [ ] Créer JwtAuthenticationFilter
- [ ] Créer SecurityConfig avec configuration
- [ ] Hasher les mots de passe avec BCrypt

### ✅ Semaine 8 : Endpoints d'authentification

- [ ] Créer AuthController
- [ ] Endpoint POST /api/auth/register
- [ ] Endpoint POST /api/auth/login
- [ ] Protéger tous les endpoints (sauf auth)
- [ ] Récupérer l'utilisateur authentifié dans les controllers
- [ ] Vérifier que les utilisateurs accèdent uniquement à leurs données

**Tests** :
- [ ] Tester registration et login
- [ ] Tester accès avec et sans token
- [ ] Tester accès à des ressources d'un autre utilisateur

### 🎓 Concepts appris en Phase 3

- Spring Security et chaîne de filtres
- JWT (JSON Web Tokens)
- Authentification vs Autorisation
- BCrypt pour hasher les mots de passe
- SecurityContextHolder
- Annotations @PreAuthorize
- Gestion des rôles

---

## 📅 PHASE 4 : Fonctionnalités avancées

**Durée estimée** : 2-3 semaines  
**Objectif** : Budget, statistiques et recherche

### ✅ Semaine 9 : Entity Budget

- [ ] Créer l'entité Budget
- [ ] Implémenter CRUD pour Budget
- [ ] Calculer automatiquement le montant dépensé
- [ ] Endpoint : GET /api/v1/budgets/status

### ✅ Semaine 10 : Statistiques et rapports

- [ ] Endpoint : GET /api/v1/statistics/summary
- [ ] Endpoint : GET /api/v1/statistics/by-category
- [ ] Endpoint : GET /api/v1/statistics/trends

### ✅ Semaine 11 : Recherche et filtrage

- [ ] Recherche de transactions
- [ ] Pagination avec Pageable
- [ ] Tri des résultats
- [ ] Endpoint : GET /api/v1/transactions/search

### 🎓 Concepts appris en Phase 4

- Requêtes complexes avec JPA Criteria API
- Pagination et tri avec Spring Data
- @Query pour requêtes personnalisées JPQL
- Agrégations (SUM, AVG, COUNT)
- DTOs pour rapports complexes

---

## 📅 PHASE 5 : Polish et documentation

**Durée estimée** : 1-2 semaines  
**Objectif** : Tests, documentation et optimisations

### ✅ Semaine 12 : Tests

- [ ] Tests unitaires pour les Services
- [ ] Tests d'intégration pour les Repositories
- [ ] Tests de Controllers avec MockMvc
- [ ] Atteindre 70%+ de couverture de code

### ✅ Semaine 13 : Documentation et finitions

- [ ] Configurer Swagger/OpenAPI
- [ ] Documenter tous les endpoints
- [ ] Créer un fichier README.md complet
- [ ] Ajouter des index sur les colonnes
- [ ] Optimiser les requêtes N+1
- [ ] Handler global avec @ControllerAdvice
- [ ] Messages d'erreur standardisés

### 🎓 Concepts appris en Phase 5

- JUnit 5 et assertions
- Mockito pour mocker les dépendances
- @WebMvcTest et @DataJpaTest
- Swagger/OpenAPI annotations
- @ControllerAdvice
- Optimisations de performance

---

## 📊 Progression recommandée

| Semaine | Focus | Livrables |
|---------|-------|-----------|
| 1 | Setup | Projet démarré, DB connectée |
| 2 | User CRUD | Endpoints User fonctionnels |
| 3 | Validation | CRUD User complet |
| 4 | Account | Relation User-Account |
| 5 | Category | Catégories système |
| 6 | Transaction | CRUD Transaction |
| 7 | Security | JWT implémenté |
| 8 | Auth | Login/Register |
| 9 | Budget | Gestion des budgets |
| 10 | Stats | Rapports et statistiques |
| 11 | Search | Recherche avancée |
| 12 | Tests | Couverture tests 70%+ |
| 13 | Doc | Swagger + README |

## 💡 Conseils pour réussir

### Stratégie d'apprentissage

1. **Ne saute pas d'étapes**
2. **Teste au fur et à mesure**
3. **Commit régulièrement**
4. **Documente pendant**
5. **Demande de l'aide** si tu bloques plus de 2h

### Code de qualité

- **DRY** (Don't Repeat Yourself)
- **SOLID** : Principes de conception objet
- **Clean Code** : Noms descriptifs, fonctions courtes
- **Tests** : Pour la logique métier critique

---

**Prêt à commencer ?** → Va à [Standards de code](CODING_STANDARDS.md)

