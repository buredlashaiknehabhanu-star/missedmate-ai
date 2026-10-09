import { pipeline } from '@xenova/transformers';

// 1. Define a mock chat history representing a chaotic group conversation
const chatHistory = `
[09:00] Alice: Hey team, we need to finalize the launch plan today.
[09:02] Bob: I'm still reviewing the marketing assets. The graphics look a bit off.
[09:05] Charlie: @Dave can you check the database cluster? It threw an out-of-memory error at midnight.
[09:15] Alice: @Bob let's fix the graphics by 3 PM. We can't delay the launch.
[09:18] Dave: @Charlie I checked the server. I restarted the node, but we need to upgrade the RAM by Friday to prevent it from happening again.
[09:22] Bob: @Alice sure, I will update the asset folder with new high-res versions by 2:30 PM.
[09:30] Charlie: Great, so Dave is upgrading RAM by Friday, and Bob is uploading new graphics by 2:30 PM today. Let's sync at 4 PM.
`;

async function runLocalAIAnalysis() {
    console.log("⏳ Initializing local AI pipeline... (This may take a moment on the first run to download the model)");
    
    // 2. Load a lightweight, local-friendly text generation model (runs via WebAssembly locally)
    const generator = await pipeline('text-generation', 'Xenova/Qwen1.5-0.5B-Chat');

    // 3. Craft a structured prompt instructing the local LLM to extract the exact requirements
    const prompt = `
You are a local-first privacy-focused assistant. Analyze the chat log below and extract a structured summary.
Strictly provide:
1. A brief 2-sentence summary of what happened.
2. A prioritized list of action items/decisions with deadlines and assignees.

Chat Log:
${chatHistory}

Analysis:
`;

    console.log("🤖 Running local processing (0% data leaves your device)...");
    
    // 4. Execute the model locally
    const output = await generator(prompt, {
        max_new_tokens: 250,
        temperature: 0.2, // Low temperature for factual, consistent extraction
        repetition_penalty: 1.1
    });

    console.log("\n=================== LOCAL AI REPORT ===================");
    console.log(output[0].generated_text.replace(prompt, '').trim());
    console.log("=======================================================");
}

runLocalAIAnalysis().catch(err => console.error("Error running local AI:", err));