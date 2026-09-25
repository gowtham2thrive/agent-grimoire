# Exemplar: Canvas Data Visualization with Virtual Semantic Mirror & Tabular Toggle

> **Scenario**: An interactive 2D data chart rendering financial performance over quarters onto an HTML `<canvas>`.
> **Accessibility Solution**:
> 1. Maintain an off-screen Virtual Semantic Mirror with synchronized button nodes for each data bar.
> 2. Draw high-contrast focus rings onto the canvas when a virtual bar receives keyboard focus.
> 3. Provide an instant 1-click / keyboard shortcut (`Alt+T`) toggle that replaces the graphical canvas with an accessible semantic `<table>`.

---

## Complete Implementation

```tsx
import React, { useState, useRef, useEffect } from "react";

interface DataPoint {
  id: string;
  label: string;
  value: number; // in $M
  growth: string;
}

const DATA: DataPoint[] = [
  { id: "q1", label: "Q1 2025", value: 12.4, growth: "+8.2%" },
  { id: "q2", label: "Q2 2025", value: 15.1, growth: "+21.7%" },
  { id: "q3", label: "Q3 2025", value: 14.8, growth: "-1.9%" },
  { id: "q4", label: "Q4 2025", value: 18.2, growth: "+22.9%" },
];

export function AccessibleCanvasChart() {
  const [showTableView, setShowTableView] = useState(false);
  const [focusedId, setFocusedId] = useState<string | null>(null);
  const canvasRef = useRef<HTMLCanvasElement>(null);

  // Keyboard shortcut: Alt+T toggles between canvas and table view
  useEffect(() => {
    const handleKeyDown = (e: KeyboardEvent) => {
      if (e.altKey && e.key.toLowerCase() === "t") {
        e.preventDefault();
        setShowTableView((prev) => !prev);
      }
    };
    window.addEventListener("keydown", handleKeyDown);
    return () => window.removeEventListener("keydown", handleKeyDown);
  }, []);

  // Canvas drawing loop: synchronizes visual focus with virtual node focus
  useEffect(() => {
    if (showTableView) return;
    const canvas = canvasRef.current;
    if (!canvas) return;
    const ctx = canvas.getContext("2d");
    if (!ctx) return;

    ctx.clearRect(0, 0, canvas.width, canvas.height);

    const barWidth = 60;
    const spacing = 40;
    const startX = 60;
    const baselineY = 220;

    DATA.forEach((pt, idx) => {
      const x = startX + idx * (barWidth + spacing);
      const barHeight = pt.value * 8;
      const y = baselineY - barHeight;

      // Draw Bar
      ctx.fillStyle = "#2563eb";
      ctx.fillRect(x, y, barWidth, barHeight);

      // Label below bar
      ctx.fillStyle = "#1e293b";
      ctx.font = "14px sans-serif";
      ctx.fillText(pt.label, x + 5, baselineY + 20);

      // Value above bar
      ctx.fillText(`$${pt.value}M`, x + 8, y - 8);

      // CRITICAL: Draw high-contrast focus ring if virtual node is focused
      if (focusedId === pt.id) {
        ctx.strokeStyle = "#000000";
        ctx.lineWidth = 3;
        ctx.strokeRect(x - 4, y - 4, barWidth + 8, barHeight + 8);
        ctx.strokeStyle = "#ffffff";
        ctx.lineWidth = 1.5;
        ctx.strokeRect(x - 2, y - 2, barWidth + 4, barHeight + 4);
      }
    });
  }, [showTableView, focusedId]);

  return (
    <section className="chart-widget" aria-labelledby="chart-title">
      <div className="header-controls">
        <h2 id="chart-title">Quarterly Revenue Trend</h2>
        <button
          type="button"
          onClick={() => setShowTableView(!showTableView)}
          className="btn-toggle-view"
          aria-expanded={showTableView}
        >
          {showTableView ? "Show Visual Chart (Alt+T)" : "Show Accessible Table (Alt+T)"}
        </button>
      </div>

      {!showTableView ? (
        <div className="canvas-wrapper" style={{ position: "relative", width: 500, height: 260 }}>
          {/* Visual Framebuffer */}
          <canvas
            ref={canvasRef}
            width={500}
            height={260}
            aria-hidden="true" /* Screen readers interact via the virtual mirror below */
          />

          {/* 
            OFF-SCREEN VIRTUAL SEMANTIC MIRROR:
            Exposes real accessible interactive nodes with matching bounding boxes 
          */}
          <div className="virtual-mirror" role="group" aria-label="Quarterly Revenue Bars">
            {DATA.map((pt, idx) => {
              const barWidth = 60;
              const spacing = 40;
              const x = 60 + idx * (barWidth + spacing);
              const barHeight = pt.value * 8;
              const y = 220 - barHeight;

              return (
                <button
                  key={pt.id}
                  type="button"
                  onFocus={() => setFocusedId(pt.id)}
                  onBlur={() => setFocusedId(null)}
                  onClick={() => alert(`Details for ${pt.label}: $${pt.value}M (${pt.growth})`)}
                  aria-label={`${pt.label}: $${pt.value} million gross revenue, ${pt.growth} growth`}
                  style={{
                    position: "absolute",
                    left: `${x}px`,
                    top: `${y}px`,
                    width: `${barWidth}px`,
                    height: `${barHeight}px`,
                    opacity: 0.001, // Invisible visually, fully focusable & hit-testable
                    cursor: "pointer",
                  }}
                />
              );
            })}
          </div>
        </div>
      ) : (
        /* THE TABULAR ALTERNATIVE CONTRACT */
        <table className="accessible-data-table">
          <caption>Quarterly Revenue Trend (2025–2026)</caption>
          <thead>
            <tr>
              <th scope="col">Quarter</th>
              <th scope="col">Gross Revenue ($M)</th>
              <th scope="col">Growth (%)</th>
            </tr>
          </thead>
          <tbody>
            {DATA.map((pt) => (
              <tr key={pt.id}>
                <th scope="row">{pt.label}</th>
                <td>${pt.value}M</td>
                <td>{pt.growth}</td>
              </tr>
            ))}
          </tbody>
        </table>
      )}
    </section>
  );
}
```
