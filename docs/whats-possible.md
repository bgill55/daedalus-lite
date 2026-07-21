# What's Possible with Daedalus-Lite

Daedalus-Lite's middleware architecture makes it infinitely extensible. The `InMemoryRouter` supports a chain of middleware functions that intercept every request before it reaches the AI model. This unlocks use cases far beyond a simple chat bot.

Here are real-world examples of what you can build by adding a few lines of middleware.

---

## 1. Financial Assistant

Add a middleware that prepends financial context and validates responses against regulatory rules.

```typescript
import type { Middleware } from './types.js';

export const financialGuardMiddleware: Middleware = async (req, next) => {
  req.prompt = `You are a financial assistant. Always include disclaimers. Never give specific stock advice. Answer in plain language.

User question: ${req.prompt}`;

  const response = await next();

  // Post-process: ensure disclaimer is present
  const content = response.choices[0].message.content;
  if (!content.includes('consult a financial advisor')) {
    response.choices[0].message.content +=
      '\n\n⚠️ *This is not financial advice. Consult a qualified professional.*';
  }

  return response;
};
```

**Use cases:** Portfolio analysis, expense tracking, tax guidance, retirement planning.

---

## 2. Code Review Bot

Inject system-level code review instructions and enforce review checklist output.

```typescript
import type { Middleware } from './types.js';

export const codeReviewMiddleware: Middleware = async (req, next) => {
  req.prompt = `You are a senior code reviewer. Review the following code for:
1. Security vulnerabilities
2. Performance issues
3. Code style violations
4. Missing error handling
5. Test coverage gaps

Format your response as a checklist with severity levels (🔴 CRITICAL, 🟡 WARNING, 🟢 INFO).

Code to review:
${req.prompt}`;

  return next();
};
```

**Use cases:** PR review automation, pre-commit hooks, CI pipeline integration.

---

## 3. Legal Document Analyzer

Add context-aware legal disclaimers and structured output parsing.

```typescript
import type { Middleware } from './types.js';

export const legalMiddleware: Middleware = async (req, next) => {
  req.prompt = `You are a legal document analyst. Identify:
- Key clauses and obligations
- Potential risks
- Missing standard provisions
- Jurisdiction-specific concerns

Document: ${req.prompt}

Output as a structured JSON object with sections: summary, risks, recommendations.`;

  return next();
};
```

**Use cases:** Contract review, compliance checking, terms-of-service analysis.

---

## 4. Medical Triage Assistant

Add HIPAA-style privacy guardrails and structured symptom analysis.

```typescript
import type { Middleware } from './types.js';

export const medicalTriageMiddleware: Middleware = async (req, next) => {
  req.prompt = `You are a medical triage assistant. You do NOT diagnose. You do NOT prescribe. You help users understand when to seek professional care.

Symptom description: ${req.prompt}

Respond with:
1. Urgency level (EMERGENCY / URGENT / ROUTINE / SELF-CARE)
2. Recommended next steps
3. Questions to ask their doctor

⚠️ Always include: "This is not medical advice. Consult a healthcare provider."`;

  return next();
};
```

**Use cases:** Symptom checker, appointment triage, medication reminder systems.

---

## 5. Customer Support Agent

Add company knowledge base context and sentiment analysis.

```typescript
import type { Middleware } from './types.js';

const knowledgeBase = `
Product: Acme SaaS Platform
Pricing: $29/mo Basic, $99/mo Pro, $299/mo Enterprise
Refund policy: 30-day money-back guarantee
Support hours: 24/7 chat, email response within 4 hours
`;

export const supportMiddleware: Middleware = async (req, next) => {
  req.prompt = `You are a customer support agent for Acme SaaS. Use the following knowledge base to answer questions. Be helpful, concise, and empathetic.

Knowledge Base:
${knowledgeBase}

Customer: ${req.prompt}`;

  const response = await next();

  // Sentiment check — if customer seems frustrated, escalate
  const content = response.choices[0].message.content;
  if (req.prompt.match(/angry|frustrated|unacceptable|refund|cancel/i)) {
    response.choices[0].message.content +=
      '\n\n---\n🤝 I\'ve flagged this for our priority support team. Someone will follow up within 30 minutes.';
  }

  return response;
};
```

**Use cases:** Help desk automation, FAQ bots, ticket triage.

---

## 6. Language Tutor

Add multi-language support and difficulty-level adaptation.

```typescript
import type { Middleware } from './types.js';

export const languageTutorMiddleware: Middleware = async (req, next) => {
  const level = 'intermediate'; // Could be dynamic from user profile
  const targetLanguage = 'Spanish';

  req.prompt = `You are a ${targetLanguage} language tutor at ${level} level. 
- Respond primarily in ${targetLanguage}
- Provide English translations in parentheses for new vocabulary
- Correct any grammar mistakes in the user's ${targetLanguage}
- Keep responses at ${level} reading level

Student: ${req.prompt}`;

  return next();
};
```

**Use cases:** Language learning, translation practice, cultural context explanations.

---

## 7. Data Extraction Pipeline

Parse unstructured text into structured data.

```typescript
import type { Middleware } from './types.js';

export const extractionMiddleware: Middleware = async (req, next) => {
  req.prompt = `Extract structured data from the following text. Return ONLY valid JSON with these fields:
- entities: [{name, type, mentions}]
- dates: [{date, context}]
- relationships: [{source, target, type}]
- summary: string

Text: ${req.prompt}`;

  return next();
};
```

**Use cases:** Invoice processing, resume parsing, news article summarization.

---

## 8. Multi-Step Workflow Orchestrator

Chain multiple middleware functions together for complex workflows.

```typescript
import { InMemoryRouter } from './router.js';
import { errorHandlerMiddleware } from './middleware.js';

const router = new InMemoryRouter({
  openaiApiKey: process.env.OPENAI_API_KEY,
  defaultModel: 'gpt-4',
  temperature: 0.7,
});

// Chain middleware in order
router.use(loggingMiddleware);       // Log all requests
router.use(authMiddleware);          // Validate API keys/tokens
router.use(rateLimitMiddleware);     // Rate limiting
router.use(contextInjector);         // Inject system context
router.use(responseFormatter);       // Format output
router.use(errorHandlerMiddleware);  // Catch errors last
```

**Use cases:** Enterprise-grade AI pipelines with logging, auth, rate limiting, and formatting.

---

## The Architecture Pattern

Every use case above follows the same pattern:

```typescript
export const myMiddleware: Middleware = async (req, next) => {
  // 1. PRE-PROCESS: Modify the request before it hits the AI
  req.prompt = `Custom instructions... ${req.prompt}`;

  // 2. PASS to the next middleware or AI model
  const response = await next();

  // 3. POST-PROCESS: Modify the response before returning to the user
  response.choices[0].message.content += '\n---\nCustom footer';

  return response;
};
```

This is the same architecture that powers **Daedalus** — the full-featured AI coding assistant built on this exact template. Daedalus extends this pattern with multi-agent orchestration, codebase indexing, session management, and 16+ built-in tools.

**Daedalus-Lite gives you the foundation. What you build on top is up to you.**
