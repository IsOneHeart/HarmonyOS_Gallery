# ArkTS Overview

ArkTS is a programming language specifically designed for HarmonyOS application development, built upon JavaScript and TypeScript with extensions and optimizations. Its design philosophy prioritizes development efficiency and runtime performance while maintaining strong compatibility with JavaScript/TypeScript ecosystems. This allows reuse of most existing JavaScript/TypeScript codebases and toolchains.

If you already have JavaScript/TypeScript experience, you'll find ArkTS quick to learn with these key considerations:

### Enhanced Type Strictness
While extending JavaScript/TypeScript capabilities, ArkTS introduces stricter type enforcement to reduce runtime overhead and improve performance:

- Variables cannot change type after declaration (e.g., a `number`-declared variable cannot be assigned `undefined`/`null`)
- Use **Union Types** to declare variables accepting multiple value types

```arkts
let x: number;
x = 1; // Valid
x = undefined;    // Type error
x = null;         // Type error

let y: number | null | undefined;
y = 1;    // Valid
y = undefined;    // Valid
y = null;         // Valid
```

### ArkUI Framework Integration
ArkTS includes the ArkUI framework for building HarmonyOS application interfaces. Key features:

Declarative Development Paradigm: Simplified UI development with intuitive component composition
Rich Component Library: Pre-built UI elements and layout containers for rapid prototyping
Responsive System: Automatic adaptation to different screen sizes and input modalities
The framework enables developers to create visually consistent, performant UIs with minimal boilerplate code while maintaining full control over custom component behavior.