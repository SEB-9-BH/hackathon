# React Team Hackathon

## 1) Explain & Synopsis

You will build a **4‑page React SPA** that calls a real API. Pick **OMDb** or **WeatherAPI**. Work in teams teams of 4-6. Use a clean Git workflow with a **Project Manager… GitHub Manager**.

Think assembly line. Plan first… then code. Small parts… frequent merges… working app.

**What you’ll practice**

* AJAX with `fetch`.
* Lifting shared state to a parent.
* `useEffect` for lifecycle.
* React Router v6 with `BrowserRouter`, `Routes`, `Route`, `Link`, `useParams`.

---

## 2) Setup

**Roles**

* **PM…GitHub Manager**. Owns repo… releases… merges api keys. (Own whole repo)
* **UI Lead**. Wireframes… css styles… accessibility. (Owns all css files)
* **Layout Lead**. Nav, Footer, 404 Page. (owns App.jsx & Components Folder)
* **Page Dev A**. Builds two pages.
* **Page Dev B**. Builds two pages.
* **QA/API/Docs**. testing… README... bug fixes. (owns Readme & Responsible for deciphering API) Does the Submission Issue in Correct Template

**Repository**

1. PM creates a fresh **Vite React** repo.
2. Strip boilerplate to **Hello World**.


**Local prereqs**

* Node 18+.
* Git.
* GitHub accounts.

**Project creation**

```bash
npm create vite@latest team-app -- --template react
cd team-app
npm i
npm i react-router-dom
npm run dev
```

**Environment variables**

* Create `.env` at project root.

  * OMDb… `VITE_OMDB_KEY=YOUR_KEY`
  * WeatherAPI… `VITE_WEATHER_KEY=YOUR_KEY`
* Vite only exposes keys that start with `VITE_`.

**Folders example**

```
src/
  main.jsx
  App.jsx
  index.css
  components/
    Nav/Nav.jsx
  pages/
    Home/Home.jsx
    Search/Search.jsx
    Details/Details.jsx
    About/About.jsx
    NotFound/NotFound.jsx
```

**Router skeleton**

```jsx
// src/main.jsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import { BrowserRouter } from "react-router-dom";
import App from "./App.jsx";
import "./index.css";

createRoot(document.getElementById("root")).render(
  <StrictMode>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </StrictMode>
);
```

```jsx
// src/App.jsx
import { Routes, Route } from "react-router-dom";
import Nav from "./components/Nav/Nav.jsx";
import Home from "./pages/Home/Home.jsx";
import Search from "./pages/Search/Search.jsx";
import Details from "./pages/Details/Details.jsx";
import About from "./pages/About/About.jsx";
import NotFound from "./pages/NotFound/NotFound.jsx";

export default function App() {
  return (
    <div className="App">
      <Nav />
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/search" element={<Search />} />
        <Route path="/details/:id" element={<Details />} />
        <Route path="/about" element={<About />} />
        <Route path="*" element={<NotFound />} />
      </Routes>
    </div>
  );
}
```

**Page stubs**

```jsx
// pages/Home/Home.jsx
export default function Home(){ return <h1>Home</h1>; }

// pages/Search/Search.jsx
export default function Search(){ return <h1>Search</h1>; }

// pages/Details/Details.jsx
import { useParams } from "react-router-dom";
export default function Details(){
  const { id } = useParams();
  return <h1>Details for {id}</h1>;
}

// pages/About/About.jsx
export default function About(){ return <h1>About</h1>; }

// pages/NotFound/NotFound.jsx
export default function NotFound(){ return <h1>Not Found</h1>; }
```

**Nav**

```jsx
// components/Nav/Nav.jsx
import { Link } from "react-router-dom";
export default function Nav(){
  return (
    <nav className="nav">
      <Link to="/"><div>HOME</div></Link>
      <Link to="/search"><div>SEARCH</div></Link>
      <Link to="/about"><div>ABOUT</div></Link>
    </nav>
  );
}
```

**Team workflow**

* PM pushes skeleton to GitHub.
* Others **fork** the repo… then `git remote add upstream <PM repo>` to sync.
* Work on **feature branches**. Open PRs to PM’s repo when ready.

**Plan before code**

* Read API docs… **omdbapi.com** or **weatherapi.com**.
* Make a quick **wireframe** and user flow.
* Build a **Trello** with lists… Backlog… In Progress… Review… Done.

---

## 3) Walkthrough… suggestions only… do not copy this

Use these steps as a guide. Adapt to your idea.

### A) API choice and helpers

**Option 1… OMDb**

