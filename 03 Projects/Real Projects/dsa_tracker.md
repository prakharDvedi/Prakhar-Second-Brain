# dsa_tracker

*Part of [[Projects]] → [[Real Projects]]*

**Repo:** [dsa_tracker](https://github.com/prakharDvedi/dsa_tracker)
**Visibility:** Public
**Language:** TypeScript

---

# DSA Tracker
https://dsa-ka-tracker.vercel.app/
## What It Is

DSA Tracker is a web application designed to help developers track their Data Structures and Algorithms (DSA) practice. It serves as a personal log for coding problems sourced from various platforms like LeetCode, HackerRank, and Codeforces. The application allows users to record their solution attempts, monitor their consistency, and visualize their progress over time.

<img width="1898" height="979" alt="image" src="https://github.com/user-attachments/assets/a34b0905-7d70-4b8d-a5bc-488450b6c4df" />

## Tech Stack

The project is built using a modern full-stack implementation:

- **Framework**: Next.js 14 (App Router)
- **Language**: TypeScript
- **Styling**: Tailwind CSS
- **Database**: SQLite
- **ORM**: Prisma
- **Visualization**: Recharts

## How It Works

The application revolves around two main entities: **Problems** and **Attempts**.

1. **Dashboard**: Provides a snapshot of your progress, including total problems solved, success rate, and visual charts for difficulty distribution and activity history.
2. **Problem Management**: You can add new problems to the tracker, specifying the title, difficulty level, platform, and related topics. These can be searched and filtered.
3. **Logging Attempts**: For every problem, you can log multiple attempts. Each attempt captures details such as time taken, status (Success/Failure), and notes on your approach or mistakes.
4. **Analysis**: The app aggregates your attempt data to calculate statistics, helping you identify weak areas (e.g., specific topics or difficulty levels).

## Structure and Routing

This project follows the Next.js App Router directory structure:

### File Structure

- `app/`: Contains the main application routes and UI components.
- `app/api/`: Hosts the backend logic and API endpoints.
- `lib/`: Utility functions and shared configuration (e.g., Prisma client).
- `prisma/`: Database schema definitions and seed scripts.

### Routing

- `/` or `/dashboard`: The main dashboard view with statistics.
- `/problems`: A searchable list of all tracked problems.
- `/problems/new`: Failure-safe form to create a new problem entry.
- `/problems/[id]`: Detailed view of a specific problem and its history.
- `/problems/[id]/attempt`: Interface to log a new practice attempt for a problem.
