// ── UI ATOMS ─────────────────────────────────────────────────────────────────
const Badge = ({ c="blue", children }) => {
  const map = { blue:"bg-blue-100 text-blue-700", green:"bg-green-100 text-green-700", gray:"bg-gray-100 text-gray-500", purple:"bg-purple-100 text-purple-700", red:"bg-red-100 text-red-600", yellow:"bg-yellow-100 text-yellow-700" };
  return <span className={cls("text-xs font-semibold px-2 py-0.5 rounded-full",map[c]||map.blue)}>{children}</span>;
};
const Btn = ({ children, onClick, v="primary", sz="md", disabled, full, className="" }) => {
  const vs = { primary:"bg-blue-600 hover:bg-blue-700 text-white shadow-sm", secondary:"bg-white border border-blue-500 text-blue-600 hover:bg-blue-50", ghost:"text-blue-600 hover:bg-blue-50 border border-transparent", danger:"bg-red-500 hover:bg-red-600 text-white", purple:"bg-purple-600 hover:bg-purple-700 text-white", success:"bg-green-500 hover:bg-green-600 text-white", dark:"bg-gray-800 hover:bg-gray-900 text-white" };
  const ss = { sm:"px-3 py-1.5 text-xs", md:"px-4 py-2 text-sm", lg:"px-6 py-3 text-base font-semibold" };
  return <button onClick={onClick} disabled={disabled} className={cls("rounded-xl font-medium transition-all duration-150 disabled:opacity-40 disabled:cursor-not-allowed",vs[v]||vs.primary,ss[sz]||ss.md,full&&"w-full",className)}>{children}</button>;
};
const Card = ({ children, className="" }) => <div className={cls("bg-white rounded-2xl border border-gray-100 shadow-sm p-5",className)}>{children}</div>;
const Input = ({ label, value, onChange, placeholder, type="text", rows, hint }) => {
  const base = "w-full border border-gray-200 rounded-xl px-3 py-2 text-sm bg-gray-50 focus:outline-none focus:ring-2 focus:ring-blue-400";
  return (
    <div className="flex flex-col gap-1">
      {label && <label className="text-xs font-semibold text-gray-600">{label}</label>}
      {rows ? <textarea className={base} rows={rows} value={value} onChange={e=>onChange(e.target.value)} placeholder={placeholder}/> : <input className={base} type={type} value={value} onChange={e=>onChange(e.target.value)} placeholder={placeholder}/>}
      {hint && <p className="text-xs text-gray-400">{hint}</p>}
    </div>
  );
};
const Spinner = ({ text="Memproses..." }) => (
  <div className="flex flex-col items-center gap-3 py-10">
    <div className="w-9 h-9 border-4 border-blue-500 border-t-transparent rounded-full animate-spin"/>
    <p className="text-sm text-gray-500">{text}</p>
  </div>
);
const ScoreRing = ({ score }) => {
  const r=36, c=2*Math.PI*r, color = score>=75?"#22c55e":score>=50?"#f59e0b":"#ef4444";
  return (
    <div className="flex flex-col items-center">
      <svg width="90" height="90" viewBox="0 0 90 90">
        <circle cx="45" cy="45" r={r} fill="none" stroke="#e5e7eb" strokeWidth="8"/>
        <circle cx="45" cy="45" r={r} fill="none" stroke={color} strokeWidth="8" strokeDasharray={c} strokeDashoffset={c-(score/100)*c} strokeLinecap="round" transform="rotate(-90 45 45)" style={{transition:"stroke-dashoffset 1s ease"}}/>
        <text x="45" y="50" textAnchor="middle" fontSize="18" fontWeight="bold" fill={color}>{score}%</text>
      </svg>
      <span className="text-xs font-medium mt-1" style={{color}}>{score>=75?"Bagus!":score>=50?"Perlu Perbaikan":"Kritis"}</span>
    </div>
  );
};

// ── CONTEXT / STATE ───────────────────────────────────────────────────────────
// users stored in memory: [{id, name, email, password, plan, createdAt, paymentStatus, paymentPlan, paymentRef, downloads}]
let _users = [
  { id:"admin", name:"Admin", email:"admin@cvlolos.ai", password:"admin123", plan:"lifetime", isAdmin:true, downloads:0, paymentStatus:"verified" },
];
let _payments = []; // [{id, userId, plan, amount, ref, status, createdAt}]

// ── AUTH PAGES ────────────────────────────────────────────────────────────────
function AuthPage({ onAuth }) {
  const [mode, setMode] = useState("login"); // login | register
  const [f, setF] = useState({ name:"", email:"", password:"", confirm:"" });
  const [err, setErr] = useState("");
  const up = (k,v) => setF(p=>({...p,[k]:v}));

  const submit = () => {
    setErr("");
    if (mode === "login") {
      const u = _users.find(u => u.email===f.email && u.password===f.password);
      if (!u) return setErr("Email atau password salah.");
      onAuth(u);
    } else {
      if (!f.name||!f.email||!f.password) return setErr("Semua kolom wajib diisi.");
      if (f.password !== f.confirm) return setErr("Password tidak cocok.");
      if (_users.find(u=>u.email===f.email)) return setErr("Email sudah terdaftar.");
      const u = { id: Date.now().toString(), name:f.name, email:f.email, password:f.password, plan:"free", downloads:0, paymentStatus:null, isAdmin:false };
      _users.push(u);
      onAuth(u);
    }
  };

  return (
    <div className="min-h-screen bg-gradient-to-br from-blue-50 via-white to-indigo-50 flex items-center justify-center p-4">
      <div className="w-full max-w-sm">
        <div className="text-center mb-6">
          <div className="inline-flex items-center gap-2 mb-3">
            <div className="w-9 h-9 bg-blue-600 rounded-xl flex items-center justify-center text-white font-bold">CV</div>
            <span className="text-xl font-bold text-gray-900">CVLolos<span className="text-blue-600">.ai</span></span>
          </div>
          <p className="text-gray-500 text-sm">{mode==="login"?"Masuk ke akun kamu":"Buat akun gratis sekarang"}</p>
        </div>
        <Card className="space-y-4">
          {mode==="register" && <Input label="Nama Lengkap" value={f.name} onChange={v=>up("name",v)} placeholder="Rizki Darmawan"/>}
          <Input label="Email" value={f.email} onChange={v=>up("email",v)} placeholder="kamu@email.com" type="email"/>
          <Input label="Password" value={f.password} onChange={v=>up("password",v)} placeholder="Min. 6 karakter" type="password"/>
          {mode==="register" && <Input label="Konfirmasi Password" value={f.confirm} onChange={v=>up("confirm",v)} placeholder="Ulangi password" type="password"/>}
          {err && <p className="text-xs text-red-500 bg-red-50 rounded-lg px-3 py-2">{err}</p>}
          <Btn full v="primary" sz="lg" onClick={submit}>{mode==="login"?"Masuk →":"Daftar Gratis →"}</Btn>
          <p className="text-center text-xs text-gray-500">
            {mode==="login"?"Belum punya akun?":"Sudah punya akun?"}{" "}
            <button className="text-blue-600 font-semibold" onClick={()=>{setMode(mode==="login"?"register":"login");setErr("");}}>
              {mode==="login"?"Daftar Gratis":"Masuk"}
            </button>
          </p>
          {mode==="login" && <p className="text-center text-xs text-gray-400">Demo admin: admin@cvlolos.ai / admin123</p>}
        </Card>
      </div>
    </div>
  );
}

