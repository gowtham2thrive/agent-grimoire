# Exemplar: Accessible Generative AI Streaming Workspace

> **Scenario**: An AI chat interface that streams continuous text tokens from an LLM and renders tool invocation cards.
> **Accessibility Challenge**: 
> - Prevent screen reader speech synthesizer crashes caused by per-token live announcements.
> - Keep user focus anchored on the prompt input throughout generation.
> - Provide an immediate keyboard shortcut to abort generation without losing partial results.
> - Announce generation start and completion via debounced status announcements.

---

## Complete Accessible React Implementation

```tsx
import React, { useState, useEffect, useRef, useCallback } from "react";

interface AgentMessage {
  id: string;
  role: "user" | "assistant";
  content: string;
  toolCall?: { name: string; status: "running" | "done" };
}

export function AccessibleStreamingWorkspace() {
  const [messages, setMessages] = useState<AgentMessage[]>([]);
  const [inputVal, setInputVal] = useState("");
  const [isGenerating, setIsGenerating] = useState(false);
  const [liveAnnouncement, setLiveAnnouncement] = useState("");
  
  const inputRef = useRef<HTMLInputElement>(null);
  const abortControllerRef = useRef<AbortController | null>(null);

  // Keyboard shortcut: Escape cancels generation and preserves partial response
  const handleAbort = useCallback(() => {
    if (isGenerating && abortControllerRef.current) {
      abortControllerRef.current.abort();
      setIsGenerating(false);
      setLiveAnnouncement("Generation stopped by user.");
      inputRef.current?.focus(); // Maintain focus stability
    }
  }, [isGenerating]);

  useEffect(() => {
    const onKeyDown = (e: KeyboardEvent) => {
      if (e.key === "Escape" && isGenerating) {
        e.preventDefault();
        handleAbort();
      }
    };
    window.addEventListener("keydown", onKeyDown);
    return () => window.removeEventListener("keydown", onKeyDown);
  }, [isGenerating, handleAbort]);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    if (!inputVal.trim() || isGenerating) return;

    const userText = inputVal;
    setInputVal("");
    const newMsgId = String(Date.now());

    // Append user message and empty assistant placeholder
    setMessages((prev) => [
      ...prev,
      { id: newMsgId, role: "user", content: userText },
      { id: `${newMsgId}-res`, role: "assistant", content: "" },
    ]);

    setIsGenerating(true);
    // DISPATCH INITIAL STATUS TO SCREEN READER
    setLiveAnnouncement("AI is processing prompt. Press Escape to abort.");

    const controller = new AbortController();
    abortControllerRef.current = controller;

    try {
      // Simulated streaming chunks (In production, replace with fetch SSE / WebSocket)
      const mockTokens = ["Analyzing ", "project ", "architecture... ", "Found ", "3 ", "invariants."];
      for (const token of mockTokens) {
        if (controller.signal.aborted) break;
        await new Promise((r) => setTimeout(r, 200));
        setMessages((prev) =>
          prev.map((msg) =>
            msg.id === `${newMsgId}-res` ? { ...msg, content: msg.content + token } : msg
          )
        );
      }
      if (!controller.signal.aborted) {
        setIsGenerating(false);
        // DISPATCH COMPLETION SUMMARY TO SCREEN READER
        setLiveAnnouncement("AI response complete. 3 findings generated. Navigate up to review.");
      }
    } catch {
      setIsGenerating(false);
      setLiveAnnouncement("Generation failed due to a network error. Press retry.");
    }
  };

  return (
    <main className="ai-workspace" aria-labelledby="workspace-title">
      <h1 id="workspace-title" className="sr-only">AI Agent Workspace</h1>

      {/* 
        PRE-RENDERED LIVE REGION:
        Exists permanently in the DOM. Debounced status updates only.
        NEVER place aria-live on the streaming text container!
      */}
      <div 
        role="status" 
        aria-live="polite" 
        aria-atomic="true" 
        className="sr-only"
      >
        {liveAnnouncement}
      </div>

      {/* Message Feed */}
      <section className="message-log" aria-label="Conversation History">
        {messages.map((msg) => (
          <article 
            key={msg.id} 
            className={`message-card ${msg.role}`}
            aria-busy={msg.role === "assistant" && isGenerating}
          >
            <h2 className="sr-only">{msg.role === "user" ? "You said:" : "AI Assistant replied:"}</h2>
            <div className="message-content">
              {msg.content}
              {msg.role === "assistant" && isGenerating && (
                <span className="blinking-cursor" aria-hidden="true">▋</span>
              )}
            </div>
          </article>
        ))}
      </section>

      {/* Persistent Prompt Bar */}
      <footer className="workspace-controls">
        <form onSubmit={handleSubmit} className="input-row">
          <label htmlFor="agent-prompt" className="sr-only">Your message to the AI</label>
          <input
            ref={inputRef}
            id="agent-prompt"
            type="text"
            value={inputVal}
            onChange={(e) => setInputVal(e.target.value)}
            placeholder="Type your instruction... (Press Esc to abort)"
            disabled={false} // Input remains interactive during generation
          />
          {isGenerating ? (
            <button 
              type="button" 
              onClick={handleAbort} 
              aria-label="Stop generating response (Escape)"
              className="btn-abort"
            >
              Stop
            </button>
          ) : (
            <button type="submit" className="btn-send">
              Send
            </button>
          )}
        </form>
      </footer>
    </main>
  );
}
```
