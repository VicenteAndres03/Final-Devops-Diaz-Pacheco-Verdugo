# EFT - Introducción a Herramientas DevOps (ISY1101)

Automatización del ciclo de integración y entrega continua (CI/CD) de una
plataforma compuesta por 2 microservicios backend (Spring Boot + MySQL) y un
frontend (React), contenedorizada con Docker y desplegada en AWS ECS (Fargate)
mediante un pipeline de GitHub Actions.

## Arquitectura

- **front_despacho**: Frontend en React (Vite), servido con Nginx.
- **back-Ventas_SpringBoot**: Microservicio de gestión de ventas (Spring Boot 3.4 + Java 17).
- **back-Despachos_SpringBoot**: Microservicio de gestión de despachos (Spring Boot 3.4 + Java 17).
- **MySQL 8.0**: Base de datos relacional compartida por ambos microservicios.

Ver el diagrama de arquitectura completo en el informe.

## Estructura del repositorio

├── back-Ventas_SpringBoot/
│   └── Springboot-API-REST/       # Dockerfile, pom.xml, código fuente
├── back-Despachos_SpringBoot/
│   └── Springboot-API-REST-DESPACHO/
├── front_despacho/                 # Dockerfile, nginx.conf, código fuente
├── docker-compose.yml               # Orquestación local
├── .github/workflows/ci-cd.yml      # Pipeline CI/CD
└── README.md

## Cómo correr el proyecto en local

```bash
cp .env.example .env
# completar .env con credenciales locales de MySQL

docker compose up --build
```

- Frontend: http://localhost:3000
- Back-Ventas: http://localhost:8081/swagger-ui/index.html
- Back-Despachos: http://localhost:8082/swagger-ui/index.html
- MySQL: localhost:3306

## CI/CD

El pipeline (`.github/workflows/ci-cd.yml`) se ejecuta en cada push a `main`:

1. **Build & Test**: compila y testea cada microservicio con Maven (usando H2 en memoria para los tests).
2. **Build de imágenes Docker** para los 3 componentes.
3. **Push a Amazon ECR** con tag `latest` y tag del commit (trazabilidad).

Los secretos de AWS se gestionan mediante GitHub Secrets (`AWS_ACCESS_KEY_ID`,
`AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN` — este último necesario por
tratarse de credenciales temporales de AWS Academy).

## Despliegue en AWS

- **Clúster ECS**: `eft2-cluster` (Fargate, sin servidor).
- **Servicios**: uno por componente (MySQL, back-ventas, back-despachos, frontend).
- **Registro de imágenes**: Amazon ECR, un repositorio por componente.
- **Seguridad**: Security Group (`eft-sg`) con reglas de entrada para los puertos 80, 8080 y 3306.

## Observabilidad

- **Logs**: cada tarea ECS envía logs a CloudWatch Logs (`/ecs/eft-*`).
- **Métricas**: CloudWatch Metrics expone CPU/memoria de los servicios ECS.
- **Logs del pipeline**: disponibles en la pestaña *Actions* de GitHub.

## Autores

Diaz, Pacheco, Verdugo — ISY1101 Introducción a Herramientas DevOps, DuocUC 2025.
