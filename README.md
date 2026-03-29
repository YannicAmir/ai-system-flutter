# Agent System for Flutter Mobile Apps

## Intro

Hey, I'm Yannic 👋 -- I build agentic systems for production apps shipped to real users. Specifically, I focus on two areas when building Agentic Systems:

1. Production & enterprise grade apps (mobile, web, backend, and serverless functions)
2. User-facing features that require AI (think: chat, analyzing docs to provide summaries to users, etc.)

This repo contains three agentic systems, all with the same goal: building Flutter mobile apps.

## Background

The goal is to build full-featured Flutter mobile apps at the speed of AI — reducing development costs while maintaining enterprise-grade quality. These agentic systems allow a team of AI engineers to collaborate on the same Flutter app(s) and ensure consistent, high-quality output.

## What's Covered

Specialized agents and subagents cover just about everything needed for a production-grade mobile app:

- Presentation (UI, Analytics, L10n, Accessibility, etc.)
- Domain
- Data
- Testing
- QA
- Bug support

## Car Factory Analogy

I like to look at Agentic Systems like car factories: raw inputs (materials, plans, requests) go in, and a finished car comes out. In between, humans and machines each play specific roles, completing processes and subprocesses. You might also have multiple factories so that if one goes down, production continues.

This system works the same way. Inputs (Figma mocks, prompts, etc.) go in, and Flutter code comes out. Custom agents each have defined responsibilities -- for example, the UI Builder responsible for implementing the UI from a given Figma mock (via connecting to Figma's MCP server). The `.github`, `.agents`, and `.claude` directories act as separate "factories" that all build the same app, so if you hit a session limit in one (e.g., Claude Code), you can seamlessly switch to another.

## Upcoming

Once I finish the user-facing AI features ( chat and SEC filing analyzer), I may open-source the agentic system I use for my serverless function app in a separate repo. That system covers:

- RAG and vector databases (Pinecone)
- Agentic system observability (LangGraph, LangSmith, LangChain)
- Agentic system evals and CI/CD (DeepEval / Confident AI)

Stack: Node.js, TypeScript, and Python.
