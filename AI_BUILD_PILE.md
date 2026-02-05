# AI Build Pile - Roberto Docs

## Smart Bot Toolkit

### Ramverk
| Verktyg | Användning |
|---------|------------|
| PyTorch | Flexibelt, research & custom models |
| TensorFlow/Keras | Produktion, enklare syntax |
| Hugging Face Transformers | LLMs, NLP, färdiga modeller |
| LangChain | LLM-applikationer, RAG, agents |

### Installation
```bash
# Grundläggande
pip install torch transformers datasets accelerate
pip install langchain openai chromadb

# Fine-tuning
pip install peft trl bitsandbytes

# Lokalt LLM
curl -fsSL https://ollama.com/install.sh | sh
ollama pull llama3
```

---

## Smart Bot Kod

### OpenAI/Claude API
```python
from openai import OpenAI

client = OpenAI(api_key="din-nyckel")

def bot(fråga):
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {"role": "system", "content": "Du är en hjälpsam assistent."},
            {"role": "user", "content": fråga}
        ]
    )
    return response.choices[0].message.content
```

### Lokalt med Ollama
```python
import ollama

def bot(fråga):
    response = ollama.chat(model="llama3", messages=[
        {"role": "user", "content": fråga}
    ])
    return response["message"]["content"]
```

### Med Minne (Konversation)
```python
historik = []

def bot(fråga):
    historik.append({"role": "user", "content": fråga})
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=historik
    )
    svar = response.choices[0].message.content
    historik.append({"role": "assistant", "content": svar})
    return svar
```

### Anthropic Claude
```python
from anthropic import Anthropic

client = Anthropic(api_key="...")
response = client.messages.create(
    model="claude-sonnet-4-20250514",
    messages=[{"role": "user", "content": "Hej!"}]
)
```

---

## Sequential Thinking / Memory System

### Vektor-databaser (RAG)
| DB | Bäst för |
|----|----------|
| ChromaDB | Snabb prototyping, lokalt |
| Pinecone | Produktion, managed |
| Weaviate | Hybrid search |
| FAISS | Stort dataset, snabbt |

### RAG Pipeline
```python
from langchain.embeddings import OpenAIEmbeddings
from langchain.vectorstores import Chroma
from langchain.chains import RetrievalQA
from langchain.llms import OpenAI

# Ladda dokument
embeddings = OpenAIEmbeddings()
vectorstore = Chroma.from_documents(documents, embeddings)

# Sök + svara
qa = RetrievalQA.from_chain_type(
    llm=OpenAI(),
    retriever=vectorstore.as_retriever()
)
svar = qa.run("Din fråga här")
```

### Sequential Thinking Agent
```python
from langchain.agents import initialize_agent, Tool
from langchain.memory import ConversationBufferMemory

memory = ConversationBufferMemory(memory_key="chat_history")

tools = [
    Tool(name="Search", func=search_func, description="Sök information"),
    Tool(name="Calculate", func=calc_func, description="Räkna ut saker"),
]

agent = initialize_agent(
    tools,
    llm,
    agent="conversational-react-description",
    memory=memory
)
```

---

## Fine-tuning (LoRA)
```python
from transformers import AutoModelForCausalLM, AutoTokenizer
from peft import LoraConfig, get_peft_model

model = AutoModelForCausalLM.from_pretrained("mistralai/Mistral-7B-v0.1")
tokenizer = AutoTokenizer.from_pretrained("mistralai/Mistral-7B-v0.1")

lora_config = LoraConfig(
    r=16,
    lora_alpha=32,
    target_modules=["q_proj", "v_proj"]
)
model = get_peft_model(model, lora_config)
```

---

## Verktyg
- **Ollama** - Kör LLMs lokalt
- **LM Studio** - GUI för lokala modeller
- **vLLM** - Snabb inference server
- **Weights & Biases** - Experiment tracking
- **Label Studio** - Data labeling

---

## AOSP / GrapheneOS Build

### Setup
```bash
# Repo tool
mkdir ~/bin
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo

# Hämta AOSP
repo init -u https://android.googlesource.com/platform/manifest
repo sync -j8

# GrapheneOS
repo init -u https://github.com/GrapheneOS/platform_manifest.git -b 14
repo sync -j8
```

### Build
```bash
source build/envsetup.sh
lunch aosp_arm64-userdebug
m -j$(nproc)
```

