# QuizCraft

QuizCraft is a retrieval-based quiz prototype. Given a Word document (`.docx`) and a topic, it finds relevant passages and uses them as context for a question-generation model. The interface offers multiple-choice, true/false, short-answer and fill-in-the-blank formats.

## How retrieval works

1. The FastAPI handler reads the document paragraphs with python-docx and groups the text into chunks.
2. The service embeds each chunk with `all-MiniLM-L6-v2`, stores it in a FAISS index and searches for passages close to the topic.
3. If the initial search misses its distance threshold, the fallback checks its local topic cache and can search Wikipedia, Britannica or the Stanford Encyclopedia of Philosophy. It adds the retrieved material to the index and searches again.
4. The service passes the retrieved context, topic, question format and count to Microsoft's `Phi-3.5-mini-instruct`.

## Running the components

| Component | Entry point |
| --- | --- |
| Web client | From `Mern/frontend/`, run `npm ci` and `npm start`. |
| Account service | From `Mern/backend/`, configure `MONGO_URI` and `JWT_SECRET` for development, run `npm ci`, then `node server.js`. The default port is 5000. |
| Generation service | From `Flask/`, serve `app:app` with `uvicorn app:app --port 8000`. Despite the directory name, this service uses FastAPI. |
| Mobile client | `NativeApp/` contains the Expo client and its quiz and results screens. |

The Python service needs a CUDA-capable environment for its configured model placement, model downloads, and dependencies for FastAPI multipart uploads, Transformers, PyTorch, Accelerate, Sentence Transformers, FAISS, python-docx, Scrapy and Google search. The repository does not include a pinned Python environment.

## Current implementation limits

- **Inputs.** Supply both the topic and a Word document. The API requires both even though the web form labels the topic optional.
- **Output length.** The model configuration sets `max_new_tokens` to 1. Increase it in `Flask/app.py` to generate a complete question set.
- **Document scope.** The FAISS index persists across requests within one process. Use a single local session and sample documents when exploring retrieval.
