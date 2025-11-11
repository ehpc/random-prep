
```jsx
// With states
import { useState } from "react";

const BOX_SIZE = 80;

function App() {
  const [position, setPosition] = useState({ x: 100, y: 100 });
  const [dragging, setDragging] = useState(false);
  const [offset, setOffset] = useState({ x: 0, y: 0 });

  const handleMouseDown = (e) => {
    setDragging(true);
    setOffset({
      x: e.clientX - position.x,
      y: e.clientY - position.y,
    });
  };

  const handleMouseMove = (e) => {
    if (!dragging) return;

    setPosition({
      x: e.clientX - offset.x,
      y: e.clientY - offset.y,
    });
  };

  const stopDragging = () => setDragging(false);

  return (
    <div
      onMouseMove={handleMouseMove}
      onMouseUp={stopDragging}
      onMouseLeave={stopDragging}
      style={{
        width: "100vw",
        height: "100vh",
        position: "relative",
        background: "#f3f4f6",
        userSelect: "none",
        overflow: "hidden",
      }}
    >
      <div
        onMouseDown={handleMouseDown}
        style={{
          width: BOX_SIZE,
          height: BOX_SIZE,
          background: "#3b82f6",
          borderRadius: 8,
          cursor: dragging ? "grabbing" : "grab",
          boxShadow: "0 10px 20px rgba(15,23,42,0.25)",
          transform: `translate(${position.x}px, ${position.y}px)`,
          transition: dragging ? "none" : "transform 0.05s linear",
          willChange: "transform",
        }}
      />
    </div>
  );
}

export default App;
```

```jsx
// No states

import { useEffect, useRef } from "react";

const BOX_SIZE = 80;

function App() {
  const boxRef = useRef(null);
  const positionRef = useRef({ x: 100, y: 100 });
  const draggingRef = useRef(false);
  const offsetRef = useRef({ x: 0, y: 0 });

  const updateTransform = () => {
    const el = boxRef.current;
    if (!el) return;
    const { x, y } = positionRef.current;
    el.style.transform = `translate(${x}px, ${y}px)`;
  };

  useEffect(() => {
    updateTransform();

    const handleMove = (e) => {
      if (!draggingRef.current) return;

      positionRef.current = {
        x: e.clientX - offsetRef.current.x,
        y: e.clientY - offsetRef.current.y,
      };

      updateTransform();
    };

    const handleUp = () => {
      draggingRef.current = false;
    };

    window.addEventListener("mousemove", handleMove);
    window.addEventListener("mouseup", handleUp);

    return () => {
      window.removeEventListener("mousemove", handleMove);
      window.removeEventListener("mouseup", handleUp);
    };
  }, []);

  const handleMouseDown = (e) => {
    const { x, y } = positionRef.current;

    draggingRef.current = true;
    offsetRef.current = {
      x: e.clientX - x,
      y: e.clientY - y,
    };

    e.preventDefault();
  };

  return (
    <div
      style={{
        width: "100vw",
        height: "100vh",
        position: "relative",
        background: "#f3f4f6",
        userSelect: "none",
        overflow: "hidden",
      }}
    >
      <div
        ref={boxRef}
        onMouseDown={handleMouseDown}
        style={{
          width: BOX_SIZE,
          height: BOX_SIZE,
          background: "#3b82f6",
          borderRadius: 8,
          position: "absolute",
          top: 0,
          left: 0,
          willChange: "transform",
          cursor: "grab",
          boxShadow: "0 10px 20px rgba(15,23,42,0.25)",
        }}
      />
    </div>
  );
}

export default App;
```