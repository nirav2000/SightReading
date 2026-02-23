import { useState, useEffect, useCallback, useRef } from "react";

// ── Storage ───────────────────────────────────────────────────────────────────
const SK = { DECKS:"fc_d8", PROG:"fc_p8", PARTIAL:"fc_part8", TOPICS:"fc_t8" };
async function sl(k) { try { const r = await window.storage.get(k); return r ? JSON.parse(r.value) : null; } catch { return null; } }
async function ss(k,v) { try { await window.storage.set(k, JSON.stringify(v)); } catch(e) { console.error(e); } }

// ── SM-2 ──────────────────────────────────────────────────────────────────────
function sm2(card, q) {
  const quality = q===2?5:q===1?3:1;
  let {ef=2.5, interval=1, repetitions=0} = card;
  if (quality>=3) { interval = repetitions===0?1:repetitions===1?6:Math.round(interval*ef); repetitions++; }
  else { repetitions=0; interval=1; }
  ef = Math.max(1.3, ef+0.1-(5-quality)*(0.08+(5-quality)*0.02));
  return {ef, interval, repetitions, nextReview: Date.now()+interval*86400000};
}

// ── JSON parser with truncation recovery ─────────────────────────────────────
function parseJSON(raw) {
  if (!raw) throw new Error("Empty input");
  let text = "";
  for (let i=0; i<raw.length; i++) {
    const code = raw.charCodeAt(i);
    if (code===9||code===10||code===13||(code>=32&&code<=127)||(code>=160)) text += raw[i];
  }
  text = text.replace(/^```[a-z]*\s*/i,"").replace(/```\s*$/,"").trim();
  // Direct parse
  try { return JSON.parse(text); } catch(_) {}
  // Find outermost [ ] or { }
  for (const [o,c] of [["[","]"],["{","}"]]) {
    const s=text.indexOf(o), e=text.lastIndexOf(c);
    if (s!==-1&&e>s) { try { return JSON.parse(text.slice(s,e+1)); } catch(_) {} }
  }
  // Truncated array — close at last complete object
  const arrStart = text.indexOf("[");
  if (arrStart !== -1) {
    const truncated = text.slice(arrStart);
    const lastComma = truncated.lastIndexOf("},");
    const closeAt = lastComma !== -1 ? lastComma+1 : truncated.lastIndexOf("}");
    if (closeAt > 0) { try { return JSON.parse(truncated.slice(0,closeAt+1)+"]"); } catch(_) {} }
  }
  // Salvage individual objects
  const objs=[]; let depth=0,start=-1,inStr=false,esc=false;
  for (let i=0;i<text.length;i++) {
    const c=text[i];
    if(esc){esc=false;continue} if(c==="\\"&&inStr){esc=true;continue}
    if(c==='"'){inStr=!inStr;continue} if(inStr)continue;
    if(c==="{"){if(depth===0)start=i;depth++;}
    else if(c==="}"){depth--;if(depth===0&&start!==-1){try{objs.push(JSON.parse(text.slice(start,i+1)));}catch(_){}start=-1;}}
  }
  if (objs.length) return objs;
  throw new Error("Cannot parse JSON: "+text.slice(0,150));
}

// ── Read response body reliably ───────────────────────────────────────────────
async function readBody(res) {
  try { const t = await res.text(); if (t&&t.trim()) return t; } catch(_) {}
  try { const buf = await res.arrayBuffer(); if (buf&&buf.byteLength>0) return new TextDecoder("utf-8").decode(buf); } catch(_) {}
  try {
    if (res.body&&res.body.getReader) {
      const reader=res.body.getReader(), chunks=[];
      while(true){const{done,value}=await reader.read();if(done)break;if(value)chunks.push(value);}
      const merged=new Uint8Array(chunks.reduce((a,c)=>a+c.length,0));
      let off=0; chunks.forEach(c=>{merged.set(c,off);off+=c.length;});
      const t=new TextDecoder("utf-8").decode(merged);
      if(t&&t.trim()) return t;
    }
  } catch(_) {}
  return "";
}

