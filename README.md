# Nursing-Learning-Module
Dynamic Learning Module for Nursing Students
import { useEffect, useState } from "react";
import { Card, CardContent } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Textarea } from "@/components/ui/textarea";

const getToday = () => new Date().toISOString().split("T")[0];

export default function LearningModule() {
  const [topics, setTopics] = useState([]);
  const [title, setTitle] = useState("");
  const [content, setContent] = useState("");
  const [currentIndex, setCurrentIndex] = useState(null);
  const [showAnswer, setShowAnswer] = useState(false);
  const [mode, setMode] = useState("review"); // "review" or "quiz"
  const [quizOptions, setQuizOptions] = useState([]);

  useEffect(() => {
    const saved = localStorage.getItem("nursingTopics");
    if (saved) setTopics(JSON.parse(saved));
  }, []);

  const addTopic = () => {
    if (!title || !content) return;
    const newTopic = {
      title,
      content,
      lastReviewed: null,
      interval: 1,
      nextReview: getToday(),
    };
    const updatedTopics = [...topics, newTopic];
    setTopics(updatedTopics);
    localStorage.setItem("nursingTopics", JSON.stringify(updatedTopics));
    setTitle("");
    setContent("");
  };

  const startReview = () => {
    const today = getToday();
    const dueTopics = topics.filter((t) => t.nextReview <= today);
    if (dueTopics.length > 0) {
      setTopics(dueTopics);
      setCurrentIndex(0);
      setMode("review");
      setShowAnswer(false);
    } else {
      alert("No topics due for review today.");
    }
  };

  const handleCorrect = () => {
    const updated = [...topics];
    const topic = updated[currentIndex];
    topic.interval *= 2;
    const nextDate = new Date();
    nextDate.setDate(nextDate.getDate() + topic.interval);
    topic.nextReview = nextDate.toISOString().split("T")[0];
    topic.lastReviewed = getToday();
    localStorage.setItem("nursingTopics", JSON.stringify(updated));
    nextCard();
  };

  const handleIncorrect = () => {
    const updated = [...topics];
    const topic = updated[currentIndex];
    topic.interval = 1;
    topic.nextReview = getToday();
    topic.lastReviewed = getToday();
    localStorage.setItem("nursingTopics", JSON.stringify(updated));
    nextCard();
  };

  const startQuiz = () => {
    if (topics.length < 4) {
      alert("Need at least 4 topics for quiz.");
      return;
    }
    setMode("quiz");
    setCurrentIndex(0);
    generateQuizOptions(0);
  };

  const generateQuizOptions = (index) => {
    const correct = topics[index];
    const others = topics.filter((_, i) => i !== index);
    const shuffled = others.sort(() => 0.5 - Math.random()).slice(0, 3);
    const options = [...shuffled, correct].sort(() => 0.5 - Math.random());
    setQuizOptions(options);
  };

  const nextCard = () => {
    if (topics.length === 0) return;
    const nextIndex = (currentIndex + 1) % topics.length;
    setCurrentIndex(nextIndex);
    setShowAnswer(false);
    if (mode === "quiz") generateQuizOptions(nextIndex);
  };

  return (
    <div className="p-4 max-w-xl mx-auto">
      <h1 className="text-2xl font-bold mb-4">Nursing School Learning Module</h1>

      <div className="mb-6">
        <Input
          placeholder="Topic Title"
          value={title}
          onChange={(e) => setTitle(e.target.value)}
          className="mb-2"
        />
        <Textarea
          placeholder="Topic Content or Explanation"
          value={content}
          onChange={(e) => setContent(e.target.value)}
          className="mb-2"
        />
        <Button onClick={addTopic}>Add Topic</Button>
      </div>

      <div className="mb-6">
        <Button onClick={startReview} className="mr-2">Spaced Review</Button>
        <Button onClick={startQuiz}>Start Quiz</Button>
        <Button onClick={nextCard} className="ml-2">Next</Button>
      </div>

      {currentIndex !== null && mode === "review" && (
        <Card className="mb-4">
          <CardContent className="p-4">
            <h2 className="text-xl font-semibold mb-2">
              {topics[currentIndex].title}
            </h2>
            {showAnswer ? (
              <>
                <p className="mb-4">{topics[currentIndex].content}</p>
                <Button onClick={handleCorrect} className="mr-2">Correct</Button>
                <Button onClick={handleIncorrect}>Incorrect</Button>
              </>
            ) : (
              <Button onClick={() => setShowAnswer(true)}>Show Answer</Button>
            )}
          </CardContent>
        </Card>
      )}

      {currentIndex !== null && mode === "quiz" && (
        <Card className="mb-4">
          <CardContent className="p-4">
            <h2 className="text-xl font-semibold mb-4">
              What is the explanation for: {topics[currentIndex].title}?
            </h2>
            {quizOptions.map((opt, i) => (
              <Button
                key={i}
                className="block w-full mb-2"
                onClick={() => {
                  if (opt.content === topics[currentIndex].content) {
                    alert("Correct!");
                  } else {
                    alert("Incorrect. Try again.");
                  }
                  nextCard();
                }}
              >
                {opt.content}
              </Button>
            ))}
          </CardContent>
        </Card>
      )}
    </div>
  );
}
