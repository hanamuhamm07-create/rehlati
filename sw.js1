const C='rehlati-v5';
const FONT_CSS='https://fonts.googleapis.com/css2?family=Amiri:wght@400;700&family=Readex+Pro:wght@300;400;500;600&display=swap';
async function precache(){
  const c=await caches.open(C),scope=self.registration.scope;
  // صفحة التطبيق نفسها
  for(const u of [scope,scope+'index.html']){try{const r=await fetch(u,{cache:'reload'});if(r.ok)await c.put(u,r);}catch(e){}}
  // الخطوط: ملف التنسيق وملفات الخطوط التي يشير إليها
  try{
    const r=await fetch(FONT_CSS);if(r.ok){
      const css=await r.clone().text();await c.put(FONT_CSS,r);
      const urls=[...new Set([...css.matchAll(/url\((https:[^)]+)\)/g)].map(m=>m[1]))];
      await Promise.all(urls.map(async u=>{try{const f=await fetch(u);if(f.ok)await c.put(u,f);}catch(e){}}));
    }
  }catch(e){}
}
self.addEventListener('install',e=>{self.skipWaiting();e.waitUntil(precache());});
self.addEventListener('activate',e=>{e.waitUntil(caches.keys().then(ks=>Promise.all(ks.filter(k=>k!==C&&k!=='rehlati-cfg').map(k=>caches.delete(k)))).then(()=>self.clients.claim()));});
self.addEventListener('fetch',e=>{
  const r=e.request;if(r.method!=='GET')return;
  const u=new URL(r.url),font=u.hostname==='fonts.googleapis.com'||u.hostname==='fonts.gstatic.com';
  if(u.origin!==self.location.origin&&!font)return;
  const keep=res=>{if(res&&(res.ok||res.type==='opaque')){const cp=res.clone();caches.open(C).then(c=>c.put(r,cp));}return res;};
  if(u.origin===self.location.origin){
    e.respondWith(fetch(r).then(keep).catch(async()=>{
      return (await caches.match(r))||(await caches.match(r,{ignoreSearch:true}))||(r.mode==='navigate'?((await caches.match(self.registration.scope))||(await caches.match(self.registration.scope+'index.html'))):undefined)||Response.error();
    }));
  }else{
    e.respondWith(caches.match(r).then(m=>m||fetch(r).then(keep)));
  }
});
self.addEventListener('notificationclick',e=>{
  e.notification.close();
  e.waitUntil(clients.matchAll({type:'window',includeUncontrolled:true}).then(l=>{for(const c of l){if('focus' in c)return c.focus();}return clients.openWindow(self.registration.scope);}));
});

async function remindIfDue(){
  try{
    const c=await caches.open('rehlati-cfg'),key=self.registration.scope+'cfg.json',r=await c.match(key);if(!r)return;
    const cfg=await r.json();if(!cfg.on)return;
    const n=new Date(),k=n.getFullYear()+'-'+String(n.getMonth()+1).padStart(2,'0')+'-'+String(n.getDate()).padStart(2,'0');
    if(cfg.last===k)return;
    const [h,m]=String(cfg.time||'17:00').split(':').map(Number),diff=n.getHours()*60+n.getMinutes()-(h*60+m);
    if(diff<0||diff>240)return;
    const wins=await clients.matchAll({type:'window'});if(wins.some(w=>w.visibilityState==='visible'))return;
    await self.registration.showNotification('حان وقت رحلتك',{body:'ابدأ بما تستطيع، والباقي خطوة بخطوة. وربما تنتظرك مراجعات.',lang:'ar',dir:'rtl',tag:'rehlati-remind'});
    cfg.last=k;await c.put(key,new Response(JSON.stringify(cfg),{headers:{'Content-Type':'application/json'}}));
  }catch(e){}
}
self.addEventListener('periodicsync',e=>{if(e.tag==='rehlati-remind')e.waitUntil(remindIfDue());});
