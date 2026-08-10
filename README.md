# 👕 AtliQ T Shirts — AI Database Q&A Assistant

A natural-language interface for querying a real MySQL sales database. Ask plain-English business questions — *"How much revenue will we generate today if we sell all Nike t-shirts after discount?"* — and get back an accurate SQL-backed answer, no SQL required.

Built with LangChain's SQL agent chain, Google PaLM, and few-shot prompting to keep the generated SQL both correct and grounded in how *this specific* database is actually queried.

---

## ✨ Features

- **Text-to-SQL** — converts natural language questions into syntactically correct MySQL queries automatically
- **Few-shot learning** — a curated bank of question → SQL → answer examples steers the LLM toward the query patterns this schema actually needs (joins, discounts, aggregations)
- **Semantic example selection** — retrieves the *most relevant* few-shot examples per question via sentence-transformer embeddings + ChromaDB, instead of stuffing the prompt with every example every time
- **Live database execution** — runs the generated query against a real MySQL instance and returns the actual result, not a guess
- **Discount-aware revenue logic** — correctly joins inventory and discount tables to answer pre- and post-discount revenue questions
- **Streamlit UI** — ask questions through a simple web app

---

## 🧠 How It Works

```
Natural language question
        │
        ▼
Semantic Example Selector (MiniLM embeddings + ChromaDB)
        │   retrieves top-k most similar few-shot examples
        ▼
Few-Shot SQL Prompt (MySQL-specific instructions + examples)
        │
        ▼
Google PaLM LLM ──► generates SQL query
        │
        ▼
SQLDatabaseChain ──► executes query against MySQL
        │
        ▼
Final natural-language answer
```

1. A MySQL database (`atliq_tshirts`) holds t-shirt inventory (`t_shirts`) and per-item `discounts`, seeded with ~100 randomly generated records via a stored procedure.
2. A bank of hand-written few-shot examples (`few_shots.py`) teaches the model the exact SQL patterns this schema needs — including the discount-join logic that's easy to get wrong.
3. For each incoming question, a semantic similarity selector picks the 2 most relevant examples to include in the prompt.
4. LangChain's `SQLDatabaseChain` generates the SQL, runs it against the live database, and returns a final answer.

---

## 📁 Project Structure

```
├── main.py                          # Streamlit app entry point
├── langchain_helper.py              # Core chain: DB connection + few-shot SQL prompt
├── few_shots.py                     # Curated question/SQL/answer examples
├── db_creation_atliq_t_shirts.sql   # Schema + seed data (t_shirts, discounts, stored procedure)
├── t_shirt_sales_llm.ipynb          # Exploratory notebook walking through the pipeline
├── atliq_tees.png                   # Reference banner/UI image
├── requirements.txt                 # Python dependencies
└── env.txt                          # Sample environment variable file
```

---

## 🚀 Getting Started

### 1. Clone the repository
```bash
git clone https://github.com/avankumar23/AI_TShirt_Sales_Assistant.git
cd AI_TShirt_Sales_Assistant
```

### 2. Set up the MySQL database
Run the schema script against a local MySQL instance:
```bash
mysql -u root -p < db_creation_atliq_t_shirts.sql
```
This creates the `atliq_tshirts` database, the `t_shirts` and `discounts` tables, and populates ~100 t-shirt records via a stored procedure.

### 3. Install Python dependencies
```bash
pip install -r requirements.txt
```

### 4. Configure environment variables
Create a `.env` file (use `env.txt` as a template):
```
GOOGLE_API_KEY='your_api_key_here'
```
Get a free key from [Google MakerSuite](https://makersuite.google.com/).

By default the app connects to MySQL with `root` / `root` on `localhost` — update the credentials in `langchain_helper.py` if yours differ.

### 5. Run the app
```bash
streamlit run main.py
```

---

## 💡 Example Questions

- *"How many t-shirts do we have left for Nike in XS size and white color?"*
- *"How much is the total price of the inventory for all S-size t-shirts?"*
- *"If we sell all Levi's t-shirts today with discounts applied, how much revenue will we generate?"*
- *"How much sales amount will be generated if we sell all large-size Nike t-shirts today after discounts?"*

---

## 🛠️ Tech Stack

| Component            | Technology                                  |
|-----------------------|----------------------------------------------|
| LLM                   | Google PaLM (`langchain.llms.GooglePalm`)    |
| SQL Generation        | LangChain `SQLDatabaseChain` (`langchain_experimental`) |
| Example Embeddings    | Sentence-Transformers (`all-MiniLM-L6-v2`)   |
| Example Vector Store  | ChromaDB                                     |
| Database              | MySQL                                        |
| UI                    | Streamlit                                    |

---

## 📌 Notes

- The few-shot examples in `few_shots.py` are the real lever for accuracy on this project — when you see the model generate a wrong join or aggregation, the fix is usually adding a new example, not rewriting the prompt.
- `sample_rows_in_table_info=3` gives the LLM a peek at real row data alongside the schema, which noticeably improves column/value grounding.
- Swap `GooglePalm` for any other LangChain-supported LLM by editing `langchain_helper.py`.

---

## 🙋 Author

Built by [**Avan Kumar**](https://github.com/avankumar23) as part of a data analytics & applied AI portfolio.