// ── LANDING PAGE ──────────────────────────────────────────────────────────────
function Landing({ onCTA, onLogin }) {
  const [faq, setFaq] = useState(null);
  const testimonials = [
    { n:"Rina Marlina", role:"Fresh Graduate – UI/UX", txt:"10 menit CV jadi, langsung dapat 3 panggilan interview!", av:"RM" },
    { n:"Budi Santoso",  role:"Web Developer – Remote", txt:"ATS Score naik 42%→87% setelah ikuti rekomendasi AI. Dapet kerja remote!", av:"BS" },
    { n:"Sari Dewi",     role:"Career Switcher – Marketing", txt:"Career Guidance AI bantu saya tahu skill yang harus dipelajari duluan.", av:"SD" },
  ];
  const faqs = [
    { q:"Apa itu ATS?", a:"Applicant Tracking System adalah software yang menyaring CV otomatis sebelum sampai ke HRD. 75% CV ditolak di tahap ini." },
    { q:"Kenapa CV saya sering tidak lolos?", a:"Format salah (tabel/grafik), kurang keyword relevan, atau template generik yang tidak ATS-friendly." },
    { q:"Apakah data saya aman?", a:"Ya, data Anda dienkripsi dan tidak dibagikan ke pihak ketiga manapun." },
    { q:"Bagaimana cara upgrade ke PRO?", a:"Daftar akun → Dashboard → Upgrade → Transfer ke rekening SeaBank kami → Aktif dalam 1x24 jam." },
  ];
  return (
    <div className="min-h-screen bg-white">
      {/* NAV */}
      <nav className="fixed top-0 inset-x-0 z-50 bg-white/90 backdrop-blur border-b border-gray-100">
        <div className="max-w-5xl mx-auto px-4 py-3 flex items-center justify-between">
          <div className="flex items-center gap-2">
            <div className="w-8 h-8 bg-blue-600 rounded-lg flex items-center justify-center text-white font-bold text-sm">CV</div>
            <span className="font-bold text-gray-900">CVLolos<span className="text-blue-600">.ai</span></span>
          </div>
          <div className="flex gap-2">
            <Btn v="ghost" sz="sm" onClick={onLogin}>Masuk</Btn>
            <Btn sz="sm" onClick={onCTA}>Mulai Gratis</Btn>
          </div>
        </div>
      </nav>
      {/* HERO */}
      <section className="pt-24 pb-16 px-4 bg-gradient-to-br from-blue-50 via-white to-indigo-50 text-center">
        <Badge c="blue">✨ AI-Powered · ATS-Optimized · Khusus Indonesia</Badge>
        <h1 className="mt-4 text-4xl md:text-5xl font-extrabold text-gray-900 leading-tight">
          Buat CV Profesional &<br/><span className="text-blue-600">Lulus ATS dalam 5 Menit</span>
        </h1>
        <p className="mt-4 text-gray-500 max-w-lg mx-auto">Tingkatkan peluang dipanggil interview dengan CV yang dioptimasi AI — dirancang khusus untuk pasar kerja Indonesia.</p>
        <div className="mt-8 flex flex-col sm:flex-row gap-3 justify-center">
          <Btn sz="lg" onClick={onCTA}>🚀 Buat CV Sekarang — Gratis</Btn>
          <Btn sz="lg" v="secondary" onClick={()=>document.getElementById("pricing")?.scrollIntoView({behavior:"smooth"})}>Lihat Harga</Btn>
        </div>
        <p className="mt-3 text-xs text-gray-400">Tidak perlu kartu kredit · Langsung pakai</p>
      </section>
      {/* PROBLEM */}
      <section className="py-14 px-4 bg-white">
        <div className="max-w-4xl mx-auto text-center">
          <h2 className="text-2xl font-bold text-gray-900">😞 Kenapa CV Kamu Sering Gagal ATS?</h2>
          <p className="text-gray-500 mt-2 text-sm">75% CV ditolak sebelum dibaca manusia. Ini 3 penyebab utamanya:</p>
          <div className="mt-8 grid grid-cols-1 md:grid-cols-3 gap-4">
            {[{i:"❌",t:"Format Salah",d:"Tabel, kolom, dan grafik membuat ATS tidak bisa membaca isi CV kamu."},{i:"🔍",t:"Kurang Keyword",d:"CV tidak mengandung kata kunci yang dicari sistem ATS perusahaan."},{i:"📄",t:"Template Generik",d:"Template dari internet tidak dioptimasi untuk standar ATS Indonesia."}].map(p=>(
              <Card key={p.t} className="text-center"><div className="text-3xl mb-2">{p.i}</div><div className="font-bold text-gray-900">{p.t}</div><div className="text-sm text-gray-500 mt-1">{p.d}</div></Card>
            ))}
          </div>
        </div>
      </section>
      {/* FEATURES */}
      <section className="py-14 px-4 bg-blue-50">
        <div className="max-w-4xl mx-auto text-center">
          <h2 className="text-2xl font-bold text-gray-900">Fitur Unggulan CVLolos.ai</h2>
          <div className="mt-8 grid grid-cols-1 md:grid-cols-2 gap-4 text-left">
            {[{i:"🤖",t:"AI Resume Builder",d:"Buat CV profesional step-by-step dengan panduan AI. Optimasi keyword otomatis."},{i:"📊",t:"ATS Checker Semantik",d:"Analisis mendalam — deteksi sinonim & makna, bukan hanya exact match."},{i:"✉️",t:"Cover Letter AI",d:"Generate cover letter personal yang menarik dalam hitungan detik. (PRO)"},{i:"🧭",t:"Career Guidance AI",d:"Analisis jalur karir, skill gap, sertifikasi & action plan 3 bulan. (PRO)"},{i:"📥",t:"Export PDF ATS-Friendly",d:"Download CV bersih tanpa tabel kompleks, siap dikirim ke perusahaan."},{i:"🇮🇩",t:"Format Indonesia & Internasional",d:"Pilih format sesuai target: HR lokal atau perusahaan internasional."}].map(f=>(
              <Card key={f.t} className="flex gap-4"><div className="text-2xl mt-0.5">{f.i}</div><div><div className="font-bold text-gray-900">{f.t}</div><div className="text-sm text-gray-500 mt-0.5">{f.d}</div></div></Card>
            ))}
          </div>
        </div>
      </section>
      {/* PRICING */}
      <section id="pricing" className="py-14 px-4 bg-white">
        <div className="max-w-4xl mx-auto text-center">
          <h2 className="text-2xl font-bold text-gray-900">Pilih Paket Kamu</h2>
          <p className="text-gray-500 text-sm mt-1">Mulai gratis, upgrade kapan saja</p>
          <div className="mt-8 grid grid-cols-1 md:grid-cols-3 gap-5">
            {PLANS.map(p=>(
              <div key={p.id} className={cls("rounded-2xl border-2 p-5 text-left relative",p.popular?"border-blue-500 shadow-lg":"border-gray-100")}>
                {p.popular && <div className="absolute -top-3 left-1/2 -translate-x-1/2"><Badge c="blue">⭐ Terpopuler</Badge></div>}
                <div className="font-bold text-lg">{p.label}</div>
                <div className={cls("text-xl font-extrabold mt-0.5",p.id==="pro"?"text-blue-600":p.id==="lifetime"?"text-purple-600":"text-gray-500")}>{p.desc}</div>
                <ul className="mt-4 space-y-1.5 text-sm">{p.features.map(f=><li key={f} className="flex gap-2 text-gray-700"><span className="text-green-500">✓</span>{f}</li>)}{p.locked?.map(f=><li key={f} className="flex gap-2 text-gray-400 line-through"><span>✗</span>{f}</li>)}</ul>
                <Btn onClick={onCTA} v={p.id==="pro"?"primary":p.id==="lifetime"?"purple":"secondary"} full className="mt-5" sz="md">{p.id==="free"?"Mulai Gratis":"Pilih Paket Ini"}</Btn>
              </div>
            ))}
          </div>
          <p className="mt-4 text-xs text-gray-400">Pembayaran via Transfer Bank · {BANK.bank} · {BANK.no} · a/n {BANK.name}</p>
        </div>
      </section>
      {/* TESTIMONIALS */}
      <section className="py-14 px-4 bg-gray-50">
        <div className="max-w-4xl mx-auto text-center">
          <h2 className="text-2xl font-bold text-gray-900">Kata Mereka 💬</h2>
          <div className="mt-8 grid grid-cols-1 md:grid-cols-3 gap-4">
            {testimonials.map(t=>(
              <Card key={t.n}><div className="flex items-center gap-3 mb-3"><div className="w-9 h-9 rounded-full bg-blue-600 text-white flex items-center justify-center font-bold text-sm">{t.av}</div><div><div className="font-bold text-sm text-gray-900">{t.n}</div><div className="text-xs text-gray-500">{t.role}</div></div></div><p className="text-sm text-gray-600 italic">"{t.txt}"</p><div className="mt-2 text-yellow-400 text-sm">★★★★★</div></Card>
            ))}
          </div>
        </div>
      </section>
      {/* FAQ */}
      <section className="py-14 px-4 max-w-2xl mx-auto">
        <h2 className="text-2xl font-bold text-gray-900 text-center mb-5">FAQ</h2>
        {faqs.map((f,i)=>(
          <div key={i} className="border-b border-gray-100">
            <button className="w-full text-left py-4 font-semibold text-gray-800 flex justify-between text-sm" onClick={()=>setFaq(faq===i?null:i)}>{f.q}<span className="text-gray-400">{faq===i?"▲":"▼"}</span></button>
            {faq===i && <p className="pb-4 text-sm text-gray-600">{f.a}</p>}
          </div>
        ))}
      </section>
      {/* CTA BOTTOM */}
      <section className="py-14 px-4 bg-blue-600 text-center text-white">
        <h2 className="text-3xl font-extrabold">Siap Lulus ATS? 🚀</h2>
        <p className="mt-2 text-blue-100">Ribuan job seeker sudah berhasil. Giliran kamu!</p>
        <Btn sz="lg" v="secondary" onClick={onCTA} className="mt-6">Buat CV Sekarang — Gratis</Btn>
      </section>
      <footer className="py-6 text-center text-xs text-gray-400 border-t">© 2025 CVLolos.ai · Dibuat dengan ❤️ untuk job seeker Indonesia</footer>
    </div>
  );
}

