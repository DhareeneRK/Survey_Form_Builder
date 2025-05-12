# Survey Form Builder (ReactJS)
## Date:

## AIM
To create a Survey Form Builder using ReactJS..

## ALGORITHM
### STEP 1 Initialize States
mode ← 'build' (default mode)

questions ← [] (holds all survey questions)

currentQuestion ← { text: '', type: 'text', options: '' } (holds input while building)

editingIndex ← null (tracks which question is being edited)

responses ← {} (user responses to survey)

submitted ← false (indicates whether survey was submitted)

### STEP 2 Switch Modes
Toggle mode between 'build' and 'fill' when the toggle button is clicked.

Reset submitted to false when entering Fill Mode.

### STEP 3 Build Mode Logic
#### a. Add/Update Question
If currentQuestion.text is not empty:

If type is "radio" or "checkbox", split currentQuestion.options by commas → optionsArray.

Construct a questionObject:

If editingIndex is not null, replace question at that index.

Else, push questionObject into questions.

Reset currentQuestion and editingIndex.

#### b. Edit Question
Set currentQuestion to the selected question’s data.

Set editingIndex to the question’s index.

#### c. Delete Question
Remove the selected question from questions.

### STEP 4 Fill Mode Logic
#### a. Render Form
For each question in questions:

If type is "text", render a text input.

If type is "radio", render radio buttons for each option.

If type is "checkbox", render checkboxes.

#### b. Capture Responses
On input change:

For "text" and "radio", update responses[index] = value.

For "checkbox":

If option exists in responses[index], remove it.

Else, add it to the array.

### STEP 5 Submit Form
When user clicks "Submit":

Set submitted = true.

Display a summary using the questions and responses.

### STEP 6 Display Summary
Iterate over questions.

Display the response from responses[i] for each question:

If it's an array, join it with commas.

If empty, show "No response".


## PROGRAM
---
App.js
import React, { useState } from "react";
import "./App.css";

function App() {
  const [mode, setMode] = useState("build");
  const [questions, setQuestions] = useState([]);
  const [currentQuestion, setCurrentQuestion] = useState({
    text: "",
    type: "text",
    options: "",
  });
  const [editingIndex, setEditingIndex] = useState(null);
  const [responses, setResponses] = useState({});
  const [submitted, setSubmitted] = useState(false);

  const handleAddOrUpdateQuestion = () => {
    if (!currentQuestion.text.trim()) return;

    const newQuestion = {
      text: currentQuestion.text,
      type: currentQuestion.type,
      options:
        currentQuestion.type === "text"
          ? []
          : currentQuestion.options.split(",").map((o) => o.trim()),
    };

    const updatedQuestions = [...questions];
    if (editingIndex !== null) {
      updatedQuestions[editingIndex] = newQuestion;
    } else {
      updatedQuestions.push(newQuestion);
    }

    setQuestions(updatedQuestions);
    setCurrentQuestion({ text: "", type: "text", options: "" });
    setEditingIndex(null);
  };

  const handleEdit = (index) => {
    const q = questions[index];
    setCurrentQuestion({
      text: q.text,
      type: q.type,
      options: q.options.join(", "),
    });
    setEditingIndex(index);
  };

  const handleDelete = (index) => {
    const updated = [...questions];
    updated.splice(index, 1);
    setQuestions(updated);
  };

  const handleResponseChange = (index, value) => {
    setResponses({ ...responses, [index]: value });
  };

  const handleCheckboxChange = (index, option) => {
    const current = responses[index] || [];
    if (current.includes(option)) {
      setResponses({
        ...responses,
        [index]: current.filter((o) => o !== option),
      });
    } else {
      setResponses({
        ...responses,
        [index]: [...current, option],
      });
    }
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    setSubmitted(true);
  };

  return (
    <>
      <div className="container">
        <div className="mode-switch">
          <h1>Survey Form Builder ({mode === "build" ? "Build" : "Fill"} Mode)</h1>
          <button
            onClick={() => {
              setMode(mode === "build" ? "fill" : "build");
              setSubmitted(false);
              setResponses({});
            }}
          >
            Switch to {mode === "build" ? "Fill Mode" : "Build Mode"}
          </button>
        </div>

        {mode === "build" ? (
          <>
            <div>
              <input
                type="text"
                placeholder="Enter question"
                value={currentQuestion.text}
                onChange={(e) =>
                  setCurrentQuestion({ ...currentQuestion, text: e.target.value })
                }
              />
              <select
                value={currentQuestion.type}
                onChange={(e) =>
                  setCurrentQuestion({ ...currentQuestion, type: e.target.value })
                }
              >
                <option value="text">Text</option>
                <option value="radio">Radio</option>
                <option value="checkbox">Checkbox</option>
              </select>
              {(currentQuestion.type === "radio" ||
                currentQuestion.type === "checkbox") && (
                <>
                  <input
                    type="text"
                    placeholder="Options (comma separated)"
                    value={currentQuestion.options}
                    onChange={(e) =>
                      setCurrentQuestion({
                        ...currentQuestion,
                        options: e.target.value,
                      })
                    }
                  />
                  <div className="options-hint">Example: Yes, No, Maybe</div>
                </>
              )}
              <button onClick={handleAddOrUpdateQuestion}>
                {editingIndex !== null ? "Update" : "Add"} Question
              </button>
            </div>

            <div>
              {questions.map((q, index) => (
                <div key={index} className="question-box">
                  <strong>{q.text}</strong> ({q.type})<br />
                  {q.options.length > 0 && (
                    <ul>
                      {q.options.map((opt, i) => (
                        <li key={i}>{opt}</li>
                      ))}
                    </ul>
                  )}
                  <button onClick={() => handleEdit(index)}>Edit</button>
                  <button className="remove-btn" onClick={() => handleDelete(index)}>Delete</button>
                </div>
              ))}
            </div>
          </>
        ) : (
          <>
            {!submitted ? (
              <form onSubmit={handleSubmit}>
                {questions.map((q, index) => (
                  <div key={index} className="question-box">
                    <label>{q.text}</label>
                    {q.type === "text" && (
                      <input
                        type="text"
                        onChange={(e) =>
                          handleResponseChange(index, e.target.value)
                        }
                      />
                    )}
                    {q.type === "radio" &&
                      q.options.map((opt, i) => (
                        <div key={i}>
                          <input
                            type="radio"
                            name={`q-${index}`}
                            value={opt}
                            onChange={(e) =>
                              handleResponseChange(index, e.target.value)
                            }
                          />{" "}
                          {opt}
                        </div>
                      ))}
                    {q.type === "checkbox" &&
                      q.options.map((opt, i) => (
                        <div key={i}>
                          <input
                            type="checkbox"
                            value={opt}
                            checked={(responses[index] || []).includes(opt)}
                            onChange={() => handleCheckboxChange(index, opt)}
                          />{" "}
                          {opt}
                        </div>
                      ))}
                  </div>
                ))}
                <button type="submit">Submit</button>
              </form>
            ) : (
              <div className="response-summary">
                <h2>Survey Summary</h2>
                {questions.map((q, index) => (
                  <div key={index}>
                    <strong>{q.text}</strong>:{" "}
                    {Array.isArray(responses[index])
                      ? responses[index].join(", ")
                      : responses[index] || "No response"}
                  </div>
                ))}
              </div>
            )}
          </>
        )}
      </div>

      <Footer />
    </>
  );
}