---

## Uppgiftsfördelning

| Agent | Ansvar |
|-------|--------|
| **Cline** | Setup, API-integration, endpoints |
| **Sixth** | Modell, system prompt, testning |

---

## Tor Integration (Direkt)

### Installation
```bash
# Linux
sudo apt install tor
sudo systemctl enable --now tor

# Verify
curl --socks5 127.0.0.1:9050 https://check.torproject.org/api/ip
```

### Python via Tor
```python
import requests

proxies = {
    'http': 'socks5h://127.0.0.1:9050',
    'https': 'socks5h://127.0.0.1:9050'
}

# All requests via Tor
response = requests.get("https://api.example.com", proxies=proxies)
```

### OpenAI/LLM via Tor
```python
import httpx
from openai import OpenAI

# Custom transport via Tor
transport = httpx.HTTPTransport(proxy="socks5://127.0.0.1:9050")
http_client = httpx.Client(transport=transport)

client = OpenAI(
    api_key="din-nyckel",
    http_client=http_client
)
```

### Tor Hidden Service (egen .onion)
```bash
# /etc/tor/torrc
HiddenServiceDir /var/lib/tor/hidden_service/
HiddenServicePort 80 127.0.0.1:8080

# Starta om
sudo systemctl restart tor

# Hämta .onion adress
sudo cat /var/lib/tor/hidden_service/hostname
```

---

## Three Apes (Privacy Layer + Memory)

**Som Orbot, men 3x bättre.** Tre lager istället för ett.

---

### Ape 1: SEE (Entry)
```
┌─────────────────────────────┐
│  VPN Tunnel In              │
│  - Tar emot all trafik      │
│  - Krypterar första lagret  │
│  - Session memory           │
└─────────────────────────────┘
```
**Minne:** Kort (session context)
**Roll:** Entry guard, första krypteringen

---

### Ape 2: THINK (Middle)
```
┌─────────────────────────────┐
│  Tor Relay                  │
│  - Anonymiserar             │
│  - Processar requests       │
│  - Working memory           │
└─────────────────────────────┘
```
**Minne:** Arbetsminne (reasoning state)
**Roll:** Anonymisering, processning

---

### Ape 3: ACT (Exit)
```
┌─────────────────────────────┐
│  Clean Exit                 │
│  - Levererar output         │
│  - Persistent storage       │
│  - Ingen trace              │
└─────────────────────────────┘
```
**Minne:** Långt (persistent, encrypted)
**Roll:** Exit node, ren output

---

### Full Pipeline
```
User → [Ape1:VPN] → [Ape2:Tor] → [Ape3:Exit] → Internet
                                      ↓
                              Clean response
```

### Implementation
```python
class ThreeApes:
    def __init__(self):
        # Ape 1: See - session memory
        self.see_memory = []

        # Ape 2: Think - working memory
        self.think_memory = {}

        # Ape 3: Act - long-term memory
        self.act_memory = VectorStore()  # ChromaDB/FAISS

    def see(self, input):
        """Ape 1: Ta emot och kontextualisera"""
        self.see_memory.append(input)
        return self._contextualize(input)

    def think(self, context):
        """Ape 2: Processa och resonera"""
        self.think_memory['current'] = context
        return self._reason(context)

    def act(self, result):
        """Ape 3: Agera och spara"""
        self.act_memory.store(result)
        return self._output(result)

    def process(self, input):
        """Full pipeline genom alla apor"""
        seen = self.see(input)
        thought = self.think(seen)
        action = self.act(thought)
        return action
```

All bot-kommunikation går genom Tor-kretsen + memory chain.

---

## Bot Modes

Clean coding machine med olika lägen:

| Mode | Beteende |
|------|----------|
| **Zen Mode** | Minimal output, fokuserad, ren kod, inga onödiga kommentarer |
| **Master Mode** | Full kraft, djup analys, optimering, expert-nivå |

### Implementation
```python
class RobertoBot:
    def __init__(self):
        self.mode = "zen"
        self.memory = []

    def set_mode(self, mode):
        self.mode = mode

    def respond(self, query):
        if self.mode == "zen":
            return self._zen_response(query)
        elif self.mode == "master":
            return self._master_response(query)

    def _zen_response(self, query):
        # Kort, ren, fokuserad
        system = "Du är en minimalistisk kodare. Ge ren kod utan förklaringar."
        return self._call_llm(system, query)

    def _master_response(self, query):
        # Djup, analytisk, expert
        system = "Du är en expert-kodare. Analysera djupt, optimera, förklara trade-offs."
        return self._call_llm(system, query)
```

