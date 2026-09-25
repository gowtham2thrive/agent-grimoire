# Multi-Archetype Rosetta Stone: Cross-Platform Implementation Guide

> **Mandate**: *Accessibility is universal; syntax is local.* 
> This reference manual maps abstract accessibility concepts into concrete, idiomatic implementations across all major computing platforms.

---

## 1 · Concrete Archetype Implementation Matrix

### 1. Modern Web (HTML5, React, CSS)
```tsx
import React, { useRef, useEffect } from "react";

interface DialogProps {
  isOpen: boolean;
  onClose: () => void;
  title: string;
  children: React.ReactNode;
}

export function AccessibleWebDialog({ isOpen, onClose, title, children }: DialogProps) {
  const dialogRef = useRef<HTMLDialogElement>(null);
  const previousFocusRef = useRef<HTMLElement | null>(null);

  useEffect(() => {
    const dialog = dialogRef.current;
    if (isOpen) {
      previousFocusRef.current = document.activeElement as HTMLElement;
      dialog?.showModal(); // Built-in focus trap, backdrop inertness, Escape listener
    } else {
      dialog?.close();
      previousFocusRef.current?.focus(); // Deterministic focus restore
    }
  }, [isOpen]);

  return (
    <dialog 
      ref={dialogRef} 
      onCancel={onClose} 
      aria-labelledby="dialog-heading"
      className="accessible-dialog"
    >
      <header className="dialog-header">
        <h2 id="dialog-heading">{title}</h2>
        <button type="button" onClick={onClose} aria-label="Close dialog">✕</button>
      </header>
      <div className="dialog-body">{children}</div>
    </dialog>
  );
}
```

---

### 2. Mobile iOS (SwiftUI)
```swift
import SwiftUI

struct AccessibleModalView: View {
    @Binding var isPresented: Bool
    let title: String
    @AccessibilityFocusState private var isHeaderFocused: Bool

    var body: some View {
        NavigationStack {
            VStack(alignment: .leading, spacing: 16) {
                Text(title)
                    .font(.title2)
                    .accessibilityAddTraits(.isHeader)
                    .accessibilityFocused($isHeaderFocused)
                
                Text("Modal content and controls go here.")
                    .font(.body)
                
                Spacer()
                
                Button("Save and Close") {
                    isPresented = false
                    UIAccessibility.post(notification: .announcement, argument: "Changes saved successfully")
                }
                .buttonStyle(.borderedProminent)
                .accessibilityHint("Saves your updates and returns to previous screen")
            }
            .padding()
            .navigationTitle(title)
            .toolbar {
                ToolbarItem(placement: .cancellationAction) {
                    Button("Close") { isPresented = false }
                        .accessibilityLabel("Dismiss modal")
                }
            }
        }
        .accessibilityAddTraits(.isModal) // Traps VoiceOver within sheet
        .onAppear {
            isHeaderFocused = true // Programmatically moves VoiceOver focus to header
        }
    }
}
```

---

### 3. Mobile Android (Jetpack Compose)
```kotlin
import androidx.compose.foundation.layout.*
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Modifier
import androidx.compose.ui.focus.FocusRequester
import androidx.compose.ui.focus.focusRequester
import androidx.compose.ui.semantics.*
import androidx.compose.ui.unit.dp

@Composable
fun AccessibleAndroidDialog(
    isOpen: Boolean,
    onDismiss: () -> Unit,
    title: String,
    onConfirm: () -> Unit
) {
    if (!isOpen) return

    val focusRequester = remember { FocusRequester() }

    AlertDialog(
        onDismissRequest = onDismiss,
        title = {
            Text(
                text = title,
                modifier = Modifier.semantics { heading() }
            )
        },
        text = {
            Text("Are you sure you want to delete this resource? This action cannot be undone.")
        },
        confirmButton = {
            Button(
                onClick = onConfirm,
                modifier = Modifier.focusRequester(focusRequester)
            ) {
                Text("Confirm Delete")
            }
        },
        dismissButton = {
            OutlinedButton(onClick = onDismiss) {
                Text("Cancel")
            }
        },
        modifier = Modifier.semantics {
            paneTitle = title // Identifies the dialog to TalkBack screen reader
        }
    )

    LaunchedEffect(Unit) {
        focusRequester.requestFocus() // Explicit initial focus placement
    }
}
```

---

### 4. Desktop Native (Windows WinUI 3 / C#)
```csharp
using Microsoft.UI.Xaml;
using Microsoft.UI.Xaml.Controls;
using Microsoft.UI.Xaml.Automation;
using Microsoft.UI.Xaml.Automation.Peers;

public sealed partial class AccessibleDialogWindow : ContentDialog
{
    private UIElement _originatingTrigger;

    public AccessibleDialogWindow(UIElement trigger, string titleText)
    {
        this.InitializeComponent();
        this._originatingTrigger = trigger;
        this.Title = titleText;
        this.DefaultButton = ContentDialogButton.Primary;

        // Ensure automation peer exposes modal dialog pattern
        AutomationProperties.SetName(this, titleText);
        AutomationProperties.SetAutomationId(this, "ConfirmationDialogId");
    }

    private void ContentDialog_Closed(ContentDialog sender, ContentDialogClosedEventArgs args)
    {
        // Restore focus to trigger on close
        if (_originatingTrigger is Control control)
        {
            control.Focus(FocusState.Programmatic);
        }
    }
}
```

