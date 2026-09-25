# Exemplar: The Modal Dialog Rosetta Stone

> **Scenario**: Implement an interactive confirmation modal dialog across three distinct platforms (Modern Web, iOS SwiftUI, and Terminal TUI).
> **Accessibility Contract**:
> - Trap focus within dialog bounds while open.
> - Announce dialog title on open.
> - Dismiss on `Escape` key.
> - Deterministically restore focus to triggering control upon close.

---

## 1 · Modern Web Implementation (HTML5 `<dialog>` + React)

Native HTML5 `<dialog>` provides built-in focus trapping, top-layer backdrop inertness, and `Escape` key handling out of the box.

```tsx
import React, { useRef, useEffect } from "react";

interface ConfirmationDialogProps {
  isOpen: boolean;
  title: string;
  message: string;
  onConfirm: () => void;
  onCancel: () => void;
}

export function AccessibleConfirmDialog({
  isOpen,
  title,
  message,
  onConfirm,
  onCancel,
}: ConfirmationDialogProps) {
  const dialogRef = useRef<HTMLDialogElement>(null);
  const triggerRef = useRef<HTMLElement | null>(null);

  useEffect(() => {
    const dialog = dialogRef.current;
    if (!dialog) return;

    if (isOpen) {
      // 1. Capture current focused element before opening
      triggerRef.current = document.activeElement as HTMLElement;
      // 2. showModal() natively traps keyboard focus, renders ::backdrop, and makes background inert
      dialog.showModal();
    } else {
      dialog.close();
      // 3. Restore focus deterministically to triggering button
      triggerRef.current?.focus();
    }
  }, [isOpen]);

  return (
    <dialog
      ref={dialogRef}
      onCancel={(e) => {
        e.preventDefault(); // Control escape lifecycle
        onCancel();
      }}
      aria-labelledby="dialog-title"
      aria-describedby="dialog-desc"
      className="accessible-confirm-dialog"
    >
      <h2 id="dialog-title">{title}</h2>
      <p id="dialog-desc">{message}</p>
      
      <div className="dialog-actions">
        <button type="button" onClick={onCancel} className="btn-secondary">
          Cancel
        </button>
        <button type="button" onClick={onConfirm} className="btn-danger">
          Delete Permanently
        </button>
      </div>
    </dialog>
  );
}
```

---

## 2 · Mobile iOS Implementation (SwiftUI)

In SwiftUI, modal sheets can leak VoiceOver navigation unless marked with `.accessibilityAddTraits(.isModal)`.

```swift
import SwiftUI

struct ConfirmationSheet: View {
    @Binding var isPresented: Bool
    let title: String
    let message: String
    let onConfirm: () -> Void
    
    @AccessibilityFocusState private var isTitleFocused: Bool

    var body: some View {
        VStack(spacing: 20) {
            Text(title)
                .font(.title3)
                .fontWeight(.bold)
                .accessibilityAddTraits(.isHeader)
                .accessibilityFocused($isTitleFocused)

            Text(message)
                .font(.body)
                .foregroundColor(.secondary)

            HStack(spacing: 16) {
                Button("Cancel", role: .cancel) {
                    isPresented = false
                }
                .buttonStyle(.bordered)

                Button("Delete Permanently", role: .destructive) {
                    onConfirm()
                    isPresented = false
                    UIAccessibility.post(
                        notification: .announcement, 
                        argument: "Resource deleted successfully"
                    )
                }
                .buttonStyle(.borderedProminent)
            }
        }
        .padding(24)
        .accessibilityElement(children: .contain)
        .accessibilityAddTraits(.isModal) // Confines VoiceOver cursor strictly within this sheet
        .onAppear {
            isTitleFocused = true // Sets initial VoiceOver landing point to title
        }
    }
}
```

---

## 3 · Terminal CLI / TUI Implementation (Rust / Ratatui)

In a terminal character grid, modal accessibility requires clearing background characters, rendering bold high-contrast borders, and intercepting keyboard navigation before background widgets.

```rust
use ratatui::{
    prelude::*,
    widgets::{Block, Borders, Clear, Paragraph},
};

pub struct TuiConfirmModal {
    pub is_open: bool,
    pub title: String,
    pub message: String,
    pub selected_index: usize, // 0: Cancel, 1: Confirm
}

impl TuiConfirmModal {
    pub fn handle_key(&mut self, key: crossterm::event::KeyCode) -> Option<bool> {
        if !self.is_open { return None; }
        
        match key {
            crossterm::event::KeyCode::Esc => {
                self.is_open = false;
                Some(false) // Cancelled via Escape
            }
            crossterm::event::KeyCode::Left | crossterm::event::KeyCode::Right | crossterm::event::KeyCode::Tab => {
                self.selected_index = 1 - self.selected_index; // Toggle selection
                None
            }
            crossterm::event::KeyCode::Enter => {
                self.is_open = false;
                Some(self.selected_index == 1) // true if Confirm selected
            }
            _ => None,
        }
    }

    pub fn render(&self, area: Rect, buf: &mut Buffer) {
        if !self.is_open { return; }

        let popup = Rect {
            x: area.width.saturating_sub(50) / 2,
            y: area.height.saturating_sub(10) / 2,
            width: 50.min(area.width),
            height: 10.min(area.height),
        };

        // 1. Clear background cells so terminal screen readers do not read bleeding background text
        Clear.render(popup, buf);

        // 2. Render high-contrast structural boundary
        let block = Block::default()
            .title(format!(" [ {} ] ", self.title))
            .borders(Borders::ALL)
            .border_style(Style::default().fg(Color::Yellow).add_modifier(Modifier::BOLD));
        block.render(popup, buf);

        // 3. Unambiguous textual focus indicators
        let cancel_str = if self.selected_index == 0 { "[ > CANCEL < ]" } else { "  Cancel  " };
        let confirm_str = if self.selected_index == 1 { "[ > DELETE < ]" } else { "  Delete  " };

        let text = format!("{}\n\n   {}    {}", self.message, cancel_str, confirm_str);
        let paragraph = Paragraph::new(text).alignment(Alignment::Center);

        let inner = Rect {
            x: popup.x + 2,
            y: popup.y + 2,
            width: popup.width - 4,
            height: popup.height - 4,
        };
        paragraph.render(inner, buf);
    }
}
```