// ── Extract text content from Anthropic envelope ──────────────────────────────
function extractText(raw) {
  let text="", searchFrom=0;
  while(true) {
    const keyPos=raw.indexOf('"text"',searchFrom); if(keyPos===-1) break;
    const colon=raw.indexOf(":",keyPos); if(colon===-1) break;
    let valStart=raw.indexOf('"',colon+1); if(valStart===-1) break;
    let j=valStart+1, valEnd=-1;
    while(j<raw.length){
      if(raw[j]==='\\'){j+=2;continue;}
      if(raw[j]==='"'){valEnd=j;break;}
      j++;
    }
    if(valEnd===-1){
      text+=raw.slice(valStart+1).replace(/\\n/g,"\n").replace(/\\"/g,'"').replace(/\\\\/g,"\\").replace(/\\t/g,"\t");
      break;
    }
    text+=raw.slice(valStart+1,valEnd).replace(/\\n/g,"\n").replace(/\\"/g,'"').replace(/\\\\/g,"\\").replace(/\\t/g,"\t");
    searchFrom=valEnd+1;
  }
  return text;
}

function delay(ms) { return new Promise(r=>setTimeout(r,ms)); }

// ── Single Claude call ────────────────────────────────────────────────────────
async function askClaude(prompt, maxTokens, onLog) {
  if (!maxTokens) maxTokens=1000;
  const maxAttempts=3;
  for(let attempt=0;attempt<maxAttempts;attempt++){
    if(attempt>0){
      const wait=Math.pow(2,attempt)*2000;
      if(onLog) onLog("warn",`Retry ${attempt}/${maxAttempts-1} — waiting ${wait/1000}s...`);
      await delay(wait);
    }
    try {
      const res=await fetch("https://api.anthropic.com/v1/messages",{
        method:"POST",headers:{"Content-Type":"application/json"},
        body:JSON.stringify({model:"claude-sonnet-4-20250514",max_tokens:maxTokens,messages:[{role:"user",content:prompt}]})
      });
      const raw=await readBody(res);
      if(!raw||!raw.trim()){
        if(attempt<maxAttempts-1) continue;
        throw new Error("Empty response after "+maxAttempts+" attempts (Status="+res.status+")");
      }
      if(!res.ok) throw new Error("API "+res.status+": "+raw.slice(0,150));
      const text=extractText(raw);
      if(!text.trim()) throw new Error("No text content in response. Raw preview: "+raw.slice(0,200));
      return parseJSON(text);
    } catch(e){
      if(attempt<maxAttempts-1&&(e.message.includes("Empty response")||e.message.includes("fetch"))) continue;
      throw e;
    }
  }
}

// ── Helpers ───────────────────────────────────────────────────────────────────
function normTopic(s){ return s.toLowerCase().replace(/[^a-z0-9 ]/g,"").replace(/\s+/g," ").trim(); }
function findMatch(input, topics) {
  const n=normTopic(input);
  for(const [canonical,deckId] of Object.entries(topics)){
    const cn=normTopic(canonical);
    if(cn===n) return {canonical,deckId,exact:true};
    const iw=new Set(n.split(" ")),tw=cn.split(" ");
    if(tw.filter(w=>iw.has(w)).length/Math.max(tw.length,1)>=0.6) return {canonical,deckId,exact:false};
  }
  return null;
}
function buildQueue(deck, dp) {
  const now=Date.now(),due=[],fresh=[];
  deck.cards.forEach((_,i)=>{ const cp=dp[i]; if(!cp)fresh.push(i); else if(cp.nextReview<=now)due.push({i,ov:now-cp.nextReview}); });
  due.sort((a,b)=>b.ov-a.ov);
  return [...due.map(x=>x.i),...fresh];
}

// ── UI ────────────────────────────────────────────────────────────────────────
const BG="#6A9BCC";
function Btn({children,onClick,color="#1f2937",textColor="#fff",style,disabled}){
  return <button disabled={disabled} onClick={onClick} style={{background:color,color:textColor,border:"none",borderRadius:999,padding:".6rem 1.2rem",cursor:disabled?"not-allowed":"pointer",fontWeight:600,fontSize:".9rem",opacity:disabled?.5:1,...style}}>{children}</button>;
}
function Page({children,style}){ return <div style={{minHeight:"100vh",background:BG,display:"flex",flexDirection:"column",alignItems:"center",justifyContent:"center",padding:"1.5rem 0",...style}}>{children}</div>; }
function Spinner(){ return <><style>{`@keyframes spin{to{transform:rotate(360deg)}}`}</style><div style={{width:48,height:48,border:"4px solid rgba(255,255,255,.3)",borderTopColor:"#fff",borderRadius:"50%",animation:"spin .8s linear infinite"}}/></>; }
function Card({children,style}){ return <div style={{background:"#fff",borderRadius:20,padding:"1.1rem 1.25rem",boxShadow:"0 2px 8px rgba(0,0,0,.08)",...style}}>{children}</div>; }
function Toggle({label,value,onChange}){
  return <label style={{display:"flex",alignItems:"center",gap:".5rem",cursor:"pointer",userSelect:"none"}}>
    <div onClick={onChange} style={{width:40,height:22,borderRadius:999,background:value?"#4A6FA5":"#d1d5db",position:"relative",transition:"background .2s",flexShrink:0}}>
      <div style={{position:"absolute",top:3,left:value?20:3,width:16,height:16,borderRadius:"50%",background:"#fff",transition:"left .2s"}}/>
    </div>
    <span style={{fontSize:".85rem",color:"#374151"}}>{label}</span>
  </label>;
}

// ── App ───────────────────────────────────────────────────────────────────────
export default function App() {
  const [ready,setReady]         = useState(false);
  const [screen,setScreen]       = useState("home");
  const [decks,setDecks]         = useState({});
  const [prog,setProg]           = useState({});
  const [topics,setTopics]       = useState({});
  const [partial,setPartial]     = useState(null);

  const [activeDeck,setActiveDeck]   = useState(null);
  const [queue,setQueue]             = useState([]);
  const [qIdx,setQIdx]               = useState(0);
  const [flipped,setFlipped]         = useState(false);
  const [flipTime,setFlipTime]       = useState(null);
  const [cardStart,setCardStart]     = useState(null);
  const [session,setSession]         = useState({easy:0,hard:0,wrong:0});
  const [animating,setAnimating]     = useState(false);
  const [usePagination,setUsePagination] = useState(false);
  const PAGE_SIZE = 20;

  const [topic,setTopic]             = useState("capitals of the world");
  const [tab,setTab]                 = useState("describe");
  const [loadMsg,setLoadMsg]         = useState("");
  const [loadPct,setLoadPct]         = useState(0);
  const [apiLog,setApiLog]           = useState([]);
  const [createError,setCreateError] = useState("");
  const [showLog,setShowLog]         = useState(true);
  const [copied,setCopied]           = useState(false);
  const logRef                       = useRef(null);

  // ── Boot: load storage and check for partial ──────────────────────────────
  useEffect(()=>{
    Promise.all([sl(SK.DECKS),sl(SK.PROG),sl(SK.TOPICS),sl(SK.PARTIAL)]).then(([d,p,t,part])=>{
      setDecks(d||{});
      setProg(p||{});
      setTopics(t||{});
      // Only restore partial if it has actual saved cards
      if(part&&part.cards&&part.cards.length>0){
        setPartial(part);
        // Pre-fill the topic input so user can see what was in progress
        setTopic(part.input||"");
      } else {
        setPartial(null);
      }
      setReady(true);
    });
  },[]);

  const saveAll = useCallback(async(d,p,t)=>{
    await Promise.all([ss(SK.DECKS,d),ss(SK.PROG,p),ss(SK.TOPICS,t)]);
  },[]);

  function addLog(level,msg,detail=""){
    const entry={time:new Date().toLocaleTimeString(),level,msg,detail};
    console.log(`[${level}] ${msg}`,detail);
    setApiLog(prev=>{
      const next=[...prev,entry];
      setTimeout(()=>{ if(logRef.current) logRef.current.scrollTop=logRef.current.scrollHeight; },50);
      return next;
    });
  }

  function copyLog(){
    const lines=apiLog.map(e=>`[${e.time}] [${e.level.toUpperCase()}] ${e.msg}${e.detail?" — "+e.detail:""}`).join("\n");
    navigator.clipboard.writeText(lines).catch(()=>{});
    setCopied(true); setTimeout(()=>setCopied(false),2000);
  }

  useEffect(()=>{
    const handler=e=>{
      if(screen!=="study")return;
      if(!flipped){if(e.key==="ArrowUp"||e.key==="ArrowDown"){e.preventDefault();doFlip();}return;}
      if(e.key==="1")doGrade(0); else if(e.key==="2")doGrade(1); else if(e.key==="3")doGrade(2);
    };
    window.addEventListener("keydown",handler);
    return ()=>window.removeEventListener("keydown",handler);
  },[screen,flipped,qIdx,queue]);

  function doFlip(){ if(!flipped){setFlipTime(Date.now());setFlipped(true);} }

  async function doGrade(quality){
    const idx=queue[qIdx];
    const ttf=flipTime?(flipTime-cardStart)/1000:null;
    const existing=(prog[activeDeck]||{})[idx]||{};
    const updated={...sm2(existing,quality),history:[...(existing.history||[]),{date:Date.now(),quality,timeToFlip:ttf}]};
    const newProg={...prog,[activeDeck]:{...(prog[activeDeck]||{}),[idx]:updated}};
    setProg(newProg);
    await saveAll(decks,newProg,topics);
    setSession(s=>({easy:s.easy+(quality===2?1:0),hard:s.hard+(quality===1?1:0),wrong:s.wrong+(quality===0?1:0)}));
    if(qIdx+1>=queue.length){setScreen("done");return;}
    setAnimating(true);
    setTimeout(()=>{setFlipped(false);setFlipTime(null);setCardStart(Date.now());setQIdx(i=>i+1);setTimeout(()=>setAnimating(false),50);},150);
  }

  function doStartStudy(deckId,freshDecks,freshProg){
    const d=freshDecks||decks,p=freshProg||prog;
    const q=buildQueue(d[deckId],p[deckId]||{});
    setActiveDeck(deckId);setQueue(q);setQIdx(0);
    setFlipped(false);setFlipTime(null);setCardStart(Date.now());
    setSession({easy:0,hard:0,wrong:0});
    setScreen("study");
  }

  async function doGenerate(){
    const input=topic.trim(); if(!input)return;

    // Check if we already have a completed deck for this topic
    const match=findMatch(input,topics);
    if(match&&decks[match.deckId]){
      doStartStudy(match.deckId);
      return;
    }

    // Check if we have partial saved cards for this exact topic — use them
    let savedCards=[];
    if(partial&&partial.input&&normTopic(partial.input)===normTopic(input)&&partial.cards&&partial.cards.length>0){
      savedCards=partial.cards;
      addLog("info",`Found ${savedCards.length} previously saved cards for "${input}" — will merge with new results`);
    }

    setCreateError("");setApiLog([]);setShowLog(true);setScreen("loading");

    const onLog=(level,msg,detail)=>addLog(level,msg,detail||"");

    try {
      setLoadMsg("Generating flashcards..."); setLoadPct(10);
      addLog("info","Sending single API call to generate all cards...");

      const prompt = tab==="paste"
        ? `Extract ALL possible flashcard pairs from this text. Be comprehensive.\nReply ONLY with a JSON array, no markdown:\n[{"front":"...","back":"...","meta":{"funFact":"..."}}]\n\nText:\n${input}`
        : `Create a COMPLETE and EXHAUSTIVE flashcard deck about: "${input}".\nInclude EVERY relevant item — do not skip any.\nFor geography: front=country name, back=capital city, meta includes continent, flag emoji, funFact.\nFor other topics: adapt front/back and meta fields appropriately.\nReply ONLY with a valid JSON array, no markdown, no explanation:\n[{"front":"...","back":"...","meta":{"continent":"...","flag":"🏳","funFact":"one interesting fact"}}]`;

      setLoadPct(15);
      const cards = await askClaude(prompt, 16000, onLog);
      if(!Array.isArray(cards)||!cards.length) throw new Error("No cards returned");
      const validCards=cards.filter(c=>c&&c.front&&c.back);
      addLog("ok","Cards received",validCards.length+" valid cards");
      setLoadPct(85);

      // Merge with any previously saved partial cards
      const merged=[...savedCards,...validCards];
      const seen=new Set();
      const deduped=merged.filter(c=>{ const k=(c.front||"").toLowerCase().trim(); if(!k||seen.has(k))return false;seen.add(k);return true; });
      addLog("ok","After dedup",deduped.length+" cards total");

      setLoadMsg(`Saving ${deduped.length} cards...`); setLoadPct(92);
      const deckId="deck_"+Date.now();
      const canonical=input.trim();
      const newDecks={...decks,[deckId]:{topic:canonical,cards:deduped,createdAt:Date.now()}};
      const newTopics={...topics,[canonical]:deckId};
      setDecks(newDecks);setTopics(newTopics);
      await Promise.all([saveAll(newDecks,prog,newTopics),ss(SK.PARTIAL,null)]);
      setPartial(null);setLoadPct(100);
      addLog("ok","Saved! Starting study...");
      doStartStudy(deckId,newDecks,prog);

    } catch(err){
      console.error(err);
      addLog("error",err.message,err.stack||"");
      // Save whatever partial cards we have so user can retry and we merge
      if(savedCards.length>0||apiLog.some(l=>l.level==="ok")){
        addLog("info","Cards from this attempt will be merged on retry");
      }
      setCreateError(err.message);
      setScreen("create");
    }
  }

  async function doDelete(deckId){
    const d={...decks};delete d[deckId];
    const p={...prog};delete p[deckId];
    const t=Object.fromEntries(Object.entries(topics).filter(([,v])=>v!==deckId));
    setDecks(d);setProg(p);setTopics(t);
    await saveAll(d,p,t);
  }

  if(!ready) return <Page><Spinner/></Page>;

  // ── HOME ──────────────────────────────────────────────────────────────────
  if(screen==="home"){
    const entries=Object.entries(decks);
    return (
      <Page>
        <div style={{width:"100%",maxWidth:560,padding:"0 1rem"}}>
          <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:"1.5rem"}}>
            <h1 style={{color:"#fff",fontSize:"1.8rem",fontWeight:700}}>My Flashcards</h1>
            <Btn color="rgba(255,255,255,.2)" onClick={()=>setScreen("stats")}>📊 Stats</Btn>
          </div>
          {partial&&partial.cards&&partial.cards.length>0&&(
            <div style={{background:"#fef3c7",border:"1px solid #fcd34d",borderRadius:16,padding:"1rem",marginBottom:"1rem",display:"flex",alignItems:"center",gap:".75rem"}}>
              <div style={{flex:1}}>
                <p style={{fontWeight:600,color:"#92400e",fontSize:".9rem"}}>⚠ Partial save: "{partial.input}"</p>
                <small style={{color:"#b45309",fontSize:".75rem"}}>{partial.cards.length} cards saved — generate again to complete and merge</small>
              </div>
              <Btn color="#f59e0b" onClick={()=>{setTopic(partial.input||"");setScreen("create");}}>Continue</Btn>
            </div>
          )}
          {!entries.length
            ?<div style={{background:"rgba(255,255,255,.2)",borderRadius:20,padding:"2rem",textAlign:"center",color:"rgba(255,255,255,.8)",marginBottom:"1rem"}}>No decks yet. Create your first one!</div>
            :<div style={{display:"flex",flexDirection:"column",gap:".75rem",marginBottom:"1rem"}}>
              {entries.map(([id,deck])=>{
                const dp=prog[id]||{};
                const due=deck.cards.filter((_,i)=>{const cp=dp[i];return !cp||cp.nextReview<=Date.now();}).length;
                return (
                  <Card key={id} style={{display:"flex",alignItems:"center",gap:".75rem"}}>
                    <div style={{flex:1,minWidth:0}}>
                      <p style={{fontWeight:600,color:"#1f2937",textTransform:"capitalize",overflow:"hidden",textOverflow:"ellipsis",whiteSpace:"nowrap"}}>{deck.topic}</p>
                      <small style={{color:"#9ca3af"}}>{deck.cards.length} cards · {Object.keys(dp).length} studied · <span style={{color:"#3b82f6",fontWeight:600}}>{due} due</span></small>
                    </div>
                    <Btn color={due>0?"#4A6FA5":"#9ca3af"} onClick={()=>doStartStudy(id)}>{due>0?`Study (${due})`:"Review"}</Btn>
                    <button style={{background:"none",border:"none",cursor:"pointer",color:"#d1d5db",fontSize:"1.1rem"}} onClick={()=>doDelete(id)}>🗑</button>
                  </Card>
                );
              })}
            </div>
          }
          <Btn color="#1f2937" onClick={()=>setScreen("create")} style={{width:"100%",padding:"1rem",fontSize:"1.05rem"}}>+ New deck</Btn>
        </div>
      </Page>
    );
  }

  // ── CREATE ────────────────────────────────────────────────────────────────
  if(screen==="create"){
    const match=findMatch(topic,topics);
    const hasPartialForTopic=partial&&partial.input&&normTopic(partial.input)===normTopic(topic)&&partial.cards&&partial.cards.length>0;
    return (
      <Page>
        <div style={{width:"100%",maxWidth:520,padding:"0 1rem"}}>
          <button style={{background:"none",border:"none",color:"rgba(255,255,255,.7)",cursor:"pointer",marginBottom:"1.5rem",fontSize:".9rem"}} onClick={()=>setScreen("home")}>← Back</button>
          <h1 style={{color:"#fff",fontSize:"2rem",fontWeight:700,textAlign:"center",marginBottom:"1.5rem"}}>Create flashcards</h1>

          <div style={{display:"flex",justifyContent:"center",marginBottom:"1.25rem"}}>
            <div style={{background:"rgba(255,255,255,.2)",padding:3,borderRadius:999,display:"inline-flex"}}>
              {["describe","paste"].map(t=>(
                <button key={t} onClick={()=>setTab(t)} style={{background:tab===t?"#fff":"transparent",color:tab===t?"#374151":"#fff",border:"none",borderRadius:999,padding:".45rem 1.25rem",cursor:"pointer",fontWeight:600,fontSize:".9rem"}}>{t==="describe"?"Describe topic":"Paste text"}</button>
              ))}
            </div>
          </div>

          <div style={{background:"#fff",borderRadius:24,padding:"1.5rem",boxShadow:"0 4px 16px rgba(0,0,0,.1)"}}>
            <textarea value={topic} onChange={e=>setTopic(e.target.value)}
              placeholder={tab==="describe"?"e.g. capitals of the world\ne.g. elements of the periodic table":"Paste text here..."}
              style={{width:"100%",height:110,border:"none",outline:"none",resize:"none",fontSize:"1.05rem",lineHeight:1.6,color:"#111827",fontFamily:"inherit"}}/>

            <div style={{borderTop:"1px solid #f3f4f6",marginTop:".75rem",paddingTop:".75rem"}}>
              <Toggle label="Paginate cards during study (20 at a time)" value={usePagination} onChange={()=>setUsePagination(v=>!v)}/>
            </div>

            {match&&decks[match.deckId]&&(
              <div style={{marginTop:".75rem",paddingTop:".75rem",borderTop:"1px solid #f3f4f6",display:"flex",alignItems:"center",gap:".75rem"}}>
                <div style={{flex:1}}>
                  <p style={{fontSize:".85rem",fontWeight:600,color:"#374151"}}>{match.exact?"✅ Deck exists:":"🔍 Similar deck:"} <em style={{textTransform:"capitalize"}}>{match.canonical}</em></p>
                  <small style={{color:"#9ca3af"}}>{decks[match.deckId].cards.length} cards · {Object.keys(prog[match.deckId]||{}).length} studied</small>
                </div>
                <Btn color="#3b82f6" onClick={()=>doStartStudy(match.deckId)}>Use existing</Btn>
              </div>
            )}

            {hasPartialForTopic&&(
              <div style={{marginTop:".75rem",paddingTop:".75rem",borderTop:"1px solid #f3f4f6"}}>
                <p style={{fontSize:".82rem",color:"#f59e0b",fontWeight:600}}>⚠ {partial.cards.length} partially saved cards found — generating again will merge results</p>
              </div>
            )}

            {Object.keys(topics).length>0&&(
              <div style={{marginTop:".75rem",paddingTop:".75rem",borderTop:"1px solid #f3f4f6"}}>
                <small style={{color:"#9ca3af",display:"block",marginBottom:".4rem"}}>Saved topics:</small>
                <div style={{display:"flex",flexWrap:"wrap",gap:".4rem"}}>
                  {Object.keys(topics).map(t=>(
                    <button key={t} onClick={()=>setTopic(t)} style={{background:"#f3f4f6",color:"#4b5563",border:"none",borderRadius:999,padding:".3rem .75rem",fontSize:".8rem",cursor:"pointer",textTransform:"capitalize"}}>{t}</button>
                  ))}
                </div>
              </div>
            )}
          </div>

          {createError&&(
            <div style={{marginTop:".75rem"}}>
              <div onClick={copyLog} style={{background:"rgba(239,68,68,.2)",border:"1px solid rgba(239,68,68,.3)",borderRadius:12,padding:".9rem",color:"#fff",fontSize:".85rem",display:"flex",alignItems:"flex-start",gap:".5rem",cursor:"pointer"}}>
                <span style={{flex:1}}>{createError}</span>
                <span style={{fontSize:".72rem",opacity:.6,flexShrink:0,whiteSpace:"nowrap"}}>{copied?"✅ Copied!":"📋 click to copy log"}</span>
                <button onClick={e=>{e.stopPropagation();setCreateError("");setShowLog(false);}} style={{background:"none",border:"none",color:"rgba(255,255,255,.6)",cursor:"pointer",fontSize:"1.1rem",lineHeight:1,flexShrink:0}}>✕</button>
              </div>
              {apiLog.length>0&&showLog&&(
                <div style={{background:"rgba(0,0,0,.3)",borderRadius:12,marginTop:".4rem",overflow:"hidden"}}>
                  <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",padding:".5rem .75rem",borderBottom:"1px solid rgba(255,255,255,.1)"}}>
                    <span style={{color:"rgba(255,255,255,.5)",fontSize:".75rem",fontFamily:"monospace"}}>API Log</span>
                    <div style={{display:"flex",gap:".5rem"}}>
                      <button onClick={copyLog} style={{background:"rgba(255,255,255,.15)",border:"none",borderRadius:6,color:"#fff",fontSize:".72rem",padding:".2rem .5rem",cursor:"pointer"}}>{copied?"✅ Copied":"📋 Copy"}</button>
                      <button onClick={()=>setShowLog(false)} style={{background:"none",border:"none",color:"rgba(255,255,255,.5)",cursor:"pointer",fontSize:".85rem"}}>Hide</button>
                    </div>
                  </div>
                  <div ref={logRef} style={{maxHeight:160,overflowY:"auto",padding:".6rem .75rem",fontFamily:"monospace",fontSize:".7rem"}}>
                    {apiLog.map((e,i)=>(
                      <div key={i} style={{marginBottom:".2rem",color:e.level==="error"?"#fca5a5":e.level==="ok"?"#86efac":e.level==="warn"?"#fde68a":"#e2e8f0"}}>
                        <span style={{opacity:.5}}>{e.time} </span>
                        <span style={{fontWeight:700}}>[{e.level.toUpperCase()}] </span>
                        {e.msg}{e.detail&&<span style={{opacity:.6}}> — {e.detail}</span>}
                      </div>
                    ))}
                  </div>
                </div>
              )}
              {apiLog.length>0&&!showLog&&(
                <button onClick={()=>setShowLog(true)} style={{background:"rgba(0,0,0,.2)",border:"none",borderRadius:8,color:"rgba(255,255,255,.6)",fontSize:".75rem",padding:".3rem .7rem",cursor:"pointer",marginTop:".3rem"}}>Show log</button>
              )}
            </div>
          )}

          <Btn color="#1f2937" onClick={doGenerate} style={{width:"100%",padding:"1rem",fontSize:"1rem",marginTop:"1.25rem"}}>Generate flashcards</Btn>
        </div>
      </Page>
    );
  }

  // ── LOADING ───────────────────────────────────────────────────────────────
  if(screen==="loading") return (
    <Page>
      <div style={{textAlign:"center",maxWidth:500,padding:"2rem",width:"100%"}}>
        <Spinner/>
        <h2 style={{color:"#fff",fontSize:"1.4rem",marginBottom:".5rem",marginTop:"1.5rem"}}>Building your deck…</h2>
        <p style={{color:"rgba(255,255,255,.7)",marginBottom:"1.5rem",minHeight:"1.4em",fontSize:".9rem"}}>{loadMsg}</p>
        <div style={{background:"rgba(255,255,255,.2)",borderRadius:999,height:10,overflow:"hidden"}}>
          <div style={{background:"#fff",height:"100%",borderRadius:999,width:`${loadPct}%`,transition:"width .5s"}}/>
        </div>
        <p style={{color:"rgba(255,255,255,.5)",fontSize:".75rem",marginTop:".4rem",marginBottom:"1rem"}}>{loadPct}%</p>
        {apiLog.length>0&&(
          <div style={{background:"rgba(0,0,0,.25)",borderRadius:12,textAlign:"left",overflow:"hidden"}}>
            <div style={{display:"flex",justifyContent:"space-between",padding:".4rem .75rem",borderBottom:"1px solid rgba(255,255,255,.08)"}}>
              <span style={{color:"rgba(255,255,255,.4)",fontSize:".72rem",fontFamily:"monospace"}}>Live log</span>
              <button onClick={copyLog} style={{background:"rgba(255,255,255,.1)",border:"none",borderRadius:6,color:"#fff",fontSize:".68rem",padding:".15rem .4rem",cursor:"pointer"}}>{copied?"✅ Copied":"📋 Copy"}</button>
            </div>
            <div ref={logRef} style={{maxHeight:180,overflowY:"auto",padding:".6rem .75rem",fontFamily:"monospace",fontSize:".7rem"}}>
              {apiLog.map((e,i)=>(
                <div key={i} style={{marginBottom:".2rem",color:e.level==="error"?"#fca5a5":e.level==="ok"?"#86efac":e.level==="warn"?"#fde68a":"#e2e8f0"}}>
                  <span style={{opacity:.5}}>{e.time} </span>
                  <span style={{fontWeight:700}}>[{e.level.toUpperCase()}] </span>
                  {e.msg}{e.detail&&<span style={{opacity:.6}}> — {e.detail}</span>}
                </div>
              ))}
            </div>
          </div>
        )}
      </div>
    </Page>
  );

  // ── STUDY ─────────────────────────────────────────────────────────────────
  if(screen==="study"){
    const deck=decks[activeDeck]; if(!deck)return null;
    const card=deck.cards[queue[qIdx]]; if(!card)return null;
    const dp=(prog[activeDeck]||{})[queue[qIdx]];
    const meta=card.meta||{};
    const metaTags=Object.entries(meta).filter(([k])=>k!=="funFact"&&k!=="flag");
    const totalPages=usePagination?Math.ceil(queue.length/PAGE_SIZE):1;
    const currentPage=usePagination?Math.floor(qIdx/PAGE_SIZE):0;
    return (
      <Page style={{justifyContent:"center",paddingBottom:"1rem"}}>
        <div style={{width:"100%",maxWidth:580,padding:"0 1.5rem"}}>
          <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:".5rem"}}>
            <button style={{background:"none",border:"none",color:"rgba(255,255,255,.7)",cursor:"pointer"}} onClick={()=>setScreen("home")}>← Home</button>
            <span style={{color:"rgba(255,255,255,.7)",fontSize:".85rem",textTransform:"capitalize",overflow:"hidden",textOverflow:"ellipsis",maxWidth:200}}>{deck.topic}</span>
            <span style={{color:"rgba(255,255,255,.7)",fontSize:".85rem"}}>{usePagination?`p${currentPage+1}/${totalPages} · `:""}{qIdx+1}/{queue.length}</span>
          </div>
          {usePagination&&totalPages>1&&(
            <div style={{display:"flex",gap:".3rem",marginBottom:".5rem",flexWrap:"wrap",justifyContent:"center"}}>
              {Array.from({length:totalPages},(_,i)=>(
                <button key={i} onClick={()=>{setQIdx(i*PAGE_SIZE);setFlipped(false);setFlipTime(null);setCardStart(Date.now());}}
                  style={{padding:".2rem .6rem",borderRadius:999,border:"none",fontSize:".75rem",cursor:"pointer",fontWeight:600,background:i===currentPage?"#fff":"rgba(255,255,255,.3)",color:i===currentPage?"#4A6FA5":"#fff"}}>
                  {i+1}
                </button>
              ))}
            </div>
          )}
          <div style={{background:"rgba(255,255,255,.2)",borderRadius:999,height:6,marginBottom:"1.25rem",overflow:"hidden"}}>
            <div style={{background:"#fff",height:"100%",borderRadius:999,width:`${(qIdx/queue.length)*100}%`,transition:"width .3s"}}/>
          </div>
          <div style={{perspective:1000,height:390}}>
            <div onClick={!flipped?doFlip:undefined} style={{position:"relative",width:"100%",height:"100%",transformStyle:"preserve-3d",transition:animating?"opacity .15s":"transform .5s",transform:flipped?"rotateX(180deg)":"rotateX(0deg)",opacity:animating?0:1,cursor:flipped?"default":"pointer"}}>
              <div style={{position:"absolute",inset:0,background:"#fff",borderRadius:24,boxShadow:"0 4px 20px rgba(0,0,0,.12)",backfaceVisibility:"hidden",WebkitBackfaceVisibility:"hidden",display:"flex",flexDirection:"column",alignItems:"center",justifyContent:"center",padding:"2rem"}}>
                <span style={{position:"absolute",top:12,right:12,fontSize:".7rem",padding:".25rem .65rem",borderRadius:999,fontWeight:600,background:dp?"#fef3c7":"#dbeafe",color:dp?"#b45309":"#1d4ed8"}}>{dp?`Every ${dp.interval}d`:"New"}</span>
                {meta.flag&&<div style={{fontSize:"3.5rem",marginBottom:".75rem"}}>{meta.flag}</div>}
                <div style={{fontSize:"2rem",fontWeight:700,color:"#1f2937",textAlign:"center"}}>{card.front}</div>
                <div style={{fontSize:".8rem",color:"#9ca3af",marginTop:"auto"}}>Click or ↑↓ to reveal</div>
              </div>
              <div style={{position:"absolute",inset:0,background:"#fff",borderRadius:24,boxShadow:"0 4px 20px rgba(0,0,0,.12)",backfaceVisibility:"hidden",WebkitBackfaceVisibility:"hidden",transform:"rotateX(180deg)",display:"flex",flexDirection:"column",alignItems:"center",justifyContent:"center",padding:"1.5rem"}}>
                {meta.flag&&<div style={{fontSize:"2.5rem",marginBottom:".25rem"}}>{meta.flag}</div>}
                <div style={{fontSize:".7rem",color:"#9ca3af",letterSpacing:".08em",textTransform:"uppercase",marginBottom:".25rem"}}>Answer</div>
                <div style={{fontSize:"1.9rem",fontWeight:700,color:"#1f2937",textAlign:"center",marginBottom:".75rem"}}>{card.back}</div>
                {metaTags.length>0&&<div style={{display:"flex",flexWrap:"wrap",gap:".4rem",justifyContent:"center",marginBottom:".5rem"}}>{metaTags.map(([k,v])=><span key={k} style={{background:"#eef4ff",color:"#4A6FA5",borderRadius:999,padding:".25rem .75rem",fontSize:".8rem",fontWeight:500}}>{String(v)}</span>)}</div>}
                {meta.funFact&&<div style={{background:"#fffbeb",borderRadius:12,padding:".75rem 1rem",fontSize:".82rem",color:"#6b7280",lineHeight:1.5,marginTop:"auto",width:"100%"}}><strong style={{color:"#d97706"}}>💡 </strong>{meta.funFact}</div>}
              </div>
            </div>
          </div>
          {flipped&&(
            <div style={{display:"flex",gap:".6rem",marginTop:"1rem"}}>
              {[{q:0,l:"Wrong",s:"key 1",bg:"#fee2e2",tc:"#dc2626",i:"✕"},{q:1,l:"Hard",s:"key 2",bg:"#fef3c7",tc:"#d97706",i:"😅"},{q:2,l:"Easy",s:"key 3",bg:"#d1fae5",tc:"#059669",i:"✓"}].map(({q,l,s,bg,tc,i})=>(
                <button key={q} onClick={()=>doGrade(q)} style={{flex:1,background:bg,color:tc,border:"none",borderRadius:16,padding:".75rem .5rem",cursor:"pointer",fontWeight:600,fontSize:".85rem",display:"flex",flexDirection:"column",alignItems:"center",gap:".15rem"}}>
                  <span style={{fontSize:"1.1rem"}}>{i}</span><span>{l}</span><small style={{fontSize:".7rem",opacity:.5}}>{s}</small>
                </button>
              ))}
            </div>
          )}
        </div>
      </Page>
    );
  }

  // ── DONE ──────────────────────────────────────────────────────────────────
  if(screen==="done"){
    const total=session.easy+session.hard+session.wrong;
    const pct=total>0?Math.round(session.easy/total*100):0;
    return (
      <Page>
        <div style={{background:"#fff",borderRadius:24,padding:"2.5rem",maxWidth:400,width:"calc(100% - 3rem)",textAlign:"center",boxShadow:"0 8px 32px rgba(0,0,0,.12)"}}>
          <div style={{fontSize:"3.5rem",marginBottom:".75rem"}}>{pct>=80?"🎉":pct>=50?"💪":"📚"}</div>
          <h2 style={{fontSize:"1.6rem",fontWeight:700,color:"#1f2937",marginBottom:".4rem"}}>Session complete!</h2>
          <p style={{color:"#6b7280",marginBottom:"1.5rem"}}>{total} cards reviewed</p>
          <div style={{display:"flex",gap:".75rem",marginBottom:"2rem"}}>
            {[{v:session.easy,l:"Easy",bg:"#d1fae5",tc:"#059669"},{v:session.hard,l:"Hard",bg:"#fef3c7",tc:"#d97706"},{v:session.wrong,l:"Wrong",bg:"#fee2e2",tc:"#dc2626"}].map(({v,l,bg,tc})=>(
              <div key={l} style={{flex:1,background:bg,borderRadius:16,padding:".9rem .5rem",textAlign:"center"}}>
                <b style={{fontSize:"1.6rem",fontWeight:700,display:"block",color:tc}}>{v}</b>
                <span style={{fontSize:".8rem",color:tc}}>{l}</span>
              </div>
            ))}
          </div>
          <Btn color="#1f2937" onClick={()=>setScreen("home")} style={{width:"100%",padding:"1rem"}}>Back to home</Btn>
        </div>
      </Page>
    );
  }

  // ── STATS ─────────────────────────────────────────────────────────────────
  if(screen==="stats"){
    const entries=Object.entries(decks);
    const allH=entries.flatMap(([id])=>Object.values(prog[id]||{}).flatMap(cp=>cp.history||[]));
    const tot=allH.length,easy=allH.filter(h=>h.quality===2).length,hard=allH.filter(h=>h.quality===1).length,wrong=allH.filter(h=>h.quality===0).length;
    const ft=allH.filter(h=>h.timeToFlip);const avgFlip=ft.length?(ft.reduce((s,h)=>s+h.timeToFlip,0)/ft.length).toFixed(1)+"s":"—";
    return (
      <Page style={{justifyContent:"flex-start",paddingTop:"2rem"}}>
        <div style={{width:"100%",maxWidth:560,padding:"0 1rem"}}>
          <div style={{display:"flex",alignItems:"center",gap:"1rem",marginBottom:"1.5rem"}}>
            <button style={{background:"none",border:"none",color:"rgba(255,255,255,.7)",cursor:"pointer"}} onClick={()=>setScreen("home")}>← Back</button>
            <h1 style={{color:"#fff",fontSize:"1.8rem",fontWeight:700}}>Statistics</h1>
          </div>
          <div style={{display:"grid",gridTemplateColumns:"repeat(3,1fr)",gap:".6rem",marginBottom:"1.5rem"}}>
            {[{l:"Reviews",v:tot,bg:"#eef4ff",tc:"#4A6FA5"},{l:"Avg flip",v:avgFlip,bg:"#f3f4f6",tc:"#374151"},{l:"Decks",v:entries.length,bg:"#eef4ff",tc:"#4A6FA5"},
              {l:"✅ Easy",v:easy,bg:"#d1fae5",tc:"#059669"},{l:"😅 Hard",v:hard,bg:"#fef3c7",tc:"#d97706"},{l:"❌ Wrong",v:wrong,bg:"#fee2e2",tc:"#dc2626"}
            ].map(({l,v,bg,tc})=>(
              <div key={l} style={{background:bg,borderRadius:16,padding:".9rem",textAlign:"center"}}>
                <b style={{fontSize:"1.5rem",fontWeight:700,display:"block",color:tc}}>{v}</b>
                <small style={{fontSize:".72rem",color:tc,opacity:.7}}>{l}</small>
              </div>
            ))}
          </div>
          <div style={{display:"flex",flexDirection:"column",gap:".6rem"}}>
            {entries.map(([id,deck])=>{
              const h=Object.values(prog[id]||{}).flatMap(cp=>cp.history||[]);
              const e=h.filter(x=>x.quality===2).length,hd=h.filter(x=>x.quality===1).length,w=h.filter(x=>x.quality===0).length,t=h.length;
              const mastered=Object.values(prog[id]||{}).filter(cp=>cp.repetitions>=3).length;
              const pct=t>0?Math.round(e/t*100):0;
              return (
                <Card key={id}>
                  <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:".6rem"}}>
                    <p style={{fontWeight:600,color:"#1f2937",textTransform:"capitalize"}}>{deck.topic}</p>
                    <span style={{fontSize:".75rem",fontWeight:600,padding:".2rem .6rem",borderRadius:999,background:pct>=70?"#d1fae5":"#fef3c7",color:pct>=70?"#059669":"#d97706"}}>{pct}% easy</span>
                  </div>
                  <div style={{display:"flex",height:8,borderRadius:999,overflow:"hidden",marginBottom:".5rem",gap:1}}>
                    {t>0?<><div style={{width:`${e/t*100}%`,background:"#10b981"}}/><div style={{width:`${hd/t*100}%`,background:"#f59e0b"}}/><div style={{width:`${w/t*100}%`,background:"#ef4444"}}/></>:<div style={{flex:1,background:"#e5e7eb"}}/>}
                  </div>
                  <div style={{display:"flex",gap:".75rem",fontSize:".75rem",color:"#6b7280",flexWrap:"wrap"}}>
                    <span>📊 {t} reviews</span><span>🏆 {mastered}/{deck.cards.length} mastered</span><span>✅{e} 😅{hd} ❌{w}</span>
                  </div>
                </Card>
              );
            })}
          </div>
        </div>
      </Page>
    );
  }

  return null;
}
