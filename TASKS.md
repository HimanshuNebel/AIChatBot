# Pending Tasks — RAG API Project

## Task 1: Restore exact-match author/title question detection (Regression)
**Priority:** High · **File:** `query.py`

The rewrite in commit `37556b3` removed `is_author_question()` / `is_title_question()` (added in `cd14ce1`) and reverted to keyword sniffing. Bug reintroduced: any question containing "name" (e.g. "what is the dataset name?") returns the document title instead of running RAG.

**To do:**
- Restore `normalize()`, `AUTHOR_QUERIES`, `TITLE_QUERIES`, `is_author_question()`, `is_title_question()` from commit `cd14ce1`
- Replace the `any(word in question_lower ...)` checks with these functions

**Acceptance criteria:**
- "what is the dataset name?" goes to RAG, not the title shortcut
- "who are the authors" still returns authors from the document record

---

## Task 2: Add API key protection to unprotected document endpoints
**Priority:** High · **File:** `app.py`

Only `/document/{doc_id}` requires `X-API-Key`. `POST /uploadDocument` and `GET /getAllUploadedDocuments` are open — anyone can upload files (disk + Groq quota abuse) or list all documents.

**To do:**
- Add `_: None = Depends(require_api_key)` to `upload_document` and `get_all_uploaded_documents`

**Acceptance criteria:**
- Both endpoints return 401 without a valid `X-API-Key` header

---

## Task 3: Clean up vestigial `sessions.last_active_at` handling
**Priority:** Low · **File:** `app.py`

Session timeout/reuse logic was removed from `initiateChat`, but `last_active_at` is still updated on every `sendChat`. Dead writes with no reader.

**To do (pick one):**
- Remove the column and the UPDATE in `sendChat`, **or**
- Keep it and add a comment documenting it as audit/analytics data

---

## Task 4: Update `test_all.py` for the new `handle_query` contract
**Priority:** Medium · **File:** `test_all.py`

`handle_query` now returns `{ok, error}` dicts instead of `None` on failure. The test's `if outcome is None` check never fires, so failures crash with `KeyError: 'conversation_id'`. `DOC_ID` is also hardcoded to `f62eee6c88a0`.

**To do:**
- Check `outcome["ok"]` instead of `outcome is None`; print `outcome["error"]` on failure
- Take `DOC_ID` from a CLI argument or environment variable

**Acceptance criteria:**
- Running against a missing doc_id prints a clean error, no traceback
- Test runs on any machine given a valid doc_id

---

## Task 5: Add `app.py` and `token_utils.py` to syntax check
**Priority:** Low · **File:** `check_syntax.py`

**To do:**
- Add `"app.py"` and `"token_utils.py"` to `FILES_TO_CHECK`

---

## Task 6: Make the project integration-ready
**Priority:** High · **Files:** new

Missing pieces for dropping this into other projects:

**To do:**
- `requirements.txt` — fastapi, uvicorn, pyjwt, python-multipart, python-dotenv, groq, chromadb, sentence-transformers, pymupdf, pdfplumber, pytesseract, Pillow, tavily-python
- `README.md` — setup steps, required env vars (`GROQ_API_KEY`, `jwt_secret_key`, `DOCUMENTS_API_KEY`, `TAVILY_API_KEY`), endpoint docs (method, path, auth, request/response examples for all 6 APIs), auth flow diagram (API key → chat token → refresh)
- CORS — add `CORSMiddleware` with configurable allowed origins (env var, not `*`)

**Acceptance criteria:**
- Fresh clone + `pip install -r requirements.txt` + `.env` → API starts and all endpoints work
- A browser-based client on an allowed origin can call the API
