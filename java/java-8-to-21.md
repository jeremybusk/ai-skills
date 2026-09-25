---
name: upgrade-java-spring-gradle
description: Assess, implement, or review migration of a Java 8 application using Gradle 6.3 to Java 21 and a compatible supported Spring Boot release. Covers Gradle checkpoints, Jakarta migration, common dependencies, tests, containers, and CI.
---

# Upgrade Java, Spring Boot, and Gradle

Modernize the application while preserving behavior, API contracts,
data integrity, and security controls.

## 1. Establish scope and starting versions

Expected starting point:
- Java: 8.
- Gradle wrapper: 6.3.
- Spring Boot and Spring Framework: inspect the repository.
- Target Java: 21.
- Target Gradle and Spring Boot: select compatible supported versions.

Verify these assumptions before making changes. If the repository differs,
report the actual versions and adapt the migration.

Match the requested mode:
- Assessment: inspect and recommend without modifying files.
- Implementation: apply changes and validate them.
- Review: examine existing changes and report actionable findings.

Do not expand a Java upgrade into an unrelated framework migration.
Upgrade Spring Boot when requested or required for compatibility.
Do not convert Maven to Gradle or plain Spring to Boot unless requested.

Read applicable AGENTS.md instructions and inspect the working tree.
Preserve user changes and keep unrelated refactoring out of the migration.

## 2. Inspect the application and establish a baseline

Identify:
- JDKs used by developers, Gradle, CI, containers, and production.
- Gradle wrapper, build scripts, plugins, and dependency repositories.
- Subprojects, buildSrc, included builds, and convention plugins.
- Spring Boot, Framework, Security, Data, and Cloud versions.
- BOMs, version constraints, lockfiles, and verification metadata.
- Internal libraries, shared modules, and custom Spring Boot starters.
- Packaging: executable JAR, WAR, external container, or native image.
- Databases, messaging, authentication, scheduled jobs, and integrations.
- Unit, integration, contract, and deployment tests.

Use the repository wrapper, not an unrelated system Gradle installation.

Useful commands, adapted to the project:
- java -version
- ./gradlew --version
- ./gradlew projects
- ./gradlew tasks --all
- ./gradlew help --warning-mode all
- ./gradlew :<module>:dependencies --configuration runtimeClasspath
- ./gradlew :<module>:dependencyInsight --dependency <name>
  --configuration runtimeClasspath

Use gradlew.bat on Windows.

Run the existing build and relevant tests with the original compatible JDK.
Record test counts and pre-existing failures.
If the baseline cannot run, explain why and continue independent inspection;
do not classify unverified failures as migration regressions.

Do not print secrets from configuration or environment variables.

## 3. Select target versions and support policy

Consult current official requirements and migration guides for:
- Spring Boot and its managed dependencies.
- Gradle runtime JDKs and Java toolchains.
- Spring Cloud release trains.
- Build plugins, application servers, and critical integrations.

Prefer releases receiving public security fixes unless the organization
has confirmed access to commercial maintenance releases.

Honor explicit version requirements, but identify compatibility or support
limitations. Do not equate Java compatibility with security maintenance.

Record:
- Current and target versions.
- Compatibility evidence and source links.
- Support status and the date checked.
- Reasons for required dependency overrides.

If current information cannot be verified, state the uncertainty.
Do not guess a latest version or support end date.

Pin selected versions. Do not introduce dynamic versions, snapshots,
milestones, release candidates, or previews by default.

Let Spring Boot manage Spring Framework and related library versions.
Align Spring Cloud separately with the selected Boot generation.
Do not independently upgrade managed dependencies without a specific reason.

## 4. Plan the migration from Gradle 6.3

Do not launch Gradle 6.3 using Java 17 or Java 21.
Start with the existing Java 8 environment.

Use this as a planning sequence, adapting it to plugin compatibility:

