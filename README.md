# word-file
단어장
import { useState, useEffect } from "react";

export default function WordFileApp() {
  const [view, setView] = useState("wordbook");
  const [wordList, setWordList] = useState([]);
  const [word, setWord] = useState("");
  const [meaning, setMeaning] = useState("");
  const [showMeaning, setShowMeaning] = useState(false);
  const [currentIndex, setCurrentIndex] = useState(0);
  const [testInput, setTestInput] = useState("");
  const [testResult, setTestResult] = useState(null);
  const [mode, setMode] = useState("en-to-kr");
  const [editingIndex, setEditingIndex] = useState(null);

  const [wrongList, setWrongList] = useState(() => {
    const saved = localStorage.getItem("wrongList");
    return saved ? JSON.parse(saved) : [];
  });
  const [wrongIndex, setWrongIndex] = useState(0);
  const [wrongInput, setWrongInput] = useState("");
  const [wrongResult, setWrongResult] = useState(null);
  const [showWrongMeaning, setShowWrongMeaning] = useState(false);

  useEffect(() => {
    localStorage.setItem("wrongList", JSON.stringify(wrongList));
  }, [wrongList]);

  const addWord = () => {
    if (word && meaning) {
      setWordList([...wordList, { word, meaning }]);
      setWord("");
      setMeaning("");
    }
  };

  const updateWord = () => {
    const newList = [...wordList];
    newList[editingIndex] = { word, meaning };
    setWordList(newList);
    setWord("");
    setMeaning("");
    setEditingIndex(null);
  };

  const deleteWord = (index) => {
    const newList = [...wordList];
    newList.splice(index, 1);
    setWordList(newList);
    if (currentIndex >= newList.length) {
      setCurrentIndex(0);
    }
  };

  const editWord = (index) => {
    setEditingIndex(index);
    setWord(wordList[index].word);
    setMeaning(wordList[index].meaning);
  };

  const toggleMeaning = () => {
    setShowMeaning(!showMeaning);
  };

  const nextWord = () => {
    setShowMeaning(false);
    setTestResult(null);
    setTestInput("");
    setCurrentIndex((currentIndex + 1) % wordList.length);
  };

  const handleTest = () => {
    const correctAnswer = mode === "en-to-kr"
      ? wordList[currentIndex].meaning.trim()
      : wordList[currentIndex].word.trim();

    if (testInput.trim().toLowerCase() === correctAnswer.toLowerCase()) {
      setTestResult("정답!");
    } else {
      setTestResult(`오답! 정답: ${correctAnswer}`);
      const wrong = wordList[currentIndex];
      setWrongList([...wrongList, { ...wrong, correct: 0, total: 1 }]);
    }
  };

  const clearWrongList = () => {
    setWrongList([]);
    localStorage.removeItem("wrongList");
    setWrongIndex(0);
    setWrongInput("");
    setWrongResult(null);
  };

  const testWrongList = () => {
    const current = wrongList[wrongIndex];
    const correct = mode === "en-to-kr" ? current.meaning.trim() : current.word.trim();
    const isCorrect = wrongInput.trim().toLowerCase() === correct.toLowerCase();

    const newList = [...wrongList];
    if (!newList[wrongIndex].total) newList[wrongIndex].total = 0;
    if (!newList[wrongIndex].correct) newList[wrongIndex].correct = 0;
    newList[wrongIndex].total++;
    if (isCorrect) newList[wrongIndex].correct++;
    setWrongList(newList);

    setWrongResult(isCorrect ? "정답!" : `오답! 정답: ${correct}`);
  };

  const nextWrong = () => {
    setWrongIndex((wrongIndex + 1) % wrongList.length);
    setWrongInput("");
    setWrongResult(null);
    setShowWrongMeaning(false);
  };

  const deleteWrong = (index) => {
    const newList = [...wrongList];
    newList.splice(index, 1);
    setWrongList(newList);
    if (wrongIndex >= newList.length) {
      setWrongIndex(0);
    }
  };

  const speak = (text) => {
    const utterance = new SpeechSynthesisUtterance(text);
    utterance.lang = "en-US";
    utterance.rate = 0.9;
    speechSynthesis.speak(utterance);
  };

  const currentItem = wordList[currentIndex];
  const currentWrongItem = wrongList[wrongIndex];

  return (
    <div style={{ maxWidth: "600px", margin: "auto", padding: "1rem" }}>
      <h1>📁 워드 파일</h1>

      <div style={{ marginBottom: "1rem" }}>
        <button onClick={() => setView("wordbook")}>단어장</button>
        <button onClick={() => setView("wrongnote")} style={{ marginLeft: "1rem" }}>오답노트</button>
      </div>

      {view === "wordbook" && (
        <div>
          <div style={{ marginBottom: "1rem" }}>
            <input
              type="text"
              placeholder="단어"
              value={word}
              onChange={(e) => setWord(e.target.value)}
            />
            <input
              type="text"
              placeholder="뜻"
              value={meaning}
              onChange={(e) => setMeaning(e.target.value)}
            />
            {editingIndex !== null ? (
              <button onClick={updateWord}>수정</button>
            ) : (
              <button onClick={addWord}>추가</button>
            )}
          </div>

          {wordList.length > 0 && currentItem && (
            <div style={{ border: "1px solid #ccc", padding: "1rem", borderRadius: "8px" }}>
              <div style={{ fontSize: "1.2rem", marginBottom: "0.5rem" }}>
                <b>현재 모드:</b> {mode === "en-to-kr" ? "영어 → 뜻" : "뜻 → 영어"}
              </div>
              <button onClick={() => setMode(mode === "en-to-kr" ? "kr-to-en" : "en-to-kr")}>모드 전환</button>

              <div style={{ fontSize: "1.5rem", marginTop: "1rem" }}>
                {mode === "en-to-kr" ? currentItem.word : currentItem.meaning}
              </div>

              {mode === "en-to-kr" && showMeaning && (
                <div style={{ fontSize: "1.2rem", margin: "0.5rem 0" }}>{currentItem.meaning}</div>
              )}

              {mode === "en-to-kr" && (
                <>
                  <button onClick={toggleMeaning}>뜻 보기</button>
                  <button onClick={() => speak(currentItem.word)}>발음 듣기</button>
                </>
              )}

              <button onClick={nextWord}>다음</button>

              <div style={{ marginTop: "1rem" }}>
                <input
                  type="text"
                  placeholder={mode === "en-to-kr" ? "뜻 입력" : "단어 입력"}
                  value={testInput}
                  onChange={(e) => setTestInput(e.target.value)}
                />
                <button onClick={handleTest}>제출</button>
                {testResult && <div>{testResult}</div>}
              </div>

              <button onClick={() => deleteWord(currentIndex)} style={{ marginTop: "1rem", background: "#f44336", color: "white" }}>
                단어 삭제하기
              </button>
              <button onClick={() => editWord(currentIndex)} style={{ marginTop: "1rem", marginLeft: "0.5rem" }}>수정</button>
            </div>
          )}
        </div>
      )}

      {view === "wrongnote" && wrongList.length > 0 && (
        <div style={{ marginTop: "2rem", background: "#fffae6", padding: "1rem", borderRadius: "8px" }}>
          <h2 style={{ color: "red" }}>❗ 오답 노트</h2>
          <ul>
            {wrongList.map((item, index) => (
              <li key={index}>
                {item.word} - {item.meaning} (정답률: {item.total ? Math.round((item.correct || 0) / item.total * 100) : 0}%)
                <button onClick={() => deleteWrong(index)} style={{ marginLeft: "1rem" }}>삭제</button>
              </li>
            ))}
          </ul>

          {currentWrongItem && (
            <div style={{ marginTop: "1rem" }}>
              <b>오답 퀴즈:</b>
              <div style={{ fontSize: "1.2rem", margin: "0.5rem 0" }}>
                {mode === "en-to-kr" ? currentWrongItem.word : currentWrongItem.meaning}
              </div>

              {mode === "en-to-kr" && showWrongMeaning && (
                <div style={{ fontSize: "1.2rem", margin: "0.5rem 0" }}>{currentWrongItem.meaning}</div>
              )}

              {mode === "en-to-kr" && (
                <>
                  <button onClick={() => setShowWrongMeaning(!showWrongMeaning)}>뜻 보기</button>
                  <button onClick={() => speak(currentWrongItem.word)}>발음 듣기</button>
                </>
              )}

              <input
                type="text"
                placeholder={mode === "en-to-kr" ? "뜻 입력" : "단어 입력"}
                value={wrongInput}
                onChange={(e) => setWrongInput(e.target.value)}
              />
              <button onClick={testWrongList}>제출</button>
              <button onClick={nextWrong}>다음</button>
              {wrongResult && <div>{wrongResult}</div>}
            </div>
          )}

          <button onClick={clearWrongList} style={{ marginTop: "1rem", background: "#2196f3", color: "white" }}>오답노트 비우기</button>
        </div>
      )}
    </div>
  );
}
