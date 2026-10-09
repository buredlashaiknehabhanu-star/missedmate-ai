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
    // State Management
let messageCount = 0;
let tasks = []; // Array to store extracted task objects
let lastSummary = "";

const keywords = ["deadline", "submit", "submission", "upload", "bring", "prepare", "complete", "finish", "attend", "meeting", "tomorrow", "today", "urgent", "remember", "send", "register", "assignment", "report", "exam", "test", "project", "required", "must", "before", "by"];

// Regex pattern to look for dates like "Oct 15", "15th Oct", "October 15th"
const datePattern = /(?:\b\d{1,2}(?:st|nd|rd|th)?\s+(?:Jan|Feb|Mar|Apr|May|Jun|Jul|Aug|Sep|Oct|Nov|Dec)[a-z]*)|(?:\b(?:Jan|Feb|Mar|Apr|May|Jun|Jul|Aug|Sep|Oct|Nov|Dec)[a-z]*\s+\d{1,2}(?:st|nd|rd|th)?)/i;

function updateStats() {
    document.getElementById("total").textContent = messageCount;
    
    const activeTasks = tasks.filter(t => !t.done).length;
    document.getElementById("count").textContent = tasks.length;
    document.getElementById("pending").textContent = activeTasks;
}

function analyze() {
    const raw = document.getElementById("message").value.trim();
    if (!raw) {
        alert("Please paste a message first!");
        return;
    }

    messageCount++;
    
    // Split the message into individual sentences
    const sentences = raw.split(/[.!?\n]+/).map(s => s.trim()).filter(s => s.length > 0);
    
    // Filter sentences that contain any of our target keywords
    const importantSentences = sentences.filter(sentence => {
        return keywords.some(word => {
            const regex = new RegExp("\\b" + word + "\\b", "i");
            return regex.test(sentence);
        });
    });

    // Take up to 3 key sentences to construct a quick TL;DR summary
    const chosen = importantSentences.length ? importantSentences.slice(0, 3) : sentences.slice(0, 2);
    lastSummary = chosen.join(" ") || "No high-priority action text found, but message received.";
    
    // Display the summary text in the UI
    document.getElementById("summary").textContent = lastSummary;

    // Scan the important sentences to dynamically build task lists
    importantSentences.forEach((sentence, index) => {
        const dateMatch = sentence.match(datePattern);
        const extractedDate = dateMatch ? dateMatch[0] : "No date mentioned";
        
        // Add unique task object to our array
        tasks.push({
            id: Date.now() + index,
            text: sentence,
            date: extractedDate,
            done: false
        });
    });

    renderTasks();
    updateStats();
}

// Render the action lists visually onto the page
function renderTasks() {
    const actionListContainer = document.getElementById("dates"); // Targets your lists display element
    actionListContainer.innerHTML = ""; // Clear out previous items

    if (tasks.length === 0) {
        actionListContainer.innerHTML = "<p style='color: #64748b;'>No tasks discovered yet.</p>";
        return;
    }

    tasks.forEach(task => {
        const item = document.createElement("div");
        item.className = "task-item";
        item.style.cssText = `
            display: flex; 
            justify-content: space-between; 
            align-items: center; 
            background: #1e293b; 
            padding: 10px; 
            margin-bottom: 8px; 
            border-radius: 6px;
            border-left: 4px solid ${task.done ? '#10b981' : '#f59e0b'};
        `;

        item.innerHTML = `
            <div style="flex: 1; margin-right: 10px; text-decoration: ${task.done ? 'line-through' : 'none'}; color: ${task.done ? '#64748b' : '#f8fafc'};">
                <strong>[${task.date}]</strong> ${task.text}
            </div>
            <button onclick="toggleTask(${task.id})" style="background: ${task.done ? '#065f46' : '#3b82f6'}; color: white; padding: 4px 8px; font-size: 12px; border-radius: 4px; border: none; cursor: pointer;">
                ${task.done ? '✓ Done' : 'Complete'}
            </button>
        `;
        actionListContainer.appendChild(item);
    });
}

function toggleTask(id) {
    const task = tasks.find(t => t.id === id);
    if (task) {
        task.done = !task.done;
        renderTasks();
        updateStats();
    }
}

function clearAll() {
    document.getElementById("message").value = "";
}

function sample() {
    document.getElementById("message").value = "CRITICAL UPDATE:\nThe final project submission deadline is set for Oct 15th via the portal. Make sure to upload your functional GitHub link before midnight. Late projects drop two full grades. Also, remember to attend the evaluation seminar this Friday morning.";
}

function downloadSummary() {
    if (!lastSummary) {
        alert("Analyze a message to generate a summary first!");
        return;
    }
    alert("📥 Downloading Summary:\n\n" + lastSummary);
}
}

runLocalAIAnalysis().catch(err => console.error("Error running local AI:", err));