---

### 5. Terminal CLI / TUI (Rust / Ratatui)
```rust
use ratatui::{
    prelude::*,
    widgets::{Block, Borders, Paragraph, Clear},
};

pub struct AccessibleTuiModal {
    pub title: String,
    pub message: String,
    pub is_open: bool,
    pub focused_button: usize, // 0: Confirm, 1: Cancel
}

impl AccessibleTuiModal {
    pub fn render(&self, area: Rect, buf: &mut Buffer) {
        if !self.is_open { return; }

        let popup_area = Rect {
            x: area.width / 4,
            y: area.height / 3,
            width: area.width / 2,
            height: 8,
        };

        // Clear background characters so screen reader text dump remains clean
        Clear.render(popup_area, buf);

        let block = Block::default()
            .title(format!(" [ {} ] ", self.title))
            .borders(Borders::ALL)
            .border_style(Style::default().fg(Color::Yellow).add_modifier(Modifier::BOLD));
        block.render(popup_area, buf);

        // High-contrast, unambiguous textual buttons (brackets indicate focus)
        let confirm_btn = if self.focused_button == 0 {
            "[ > CONFIRM < ]"
        } else {
            "  Confirm  "
        };
        let cancel_btn = if self.focused_button == 1 {
            "[ > CANCEL < ]"
        } else {
            "  Cancel  "
        };

        let content = format!("{}\n\n  {}    {}", self.message, confirm_btn, cancel_btn);
        let paragraph = Paragraph::new(content).style(Style::default().fg(Color::White));
        
        let inner_area = Rect {
            x: popup_area.x + 2,
            y: popup_area.y + 2,
            width: popup_area.width - 4,
            height: popup_area.height - 4,
        };
        paragraph.render(inner_area, buf);
    }
}
```

---

### 6. AI Agent Streaming & Generative UI (TypeScript / React)
```tsx
import React, { useState, useEffect, useRef } from "react";

interface StreamingMessageProps {
  prompt: string;
  streamText: string;
  isStreaming: boolean;
  onCancel: () => void;
}

export function AccessibleStreamingAgentUI({ 
  prompt, 
  streamText, 
  isStreaming, 
  onCancel 
}: StreamingMessageProps) {
  const [statusMessage, setStatusMessage] = useState("");
  const promptInputRef = useRef<HTMLInputElement>(null);

  useEffect(() => {
    if (isStreaming) {
      setStatusMessage("AI is generating response. Press Escape to abort.");
    } else if (streamText.length > 0) {
      setStatusMessage("Response generation complete. Navigate down to read summary.");
    }
  }, [isStreaming, streamText]);

  // Global keydown handler to allow immediate abort without mouse
  useEffect(() => {
    const handleKeyDown = (e: KeyboardEvent) => {
      if (e.key === "Escape" && isStreaming) {
        e.preventDefault();
        onCancel();
        setStatusMessage("Generation cancelled by user.");
        promptInputRef.current?.focus(); // Keep focus stable on input
      }
    };
    window.addEventListener("keydown", handleKeyDown);
    return () => window.removeEventListener("keydown", handleKeyDown);
  }, [isStreaming, onCancel]);

  return (
    <section className="agent-chat-pane" aria-labelledby="chat-heading">
      <h2 id="chat-heading" className="sr-only">AI Assistant Conversation</h2>

      {/* Persistent live region: Debounced announcements only (NO PER-TOKEN SPAM) */}
      <div 
        role="status" 
        aria-live="polite" 
        aria-atomic="true" 
        className="sr-only"
      >
        {statusMessage}
      </div>

      {/* Streaming output: Silent to live regions during generation */}
      <article 
        className="agent-response-card" 
        aria-busy={isStreaming}
      >
        <header className="card-header">
          <h3>Response to: "{prompt}"</h3>
          {isStreaming && (
            <button 
              type="button" 
              onClick={onCancel} 
              aria-label="Stop generation (Esc)"
              className="stop-button"
            >
              Stop
            </button>
          )}
        </header>

        <div className="markdown-stream-body">
          {streamText}
          {isStreaming && <span className="cursor-pulse" aria-hidden="true">▋</span>}
        </div>
      </article>

      <form onSubmit={(e) => e.preventDefault()} className="prompt-form">
        <label htmlFor="agent-input" className="sr-only">Ask a follow-up</label>
        <input 
          ref={promptInputRef}
          id="agent-input" 
          type="text" 
          placeholder="Ask follow-up... (Esc stops active response)"
        />
      </form>
    </section>
  );
}
```
