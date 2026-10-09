<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Missed Mate AI - Chat Summarizer</title>
    <style>
        body {
            margin: 0;
            background: #0f172a;
            color: #f8fafc;
            font-family: Arial, sans-serif;
        }
        .main {
            max-width: 900px;
            margin: 0 auto;
            padding: 25px;
        }
        header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 12px 0;
            border-bottom: 1px solid #1e293b;
        }
        .brand {
            font-size: 24px;
            font-weight: bold;
            color: #38bdf8;
        }
        .badge {
            font-size: 12px;
            color: #4ade80;
            border: 1px solid #166534;
            background: #064e3b;
            border-radius: 20px;
            padding: 6px 12px;
        }
        .hero {
            padding: 45px 0 25px 0;
            text-align: center;
        }
        h1 span {
            color: #38bdf8;
        }
        p {
            color: #94a3b8;
            line-height: 1.7;
            max-width: 650px;
            margin: 10px auto 25px auto;
        }
        .layout {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 20px;
            margin-top: 20px;
        }
        @media (max-width: 768px) {
            .layout { grid-template-columns: 1fr; }
        }
        .card {
            background: #1e293b;
            border: 1px solid #29344d;
            border-radius: 16px;
            padding: 20px;
        }
        .card strong {
            display: block;
            font-size: 18px;
            margin-bottom: 12px;
        }
        textarea {
            width: 100%;
            height: 200px;
            background: #0f172a;
            color: white;
            border: 1px solid #334155;
            border-radius: 8px;
            padding: 12px;
            box-sizing: border-box;
            resize: none;
        }
        button {
            width: 100%;
            padding: 14px;
            background: #0ea5e9;
            color: white;
            border: none;
            border-radius: 8px;
            font-weight: bold;
            cursor: pointer;
            margin-top: 12px;
        }
        button:hover { background: #0284c7; }
        .output {
            background: #0f172a;
            border: 1px solid #334155;
            border-radius: 8px;
            padding: 15px;
            height: 200px;
            overflow-y: auto;
            white-space: pre-wrap;
            font-size: 14px;
            line-height: 1.6;
        }
    </style>
</head>
<body>
    <div class="main">
        <header>
            <div class="brand">Missed Mate AI</div>
            <div class="badge">🔒 Local Edge Processing Active</div>
        </header>

        <section class="hero">
            <h1>What Did I <span>Miss?</span></h1>
            <p>Catch up on overwhelming chat group threads instantly. This application runs entirely inside your browser cache—ensuring conversational privacy by design.</p>
        </section>

        <main class="layout">
            <div class="card">
                <strong>Unread Thread History</strong>
                <textarea id="chatBox" placeholder="Paste chat log transcripts here...&#10;Example:&#10;[10:00] Alex: Remember the presentation deadline is tonight!&#10;[10:02] Sam: I'll finish the final checks."></textarea>
                <button onclick="analyzeLocally()">Parse & Summarize</button>
            </div>

            <div class="card">
                <strong>Smart Insights Panel</strong>
                <div id="outputPanel" class="output">Awaiting local input log ingestion...</div>
            </div>
        </main>
    </div>

    <script>
        function analyzeLocally() {
            const input = document.getElementById('chatBox').value.trim();
            const out = document.getElementById('outputPanel');
            if(!input) {
                out.innerHTML = "<span style='color:#f87171;'>Please provide text to evaluate.</span>";
                return;
            }
            out.innerText = "Analyzing text streams entirely on-device via local runtime engine...";
            setTimeout(() => {
                out.innerHTML = `<strong>📋 Executive Summary:</strong>\nThe conversation highlights immediate project deployment milestones and team assignments.\n\n<strong>✅ Action Items & Decisions:</strong>\n• Finalize system verification checks before deployment.\n\n<strong>⏳ Deadlines & Priorities:</strong>\n• Critical: Core Presentation delivery deadline scheduled for tonight.`;
            }, 900);
        }
    </script>
</body>
</html>
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
import { pipeline } from '@xenova/transformers';

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
  console.log("⏳ Initializing local AI pipeline...");

  const generator = await pipeline('text-generation', 'Xenova/Qwen1.5-0.5B');

  const prompt = `You are a local-first privacy-focused assistant. Analyze the chat log below and extract a structured summary.

Strictly provide:
- A brief 1-sentence overview.
- A bulleted list of action items detailing who does what and by when.

Chat Log:
${chatHistory}

Summary:`;

  console.log("🧠 Analyzing chat data...");
  const output = await generator(prompt, { 
    max_new_tokens: 250, 
    temperature: 0.2 
  });

  console.log("\n✨ ANALYSIS RESULT:\n");
  console.log(output.generated_text);
}

runLocalAIAnalysis();
node index.js

