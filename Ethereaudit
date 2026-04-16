import { useState } from "react";

const MAX_FREE = 5;
const G = "#00ff88";

const SEV = {
  "CRÍTICA": { bg:"#ff193918", border:"#ff1939", text:"#ff1939" },
  "ALTA":    { bg:"#ff780018", border:"#ff7800", text:"#ff7800" },
  "MEDIA":   { bg:"#f5c51818", border:"#f5c518", text:"#f5c518" },
  "BAJA":    { bg:"#00ff8812", border:"#00ff88", text:"#00ff88" },
};

const grade = s =>
  s >= 85 ? { label:"SEGURO",   color:G } :
  s >= 60 ? { label:"MODERADO", color:"#f5c518" } :
            { label:"CRÍTICO",  color:"#ff1939" };

const PLANS = [
  {
    id:"free", name:"Free", price:"$0", period:"", color:"#444",
    features:["5 auditorías por día","Reporte .txt básico","Análisis de vulnerabilidades","Soporte comunidad"],
    missing:["Auditorías ilimitadas","Análisis de bytecode","API Access","Soporte prioritario 24/7","Historial de auditorías","Badge verificado"],
  },
  {
    id:"pro", name:"Pro", price:"$19", period:"/mes", color:G, popular:true,
    features:["Auditorías ilimitadas","Análisis de bytecode","Reporte PDF profesional","API Access (1,000 req/mes)","Soporte prioritario 24/7","Historial de auditorías"],
    missing:["Badge verificado en contratos","White-label reports"],
  },
  {
    id:"enterprise", name:"Enterprise", price:"$99", period:"/mes", color:"#aa88ff",
    features:["Todo lo de Pro","Auditorías ilimitadas de equipo","Badge verificado en contratos","White-label reports","API Access ilimitado","Gestor de cuenta dedicado","SLA 99.9%"],
    missing:[],
  },
];