```jsx
// src/lib/omdb.js
const KEY = import.meta.env.VITE_OMDB_KEY;

export async function searchMovies(term, page=1){
  if(!term) return { items:[], total:0 };
  const url = `https://www.omdbapi.com/?apikey=${KEY}&s=${encodeURIComponent(term)}&page=${page}`;
  const res = await fetch(url);
  const data = await res.json();
  if(data.Response === "True"){
    return { items: data.Search, total: Number(data.totalResults||0) };
  }
  return { items:[], total:0, error: data.Error || "No results" };
}

export async function getMovieById(id){
  const url = `https://www.omdbapi.com/?apikey=${KEY}&i=${id}&plot=full`;
  const res = await fetch(url);
  return await res.json();
}
```

**Option 2… WeatherAPI**

```jsx
// src/lib/weather.js
const KEY = import.meta.env.VITE_WEATHER_KEY;
const BASE = "https://api.weatherapi.com/v1";

export async function searchCity(q){
  const url = `${BASE}/search.json?key=${KEY}&q=${encodeURIComponent(q)}`;
  const r = await fetch(url);
  return await r.json(); // array of matches
}

export async function getForecast(q, days=3){
  const url = `${BASE}/forecast.json?key=${KEY}&q=${encodeURIComponent(q)}&days=${days}`;
  const r = await fetch(url);
  return await r.json();
}
```

Pick one API. Do not mix both in one app today.

### B) Search page… state up… UI pure

**OMDb example**

```jsx
// pages/Search/Search.jsx
import { useEffect, useState } from "react";
import { Link } from "react-router-dom";
import { searchMovies } from "../../lib/omdb.js";

export default function Search(){
  const [term, setTerm] = useState("");
  const [page, setPage] = useState(1);
  const [items, setItems] = useState([]);
  const [total, setTotal] = useState(0);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState("");

  useEffect(() => {
    let ignore = false;
    async function run(){
      if(!term){ setItems([]); setTotal(0); return; }
      setLoading(true);
      setError("");
      const { items:arr, total:t, error:e } = await searchMovies(term, page);
      if(!ignore){
        setItems(arr);
        setTotal(t);
        if(e) setError(e);
        setLoading(false);
      }
    }
    run();
    return () => { ignore = true; };
  }, [term, page]);

  const pages = Math.max(1, Math.ceil(total/10));

  return (
    <section className="container">
      <h1>Search</h1>

      <form onSubmit={(e)=>{ e.preventDefault(); setPage(1); setTerm(e.currentTarget.q.value.trim()); }} className="search">
        <input name="q" placeholder="Search movies…" />
        <button>Go</button>
      </form>

      {loading && <p className="muted">Loading…</p>}
      {error && <p className="error">{error}</p>}

      <div className="grid">
        {items.map(m => (
          <article key={m.imdbID} className="card-sm">
            <img src={m.Poster !== "N/A" ? m.Poster : "/placeholder.png"} alt={m.Title} />
            <div style={{ padding: ".5rem .75rem" }}>
              <strong>{m.Title}</strong>
              <p>{m.Year}</p>
              <Link to={`/details/${m.imdbID}`}><button>Details</button></Link>
            </div>
          </article>
        ))}
      </div>

      {pages > 1 && (
        <div className="pager">
          <button disabled={page<=1} onClick={()=>setPage(p=>p-1)}>Prev</button>
          <span>{page} / {pages}</span>
          <button disabled={page>=pages} onClick={()=>setPage(p=>p+1)}>Next</button>
        </div>
      )}
    </section>
  );
}
```

**Weather variant**

* `Search` lists city matches from `searchCity`.
* `Details` accepts a city query like `:id` of `"Atlanta"` or `"33.75,-84.39"`.

### C) Details page… URL params… fetch with cleanup

**OMDb example**

```jsx
// pages/Details/Details.jsx
import { useParams } from "react-router-dom";
import { useEffect, useState } from "react";
import { getMovieById } from "../../lib/omdb.js";

