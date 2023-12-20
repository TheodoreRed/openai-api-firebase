
---

# Setting Up OpenAI API with Firebase

Integrating the OpenAI API into your Firebase project involves several steps, from installing necessary packages to setting up environment variables and creating relevant routes and services. Follow these steps to get started:

## Prerequisites

- **OpenAI API Key**: Create an account with OpenAI. Navigate to the API keys tab and click "Create new secret key" to generate your API key.

## Steps for Setup

### 1) Install OpenAI Package
In your Firebase functions directory, install the OpenAI npm package:

```bash
npm install openai
```

### 2) Configure Local Environment Variables
Create a `.runtimeconfig.json` file in your functions directory with the following content:

```json
{
  "openai": {
    "key": "your_openai_api_key_here"
  },
  "mongodb": {
    "uri": "your_mongodb_uri_here"
  }
}
```

**Note**: Replace `your_openai_api_key_here` and `your_mongodb_uri_here` with your actual keys.

### 3) Run Firebase Emulator Locally
To make these environment variables available when running your Firebase functions locally, use:

```bash
npm run serve:dev
```

or

```bash
firebase emulators:start
```

### 4) Set OpenAI API Key in Firebase Functions Configuration
Use the Firebase CLI to set your OpenAI API key in the Firebase Functions configuration:

```bash
firebase functions:config:set openai.key="your_openai_api_key_here"
```

### 5) Create OpenAI/ChatGPT Router
Create an Express router to handle OpenAI/ChatGPT interactions:

```javascript
import express from 'express';
import * as functions from 'firebase-functions';
import OpenAI from 'openai';

const openaiRouter = express.Router();
const openai = new OpenAI(functions.config().openai.key);

openaiRouter.post('/generate-text', async (req, res) => {
    const prompt = req.body.prompt;

    try {
        const chatCompletion = await openai.chat.completions.create({
            messages: [
                {
                    role: "user",
                    content: prompt,
                },
            ],
            model: "gpt-3.5-turbo",
        });

        res.status(200).json(chatCompletion.choices[0].message.content);
    } catch (error) {
        console.error('Error with OpenAI:', error);
        res.status(500).send('Error generating text');
    }
});

export default openaiRouter;
```

### 6) Update Main Server File
In your main server file (e.g., `index.ts`), add the OpenAI router:

```javascript
import openaiRouter from './routes/openaiRouter';

app.use("/openai", openaiRouter);
```

### 7) Create OpenAI Service for Frontend Interaction
In your services folder, create a file (e.g., `openAiAPI.ts`) for frontend interaction with OpenAI:

```javascript
import axios from "axios";

const baseUrl: string =
  import.meta.env.VITE_APP_BASE_URL ??
  "NOT FOUND CHECK services FOLDER AND .env.local";

export const generateTextWithOpenAI = async (prompt: string) => {
try {
    const response = await axios.post(`${baseUrl}/openai/generate-text`, {
      prompt,
    });
    return response.data;
  } catch (error) {
    console.error("Error generating text with OpenAI:", error);
    throw error; // Rethrow the error for handling it in the calling component
  }
};
```

### 8) Use the OpenAI Service in Your React Application
Import and use `generateTextWithOpenAI` in your React components to interact with OpenAI.

## Security and Best Practices

- **Never Expose API Keys**: Keep your API keys secure. Do not push them to public repositories.
- **Error Handling**: Implement robust error handling in both your Firebase functions and frontend services.
- **Environment Variables**: Use environment variables for different stages (development, staging, production) to manage API keys and other sensitive data.
- **Monitor API Usage**: Regularly monitor your OpenAI API usage to avoid unexpected costs and rate limits.
- **Testing**: Thoroughly test the integration in a development or staging environment before deploying to production.

## Conclusion

By following these steps, you can successfully integrate OpenAI's API into your Firebase project, enabling powerful AI-driven features in your application.

---