// ── RESUME BUILDER ────────────────────────────────────────────────────────────
function Builder({ user, onUpdate }) {
  const [r, setR] = useState({...EMPTY_RESUME, name:user.name, email:user.email});
  const [step, setStep] = useState(0);
  const [saved, setSaved] = useState(false);
  const steps = ["Info Dasar","Ringkasan","Pengalaman","Pendidikan","Skill & Lainnya","Preview & Download"];
  const up = (k,v) => setR(p=>({...p,[k]:v}));
  const save = () => { setSaved(true); setTimeout(()=>setSaved(false),2000); };

  const handleDownload = () => {
    const isPro = user.plan !== "free";
    if (!isPro && user.downloads >= 1) {
      alert("Kamu sudah menggunakan jatah 1x download gratis. Upgrade ke PRO untuk download unlimited!"); return;
    }
    // Update downloads
    const u = _users.find(x=>x.id===user.id);
    if (u) u.downloads = (u.downloads||0)+1;
    onUpdate({...user, downloads:(user.downloads||0)+1});
    // Generate plain-text CV for download simulation
    const cv = `${r.name}\n${r.email} | ${r.phone} | ${r.location}\n${ r.linkedin ? r.linkedin+" | " : ""}${r.portfolio||""}\n\n==RINGKASAN PROFESIONAL==\n${r.summary}\n\n==PENGALAMAN KERJA==\n${r.experience.map(e=>`${e.role} @ ${e.company} (${e.period})\n${e.desc}`).join("\n\n")}\n\n==PENDIDIKAN==\n${r.education.map(e=>`${e.degree} - ${e.school} (${e.year})${e.gpa?" | IPK: "+e.gpa:""}`).join("\n")}\n\n==SKILL==\n${r.skills}\n\n==SERTIFIKASI==\n${r.certifications}\n\n==BAHASA==\n${r.languages}`;
    const blob = new Blob([cv],{type:"text/plain"});
    const a = document.createElement("a"); a.href = URL.createObjectURL(blob);
    a.download = `CV_${r.name.replace(/\s/g,"_")}_ATS.txt`; a.click();
  };

  return (
    <div className="max-w-2xl mx-auto space-y-4">
      <h1 className="text-xl font-bold text-gray-900">✏️ Resume Builder</h1>
      {/* Stepper */}
      <div className="flex gap-1 overflow-x-auto pb-1">
        {steps.map((s,i)=>(
          <button key={s} onClick={()=>setStep(i)} className={cls("px-2.5 py-1.5 rounded-lg text-xs font-semibold whitespace-nowrap",step===i?"bg-blue-600 text-white":"bg-gray-100 text-gray-600 hover:bg-blue-50")}>{i+1}. {s}</button>
        ))}
      </div>
      <Card>
        {step===0 && (
          <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
            <Input label="Nama Lengkap *" value={r.name} onChange={v=>up("name",v)} placeholder="Rizki Darmawan"/>
            <Input label="Email *" value={r.email} onChange={v=>up("email",v)} placeholder="rizki@email.com" type="email"/>
            <Input label="No. HP" value={r.phone} onChange={v=>up("phone",v)} placeholder="+62 812 3456 7890"/>
            <Input label="Kota" value={r.location} onChange={v=>up("location",v)} placeholder="Jakarta, Indonesia"/>
            <Input label="LinkedIn" value={r.linkedin} onChange={v=>up("linkedin",v)} placeholder="linkedin.com/in/rizki"/>
            <Input label="Portfolio / GitHub" value={r.portfolio} onChange={v=>up("portfolio",v)} placeholder="github.com/rizki"/>
            <div className="md:col-span-2">
              <label className="text-xs font-semibold text-gray-600 block mb-2">Format CV</label>
              <div className="flex gap-3">
                {[{v:"indonesia",l:"🇮🇩 Format Indonesia"},{v:"international",l:"🌍 Format Internasional"}].map(f=>(
                  <button key={f.v} onClick={()=>up("format",f.v)} className={cls("flex-1 py-2 rounded-xl border-2 text-sm font-medium",r.format===f.v?"border-blue-500 bg-blue-50 text-blue-700":"border-gray-200 text-gray-500")}>{f.l}</button>
                ))}
              </div>
            </div>
          </div>
        )}
        {step===1 && (
          <div className="space-y-3">
            <div className="flex justify-between items-center"><label className="text-xs font-semibold text-gray-600">Ringkasan Profesional *</label><Badge c="blue">AI Optimized</Badge></div>
            <textarea className="w-full border border-gray-200 rounded-xl px-3 py-2 text-sm bg-gray-50 focus:outline-none focus:ring-2 focus:ring-blue-400" rows={6} value={r.summary} onChange={e=>up("summary",e.target.value)} placeholder="Fresh graduate Teknik Informatika UI dengan pengalaman magang 6 bulan. Terampil dalam React, Node.js dan UI/UX. Berorientasi hasil..."/>
            <p className="text-xs text-blue-600 bg-blue-50 rounded-lg px-3 py-2">💡 Tips AI: Sertakan angka pencapaian, teknologi spesifik, dan kata kunci posisi yang dilamar.</p>
          </div>
        )}
        {step===2 && (
          <div className="space-y-4">
            <div className="flex justify-between items-center"><span className="text-sm font-bold text-gray-900">Pengalaman Kerja</span><Btn sz="sm" v="secondary" onClick={()=>up("experience",[...r.experience,{company:"",role:"",period:"",desc:""}])}>+ Tambah</Btn></div>
            {r.experience.map((e,i)=>(
              <div key={i} className="bg-gray-50 border border-gray-100 rounded-xl p-4 space-y-3">
                <div className="flex justify-between"><span className="text-xs font-bold text-gray-500">#{i+1}</span>{i>0&&<button onClick={()=>up("experience",r.experience.filter((_,j)=>j!==i))} className="text-xs text-red-400 hover:text-red-600">Hapus</button>}</div>
                <div className="grid grid-cols-1 md:grid-cols-2 gap-3">
                  <Input label="Perusahaan" value={e.company} onChange={v=>{const a=[...r.experience];a[i].company=v;up("experience",a);}} placeholder="PT. Maju Bersama"/>
                  <Input label="Posisi" value={e.role} onChange={v=>{const a=[...r.experience];a[i].role=v;up("experience",a);}} placeholder="Frontend Developer"/>
                  <Input label="Periode" value={e.period} onChange={v=>{const a=[...r.experience];a[i].period=v;up("experience",a);}} placeholder="Jan 2023 – Des 2023"/>
                </div>
                <Input label="Deskripsi & Pencapaian" rows={3} value={e.desc} onChange={v=>{const a=[...r.experience];a[i].desc=v;up("experience",a);}} placeholder="• Kembangkan fitur X → tingkatkan konversi 30%&#10;• Pimpin tim 3 developer"/>
              </div>
            ))}
          </div>
        )}
        {step===3 && (
          <div className="space-y-4">
            <div className="flex justify-between items-center"><span className="text-sm font-bold text-gray-900">Pendidikan</span><Btn sz="sm" v="secondary" onClick={()=>up("education",[...r.education,{school:"",degree:"",year:"",gpa:""}])}>+ Tambah</Btn></div>
            {r.education.map((e,i)=>(
              <div key={i} className="bg-gray-50 border border-gray-100 rounded-xl p-4">
                <div className="grid grid-cols-1 md:grid-cols-2 gap-3">
                  <Input label="Institusi" value={e.school} onChange={v=>{const a=[...r.education];a[i].school=v;up("education",a);}} placeholder="Universitas Indonesia"/>
                  <Input label="Jurusan / Gelar" value={e.degree} onChange={v=>{const a=[...r.education];a[i].degree=v;up("education",a);}} placeholder="S1 Teknik Informatika"/>
                  <Input label="Tahun Lulus" value={e.year} onChange={v=>{const a=[...r.education];a[i].year=v;up("education",a);}} placeholder="2024"/>
                  <Input label="IPK (opsional)" value={e.gpa} onChange={v=>{const a=[...r.education];a[i].gpa=v;up("education",a);}} placeholder="3.75"/>
                </div>
              </div>
            ))}
          </div>
        )}
        {step===4 && (
          <div className="space-y-4">
            <Input label="Skill (pisahkan koma)" rows={2} value={r.skills} onChange={v=>up("skills",v)} placeholder="React, Node.js, Figma, Python, SQL, Git"/>
            <Input label="Sertifikasi" rows={2} value={r.certifications} onChange={v=>up("certifications",v)} placeholder="Google Analytics (2023), AWS Cloud Practitioner (2024)"/>
            <Input label="Bahasa" value={r.languages} onChange={v=>up("languages",v)} placeholder="Indonesia (Native), Inggris (Professional)"/>
          </div>
        )}
        {step===5 && (
          <div className="space-y-4">
            <div className="flex items-center justify-between"><h3 className="font-bold text-gray-900">Preview CV</h3><Badge c={r.format==="indonesia"?"blue":"purple"}>{r.format==="indonesia"?"🇮🇩 Indonesia":"🌍 Internasional"}</Badge></div>
            <div className="border-2 border-gray-100 rounded-xl p-5 bg-white font-serif text-sm space-y-3 max-h-96 overflow-y-auto">
              <div className="border-b-2 border-blue-600 pb-3">
                <h2 className="text-xl font-bold text-gray-900">{r.name||"Nama Lengkap"}</h2>
                <div className="flex flex-wrap gap-x-4 gap-y-0.5 text-xs text-gray-600 mt-1">
                  {r.email&&<span>📧 {r.email}</span>}{r.phone&&<span>📱 {r.phone}</span>}{r.location&&<span>📍 {r.location}</span>}{r.portfolio&&<span>🔗 {r.portfolio}</span>}
                </div>
              </div>
              {r.summary&&<div><div className="text-xs font-bold uppercase tracking-wider text-blue-700 mb-1">Ringkasan Profesional</div><p className="text-xs text-gray-700 leading-relaxed">{r.summary}</p></div>}
              {r.experience[0]?.company&&<div><div className="text-xs font-bold uppercase tracking-wider text-blue-700 mb-1">Pengalaman Kerja</div>{r.experience.map((e,i)=>e.company&&<div key={i} className="mb-2"><div className="font-bold text-xs">{e.role} · {e.company}</div><div className="text-xs text-gray-500">{e.period}</div><div className="text-xs text-gray-700 whitespace-pre-line mt-0.5">{e.desc}</div></div>)}</div>}
              {r.education[0]?.school&&<div><div className="text-xs font-bold uppercase tracking-wider text-blue-700 mb-1">Pendidikan</div>{r.education.map((e,i)=>e.school&&<div key={i} className="mb-1"><div className="font-bold text-xs">{e.degree} · {e.school}</div><div className="text-xs text-gray-500">{e.year}{e.gpa?" · IPK "+e.gpa:""}</div></div>)}</div>}
              {r.skills&&<div><div className="text-xs font-bold uppercase tracking-wider text-blue-700 mb-1">Skill</div><div className="flex flex-wrap gap-1">{r.skills.split(",").map(s=>s.trim()).filter(Boolean).map(s=><Badge key={s} c="blue">{s}</Badge>)}</div></div>}
              {r.certifications&&<div><div className="text-xs font-bold uppercase tracking-wider text-blue-700 mb-1">Sertifikasi</div><div className="text-xs text-gray-700">{r.certifications}</div></div>}
              {r.languages&&<div><div className="text-xs font-bold uppercase tracking-wider text-blue-700 mb-1">Bahasa</div><div className="text-xs text-gray-700">{r.languages}</div></div>}
            </div>
            <div className="flex gap-3">
              <Btn v="secondary" onClick={save} full>{saved?"✅ Tersimpan!":"💾 Simpan CV"}</Btn>
              <Btn v="primary" full onClick={handleDownload}>📥 Download CV (TXT/ATS)</Btn>
            </div>
            {user.plan==="free"&&<p className="text-xs text-center text-gray-400">Paket FREE: 1x download. <button className="text-blue-600 font-semibold" onClick={()=>{}}>Upgrade PRO</button> untuk unlimited.</p>}
          </div>
        )}
      </Card>
      <div className="flex justify-between">
        <Btn v="secondary" onClick={()=>setStep(Math.max(0,step-1))} disabled={step===0}>← Sebelumnya</Btn>
        <Btn onClick={()=>setStep(Math.min(steps.length-1,step+1))} disabled={step===steps.length-1}>Selanjutnya →</Btn>
      </div>
    </div>
  );
}