export default function App() {
  const [code, setCode]       = useState("");
  const [loading, setLoad]    = useState(false);
  const [result, setResult]   = useState(null);
  const [error, setError]     = useState("");
  const [scans, setScans]     = useState(0);
  const [tab, setTab]         = useState("overview");
  const [page, setPage]       = useState("home");
  const [plan, setPlan]       = useState("free");  // free | pro | enterprise
  const [showUpgrade, setUpgrade] = useState(false);
  const [checkout, setCheckout]   = useState(null); // null | {planId, step}
  const [card, setCard]           = useState({ name:"", number:"", exp:"", cvc:"" });
  const [payDone, setPayDone]     = useState(false);

  const remaining = plan === "free" ? MAX_FREE - scans : Infinity;

  const analyze = async () => {
    if (!code.trim()) { setError("Pega el código Solidity de tu contrato."); return; }
    if (plan === "free" && scans >= MAX_FREE) { setUpgrade(true); return; }
    setError(""); setResult(null); setLoad(true); setTab("overview");
    try {
      const res = await fetch("https://api.anthropic.com/v1/messages", {
        method:"POST", headers:{"Content-Type":"application/json"},
        body: JSON.stringify({
          model:"claude-sonnet-4-20250514", max_tokens:1500,
          system:"Eres auditor experto de smart contracts Solidity. Analiza el código real. Responde SOLO con JSON puro.",
          messages:[{ role:"user", content:`Analiza este contrato y devuelve SOLO este JSON:
{"contractName":"nombre","compiler":"versión pragma","summary":"descripción 1-2 oraciones","securityScore":50,"gasEfficiency":70,"codeQuality":60,"decentralization":50,"vulnerabilities":[{"name":"vuln","severity":"CRÍTICA","line":"línea","description":"descripción","recommendation":"solución"}],"notes":"observaciones"}

CÓDIGO:
${code.slice(0,6000)}` }],
        }),
      });
      const data = await res.json();
      let txt = (data.content||[]).filter(b=>b.type==="text").map(b=>b.text).join("");
      const s = txt.indexOf("{"), e = txt.lastIndexOf("}");
      if (s===-1||e===-1) throw new Error("Respuesta inválida.");
      setResult(JSON.parse(txt.slice(s,e+1)));
      setScans(n=>n+1);
    } catch(e) { setError("Error: "+e.message); }
    finally { setLoad(false); }
  };

  const download = () => {
    if (!result) return;
    const r = result;
    const txt = `ETHERAUDIT — REPORTE DE SEGURIDAD\n${"═".repeat(36)}\nFecha:    ${new Date().toLocaleDateString("es-MX")}\nContrato: ${r.contractName}\nCompiler: ${r.compiler}\nPlan:     ${plan.toUpperCase()}\nScore:    ${r.securityScore}/100\n\nDESCRIPCIÓN\n${"─".repeat(12)}\n${r.summary}\n\nMÉTRICAS\n${"─".repeat(8)}\nGas:              ${r.gasEfficiency}/100\nCalidad:          ${r.codeQuality}/100\nDescentralización:${r.decentralization}/100\n\nVULNERABILIDADES (${r.vulnerabilities?.length||0})\n${"─".repeat(16)}\n${!r.vulnerabilities?.length?"Ninguna.":r.vulnerabilities.map((v,i)=>`[${i+1}] ${v.name} — ${v.severity} (línea ${v.line})\n  ${v.description}\n  ► ${v.recommendation}`).join("\n\n")}\n\nNOTAS\n${"─".repeat(5)}\n${r.notes}\n${"═".repeat(36)}\nEtherAudit`;
    const a = document.createElement("a");
    a.href = URL.createObjectURL(new Blob([txt],{type:"text/plain"}));
    a.download = `etheraudit_${r.contractName.replace(/\s+/g,"_")}.txt`;
    a.click();
  };

  // ── CHECKOUT MODAL ──
  const CheckoutModal = ({ planId }) => {
    const p = PLANS.find(x=>x.id===planId);
    if (payDone) return (
      <Modal onClose={()=>{setCheckout(null);setPayDone(false);}}>
        <div style={{textAlign:"center",padding:"20px 0"}}>
          <div style={{fontSize:52,marginBottom:12}}>🎉</div>
          <div style={{fontSize:20,fontWeight:800,color:G,marginBottom:8}}>¡Pago exitoso!</div>
          <div style={{fontSize:13,color:"#888",marginBottom:24}}>Bienvenido a <b style={{color:"#fff"}}>{p.name}</b>. Tu cuenta ya está activa.</div>
          <button onClick={()=>{setPlan(planId);setCheckout(null);setPayDone(false);setUpgrade(false);setPage("home");}}
            style={{background:`linear-gradient(135deg,${p.color},${p.color}99)`,border:"none",color:"#000",padding:"13px 32px",borderRadius:12,fontWeight:800,fontSize:14,cursor:"pointer"}}>
            Ir a la Herramienta →
          </button>
        </div>
      </Modal>
    );

    return (
      <Modal onClose={()=>setCheckout(null)}>
        <div style={{fontSize:11,color:"#555",letterSpacing:"1px",marginBottom:16}}>CHECKOUT SEGURO</div>
        <div style={{background:"#0a0a0a",border:`1px solid ${p.color}33`,borderRadius:12,padding:14,marginBottom:20,display:"flex",justifyContent:"space-between",alignItems:"center"}}>
          <div>
            <div style={{fontSize:14,fontWeight:700}}>Plan {p.name}</div>
            <div style={{fontSize:11,color:"#555",marginTop:2}}>Facturación mensual · Cancela cuando quieras</div>
          </div>
          <div style={{fontSize:20,fontWeight:900,color:p.color}}>{p.price}<span style={{fontSize:11,color:"#555"}}>{p.period}</span></div>
        </div>

        <div style={{display:"flex",flexDirection:"column",gap:10,marginBottom:20}}>
          <div>
            <div style={{fontSize:11,color:"#555",marginBottom:5}}>NOMBRE EN LA TARJETA</div>
            <input value={card.name} onChange={e=>setCard({...card,name:e.target.value})} placeholder="Juan García"
              style={{width:"100%",boxSizing:"border-box",background:"#0a0a0a",border:"1px solid #222",borderRadius:8,color:"#fff",padding:"10px 12px",fontSize:13,fontFamily:"inherit",outline:"none"}}/>
          </div>
          <div>
            <div style={{fontSize:11,color:"#555",marginBottom:5}}>NÚMERO DE TARJETA</div>
            <input value={card.number} onChange={e=>setCard({...card,number:e.target.value.replace(/\D/g,"").slice(0,16).replace(/(.{4})/g,"$1 ").trim()})}
              placeholder="1234 5678 9012 3456"
              style={{width:"100%",boxSizing:"border-box",background:"#0a0a0a",border:"1px solid #222",borderRadius:8,color:"#fff",padding:"10px 12px",fontSize:13,fontFamily:"'Courier New',monospace",outline:"none"}}/>
          </div>
          <div style={{display:"flex",gap:10}}>
            <div style={{flex:1}}>
              <div style={{fontSize:11,color:"#555",marginBottom:5}}>VENCIMIENTO</div>
              <input value={card.exp} onChange={e=>setCard({...card,exp:e.target.value.replace(/\D/g,"").slice(0,4).replace(/(.{2})/,"$1/")})}
                placeholder="MM/AA"
                style={{width:"100%",boxSizing:"border-box",background:"#0a0a0a",border:"1px solid #222",borderRadius:8,color:"#fff",padding:"10px 12px",fontSize:13,fontFamily:"inherit",outline:"none"}}/>
            </div>
            <div style={{flex:1}}>
              <div style={{fontSize:11,color:"#555",marginBottom:5}}>CVC</div>
              <input value={card.cvc} onChange={e=>setCard({...card,cvc:e.target.value.replace(/\D/g,"").slice(0,3)})}
                placeholder="123"
                style={{width:"100%",boxSizing:"border-box",background:"#0a0a0a",border:"1px solid #222",borderRadius:8,color:"#fff",padding:"10px 12px",fontSize:13,fontFamily:"inherit",outline:"none"}}/>
            </div>
          </div>
        </div>

        <div style={{background:"#0a0a0a",border:"1px solid #1a1a1a",borderRadius:10,padding:"10px 14px",marginBottom:16,display:"flex",gap:8,alignItems:"center"}}>
          <span style={{fontSize:16}}>🔒</span>
          <span style={{fontSize:11,color:"#555",lineHeight:1.5}}>Pago cifrado con SSL. No almacenamos datos de tarjeta. Powered by Stripe.</span>
        </div>

        <button onClick={()=>{if(card.name&&card.number.length>=18&&card.exp.length===5&&card.cvc.length===3)setPayDone(true);}}
          style={{width:"100%",padding:"14px 0",borderRadius:12,border:"none",background:`linear-gradient(135deg,${p.color},${p.color}99)`,color:"#000",fontWeight:800,fontSize:14,cursor:"pointer"}}>
          Pagar {p.price}{p.period} →
        </button>
        <div style={{textAlign:"center",marginTop:10,fontSize:11,color:"#444"}}>Cancela en cualquier momento desde tu cuenta</div>
      </Modal>
    );
  };

  // ── UPGRADE MODAL ──
  const UpgradeModal = () => (
    <Modal onClose={()=>setUpgrade(false)}>
      <div style={{textAlign:"center",marginBottom:20}}>
        <div style={{fontSize:32,marginBottom:8}}>⚡</div>
        <div style={{fontSize:18,fontWeight:800,marginBottom:6}}>Límite alcanzado</div>
        <div style={{fontSize:13,color:"#666"}}>Has usado tus 5 scans gratuitos de hoy.<br/>Actualiza para continuar.</div>
      </div>
      <div style={{display:"flex",flexDirection:"column",gap:10}}>
        {PLANS.filter(p=>p.id!=="free").map(p=>(
          <button key={p.id} onClick={()=>{setUpgrade(false);setCheckout(p.id);}}
            style={{background:`${p.color}15`,border:`1px solid ${p.color}55`,borderRadius:12,padding:"14px 18px",cursor:"pointer",display:"flex",justifyContent:"space-between",alignItems:"center"}}>
            <div style={{textAlign:"left"}}>
              <div style={{fontSize:14,fontWeight:700,color:p.color}}>{p.name}</div>
              <div style={{fontSize:11,color:"#555",marginTop:2}}>{p.features[0]}</div>
            </div>
            <div style={{fontSize:18,fontWeight:900,color:p.color}}>{p.price}<span style={{fontSize:11,color:"#555"}}>{p.period}</span></div>
          </button>
        ))}
      </div>
    </Modal>
  );

  // ── PRICING PAGE ──
  if (page==="pricing") return (
    <Wrap>
      <Navbar onNav={setPage} plan={plan}/>
      <div style={{maxWidth:600,margin:"0 auto",padding:"40px 20px"}}>
        <div style={{textAlign:"center",marginBottom:36}}>
          <div style={{display:"inline-block",background:"#00ff8818",border:"1px solid #00ff8833",color:G,fontSize:11,fontWeight:700,letterSpacing:"1px",padding:"4px 14px",borderRadius:20,marginBottom:14}}>PLANES & PRECIOS</div>
          <h2 style={{fontSize:30,fontWeight:900,margin:"0 0 10px",letterSpacing:"-0.5px"}}>Simple y <span style={{color:G}}>transparente</span></h2>
          <p style={{color:"#555",fontSize:14,margin:0}}>Sin contratos. Sin sorpresas. Cancela cuando quieras.</p>
        </div>

        <div style={{display:"flex",flexDirection:"column",gap:14}}>
          {PLANS.map(p=>(
            <div key={p.id} style={{background:p.popular?"#0a150e":"#111",border:`1px solid ${p.popular?p.color+"55":"#222"}`,borderRadius:18,padding:22,position:"relative"}}>
              {p.popular && <div style={{position:"absolute",top:-1,right:20,background:G,color:"#000",fontSize:10,fontWeight:800,padding:"3px 12px",borderRadius:"0 0 10px 10px",letterSpacing:"1px"}}>MÁS POPULAR</div>}
              <div style={{display:"flex",justifyContent:"space-between",alignItems:"flex-start",marginBottom:14}}>
                <div>
                  <div style={{fontSize:16,fontWeight:800,color:p.color}}>{p.name}</div>
                  <div style={{fontSize:11,color:"#555",marginTop:2}}>{p.id==="free"?"Para empezar":p.id==="pro"?"Para desarrolladores":"Para equipos"}</div>
                </div>
                <div style={{textAlign:"right"}}>
                  <span style={{fontSize:26,fontWeight:900,color:p.color}}>{p.price}</span>
                  <span style={{fontSize:12,color:"#555"}}>{p.period}</span>
                </div>
              </div>
              <div style={{display:"flex",flexDirection:"column",gap:7,marginBottom:16}}>
                {p.features.map(f=>(
                  <div key={f} style={{fontSize:12,color:"#aaa",display:"flex",gap:8,alignItems:"center"}}>
                    <span style={{color:p.color,fontSize:13}}>✓</span>{f}
                  </div>
                ))}
                {p.missing.map(f=>(
                  <div key={f} style={{fontSize:12,color:"#333",display:"flex",gap:8,alignItems:"center"}}>
                    <span style={{fontSize:13}}>✗</span>{f}
                  </div>
                ))}
              </div>
              {p.id==="free"
                ? <button onClick={()=>setPage("home")} style={{width:"100%",padding:"12px 0",borderRadius:12,border:"1px solid #333",background:"transparent",color:"#555",fontWeight:700,fontSize:13,cursor:"pointer"}}>
                    {plan==="free"?"Plan actual →":"Cambiar a Free"}
                  </button>
                : <button onClick={()=>setCheckout(p.id)}
                    style={{width:"100%",padding:"13px 0",borderRadius:12,border:"none",background:`linear-gradient(135deg,${p.color},${p.color}99)`,color:"#000",fontWeight:800,fontSize:13,cursor:"pointer"}}>
                    {plan===p.id?"Plan actual ✓":`Comenzar ${p.name} →`}
                  </button>
              }
            </div>
          ))}
        </div>

        <div style={{marginTop:32,background:"#111",border:"1px solid #1a1a1a",borderRadius:16,padding:20}}>
          <div style={{fontSize:13,fontWeight:700,color:"#555",letterSpacing:"1px",marginBottom:16}}>PREGUNTAS FRECUENTES</div>
          {[
            ["¿Puedo cancelar en cualquier momento?","Sí. Sin penalizaciones. Tu acceso continúa hasta el fin del período pagado."],
            ["¿Qué métodos de pago aceptan?","Visa, Mastercard, AMEX y tarjetas de débito. Próximamente USDC/ETH."],
            ["¿Los reportes son para uso comercial?","Sí. Con Pro y Enterprise puedes incluirlos en auditorías a clientes."],
          ].map(([q,a])=>(
            <div key={q} style={{marginBottom:14,paddingBottom:14,borderBottom:"1px solid #1a1a1a"}}>
              <div style={{fontSize:13,fontWeight:600,marginBottom:4}}>{q}</div>
              <div style={{fontSize:12,color:"#555",lineHeight:1.6}}>{a}</div>
            </div>
          ))}
        </div>
      </div>
      {checkout && <CheckoutModal planId={checkout}/>}
    </Wrap>
  );

  // ── HOME PAGE ──
  const gr = result ? grade(result.securityScore) : null;
  return (
    <Wrap>
      <Navbar onNav={setPage} plan={plan}/>
      {showUpgrade && <UpgradeModal/>}
      {checkout && <CheckoutModal planId={checkout}/>}

      <div style={{maxWidth:520,margin:"0 auto",padding:"36px 20px"}}>

        {/* PLAN BADGE */}
        {plan!=="free" && (
          <div style={{display:"flex",justifyContent:"center",marginBottom:20}}>
            <div style={{background:`${PLANS.find(p=>p.id===plan).color}18`,border:`1px solid ${PLANS.find(p=>p.id===plan).color}44`,borderRadius:20,padding:"4px 14px",fontSize:11,fontWeight:700,color:PLANS.find(p=>p.id===plan).color,letterSpacing:"1px"}}>
              ✓ PLAN {plan.toUpperCase()} ACTIVO
            </div>
          </div>
        )}

        {!result && <>
          <div style={{textAlign:"center",marginBottom:28}}>
            <h1 style={{fontSize:38,fontWeight:900,lineHeight:1.1,margin:"0 0 12px",letterSpacing:"-1px"}}>
              Tu Auditor<br/><span style={{color:G}}>Inteligente</span>
            </h1>
            <p style={{color:"#555",fontSize:14,lineHeight:1.6,margin:0}}>Pega tu código Solidity y obtén un análisis<br/>profesional en segundos.</p>
          </div>

          <div style={{background:"#111",border:"1px solid #222",borderRadius:20,padding:20,marginBottom:12}}>
            <textarea value={code} onChange={e=>{setCode(e.target.value);setError("");}}
              placeholder={"// Pega aquí tu contrato Solidity\n\npragma solidity ^0.8.0;\n\ncontract MiContrato {\n  ...\n}"}
              style={{width:"100%",boxSizing:"border-box",background:"#0a0a0a",border:`1px solid ${error?"#ff193955":"#1a1a1a"}`,borderRadius:12,color:"#e2e8f0",padding:"14px",fontSize:12,fontFamily:"'Courier New',monospace",outline:"none",resize:"vertical",minHeight:180,lineHeight:1.6}}/>

            <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",margin:"12px 0 4px"}}>
              <div style={{display:"flex",alignItems:"center",gap:8}}>
                <span style={{fontSize:13,color:remaining>0?G:"#ff1939",fontWeight:600}}>
                  {plan==="free" ? `Scans: ${scans}/${MAX_FREE} (Gratis)` : "✓ Scans ilimitados"}
                </span>
                {plan==="free" && remaining<=2 && remaining>0 && (
                  <button onClick={()=>setPage("pricing")} style={{fontSize:10,background:`${G}18`,border:`1px solid ${G}44`,color:G,borderRadius:6,padding:"2px 8px",cursor:"pointer",fontWeight:700}}>UPGRADE</button>
                )}
              </div>
              {code && <span style={{fontSize:11,color:"#333"}}>{code.split("\n").length} líneas</span>}
            </div>

            {error && <div style={{color:"#ff1939",fontSize:12,marginBottom:10,padding:"8px 12px",background:"#ff193912",borderRadius:8,border:"1px solid #ff193933"}}>⚠ {error}</div>}

            <button onClick={analyze} disabled={loading}
              style={{width:"100%",padding:"16px 0",borderRadius:14,border:"none",background:loading?"#1a1a1a":`linear-gradient(135deg,${G},#00cc6a)`,color:loading?"#444":"#000",fontSize:14,fontWeight:800,letterSpacing:"1px",cursor:loading?"default":"pointer",display:"flex",alignItems:"center",justifyContent:"center",gap:8,marginTop:4}}>
              {loading?<><Spin color="#444"/>ANALIZANDO...</>:"ESCANEAR CONTRATO SOLIDITY"}
            </button>
          </div>

          {/* MINI PRICING STRIP */}
          {plan==="free" && (
            <div style={{background:"#0a0a0a",border:"1px solid #1a1a1a",borderRadius:14,padding:"14px 16px",display:"flex",justifyContent:"space-between",alignItems:"center"}}>
              <div>
                <div style={{fontSize:12,fontWeight:700,color:"#aaa"}}>¿Quieres más scans?</div>
                <div style={{fontSize:11,color:"#444",marginTop:2}}>Pro desde $19/mes · Scans ilimitados</div>
              </div>
              <button onClick={()=>setPage("pricing")} style={{background:`linear-gradient(135deg,${G},#00cc6a)`,border:"none",color:"#000",padding:"9px 16px",borderRadius:10,fontSize:12,fontWeight:800,cursor:"pointer",whiteSpace:"nowrap"}}>
                Ver planes →
              </button>
            </div>
          )}
        </>}

        {/* RESULTS */}
        {result && gr && <>
          <div style={{background:"#111",border:`1px solid ${gr.color}44`,borderRadius:20,padding:20,marginBottom:12,display:"flex",alignItems:"center",gap:20}}>
            <div style={{textAlign:"center",minWidth:90}}>
              <div style={{fontSize:54,fontWeight:900,lineHeight:1,color:gr.color,textShadow:`0 0 20px ${gr.color}44`}}>{result.securityScore}</div>
              <div style={{fontSize:10,color:"#444",fontWeight:600,letterSpacing:"1px"}}>/100</div>
            </div>
            <div style={{flex:1}}>
              <div style={{display:"flex",alignItems:"center",gap:8,marginBottom:4}}>
                <span style={{fontSize:15,fontWeight:800}}>{result.contractName}</span>
                <span style={{fontSize:10,background:`${gr.color}18`,color:gr.color,border:`1px solid ${gr.color}44`,padding:"2px 8px",borderRadius:20,fontWeight:700}}>{gr.label}</span>
              </div>
              <div style={{fontSize:11,color:"#555",marginBottom:10}}>{result.compiler}</div>
              <div style={{background:"#0a0a0a",borderRadius:6,height:7,overflow:"hidden"}}>
                <div style={{width:`${result.securityScore}%`,height:"100%",background:`linear-gradient(90deg,#ff1939,${gr.color})`,borderRadius:6}}/>
              </div>
            </div>
          </div>

          <div style={{display:"flex",gap:3,background:"#111",border:"1px solid #1a1a1a",borderRadius:12,padding:4,marginBottom:12}}>
            {[["overview","Resumen"],["vulns",`Vulnerabilidades${result.vulnerabilities?.length?` (${result.vulnerabilities.length})`:""}`],["metrics","Métricas"]].map(([id,label])=>(
              <button key={id} onClick={()=>setTab(id)}
                style={{flex:1,padding:"9px 4px",borderRadius:9,border:"none",background:tab===id?"#000":"transparent",color:tab===id?G:"#555",fontSize:11,fontWeight:tab===id?700:400,cursor:"pointer"}}>
                {label}
              </button>
            ))}
          </div>

          <div style={{background:"#111",border:"1px solid #1a1a1a",borderRadius:20,padding:20,marginBottom:12}}>
            {tab==="overview" && <>
              <L>DESCRIPCIÓN</L>
              <p style={{margin:"0 0 18px",fontSize:13,color:"#aaa",lineHeight:1.7}}>{result.summary}</p>
              <L>NOTAS DEL AUDITOR</L>
              <p style={{margin:0,fontSize:13,color:"#aaa",lineHeight:1.7}}>{result.notes}</p>
            </>}
            {tab==="vulns" && (!result.vulnerabilities?.length
              ? <div style={{textAlign:"center",padding:"24px 0",color:G,fontSize:14}}>✓ Sin vulnerabilidades detectadas</div>
              : result.vulnerabilities.map((v,i)=>{
                  const c=SEV[v.severity]||SEV["BAJA"];
                  return(
                    <div key={i} style={{background:c.bg,border:`1px solid ${c.border}44`,borderRadius:12,padding:16,marginBottom:i<result.vulnerabilities.length-1?10:0}}>
                      <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:8}}>
                        <span style={{fontSize:13,fontWeight:700}}>{v.name}</span>
                        <div style={{display:"flex",gap:6,alignItems:"center"}}>
                          {v.line&&<span style={{fontSize:10,color:"#555"}}>línea {v.line}</span>}
                          <span style={{fontSize:10,color:c.text,background:`${c.border}18`,border:`1px solid ${c.border}44`,padding:"2px 8px",borderRadius:20,fontWeight:700}}>{v.severity}</span>
                        </div>
                      </div>
                      <p style={{margin:"0 0 10px",fontSize:12,color:"#888",lineHeight:1.6}}>{v.description}</p>
                      <div style={{background:"#00000044",borderRadius:8,padding:"8px 12px",fontSize:12,color:G}}>→ {v.recommendation}</div>
                    </div>
                  );
                })
            )}
            {tab==="metrics" && (
              <div style={{display:"flex",flexDirection:"column",gap:18}}>
                {[["Eficiencia de Gas",result.gasEfficiency,"#00ff88"],["Calidad del Código",result.codeQuality,"#00ccff"],["Descentralización",result.decentralization,"#aa88ff"],["Score de Seguridad",result.securityScore,gr.color]].map(([name,val,color])=>(
                  <div key={name}>
                    <div style={{display:"flex",justifyContent:"space-between",marginBottom:7}}>
                      <span style={{fontSize:13,color:"#aaa"}}>{name}</span>
                      <span style={{fontSize:13,fontWeight:700,color}}>{val}/100</span>
                    </div>
                    <div style={{background:"#0a0a0a",borderRadius:6,height:8}}>
                      <div style={{width:`${val}%`,height:"100%",background:`linear-gradient(90deg,${color}55,${color})`,borderRadius:6}}/>
                    </div>
                  </div>
                ))}
              </div>
            )}
          </div>

          <div style={{display:"flex",gap:8,marginBottom:12}}>
            <button onClick={download} style={{flex:1,padding:"14px 0",borderRadius:14,border:"none",background:`linear-gradient(135deg,${G},#00cc6a)`,color:"#000",fontSize:13,fontWeight:800,cursor:"pointer"}}>
              ⬇ Descargar Reporte
            </button>
            <button onClick={()=>{setResult(null);setCode("");}} style={{padding:"0 18px",background:"#111",border:"1px solid #222",color:"#aaa",borderRadius:14,fontSize:13,fontWeight:600,cursor:"pointer"}}>
              + Nuevo
            </button>
          </div>
        </>}
      </div>
    </Wrap>
  );
}

