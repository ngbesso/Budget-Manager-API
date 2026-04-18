# 🚀 Quick Start - Budget Manager API

Bienvenue ! Ce guide te permet de démarrer rapidement.

## ⚡ En 3 étapes

### 1. Installer les outils nécessaires

- ☕ **Java 17+** : [Télécharger](https://www.oracle.com/java/technologies/downloads/#java17)
- 📦 **Maven 3.9+** : [Télécharger](https://maven.apache.org/download.cgi)
- 🐘 **PostgreSQL 15+** : [Télécharger](https://www.postgresql.org/download/)
- 💻 **IntelliJ IDEA** : [Télécharger Community Edition](https://www.jetbrains.com/idea/download/)

### 2. Créer la base de données

```bash
# Ouvrir le terminal PostgreSQL
psql -U postgres

# Dans psql, exécuter :
CREATE DATABASE budget_db;
CREATE USER budget_user WITH ENCRYPTED PASSWORD 'budget_password';
GRANT ALL PRIVILEGES ON DATABASE budget_db TO budget_user;
\q
```

### 3. Créer le projet Spring Boot

1. Va sur [start.spring.io](https://start.spring.io)
2. Configure :
   - Project: **Maven**
   - Language: **Java**
   - Spring Boot: **3.2.2** (ou dernière version stable)
   - Group: **com.budget**
   - Artifact: **api**
   - Java: **17**

3. Ajoute ces dépendances :
   - Spring Web
   - Spring Data JPA
   - Spring Security
   - PostgreSQL Driver
   - Lombok
   - Validation
   - Spring Boot DevTools

4. Clique **Generate** et décompresse le ZIP

5. Ouvre le projet dans IntelliJ IDEA

## 📝 Configuration

### Fichier `application.yml`

Crée `src/main/resources/application.yml` :

```yaml
spring:
  application:
    name: budget-api
  
  datasource:
    url: jdbc:postgresql://localhost:5432/budget_db
    username: budget_user
    password: budget_password
    driver-class-name: org.postgresql.Driver
  
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        format_sql: true
        dialect: org.hibernate.dialect.PostgreSQLDialect

server:
  port: 8080

logging:
  level:
    com.budget.api: DEBUG
```

## ✅ Vérifier l'installation

```bash
# Dans le terminal, à la racine du projet
mvn clean install

# Si succès, lancer l'application
mvn spring-boot:run
```

Tu devrais voir :
```
Started BudgetApiApplication in X.XXX seconds
```

## 📚 Prochaines étapes

Maintenant que ton environnement est prêt :

1. **[Lis l'architecture](docs/ARCHITECTURE.md)** pour comprendre la structure
2. **[Consulte la roadmap](docs/ROADMAP.md)** pour le plan d'implémentation
3. **[Commence par la Phase 1](docs/ROADMAP.md#phase-1-fondations-du-projet)** : Créer l'entité User

## 📖 Documentation complète

- [Guide d'installation détaillé](docs/INSTALLATION.md)
- [Architecture du projet](docs/ARCHITECTURE.md)
- [Roadmap de développement](docs/ROADMAP.md)
- [Standards de code](docs/CODING_STANDARDS.md)
- [Guide API](docs/API_GUIDE.md)

## 🆘 Besoin d'aide ?

### Problèmes courants

**L'application ne démarre pas**
```bash
# Vérifier que PostgreSQL tourne
sudo systemctl status postgresql  # Linux
brew services list                # macOS
```

**Port 8080 déjà utilisé**
- Change le port dans `application.yml` : `server.port: 8081`

**Erreur de connexion DB**
- Vérifie que la base de données existe
- Vérifie username/password dans `application.yml`

## 🎯 Ton premier objectif

**Créer le CRUD User complet** (Semaines 1-3)

Suis la [Phase 1 de la roadmap](docs/ROADMAP.md#phase-1-fondations-du-projet) étape par étape.

**Bonne chance ! 🚀**
