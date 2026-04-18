# 🔧 Guide d'Installation

Ce guide t'accompagne dans la configuration complète de ton environnement de développement.

## Prérequis

### 1. Java Development Kit (JDK) 17+

**Vérifier si Java est installé :**
```bash
java -version
```

**Installation :**

- **Windows** : Télécharge [Amazon Corretto 17](https://docs.aws.amazon.com/corretto/latest/corretto-17-ug/downloads-list.html) ou [Oracle JDK](https://www.oracle.com/java/technologies/downloads/)
- **macOS** : 
  ```bash
  brew install openjdk@17
  ```
- **Linux (Ubuntu/Debian)** :
  ```bash
  sudo apt update
  sudo apt install openjdk-17-jdk
  ```

**Configurer JAVA_HOME :**

- **Windows** : 
  - Panneau de configuration → Système → Paramètres système avancés → Variables d'environnement
  - Nouvelle variable système : `JAVA_HOME` = `C:\Program Files\Java\jdk-17`
  - Ajouter à Path : `%JAVA_HOME%\bin`

- **macOS/Linux** (ajouter à `~/.bashrc` ou `~/.zshrc`) :
  ```bash
  export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
  export PATH=$JAVA_HOME/bin:$PATH
  ```

### 2. Maven 3.9+

**Vérifier l'installation :**
```bash
mvn -version
```

**Installation :**

- **Windows** : Télécharge [Maven](https://maven.apache.org/download.cgi) et ajoute `bin/` au Path
- **macOS** :
  ```bash
  brew install maven
  ```
- **Linux** :
  ```bash
  sudo apt install maven
  ```

### 3. PostgreSQL 15+

**Installation :**

- **Windows** : [PostgreSQL Installer](https://www.postgresql.org/download/windows/)
- **macOS** :
  ```bash
  brew install postgresql@15
  brew services start postgresql@15
  ```
- **Linux** :
  ```bash
  sudo apt install postgresql-15 postgresql-contrib
  sudo systemctl start postgresql
  ```

**Configuration initiale :**

```bash
# Se connecter à PostgreSQL
sudo -u postgres psql

# Dans psql, créer la base de données et l'utilisateur
CREATE DATABASE budget_db;
CREATE USER budget_user WITH ENCRYPTED PASSWORD 'budget_password';
GRANT ALL PRIVILEGES ON DATABASE budget_db TO budget_user;
\q
```

### 4. IDE (Environnement de développement)

**Recommandé : IntelliJ IDEA**
- [Community Edition (gratuit)](https://www.jetbrains.com/idea/download/)
- Configuration automatique de Spring Boot
- Meilleur support Maven et JPA

**Alternative : Eclipse / VSCode**
- Eclipse : [Télécharger](https://www.eclipse.org/downloads/)
- VSCode : Installer les extensions Java

### 5. Git

```bash
# Vérifier
git --version

# Installation
# Windows : https://git-scm.com/download/win
# macOS : brew install git
# Linux : sudo apt install git
```

**Configuration initiale :**
```bash
git config --global user.name "Ton Nom"
git config --global user.email "ton.email@example.com"
```

### 6. Postman (pour tester l'API)

Télécharge [Postman](https://www.postman.com/downloads/) ou utilise l'alternative [Insomnia](https://insomnia.rest/download).

## 📦 Initialisation du projet

### Option 1 : Utiliser Spring Initializr (Recommandé)

1. Va sur [start.spring.io](https://start.spring.io)
2. Configure :
   - **Project** : Maven
   - **Language** : Java
   - **Spring Boot** : 3.2.x (dernière stable)
   - **Group** : com.budget
   - **Artifact** : api
   - **Name** : Budget API
   - **Packaging** : Jar
   - **Java** : 17

3. Ajoute les dépendances :
   - Spring Web
   - Spring Data JPA
   - Spring Security
   - PostgreSQL Driver
   - Lombok
   - Validation
   - Spring Boot DevTools

4. Clique sur **Generate** et décompresse le fichier

### Option 2 : Cloner le projet de base

```bash
# Créer le dossier projet
mkdir budget-api
cd budget-api

# Initialiser git
git init
```

## 🔧 Configuration du projet

### 1. Structure Maven (pom.xml)

Crée ou vérifie ton `pom.xml` avec les dépendances nécessaires :

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.2</version>
        <relativePath/>
    </parent>
    
    <groupId>com.budget</groupId>
    <artifactId>api</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>Budget API</name>
    <description>Personal Budget Management API</description>
    
    <properties>
        <java.version>17</java.version>
    </properties>
    
    <dependencies>
        <!-- Spring Boot Starters -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        
        <!-- Database -->
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>
        
        <!-- Lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        
        <!-- JWT -->
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-api</artifactId>
            <version>0.11.5</version>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-impl</artifactId>
            <version>0.11.5</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-jackson</artifactId>
            <version>0.11.5</version>
            <scope>runtime</scope>
        </dependency>
        
        <!-- Swagger/OpenAPI -->
        <dependency>
            <groupId>org.springdoc</groupId>
            <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
            <version>2.3.0</version>
        </dependency>
        
        <!-- DevTools -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        
        <!-- Testing -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

### 2. Configuration de l'application (application.yml)

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
      ddl-auto: update  # En dev. Utiliser 'validate' en prod
    show-sql: true
    properties:
      hibernate:
        format_sql: true
        dialect: org.hibernate.dialect.PostgreSQLDialect
  
  security:
    jwt:
      secret-key: votre-secret-key-tres-securise-de-minimum-256-bits
      expiration: 86400000  # 24 heures en millisecondes

server:
  port: 8080
  error:
    include-message: always
    include-binding-errors: always

logging:
  level:
    com.budget.api: DEBUG
    org.springframework.security: DEBUG

springdoc:
  api-docs:
    path: /api-docs
  swagger-ui:
    path: /swagger-ui.html
```

### 3. Structure des dossiers

```bash
mkdir -p src/main/java/com/budget/api/{config,controller,dto,entity,repository,service,security,exception,util}
mkdir -p src/main/resources/db/migration
mkdir -p src/test/java/com/budget/api
```

## ✅ Vérification de l'installation

### 1. Compiler le projet

```bash
mvn clean install
```

**Résultat attendu** : `BUILD SUCCESS`

### 2. Créer la classe principale

Crée `src/main/java/com/budget/api/BudgetApiApplication.java` :

```java
package com.budget.api;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class BudgetApiApplication {
    public static void main(String[] args) {
        SpringApplication.run(BudgetApiApplication.class, args);
    }
}
```

### 3. Lancer l'application

```bash
mvn spring-boot:run
```

**Résultat attendu** :
```
Started BudgetApiApplication in X.XXX seconds
```

### 4. Tester la connexion

Ouvre ton navigateur : `http://localhost:8080`

Tu devrais voir une page de login Spring Security (normal, on n'a pas encore de endpoints).

## 🎉 Installation terminée !

Tu es maintenant prêt à commencer le développement. Passe à la suite :

➡️ **Prochaine étape** : [Architecture détaillée](ARCHITECTURE.md)

## 🆘 Problèmes courants

### Erreur de connexion PostgreSQL
```
Cause: org.postgresql.util.PSQLException: Connection refused
```
**Solution** : Vérifie que PostgreSQL est démarré
```bash
# Linux/macOS
sudo systemctl status postgresql
# ou
brew services list

# Redémarrer si nécessaire
sudo systemctl start postgresql
```

### Port 8080 déjà utilisé
**Solution** : Change le port dans `application.yml`
```yaml
server:
  port: 8081
```

### Maven ne télécharge pas les dépendances
**Solution** : Supprime le cache et relance
```bash
mvn clean
mvn dependency:purge-local-repository
mvn install
```

### Lombok ne fonctionne pas dans l'IDE
**IntelliJ** : 
1. File → Settings → Plugins
2. Chercher "Lombok" et installer
3. File → Settings → Build → Compiler → Annotation Processors
4. Cocher "Enable annotation processing"

**Eclipse** :
1. Télécharger `lombok.jar`
2. Exécuter : `java -jar lombok.jar`
3. Sélectionner Eclipse et installer
