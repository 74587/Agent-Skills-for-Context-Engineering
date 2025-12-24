# LLM-as-a-Judge Skills

A practical implementation of LLM evaluation skills demonstrating how to transform research insights into production-ready agent capabilities.

## Overview

This example demonstrates how to build **evaluation skills** for AI agents based on [Eugene Yan's LLM-Evaluators research](https://eugeneyan.com/writing/llm-evaluators/) and [Vercel AI SDK 6](https://vercel.com/blog/ai-sdk-6) patterns.

## Skills Applied

| Skill | Application |
|-------|-------------|
| **context-fundamentals** | Structured evaluation context with clear prompt/response/criteria separation |
| **tool-design** | Zod schemas for type-safe inputs, graceful error handling, structured outputs |
| **evaluation** | LLM-as-a-Judge patterns: direct scoring, pairwise comparison, rubric generation |
| **context-optimization** | Chain-of-thought prompting for reliable scoring, position bias mitigation |

## Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        LLM-as-a-Judge Skills                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────────┐  │
│  │   Skills    │    │   Prompts   │    │         Tools           │  │
│  │  (MD docs)  │───▶│  (templates)│───▶│  (TypeScript impl)      │  │
│  └─────────────┘    └─────────────┘    └─────────────────────────┘  │
│         │                                         │                   │
│         │                                         ▼                   │
│         │                              ┌─────────────────────────┐  │
│         └─────────────────────────────▶│    EvaluatorAgent       │  │
│                                         │  ├── score()            │  │
│                                         │  ├── compare()          │  │
│                                         │  ├── generateRubric()   │  │
│                                         │  └── chat()             │  │
│                                         └─────────────────────────┘  │
│                                                                       │
└─────────────────────────────────────────────────────────────────────┘
```

## Key Research Insights Implemented

From Eugene Yan's [Evaluating the Effectiveness of LLM-Evaluators](https://eugeneyan.com/writing/llm-evaluators/):

### 1. Scoring Approach Selection

| Approach | Best For | Implementation |
|----------|----------|----------------|
| **Direct Scoring** | Objective criteria (accuracy, completeness) | `directScore` tool with rubric support |
| **Pairwise Comparison** | Subjective preferences (tone, style) | `pairwiseCompare` tool with position swapping |

### 2. Bias Mitigation Techniques

- **Position Bias**: Automatic position swapping in pairwise comparisons
- **Length Bias**: Explicit prompting to ignore response length
- **Chain-of-Thought**: Required justification with evidence for all scores

### 3. Evaluation Metrics

- **Classification**: Recall, precision for binary judgments
- **Correlation**: Cohen's κ, Spearman's ρ for ordinal scales
- **Consistency**: Position swap agreement rate

## Tools Implemented

### directScore

Scores a single response against weighted criteria with rubric support.

```typescript
const result = await executeDirectScore({
  response: 'AI-generated response',
  prompt: 'Original prompt',
  criteria: [
    { name: 'Accuracy', description: 'Factual correctness', weight: 0.5 },
    { name: 'Clarity', description: 'Easy to understand', weight: 0.5 }
  ],
  rubric: { scale: '1-5' }
});
// Returns: { overallScore: 4.2, scores: [...], summary: {...} }
```

### pairwiseCompare

Compares two responses with position bias mitigation.

```typescript
const result = await executePairwiseCompare({
  responseA: 'First response',
  responseB: 'Second response',
  prompt: 'Original prompt',
  criteria: ['accuracy', 'completeness'],
  swapPositions: true  // Mitigates position bias
});
// Returns: { winner: 'A', confidence: 0.85, positionConsistency: {...} }
```

### generateRubric

Creates domain-specific scoring rubrics.

```typescript
const result = await executeGenerateRubric({
  criterionName: 'Code Readability',
  criterionDescription: 'How easy the code is to understand',
  scale: '1-5',
  domain: 'software engineering',
  strictness: 'balanced'
});
// Returns: { levels: [...], scoringGuidelines: [...], edgeCases: [...] }
```

## Test Results

All 19 tests pass successfully:

```
 ✓ tests/skills.test.ts (10 tests) 159317ms
 ✓ tests/evaluation.test.ts (9 tests) 216353ms

 Test Files  2 passed (2)
      Tests  19 passed (19)
   Duration  216.66s
```

### Key Learnings from Testing

1. **Position bias is real**: Tests confirm that swapping positions detects and mitigates bias
2. **Chain-of-thought improves quality**: Justifications >20 characters correlate with reliable scores
3. **Domain-specific rubrics matter**: Software engineering rubrics include relevant terminology
4. **Context affects evaluation**: Medical context allowed technical terminology to score appropriately

## Project Structure

```
llm-as-judge-skills/
├── skills/                    # Foundational knowledge (MD)
│   ├── llm-evaluator/
│   ├── context-fundamentals/
│   └── tool-design/
├── tools/                     # Tool documentation (MD)
│   └── evaluation/
├── prompts/                   # Prompt templates (MD)
│   └── evaluation/
├── src/                       # TypeScript implementation
│   ├── tools/evaluation/
│   └── agents/
├── tests/                     # Test suite
└── examples/                  # Usage examples
```

## Quick Start

```bash
git clone https://github.com/muratcankoylan/llm-as-judge-skills.git
cd llm-as-judge-skills
npm install
cp env.example .env  # Add OPENAI_API_KEY
npm test
```

## Repository

Full implementation: [github.com/muratcankoylan/llm-as-judge-skills](https://github.com/muratcankoylan/llm-as-judge-skills)

## References

- [Eugene Yan: Evaluating the Effectiveness of LLM-Evaluators](https://eugeneyan.com/writing/llm-evaluators/)
- [Vercel AI SDK 6](https://vercel.com/blog/ai-sdk-6)
- [MT-Bench and Chatbot Arena Paper](https://arxiv.org/abs/2306.05685)

