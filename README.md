# skill-config-service

Spring Cloud Config Server repository for the skill-engine platform.

## Layout

```
<service>/application.yml        # shared defaults (every environment)
<service>/application-prod.yml   # overrides applied only when profile "prod" is active
```

Services: `auth-service`, `gateway`, `skill-engine`, `worker-service`, `job-service`, `admin-bff`.

The Config Server uses `search-paths: "{application}"`, so for an app named
`auth-service` it reads files inside the `auth-service/` directory.

## Profiles

The profile is chosen by the client via `SPRING_PROFILES_ACTIVE`.

- No profile set  -> Config Server serves `/auth-service/default` (application.yml only)
- `prod` active    -> Config Server serves `/auth-service/prod`  (application.yml + application-prod.yml, prod wins)

To activate for a service, add to `infra/docker-compose.yml`:

```yaml
environment:
  SPRING_PROFILES_ACTIVE: prod
```

## Precedence (important)

Spring resolves properties in this order, later wins:

1. `application.yml` (packaged) + Config Server (imported)  -> lowest
2. OS environment variables (docker-compose `environment:`)  -> higher
3. Java system properties / command line                     -> highest

So an env var in docker-compose overrides the Config Server. Tunables that should be
changeable from here must NOT also be set in docker-compose. Secrets and DB credentials
are intentionally kept in the compose environment.

## Applying a change at runtime (no container restart)

```bash
# 1. commit + push this repo
# 2. make Config Server re-pull
docker exec config-server curl -s -X POST http://localhost:8888/actuator/refresh
# 3. reload the affected service
docker exec <service> curl -k -s -X POST https://localhost:8443/actuator/refresh
```

Note: the `prod` profile disables the `refresh` endpoint (exposure = health,info) for safety.
