The OpenAI API is **stateless**. This means the model does not "remember" what you said 10 seconds ago unless you send the entire conversation back with every new request.

In a production environment (like the JS backend), you manage this by maintaining an array of message objects that grows as the conversation progresses.

### The Message Loop Logic
To keep context, your messages array must always follow this structure:
1.  **System Message:** The "identity" of the bot. Sent only once at the start.
2.  **User/Assistant Pairs:** Every time the user speaks, you `push` their message. Every time the AI responds, you `push` that too.

---

### Implementation: The "Chat Loop" in JavaScript


```javascript
import OpenAI from "openai";
import readline from "readline";

const openai = new OpenAI({ apiKey: "YOUR_API_KEY" });

// 1. Initialize history with the 'System' instruction
let messages = [
  { 
    role: "system", 
    content: "You are a helpful GenAI assistant. Keep answers concise." 
  }
];

const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout
});

async function chat() {
  rl.question("User: ", async (userInput) => {
    if (userInput.toLowerCase() === "exit") return rl.close();

    // 2. Add the User's new message to the context
    messages.push({ role: "user", content: userInput });

    try {
      const response = await openai.chat.completions.create({
        model: "gpt-4o",
        messages: messages, // Send the ENTIRE history
      });

      const assistantReply = response.choices[0].message.content;
      console.log(`\nAI: ${assistantReply}\n`);

      // 3. Add the AI's response to the context so it remembers it next time
      messages.push({ role: "assistant", content: assistantReply });

    } catch (error) {
      console.error("Error calling OpenAI:", error.message);
    }

    chat(); // Repeat the loop
  });
}

console.log("--- AI Chat Started (Type 'exit' to stop) ---");
chat();
```

---


### Industry Challenges & Solutions
When building this for a real app, you will run into two main issues:

1.  **The Token Limit:** Eventually, the `messages` array becomes too large for the model's "Context Window."
    *   *Solution:* **Sliding Window.** You remove the oldest messages (after the System prompt) once the array hits a certain length.
2.  **Cost:** More tokens in the history = more money per request.
    *   *Solution:* **Summarization.** Every 10 messages, ask the model to summarize the previous conversation into one "Summary" message to save space.