1. Gradle 6.3 with Java 8: establish the baseline.
2. A verified Gradle 6.9.x patch with Java 8: address 6.x deprecations.
3. A verified Gradle 7.6.x patch: resolve Gradle 7 breaking changes.
4. A compatible Gradle 8.x release: resolve Gradle 8 breaking changes.
5. Java 21 with a selected supported Gradle release and compatible Boot.
6. Gradle 9.x only if selected as the final target and supported by the
   application's plugins and framework.

These are migration checkpoints, not mandatory production releases.
Do not deploy unsupported intermediate versions as the final result.

For each checkpoint record:
- JVM running Gradle.
- JDK compiling and testing the application.
- Application runtime JDK.
- Gradle, Boot, Spring Cloud, and relevant plugin versions.

Gradle 7.6.x can be a bridge from Java 8 to Java 17, but verify application
and plugin compatibility before changing its runtime JDK.

Java 21 compilation through toolchains is supported from Gradle 8.4.
Running Gradle itself on Java 21 requires Gradle 8.5 or newer.
These are compatibility floors, not recommendations to choose old patches.
The selected Boot version may impose a higher Gradle minimum.

Do not assume every Boot 3.x release supports Java 21.

Interleave framework and Gradle upgrades where their compatibility ranges
require it. Do not upgrade all build tools independently to their newest
major versions.

Validate each meaningful checkpoint before proceeding. Resolve migration
regressions or document why validation is blocked.

## 5. Upgrade Gradle build logic

Read upgrade guides for every crossed major version.

Before moving beyond Gradle 6:
- Replace compile with implementation or api as appropriate.
- Use api only when dependencies form part of a library's public API,
  with the java-library plugin where needed.
- Replace runtime with runtimeOnly.
- Replace testCompile with testImplementation.
- Replace testRuntime with testRuntimeOnly.
- Inspect custom source sets and configurations extending removed names.
- Migrate legacy publishing configuration when the old maven plugin is used.
- Review obsolete repositories and find trusted sources for affected
  artifacts; do not replace repository definitions blindly.

Across subsequent versions:
- Fix removed task properties, APIs, and plugin incompatibilities.
- Review Groovy and Kotlin build-script compatibility.
- Update buildSrc, included builds, and convention plugins.
- Review archive tasks, publishing metadata, generated sources, and
  annotation-processor configuration.
- Preserve dependency visibility and publication behavior.

Upgrade through the wrapper task when possible.
After selecting a new distribution, regenerate wrapper files with that
version as needed so scripts, JAR, and properties are updated.

Verify the distribution checksum against the official distribution.
If the existing build cannot execute the wrapper task, use an official
verified bootstrap method and explain the workaround.

Review lockfile and dependency-verification changes.
Do not disable verification or accept unexpected dependency changes blindly.

Do not introduce configuration caching, a new build DSL, or other unrelated
build redesigns solely to complete the migration.

## 6. Plan Spring Boot checkpoints

Determine the starting Boot version before choosing a route.

When applicable:
- For Boot 1.x, follow the Boot 2 migration guidance first.
- Reach the appropriate latest Boot 2.7 patch before crossing to Boot 3.
- Reach the appropriate latest Boot 3.5 patch before crossing to Boot 4.
- Follow migration and release notes for intervening releases.

Use intermediate versions only to isolate migration work.
Choose a supported final destination compatible with Java 21.

Before crossing a major boundary:
- Resolve relevant deprecated API usage.
- Check internal starters and third-party integrations.
- Validate a compatible JDK and Gradle combination.
- Run focused tests for behavior affected by that boundary.

For Boot 2.7 to 3, consider the documented Spring Security 5.8 bridge
when it helps migrate an older security configuration.

Use spring-boot-properties-migrator when applicable.
Remove it after updating configuration properties.

Inspect custom auto-configuration registration:
- Migrate affected EnableAutoConfiguration registrations from
  spring.factories to AutoConfiguration.imports for Boot 3.
- Preserve unrelated spring.factories entries.

For Boot 4, additionally review:
- Starter and test-module changes.
- Jackson migration.
- Servlet-container requirements.
- Removed deprecated APIs.
- Changes affecting custom extensions.

