# Langchain Language Correctness Detector

This project implements a simple Langchain language correctness detector that detects grammatical errors, sentiment, aggressiveness, and provides solutions for the errors in the text.

- [Langchain Language Correctness Detector](#langchain-language-correctness-detector)
  - [Features](#features)
  - [Stack Used](#stack-used)
  - [Installation](#installation)
  - [Usage](#usage)
  - [Code Explanation](#code-explanation)
    - [Imports and Environment Setup](#imports-and-environment-setup)
    - [System Template and Schema Definition](#system-template-and-schema-definition)
    - [Prompt Template and Model Selection](#prompt-template-and-model-selection)
    - [Main Function](#main-function)
  - [Prompts Used for Detecting Correctness](#prompts-used-for-detecting-correctness)
  - [Examples](#examples)
    - [OpenAI](#openai)
    - [Gemini](#gemini)
  - [License](#license)
  - [Contributing](#contributing)
  - [Conclusion](#conclusion)


## Features

- Detects grammatical errors in the text.
- Analyzes the sentiment of the text.
- Measures the aggressiveness of the text.
- Provides solutions for the detected errors.

## Stack Used

- **Node.js**: JavaScript runtime environment.
- **TypeScript**: Typed superset of JavaScript.
- **Langchain**: Language processing library.
- **OpenAI API**: For language model capabilities.
- **Google Cloud**: For additional language processing services.

## Installation

1. Clone the repository:
    ```sh
    git clone https://github.com/xavidop/langchain-example.git
    cd langchain-example
    ```

2. Install the dependencies:
    ```sh
    yarn install
    ```

3. Create a `.env` file in the root directory and add your OpenAI API key and Google Application credentials:
    ```env
    OPENAI_API_KEY="your-openai-api-key"
    GOOGLE_APPLICATION_CREDENTIALS=credentials.json
    LLM_PROVIDER='OPENAI'
    ```

## Usage

1. Build the project:
    ```sh
    yarn run build
    ```

2. Start the application:
    ```sh
    yarn start
    ```

3. For development, you can use:
    ```sh
    yarn run dev
    ```
## Code Explanation

### Imports and Environment Setup
```typescript
import { ChatOpenAI, ChatOpenAICallOptions } from "@langchain/openai";
import { ChatVertexAI } from "@langchain/google-vertexai";
import { ChatPromptTemplate } from "@langchain/core/prompts";
import { z } from "zod";
import * as dotenv from "dotenv";

// Load environment variables from .env file
dotenv.config();
```
- **Imports:** The code imports necessary modules from Langchain, Zod for schema validation, and dotenv for environment variable management.
- **Environment Setup:** Loads environment variables from a `.env` file.

### System Template and Schema Definition

```typescript
const systemTemplate = "You are an expert in {language}, you have to detect grammar problems sentences";

const classificationSchema = z.object({
  sentiment: z.enum(["happy", "neutral", "sad", "angry", "frustrated"]).describe("The sentiment of the text"),
  aggressiveness: z.number().int().min(1).max(10).describe("How aggressive the text is on a scale from 1 to 10"),
  correctness: z.number().int().min(1).max(10).describe("How the sentence is correct grammatically on a scale from 1 to 10"),
  errors: z.array(z.string()).describe("The errors in the text. Specify the proper way to write the text and where it is wrong. Explain it in a human-readable way. Write each error in a separate string"),
  solution: z.string().describe("The solution to the errors in the text. Write the solution in {language}"),
  language: z.string().describe("The language the text is written in"),
});
```
- **System Template:** Defines a template for the system message, indicating the language and the task of detecting grammar problems.
- **Classification Schema:** Uses Zod to define a schema for the expected output, including sentiment, aggressiveness, correctness, errors, solution, and language.

### Prompt Template and Model Selection

```typescript
const promptTemplate = ChatPromptTemplate.fromMessages([
  ["system", systemTemplate],
  ["user", "{text}"],
]);

let model: any;
if (process.env.LLM_PROVIDER == "OPENAI") {
  model = new ChatOpenAI({ 
    model: "gpt-4",
    temperature: 0,
  });
} else {
  model = new ChatVertexAI({ 
    model: "gemini-1.5-pro-001",
    temperature: 0,
  });
}
```
- **Prompt Template:** Creates a prompt template using the system message and user input.
- **Model Selection:** Selects the language model based on the LLM_PROVIDER environment variable. It can either be OpenAI's GPT-4 or Google's Vertex AI.

### Main Function
```typescript
export const run = async () => {
  const llmWihStructuredOutput = model.withStructuredOutput(classificationSchema, {
    name: "extractor",
  });

  const chain = await promptTemplate.pipe(llmWihStructuredOutput);

  const result = await chain.invoke({ language: "Spanish", text: "Yo soy enfadado" });

  console.log({ result });
};

run();
```

- **Structured Output:** Configures the model to use the defined classification schema.
- **Pipeline:** Creates a pipeline by combining the prompt template and the structured output model.
- **Invocation:** Invokes the pipeline with a sample text in Spanish, and logs the result.

## Prompts Used for Detecting Correctness

The following prompts are used to detect the correctness of the text:

1. **Grammatical Errors**:
    ```plaintext
    "Please check the following text for grammatical errors: {text}"
    ```

2. **Sentiment Analysis**:
    ```plaintext
    "Analyze the sentiment of the following text: {text}"
    ```

3. **Aggressiveness Detection**:
    ```plaintext
    "Measure the aggressiveness of the following text: {text}"
    ```

4. **Error Solutions**:
    ```plaintext
    "Provide solutions for the errors found in the following text: {text}"
    ```

## Examples

This project can be used with different language models to detect language correctness. Here are some examples using OpenAI and Gemini models.

### OpenAI

With OpenAI's GPT-4 model, the system can detect grammatical errors, sentiment, and aggressiveness in the text.

```json
{ language: "Spanish", text: "Yo soy enfadado" }
```

```json
{
  result: {
    sentiment: 'angry',
    aggressiveness: 2,
    correctness: 7,
    errors: [
      "The correct form of the verb 'estar' should be used instead of 'ser' when expressing emotions or states."
    ],
    solution: 'Yo estoy enfadado',
    language: 'Spanish'
  }
}
```

### Gemini

With Google's Vertex AI Gemini model, the output is quite similar:

Input:
```json
{ language: "Spanish", text: "Yo soy enfadado" }
```

Output:
```json
{
  result: {
    sentiment: 'angry',
    aggressiveness: 1,
    correctness: 8,
    errors: [
      'The correct grammar is "estoy enfadado" because "ser" is used for permanent states and "estar" is used for temporary states. In this case, being angry is a temporary state.'
    ],
    solution: 'Estoy enfadado',
    language: 'Spanish'
  }
}
```

## License

This project is licensed under the Apache License, Version 2.0. See the [LICENSE](LICENSE) file for more details.

## Contributing

Contributions are welcome! Please open an issue or submit a pull request for any changes.

## Conclusion

This project demonstrates how to use Langchain to detect language correctness using different language models. By combining the system template, classification schema, prompt template, and language model, you can create a powerful language processing system. OpenAI and Gemini models provide accurate results for detecting grammatical errors, sentiment, and aggressiveness in the text.