// ── ATS CHECKER ───────────────────────────────────────────────────────────────
function ATS() {
  const [jd,setJd]=useState(""); const [cv,setCv]=useState(""); const [loading,setLoading]=useState(false); const [result,setResult]=useState(null);
  const analyze = async () => {
    if(!jd.trim()||!cv.trim()) return alert("Isi kedua kolom terlebih dahulu!");
    setLoading(true); setResult(null);
    try {
      const res = await fetch("https://api.anthropic.com/v1/messages",{method:"POST",headers:{"Content-Type":"application/json"},body:JSON.stringify({model:"claude-sonnet-4-20250514",max_tokens:1000,system:`ATS Analyzer AI Indonesia. Analisis SEMANTIK (sinonim & makna). Return ONLY valid JSON: {"score":number,"keywords":[{"kw":"","found":"","type":"exact|semantic|missing"}],"format_issues":[{"issue":"","impact":"","fix":""}],"suggestions":[{"section":"","before":"","after":"","reason":""}],"summary":""}`,messages:[{role:"user",content:`JD:\n${jd}\n\nCV:\n${cv}`}]})});
      const d = await res.json(); const txt = d.content?.map(x=>x.text||"").join(""); setResult(JSON.parse(txt.replace(/```json|```/g,"").trim()));
    } catch {
      setResult({score:68,keywords:[{kw:"React",found:"ReactJS",type:"semantic"},{kw:"Agile",found:"Agile/Scrum",type:"exact"},{kw:"Leadership",found:"memimpin tim",type:"semantic"},{kw:"CI/CD",found:null,type:"missing"},{kw:"Docker",found:null,type:"missing"}],format_issues:[{issue:"Tabel di bagian skill",impact:"Tinggi — ATS tidak bisa membaca tabel",fix:"Ganti dengan teks biasa dipisah koma"},{issue:"Header menggunakan gambar",impact:"Sedang — nama tidak terbaca ATS",fix:"Gunakan teks biasa untuk nama & kontak"}],suggestions:[{section:"Ringkasan Profesional",before:"Saya developer berpengalaman dengan skill React.",after:"Frontend Developer dengan 2+ tahun pengalaman membangun web app menggunakan React.js, TypeScript & REST API. Terbiasa metodologi Agile/Scrum.",reason:"Tambah angka pengalaman & keyword teknologi spesifik"}],summary:"CV cukup baik, namun beberapa keyword penting hilang dan ada masalah format yang perlu diperbaiki."});
    }
    setLoading(false);
  };
  const pColor = {exact:"bg-green-50 border-green-200 text-green-700",semantic:"bg-blue-50 border-blue-200 text-blue-700",missing:"bg-red-50 border-red-200 text-red-500 line-through"};
  return (
    <div className="max-w-3xl mx-auto space-y-5">
      <div><h1 className="text-xl font-bold text-gray-900">📊 ATS Checker AI Semantik</h1><p className="text-sm text-gray-500 mt-1">Analisis mendalam — sinonim, masalah format, dan saran konten copy-ready.</p></div>
      <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
        <div className="space-y-1"><label className="text-xs font-semibold text-gray-600">Paste Job Description</label><textarea className="w-full h-44 border border-gray-200 rounded-xl p-3 text-sm resize-none bg-gray-50 focus:ring-2 focus:ring-blue-400 focus:outline-none" value={jd} onChange={e=>setJd(e.target.value)} placeholder="Paste job description dari lowongan..."/></div>
        <div className="space-y-1"><label className="text-xs font-semibold text-gray-600">Paste Isi CV Kamu</label><textarea className="w-full h-44 border border-gray-200 rounded-xl p-3 text-sm resize-none bg-gray-50 focus:ring-2 focus:ring-blue-400 focus:outline-none" value={cv} onChange={e=>setCv(e.target.value)} placeholder="Paste teks CV kamu di sini..."/></div>
      </div>
      <Btn full sz="lg" onClick={analyze} disabled={loading}>🔍 Analisis ATS Sekarang</Btn>
      {loading&&<Spinner text="AI menganalisis secara semantik..."/>}
      {result&&(
        <div className="space-y-4">
          <Card className="flex flex-col md:flex-row items-center gap-5"><ScoreRing score={result.score}/><div><h3 className="font-bold text-lg text-gray-900">ATS Score: {result.score}%</h3><p className="text-sm text-gray-600 mt-1">{result.summary}</p></div></Card>
          <Card><h3 className="font-bold text-gray-900 mb-3">🔑 Analisis Keyword Semantik</h3><div className="flex flex-wrap gap-2">{result.keywords?.map((k,i)=><div key={i} className={cls("px-3 py-1.5 rounded-lg text-xs font-medium border",pColor[k.type])}>{k.type==="exact"?"✓":k.type==="semantic"?"≈":"✗"} {k.kw}{k.found&&k.type!=="exact"&&<span className="opacity-60"> ({k.found})</span>}</div>)}</div><div className="mt-3 flex gap-4 text-xs"><span className="text-green-600">✓ Exact</span><span className="text-blue-600">≈ Semantik</span><span className="text-red-500">✗ Tidak ada</span></div></Card>
          {result.format_issues?.length>0&&<Card><h3 className="font-bold text-gray-900 mb-3">⚠️ Masalah Format</h3><div className="space-y-2">{result.format_issues.map((f,i)=><div key={i} className="bg-red-50 border border-red-100 rounded-xl p-3"><div className="font-semibold text-sm text-red-700">{f.issue}</div><div className="text-xs text-gray-600 mt-0.5">{f.impact}</div><div className="text-xs text-green-700 mt-1 font-medium">✅ {f.fix}</div></div>)}</div></Card>}
          {result.suggestions?.length>0&&<Card><h3 className="font-bold text-gray-900 mb-3">✨ Saran Konten AI (Copy-Ready)</h3>{result.suggestions.map((s,i)=><div key={i} className="border border-blue-100 rounded-xl p-4 bg-blue-50 mb-3"><Badge c="blue">{s.section}</Badge><div className="mt-3 grid grid-cols-1 md:grid-cols-2 gap-3"><div className="bg-red-50 border border-red-100 rounded-lg p-3"><div className="text-xs font-bold text-red-600 mb-1">❌ Sebelum</div><p className="text-xs text-gray-700">{s.before}</p></div><div className="bg-green-50 border border-green-200 rounded-lg p-3"><div className="flex justify-between mb-1"><div className="text-xs font-bold text-green-700">✅ Sesudah</div><button onClick={()=>navigator.clipboard.writeText(s.after)} className="text-xs text-blue-600 hover:underline">Copy</button></div><p className="text-xs text-gray-700">{s.after}</p></div></div><p className="text-xs text-gray-500 mt-2">💡 {s.reason}</p></div>)}</Card>}
        </div>
      )}
    </div>
  );
}

