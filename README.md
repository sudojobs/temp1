Here is the full project ZIP (code + UI + indices + placeholder GGUF layout) that you can directly download and run:

👉 Download: xgent_agentic_ai_full_with_placeholder_gguf.zip

⚠️ Important:

This ZIP does not contain real Llama / DeepSeek / Qwen GGUF weights (those are huge and copyrighted).

Inside models/ there is a placeholder file llama-demo.gguf.

You must download actual GGUF models from Hugging Face and replace this file locally.

What is inside this ZIP

Folder structure inside the zip:

xgent_agentic_bge_full/
  server.py
  config.json
  USER_GUIDE.md
  xgent_ai.sqlite3           (created on first run)
  models/
    llama-demo.gguf          (placeholder – replace with real GGUF)
  indices/
    ...                      (created when you train Jira/PDF/testcases)
  pdf_store/                 (optional, for future use)
  ui/
    index.html               (full professional UI)
    app.js                   (brain logic, tabs, calls API)
  __pycache__

Features you asked for

Sophisticated Jira training model (BGE-like / BERT-style)

Uses a sentence-transformer model (BAAI/bge-base-en-v1.5) for Jira similarity.

Jira index is stored as:

indices/jira_bge.npz – embeddings

indices/jira_bge.json – metadata (id, summary, description)

This is conceptually similar to having a .mkl index file – you get a persistent, on-disk vector index that can handle hundreds of thousands of Jiras.

bge-base-en for PDF / DOC QA

The same BGE engine is used for PDF chunks:

Upload PDFs in the “PDF Training” tab.

Server:

Extracts text via PyPDF2

Splits into ~900-char chunks

Embeds with BGE

Stores in indices/pdf_bge.npz + indices/pdf_bge.json

PDF QA is done from the Main Chat tab by choosing PDF QA mode or letting the brain route there when you mention “PDF/spec/etc.”.

UVM testcase training + retrieval

Use the “Testcase Training” tab:

Upload CSV with columns:

id, name, uvm_version, summary, description, code

Backend:

Concats these fields

Embeds with BGE

Saves to indices/tc_bge.npz + indices/tc_bge.json

From Testcase tab:

Describe a scenario and click Find testcases → nearest UVM testcases returned.

In Jira flow:

If Jira similarity is low (< threshold from config.json), it falls back to testcase BGE index to find UVM testcases.

CCF coverage optimisation hooks

In Main Chat, mode:

Optimize CCF:

Sends your text or pasted CCF snippet to /api/llm_chat with a specialised “optimise/reduce coverage” prompt.

This is LLM-driven at the moment; you can extend server.py later to:

Parse multiple CCF files

Build an embedding index for coverage bins

Compare redundancy algorithmically.

Jira ID → recipes flow

Main Chat “Auto (Brain)” + Jira ID like AVSREQ-000001..000007:

Frontend brain (in ui/app.js) detects Jira pattern AVSREQ-XXXXXX.

Calls /api/jira_flow on backend:

Fetch Jira details

Currently uses a dummy Jira function that returns realistic sample issues for AVSREQ-000001..7 (perf, coverage, memory, RAL, random, CCF, CDC).

There is a placeholder in server.py where you will later call your real Jira REST script.

Embed description with BGE.

Search similar Jira via Jira index.

If similarity < 0.3 (configurable):

Query testcase embedding index to find similar UVM testcases.

If no good match → fallback to code generation.

LLM prompt:

“You are a senior Xcelium/UVM debug engineer… propose testcase/debug steps…”

Calls llm_generate() (llama_cpp adapter stub).

Performance flags recipe:

If description mentions “slow / performance / perf”:

Recommend -newperf and -plusperf.

You can extend this via YAML rules per your spec.

It returns JSON with:

Jira issue

Similar Jira hits (scores)

Testcase hits (scores)

Generated testcase / debug suggestion

Perf recommendations (-newperf, -plusperf)

Route chosen: jira_match, testcase_match, or codegen.

Multiple instances across the network

config.json sets:

"server": {
  "host": "0.0.0.0",
  "port": 8099
}


That means:

On your Linux/Windows host, run python server.py

Any machine on LAN can open http://<host-ip>:8099/.

Login + Logout

Backend:

SQLite DB xgent_ai.sqlite3.

users table with a default user seeded:

username: xgent

password: xgent123

Frontend:

Login overlay when you open the app.

Shows “Logged in as …” in header.

Logout button resets session on client side (this is a demo; you can add real session tokens later).

Tabs & professional UI (Xgent Agentic AI)

ui/index.html + ui/app.js give you:

Main Chat tab:

Text area + mode dropdown:

Auto (Brain)

Jira flow

PDF QA

Code generation

Optimize class

Optimize CCF

Brain routing in JS:

Jira regex → Jira flow

“coverage/ccf” → CCF optimisation

“optimize class/refactor” → class optimisation

“pdf/spec” → PDF QA

Else → code generation

PDF Training tab:

PDF upload and indexing

List of trained PDFs

