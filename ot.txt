import { useState, useEffect, useCallback } from "react";

const ADMIN_PASSWORD = "admin123";

const DEFAULT_CATEGORIES = ["መድሃኒቶች", "ክትባቶች", "ምግቦች", "አቅርቦቶች", "ሌሎች"];

const DEFAULT_PRODUCTS = [
  { id: 1, name: "Amoxicillin 250mg", category: "መድሃኒቶች", price: 45, unit: "ጥቅል" },
  { id: 2, name: "Ivermectin", category: "መድሃኒቶች", price: 120, unit: "ጠርሙስ" },
  { id: 3, name: "Rabies Vaccine", category: "ክትባቶች", price: 350, unit: "ዶዝ" },
  { id: 4, name: "FMD Vaccine", category: "ክትባቶች", price: 280, unit: "ዶዝ" },
  { id: 5, name: "Dog Food Premium", category: "ምግቦች", price: 890, unit: "ኪሎ" },
  { id: 6, name: "Syringe 5ml", category: "አቅርቦቶች", price: 15, unit: "ቁጥር" },
];

const categoryColors = {
  "መድሃኒቶች": "#2d8a4e",
  "ክትባቶች": "#1a6b8a",
  "ምግቦች": "#8a6b1a",
  "አቅርቦቶች": "#6b1a8a",
  "ሌሎች": "#8a1a1a",
};

const getCategoryColor = (cat) => categoryColors[cat] || "#555";

export default function App() {
  const [screen, setScreen] = useState("home"); // home | admin | employee
  const [products, setProducts] = useState(DEFAULT_PRODUCTS);
  const [categories, setCategories] = useState(DEFAULT_CATEGORIES);
  const [loaded, setLoaded] = useState(false);

  // Load from storage
  useEffect(() => {
    async function load() {
      try {
        const p = await window.storage.get("vp_products");
        if (p) setProducts(JSON.parse(p.value));
        const c = await window.storage.get("vp_categories");
        if (c) setCategories(JSON.parse(c.value));
      } catch {}
      setLoaded(true);
    }
    load();
  }, []);

  const saveProducts = useCallback(async (data) => {
    setProducts(data);
    try { await window.storage.set("vp_products", JSON.stringify(data)); } catch {}
  }, []);

  const saveCategories = useCallback(async (data) => {
    setCategories(data);
    try { await window.storage.set("vp_categories", JSON.stringify(data)); } catch {}
  }, []);

  if (!loaded) return <div style={styles.loading}>እየጫነ ነው...</div>;

  if (screen === "employee") return <EmployeeView products={products} categories={categories} onBack={() => setScreen("home")} />;
  if (screen === "admin") return <AdminView products={products} categories={categories} saveProducts={saveProducts} saveCategories={saveCategories} onBack={() => setScreen("home")} />;
  return <Home onAdmin={() => setScreen("admin")} onEmployee={() => setScreen("employee")} />;
}

function Home({ onAdmin, onEmployee }) {
  const [showLogin, setShowLogin] = useState(false);
  const [pwd, setPwd] = useState("");
  const [error, setError] = useState("");

  const handleLogin = () => {
    if (pwd === ADMIN_PASSWORD) { onAdmin(); }
    else { setError("የተሳሳተ የይለፍ ቃል!"); setPwd(""); }
  };

  return (
    <div style={styles.homeWrap}>
      <div style={styles.homeCard}>
        <div style={styles.logo}>🐾</div>
        <h1 style={styles.homeTitle}>የእንስሳት ፋርማሲ</h1>
        <p style={styles.homeSubtitle}>የዋጋ አስተዳደር ስርዓት</p>
        <div style={styles.homeBtns}>
          <button style={styles.btnEmployee} onClick={onEmployee}>
            👁 የዋጋ ዝርዝር ማየት
          </button>
          <button style={styles.btnAdmin} onClick={() => { setShowLogin(true); setError(""); setPwd(""); }}>
            🔐 አስተዳዳሪ ግባ
          </button>
        </div>
      </div>

      {showLogin && (
        <div style={styles.overlay} onClick={() => setShowLogin(false)}>
          <div style={styles.modal} onClick={e => e.stopPropagation()}>
            <h2 style={styles.modalTitle}>🔐 አስተዳዳሪ መግቢያ</h2>
            <input
              style={styles.input}
              type="password"
              placeholder="የይለፍ ቃል"
              value={pwd}
              onChange={e => { setPwd(e.target.value); setError(""); }}
              onKeyDown={e => e.key === "Enter" && handleLogin()}
              autoFocus
            />
            {error && <p style={styles.errorTxt}>{error}</p>}
            <div style={{ display: "flex", gap: 10 }}>
              <button style={styles.btnPrimary} onClick={handleLogin}>ግባ</button>
              <button style={styles.btnSecondary} onClick={() => setShowLogin(false)}>ሰርዝ</button>
            </div>
            <p style={{ fontSize: 11, color: "#aaa", marginTop: 10 }}>ነባሪ ይለፍ ቃል: admin123</p>
          </div>
        </div>
      )}
    </div>
  );
}

