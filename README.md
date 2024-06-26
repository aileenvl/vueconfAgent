Sure, here's the updated README:

# AI Agent for VueConfUS Conference Q&A with Vue, Vite, Gemini Model, OpenAI Models, Supabase, and LangChain.js

This project is a basic AI agent for handling conference Q&A built using Vue, Vite, the Gemini model, OpenAI models, Supabase for vector database, and LangChain.js. The agent can process conference-related questions from a text file and embed them into the vector database using Supabase.

## Requirements

- Node.js (v14.0.0 or later)
- Vite CLI
- Supabase account
- OpenAI API key (for text embedding)
- API key for any language model of your choice (e.g., Gemini, Mistral, Claude3) for generating responses

In this project, we use OpenAI for text embedding, which is the process of converting text into numerical vectors that can be stored in a vector database like Supabase. However, you can use any other language model API of your choice for generating responses to the user's queries.

## Getting Started

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