---

## Constructor Thinking (Sequential Memory)

Boten bygger upp resonemang steg för steg, som en constructor.

### Flöde
```
Input → Decompose → Build Steps → Chain → Output
```

| Steg | Funktion |
|------|----------|
| **Decompose** | Bryt ner problemet i delar |
| **Build** | Konstruera lösning bit för bit |
| **Chain** | Länka stegen logiskt |
| **Memory** | Spara kontext mellan steg |

### Implementation
```python
class ConstructorThinking:
    def __init__(self):
        self.steps = []
        self.memory = {}

    def decompose(self, problem):
        # Bryt ner i sub-problems
        return self._call_llm(f"Bryt ner detta problem i steg: {problem}")

    def build_step(self, step, context):
        # Bygg ett steg med kontext från tidigare
        self.memory[step] = context
        return self._call_llm(f"Lös steg: {step}\nKontext: {context}")

    def chain(self, steps):
        result = ""
        for i, step in enumerate(steps):
            context = result if result else "Start"
            result = self.build_step(step, context)
            self.steps.append({"step": step, "result": result})
        return result

    def think(self, problem):
        # Full constructor flow
        sub_problems = self.decompose(problem)
        return self.chain(sub_problems)
```

### Med LangChain
```python
from langchain.chains import SequentialChain
from langchain.prompts import PromptTemplate

# Steg 1: Analysera
analyze = LLMChain(
    llm=llm,
    prompt=PromptTemplate(template="Analysera: {input}")
)

# Steg 2: Planera
plan = LLMChain(
    llm=llm,
    prompt=PromptTemplate(template="Planera baserat på: {analysis}")
)

# Steg 3: Implementera
implement = LLMChain(
    llm=llm,
    prompt=PromptTemplate(template="Implementera: {plan}")
)

# Chain ihop
chain = SequentialChain(chains=[analyze, plan, implement])
```

---

## Ultimate Setup (Linux Mint)

```
┌────────────────────────────────────────────┐
│              LINUX MINT                    │
├────────────────────────────────────────────┤
│  RAM: ~90% → Ollama/LLM                    │
│  - Större modell = bättre output           │
│  - codellama:34b eller llama3:70b          │
│  - GPU offload om tillgängligt             │
├────────────────────────────────────────────┤
│  CPU: Constructor Thinking pipeline        │
│  SSD: ChromaDB vektor-minne                │
├────────────────────────────────────────────┤
│           ↓ OUTPUT ↓                       │
│  Cloud sync (Nextcloud/rsync)              │
│  - Kod → repo                              │
│  - Resultat → cloud storage                │
└────────────────────────────────────────────┘
```

### Config
```bash
# /etc/systemd/system/ollama.service.d/override.conf
[Service]
Environment="OLLAMA_MAX_LOADED_MODELS=1"
Environment="OLLAMA_NUM_PARALLEL=1"
Environment="OLLAMA_MAX_QUEUE=1"

# Ge Ollama max RAM
sudo sysctl vm.swappiness=10
```

### Cloud Output
```python
import subprocess

def sync_to_cloud(local_path, remote_path):
    # Nextcloud via rclone
    subprocess.run(["rclone", "sync", local_path, f"nextcloud:{remote_path}"])

def push_to_repo(path, msg):
    subprocess.run(["git", "-C", path, "add", "."])
    subprocess.run(["git", "-C", path, "commit", "-m", msg])
    subprocess.run(["git", "-C", path, "push"])
```

---

## Security (Online Mode)

När Roberto jobbar på internet.

### Three Apes VPN Layer
```
┌─────────────────────────────┐
│  Ape 1: Entry (VPN in)      │
│     ↓                       │
│  Ape 2: Tor relay           │
│     ↓                       │
│  Ape 3: Exit (clean out)    │
└─────────────────────────────┘
```

### Security Stack
```
┌─────────────────────────────┐
│  1. VPN (first hop)         │
│  2. Tor (anonymitet)        │
│  3. TLS/SSL (kryptering)    │
│  4. Input sanitering        │
│  5. Rate limiting           │
│  6. Ingen loggning          │
└─────────────────────────────┘
```