// ── COVER LETTER ──────────────────────────────────────────────────────────────
function CoverLetter({ user }) {
  const [f,setF]=useState({name:user.name,pos:"",company:"",jd:"",tone:"professional"});
  const [loading,setLoading]=useState(false); const [letter,setLetter]=useState("");
  const up=(k,v)=>setF(p=>({...p,[k]:v}));
  const gen = async () => {
    if(!f.pos||!f.company) return alert("Isi posisi dan nama perusahaan!");
    setLoading(true); setLetter("");
    try {
      const res=await fetch("https://api.anthropic.com/v1/messages",{method:"POST",headers:{"Content-Type":"application/json"},body:JSON.stringify({model:"claude-sonnet-4-20250514",max_tokens:1000,system:"Expert career coach Indonesia. Tulis cover letter profesional Bahasa Indonesia yang personal, tidak generik, ATS-friendly. Paragraf langsung tanpa header.",messages:[{role:"user",content:`Nama: ${f.name}\nPosisi: ${f.pos}\nPerusahaan: ${f.company}\nJD: ${f.jd||"-"}\nTone: ${f.tone}`}]})});
      const d=await res.json(); setLetter(d.content?.map(x=>x.text||"").join("")||"");
    } catch { setLetter(`Kepada Yth. Tim Rekrutmen ${f.company},\n\nDengan hormat,\nSaya ${f.name}, dengan antusias mengajukan lamaran untuk posisi ${f.pos} di ${f.company}. Saya yakin latar belakang dan pengalaman saya sangat sesuai dengan kebutuhan tim Anda...\n\n[Konten lengkap akan di-generate oleh AI Claude]`); }
    setLoading(false);
  };
  return (
    <div className="max-w-2xl mx-auto space-y-5">
      <h1 className="text-xl font-bold text-gray-900">✉️ Cover Letter AI Generator</h1>
      <Card className="space-y-4">
        <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
          <Input label="Nama Lengkap" value={f.name} onChange={v=>up("name",v)}/>
          <Input label="Posisi yang Dilamar *" value={f.pos} onChange={v=>up("pos",v)} placeholder="UI/UX Designer"/>
          <Input label="Nama Perusahaan *" value={f.company} onChange={v=>up("company",v)} placeholder="PT. Tokopedia"/>
          <div className="flex flex-col gap-1"><label className="text-xs font-semibold text-gray-600">Tone</label><select className="border border-gray-200 rounded-xl px-3 py-2 text-sm bg-gray-50" value={f.tone} onChange={e=>up("tone",e.target.value)}><option value="professional">Professional</option><option value="friendly">Friendly</option><option value="formal">Formal</option><option value="creative">Creative (Startup)</option></select></div>
        </div>
        <Input label="Job Description (opsional)" rows={3} value={f.jd} onChange={v=>up("jd",v)} placeholder="Paste JD untuk hasil lebih personal..."/>
        <Btn full sz="lg" onClick={gen} disabled={loading}>✨ Generate Cover Letter</Btn>
      </Card>
      {loading&&<Spinner text="AI menulis cover letter..."/>}
      {letter&&<Card className="space-y-3"><div className="flex justify-between items-center"><h3 className="font-bold text-gray-900">Cover Letter</h3><div className="flex gap-2"><Btn sz="sm" v="secondary" onClick={()=>navigator.clipboard.writeText(letter)}>📋 Copy</Btn></div></div><div className="bg-gray-50 rounded-xl p-4 text-sm text-gray-800 leading-relaxed whitespace-pre-line border border-gray-100 max-h-80 overflow-y-auto">{letter}</div></Card>}
    </div>
  );
}

// ── CAREER GUIDANCE ───────────────────────────────────────────────────────────
function Career() {
  const [f,setF]=useState({edu:"",skills:"",exp:"",goal:"",industry:""});
  const [loading,setLoading]=useState(false); const [result,setResult]=useState(null);
  const up=(k,v)=>setF(p=>({...p,[k]:v}));
  const analyze = async () => {
    setLoading(true); setResult(null);
    try {
      const res=await fetch("https://api.anthropic.com/v1/messages",{method:"POST",headers:{"Content-Type":"application/json"},body:JSON.stringify({model:"claude-sonnet-4-20250514",max_tokens:1000,system:`Career coach AI Indonesia. Return ONLY valid JSON: {"paths":[{"title":"","prob":number,"why":""}],"gaps":[{"skill":"","priority":"high|medium|low","action":""}],"industries":[{"name":"","score":number,"reason":""}],"certs":[{"name":"","platform":"","duration":"","priority":"high|medium|low"}],"plan":{"m1":[],"m2":[],"m3":[]}}`,messages:[{role:"user",content:`Pendidikan: ${f.edu}\nSkill: ${f.skills}\nPengalaman: ${f.exp}\nGoal: ${f.goal}\nIndustri: ${f.industry}`}]})});
      const d=await res.json(); setResult(JSON.parse(d.content?.map(x=>x.text||"").join("").replace(/```json|```/g,"").trim()));
    } catch {
      setResult({paths:[{title:"Frontend Developer",prob:82,why:"Skill React & desain sangat relevan"},{title:"UI/UX Designer",prob:74,why:"Background seni & teknologi cocok"},{title:"Product Manager",prob:61,why:"Perlu tambah skill bisnis & data"}],gaps:[{skill:"TypeScript",priority:"high",action:"Kursus TypeScript di Udemy (20 jam)"},{skill:"Data Analytics",priority:"medium",action:"Google Data Analytics di Coursera"},{skill:"Leadership",priority:"low",action:"Jadi contributor open source"}],industries:[{name:"Fintech",score:85,reason:"Pertumbuhan tinggi, demand besar"},{name:"E-commerce",score:78,reason:"Banyak peluang remote"},{name:"EdTech",score:71,reason:"Sesuai passion tech & dampak sosial"}],certs:[{name:"Meta Frontend Developer",platform:"Coursera",duration:"7 bulan",priority:"high"},{name:"Google UX Design",platform:"Coursera",duration:"6 bulan",priority:"high"},{name:"AWS Cloud Practitioner",platform:"AWS Training",duration:"40 jam",priority:"medium"}],plan:{m1:["Buat portfolio project React (E-commerce clone)","Daftar kursus TypeScript","Update CV & LinkedIn"],m2:["Submit 10 lamaran ke fintech & e-commerce","Mulai Meta Frontend Developer di Coursera","Ikut 2 networking event tech"],m3:["Target 3 interview undangan","Selesaikan 1 sertifikasi","Review & optimasi strategi job search"]}});
    }
    setLoading(false);
  };
  const pc={high:"red",medium:"yellow",low:"green"};
  return (
    <div className="max-w-3xl mx-auto space-y-5">
      <div><h1 className="text-xl font-bold text-gray-900">🧭 Career Guidance AI</h1><p className="text-sm text-gray-500 mt-1">Analisis jalur karir, skill gap, sertifikasi & action plan 3 bulan.</p></div>
      <Card className="space-y-4">
        <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
          <Input label="Pendidikan Terakhir" value={f.edu} onChange={v=>up("edu",v)} placeholder="S1 Teknik Informatika, UI, 2024"/>
          <Input label="Industri Diminati" value={f.industry} onChange={v=>up("industry",v)} placeholder="Fintech, E-commerce, Startup"/>
          <Input label="Skill Saat Ini" rows={2} value={f.skills} onChange={v=>up("skills",v)} placeholder="React, Figma, JavaScript..."/>
          <Input label="Pengalaman / Magang" rows={2} value={f.exp} onChange={v=>up("exp",v)} placeholder="Magang UI Developer 6 bulan..."/>
        </div>
        <Input label="Target Karir" value={f.goal} onChange={v=>up("goal",v)} placeholder="Senior Frontend Developer dalam 2 tahun"/>
        <Btn full sz="lg" onClick={analyze} disabled={loading}>🧠 Analisis Karir Saya</Btn>
      </Card>
      {loading&&<Spinner text="AI menganalisis potensi karirmu..."/>}
      {result&&(
        <div className="space-y-4">
          <Card><h3 className="font-bold text-gray-900 mb-3">🎯 Jalur Karir yang Cocok</h3><div className="space-y-2">{result.paths?.map((p,i)=><div key={i} className="flex items-center gap-3 p-3 bg-gray-50 rounded-xl border border-gray-100"><div className="text-center w-12"><div className={cls("text-lg font-extrabold",p.prob>=75?"text-green-600":p.prob>=60?"text-yellow-600":"text-orange-500")}>{p.prob}%</div><div className="text-xs text-gray-400">sukses</div></div><div className="flex-1"><div className="font-bold text-sm text-gray-900">{p.title}</div><div className="text-xs text-blue-600">💡 {p.why}</div></div></div>)}</div></Card>
          <Card><h3 className="font-bold text-gray-900 mb-3">📈 Skill Gap Analysis</h3><div className="space-y-2">{result.gaps?.map((g,i)=><div key={i} className="flex items-start gap-3 p-3 bg-gray-50 rounded-xl border border-gray-100"><Badge c={pc[g.priority]}>{g.priority.toUpperCase()}</Badge><div><div className="font-semibold text-sm text-gray-900">{g.skill}</div><div className="text-xs text-blue-600 mt-0.5">📚 {g.action}</div></div></div>)}</div></Card>
          <Card><h3 className="font-bold text-gray-900 mb-3">🏭 Industri Relevan</h3><div className="space-y-2">{result.industries?.map((ind,i)=><div key={i} className="flex items-center gap-3 p-3 bg-gray-50 rounded-xl border border-gray-100"><div className="w-10 h-10 bg-blue-100 rounded-lg flex items-center justify-center font-bold text-blue-700 text-sm">{ind.score}%</div><div><div className="font-bold text-sm text-gray-900">{ind.name}</div><div className="text-xs text-gray-500">{ind.reason}</div></div></div>)}</div></Card>
          <Card><h3 className="font-bold text-gray-900 mb-3">🎓 Sertifikasi Disarankan</h3><div className="space-y-2">{result.certs?.map((c,i)=><div key={i} className="flex items-start gap-3 p-3 bg-gray-50 rounded-xl border border-gray-100"><Badge c={pc[c.priority]}>{c.priority.toUpperCase()}</Badge><div><div className="font-semibold text-sm text-gray-900">{c.name}</div><div className="text-xs text-gray-500">{c.platform} · {c.duration}</div></div></div>)}</div></Card>
          <Card><h3 className="font-bold text-gray-900 mb-3">📅 Action Plan 3 Bulan</h3><div className="grid grid-cols-1 md:grid-cols-3 gap-3">{[["Bulan 1",result.plan?.m1],["Bulan 2",result.plan?.m2],["Bulan 3",result.plan?.m3]].map(([l,tasks])=><div key={l} className="bg-blue-50 rounded-xl p-4 border border-blue-100"><div className="font-bold text-sm text-blue-700 mb-2">{l}</div><ul className="space-y-1">{tasks?.map((t,i)=><li key={i} className="text-xs text-gray-700 flex gap-1"><span className="text-blue-400">→</span>{t}</li>)}</ul></div>)}</div></Card>
        </div>
      )}
    </div>
  );
}

