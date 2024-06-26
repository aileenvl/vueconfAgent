
# AI Agent for VueConfUS Conference Q&A with Vue, Vite, Gemini Model, OpenAI Models, Supabase, and LangChain.js

This project is a basic AI agent for handling conference Q&A built using Vue, Vite, the Gemini model, OpenAI models, Supabase for vector database, and LangChain.js. The agent can process conference-related questions from a text file and embed them into the vector database using Supabase.

## Requirements

- Node.js (v14.0.0 or later)
- Vite CLI
- Supabase account https://supabase.com/ you can have free DB (Free plan. Supabase provides two "Free organizations". Each organization can run a Nano instance for free. This is a great way to get started with Supabase and try out the platform.)
- LangChain.js https://js.langchain.com/v0.2/docs/introduction/
- OpenAI API key (for text embedding) https://openai.com/
- API key for any language model of your choice (e.g., Gemini, Mistral, Claude3) for generating responses
  https://gemini.google.com/
  https://mistral.ai/
  https://www.anthropic.com/claude

In this project, we use OpenAI for text embedding, which is the process of converting text into numerical vectors that can be stored in a vector database like Supabase. However, you can use any other language model API of your choice for generating responses to the user's queries.

## Getting Started

## Supabase
Sure, here are the steps to create a vector database in Supabase manually:

1. Create a new Supabase project by signing up or logging in to your Supabase account.
2. In the project dashboard, click on the "Database" tab.
3. Click on the "SQL Editor" tab.
4. Create a new table with a vector column by running the following SQL command:
```sql
CREATE TABLE documents (
  id SERIAL PRIMARY KEY,
  content TEXT,
  embedding vector(1024)
);
```
5. Insert data into the vector column by running the following SQL command:
```sql
INSERT INTO documents (content, embedding)
VALUES (
  'Your document content here',
  vector(1.234, -0.567, ..., 0.987)
);
```
Note: The vector values should be enclosed in parentheses and separated by commas.

6. To query the vector column and find the most similar documents to a given query embedding, you can use the `<=>` operator in a SQL query. Here is an example using the `match_vueconf_docs` function I provided earlier:
```sql
CREATE OR REPLACE FUNCTION match_vueconf_docs (
  query_embedding vector(1024),
  match_threshold float,
  match_count int
)
RETURNS TABLE (
  id bigint,
  content text,
  similarity float
)
AS $$
  SELECT
    id,
    content,
    1 - (embedding <=> $1) AS similarity
  FROM documents
  WHERE 1 - (embedding <=> $1) > $2
  ORDER BY (embedding <=> $1) ASC
  LIMIT $3;
$$ LANGUAGE SQL STABLE;

SELECT * FROM match_vueconf_docs(
  vector(1.234, -0.567, ..., 0.987),
  0.7,
  10
);
```
This query will return the 10 documents in the "documents" table that are most similar to the given query embedding, with a similarity score greater than 0.7.

## Project

1. Clone the repository: `git clone https://github.com/yourusername/ai-agent.git`
2. Install dependencies: `npm install`
3. Set up environment variables:
   - Create a `.env` file in the root directory and add your Supabase and OpenAI API keys.
   - `VITE_SUPABASE_URL=your_supabase_url`
   - `VITE_SUPABASE_API_KEY=your_supabase_anon_key`
   - `VITE_OPENAI_API_KEY=your_openai_api_key`
4. Start the development server: `npm run dev`

## Usage

1. Add the data-related you want to process in the `assets` directory.
2. The agent will take the text, pdf or data source to process and embed the questions into the Supabase vector database, Im currently using a txt so thats the example function youll see.
3. You can interact with the agent through the Vue interface by asking questions.

**Note:** This is a basic example and does not cover edge cases, such as follow-up questions or more complex queries.

## Technologies Used

- Vue: A progressive JavaScript framework for building user interfaces.
- Vite: A build tool that aims to provide a faster and leaner development experience for modern web projects.
- Gemini Model: A large-scale model that understands complex instructions and generates high-quality responses.
- OpenAI Models: Used for language understanding and generation.
- Supabase: A backend-as-a-service that provides a PostgreSQL database and Realtime subscriptions.
- LangChain.js: A JavaScript library for building language models.

## Contributing

Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

## License

[MIT](https://choosealicense.com/licenses/mit/)

---