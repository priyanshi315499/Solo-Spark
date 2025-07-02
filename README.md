// App.js
import React, { createContext, useContext, useState } from 'react'
import ReactDOM from 'react-dom/client'

// --- Context & Quest Logic ---
const QuestContext = createContext()
const useQuest = () => useContext(QuestContext)

function generateDailyQuest(profile) {
  let description = 'Reflect on your day in writing'
  let pts = 10
  if (profile.mood === 'stressed') {
    description = 'Take 5 minutes to breathe deeply and note how you feel'
    pts = 15
  } else if (profile.mood === 'happy') {
    description = 'Do one kind thing for someone and write about it'
    pts = 12
  }
  return { id: Date.now(), description, points: pts }
}

// --- Main App Provider ---
function QuestProvider({ children }) {
  const [profile, setProfile] = useState({ mood: null })
  const [dailyQuest, setDailyQuest] = useState(null)
  const [points, setPoints] = useState(0)

  const updateProfile = (newProfile) => {
    const updated = { ...profile, ...newProfile }
    setProfile(updated)
    const quest = generateDailyQuest(updated)
    setDailyQuest(quest)
  }

  const completeQuest = (reflection) => {
    console.log('Reflection:', reflection)
    setPoints((p) => p + (dailyQuest?.points || 0))
    setDailyQuest(null)
  }

  return (
    <QuestContext.Provider value={{
      profile,
      dailyQuest,
      points,
      updateProfile,
      completeQuest
    }}>
      {children}
    </QuestContext.Provider>
  )
}

// --- Components ---
function MoodSurvey() {
  const [mood, setMood] = useState('')
  const { updateProfile } = useQuest()

  const submit = (e) => {
    e.preventDefault()
    updateProfile({ mood })
  }

  return (
    <form onSubmit={submit} style={styles.form}>
      <h2>How are you feeling today?</h2>
      <select value={mood} onChange={e => setMood(e.target.value)} required style={styles.select}>
        <option value="">Select mood</option>
        <option value="happy">😊 Happy</option>
        <option value="stressed">😣 Stressed</option>
        <option value="neutral">😐 Neutral</option>
      </select>
      <button type="submit" style={styles.button}>Get your quest</button>
    </form>
  )
}

function ReflectionForm() {
  const [text, setText] = useState('')
  const { completeQuest, dailyQuest } = useQuest()

  const submit = e => {
    e.preventDefault()
    const reflection = { text, questId: dailyQuest.id, timestamp: Date.now() }
    completeQuest(reflection)
    setText('')
  }

  return (
    <form onSubmit={submit} style={styles.form}>
      <h4>Reflect on your quest:</h4>
      <textarea
        value={text} onChange={e => setText(e.target.value)}
        placeholder="Your thoughts..." required style={styles.textarea}
      />
      <button type="submit" style={styles.button}>
        Submit Reflection (Earn {dailyQuest.points} pts)
      </button>
    </form>
  )
}

function QuestView() {
  const { dailyQuest } = useQuest()
  if (!dailyQuest) return <p>No quest yet for today. Start with the mood survey above.</p>

  return (
    <div style={styles.questBox}>
      <h3>Today's Quest:</h3>
      <p>{dailyQuest.description}</p>
      <ReflectionForm />
    </div>
  )
}

function PointsDisplay() {
  const { points } = useQuest()
  return (
    <div style={styles.points}>
      Your Spark Points: <strong>{points}</strong>
    </div>
  )
}

// --- Main App ---
function App() {
  return (
    <QuestProvider>
      <div style={styles.container}>
        <h1>🌱 Personal Growth Quest</h1>
        <PointsDisplay />
        <MoodSurvey />
        <QuestView />
      </div>
    </QuestProvider>
  )
}

// --- Inline Styles ---
const styles = {
  container: {
    maxWidth: 600, margin: '0 auto', padding: 20, fontFamily: 'sans-serif'
  },
  form: { marginBottom: 20 },
  select: { display: 'block', width: '100%', padding: 8, margin: '8px 0' },
  textarea: { width: '100%', height: 80, padding: 8, margin: '8px 0' },
  button: { padding: '10px 20px', backgroundColor: '#4caf50', color: '#fff', border: 'none', cursor: 'pointer' },
  questBox: { background: '#f9f9f9', padding: 15, borderRadius: 5, marginBottom: 20 },
  points: { marginBottom: 10, fontSize: 18 }
}

// --- Render App ---
const root = ReactDOM.createRoot(document.getElementById('root'))
root.render(<App />)
