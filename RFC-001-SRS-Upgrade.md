# RFC 001: Spaced Repetition System (SRS) Upgrade

## Summary
Upgrade the simple flashcard app to a smart Spaced Repetition System (SRS). The app will track user progress per word and prioritize "hard/mistaken" words while reducing the frequency of "mastered" words. It will also randomize word presentation order.

## Goals
1.  **Randomized Learning**: Break the fixed sequential order of words.
2.  **Mistake Focus**: Words marked as "Unknown" or answered incorrectly in quizzes appear more frequently.
3.  **Progress Tracking**: Mark words as "Mastered" to reduce their frequency.
4.  **Persistence**: Save learning progress to `localStorage` so data survives page reloads.

## Technical Design

### 1. Data Structure Extension
The `iseeVocabulary` array remains the source of truth (read-only).
A new state object `userProgress` will be introduced and stored in `localStorage`:

```javascript
const userProgress = {
  "Abandon": { status: "new", nextReview: 0, streak: 0 },
  "Abundant": { status: "mastered", nextReview: 1709823322, streak: 3 },
  // ... key is the word string
};
```

### 2. Status Lifecycle
*   **NEW**: Default state. High priority to introduce.
*   **LEARNING**: User clicked "Hard/Again" or got a quiz wrong. Very High priority.
*   **MASTERED**: User clicked "Easy/Good" or got quiz right. Low priority (only shows when `nextReview` timestamp expires).

### 3. Algorithm: "Next Word" Selection
Instead of `index++`, the `getNextWord()` function will:
1.  **Filter**: Find all candidates eligible for review (Status is NEW/LEARNING, or MASTERED but `nextReview` time has passed).
2.  **Weighting**:
    *   `LEARNING`: Weight 100
    *   `NEW`: Weight 50
    *   `MASTERED` (Review due): Weight 20
3.  **Random Pick**: Weighted random selection from candidates.

### 4. UI Changes
*   **Learn View**:
    *   Replace single "Next Word" button with two buttons:
        *   ❌ **Hard / Forgot** (Sets status to `LEARNING`, resets streak)
        *   ✅ **Easy / Knew it** (Sets status to `MASTERED`, increments streak, sets future review time)
*   **Quiz Views**:
    *   Correct answer -> Promotes word towards `MASTERED`.
    *   Wrong answer -> Demotes word to `LEARNING` (High Priority).

### 5. Implementation Plan
1.  Add `ProgressManager` class to handle storage and logic.
2.  Refactor `renderFlashcard` to use `ProgressManager.getNextWord()`.
3.  Update UI buttons in "Learn" view.
4.  Hook Quiz results into `ProgressManager.updateWord(word, isCorrect)`.

## Future Considerations
- Export/Import progress (JSON).
- Daily goals visualization.
