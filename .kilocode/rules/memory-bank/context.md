# Context

## Current State
Java Spring Boot scaffolder template has been implemented. The GitHub Actions plugin has been wired into the CI/CD tab. CI/CD workflow fixed to specify gradle-version so it doesn't require gradlew wrapper scripts.

## What Was Done
1. Created `examples/spring-boot-template/template.yaml` - Scaffolder template definition with wizard form (name, description, owner, javaPackage, springBootVersion, repoUrl)
2. Created template content skeleton under `examples/spring-boot-template/content/`:
   - `catalog-info.yaml` - Backstage catalog descriptor with `github.com/project-slug` annotation
   - `build.gradle` - Spring Boot + Gradle build with Java 21 toolchain
   - `settings.gradle` - Gradle settings
   - `gradle/wrapper/gradle-wrapper.properties` - Gradle 8.13 wrapper config
   - `Dockerfile` - Multi-stage build (Gradle build -> Eclipse Temurin 21 JRE)
   - `.gitignore` - Java/Gradle gitignore
   - `README.md` - Project documentation
   - `.github/workflows/ci-cd.yml` - GitHub Actions pipeline (build, test, deploy to GHCR)
   - `src/main/java/${{values.javaPackageDir}}/Application.java` - Spring Boot main class
   - `src/main/java/${{values.javaPackageDir}}/controller/HealthController.java` - Health endpoint
   - `src/main/resources/application.yml` - Spring config with actuator
   - `src/test/java/${{values.javaPackageDir}}/ApplicationTests.java` - Context load test
3. Added Spring Boot template location to `app-config.yaml` catalog locations
4. Installed `@backstage-community/plugin-github-actions` (v0.21.1) in frontend package
5. Updated `EntityPage.tsx` - Replaced CI/CD placeholder with GitHub Actions plugin, removed unused Button import
6. Fixed CI/CD workflow: added `checks: write` permission (dorny/test-reporter needs it for GitHub Checks API)
7. Migrated from deprecated `gradle/gradle-build-action@v3` to `gradle/actions/setup-gradle@v3` with separate run steps

## What Does Not Exist Yet
- gradlew / gradlew.bat wrapper scripts (not needed for CI since gradle-build-action now uses gradle-version: '8.13'; README instructs `gradle wrapper` for local dev)
- Kubernetes cluster configuration
- Proper RBAC (still using allow-all policy)
- Production catalog sources
- End-to-end testing of the full scaffolder flow

## Next Steps
1. Test the full flow locally with `yarn start` (requires GITHUB_TOKEN env var)
2. Verify the template appears in the "Create..." menu
3. Test scaffolding a project to confirm Nunjucks templating works correctly
4. Verify CI/CD tab shows GitHub Actions for entities with the project-slug annotation

## Recent Changes
- Fixed CI/CD workflow: added `checks: write` permission so dorny/test-reporter can create GitHub Check Runs
- Migrated CI/CD workflow from deprecated `gradle/gradle-build-action@v3` to `gradle/actions/setup-gradle@v3` with separate `run: gradle ...` steps
- Previously: added `gradle-version: '8.13'` so Gradle is downloaded directly instead of requiring missing gradlew wrapper script
- Created Java Spring Boot scaffolder template (13 files)
- Wired GitHub Actions plugin into EntityPage CI/CD tab
- Updated app-config.yaml with new template location
