<template>
  <div class="container">
    <h1 class="title">Vue Conf</h1>
    <p class="subtitle">Agent</p>
    <p v-if="loading">Loading...</p>
    <ul v-else>
      <li v-for="(match, index) in matches" :key="index">
        {{ match.content }}
      </li>
    </ul>
    <form @submit.prevent="handleSubmit">
      <label for="query">Ask a question:</label>
      <input id="query" v-model="query" type="text" name="query">
      <div class="card flex justify-content-center">
      </div>
      <button class="submit_buttom" type="submit">Submit</button>
    </form>
    <p v-if="response">{{ response }}</p>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue';
import { GoogleGenerativeAI } from "@google/generative-ai";
import OpenAI from "openai";
import { RecursiveCharacterTextSplitter } from "langchain/text_splitter";
import { createClient } from "@supabase/supabase-js";

const geminiApiKey = import.meta.env.VITE_GEMINI_API_KEY;
const openAIKey = import.meta.env.VITE_OPENAI_API_KEY;
const supabaseUrl = import.meta.env.VITE_SUPABASE_URL;
const supabaseKey = import.meta.env.VITE_SUPABASE_API_KEY;

export default {
  setup() {
    const client = new GoogleGenerativeAI(geminiApiKey);

    const clientEmbedding = new OpenAI({apiKey:openAIKey,
      dangerouslyAllowBrowser: true });

    const supabase = createClient(supabaseUrl, supabaseKey);
    const chatResponse = ref(null);
    const vueInfoChunks = ref(null);
    const vueconf = ref('');
    const query = ref('');
    const loading = ref(false);
    const matches = ref([]);
    const response = ref('');
  
  // Step 1: Loading the text file
  async function loadTextFile() {
    try {
      const module = await import('./assets/vueconfData.txt');
      vueconf.value = await fetch(module.default).then(res => res.text());
    } catch (error) {
      console.error('Error loading text file:', error);
    }
  }
  // Step 2: Splitting the document into chunks
  async function splitDocument(text) {
    try {
      const splitter = new RecursiveCharacterTextSplitter({
        chunkSize: 400,
        chunkOverlap: 20
      });
      const output = await splitter.createDocuments([text]);
      const stringFormat = output.map((chunk) => chunk.pageContent);
      vueInfoChunks.value = stringFormat;
    } catch (error) {
      console.error(error);
    }
  }
  //Step 3: Creating embeddings for each chunk with openAI text-embedding-ada-002 model
  
  async function createEmbeddings(chunks) {
    try {
      const embeddings = await clientEmbedding.embeddings.create({
        model: "text-embedding-ada-002",
        input:  chunks,
      });
      const data = chunks.map((chunk, i) => {
        return {
          content: chunk,
          embedding: embeddings.data[i].embedding
        }
      });
      console.log(data);
      return data;
      
    } catch (error) {
      console.error(error);
    }
  }

  const handleSubmit = async () => {
    loading.value = true;
    const embedding = await createEmbedding(query.value);
    const context = await retrieveMatches(embedding);
    response.value = await generateChatResponse(context, query.value);
    loading.value = false;
  };

// 2. Creating an embedding of the input with same model
async function createEmbedding(input) {
  const embeddingResponse = await clientEmbedding.embeddings.create({
        model: "text-embedding-ada-002",
        input:  input,
      });
  return embeddingResponse.data[0].embedding;
}

// 3. Retrieving similar embeddings / text chunks
async function retrieveMatches(embedding) {
  const { data } = await supabase.rpc('match_vueconf_docs', {
    query_embedding: embedding,
    match_threshold: 0.50,
    match_count: 8 
  });
  console.log(data, 'data')
  if (data) {
    return  data.map(chunk => chunk.content).join(" ");;
  } else {
    return [];
  }
}

// 4. Combining the input and the context in a prompt 
// and using the chat API to generate a response 
async function generateChatResponse(context, query) {
      const model = client.getGenerativeModel({ model: "gemini-pro" });

      const chatHistory = [
        {
          role: "user",
          parts: [{ text: `User has a question` }],
        },
        {
          role: "model",
          parts: [{ text: `Here is some context: ${context}` }],
        },
      ];

      const chat = model.startChat({
        history: chatHistory,
        generationConfig: {
          maxOutputTokens: 1000,
        },
      });
      
      const result = await chat.sendMessage(query);
        const response = await result.response;
        const text = await response.text();
        return text;
    }
      onMounted(async () => {
        await loadTextFile();
        await splitDocument(vueconf.value);
        //const data = await createEmbeddings(vueInfoChunks.value);
        //await supabase.from('vueconf_docs').insert(data);
      });
    return {
      query,
      loading,
      matches,
      response,
      handleSubmit
    };
  }
};
</script>

<style scoped>
.container{
  display: flex;
  flex-direction: column;
  align-items: center;
}

label{
  display: block;
}

.title {
  font-size: 5em;
  margin: 0;
}
.submit_buttom {
  margin-top: 1em;
  padding: 0.5em 1em;
  background-color: #35485E;
  color: white;
  border: none;
  border-radius: 0.5em;
  cursor: pointer;
}

input {
  background-color: #fff;
  border: 1px solid #ccc;
  border-radius: 4px;
  padding: 8px 12px;
  width: 250px;
  box-sizing: border-box;
  transition: box-shadow 0.3s, border-color 0.3s;
  color: #000;
}

.subtitle {
  font-size: 2em;
  margin-bottom: 1em;
  color: #35485E;
}
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-
}
</style>