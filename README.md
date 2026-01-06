# Test de charge & Observabilité

Microservices Spring Boot avec gestion de concurrence, verrous DB, Resilience4j et métriques Actuator.

## Architecture

**Services:**

- **Book Service** (3 instances: 8081, 8083, 8084): Gestion de livres avec verrous DB
- **Pricing Service** (8082): Tarification avec fallback
- **MySQL** (3306): Base de données bookdb

## Setup

```bash
# Démarrer tous les services
docker-compose up -d

# Rebuild si nécessaire
docker-compose up -d --build

# Vérifier l'état
docker-compose ps
docker logs tp26-book-service-1 --tail 100

# Arrêter
docker-compose down
```

## Demo

![Architecture](image-5.png)
![Services](image-6.png)

### Test de charge: 50 emprunts parallèles

![Load Test](image-7.png)
![Results](image-8.png)

### Verrou DB: Stock jamais négatif

![DB Lock](image-9.png)

### Résilience: Pricing down → Fallback

![Resilience 1](image-10.png)
![Resilience 2](image-11.png)
![Resilience 3](image-12.png)
![Resilience 4](image-13.png)

## Test de Charge

**PowerShell (Windows):**

```powershell
.\loadtest.ps1 -BookId 1 -Requests 50
```

**Bash (Linux/Mac):**

```bash
chmod +x loadtest.sh
./loadtest.sh 1 50
```

**Résultats attendus (stock initial = 10, requests = 50):**

- **200 (Success)**: ~10 emprunts réussis
- **409 (Conflict)**: ~40 stock épuisé (normal)
- **Other**: 0 (erreurs si > 0)

Les requêtes sont réparties sur les 3 instances automatiquement.

## Endpoints

**Book Service:**

- Instance 1: http://localhost:8081
- Instance 2: http://localhost:8083
- Instance 3: http://localhost:8084
- Health: `/actuator/health`
- API: `/api/books`

**Pricing Service:**

- URL: http://localhost:8082
- Health: `/actuator/health`
- API: `/api/pricing`

**MySQL:**

- Host: localhost:3306
- Database: bookdb
- User: bookuser / bookpass

## Debug

**JAR corrompu:**

```bash
cd <service-name>
./mvnw clean package -DskipTests
docker-compose build --no-cache <service-name>
docker-compose up -d --no-deps <service-name>
```

**MySQL:**

```bash
docker exec -it tp26-mysql-1 mysql -u bookuser -pbookpass bookdb
```

## Technologies

- Spring Boot 3.2.1
- Java 21
- MySQL
- Docker & Docker Compose
- Resilience4j
- Spring Boot Actuator
