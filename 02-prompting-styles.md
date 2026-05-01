## 1. Zero-Shot Prompting
This is the simplest form. You provide an instruction with **zero** examples. In production, this is used for simple tasks like summarization or basic sentiment analysis where the model's internal knowledge is sufficient.

### JS Example
```javascript
import OpenAI from "openai";
const openai = new OpenAI();

const response = await openai.chat.completions.create({
  model: "gpt-4o",
  messages: [
    { 
      role: "system", 
      content: "You are a professional editor. Summarize the following text in 3 bullet points." 
    },
    { 
      role: "user", 
      content: "GenAI is transforming the way we write code..." 
    }
  ],
});
```

---

## 2. Few-Shot Prompting
Industry standard for **format control**. You provide a few (usually 3–5) pairs of input/output examples to "show, not just tell" the model how to behave. This is essential for ensuring your JSON outputs or specific writing styles remain consistent.



### JS Example
```javascript
const response = await openai.chat.completions.create({
  model: "gpt-4o",
  messages: [
    { role: "system", content: "Convert user requests into a valid JSON schema." },
    // Example 1
    { role: "user", content: "I need a user named John aged 25" },
    { role: "assistant", content: '{"name": "John", "age": 25}' },
    // Example 2
    { role: "user", content: "Create a product called Laptop for 1200 dollars" },
    { role: "assistant", content: '{"item": "Laptop", "price": 1200}' },
    // The actual task
    { role: "user", content: "Add a car called Tesla for 60000" }
  ],
});
```

---

## 3. Chain-of-Thought (CoT)
Used for **reasoning**. By adding the instruction *"Let’s think step by step,"* or providing an example that shows the reasoning process, you significantly reduce "hallucinations" in math, logic, or complex coding tasks.



### JS Example (Zero-Shot CoT)
```javascript
const response = await openai.chat.completions.create({
  model: "gpt-4o",
  messages: [
    { 
      role: "system", 
      content: "You are a logic assistant. Always think step-by-step before providing the final answer." 
    },
    { 
      role: "user", 
      content: "If I have 3 apples and give 1 to a friend, then buy 2 more, how many do I have? Explain each step." 
    }
  ],
});
```

---

## 4. ReAct (Reason + Act)
This is the foundation of **AI Agents**. The model doesn't just "talk"; it follows a loop: **Thought → Action → Observation**. It thinks about what to do, calls a tool (like a database or search), sees the result, and then decides the next move.

### JS Example (Using Function Calling)
In industry, we implement ReAct using "Tools."

```javascript
const tools = [{
  type: "function",
  function: {
    name: "get_weather",
    parameters: { type: "object", properties: { location: { type: "string" } } }
  }
}];

const response = await openai.chat.completions.create({
  model: "gpt-4o",
  messages: [{ role: "user", content: "What's the weather in Bengaluru?" }],
  tools: tools,
});

// The model will return a "tool_call" instead of a string, 
// signaling it wants to 'Act'.
```

---

## Industry Best Practices
*   **Use Delimiters:** Use `###`, `"""`, or XML tags like `<context></context>` to separate instructions from the data.
*   **The "System" Role:** Always put your core rules and "identity" in the `system` message, not the `user` message.
*   **Output Consistency:** Always specify the format (e.g., "Output only valid JSON") to make your JS backend's `JSON.parse()` calls reliable.

---

## Chain of Thought Prompting Deep Dive
### The "Think-Do-Evaluate" Pattern
This pattern is used when accuracy is more important than speed. It follows three distinct steps:
1.  **Thinking:** Breaking the problem into sub-tasks.
2.  **Evaluating:** Checking the logic of the sub-tasks for errors.
3.  **Final Response:** Delivering the verified result to the user.



### Detailed JavaScript Example (OpenAI SDK)

This example shows how to prompt the model to use a structured "Inner Monologue." We will use a complex logic puzzle as the input.

```javascript
import OpenAI from "openai";

const openai = new OpenAI();

async function runStructuredCoT() {
  const systemPrompt = `
    You are a logical reasoning engine. For every problem, follow these exact steps:
    
    1. <THINK>: Break the problem down. Identify variables and constraints.
    2. <EVALUATE>: Review your reasoning in step 1. Look for logical fallacies or math errors.
    3. <RESPONSE>: Provide the final, verified answer.
    
    Always use the XML tags <THINK>, <EVALUATE>, and <RESPONSE>.
  `;

  const userProblem = `
    A farmer has 17 sheep. All but 9 run away. 
    The farmer then buys 3 more sheep. 
    How many sheep does the farmer have now?
  `;

  try {
    const completion = await openai.chat.completions.create({
      model: "gpt-4o", // High-reasoning model
      messages: [
        { role: "system", content: systemPrompt },
        { role: "user", content: userProblem }
      ],
      temperature: 0.2 // Lower temperature for more deterministic/logical output
    });

    const output = completion.choices[0].message.content;
    console.log("--- Model Output ---");
    console.log(output);

    // Industry Tip: In a real app, you can use RegEx to hide the <THINK> tags 
    // from the end-user and only show the <RESPONSE> content.
    const finalAnswer = output.split("<RESPONSE>")[1]?.replace("</RESPONSE>", "").trim();
    console.log("\n--- Extracted Final Answer for UI ---");
    console.log(finalAnswer);

  } catch (error) {
    console.error("Error:", error);
  }
}

runStructuredCoT();
```

---


### Why this is "Industry Standard"
1.  **Debugging:** If the model gives a wrong answer, you can look at the `<THINK>` logs to see exactly where the logic failed.
2.  **Consistency:** By forcing an `<EVALUATE>` step, you utilize the model's "Self-Consistency" property, which significantly improves performance on benchmarks like GSM8K (math word problems).
3.  **Parsing:** Using tags like `<RESPONSE>` allows your JavaScript backend to easily strip out the "messy" thinking process before sending the clean answer to your frontend Chat UI.