### Implementation
```python
import os
import hashlib
from functools import lru_cache

class RobertoSecure:
    def __init__(self):
        self.api_key = os.environ.get("API_KEY")  # Aldrig hårdkoda
        self.rate_limit = {}

    def sanitize(self, input):
        # Ta bort farliga tecken
        dangerous = ["<script>", "DROP TABLE", "../", "eval("]
        for d in dangerous:
            input = input.replace(d, "")
        return input.strip()[:10000]  # Max längd

    def rate_check(self, user_id, limit=10):
        count = self.rate_limit.get(user_id, 0)
        if count >= limit:
            raise Exception("Rate limit")
        self.rate_limit[user_id] = count + 1
        return True

    def hash_sensitive(self, data):
        # Hasha känslig data innan loggning
        return hashlib.sha256(data.encode()).hexdigest()[:16]

    def process(self, user_id, input):
        self.rate_check(user_id)
        clean = self.sanitize(input)
        # Processa via Tor...
        return self._call_via_tor(clean)
```

### Env
```bash
# .env (ALDRIG committa)
API_KEY=xxx
TOR_SOCKS=socks5://127.0.0.1:9050

# .gitignore
.env
*.key
*.pem
credentials/
```

### Principer
| Regel | Varför |
|-------|--------|
| Aldrig logga input | Privacy |
| Tor för alla requests | Anonymitet |
| Sanitera allt | Injection-skydd |
| Rate limit | DoS-skydd |
| Env vars för secrets | Läcker inte i kod |

---

## Offline Mode

Helt lokal, ingen internet.

### Stack
| Komponent | Funktion |
|-----------|----------|
| **Ollama** | Lokal LLM (codellama/llama3) |
| **ChromaDB** | Vektor-minne på disk |
| **SQLite** | Session/config |

### Setup
```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama pull codellama
```

### Implementation
```python
import ollama
import chromadb

class RobertoOffline:
    def __init__(self):
        self.llm = "codellama"
        self.db = chromadb.PersistentClient(path="./memory")
        self.collection = self.db.get_or_create_collection("mem")

    def _call_llm(self, system, query):
        return ollama.chat(
            model=self.llm,
            messages=[
                {"role": "system", "content": system},
                {"role": "user", "content": query}
            ]
        )["message"]["content"]

    def store(self, text):
        self.collection.add(documents=[text], ids=[str(hash(text))])

    def recall(self, query, n=3):
        return self.collection.query(query_texts=[query], n_results=n)["documents"]
```

---

## Claude Code Tools (Tillgängliga)

| Verktyg | Funktion | Användning |
|---------|----------|------------|
| **Bash** | Kör kommandon | git, npm, builds, system |
| **Read** | Läs filer | Kod, config, bilder, PDF |
| **Write** | Skapa filer | Ny kod, config |
| **Edit** | Redigera filer | Ändra befintlig kod |
| **Glob** | Hitta filer | `**/*.py`, `src/**/*.ts` |
| **Grep** | Sök innehåll | Regex i kodbas |
| **WebFetch** | Hämta webbsidor | Dokumentation, API docs |
| **WebSearch** | Söka internet | Research, lösningar |
| **Task** | Sub-agenter | Explore, Plan, Bash agents |
| **TodoWrite** | Spåra tasks | Progress tracking |
| **NotebookEdit** | Jupyter | .ipynb redigering |

### Sub-agenter (Task tool)

| Agent | Specialitet |
|-------|-------------|
| **Explore** | Snabb kodbas-utforskning |
| **Plan** | Arkitektur, implementation strategy |
| **Bash** | Komplexa shell-operationer |
| **general-purpose** | Multi-step research |

### Parallell Execution

```python
# Kan köra flera verktyg samtidigt:
# - Parallella sökningar
# - Flera filer samtidigt
# - Background tasks
```

---

## TODO: Roberto Docs

- [x] Three Apes koncept - Tor-lager
- [x] Bot modes - Zen + Master
- [x] Constructor Thinking - sequential memory
- [ ] Burnouts logik - implementera
- [ ] AOSP/GrapheneOS integration
- [ ] App framework setup

---

*Genererad för Roberto Docs projekt*