function EmployeeView({ products, categories, onBack }) {
  const [activeCategory, setActiveCategory] = useState("ሁሉም");
  const [search, setSearch] = useState("");

  const filtered = products.filter(p => {
    const matchCat = activeCategory === "ሁሉም" || p.category === activeCategory;
    const matchSearch = p.name.toLowerCase().includes(search.toLowerCase());
    return matchCat && matchSearch;
  });

  return (
    <div style={styles.pageWrap}>
      <div style={styles.header}>
        <button style={styles.backBtn} onClick={onBack}>← ተመለስ</button>
        <h1 style={styles.pageTitle}>🐾 የዋጋ ዝርዝር</h1>
        <div style={{ width: 70 }} />
      </div>

      <div style={styles.searchBar}>
        <input style={styles.searchInput} placeholder="🔍 ምርት ፈልግ..." value={search} onChange={e => setSearch(e.target.value)} />
      </div>

      <div style={styles.catTabs}>
        {["ሁሉም", ...categories].map(c => (
          <button
            key={c}
            style={{ ...styles.catTab, ...(activeCategory === c ? styles.catTabActive : {}), ...(c !== "ሁሉም" ? { borderColor: getCategoryColor(c) } : {}) }}
            onClick={() => setActiveCategory(c)}
          >
            {c}
          </button>
        ))}
      </div>

      <div style={styles.productGrid}>
        {filtered.length === 0 && <p style={styles.emptyTxt}>ምርት አልተገኘም</p>}
        {filtered.map(p => (
          <div key={p.id} style={styles.productCard}>
            <div style={{ ...styles.catBadge, background: getCategoryColor(p.category) }}>{p.category}</div>
            <p style={styles.productName}>{p.name}</p>
            <div style={styles.priceRow}>
              <span style={styles.price}>{p.price.toLocaleString()} ብር</span>
              <span style={styles.unit}>/ {p.unit}</span>
            </div>
          </div>
        ))}
      </div>
      <p style={styles.countTxt}>{filtered.length} ምርቶች</p>
    </div>
  );
}