Do not assume a successful version change in the build file completes
the framework migration.

## 7. Configure Java 21 and resolve JDK incompatibilities

Once the selected Gradle and framework support it:
- Configure Java 21 toolchains.
- Set compilation release settings where appropriate.
- Align Java, Kotlin, and Groovy bytecode targets.
- Ensure tests actually execute on Java 21.
- Preserve parameter-name metadata required by Spring.
- Align CI, IDE configuration, Docker build stages, and runtime images.
- Remove obsolete assumptions about JAVA_HOME and JRE directory layout.

Check for:
- Reflective access failures such as InaccessibleObjectException.
- Bytecode tools rejecting Java 21 class files.
- Removed bundled JAXB, JAX-WS, Activation, or CORBA APIs.
- Nashorn usage and other removed components.
- References to rt.jar, tools.jar, or extension directories.
- URLClassLoader casts and custom class-loading assumptions.
- Removed JVM flags, CMS options, and obsolete GC logging.
- Platform-default encoding and locale-sensitive behavior.
- TLS, certificates, keystores, and cryptographic compatibility.
- JNI libraries and architecture-specific native dependencies.
- Monitoring or coverage agents incompatible with Java 21.

Use jdeps and jdeprscan where helpful, with the relevant application
classes and dependency classpath. Their results do not replace runtime tests.

Prefer supported APIs and updated dependencies.
Treat --add-opens and --add-exports as narrowly scoped temporary workarounds.
Record the dependency and reason for every remaining workaround.

Do not introduce modules, preview features, virtual threads, or widespread
language-feature rewrites solely to complete this migration.

## 8. Migrate Java EE APIs to Jakarta where required

The framework migration determines which Jakarta changes are necessary.
Changing the JDK alone does not require replacing every javax package.

Common migrations:
- javax.persistence -> jakarta.persistence
- javax.servlet -> jakarta.servlet
- javax.validation -> jakarta.validation
- javax.annotation -> jakarta.annotation for migrated Java EE annotations
- javax.transaction -> jakarta.transaction for migrated Java EE APIs
- javax.jms -> jakarta.jms
- javax.mail -> jakarta.mail
- javax.xml.bind -> jakarta.xml.bind
- javax.xml.ws -> jakarta.xml.ws
- javax.ws.rs -> jakarta.ws.rs when JAX-RS libraries are used
- javax.inject -> jakarta.inject when required by the selected framework

Never perform a blanket javax-to-jakarta replacement.

Java SE packages remain, including:
- javax.sql
- javax.crypto
- javax.net
- javax.naming
- javax.management
- javax.xml.parsers
- javax.xml.transform
- javax.transaction.xa

Update:
- Imports and dependency coordinates.
- API implementations and runtime providers.
- Generated sources and code-generation plugins.
- XML descriptors and relevant schemas.
- Service-provider files and reflective class-name strings.
- Tests, fixtures, internal libraries, and shared contracts.

Regenerate generated code from its source definitions where possible.
Inspect transitive dependencies for incompatible legacy APIs.
Adding both javax and Jakarta artifacts does not make their types compatible.

## 9. Review common application dependencies

Review only components present in the application.

| Area | Components and checks |
|---|---|
| Persistence | Hibernate, Spring Data, JDBC drivers, HikariCP; mappings, queries, sequences, dialects, transactions |
| Schema migration | Flyway, Liquibase; database compatibility, required modules, migration checksums |
| Security | Spring Security, OAuth2/OIDC, JWT, SAML; filter chains, matchers, roles, sessions, CSRF, CORS |
| Serialization | Jackson, Gson, JSON-B; dates, enums, nulls, custom serializers, wire compatibility |
| HTTP | Apache HttpClient, OkHttp, Reactor Netty; TLS, proxies, pools, timeouts |
| Messaging | Kafka, RabbitMQ, JMS; clients, converters, retries, transactions, message formats |
| Logging | SLF4J, Logback, Log4j2; bindings/providers, bridges, configuration |
| Observability | Micrometer, tracing, OpenTelemetry, APM agents; instrumentation and propagation |
| Code generation | Lombok, MapStruct, QueryDSL, JAXB, OpenAPI generators; processors and generated APIs |
| Testing | JUnit, Mockito, Byte Buddy, JaCoCo, AssertJ, Testcontainers, WireMock; test discovery and compatibility |
| API documentation | Springfox, springdoc, Swagger/OpenAPI; framework compatibility and endpoint behavior |
| Caching and jobs | Redis clients, cache providers, Quartz, Spring Batch; serialization, schedules, metadata schemas |
| Other integrations | Mail, SOAP/JAX-WS, JAX-RS, cloud SDKs, internal clients |

