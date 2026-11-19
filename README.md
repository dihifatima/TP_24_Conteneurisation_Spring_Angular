# Smart House

Application web de gestion d'une maison connectée : on y **ajoute, classe par catégorie, active/désactive et supprime** les appareils de la maison (lampes, climatiseur, TV, etc.).

- **Backend** : API REST Spring Boot 3 (Java 17) + MySQL
- **Frontend** : Angular 14 + PrimeNG / Tailwind CSS / Bootstrap
- **Déploiement** : Docker Compose (MySQL, backend, frontend/Nginx, phpMyAdmin)

---

## Fonctionnalités

| Module | Fonctionnalités |
|---|---|
| **Appareils** | Lister, ajouter (avec photo), supprimer, allumer / éteindre un appareil, **tout allumer / tout éteindre** en un clic |
| **Catégories** | Lister, ajouter, modifier, supprimer |

Chaque appareil appartient à une catégorie (relation `ManyToOne`). La photo est envoyée en base64 et stockée en `LONGTEXT`.

---

## Architecture

```
smarthouse/
├── Smart_Home_back/          # API Spring Boot
│   ├── src/main/java/com/example/appareil/
│   │   ├── entity/           # Appareil, Categorie
│   │   ├── repository/       # Spring Data JPA
│   │   ├── service/          # Logique métier
│   │   └── cotroller/        # Contrôleurs REST
│   ├── src/main/resources/application.properties
│   ├── pom.xml
│   └── Dockerfile            # Build Maven multi-stage → JRE 17 Alpine
├── smartHome-front/          # Application Angular
│   ├── src/app/
│   │   ├── modules/          # navbar, categories, list-appareil
│   │   └── controller/       # model, service (HttpClient), utils
│   ├── src/environments/     # URL de l'API
│   └── Dockerfile            # Build Node 14 → Nginx
├── docker-compose.yml
└── .env
```

### Modèle de données

```
Categorie (id, label)  1 ───── *  Appareil (id, label, description, state, photo, categorie_id)
```

---

## Stack technique

| Couche | Technologies |
|---|---|
| Backend | Java 17, Spring Boot 3.1.5, Spring Web, Spring Data JPA, Spring Data REST, Lombok, MySQL Connector/J |
| Base de données | MySQL 8.0 (schéma généré automatiquement : `ddl-auto=update`) |
| Frontend | Angular 14, TypeScript 4.7, PrimeNG 14, PrimeFlex, PrimeIcons, Tailwind CSS 3, Bootstrap 5 |
| DevOps | Docker, Docker Compose, Nginx, phpMyAdmin |

---

## Démarrage rapide (Docker)

**Prérequis** : Docker et Docker Compose.

```bash
git clone <url-du-depot>
cd smarthouse
docker compose up --build
```

| Service | URL |
|---|---|
| Frontend | http://localhost |
| API backend | http://localhost:8085/api/controller |
| phpMyAdmin | http://localhost:8081 |
| MySQL | `localhost:3306` |

Pour arrêter : `docker compose down` (ajouter `-v` pour supprimer aussi les données MySQL).

---

## Lancement en local (sans Docker)

### 1. Base de données

Créer la base MySQL :

```sql
CREATE DATABASE `smart-house`;
```

Vérifier les identifiants dans `Smart_Home_back/src/main/resources/application.properties` (par défaut : `root` sans mot de passe sur `localhost:3306`).

### 2. Backend

**Prérequis** : JDK 17.

```bash
cd Smart_Home_back
./mvnw spring-boot:run
```

L'API démarre sur http://localhost:8085.

### 3. Frontend

**Prérequis** : Node.js 14+ et npm.

```bash
cd smartHome-front
npm install
npm start
```

L'application est servie sur http://localhost:4200.

---

## API REST

Base URL : `http://localhost:8085/api/controller`

### Appareils — `/appareil`

| Méthode | Endpoint | Description |
|---|---|---|
| `GET` | `/appareil/` | Liste de tous les appareils |
| `GET` | `/appareil/id/{id}` | Détail d'un appareil |
| `POST` | `/appareil/` | Créer un appareil (la catégorie doit exister) |
| `PUT` | `/appareil/id/{id}` | Modifier l'état (`state`) d'un appareil |
| `DELETE` | `/appareil/id/{id}` | Supprimer un appareil |
| `GET` | `/appareil/update/state/{state}` | Allumer (`true`) / éteindre (`false`) tous les appareils |

### Catégories — `/categorie`

| Méthode | Endpoint | Description |
|---|---|---|
| `GET` | `/categorie/` | Liste des catégories (avec leurs appareils) |
| `GET` | `/categorie/id/{id}` | Détail d'une catégorie |
| `POST` | `/categorie/` | Créer une catégorie |
| `PUT` | `/categorie/id/{id}` | Modifier le libellé |
| `DELETE` | `/categorie/id/{id}` | Supprimer une catégorie |

### Exemples

```bash
# Créer une catégorie
curl -X POST http://localhost:8085/api/controller/categorie/ \
  -H "Content-Type: application/json" \
  -d '{"label": "Éclairage"}'

# Créer un appareil dans cette catégorie
curl -X POST http://localhost:8085/api/controller/appareil/ \
  -H "Content-Type: application/json" \
  -d '{"label": "Lampe salon", "description": "LED 12W", "state": false, "categorie": {"id": 1}}'

# Tout éteindre
curl http://localhost:8085/api/controller/appareil/update/state/false
```

---

## Configuration

| Paramètre | Fichier | Valeur par défaut |
|---|---|---|
| Port backend | `application.properties` | `8085` |
| URL MySQL | `application.properties` / variable `SPRING_DATASOURCE_URL` | `jdbc:mysql://localhost:3306/smart-house` |
| Identifiants MySQL | `SPRING_DATASOURCE_USERNAME` / `SPRING_DATASOURCE_PASSWORD` | `root` / *(vide)* en local, `root` / `root` avec Docker |
| URL de l'API côté front | `smartHome-front/src/environments/environment*.ts` | `http://localhost:8085/api/controller/` |

> **Sécurité** : les mots de passe de `docker-compose.yml` et le fichier `.env` sont prévus pour le développement. Ne pas les utiliser en production et ne pas committer de secrets réels.

---

## Tests

```bash
# Backend
cd Smart_Home_back && ./mvnw test

# Frontend
cd smartHome-front && npm test
```

---
<img width="1902" height="965" alt="image" src="https://github.com/user-attachments/assets/ae37fc8d-4540-4ab2-a38e-db3d72aa27ed" />
<img width="1465" height="361" alt="image" src="https://github.com/user-attachments/assets/0321331b-f975-47dd-bd55-fe86d908f97c" />


## Pistes d'amélioration

- Authentification et gestion des utilisateurs (Spring Security + JWT)
- Stockage des photos sur disque / object storage plutôt qu'en base64
- Gestion d'erreurs centralisée (`@ControllerAdvice`) et validation des entrées
- Communication temps réel avec les appareils (MQTT / WebSocket)
- Tests unitaires et d'intégration

---

## Auteur

Projet réalisé par **Dihi Fatima** — EMSI Marrakech.