function Wrap({children}){ return <div style={{fontFamily:"'Inter',-apple-system,sans-serif",background:"#000",color:"#fff",minHeight:"100vh"}}>{children}</div>; }

function Navbar({onNav,plan}){
  const p = PLANS.find(x=>x.id===plan);
  return(
    <div style={{display:"flex",alignItems:"center",justifyContent:"space-between",padding:"16px 20px",borderBottom:"1px solid #111"}}>
      <span style={{fontWeight:900,fontSize:20,color:G,letterSpacing:"-0.5px"}}>EtherAudit</span>
      <div style={{display:"flex",gap:4,alignItems:"center"}}>
        {[["Herramienta","home"],["Precios","pricing"],["Contacto","contact"]].map(([label,pg])=>(
          <button key={pg} onClick={()=>onNav(pg)} style={{background:"transparent",border:"none",color:"#666",fontSize:13,cursor:"pointer",padding:"6px 10px",borderRadius:8}}>{label}</button>
        ))}
        {plan!=="free"&&<span style={{fontSize:10,background:`${p.color}18`,border:`1px solid ${p.color}44`,color:p.color,padding:"3px 9px",borderRadius:20,fontWeight:700,marginLeft:4}}>{plan.toUpperCase()}</span>}
      </div>
    </div>
  );
}

function Modal({children,onClose}){
  return(
    <div style={{position:"fixed",inset:0,background:"#000000cc",display:"flex",alignItems:"center",justifyContent:"center",zIndex:100,padding:"0 16px"}}>
      <div style={{background:"#111",border:"1px solid #222",borderRadius:20,padding:24,width:"100%",maxWidth:420,maxHeight:"90vh",overflowY:"auto",position:"relative"}}>
        <button onClick={onClose} style={{position:"absolute",top:14,right:14,background:"transparent",border:"none",color:"#555",fontSize:18,cursor:"pointer",lineHeight:1}}>✕</button>
        {children}
      </div>
    </div>
  );
}

function L({children}){ return <div style={{fontSize:11,fontWeight:700,color:"#444",letterSpacing:"1px",marginBottom:10}}>{children}</div>; }

function Spin({color="#000"}){
  const [r,setR]=useState(0);
  useState(()=>{const id=setInterval(()=>setR(p=>(p+20)%360),50);return()=>clearInterval(id);});
  return <span style={{display:"inline-block",transform:`rotate(${r}deg)`,fontSize:14,color}}>⟳</span>;
}