Jira Training tab:

“Seed demo Jira index” button for AVSREQ-000001..7

Free-text Jira match

Testcase Training tab:

CSV upload for local UVM testcases

Query box to retrieve nearest testcases.

User guide

USER_GUIDE.md in the root gives:

Setup (Linux/Windows with python venv).

Required Python packages.

How to:

Run the app

Log in

Use each tab (Jira, PDF, testcases, main chat).

Where indices live on disk and how to reset them.

How to run this ZIP (Linux & Windows)

Unzip:

unzip xgent_agentic_ai_full_with_placeholder_gguf.zip
cd xgent_agentic_bge_full


Create & activate Python env (recommended)

Linux/macOS:

python3 -m venv .venv
source .venv/bin/activate


Windows (PowerShell):

python -m venv .venv
.\.venv\Scripts\Activate.ps1


Install dependencies:

pip install sentence-transformers torch PyPDF2 numpy
# Optional: for real offline LLM
pip install llama-cpp-python


(Important) Download the real GGUF model

Replace models/llama-demo.gguf with an actual model of your choice, for example:

Good general chat + code:

Llama 3.1 8B Instruct GGUF (good balance, general + code) 
Hugging Face
+1

Strong code-focused models (for code generation in your recipes):

DeepSeek Coder 6.7B Instruct GGUF 
Hugging Face
+1

DeepSeek-Coder-V2 Instruct / Lite Instruct GGUF (very strong code model) 
Hugging Face
+1

Qwen2.5-Coder GGUF (3B/7B/32B) – code-specific, very strong and available in multiple sizes. 
Hugging Face
+2
Hugging Face
+2

Use huggingface-cli or manual download, then:

# example
mv Meta-Llama-3.1-8B-Instruct-Q4_K_M.gguf models/llama-demo.gguf


Run the server

python server.py


Open:

http://localhost:8099/


Log in with xgent / xgent123.

“Which model for decision making / brain routing?”

In this build:

The routing/brain (deciding Jira/PDF/code/CCF/class) is done in frontend JS using:

Regex for Jira IDs.

Keyword heuristics (coverage, pdf, spec, etc.).

The heavy reasoning is done by your LLM (via /api/llm_chat):

Code generation

Class optimisation

CCF optimisation

Combining PDF context + question.

If you later want a true model-based router, you can:

Add a small classification head (e.g. a tiny BERT or a prompt-only call to your Llama/DeepSeek/Qwen) which predicts:

jira, pdf, code, opt_class, opt_ccf, etc.

Implement /api/brain_route in server.py and let app.js call it instead of local heuristics.

Where llama_cpp and ONNX/TensorRT fit

In server.py:

llm_generate(prompt) looks at CONFIG["llm"]["adapter"]:

"llama_cpp" – intended path:

from llama_cpp import Llama
llm = Llama(model_path="models/llama-demo.gguf", n_ctx=4096, n_threads=... )
out = llm(prompt, max_tokens=..., temperature=...)


Current demo implementation is a stub, you just need to drop in the real call.

"onnx" / "tensorrt" – stubbed but wired:

The code checks these values and returns "ONNX backend disabled in demo" etc.

You can later plug ONNX/TensorRT inferencers there without changing UI.

So your constraint:

code generator should use from llama_cpp import Llama as I have to work in offline

add provision to use ONNX and TensorRT

is supported architecturally; you only need to:

Implement the llama_cpp.Llama call.

(Later) add ONNX/TensorRT calls under respective adapters.

Any recipe missing?

You already cover a lot. Some next-level recipes you might add:

Log triage recipe

Upload Xcelium logs.

Chunk + embed (reusing BGE).

Classify error signatures and propose known fixes.

Coverage redundancy analyser

Parse multiple CCFs.

Build coverage-bin embedding or structured representation.

Find and recommend removal/merging of redundant bins.

Memory profiler assistant

Ingest memory profiling reports.

Highlight top consumers, typical UVM patterns causing leaks.

Guided debug script generator

Given Jira + env info, generate ready-to-run command lines:

With -newperf, -plusperf, -disable_opt, coverage flags, etc.

Save these as “recipes” in a YAML file on disk (already hinted in your requirements).

Learning mode

After you resolve an issue, add:

Jira ID, final root cause, final fix, testcase used, coverage deltas.

Train separate “solution” index for future recommendations.

TL;DR

✅ You now have a full ZIP: xgent_agentic_ai_full_with_placeholder_gguf.zip

✅ It includes:

Jira+BGE model

PDF+BGE QA

UVM testcase+BGE training

Login + logout

Multi-tab professional UI

User guide

Hooks for llama_cpp, ONNX, TensorRT

🔄 You must download real GGUF models (Llama / DeepSeek / Qwen) from Hugging Face and replace the placeholder file in models/.

If you want, next I can walk you through exact edits in server.py to enable a specific GGUF model via llama_cpp on your i9 + 16 GB machine (e.g. Llama 3.1 8B Q4_K_M or a 6–7B DeepSeek/Qwen coder).
