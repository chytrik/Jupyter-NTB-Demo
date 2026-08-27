# RAG Demo — Retrieval-Augmented Generation

## Co notebook ukazuje

Porovnání odpovědí LLM **bez RAG** (model halucinuje) a **s RAG** (model odpovídá z dokumentů).
Notebook pracuje s fiktivní firmou **Nexus Technologies** a jejím interním knowledge base —
všechna fakta jsou vymyšlená, takže model je nemůže znát z tréninku.

## Prerekvizity

- **Qwen3-8B** nasazený na clusteru přes KServe (LLMInferenceService)
- Endpoint modelu v proměnné prostředí `MODEL_URL`, např.:
  `export MODEL_URL="https://<kserve-service>.<namespace>.svc.cluster.local:8000/v1"`
- Python balíčky: `openai`, `scikit-learn`, `httpx` (notebook je nainstaluje)

## Syntetická data (`data/`)

| Soubor | Obsah | Příklad otázky |
|--------|-------|----------------|
| `security-policy.txt` | Hesla, klasifikace dat, incident response, VPN, acceptable use | Jaká je minimální délka hesla? (→ 16 znaků) |
| `product-catalog.txt` | 4 enterprise produkty s cenami a specifikacemi | Kolik stojí Nexus Vault HSM? (→ $28,500, FIPS 140-3) |
| `hr-handbook.txt` | PTO, rodičovská dovolená, learning budget, expenses, WFH | Jaký je learning budget pro AI certifikace? (→ EUR 4,500) |
| `architecture-decisions.txt` | 3 ADR — event-driven (Kafka), CockroachDB migrace, interní LLM | Proč migrace na CockroachDB? (→ Black Friday incident) |

## Průběh notebooku

1. **Setup** — připojení k Qwen3-8B přes OpenAI-kompatibilní API
2. **Load & chunk** — načtení 4 dokumentů, rozdělení na chunky (~500 znaků)
3. **Index** — TF-IDF vektorový index (lightweight, bez GPU)
4. **Chat funkce** — `ask_without_rag()` vs `ask_with_rag()` s retrieval kontextem
5. **Porovnání** — 4 připravené otázky side-by-side (bez RAG vs s RAG)
6. **Interaktivní dotaz** — vlastní otázka

## Klíčový takeaway

Bez RAG model tipuje a vymýšlí (plausible ale špatné odpovědi). S RAG odpovídá přesně
z dokumentů a cituje zdroj. V produkci se TF-IDF nahradí dense embeddingsy a vector DB.

## Technické poznámky

- Retrieval: TF-IDF + cosine similarity (scikit-learn)
- KServe endpoint používá self-signed TLS → `httpx.Client(verify=False)`
- Qwen3 má thinking mode → vypnutý přes `extra_body={"chat_template_kwargs": {"enable_thinking": False}}`