Prefer Boot-managed versions where provided.
Document replacements for abandoned or incompatible libraries.
Do not upgrade every dependency to its newest major release automatically.

## 10. Validate application behavior

Run repository checks and focused regression tests for migration risks.

Verify:
- Startup with relevant profiles.
- Authentication, authorization, denied requests, sessions, and logout.
- REST routes, trailing slashes, errors, JSON, and XML contracts.
- Database reads, writes, queries, transactions, and rollback.
- Schema migration against representative disposable databases.
- Messaging, retries, transactions, and serialization.
- Scheduled jobs, batch processing, file imports, and exports.
- External HTTPS, database, mail, and identity-provider connections.
- Logs, metrics, traces, and monitoring agents.
- Packaged JAR/WAR or container startup on the intended Java 21 runtime.
- Health checks, graceful shutdown, resource limits, and memory behavior.

Confirm expected tests actually executed.
Investigate zero-test runs or unexpected reductions in test counts.
Run integration-test tasks that are not included in the default lifecycle.

Do not weaken assertions, disable security, or skip failing tests merely
to obtain a passing build.

Use disposable or explicitly authorized test infrastructure.
Never use production database migrations as a validation step.

Run available dependency and security checks against resolved dependencies.
Report findings accurately without inventing vulnerabilities.
Compare representative performance with the baseline when operational
requirements or observed changes justify it.

## 11. Completion and delivery

The migration is complete only when:
- The selected versions are compatible and their support status is recorded.
- Required build and test checks pass.
- The packaged application starts on the target runtime.
- Relevant integration behavior is validated.
- CI and deployment configuration use the intended JDK and build versions.
- Temporary migration dependencies are removed.
- Remaining overrides and workarounds are documented.

If required validation cannot run, report the migration as incompletely
validated and identify the missing infrastructure, access, or other blocker.

Deliver:
- Current-to-final version table.
- Main code, dependency, configuration, and deployment changes.
- Commands/checks executed and outcomes.
- Baseline failures versus introduced regressions.
- Outstanding risks and validation gaps.
- Deployment and rollback considerations, including database and message
  compatibility.

Do not claim success based only on compilation.
Do not publish artifacts or deploy solely because the local migration passed.

## Official references

Use version-specific documentation for each migration checkpoint.

- Java 21 migration:
  https://docs.oracle.com/en/java/javase/21/migrate/
- Gradle Java compatibility:
  https://docs.gradle.org/current/userguide/compatibility.html
- Gradle wrapper:
  https://docs.gradle.org/current/userguide/gradle_wrapper.html
- Gradle 6 to 7:
  https://docs.gradle.org/current/userguide/upgrading_version_6.html
- Gradle 7 to 8:
  https://docs.gradle.org/current/userguide/upgrading_version_7.html
- Gradle 8 upgrade guidance:
  https://docs.gradle.org/current/userguide/upgrading_version_8.html
- Gradle 9 upgrade guidance:
  https://docs.gradle.org/current/userguide/upgrading_version_9.html
- Spring Boot requirements:
  https://docs.spring.io/spring-boot/system-requirements.html
- Spring Boot migration guides and release notes:
  https://github.com/spring-projects/spring-boot/wiki
- Spring Cloud:
  https://spring.io/projects/spring-cloud
