Below you’ll find **conceptual, minimal code examples** demonstrating how you might implement each of the listed prompt engineering techniques. These examples use the [OpenAI Python library](https://github.com/openai/openai-python) for simplicity, but you can adapt them to other frameworks (e.g., LangChain, Hugging Face Transformers, etc.). 

> **Note:** Some techniques (like RAG, ReAct, Tree-of-Thoughts) can get quite involved and often require more advanced pipelines or external tools. The snippets below illustrate only **basic** patterns to help you get started.

---

## 1. Zero-Shot Prompting

**Goal**: Ask a question or request an action without providing any examples or context.

```python
import openai

openai.api_key = "YOUR_API_KEY"

prompt = "Explain the theory of relativity in simple terms."

response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[
        {"role": "user", "content": prompt}
    ]
)

print(response.choices[0].message.content)
```

**Explanation**:  
- You simply provide the user query with no prior examples.  
- The model infers context from its own training.

---

## 2. Few-Shot Prompting

**Goal**: Provide a small set of examples (Q&A or instructions) to guide the model’s response.

```python
import openai

openai.api_key = "YOUR_API_KEY"

few_shot_prompt = """
You are an expert translator. Translate the following English sentences into French.

Example 1:
Input: "Hello, how are you?"
Output: "Bonjour, comment allez-vous?"

Example 2:
Input: "I love programming."
Output: "J'aime programmer."

Now translate this sentence:
Input: "What time is it?"
Output:
"""

response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[
        {"role": "user", "content": few_shot_prompt}
    ]
)

print(response.choices[0].message.content)
```

**Explanation**:  
- You include 1–3 (or more) examples to show the model **how** to respond.  
- This improves accuracy for tasks requiring a specific style or format.

---

## 3. Chain-of-Thought Prompting

**Goal**: Encourage the model to reason step by step. You can ask it to show or use a hidden chain-of-thought. Below is a **simple** illustration.  

> **Important:** If you are using chain-of-thought for internal reasoning, consider keeping the detailed reasoning hidden in the final user-facing response. This example shows an explicit chain-of-thought for demonstration.

```python
import openai

openai.api_key = "YOUR_API_KEY"

chain_of_thought_prompt = """
Solve the following math problem step by step:
Question: A train travels 60 miles per hour. How many miles will it travel in 2.5 hours?

Let's break down the steps carefully:
"""

response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[
        {"role": "user", "content": chain_of_thought_prompt}
    ]
)

print(response.choices[0].message.content)
```

**Explanation**:  
- By instructing “step by step,” you encourage the model to provide intermediate reasoning.  
- This can lead to more accurate and interpretable answers.

---

## 4. Program-Aided Language (PAL)

**Goal**: Have the model generate code (e.g., Python) to solve a problem, then execute that code to get the final answer.

```python
import openai
import subprocess
import tempfile

openai.api_key = "YOUR_API_KEY"

pal_prompt = """
You are given a math problem. Generate Python code that computes the answer, then I'll run the code.

Question: What is the sum of the squares of the first 10 natural numbers?
Answer in Python code only.
"""

response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[
        {"role": "user", "content": pal_prompt}
    ]
)

code_snippet = response.choices[0].message.content
print("Generated Code:\n", code_snippet)

# Optionally, execute the code if it's valid Python:
try:
    with tempfile.NamedTemporaryFile(mode='w', delete=False, suffix='.py') as f:
        f.write(code_snippet)
        temp_filename = f.name

    result = subprocess.check_output(["python", temp_filename], universal_newlines=True)
    print("Execution Result:\n", result)
except Exception as e:
    print("Error running code:", e)
```

**Explanation**:  
- The model writes Python code that solves the problem.  
- You execute the code to get the final answer, reducing the risk of arithmetic or logical mistakes in the direct text output.

---

## 5. Retrieval-Augmented Generation (RAG)

**Goal**: Combine external knowledge (retrieval) with the model’s generative capabilities. Typically done via a vector database or search API. Below is a **conceptual** example using a fictional `get_relevant_texts()` function to retrieve documents:

```python
import openai

openai.api_key = "YOUR_API_KEY"

def get_relevant_texts(query):
    # Placeholder for a real retrieval step (e.g., vector database search)
    # Return top documents as a single string or multiple chunks
    return "Document 1 content ... Document 2 content ..."

user_query = "Explain the significance of the Eiffel Tower in world history."

# 1. Retrieve relevant documents
retrieved_docs = get_relevant_texts(user_query)

# 2. Combine retrieval with user query
rag_prompt = f"""
Use the following retrieved documents to answer the question accurately.

Documents:
{retrieved_docs}

Question: {user_query}

Answer:
"""

response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[
        {"role": "system", "content": "You are a helpful research assistant."},
        {"role": "user", "content": rag_prompt}
    ]
)

print(response.choices[0].message.content)
```

**Explanation**:  
- You first **retrieve** relevant data from an external source.  
- The model then uses that data to ground its answer, improving factual correctness.

---

## 6. ReAct (Reason + Act)

**Goal**: The model alternates between reasoning steps (“Thought”) and actions (like searching or calculating) in an interactive loop. Below is a simplified example that shows how you might format the prompt; a real implementation would parse and execute “Action” steps programmatically.

```python
import openai

openai.api_key = "YOUR_API_KEY"

react_prompt = """
You are a problem-solving AI. When faced with a query, first think aloud (Thought), then propose an action (Action), then provide the result (Observation), and repeat as needed. Finally, provide the answer (Answer).

Query: What is the capital of France?

Thought 1: I should recall that the capital of France is Paris.
Action 1: No external action needed.
Observation 1: (No external data required)
Answer: Paris
"""

response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[
        {"role": "user", "content": react_prompt}
    ]
)

print(response.choices[0].message.content)
```

**Explanation**:  
- In ReAct, the model “thinks” (Thought) about the question, decides what to do (Action), and interprets results (Observation).  
- The final “Answer” is produced after these steps.

---

## 7. Reflection

**Goal**: After producing an initial answer, the model reflects on its own output and revises it if needed.

```python
import openai

openai.api_key = "YOUR_API_KEY"

initial_prompt = "Explain quantum mechanics in one sentence."

# 1. Get initial answer
response1 = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[{"role": "user", "content": initial_prompt}]
)

initial_answer = response1.choices[0].message.content
print("Initial Answer:\n", initial_answer)

# 2. Ask the model to reflect on its own answer
reflection_prompt = f"""
Your initial answer was:
{initial_answer}

Reflect on this answer and see if it is accurate, clear, or needs improvement. Provide a revised answer if necessary.
"""

response2 = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[{"role": "user", "content": reflection_prompt}]
)

revised_answer = response2.choices[0].message.content
print("\nRevised Answer:\n", revised_answer)
```

**Explanation**:  
- You gather an initial answer, then prompt the model to **review** and **improve** it.  
- This can catch mistakes or add clarity.

---

## 8. Tree-of-Thoughts (ToT)

**Goal**: Explore multiple reasoning paths (“branches”) before finalizing an answer. Full ToT can be complex; below is a **very simplified** version illustrating how you might structure multiple calls to the model.

```python
import openai

openai.api_key = "YOUR_API_KEY"

question = "What's the best strategy to solve a 4x4 Rubik's Cube quickly?"

# 1. Generate multiple solution "branches"
branch_prompt = f"""
Generate 3 different approaches or solution paths to answer the question:
'{question}'
"""

branch_response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[{"role": "user", "content": branch_prompt}]
)

branches = branch_response.choices[0].message.content.split("\n")

# 2. Evaluate each branch
evaluations = []
for i, branch in enumerate(branches):
    eval_prompt = f"""
Question: {question}
Proposed solution path: {branch}

Evaluate this approach for clarity, feasibility, and completeness.
"""
    eval_response = openai.ChatCompletion.create(
        model="gpt-3.5-turbo",
        messages=[{"role": "user", "content": eval_prompt}]
    )
    evaluations.append((branch, eval_response.choices[0].message.content))

# 3. Decide on the best approach
final_prompt = "Based on the evaluations, choose the best solution and present it clearly."
decision_input = "\n\n".join([f"Branch: {b}\nEvaluation: {e}" for b, e in evaluations])

response_final = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[
        {"role": "user", "content": decision_input},
        {"role": "user", "content": final_prompt}
    ]
)

print("Final Chosen Strategy:\n", response_final.choices[0].message.content)
```

**Explanation**:  
- **Step 1**: Generate multiple possible solution paths.  
- **Step 2**: Evaluate each path.  
- **Step 3**: Choose the best path or combine them.  
- This fosters **divergent thinking** before converging on a final answer.

---

## 9. Self-Consistency

**Goal**: Sample multiple outputs for the **same** query, then pick the most common or most consistent result.

```python
import openai
from collections import Counter

openai.api_key = "YOUR_API_KEY"

question = "What is the capital of Germany?"

num_samples = 5
answers = []

for _ in range(num_samples):
    response = openai.ChatCompletion.create(
        model="gpt-3.5-turbo",
        temperature=1.0,  # higher temperature for varied outputs
        messages=[
            {"role": "user", "content": question}
        ]
    )
    answer = response.choices[0].message.content.strip()
    answers.append(answer)

print("All sampled answers:\n", answers)

# Pick the most common answer
counter = Counter(answers)
most_common_answer, count = counter.most_common(1)[0]

print("\nMost Consistent Answer:", most_common_answer)
```

**Explanation**:  
- You ask the model multiple times (with some randomness).  
- Then, you select the **most frequent** or best-supported answer.  
- This can improve reliability on factual questions.

---

## 10. Least-to-Most Prompting

**Goal**: Break a complex question into simpler sub-questions, solve them step by step, then combine answers.

```python
import openai

openai.api_key = "YOUR_API_KEY"

complex_question = "Explain how to design a rocket engine that can reach low Earth orbit."

least_to_most_prompt = f"""
Break down the question into smaller sub-questions in order of difficulty. 
Then answer each sub-question, building up to the final answer.

Question: {complex_question}
"""

response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[{"role": "user", "content": least_to_most_prompt}]
)

print(response.choices[0].message.content)
```

**Explanation**:  
- The model first identifies simpler sub-problems.  
- It solves each incrementally, then integrates them into the final solution.

---

## 11. Directional Stimulus

**Goal**: Nudge the model in a specific direction or style (e.g., formal, humorous, concise).

```python
import openai

openai.api_key = "YOUR_API_KEY"

directional_prompt = """
Write a short paragraph about the importance of sleep. 
Use a lighthearted, humorous tone.
"""

response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[{"role": "user", "content": directional_prompt}]
)

print(response.choices[0].message.content)
```

**Explanation**:  
- You explicitly instruct the model to adopt a certain **tone**, **style**, or **perspective**.  
- This guides the style of the response.

---

## 12. Generated Knowledge

**Goal**: Have the model generate relevant background info or context **before** answering the main query.

```python
import openai

openai.api_key = "YOUR_API_KEY"

step1_prompt = """
Generate key background information about the history of blockchain technology. 
Do not answer the user's question yet; just provide the relevant context.
"""

response1 = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[{"role": "user", "content": step1_prompt}]
)

background_info = response1.choices[0].message.content
print("Background Info:\n", background_info)

# Now use the background info to answer the final question
final_question = "How could blockchain impact the future of digital identity?"

step2_prompt = f"""
Using the following background info, answer the question:
Background Info: {background_info}
Question: {final_question}
"""

response2 = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[{"role": "user", "content": step2_prompt}]
)

print("\nFinal Answer:\n", response2.choices[0].message.content)
```

**Explanation**:  
- **Step 1**: The model generates relevant knowledge or context.  
- **Step 2**: You feed that context back into the model to produce the final answer.  
- This structure can improve the quality and depth of the final response.

---

# Wrap-Up

These examples illustrate **basic** patterns for each prompt engineering technique:

- **Zero-Shot**: Minimal context, rely on the model’s training.  
- **Few-Shot**: Provide examples to guide the style or format.  
- **Chain-of-Thought**: Encourage step-by-step reasoning.  
- **Program-Aided (PAL)**: Generate and execute code for reliable results.  
- **RAG**: Integrate external documents or databases.  
- **ReAct**: Alternate between “Thought” and “Action” steps.  
- **Reflection**: Let the model revise its own answer.  
- **Tree-of-Thoughts**: Explore multiple solution paths, then converge.  
- **Self-Consistency**: Sample multiple times and pick the best/most common.  
- **Least-to-Most**: Decompose complex tasks into smaller steps.  
- **Directional Stimulus**: Nudge the model toward a certain style or angle.  
- **Generated Knowledge**: Generate context or background before answering.

Feel free to **mix and match** these techniques and expand the code to suit your specific use cases!