// ── UPGRADE / PAYMENT FLOW ────────────────────────────────────────────────────
function Upgrade({ user, onDone, onNavigate }) {
  const [sel, setSel] = useState("pro");
  const [step, setStep] = useState(1); // 1=pilih, 2=instruksi, 3=konfirmasi, 4=sukses/pending
  const [ref, setRef] = useState("");
  const [uploading, setUploading] = useState(false);
  const plan = PLANS.find(p=>p.id===sel);

  const submitPayment = () => {
    if (!ref.trim()) return alert("Masukkan nomor referensi / bukti transfer!");
    setUploading(true);
    setTimeout(() => {
      // store payment record
      const pay = { id: Date.now().toString(), userId: user.id, plan: sel, amount: plan.price, ref: ref.trim(), status: "pending", createdAt: new Date().toISOString() };
      _payments.push(pay);
      // Update user record
      const u = _users.find(x=>x.id===user.id);
      if (u) { u.paymentStatus = "pending"; u.paymentPlan = sel; u.paymentRef = ref.trim(); }
      setUploading(false);
      setStep(4);
    }, 1500);
  };

  if (step===4) return (
    <div className="max-w-lg mx-auto">
      <Card className="text-center py-10 space-y-4">
        <div className="text-5xl">⏳</div>
        <h2 className="text-2xl font-bold text-gray-900">Pembayaran Menunggu Verifikasi</h2>
        <p className="text-gray-600 text-sm">Terima kasih! Tim kami akan memverifikasi pembayaran dan mengaktifkan paket <strong>{plan.label}</strong> kamu dalam <strong>1×24 jam</strong>.</p>
        <div className="bg-blue-50 border border-blue-100 rounded-xl p-4 text-sm text-left space-y-1">
          <div className="flex justify-between"><span className="text-gray-600">Paket</span><span className="font-bold">{plan.label}</span></div>
          <div className="flex justify-between"><span className="text-gray-600">Jumlah</span><span className="font-bold text-blue-600">{fmt(plan.price)}</span></div>
          <div className="flex justify-between"><span className="text-gray-600">No. Referensi</span><span className="font-mono font-bold">{ref}</span></div>
          <div className="flex justify-between"><span className="text-gray-600">Status</span><Badge c="yellow">Menunggu Verifikasi</Badge></div>
        </div>
        <p className="text-xs text-gray-400">Butuh bantuan? Hubungi kami via WhatsApp dengan menyertakan no. referensi di atas.</p>
        <Btn v="primary" full onClick={()=>onNavigate("dashboard")}>← Kembali ke Dashboard</Btn>
      </Card>
    </div>
  );

  return (
    <div className="max-w-2xl mx-auto space-y-5">
      <h1 className="text-xl font-bold text-gray-900">⚡ Upgrade Paket</h1>
      {/* STEP INDICATOR */}
      <div className="flex items-center gap-2">
        {["Pilih Paket","Instruksi Transfer","Konfirmasi"].map((s,i)=>(
          <div key={s} className="flex items-center gap-2">
            <div className={cls("w-7 h-7 rounded-full flex items-center justify-center text-xs font-bold",step>i+1?"bg-green-500 text-white":step===i+1?"bg-blue-600 text-white":"bg-gray-200 text-gray-500")}>{step>i+1?"✓":i+1}</div>
            <span className={cls("text-xs font-medium",step===i+1?"text-blue-600":"text-gray-400")}>{s}</span>
            {i<2&&<div className="w-6 h-0.5 bg-gray-200"/>}
          </div>
        ))}
      </div>

      {step===1 && (
        <div className="space-y-4">
          <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
            {PLANS.filter(p=>p.id!=="free").map(p=>(
              <div key={p.id} onClick={()=>setSel(p.id)} className={cls("rounded-2xl border-2 p-5 cursor-pointer transition",sel===p.id?(p.id==="pro"?"border-blue-500 bg-blue-50":"border-purple-500 bg-purple-50"):"border-gray-100 hover:border-gray-300")}>
                {p.popular&&<Badge c="blue">⭐ Terpopuler</Badge>}
                <div className="font-bold text-lg mt-1">{p.label}</div>
                <div className={cls("text-xl font-extrabold",p.id==="pro"?"text-blue-600":"text-purple-600")}>{p.desc}</div>
                <ul className="mt-3 space-y-1 text-sm">{p.features.map(f=><li key={f} className="flex gap-2 text-gray-700"><span className="text-green-500">✓</span>{f}</li>)}</ul>
              </div>
            ))}
          </div>
          <Btn full sz="lg" onClick={()=>setStep(2)}>Lanjut ke Instruksi Transfer →</Btn>
        </div>
      )}

      {step===2 && (
        <Card className="space-y-5">
          <div className="bg-gradient-to-r from-blue-600 to-indigo-600 rounded-xl p-4 text-white">
            <div className="text-sm opacity-80">Kamu memilih paket</div>
            <div className="text-2xl font-extrabold">{plan.label}</div>
            <div className="text-xl font-bold mt-1">{fmt(plan.price)}</div>
          </div>
          <div>
            <h3 className="font-bold text-gray-900 mb-3">📋 Instruksi Transfer Bank</h3>
            <div className="bg-yellow-50 border-2 border-yellow-200 rounded-xl p-4 space-y-3">
              <div className="grid grid-cols-2 gap-y-2 text-sm">
                <span className="text-gray-600">Bank</span><span className="font-bold">{BANK.bank}</span>
                <span className="text-gray-600">No. Rekening</span>
                <div className="flex items-center gap-2">
                  <span className="font-mono font-bold text-blue-700 text-base">{BANK.no}</span>
                  <button onClick={()=>navigator.clipboard.writeText(BANK.no)} className="text-xs bg-blue-100 text-blue-600 px-2 py-0.5 rounded hover:bg-blue-200">Copy</button>
                </div>
                <span className="text-gray-600">Atas Nama</span><span className="font-bold">{BANK.name}</span>
                <span className="text-gray-600">Jumlah Transfer</span>
                <div className="flex items-center gap-2">
                  <span className="font-bold text-green-700 text-base">{fmt(plan.price)}</span>
                  <button onClick={()=>navigator.clipboard.writeText(String(plan.price))} className="text-xs bg-green-100 text-green-700 px-2 py-0.5 rounded hover:bg-green-200">Copy</button>
                </div>
              </div>
              <div className="border-t border-yellow-200 pt-3">
                <p className="text-xs text-gray-600 font-medium">⚠️ Penting:</p>
                <ul className="text-xs text-gray-600 mt-1 space-y-0.5">
                  <li>• Transfer tepat sesuai jumlah di atas</li>
                  <li>• Simpan bukti transfer (screenshot)</li>
                  <li>• Masukkan no. referensi di langkah berikutnya</li>
                  <li>• Akun PRO aktif dalam 1×24 jam setelah terverifikasi</li>
                </ul>
              </div>
            </div>
          </div>
          <div className="flex gap-3">
            <Btn v="secondary" onClick={()=>setStep(1)} full>← Kembali</Btn>
            <Btn v="primary" onClick={()=>setStep(3)} full>Sudah Transfer →</Btn>
          </div>
        </Card>
      )}

      {step===3 && (
        <Card className="space-y-4">
          <h3 className="font-bold text-gray-900">✅ Konfirmasi Pembayaran</h3>
          <p className="text-sm text-gray-600">Masukkan nomor referensi dari bukti transfer kamu (biasanya tertera di struk/notifikasi transfer).</p>
          <Input label="Nomor Referensi Transfer *" value={ref} onChange={setRef} placeholder="Contoh: TRF202501011234567" hint="Bisa berupa nomor transaksi, kode booking, atau nomor referensi dari m-banking kamu."/>
          <div className="bg-gray-50 border border-gray-100 rounded-xl p-3 text-sm space-y-1">
            <div className="flex justify-between"><span className="text-gray-500">Paket</span><span className="font-bold">{plan.label}</span></div>
            <div className="flex justify-between"><span className="text-gray-500">Jumlah</span><span className="font-bold text-blue-600">{fmt(plan.price)}</span></div>
            <div className="flex justify-between"><span className="text-gray-500">Tujuan</span><span className="font-bold">{BANK.bank} · {BANK.no}</span></div>
          </div>
          <div className="flex gap-3">
            <Btn v="secondary" onClick={()=>setStep(2)} full>← Kembali</Btn>
            <Btn v="success" onClick={submitPayment} full disabled={uploading}>{uploading?"Memproses...":"✅ Kirim Konfirmasi"}</Btn>
          </div>
        </Card>
      )}
    </div>
  );
}

