// App.jsx
import React, { useEffect, useState } from "react";

export default function App() {
  const [dark, setDark] = useState(false);
  const [logged, setLogged] = useState(false);

  const [user, setUser] = useState({
    name: "",
    email: "",
  });

  const [notes, setNotes] = useState([]);
  const [search, setSearch] = useState("");
  const [category, setCategory] = useState("All");

  const empty = {
    title: "",
    content: "",
    category: "General",
    tags: "",
  };

  const [form, setForm] = useState(empty);

  useEffect(() => {
    const saved = localStorage.getItem("notes-app");

    if (saved) {
      const data = JSON.parse(saved);

      setNotes(data.notes || []);
      setUser(data.user || user);
      setLogged(data.logged || false);
      setDark(data.dark || false);
    }
  }, []);

  useEffect(() => {
    localStorage.setItem(
      "notes-app",
      JSON.stringify({
        notes,
        user,
        logged,
        dark,
      })
    );
  }, [notes, user, logged, dark]);

  const login = () => {
    if (!user.name || !user.email) {
      alert("Fill all fields");
      return;
    }

    setLogged(true);
  };

  const addNote = () => {
    if (!form.title || !form.content) return;

    setNotes([
      {
        id: Date.now(),
        ...form,
        pinned: false,
        date: new Date().toLocaleString(),
      },
      ...notes,
    ]);

    setForm(empty);
  };

  const remove = (id) => {
    setNotes(notes.filter((n) => n.id !== id));
  };

  const pin = (id) => {
    setNotes(
      notes.map((n) =>
        n.id === id
          ? {
              ...n,
              pinned: !n.pinned,
            }
          : n
      )
    );
  };

  const exportNotes = () => {
    const file = new Blob(
      [JSON.stringify(notes, null, 2)],
      {
        type: "application/json",
      }
    );

    const a = document.createElement("a");

    a.href = URL.createObjectURL(file);

    a.download = "notes.json";

    a.click();
  };

  const filtered = notes.filter((n) => {
    const s =
      n.title.toLowerCase().includes(search.toLowerCase()) ||
      n.content.toLowerCase().includes(search.toLowerCase());

    const c =
      category === "All"
        ? true
        : n.category === category;

    return s && c;
  });

  const categories = [
    "All",
    ...new Set(notes.map((n) => n.category)),
  ];

  const styles = {
    bg: dark ? "#0f172a" : "#f6f7fb",
    card: dark ? "#1e293b" : "#fff",
    text: dark ? "#fff" : "#111",
  };

  if (!logged) {
    return (
      <div
        style={{
          background: styles.bg,
          color: styles.text,
          minHeight: "100vh",
          display: "grid",
          placeItems: "center",
        }}
      >
        <div
          style={{
            background: styles.card,
            padding: 30,
            width: 350,
            borderRadius: 20,
          }}
        >
          <h1>Notes Maker</h1>

          <input
            placeholder="Name"
            value={user.name}
            onChange={(e) =>
              setUser({
                ...user,
                name: e.target.value,
              })
            }
            style={input}
          />

          <input
            placeholder="Email"
            value={user.email}
            onChange={(e) =>
              setUser({
                ...user,
                email: e.target.value,
              })
            }
            style={input}
          />

          <button
            onClick={login}
            style={btn}
          >
            Login
          </button>
        </div>
      </div>
    );
  }

  return (
    <div
      style={{
        background: styles.bg,
        color: styles.text,
        minHeight: "100vh",
      }}
    >
      <header
        style={{
          padding: 20,
          display: "flex",
          justifyContent: "space-between",
        }}
      >
        <h2>📝 Notes Maker</h2>

        <div>
          <button
            style={btn}
            onClick={() =>
              setDark(!dark)
            }
          >
            {dark ? "☀️" : "🌙"}
          </button>

          <button
            style={btn}
            onClick={exportNotes}
          >
            Export
          </button>
        </div>
      </header>

      <div
        style={{
          display: "grid",
          gridTemplateColumns:
            "350px 1fr",
          gap: 20,
          padding: 20,
        }}
      >
        <div
          style={{
            background: styles.card,
            padding: 20,
            borderRadius: 20,
          }}
        >
          <h3>Create Note</h3>

          <input
            style={input}
            placeholder="Title"
            value={form.title}
            onChange={(e) =>
              setForm({
                ...form,
                title: e.target.value,
              })
            }
          />

          <textarea
            style={{
              ...input,
              height: 150,
            }}
            placeholder="Write..."
            value={form.content}
            onChange={(e) =>
              setForm({
                ...form,
                content:
                  e.target.value,
              })
            }
          />

          <input
            style={input}
            placeholder="Category"
            value={form.category}
            onChange={(e) =>
              setForm({
                ...form,
                category:
                  e.target.value,
              })
            }
          />

          <input
            style={input}
            placeholder="Tags"
            value={form.tags}
            onChange={(e) =>
              setForm({
                ...form,
                tags:
                  e.target.value,
              })
            }
          />

          <button
            style={btn}
            onClick={addNote}
          >
            Add Note
          </button>

          <hr />

          <h4>User</h4>

          <p>{user.name}</p>
          <p>{user.email}</p>

          <p>Total Notes: {notes.length}</p>
        </div>

        <div>
          <div
            style={{
              display: "flex",
              gap: 10,
            }}
          >
            <input
              style={{
                ...input,
                flex: 1,
              }}
              placeholder="Search"
              value={search}
              onChange={(e) =>
                setSearch(
                  e.target.value
                )
              }
            />

            <select
              style={input}
              onChange={(e) =>
                setCategory(
                  e.target.value
                )
              }
            >
              {categories.map(
                (c) => (
                  <option key={c}>
                    {c}
                  </option>
                )
              )}
            </select>
          </div>

          <div
            style={{
              display: "grid",
              gap: 20,
              marginTop: 20,
            }}
          >
            {filtered.map((n) => (
              <div
                key={n.id}
                style={{
                  background:
                    styles.card,
                  padding: 20,
                  borderRadius: 20,
                }}
              >
                <h3>
                  {n.pinned
                    ? "📌"
                    : ""}
                  {n.title}
                </h3>

                <p>{n.content}</p>

                <small>
                  {n.date}
                </small>

                <p>
                  #{n.category}
                </p>

                <p>
                  {n.tags}
                </p>

                <button
                  style={btn}
                  onClick={() =>
                    pin(n.id)
                  }
                >
                  Pin
                </button>

                <button
                  style={btn}
                  onClick={() =>
                    remove(
                      n.id
                    )
                  }
                >
                  Delete
                </button>
              </div>
            ))}
          </div>
        </div>
      </div>
    </div>
  );
}

const input = {
  width: "100%",
  padding: 12,
  marginTop: 10,
  borderRadius: 10,
  border: "1px solid #ddd",
};

const btn = {
  padding: 12,
  marginTop: 10,
  border: 0,
  borderRadius: 10,
  cursor: "pointer",
};
