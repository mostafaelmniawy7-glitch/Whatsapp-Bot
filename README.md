# AI WhatsApp Business Bot

An intelligent WhatsApp chatbot that receives customer messages, classifies them with AI, auto-replies, and manages customer conversations at scale.

![Status](https://img.shields.io/badge/status-ready--for--deployment-brightgreen)
![n8n](https://img.shields.io/badge/n8n-workflow-orange)
![OpenAI](https://img.shields.io/badge/OpenAI-GPT--4o--mini-blue)
![Supabase](https://img.shields.io/badge/Supabase-PostgreSQL-green)
![WhatsApp](https://img.shields.io/badge/WhatsApp-Cloud%20API-success)

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Database Schema](#database-schema)
- [Setup](#setup)
- [API](#api)
- [Use Cases](#use-cases)
- [Screenshots](#screenshots)
- [Author](#author)

---

## Overview

An end-to-end WhatsApp automation system that transforms a standard WhatsApp Business number into an intelligent assistant. Every incoming message is analyzed by AI, classified by intent, and answered automatically.

Built with **n8n**, **Meta WhatsApp Cloud API**, **OpenAI GPT-4o-mini**, and **Supabase**.

---

## Features

### AI-Powered Message Analysis
- Intent detection (order, question, complaint, greeting)
- Sentiment analysis (positive, neutral, negative, angry)
- Auto-generated replies in the customer's language
- Human handoff detection for angry customers

### Automated Replies
- Instant responses 24/7
- Context-aware answers
- Multi-language support
- Short, friendly, and professional tone

### Message Persistence
- Full conversation history in Supabase
- Session tracking per phone number
- Direction tracking (inbound / outbound)

### WhatsApp Cloud API Integration
- Webhook verification with Meta
- Real-time message ingestion
- Automated message sending

---

## System Architecture

## 1. WhatsApp Cloud API Integration
Meta's WhatsApp Cloud API sends real-time webhooks to the n8n endpoint whenever a customer sends a message to the business number. The API also handles outbound messages and webhook verification.

## 2. Webhook Verification
A dedicated GET webhook handles Meta's verification challenge. When Meta sends `hub.mode=subscribe` with a `hub.challenge` value, the system responds with the same challenge to confirm ownership.

## 3. Message Parsing
A Code node extracts structured data from Meta's webhook payload: phone number, sender name, message text, message ID, and message type. It validates the payload and skips malformed requests.

## 4. Message Validation
An IF node checks whether the parsed payload contains a valid message. Empty or malformed events (such as status updates) are filtered out to prevent unnecessary processing.

## 5. Persistence
Valid incoming messages are stored in Supabase's `bot_whatsapp_messages` table with full metadata including phone number, sender name, message text, direction, and business identifier.

## 6. AI Analysis
OpenAI GPT-4o-mini analyzes the customer message and returns structured JSON containing:
- Intent classification (order / question / complaint / greeting)
- Sentiment analysis (positive / neutral / negative / angry)
- Auto-generated reply in the customer's language
- Human handoff flag for escalated conversations

## 7. Response Parsing
A Code node normalizes the AI response into a flat object, merging it with the original message metadata for consistent downstream processing.

## 8. Outbound Message
An HTTP Request node sends the AI-generated reply back to the customer via Meta's Cloud API endpoint (`/messages`) using the configured Phone Number ID and Permanent Access Token.

## 9. External Services
- **WhatsApp Cloud API** — Message ingestion and delivery
- **OpenAI API** — Intent analysis and reply generation
- **Supabase (PostgreSQL)** — Message and session persistence

## Data Flow
WhatsApp Cloud API → Webhook → Is Verification? → Parse Message → Validate → Save to Supabase → AI Analyzer → Parse Response → Send Reply via Cloud API
