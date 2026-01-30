# AI Interview Simulator

An **AI-powered** frontend application that simulates a real technical interview environment using timed questions, adaptive difficulty, and automatic scoring. It helps students and job seekers practice under realistic constraints for HR and technical rounds.

## Features

- Interactive multi-round interview flow (Home → Interview → Result).
- Timed questions with visual countdown using a reusable `Timer` component.
- Dynamic progress visualization via `ProgressBar`.
- Difficulty-adjusted questioning using custom interview logic.
- Mock AI questions stored in structured `interview.data.js`.
- Automatic scoring and detailed result summary with feedback.
- Global state management for interview session using React Context.
- Clean, modular React architecture with reusable UI components and custom hooks.

## Tech Stack

- React (component-based UI)
- React Router (client-side routing)
- Custom React Hooks (`useTimer`, `useInterview`)
- Context API for global state
- Plain CSS modules and global styles

## Project Structure

```bash
src/
├── assets/
│   └── icons/
├── components/
│   ├── Button/
│   │   ├── Button.jsx
│   │   └── Button.css
│   ├── Timer/
│   │   ├── Timer.jsx
│   │   └── Timer.css
│   ├── ProgressBar/
│   │   ├── ProgressBar.jsx
│   │   └── ProgressBar.css
├── features/
│   └── interview/
│       ├── Interview.jsx
│       ├── QuestionCard.jsx
│       ├── ResultSummary.jsx
│       ├── AnalyticsDashboard.jsx
│       ├── interview.data.js
│       ├── interview.logic.js
│       ├── interview.styles.css
├── hooks/
│   ├── useTimer.js
│   ├── useInterview.js
├── pages/
│   ├── Home.jsx
│   ├── InterviewPage.jsx
│   └── ResultPage.jsx
├── routes/
│   └── AppRoutes.jsx
├── context/
│   └── InterviewContext.jsx
├── utils/
│   ├── shuffleArray.js
│   └── calculateScore.js
├── styles/
│   └── global.css
├── App.jsx
└── main.jsx
```

- `components/`: Reusable UI building blocks (buttons, timers, progress bars).
- `features/interview/`: Core interview experience, data, and logic.
- `hooks/`: Reusable logic for timer and interview state.
- `pages/`: Top-level pages mapped to routes.
- `routes/`: Central route configuration.
- `context/`: Global interview state provider.
- `utils/`: Helper functions for randomness and scoring.
- `styles/`: Global styling.

## Architecture Flow

```
User Action
   ↓
Interview.jsx (Feature Container)
   ↓
useInterview (custom hook)
   ↓
interview.logic.js (AI Brain 🧠)
   ↓
State update → UI re-render
```

## How It Works (Logic Overview)

- `interview.data.js` contains a pool of categorized questions (e.g., easy/medium/hard, HR/technical).
- `useInterview` manages the interview state: current question, score, difficulty adjustments, and navigation between questions.
- `useTimer` controls countdown for each question and triggers auto-submit or move to next question when time expires.
- `calculateScore.js` computes final score and feedback based on correct answers and difficulty.
- `ResultSummary.jsx` shows total score, time taken, strengths, and weak areas.
- `AnalyticsDashboard.jsx` displays detailed analytics including question-by-question breakdown.

## Core Logic Explanation

### interview.data.js (Question Bank)

```js
export const questions = {
  frontend: [
    {
      id: 1,
      level: "easy",
      question: "What is React?",
      options: ["Library", "Framework", "Language", "Browser"],
      answer: "Library",
    },
    {
      id: 2,
      level: "medium",
      question: "What is useEffect used for?",
      options: ["Rendering", "Side effects", "Styling", "Routing"],
      answer: "Side effects",
    },
  ],
  // backend, dsa, etc. can be added similarly
};
```

### interview.logic.js (AI Brain)

```js
// Decides next question difficulty based on current score
export function getNextDifficulty(score) {
  if (score < 40) return "easy";
  if (score < 70) return "medium";
  return "hard";
}

// Calculates overall score in percentage
export function calculateScore(correct, total) {
  return Math.round((correct / total) * 100);
}
```

