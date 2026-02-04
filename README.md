# Women's Soccer Hub

> Empowering Female Soccer Players

This variant focuses on creating a comprehensive resource platform for female soccer players, coaches, and teams. It includes features such as training plans, nutrition advice, and mental preparation techniques, as well as a community forum for discussion and networking. By providing valuable resources and support, Women's Soccer Hub aims to empower female soccer players to improve their performance and achieve their goals.

## Features

- Training plans and workout routines
- Nutrition and wellness advice
- Mental preparation techniques and coaching

## Tech Stack

- **Framework:** Next.js 14 (App Router)
- **Database:** Supabase (PostgreSQL)
- **Auth:** Supabase Auth
- **Styling:** Tailwind CSS
- **Language:** TypeScript

## Getting Started

1. Clone this repository
2. Copy `.env.example` to `.env.local` and fill in your credentials
3. Run `npm install`
4. Run `npm run dev`

## Project Structure

```
├── app/                  # Next.js App Router pages
├── components/           # React components
├── lib/                  # Utilities and helpers
├── supabase/            # Database schema
└── INSTRUCTIONS.md      # Detailed build guide for AI assistants
```

## Database

This project uses 2 main entities:
- **User**: A registered user of the platform
- **Post**: A discussion post created by a user

## Build Instructions

For detailed step-by-step build instructions, see [`INSTRUCTIONS.md`](./INSTRUCTIONS.md).

This file contains comprehensive guidance for building this project with AI coding assistants like Claude Code, Cursor, or Windsurf.

---

*Generated with [Claudery](https://claudery.io) - AI-powered blueprint generator*