export default function Details(){
  const { id } = useParams();
  const [movie, setMovie] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState("");

  useEffect(()=>{
    if(!id) return;
    const controller = new AbortController();
    async function load(){
      try{
        setLoading(true);
        setError("");
        const data = await getMovieById(id);
        if(data.Response === "True") setMovie(data);
        else setError(data.Error || "Not found");
      }catch(e){
        if(e.name !== "AbortError") setError("Network error");
      }finally{
        setLoading(false);
      }
    }
    load();
    return ()=>controller.abort();
  }, [id]);

  if(loading) return <p className="muted">Loading…</p>;
  if(error) return <p className="error">{error}</p>;
  if(!movie) return <p className="muted">No data</p>;

  return (
    <article className="card">
      <img src={movie.Poster} alt={movie.Title} />
      <div className="card-body">
        <h2>{movie.Title}</h2>
        <p>{movie.Genre} • {movie.Year}</p>
        <p><strong>Rated:</strong> {movie.Rated} • <strong>Runtime:</strong> {movie.Runtime}</p>
        <p className="plot">{movie.Plot}</p>
      </div>
    </article>
  );
}
```

### D) Minimum styling

```css
/* index.css */
.container { max-width: 960px; margin: 0 auto; padding: 1rem; }
.nav { display: flex; gap: 1rem; align-items: center; padding: 12px 16px; background:#000; color:#fff; font-weight:700; }
.nav a { color:#fff; text-decoration:none; }
.grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(150px,1fr)); gap: 1rem; }
.card-sm { background: #0b1324; border: 1px solid #1f2937; border-radius: 10px; overflow: hidden; }
.card-sm img { width: 100%; aspect-ratio: 2/3; object-fit: cover; }
.pager { display: flex; gap: .5rem; justify-content: center; margin: 1rem 0; }
.muted { color:#9ca3af; }
.error { color:#ef4444; }
```

### E) Git… suggested flow

**PM… once**

```bash
git init
git add -A
git commit -m "chore: init vite app…router skeleton"
git branch -M main
git remote add origin <PM_REPO_URL>
git push -u origin main
```

**All others**

```bash
# On GitHub… fork PM repo
git clone <YOUR_FORK_URL>
cd team-app
git remote add upstream <PM_REPO_URL>
git fetch upstream
git checkout main
git pull --rebase upstream main
git push origin main
```

**Feature work**

```bash
git checkout -b feat/search-page
# code… commit small
git add -A
git commit -m "feat: search page grid…pagination"
git push -u origin feat/search-page
# Open PR to PM's repo… reviewer is PM
```

**Sync often**

```bash
git fetch upstream
git checkout main
git pull --rebase upstream main
git push origin main
```

**Rules**

* No direct pushes to `main`.
* One owner per file at a time.
* Small PRs… clear descriptions.
* Resolve conflicts locally… never break `main`.

### F) Team planning prompts

* Wireframe the four pages… note loading… error… empty states.
* Trello cards… one owner… acceptance per card.
* Decide stretch early… favorites for OMDb… or 3‑day forecast for Weather.

---

## 4) Acceptance Criteria

**Architecture**

* Uses **React Router v6** simple API… `BrowserRouter`… `Routes`… `Route`… `Link`… `useParams`.
* At least **4 routed pages**… Home… Search… Details… About… plus NotFound.

**Data**

* Real API calls to **OMDb** or **WeatherAPI** with keys in `.env`.
* `useEffect` drives fetches.
* Shows **loading**, **error**, and **empty** states.

**UI**

* Responsive grid for results.
* Details page shows meaningful fields.
* Basic accessibility… labeled inputs… keyboard friendly nav.

**Process**

* Wireframe images included in ReadME.
* Trello board link with visible tasks and owners.
* Clean Git history… feature branches… PRs… squash or merge. No direct pushes to `main`.

**Docs**

* README with setup, run steps, team roles, API choice, screenshots, known issues, next steps.

**Stretch earns kudos**

* Watchlist stored in `localStorage`.
* Pagination or filters.
* Simple chart for Weather forecast.(hardddd to do)

---

## 5) Submission Instructions

**You will submit with a GitHub Issue… not a Pull Request.**

1. Go to this repository.
2. Click **Issues**… **New issue**.
3. Title format… `React Hackathon Submission … Team <Team Name>`.
4. Paste the template below into the body and fill it out.
5. Add the label **submission** if available.
6. Submit the issue. That issue is your official hand‑in.

**Issue body template**

```
### Team
PM/GitHub Manager: <name>
Layout Lead: <name>
UI Lead: <name>
Page Dev A: <name>
Page Dev B: <name>
QA/API/Docs: <name>

### Links
Repo: <URL>
Live Surge Demo (optional): <URL>
Trello: <URL>
Wireframe: <URL or /docs/wireframe.png>

### API Choice
OMDb or WeatherAPI … why we chose it:

### How to Run
1) .env with VITE_* key
2) npm i
3) npm run dev

### What Works
- <bullet>
- <bullet>

### Known Issues
- <bullet>

### Screenshots
- Home
- Search
- Details

### Contributors Proof
We used branches and PRs. Instructors can verify via GitHub History and Blame.
```

---

### Key takeaways

Keep Router simple to match prior lessons.
Plan first… wireframe and Trello… then code.
Lift shared state to the parent… keep renders pure.
Handle loading… error… empty… always.
Submit with a **GitHub Issue**… not a PR.