function AdminView({ products, categories, saveProducts, saveCategories, onBack }) {
  const [tab, setTab] = useState("products");
  const [editItem, setEditItem] = useState(null);
  const [showForm, setShowForm] = useState(false);
  const [form, setForm] = useState({ name: "", category: "", price: "", unit: "ቁጥር" });
  const [newCat, setNewCat] = useState("");
  const [search, setSearch] = useState("");
  const [filterCat, setFilterCat] = useState("ሁሉም");

  const openAdd = () => {
    setEditItem(null);
    setForm({ name: "", category: categories[0] || "", price: "", unit: "ቁጥር" });
    setShowForm(true);
  };

  const openEdit = (p) => {
    setEditItem(p);
    setForm({ name: p.name, category: p.category, price: String(p.price), unit: p.unit });
    setShowForm(true);
  };

  const handleSave = () => {
    if (!form.name || !form.price || isNaN(Number(form.price))) return;
    if (editItem) {
      saveProducts(products.map(p => p.id === editItem.id ? { ...p, ...form, price: Number(form.price) } : p));
    } else {
      const id = Date.now();
      saveProducts([...products, { id, ...form, price: Number(form.price) }]);
    }
    setShowForm(false);
  };

  const handleDelete = (id) => {
    if (confirm("ይሄን ምርት ሊሰርዙ ይፈልጋሉ?")) saveProducts(products.filter(p => p.id !== id));
  };

  const addCategory = () => {
    if (!newCat.trim() || categories.includes(newCat.trim())) return;
    saveCategories([...categories, newCat.trim()]);
    setNewCat("");
  };

  const deleteCategory = (cat) => {
    if (products.some(p => p.category === cat)) { alert("ይህ ምድብ ምርቶች ስላሉት ሊሰረዝ አይችልም!"); return; }
    if (confirm(`"${cat}" ምድብ ሊሰርዙ ይፈልጋሉ?`)) saveCategories(categories.filter(c => c !== cat));
  };

  const filtered = products.filter(p => {
    const matchCat = filterCat === "ሁሉም" || p.category === filterCat;
    const matchSearch = p.name.toLowerCase().includes(search.toLowerCase());
    return matchCat && matchSearch;
  });

  return (
    <div style={styles.pageWrap}>
      <div style={styles.header}>
        <button style={styles.backBtn} onClick={onBack}>← ተመለስ</button>
        <h1 style={styles.pageTitle}>⚙️ አስተዳዳሪ</h1>
        <div style={{ width: 70 }} />
      </div>

      <div style={styles.adminTabs}>
        <button style={{ ...styles.adminTab, ...(tab === "products" ? styles.adminTabActive : {}) }} onClick={() => setTab("products")}>ምርቶች</button>
        <button style={{ ...styles.adminTab, ...(tab === "categories" ? styles.adminTabActive : {}) }} onClick={() => setTab("categories")}>ምድቦች</button>
      </div>

      {tab === "products" && (
        <>
          <div style={styles.searchBar}>
            <input style={styles.searchInput} placeholder="🔍 ምርት ፈልግ..." value={search} onChange={e => setSearch(e.target.value)} />
            <button style={styles.addBtn} onClick={openAdd}>+ አስገባ</button>
          </div>
          <div style={styles.catTabs}>
            {["ሁሉም", ...categories].map(c => (
              <button key={c} style={{ ...styles.catTab, ...(filterCat === c ? styles.catTabActive : {}) }} onClick={() => setFilterCat(c)}>{c}</button>
            ))}
          </div>
          <div style={styles.productGrid}>
            {filtered.length === 0 && <p style={styles.emptyTxt}>ምርት አልተገኘም</p>}
            {filtered.map(p => (
              <div key={p.id} style={styles.productCard}>
                <div style={{ ...styles.catBadge, background: getCategoryColor(p.category) }}>{p.category}</div>
                <p style={styles.productName}>{p.name}</p>
                <div style={styles.priceRow}>
                  <span style={styles.price}>{p.price.toLocaleString()} ብር</span>
                  <span style={styles.unit}>/ {p.unit}</span>
                </div>
                <div style={styles.cardActions}>
                  <button style={styles.editBtn} onClick={() => openEdit(p)}>✏️ ቀይር</button>
                  <button style={styles.deleteBtn} onClick={() => handleDelete(p.id)}>🗑 ሰርዝ</button>
                </div>
              </div>
            ))}
          </div>
          <p style={styles.countTxt}>{filtered.length} ምርቶች</p>
        </>
      )}

      {tab === "categories" && (
        <div style={styles.catManage}>
          <div style={styles.addCatRow}>
            <input style={styles.input} placeholder="አዲስ ምድብ ስም..." value={newCat} onChange={e => setNewCat(e.target.value)} onKeyDown={e => e.key === "Enter" && addCategory()} />
            <button style={styles.btnPrimary} onClick={addCategory}>+ አስገባ</button>
          </div>
          {categories.map(c => (
            <div key={c} style={styles.catRow}>
              <div style={{ ...styles.catDot, background: getCategoryColor(c) }} />
              <span style={styles.catName}>{c}</span>
              <span style={styles.catCount}>{products.filter(p => p.category === c).length} ምርቶች</span>
              <button style={styles.deleteBtn} onClick={() => deleteCategory(c)}>🗑</button>
            </div>
          ))}
        </div>
      )}

      {showForm && (
        <div style={styles.overlay} onClick={() => setShowForm(false)}>
          <div style={styles.modal} onClick={e => e.stopPropagation()}>
            <h2 style={styles.modalTitle}>{editItem ? "✏️ ምርት ቀይር" : "➕ አዲስ ምርት"}</h2>
            <label style={styles.label}>የምርት ስም</label>
            <input style={styles.input} placeholder="ለምሳሌ: Amoxicillin 250mg" value={form.name} onChange={e => setForm({ ...form, name: e.target.value })} />
            <label style={styles.label}>ምድብ</label>
            <select style={styles.input} value={form.category} onChange={e => setForm({ ...form, category: e.target.value })}>
              {categories.map(c => <option key={c} value={c}>{c}</option>)}
            </select>
            <label style={styles.label}>ዋጋ (ብር)</label>
            <input style={styles.input} type="number" placeholder="ለምሳሌ: 250" value={form.price} onChange={e => setForm({ ...form, price: e.target.value })} />
            <label style={styles.label}>መለኪያ</label>
            <select style={styles.input} value={form.unit} onChange={e => setForm({ ...form, unit: e.target.value })}>
              {["ቁጥር", "ጥቅል", "ጠርሙስ", "ዶዝ", "ኪሎ", "ሊትር", "ሳጥን"].map(u => <option key={u} value={u}>{u}</option>)}
            </select>
            <div style={{ display: "flex", gap: 10, marginTop: 5 }}>
              <button style={styles.btnPrimary} onClick={handleSave}>💾 አስቀምጥ</button>
              <button style={styles.btnSecondary} onClick={() => setShowForm(false)}>ሰርዝ</button>
            </div>
          </div>
        </div>
      )}
    </div>
  );
}