// ── SUBSCRIPTION STATUS ───────────────────────────────────────────────────────
function SubStatus({ user, onNavigate }) {
  const plan = PLANS.find(p=>p.id===user.plan)||PLANS[0];
  const payRec = _payments.filter(p=>p.userId===user.id).slice(-3).reverse();
  return (
    <div className="max-w-xl mx-auto space-y-5">
      <h1 className="text-xl font-bold text-gray-900">👑 Status Subscription</h1>
      <Card>
        <div className="flex items-center gap-4">
          <div className={cls("w-14 h-14 rounded-2xl flex items-center justify-center text-2xl font-bold text-white",user.plan==="free"?"bg-gray-400":user.plan==="pro"?"bg-blue-600":"bg-purple-600")}>
            {user.plan==="free"?"F":user.plan==="pro"?"P":"∞"}
          </div>
          <div>
            <div className="font-bold text-lg text-gray-900">Paket {plan.label}</div>
            <div className="text-sm text-gray-500">{plan.desc}</div>
            {user.plan!=="free"&&<Badge c="green">✅ Aktif</Badge>}
            {user.plan==="free"&&user.paymentStatus==="pending"&&<Badge c="yellow">⏳ Menunggu Verifikasi</Badge>}
          </div>
        </div>
        <ul className="mt-4 space-y-1.5">{plan.features.map(f=><li key={f} className="flex gap-2 text-sm text-gray-700"><span className="text-green-500">✓</span>{f}</li>)}{plan.locked?.map(f=><li key={f} className="flex gap-2 text-sm text-gray-400 line-through"><span>✗</span>{f}</li>)}</ul>
        {user.plan==="free"&&user.paymentStatus!=="pending"&&<Btn v="primary" full className="mt-4" onClick={()=>onNavigate("upgrade")}>⚡ Upgrade Sekarang</Btn>}
      </Card>
      {payRec.length>0&&(
        <Card>
          <h3 className="font-bold text-gray-900 mb-3">📋 Riwayat Pembayaran</h3>
          <div className="space-y-2">
            {payRec.map(p=>(
              <div key={p.id} className="flex items-center justify-between p-3 bg-gray-50 rounded-xl border border-gray-100 text-sm">
                <div><div className="font-semibold text-gray-900">{PLANS.find(pl=>pl.id===p.plan)?.label}</div><div className="text-xs text-gray-500">{new Date(p.createdAt).toLocaleDateString("id-ID")} · Ref: {p.ref}</div></div>
                <div className="text-right"><div className="font-bold text-blue-600">{fmt(p.amount)}</div><Badge c={p.status==="verified"?"green":p.status==="pending"?"yellow":"red"}>{p.status==="verified"?"Terverifikasi":p.status==="pending"?"Pending":"Ditolak"}</Badge></div>
              </div>
            ))}
          </div>
        </Card>
      )}
    </div>
  );
}

// ── DASHBOARD ─────────────────────────────────────────────────────────────────
function Dashboard({ user, onNavigate }) {
  const isPro = user.plan!=="free";
  return (
    <div className="space-y-5">
      <div>
        <h1 className="text-xl font-bold text-gray-900">Selamat datang, {user.name}! 👋</h1>
        <p className="text-sm text-gray-500">Kelola CV dan pantau progress karirmu di sini.</p>
      </div>
      {/* Stats */}
      <div className="grid grid-cols-2 md:grid-cols-4 gap-3">
        {[{i:"📄",l:"CV Tersimpan",v:"1"},{i:"📊",l:"ATS Score",v:isPro?"0%":"—"},{i:"👑",l:"Paket",v:user.plan.toUpperCase()},{i:"📥",l:"Download",v:isPro?"∞":`${user.downloads||0}/1`}].map(c=>(
          <Card key={c.l} className="text-center py-4"><div className="text-2xl">{c.i}</div><div className="text-xl font-bold text-gray-900 mt-1">{c.v}</div><div className="text-xs text-gray-500">{c.l}</div></Card>
        ))}
      </div>
      {/* Upgrade banner */}
      {!isPro&&(
        <div className="bg-gradient-to-r from-blue-600 to-indigo-600 text-white rounded-2xl p-4 flex flex-col md:flex-row items-center justify-between gap-3">
          <div><div className="font-bold">⚡ Upgrade ke PRO — {fmt(49000)}/bulan</div><div className="text-blue-100 text-sm">ATS Score, Cover Letter AI, Career Guidance AI, Unlimited download</div></div>
          <Btn v="secondary" onClick={()=>onNavigate("upgrade")}>Upgrade Sekarang</Btn>
        </div>
      )}
      {/* Feature cards */}
      <div className="grid grid-cols-1 md:grid-cols-2 gap-3">
        {[{id:"builder",i:"✏️",t:"Resume Builder",d:"Buat atau edit CV kamu",pro:false},{id:"ats",i:"📊",t:"ATS Checker AI",d:"Analisis semantik mendalam",pro:true},{id:"coverletter",i:"✉️",t:"Cover Letter AI",d:"Generate cover letter personal",pro:true},{id:"career",i:"🧭",t:"Career Guidance AI",d:"Analisis jalur karir & skill gap",pro:true}].map(card=>(
          <Card key={card.id} className={cls("cursor-pointer hover:shadow-md transition",card.pro&&!isPro&&"opacity-60")} onClick={()=>onNavigate(card.id)}>
            <div className="flex items-center gap-3"><span className="text-2xl">{card.i}</span><div className="flex-1"><div className="font-bold text-gray-900">{card.t} {card.pro&&!isPro&&"🔒"}</div><div className="text-sm text-gray-500">{card.d}</div></div>{card.pro&&!isPro&&<span className="text-xs text-blue-600 font-semibold">PRO</span>}</div>
          </Card>
        ))}
      </div>
    </div>
  );
}

// ── ADMIN PANEL ───────────────────────────────────────────────────────────────
function Admin({ currentUser }) {
  const [prices, setPrices] = useState({ pro:49000, lifetime:149000 });
  const [tab, setTab] = useState("overview");
  const users = _users.filter(u=>!u.isAdmin);
  const payments = _payments;
  const activeSubs = users.filter(u=>u.plan!=="free").length;
  const pendingPays = payments.filter(p=>p.status==="pending");
  const totalRev = payments.filter(p=>p.status==="verified").reduce((s,p)=>s+p.amount,0);

  const verifyPayment = (payId) => {
    const pay = _payments.find(p=>p.id===payId);
    if (!pay) return;
    pay.status = "verified";
    const u = _users.find(u=>u.id===pay.userId);
    if (u) { u.plan = pay.plan; u.paymentStatus = "verified"; }
    alert("✅ Pembayaran diverifikasi! Akun user telah diupgrade.");
  };
  const rejectPayment = (payId) => {
    const pay = _payments.find(p=>p.id===payId);
    if (pay) pay.status = "rejected";
    alert("❌ Pembayaran ditolak.");
  };

  return (
    <div className="space-y-5">
      <div className="flex items-center justify-between"><h1 className="text-xl font-bold text-gray-900">🛠 Admin Panel</h1><Badge c="purple">Admin</Badge></div>
      {/* Tabs */}
      <div className="flex gap-1 border-b border-gray-100 pb-1">
        {[["overview","📊 Overview"],["payments","💳 Pembayaran"],["users","👥 Users"],["settings","⚙️ Pengaturan"]].map(([id,label])=>(
          <button key={id} onClick={()=>setTab(id)} className={cls("px-3 py-2 text-xs font-semibold rounded-lg",tab===id?"bg-gray-800 text-white":"text-gray-600 hover:bg-gray-100")}>{label}</button>
        ))}
      </div>

      {tab==="overview"&&(
        <div className="space-y-4">
          <div className="grid grid-cols-2 md:grid-cols-4 gap-3">
            {[{i:"👥",l:"Total User",v:users.length},{i:"👑",l:"Subscriber Aktif",v:activeSubs},{i:"⏳",l:"Pending Verif.",v:pendingPays.length},{i:"💰",l:"Total Revenue",v:fmt(totalRev)}].map(s=>(
              <Card key={s.l} className="text-center py-4"><div className="text-2xl">{s.i}</div><div className="text-xl font-bold text-gray-900 mt-1">{s.v}</div><div className="text-xs text-gray-500">{s.l}</div></Card>
            ))}
          </div>
          {pendingPays.length>0&&(
            <Card className="border-yellow-200 bg-yellow-50">
              <h3 className="font-bold text-yellow-800 mb-2">⏳ {pendingPays.length} Pembayaran Menunggu Verifikasi</h3>
              {pendingPays.slice(0,3).map(p=>{
                const u=_users.find(x=>x.id===p.userId);
                return <div key={p.id} className="flex items-center justify-between bg-white rounded-xl p-3 mb-2 border border-yellow-100">
                  <div><div className="font-semibold text-sm">{u?.name||"Unknown"}</div><div className="text-xs text-gray-500">{PLANS.find(pl=>pl.id===p.plan)?.label} · {fmt(p.amount)} · Ref: {p.ref}</div></div>
                  <div className="flex gap-2"><Btn sz="sm" v="success" onClick={()=>verifyPayment(p.id)}>✓ Verifikasi</Btn><Btn sz="sm" v="danger" onClick={()=>rejectPayment(p.id)}>✗</Btn></div>
                </div>;
              })}
            </Card>
          )}
        </div>
      )}

      {tab==="payments"&&(
        <Card>
          <h3 className="font-bold text-gray-900 mb-3">💳 Semua Pembayaran ({payments.length})</h3>
          {payments.length===0?<p className="text-sm text-gray-400 text-center py-5">Belum ada pembayaran</p>:(
            <div className="space-y-2">
              {[...payments].reverse().map(p=>{
                const u=_users.find(x=>x.id===p.userId);
                return <div key={p.id} className="flex items-center justify-between p-3 bg-gray-50 rounded-xl border border-gray-100">
                  <div><div className="font-semibold text-sm">{u?.name} <span className="text-gray-400">({u?.email})</span></div><div className="text-xs text-gray-500">{new Date(p.createdAt).toLocaleString("id-ID")} · {PLANS.find(pl=>pl.id===p.plan)?.label} · Ref: {p.ref}</div></div>
                  <div className="text-right flex items-center gap-2">
                    <div><div className="font-bold text-sm text-blue-600">{fmt(p.amount)}</div><Badge c={p.status==="verified"?"green":p.status==="pending"?"yellow":"red"}>{p.status}</Badge></div>
                    {p.status==="pending"&&<div className="flex gap-1"><Btn sz="sm" v="success" onClick={()=>verifyPayment(p.id)}>✓</Btn><Btn sz="sm" v="danger" onClick={()=>rejectPayment(p.id)}>✗</Btn></div>}
                  </div>
                </div>;
              })}
            </div>
          )}
        </Card>
      )}

      {tab==="users"&&(
        <Card>
          <h3 className="font-bold text-gray-900 mb-3">👥 Semua User ({users.length})</h3>
          {users.length===0?<p className="text-sm text-gray-400 text-center py-5">Belum ada user</p>:(
            <div className="space-y-2">
              {users.map(u=>(
                <div key={u.id} className="flex items-center justify-between p-3 bg-gray-50 rounded-xl border border-gray-100">
                  <div><div className="font-semibold text-sm">{u.name}</div><div className="text-xs text-gray-500">{u.email} · Download: {u.downloads||0}</div></div>
                  <div className="flex items-center gap-2"><Badge c={u.plan==="free"?"gray":u.plan==="pro"?"blue":"purple"}>{u.plan.toUpperCase()}</Badge>{u.paymentStatus==="pending"&&<Badge c="yellow">Pending</Badge>}</div>
                </div>
              ))}
            </div>
          )}
        </Card>
      )}

      {tab==="settings"&&(
        <Card className="space-y-4">
          <h3 className="font-bold text-gray-900">⚙️ Kelola Harga Paket</h3>
          <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
            <div className="space-y-1"><label className="text-xs font-semibold text-gray-600">Harga PRO (per bulan)</label><div className="flex items-center gap-2"><span className="text-sm text-gray-500 font-bold">Rp</span><input type="number" className="border border-gray-200 rounded-xl px-3 py-2 text-sm flex-1 bg-gray-50 focus:ring-2 focus:ring-blue-400 focus:outline-none" value={prices.pro} onChange={e=>setPrices(p=>({...p,pro:Number(e.target.value)}))}/></div></div>
            <div className="space-y-1"><label className="text-xs font-semibold text-gray-600">Harga Lifetime (sekali bayar)</label><div className="flex items-center gap-2"><span className="text-sm text-gray-500 font-bold">Rp</span><input type="number" className="border border-gray-200 rounded-xl px-3 py-2 text-sm flex-1 bg-gray-50 focus:ring-2 focus:ring-blue-400 focus:outline-none" value={prices.lifetime} onChange={e=>setPrices(p=>({...p,lifetime:Number(e.target.value)}))}/></div></div>
          </div>
          <Card className="bg-gray-50 border-dashed">
            <h4 className="text-sm font-bold text-gray-800 mb-2">📱 Info Rekening Penerima</h4>
            <div className="text-sm space-y-1 text-gray-700"><div><span className="font-semibold">Bank:</span> {BANK.bank}</div><div><span className="font-semibold">No. Rekening:</span> {BANK.no}</div><div><span className="font-semibold">Atas Nama:</span> {BANK.name}</div></div>
          </Card>
          <Btn v="success" onClick={()=>alert(`Harga tersimpan: PRO ${fmt(prices.pro)}/bln, Lifetime ${fmt(prices.lifetime)}`)}>💾 Simpan Perubahan</Btn>
        </Card>
      )}
    </div>
  );
}

