# Orion-HQ Build and Deployment

## 1. Purpose and Scope
Orion-HQ is the local orchestration stack: Python API, Python agent gateway, Vue/Vite web application, PostgreSQL, MinIO, Redis, and Keycloak. The root Makefile owns the development lifecycle, while `infra/docker-compose.yml`, the two Python `pyproject.toml` files, and `web/package.json` define the services and dependencies.

## 2. Prerequisites
- Docker and Docker Compose, Python 3, Node.js/npm, curl, and `make doctor` success.
- Network access to install the API/gateway editable dev packages and web dependencies.
- Local ports for Postgres 5432, MinIO 9000, Keycloak 8081, API 8000, web 5173, and gateway 50051.
- Dev credentials only: `make seed EMAIL=...` requires a chosen email; the compose stack contains development secrets and must not be treated as production configuration.

## 3. Setup
```bash
make doctor
make install
make db-up
make migrate
make seed EMAIL=admin@example.com
```
`install` creates `.venv`, installs `./api[dev]` and `./agent-gateway[dev]` editable, and runs `npm install` in `web/`. `db-up` starts dependencies, waits for PostgreSQL/MinIO/Keycloak, creates `orionhq_test`, and creates the `orionhq` bucket.

## 4. Build and Compile
```bash
make build
```
This runs `npm run build` in `web/` and creates the production web bundle. The Makefile has no Python binary packaging target; API and gateway are run from the venv. Production images/deploy manifests are not produced by this root Makefile.

## 5. Test and Lint
```bash
make test
make test-api
make test-gateway
make test-web
make lint
make lint-api
make lint-gateway
make lint-web
```
API and gateway pytest commands use `ORIONHQ_TEST_DATABASE_URL` and `ORIONHQ_S3_ENDPOINT_URL`; the web command runs `npx vitest run`. Lint uses Ruff for Python and npm ESLint plus `vue-tsc --noEmit` for the web. Ensure `db-up` and `make install` have completed before integration tests.

## 6. Packaging and Artifacts
The repository does not define a packaging target. Web artifacts are under `web/dist` for the Vite build flow and the Next/Vite package output as configured by the web project; Python installs remain in `.venv`. Runtime logs and PID files are under `.run`. Publishing containers or deploying to cloud infrastructure requires a separate credentialed pipeline and is outside this Makefile.

## 7. Deploy, Install, and Run
```bash
make run
make logs
make stop
make db-down
```
`run` starts dependencies, applies migrations, and launches API, gateway, and web in the background with PID/log files. It prints LAN URLs for web (5173), API (8000), Keycloak admin (8081), and gateway (50051). Use the printed LAN IP consistently because OIDC redirects/cookies are origin-sensitive. The development login is `admin@example.com` / `devpassword` after seeding; replace it for any non-development environment. `stop` leaves Docker data services running; use `db-down` to stop them.

## 8. Cleanup
```bash
make clean
make db-down
```
`clean` removes the venv, `.run`, web dependencies/dist/cache, Python caches, and test/lint caches. `db-down` stops containers but intentionally preserves Docker volumes and database data.

## 9. Hardware, Cloud, and Manual Verification Caveats
HQ itself has no direct hardware requirement, but meaningful acceptance requires a reachable Orion client/HAL and real authentication/storage flows. Manually verify migrations, tenant/admin seed, OIDC redirects, MinIO uploads, gRPC gateway connectivity, SSE/live events, and API authorization. No cloud credentials are supplied by the Makefile; configure external identity, object storage, database, TLS, secrets, and network policy outside this development stack.
