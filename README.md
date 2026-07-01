DFA Workshop — Full-Stack Refactor
This project splits the original standalone workshop into a clean, decoupled full-stack architecture.

backend/ — Spring Boot REST API (Java 17, Spring Data JPA, H2 in-memory database).

frontend/ — Lean static client (HTML/CSS/JS) decoupled from the backend.

Backend Architecture & Layering
The backend is built with strict separation of concerns to ensure testability and maintainability. Controllers never interact with the database layers directly.

model/: WorkshopSection and CodeSnippet JPA entities maintaining a @OneToMany / @ManyToOne relationship.

dto/: WorkshopSectionDTO, CodeSnippetDTO, and a WorkshopMapper. This ensures the API layer never directly exposes raw database entities.

repository/: WorkshopSectionRepository and CodeSnippetRepository extending JpaRepository.

service/: Core business logic encapsulated in a WorkshopService interface and WorkshopServiceImpl implementation.

controller/: A thin @RestController layer exposing clean REST endpoints.

exception/: Centralized error handling via @ControllerAdvice (GlobalExceptionHandler). Unhandled exceptions or missing resources map to structured JSON payloads (ApiError) rather than leaking raw stack traces to the client.

resources/: Includes application.properties configuration and a data.sql script to automatically seed the H2 database on startup with the live workshop content (Setup & Libraries, Python Basics, Your First Chart, Real Data, Final Visualization).

REST Endpoints
GET /api/v1/sections — Fetches all workshop sections.

GET /api/v1/sections/{id} — Fetches a specific section by ID.

GET /api/v1/tracks/{trackId} — Fetches sections filtered by track.

Frontend Refactor
The frontend was refactored away from a monolithic single-file structure into modern, maintainable assets:

Separation of Concerns: Extracted all inline styles into styles.css and all execution logic into app.js, leaving index.html as a clean, semantic markup shell.

Dead Code Elimination: Cleaned up a redundant, unused navigation script block (goHome/goAbout) that was sitting legacy in the original file, ensuring only the active UI wiring (window.goHome/window.goTo) remains.

Dynamic Data Fetching: app.js is configured to hit GET /api/v1/sections on load to dynamically populate section labels from the Spring Boot API. It includes a graceful local fallback so the static page remains functional even if the backend is offline.

Local Setup & Running
Note: Because this requires pulling down dependencies from Maven Central, ensure you have an active network connection for the initial build.

1. Spin up the Backend
Bash
cd backend
mvn spring-boot:run
2. Verify the Services
API Root: http://localhost:8080/api/v1/sections

Error Handling Check: http://localhost:8080/api/v1/sections/99 (Should return a clean 404 JSON payload, not a white-label stack trace).

H2 Console: http://localhost:8080/h2-console (JDBC URL: jdbc:h2:mem:workshopdb)

Future Scope & Production Considerations
To keep the scope of this refactor focused on core architecture, a few production-level features were intentionally left for the next iteration:

Media Assets & Transcripts: Narrator subtitle transcripts (SUBS) and audio assets should ideally be moved to an external blob storage solution (or served as static assets) mapped to a relational Transcript entity, rather than being bundled in the frontend.

Security: For a public-facing, read-only content API, authentication was omitted to keep things lightweight. In a production environment, Spring Security would be integrated to lock down write/delete endpoints.