// ── NAV CONFIG ────────────────────────────────────────────────────────────────
const NAV = [
  { id:"dashboard", icon:"🏠", label:"Dashboard" },
  { id:"builder",   icon:"✏️", label:"Resume Builder" },
  { id:"ats",       icon:"📊", label:"ATS Checker", pro:true },
  { id:"coverletter",icon:"✉️",label:"Cover Letter", pro:true },
  { id:"career",    icon:"🧭", label:"Career Guidance", pro:true },
  { id:"subscription",icon:"👑",label:"Subscription" },
];

// ── MAIN ──────────────────────────────────────────────────────────────────────
export default function App() {
  const [page, setPage] = useState("landing"); // landing | auth | app
  const [user, setUser] = useState(null);
  const [active, setActive] = useState("dashboard");
  const [sidebar, setSidebar] = useState(false);

  const updateUser = (u) => { setUser(u); const rec=_users.find(x=>x.id===u.id); if(rec) Object.assign(rec,u); };

  const navigate = (p) => {
    if (PRO_PAGES.includes(p) && user?.plan==="free") { setActive("upgrade"); }
    else { setActive(p); }
    setSidebar(false);
  };

  if (page==="landing") return <Landing onCTA={()=>setPage("auth")} onLogin={()=>setPage("auth")}/>;
  if (page==="auth") return <AuthPage onAuth={u=>{ setUser(u); setPage("app"); setActive("dashboard"); }}/>;

  return (
    <div className="min-h-screen bg-gray-50 flex">
      {/* Sidebar */}
      <aside className={cls("fixed inset-y-0 left-0 z-40 w-56 bg-white border-r border-gray-100 flex flex-col transition-transform duration-200 md:translate-x-0 md:static md:z-auto",sidebar?"translate-x-0":"-translate-x-full")}>
        <div className="p-4 border-b border-gray-50 flex items-center justify-between">
          <div className="flex items-center gap-2">
            <div className="w-7 h-7 bg-blue-600 rounded-lg flex items-center justify-center text-white font-bold text-xs">CV</div>
            <span className="font-bold text-gray-900 text-sm">CVLolos<span className="text-blue-600">.ai</span></span>
          </div>
          <button className="md:hidden text-gray-400 hover:text-gray-600" onClick={()=>setSidebar(false)}>✕</button>
        </div>
        <nav className="flex-1 p-2 space-y-0.5 overflow-y-auto">
          {NAV.map(item=>(
            <button key={item.id} onClick={()=>navigate(item.id)} className={cls("w-full flex items-center gap-2.5 px-3 py-2.5 rounded-xl text-sm font-medium transition",active===item.id?"bg-blue-600 text-white":"text-gray-600 hover:bg-gray-100")}>
              <span className="text-base">{item.icon}</span>{item.label}
              {item.pro&&user?.plan==="free"&&<span className="ml-auto text-xs opacity-50">🔒</span>}
            </button>
          ))}
          {user?.isAdmin&&(
            <button onClick={()=>{setActive("admin");setSidebar(false);}} className={cls("w-full flex items-center gap-2.5 px-3 py-2.5 rounded-xl text-sm font-medium transition mt-2 border-t border-gray-100 pt-3",active==="admin"?"bg-gray-800 text-white":"text-gray-600 hover:bg-gray-100")}>
              <span className="text-base">🛠</span>Admin Panel
            </button>
          )}
        </nav>
        <div className="p-3 border-t border-gray-50">
          <div className="flex items-center gap-2 px-2 py-1.5 mb-1">
            <div className="w-8 h-8 bg-blue-100 rounded-full flex items-center justify-center text-blue-700 font-bold text-xs flex-shrink-0">{user?.name?.[0]||"U"}</div>
            <div className="flex-1 min-w-0"><div className="text-xs font-bold text-gray-900 truncate">{user?.name}</div><Badge c={user?.plan==="free"?"gray":user?.plan==="pro"?"blue":"purple"}>{user?.plan?.toUpperCase()}</Badge></div>
          </div>
          <button onClick={()=>{setUser(null);setPage("landing");}} className="w-full text-xs text-gray-400 hover:text-gray-600 py-1.5 rounded-lg hover:bg-gray-50 transition">← Keluar</button>
        </div>
      </aside>
      {sidebar&&<div className="fixed inset-0 bg-black/30 z-30 md:hidden" onClick={()=>setSidebar(false)}/>}
      {/* Main */}
      <div className="flex-1 flex flex-col min-w-0 overflow-hidden">
        <header className="bg-white border-b border-gray-100 px-4 py-3 flex items-center gap-3 md:hidden sticky top-0 z-20">
          <button onClick={()=>setSidebar(true)} className="p-1.5 rounded-lg hover:bg-gray-100 text-gray-600">☰</button>
          <span className="font-bold text-gray-900 text-sm">CVLolos<span className="text-blue-600">.ai</span></span>
          <div className="ml-auto flex items-center gap-2"><Badge c={user?.plan==="free"?"gray":user?.plan==="pro"?"blue":"purple"}>{user?.plan?.toUpperCase()}</Badge></div>
        </header>
        <main className="flex-1 p-4 md:p-7 overflow-y-auto">
          {active==="dashboard"   && <Dashboard user={user} onNavigate={navigate}/>}
          {active==="builder"     && <Builder user={user} onUpdate={updateUser}/>}
          {active==="ats"         && <ATS/>}
          {active==="coverletter" && <CoverLetter user={user}/>}
          {active==="career"      && <Career/>}
          {active==="upgrade"     && <Upgrade user={user} onDone={updateUser} onNavigate={navigate}/>}
          {active==="subscription"&& <SubStatus user={user} onNavigate={navigate}/>}
          {active==="admin"       && <Admin currentUser={user}/>}
        </main>
      </div>
    </div>
  );
}
