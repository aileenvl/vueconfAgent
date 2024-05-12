<template>
  <div>
    <h1>Vue Conf</h1>
    <p v-if="loading">Loading...</p>
    <ul v-else>
      <li v-for="(match, index) in matches" :key="index">
        {{ match.content }}
      </li>
    </ul>
    <form @submit.prevent="handleSubmit">
      <label for="query">Ask a question:</label>
      <input id="query" v-model="query" type="text" name="query">
      <button type="submit">Submit</button>
    </form>
    <p v-if="response">{{ response }}</p>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue';
import MistralClient from '@mistralai/mistralai';
import { GenerativeModel } from "@google/generative-ai";
import { RecursiveCharacterTextSplitter } from "langchain/text_splitter";
import { createClient } from "@supabase/supabase-js";
// import { tools, getPaymentDate, getPaymentStatus } from './utils/tools';

const mistralApiKey = import.meta.env.VITE_MISTRAL_API_KEY;
const geminiApiKey = import.meta.env.VITE_GEMINI_API_KEY;
const supabaseUrl = import.meta.env.VITE_SUPABASE_URL;
const supabaseKey = import.meta.env.VITE_SUPABASE_API_KEY;

export default {
  setup() {
    const client = new GenerativeModel(geminiApiKey);
    const supabase = createClient(supabaseUrl, supabaseKey);
    const chatResponse = ref(null);
    const vueInfoChunks = ref(null);
    const vueconf = ref('');
    const query = ref('');
    const loading = ref(false);
    const matches = ref([]);
    const response = ref('');
  
  
  async function loadTextFile() {
    try {
      const module = await import('./assets/vueconfData.txt');
      vueconf.value = await fetch(module.default).then(res => res.text());
    } catch (error) {
      console.error('Error loading text file:', error);
    }
  }

  async function splitDocument(text) {
    console.log('Splitting document');
    try {
      const splitter = new RecursiveCharacterTextSplitter({
        chunkSize: 250,
        chunkOverlap: 40
      });
      const output = await splitter.createDocuments([text]);
      const stringFormat = output.map((chunk) => chunk.pageContent);
      vueInfoChunks.value = stringFormat;
      console.log(vueInfoChunks.value);
    } catch (error) {
      console.error(error);
    }
  }

  async function createEmbeddings(chunks) {
    try {
      const embeddings = await client.embedContent({
        model: 'embedding-001',
        input: chunks
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

    const embedding = await createEmbedding(input);
    const context = await retrieveMatches(embedding);
    console.log('Context:', context)
    response.value = await generateChatResponse(context, input);

    loading.value = false;
  };

const input = "Who is the person that is givin the Nuxt 3 talk?";
// 2. Creating an embedding of the input
async function createEmbedding(input) {
  const embeddingResponse = await client.embedContent({
    model: 'embedding-001',
    input: [input]
  });
  return embeddingResponse.data[0].embedding;
}

// 3. Retrieving similar embeddings / text chunks (aka "context")
async function retrieveMatches(embedding) {
  const { data } = await supabase.rpc('match_vueconf_docs', {
    query_embedding: embedding,
    match_threshold: 0.58,
    match_count: 8 
  });
  if (data) {
    return  data.map(chunk => chunk.content).join(" ");;
  } else {
    return [];
  }
}

// 4. Combining the input and the context in a prompt 
// and using the chat API to generate a response 
async function generateChatResponse(context, query) {
  const response = await client.chat({
        model: 'gemini-pro',
        messages: [{
            role: 'user',
            content: `Handbook context: ${context} - Question: ${query}`
        }]
    });
    return response.choices[0].message.content;
}


   

    async function getChatResponse() {
      try {
        const response = await client.chatStream({
          model: 'mistral-tiny',
          messages: [{ role: 'user', content: 'What is the best French cheese?' }],
          temperature: 0.5
        });
        chatResponse.value = response;
      } catch (error) {
        console.error(error);
      }
    }

   
    // const handleSubmit = async () => {
    //   loading.value = true;
    //   response.value = await agent(query.value);
    //   loading.value = false;
    // };

    async function agent(query) {
      const messages = [{ role: "user", content: query }];

      for (let i = 0; i < 5; i++) {
        const response = await client.chat( {
            model: 'mistral-large-latest',
            messages: messages,
            tools: tools,
            verbose: true
        });
        
        messages.push(response.choices[0].message);

        // if the finishReason is 'stop', then simply return the 
        // response from the assistant
        if (response.choices[0].finish_reason === 'stop') {
            return response.choices[0].message.content;
        } else if (response.choices[0].finish_reason === 'tool_calls') {
            const functionObject = response.choices[0].message.tool_calls[0].function;
            const functionName = functionObject.name;
            const functionArgs = JSON.parse(functionObject.arguments);
            const functionResponse = availableFunctions[functionName](functionArgs);
            messages.push({
                role: 'tool',
                name: functionName,
                content: functionResponse 
            });
        }
    }
  }
  
      onMounted(async () => {
        //await loadTextFile();
        //await splitDocument(vueconf.value);
        //const data = await createEmbeddings(vueInfoChunks.value);
        //await supabase.from('vueconf_docs').insert(data);
        //await getChatResponse();
        // 
        console.log('Data inserted');
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