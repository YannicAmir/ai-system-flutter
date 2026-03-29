# Agent System for Flutter Mobile Apps

## Intro

Hey, I'm Yannic 👋 -- I build agentic systems for products to be used in production with real users. Particulary, I build agentic systems for production & enterprise grade apps (mobile, web, backend, serverless function) and user facing features that require AI (think chat, analizing doc to provide summary to user, etc.). This repo contains 3 agentic systems all with the same goal: building Flutter mobile apps.

## Background

The goal is to build a full-featured Flutter mobile app with the speed of AI (which also means reduced dev costs), while maintaining enterprise-grade quality. The agentic systems in this repo scale across a team of AI Engineers.

## What's Covered

There are specialized agents and subagents for just about everything you need for an enterprise/production-grade mobile app:

- Presentation (UI, Analytics, L10n, Accessibility, etc.)
- Domain
- Data
- Testing
- QA
- Bug support

## Car Factory Analogy

I like to look at Agentic Systems like car factories. In a car factory you put in some sort of request, materials, & plans (your inputs) and get out a car (output). In between there are humans and machines, with certain responsibilities, completing processes and subprocesses. Also, you may want to have several factories just in case one or more go down, so you can continue to produce cars. So how does this connect?

This Agentic System takes in inputs (mocks, prompts, etc) and produces code for mobile app (the output or "car"). There are many custom agents that have specific responsibilities -- for instance the UI Builder responsible for implementing the the UI from a given Figma mock (via connecting to Figma's MCP server). There are `.github`, `.agents`, and `.claude` "factories" that all build the same app, just in case you hit a session limit with Claude Code, you can switch over to Antigravity etc. etc.

## Upcoming

Once I finish building out the user-facing AI features (chat, SEC-filing analyzer), I may make the agentic system, that I use to build my serverless function app, public in a separate repo as well. They cover RAG & Vector DBs (with Pinecone), Agentic System observability (with LangGraph, LangSmith, and LangChain), and Agentic System eval & CICD (with DeepEval / Confident AI). Specifically for: Node.js, TypeScript, and Python.
