# AutoE2E

Source code and benchmark subjects for "AutoE2E: Feature-Driven End-To-End Test Generation."

![AutoE2E Workflow](./workflow.png)

## Requirements

Install the required packages using the following command:

```bash
pip install -r requirements.txt
```

## Usage

Before running the project, you need to create and configure a `.env` file with the required environment variables.

### Step 1: Create the .env file

Create your `.env` file from the provided template:

```bash
cp .env.example .env
```

This file is required before running `main.py`. Without it, you will encounter runtime errors about missing environment variables.

### Step 2: Configure Environment Variables

Open the newly created `.env` file and populate it with your own API keys and configuration values. The required variables are:

1. `APP_NAME`: The name of the application you want to generate E2E test cases for. This needs to match one of the configs in `./configs` folder.

2. `ANTHROPIC_API_KEY`: The API key for the Anthropic platform used for primary model reasoning.

   - Sign up at [Anthropic Console](https://console.anthropic.com/)
   - Navigate to the API Keys section
   - Click "Create Key" and give it a name
   - Copy the generated API key to your `.env` file
   - **Important**: Verify that the model version in `autoe2e/llm_api_call.py` is the latest version, otherwise the application will crash
   - **Optional**: You can switch from Anthropic's Sonnet to OpenAI's GPT-4 by changing `sonnet_chain` to `gpt4o_chain` in `autoe2e/infer_utils.py` (requires OpenAI API key)

3. `OPENAI_API_KEY`: The API key for OpenAI platform used for generating feature embeddings.

   - Sign up or log in at [OpenAI Platform](https://platform.openai.com/)
   - Navigate to [API Keys](https://platform.openai.com/api-keys)
   - Click "Create new secret key" and give it a name
   - Copy the generated API key to your `.env` file (you won't be able to see it again)
   - **Note**: This is required even if using Anthropic for reasoning, as OpenAI's embedding model is used to create feature embeddings for database storage

4. `ATLAS_URI`: The MongoDB Atlas URI for storing the Action-Feature Database (AFD) and Feature Database (FD).

### Step 3: Run the Project

Once your `.env` file is configured with the appropriate values, you can run the project using the following command:

```bash
python main.py
```

## LLM Prompts

The prompts used for different parts of our workflow is available in `./autoe2e/prompts.py` file. We use the following prompt for context extraction:

> Given the provided information about a webpage, your task is to provide a brief and abstract description of the webpage's primary purpose or function.
> Output Guidelines:
>
> - Brevity: Keep the description concise (aim for 1-2 sentences).
> - Abstraction: Avoid specific details or variable names. Use general terms to describe the content and function. (Example: Instead of "a page showing results for searching for a TV," say "a page displaying search results for a product query.")
> - Focus on Purpose: Prioritize describing the main intent of the page. What is it designed for the user to do or learn?
> - No Extra Explanations: Just provide the context. Avoid adding commentary or assumptions.

and the following for feature extraction:

> Given a webpage's purpose and content (webpage_context), the outerHTML of an action element (action_element), and optionally the user's last action that led to this state, your task is to infer the most likely functionalities associated with that action element.
> These functionalities should be user-centric actions that produce measurable outcomes within the application, are testable through E2E testing, and are essential to the presence of the action element.
> Output Format:
> Your is enclosed in two tags:
> \<Reasoning>:
>
> - An enumerated list of at most five functionalities potentially connected to the element.
> - For each functionality, answer the following questions concisely:
>   1. Would developers write E2E test cases for this in the real world? It should be non-navigational, not menu-related, and not validation.
>   2. Is the functionality a final user goal in itself or is it always a step in doing something else?
>   3. Is this overly abstract/vague? If so, break it down into more testable sub-functionalities.
> - Avoid repeating the questions in your responses every time.
>   \<Response>:
> - A JSON array of objects, each containing:
>   - probability: (0.0 to 1.0) Likelihood of this functionality exists.
>   - feature: A concise description of the user action (e.g., "add item to cart").
> - Sorted by probability in descending order.
> - Parsable by `json.loads`.
> - Can be an empty array if no valid functionalities are found.

Furthermore, the baseline prompts are available in `./baseline-prompts.md`.

## Subjects

The subjects used in our evaluations are available in `./benchmark` folder. Furthermore, the server created for tracking the execution of features is available in `./benchmark/_log-server` folder. You need to have a `Redis` server installed and running to be able to use the server.

To run the server:

```bash
cd benchmark/_log-server

pip install -r requirements.txt

flask --app extract.py --debug run
```

### Server Endpoints

The server has the following endpoints:

1. `/start-evaluate/<app-name>`: Start the coverage evaluation for the given application.
2. `/end-evaluate`: End the coverage evaluation for the given application. It will return the coverage rate.

To test the server, you can run the `PetClinic` application located in `./benchmark/pet-clinic` and use the server.