const Footer = () => (
  <footer>
    <hr />
    <p>Dhareene R K(212222040035)</p>
  </footer>
);

export default App;

---




---
App.css
body {
  font-family: 'Segoe UI', sans-serif;
  background: #f0f4f8;
  margin: 0;
  padding: 0;
}

.container {
  max-width: 800px;
  margin: 2rem auto;
  background: #fff;
  padding: 2rem;
  border-radius: 12px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
}

h1 {
  margin-bottom: 1rem;
}

input[type="text"],
select {
  display: block;
  width: 100%;
  margin: 0.5rem 0 1rem;
  padding: 0.6rem;
  border-radius: 6px;
  border: 1px solid #ccc;
}

button {
  background-color: #2563eb;
  color: white;
  border: none;
  padding: 0.6rem 1.2rem;
  margin: 0.5rem 0;
  border-radius: 6px;
  cursor: pointer;
}

button:hover {
  background-color: #1d4ed8;
}

.remove-btn {
  background-color: #dc2626;
  margin-left: 0.5rem;
}

.remove-btn:hover {
  background-color: #b91c1c;
}

.mode-switch {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.question-box {
  background-color: #f9fafb;
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 1rem;
  margin-bottom: 1rem;
}

.response-summary {
  background-color: #dbeafe;
  padding: 1rem;
  border-radius: 8px;
  margin-top: 1rem;
}

.options-hint {
  font-size: 0.85rem;
  color: #6b7280;
  margin-top: -0.75rem;
  margin-bottom: 0.5rem;
}
footer {
  margin-top: 2rem;
  padding: 1rem;
  text-align: center;
  color: #6b7280;
  font-size: 0.9rem;
  background-color: #f3f4f6;
  border-top: 1px solid #e5e7eb;
}

---
## OUTPUT

![web 125](https://github.com/user-attachments/assets/3b2755b3-e346-48e6-9698-5fd9a573f982)

## RESULT
The program for creating Survey Form Builder using ReactJS is executed successfully.
