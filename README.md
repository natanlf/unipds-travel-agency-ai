# Mundo Viagens AI - Travel Agency Assistant

This project is an AI-powered travel agency assistant built with **Quarkus**, **LangChain4j**, and **Ollama**. It uses **RAG (Retrieval-Augmented Generation)** to provide accurate information about travel packages and **MCP (Model Context Protocol)** to perform actions like managing bookings through a specialized service.

---

## 🏗️ Architecture

The project is divided into two main modules:

1.  **`travel-agency-ai`**: The core AI service. It uses LangChain4j to integrate with the Ollama LLM, manages conversation memory, performs RAG using a PgVector database, connects to external tools via MCP, and uses **Guardrails** for input security.
2.  **`mcp-booking-server`**: A dedicated MCP server that exposes tools for booking management (querying details, canceling reservations, listing by category).

### 🛠️ Key Technologies

*   **[Quarkus](https://quarkus.io/)**: The Java framework powering both services.
*   **[LangChain4j](https://github.com/langchain4j/langchain4j)**: Integration layer for Large Language Models.
*   **[Ollama](https://ollama.com/)**: Local LLM runner (defaults to `gpt-oss:20b`).
*   **[MCP (Model Context Protocol)](https://modelcontextprotocol.io/)**: Used for seamless tool invocation between the AI agent and the booking server.
*   **[PgVector](https://github.com/pgvector/pgvector)**: Vector database for RAG (Retrieval-Augmented Generation).
*   **[Guardrails](https://docs.langchain4j.dev/tutorials/ai-services/#guardrails)**: Used for input/output validation and security (e.g., preventing prompt injection).
*   **Embedding Model**: `nomic-embed-text` (via Ollama).

---

## 🚀 Getting Started

### Prerequisites

*   **Java 21+**
*   **Maven 3.9+**
*   **Ollama** installed and running.
    *   Pull required models:
        ```bash
        ollama pull gpt-oss:20b
        ollama pull nomic-embed-text
        ```
*   **PostgreSQL with PgVector** extension enabled (or use the provided Docker Compose if available, otherwise ensure connection properties in `application.properties` are correct).

### Running the Project

#### 1. Start the MCP Booking Server
Navigate to the server directory and run:
```bash
cd mcp-booking-server
./mvnw quarkus:dev -Dquarkus.http.port=8081
```
The MCP server will be available at `http://localhost:8081/mcp/sse/`.

#### 2. Start the AI Assistant
Navigate to the AI assistant directory and run:
```bash
cd travel-agency-ai
./mvnw quarkus:dev
```
The assistant will be available at `http://localhost:8080/travel`.

---

## 💬 API Usage

The main entry point for the assistant is a POST request to `/travel`. It requires a user name header for session/memory management.

### Example Request
```bash
curl -X POST http://localhost:8080/travel \
     -H "Content-Type: text/plain" \
     -H "X-User-Name: Natan" \
     -d "Quais são os detalhes do pacote Amazônia?"
```

### Available Tools (via MCP)
*   `getBookingDetails(bookingId)`: Retrieve complete details of a specific booking.
*   `cancelBooking(bookingId, name)`: Cancel an existing booking.
*   `listPackagesByCategory(category)`: List available travel packages by category (e.g., `ADVENTURE`, `TREASURES`).

---

## 🛡️ Security & Guardrails

To ensure safe interactions, the project implements **Input Guardrails** to prevent prompt injection and malicious use:

*   **`InjectionGuard`**: An implementation of `InputGuardrail` that intercepts user messages before they reach the LLM.
*   **`PromptSecurityExpert`**: A specialized AI service that analyzes incoming prompts for potential attacks or instruction overrides.

If a malicious prompt is detected, the system automatically blocks the request with a security warning.

---

## 📂 Project Structure

*   `travel-agency-ai/src/main/resources/rag/`: Contains documents (`pacotes-viagem.md`) used for RAG to provide info on travel packages.
*   `travel-agency-ai/src/main/java/com/natancode/ai/`: AI logic, configuration, and REST endpoints.
*   `mcp-booking-server/src/main/java/com/natancode/travel/`: Booking service logic and MCP tool definitions.

---

## 📝 Configuration

Key settings can be found in `travel-agency-ai/src/main/resources/application.properties`:
*   `quarkus.langchain4j.ollama.base-url`: Ollama API endpoint.
*   `quarkus.langchain4j.pgvector.*`: Vector database settings for RAG.
*   `quarkus.langchain4j.mcp.booking-server.*`: Connection details for the MCP server.
