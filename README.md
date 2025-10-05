# Agentic Chunker API Documentation

![Python SDK](https://img.shields.io/badge/Python%20SDK-Coming%20Soon-blue)

Welcome to the official API documentation for **Agentic Chunker**.

Agentic Chunker is a powerful API designed to intelligently process and structure raw text content, specifically for Retrieval-Augmented Generation (RAG) applications. By leveraging LLM inference, our service transforms documents into hierarchy-aware, contextually rich chunks, significantly improving the quality and precision of your RAG model's retrieval.

## Table of Contents

- [Getting Started](#getting-started)
  - [1. Authentication](#1-authentication)
  - [2. Bring Your Own Key (BYOK)](#2-bring-your-own-key-byok)
- [API Endpoints](#api-endpoints)
  - [POST `/chunk/hierarchy`](#post-chunkhierarchy)
  - [POST `/chunk/sentence`](#post-chunksentence)
- [Supported Models](#supported-models)
- [Error Handling](#error-handling)
- [SDKs](#sdks)

## Getting Started

To begin using the Agentic Chunker API, you'll need an API key and a valid LLM Provider key (BYOK).

### 1. Authentication

All API requests must be authenticated using an `X-API-Key` header.

1.  **Generate your API Key** from the dashboard: [https://hierarchychunker.codeaxion.com/app/dashboard](https://hierarchychunker.codeaxion.com/app/dashboard)
2.  Include the key in the `X-API-Key` header of your requests.

```
X-API-Key: <YOUR_API_KEY>
Content-Type: application/json
```

### 2. Bring Your Own Key (BYOK)

Agentic Chunker operates on a Bring Your Own Key (BYOK) model. To use the service, you must provide your own API keys for the LLM providers (e.g., OpenAI, Anthropic) you wish to use. This gives you full control over your models and billing.
The API will not function without a valid provider key configured in your account.

1. Navigate to the **Integrations** page in your dashboard: [https://hierarchychunker.codeaxion.com/app/integrations](https://hierarchychunker.codeaxion.com/app/integrations)
2. Add and save the API keys for the LLM providers you intend to use.

Once configured, our API will use your keys for all LLM inference tasks associated with your account.

---

## API Endpoints

The base URL for all API endpoints is: `https://api.agenticchunker.codeaxion.com`

### `POST /chunk/hierarchy`

This endpoint uses LLM inference to understand and chunk a document based on its semantic and structural hierarchy. It ensures context flows naturally and retrievers know exactly where each piece of content belongs.

**Key Features:**

*   **Hierarchy-Aware Chunking**: Merges parent headers with their sub-sections, maintaining logical context.
*   **Nested Structure Preservation**: Retains depth and numbering (e.g., 1 → 1.1 → 1.2), keeping subheadings within the correct chunk.
*   **Multi-Level Support**: Preserves context integrity across all levels (Title → Subtitle → Section → Subsection).
*   **Cross-Page Context Continuity**: Links chunks across page breaks to maintain a seamless flow of information.

#### Request Body

| Parameter        | Type     | Required | Description                                                                                                                              |
| ---------------- | -------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| `content`        | `string` | Yes      | The raw text or Markdown content to be chunked.                                                                                          |
| `model`          | `string` | Yes      | The model to use for inference, formatted as `[provider]/[model-name]`. Example: `"openai/gpt-4.1-mini"`.                                  |
| `options`        | `object` | No       | An object containing chunking configuration options.                                                                                     |
| `previous_chunk` | `object` | No       | An object representing the last chunk of the previous page, used to enable cross-page context continuity. See details below.             |

#### `options` Object

| Parameter                  | Type      | Default | Description                                                                                   |
| -------------------------- | --------- | ------- | --------------------------------------------------------------------------------------------- |
| `min_words`                | `integer` | `50`    | A soft guidance for the minimum number of words per chunk.                                    |
| `max_words`                | `integer` | `250`   | A soft guidance for the maximum number of words per chunk.                                    |
| `max_chars`                | `integer` | `1000`  | Maximum characters per chunk before attempting a semantic split.                              |
| `enable_semantic_chunking` | `boolean` | `false` | If `true`, breaks large chunks into smaller, semantically coherent parts.                     |
| `enable_recursion`         | `boolean` | `false` | If `true`, re-chunks any oversized semantic chunk again. Requires `enable_semantic_chunking`. |
| `split_tables`             | `boolean` | `true`  | If `true`, tables are extracted and stored as separate chunks.                                |

#### Cross-Page Context Continuity

To maintain context across page breaks in a large document, pass the last chunk from the previous page in the `previous_chunk` field when processing the next page. The system will detect if the first chunk of the new page is a continuation and link them.

**`previous_chunk` Object:**

```json
{
  "previous_chunk": {
    "chunk_id": 24,
    "page_content": "The last sentence of the previous page..."
  }
}
```

#### Example Request (Python)

```python
import requests
import json

api_url = "https://api.agenticchunker.codeaxion.com/chunk/hierarchy"
api_key = "<YOUR_API_KEY>"

content = """
Magistrates' Courts (Licensing) Rules (Northern Ireland) 1997
SR (NI) 1997/62
Reproduced with permission from the LEXISNEXIS website “Valentine’s All the Laws
of Northern Ireland”
[am. SR (NI) 2004/451, SR (NI) 2016/405]
[Department of Justice]
The Lord Chancellor, in exercise of the powers conferred upon him by Article 13 of
the Magistrates' Courts (NI) Order 1981, and of all other powers enabling him on that
behalf, on the advice of the Magistrates' Courts Rules Committee and after
consultation wit
"""

payload = {
    "content": content,
    "model": "openai/gpt-4.1-mini",
    "options": {
        "min_words": 50,
        "max_words": 250,
        "max_chars": 1000,
        "enable_semantic_chunking": False
    }
}

headers = {
    "Content-Type": "application/json",
    "X-API-Key": api_key
}

response = requests.post(api_url, json=payload, headers=headers)

print(json.dumps(response.json(), indent=2))
```

#### Example Success Response (`200 OK`)

The API returns an array of chunk objects. Each object contains the chunked `page_content` and a `metadata` object detailing its position in the document's hierarchy.

```json
[
  {
    "metadata": {
      "Title": "Magistrates' Courts (Licensing) Rules (Northern Ireland) 1997",
      "Subtitle": "SR (NI) 1997/62"
    },
    "page_content": "Magistrates' Courts (Licensing) Rules (Northern Ireland) 1997\nSR (NI) 1997/62\nReproduced with permission..."
  },
  {
    "metadata": {
      "Title": "Magistrates' Courts (Licensing) Rules (Northern Ireland) 1997",
      "Section Header (1)": "PART I",
      "Section Header (1.1)": "Citation and commencement"
    },
    "page_content": "PART I\nCitation and commencement\n1. These Rules may be cited as the Magistrates' Courts (Licensing) Rules (Northern\nIreland) 1997..."
  },
  {
    "metadata": {
      "Title": "Magistrates' Courts (Licensing) Rules (Northern Ireland) 1997",
      "Section Header (2)": "PART II",
      "Section Header (2.1)": "RENEWAL OF LICENCES",
      "Section Header (2.1.1)": "Applications for the renewal of licences"
    },
    "page_content": "PART II\nRENEWAL OF LICENCES\nApplications for the renewal of licences\n4. - (1) Notice of application for the renewal of a licence..."
  }
]
```

If context continuity is detected, the first chunk of the response will include a `continued_from_chunk` key:

```json
[
    {
        "continued_from_chunk": "chunk_9",
        "metadata": { "...": "..." },
        "page_content": "…"
    }
]
```

---

### `POST /chunk/sentence`

This endpoint provides a fast and effective way to split text into semantically coherent chunks based on sentence boundaries. It's ideal for less structured text or when a full hierarchical breakdown is not required.

#### Request Body

The request body is similar to the `/hierarchy` endpoint but typically uses fewer options.

| Parameter | Type     | Required | Description                                                                                      |
| --------- | -------- | -------- | ------------------------------------------------------------------------------------------------ |
| `content` | `string` | Yes      | The raw text content to be chunked.                                                              |
| `model`   | `string` | No       | An optional model to guide the chunking. Semantic splitting works well even without a model.     |
| `options` | `object` | No       | An object containing chunking configuration options (`min_words`, `max_words`, `max_chars`, etc.). |

#### Example Request (cURL)

```bash
import requests
import json

url = "https://api.agenticchunker.codeaxion.com/chunk/sentence"

headers = {
    "Content-Type": "application/json",
    "X-API-Key": "<YOUR_API_KEY>"
}

payload = {
    "content": "in Form 1 served on the clerk of petty sessions... (rest of content)",
    "options": {
        "min_words": 50,
        "max_words": 250,
        "max_chars": 1000
    }
}

response = requests.post(url, headers=headers, json=payload)

# Pretty print JSON response
print(json.dumps(response.json(), indent=2))

```

#### Example Success Response (`200 OK`)

The API returns a simple array of chunk objects. The `metadata` field is typically empty for this endpoint.

```json
[
  {
    "metadata": {},
    "page_content": "in Form 1 served on the clerk of petty sessions, and to the copies required to be served by paragraph 3 of Schedule 4, a notice in Form 15, 16, 17 or 20, as may be appropriate..."
  },
  {
    "metadata": {},
    "page_content": "PART III\nDOCUMENTS\nDocuments to be lodged with applications or produced to the court\n5. - (1) Where application is made for the renewal of a licence for an hotel or a guest-house a current certificate, issued by the Northern Ireland Tourist Board..."
  }
]
```

---

## Supported Models

You can specify any of the following models in the `model` field of your request payload, formatted as `[provider]/[model-name]`.

<details>
<summary><strong>Click to view all supported models</strong></summary>

#### OpenAI
- `openai/gpt-5`
- `openai/gpt-5-mini`
- `openai/gpt-5-nano`
- `openai/gpt-4.1`
- `openai/gpt-4.1-mini`
- `openai/gpt-4.1-nano`
- `openai/gpt-4o`
- `openai/gpt-4o-mini`

#### Anthropic
- `anthropic/claude-sonnet-4-20250514`
- `anthropic/claude-3-7-sonnet-latest`
- `anthropic/claude-3-7-sonnet-20250219`
- `anthropic/claude-3-5-haiku-latest`
- `anthropic/claude-3-5-haiku-20241022`
- `anthropic/claude-3-haiku-20240307`

#### Google AI
- `google/gemini-2.5-flash`
- `google/gemini-2.5-flash-lite`
- `google/gemini-2.0-flash`
- `google/gemini-2.0-flash-lite`

#### Groq
- `groq/llama-3.3-70b-versatile`
- `groq/openai/gpt-oss-120b`
- `groq/openai/gpt-oss-20b`
- `groq/meta-llama/llama-4-scout-17b-16e-instruct`
- `groq/meta-llama/llama-4-maverick-17b-128e-instruct`
- `groq/deepseek-r1-distill-llama-70b`
- `groq/moonshotai/kimi-k2-instruct`

#### Mistral
- `mistral/mistral-medium-latest`
- `mistral/magistral-medium-latest`
- `mistral/mistral-small-latest`
- `mistral/mistral-large-latest`
- `mistral/open-mistral-7b`
- `mistral/open-mixtral-8x7b`
- `mistral/open-mixtral-8x22b`
- `mistral/ministral-8b-latest`
- `mistral/ministral-3b-latest`

</details>

---

## Error Handling

The API uses standard HTTP status codes to indicate the success or failure of a request. In case of an error, the response body will contain a JSON object with an `error` code and a descriptive `message`.

| Status Code | Error Code                   | Description                                                                                          |
| ----------- | ---------------------------- | ---------------------------------------------------------------------------------------------------- |
| `400`       | `MISSING_PARAMETER`          | A required parameter (e.g., `content`) was not included in the request.                              |
| `400`       | `UNKNOWN_PROVIDER`           | The provider specified in the `model` string is not recognized.                                      |
| `400`       | `LLM_INITIALIZATION_FAILED`  | The selected LLM client could not be initialized, possibly due to a misconfiguration.                |
| `401`       | `INVALID_API_KEY`            | The `X-API-Key` provided is invalid or does not exist.                                               |
| `401`       | `AUTHENTICATION_ERROR`       | Authentication with the backend LLM provider failed (check your BYOK settings).                      |
| `401`       | `PROVIDER_API_KEY_NOT_FOUND` | BYOK is configured, but the API key for the requested provider is missing from your integrations.    |
| `402`       | `TOKEN_LIMIT_EXCEEDED`       | Your account has exceeded its usage limits. Please upgrade your plan to continue.                    |
| `403`       | `INACTIVE_API_KEY`           | The provided `X-API-Key` has been deactivated.                                                       |
| `404`       | `MODEL_NOT_FOUND`            | The requested LLM model was not found or is not supported.                                           |
| `500`       | `API_KEY_PROCESS_FAILED`     | A server-side error occurred while processing a stored provider API key.                             |
| `500`       | `INTERNAL_SERVER_ERROR`      | An unexpected server error occurred. Please try your request again later.                            |

#### Example Error Response
All error responses follow this structure.

```json
{
  "error": "INVALID_API_KEY",
  "message": "The provided API key is invalid or does not exist."
}
```
---

## SDKs

We are actively developing a **Python SDK** to make integration even easier and more seamless. Stay tuned for its release