const styles = {
  loading: { display: "flex", alignItems: "center", justifyContent: "center", height: "100vh", fontFamily: "sans-serif", color: "#555", background: "#f0f4f0" },
  homeWrap: { minHeight: "100vh", background: "linear-gradient(135deg, #1a3a2a 0%, #2d5a3d 50%, #1a4a3a 100%)", display: "flex", alignItems: "center", justifyContent: "center", padding: 20 },
  homeCard: { background: "rgba(255,255,255,0.96)", borderRadius: 24, padding: "48px 40px", textAlign: "center", maxWidth: 380, width: "100%", boxShadow: "0 20px 60px rgba(0,0,0,0.3)" },
  logo: { fontSize: 64, marginBottom: 8 },
  homeTitle: { fontFamily: "'Segoe UI', sans-serif", fontSize: 26, fontWeight: 800, color: "#1a3a2a", margin: "0 0 6px" },
  homeSubtitle: { color: "#5a7a6a", fontSize: 14, margin: "0 0 36px" },
  homeBtns: { display: "flex", flexDirection: "column", gap: 14 },
  btnEmployee: { padding: "16px 24px", borderRadius: 14, border: "2px solid #2d8a4e", background: "#fff", color: "#2d8a4e", fontSize: 16, fontWeight: 700, cursor: "pointer" },
  btnAdmin: { padding: "16px 24px", borderRadius: 14, border: "none", background: "linear-gradient(135deg, #1a3a2a, #2d5a3d)", color: "#fff", fontSize: 16, fontWeight: 700, cursor: "pointer" },
  overlay: { position: "fixed", inset: 0, background: "rgba(0,0,0,0.6)", display: "flex", alignItems: "center", justifyContent: "center", zIndex: 100, padding: 20 },
  modal: { background: "#fff", borderRadius: 20, padding: 28, width: "100%", maxWidth: 380, boxShadow: "0 12px 40px rgba(0,0,0,0.25)" },
  modalTitle: { fontFamily: "'Segoe UI', sans-serif", fontSize: 18, fontWeight: 800, color: "#1a3a2a", margin: "0 0 18px" },
  input: { width: "100%", padding: "12px 14px", borderRadius: 10, border: "1.5px solid #cde0d0", fontSize: 15, marginBottom: 14, boxSizing: "border-box", outline: "none", fontFamily: "inherit" },
  label: { display: "block", fontSize: 13, fontWeight: 600, color: "#3a5a4a", marginBottom: 5 },
  btnPrimary: { flex: 1, padding: "13px 0", borderRadius: 10, border: "none", background: "linear-gradient(135deg, #2d5a3d, #2d8a4e)", color: "#fff", fontSize: 15, fontWeight: 700, cursor: "pointer" },
  btnSecondary: { flex: 1, padding: "13px 0", borderRadius: 10, border: "1.5px solid #ccc", background: "#fff", color: "#555", fontSize: 15, fontWeight: 600, cursor: "pointer" },
  errorTxt: { color: "#c0392b", fontSize: 13, margin: "-8px 0 12px" },
  pageWrap: { minHeight: "100vh", background: "#f4f7f5", fontFamily: "'Segoe UI', sans-serif" },
  header: { background: "linear-gradient(135deg, #1a3a2a, #2d5a3d)", padding: "18px 20px", display: "flex", alignItems: "center", justifyContent: "space-between", position: "sticky", top: 0, zIndex: 10 },
  pageTitle: { color: "#fff", fontSize: 18, fontWeight: 800, margin: 0 },
  backBtn: { background: "rgba(255,255,255,0.18)", border: "none", color: "#fff", padding: "8px 14px", borderRadius: 10, cursor: "pointer", fontSize: 13, fontWeight: 600 },
  searchBar: { padding: "14px 16px", display: "flex", gap: 10 },
  searchInput: { flex: 1, padding: "12px 14px", borderRadius: 12, border: "1.5px solid #cde0d0", fontSize: 15, outline: "none", background: "#fff" },
  addBtn: { padding: "12px 18px", borderRadius: 12, border: "none", background: "linear-gradient(135deg, #2d5a3d, #2d8a4e)", color: "#fff", fontSize: 15, fontWeight: 700, cursor: "pointer", whiteSpace: "nowrap" },
  catTabs: { display: "flex", gap: 8, padding: "0 16px 12px", overflowX: "auto", scrollbarWidth: "none" },
  catTab: { padding: "7px 16px", borderRadius: 20, border: "1.5px solid #ccc", background: "#fff", color: "#555", fontSize: 13, fontWeight: 600, cursor: "pointer", whiteSpace: "nowrap", flexShrink: 0 },
  catTabActive: { background: "#1a3a2a", color: "#fff", border: "1.5px solid #1a3a2a" },
  productGrid: { display: "grid", gridTemplateColumns: "repeat(auto-fill, minmax(160px, 1fr))", gap: 12, padding: "0 16px 16px" },
  productCard: { background: "#fff", borderRadius: 16, padding: 14, boxShadow: "0 2px 12px rgba(0,0,0,0.07)", display: "flex", flexDirection: "column", gap: 6 },
  catBadge: { alignSelf: "flex-start", padding: "3px 10px", borderRadius: 12, color: "#fff", fontSize: 11, fontWeight: 700 },
  productName: { fontSize: 14, fontWeight: 700, color: "#1a3a2a", margin: 0, lineHeight: 1.4 },
  priceRow: { display: "flex", alignItems: "baseline", gap: 4 },
  price: { fontSize: 18, fontWeight: 800, color: "#2d8a4e" },
  unit: { fontSize: 12, color: "#888" },
  cardActions: { display: "flex", gap: 6, marginTop: 4 },
  editBtn: { flex: 1, padding: "7px 0", borderRadius: 8, border: "1.5px solid #2d8a4e", background: "#fff", color: "#2d8a4e", fontSize: 12, fontWeight: 700, cursor: "pointer" },
  deleteBtn: { flex: 1, padding: "7px 0", borderRadius: 8, border: "1.5px solid #e74c3c", background: "#fff", color: "#e74c3c", fontSize: 12, fontWeight: 700, cursor: "pointer" },
  emptyTxt: { color: "#aaa", textAlign: "center", gridColumn: "1/-1", paddingTop: 40, fontSize: 15 },
  countTxt: { textAlign: "center", color: "#999", fontSize: 12, paddingBottom: 20 },
  adminTabs: { display: "flex", margin: "14px 16px 0", gap: 8 },
  adminTab: { flex: 1, padding: "11px 0", borderRadius: 12, border: "1.5px solid #ccc", background: "#fff", color: "#555", fontSize: 15, fontWeight: 600, cursor: "pointer" },
  adminTabActive: { background: "#1a3a2a", color: "#fff", border: "1.5px solid #1a3a2a" },
  catManage: { padding: "14px 16px" },
  addCatRow: { display: "flex", gap: 10, marginBottom: 16 },
  catRow: { display: "flex", alignItems: "center", gap: 10, background: "#fff", borderRadius: 12, padding: "12px 14px", marginBottom: 8, boxShadow: "0 1px 6px rgba(0,0,0,0.06)" },
  catDot: { width: 14, height: 14, borderRadius: "50%", flexShrink: 0 },
  catName: { flex: 1, fontWeight: 700, color: "#1a3a2a", fontSize: 15 },
  catCount: { fontSize: 12, color: "#888" },
};