### useInterview.js (Heart of the App)

```js
import { useState } from "react";
import { questions } from "../features/interview/interview.data";
import { calculateScore } from "../utils/calculateScore";

export default function useInterview() {
  const [current, setCurrent] = useState(0);
  const [correct, setCorrect] = useState(0);

  const questionList = questions.frontend;

  function submitAnswer(selected) {
    if (selected === questionList[current].answer) {
      setCorrect(prev => prev + 1);
    }
    setCurrent(prev => prev + 1);
  }

  const score = calculateScore(correct, questionList.length);

  return {
    current,
    question: questionList[current],
    submitAnswer,
    score,
    isFinished: current >= questionList.length,
  };
}
```

### Interview.jsx (Feature Container)

```jsx
import useInterview from "../../hooks/useInterview";
import QuestionCard from "./QuestionCard";
import ResultSummary from "./ResultSummary";
import AnalyticsDashboard from "./AnalyticsDashboard";

function Interview() {
  const { question, submitAnswer, isFinished, score } = useInterview();

  if (isFinished)
    return (
      <>
        <ResultSummary score={score} />
        <AnalyticsDashboard />
      </>
    );

  return <QuestionCard question={question} onAnswer={submitAnswer} />;
}

export default Interview;
```

### QuestionCard.jsx (with Auto-submit Timer)

```jsx
import useTimer from "../../hooks/useTimer";

function QuestionCard({ question, onAnswer }) {
  const { timeLeft, resetTimer } = useTimer(15, () => {
    onAnswer(null); // auto-submit when time is over
  });

  function handleClick(option) {
    onAnswer(option);
    resetTimer();
  }

  return (
    <div className="question-card">
      <p>⏱️ {timeLeft}s</p>
      <h2>{question.question}</h2>

      {question.options.map(opt => (
        <button key={opt} onClick={() => handleClick(opt)}>
          {opt}
        </button>
      ))}
    </div>
  );
}

export default QuestionCard;
```

### InterviewContext.jsx (Global State)

```jsx
import { createContext, useContext, useReducer } from "react";

const InterviewContext = createContext();

const initialState = {
  currentQuestion: 0,
  correctAnswers: 0,
  answers: [],
};

function interviewReducer(state, action) {
  switch (action.type) {
    case "SUBMIT_ANSWER":
      return {
        ...state,
        currentQuestion: state.currentQuestion + 1,
        correctAnswers: action.payload.isCorrect
          ? state.correctAnswers + 1
          : state.correctAnswers,
        answers: [...state.answers, action.payload],
      };

    case "RESET":
      return initialState;

    default:
      return state;
  }
}

export function InterviewProvider({ children }) {
  const [state, dispatch] = useReducer(interviewReducer, initialState);

  return (
    <InterviewContext.Provider value={{ state, dispatch }}>
      {children}
    </InterviewContext.Provider>
  );
}

export function useInterviewContext() {
  return useContext(InterviewContext);
}
```

## Getting Started

### Prerequisites

- Node.js and npm installed on your machine.

### Installation

```bash
# clone the repository
git clone https://github.com/nitinmehra1017/ai-interview-simulator.git

cd ai-interview-simulator

# install dependencies
npm install
```

### Run in development mode

```bash
npm run dev
```

Then open the URL shown in the terminal (for Vite it is usually `http://localhost:5173`).

### Build for production

```bash
npm run build
```

### Preview production build (optional)

```bash
npm run preview
```

## Possible Enhancements

- Integrate a real LLM backend API for dynamic question generation and feedback.
- Add user authentication and history of past interviews.
- Support multiple domains (DSA, web dev, DBMS, OS, HR).
- Export performance reports as PDF for students.
- Add voice recognition for verbal answers.
- Include video recording for mock interviews.

## Final Year Project Notes

This application was built as a final-year project titled **"AI Interview Simulator"**, focusing on improving interview readiness using simulated AI-based assessments, timed sessions, and analytics. It demonstrates skills in React, clean code architecture, state management, and basic AI-driven logic design.

## License

This project is available for educational purposes.

## Author

**Nitin Mehra** - [GitHub](https://github.com/nitinmehra1017)
