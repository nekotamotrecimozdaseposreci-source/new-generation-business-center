const crypto = require('node:crypto');
const groups=['komunikacija','sestomesecni','akademija','licni','beauty','skincare','wellness','lifestyle','anketa'];

module.exports=async(req,res)=>{
 res.setHeader('Cache-Control','no-store');

 const url=process.env.UPSTASH_REDIS_REST_URL||process.env.KV_REST_API_URL||process.env.STORAGE_REST_API_URL||process.env.STORAGE_REST_URL;
 const token=process.env.UPSTASH_REDIS_REST_TOKEN||process.env.KV_REST_API_TOKEN||process.env.STORAGE_REST_API_TOKEN||process.env.STORAGE_REST_TOKEN;

 async function redis(...args){
 const r=await fetch(url,{
 method:'POST',
 headers:{
 Authorization:`Bearer ${token}`,
 'Content-Type':'application/json'
 },
 body:JSON.stringify(args)
 });
 const j=await r.json();
 if(!r.ok||j.error)throw Error('Redis');
 return j.result;
 }

 try{
 if(req.method==='GET'){
 const rows=await redis('LRANGE','ngbc:reviews',0,199);
 return res.json(rows.map(x=>JSON.parse(x)));
 }

 if(req.method!=='POST')return res.status(405).end();

 const b=typeof req.body==='string'?JSON.parse(req.body):req.body||{};
 const {group,rating}=b;
 const name=typeof b.name==='string'?b.name.trim():'';
 const comment=typeof b.comment==='string'?b.comment.trim():'';

 if(!groups.includes(group)||!Number.isInteger(rating)||rating<1||rating>5||name.length<2||name.length>80||comment.length<5||comment.length>1000||b.website)
 return res.status(400).json({error:'Proverite podatke.'});

 const ip=String(req.headers['x-forwarded-for']||req.socket.remoteAddress||'unknown').split(',')[0];
 const hash=crypto.createHash('sha256').update(ip).digest('hex');

 if(!await redis('SET','ngbc:limit:'+hash,'1','NX','EX',60))
 return res.status(429).json({error:'Sačekajte minut.'});

 const review={
 id:crypto.randomUUID(),
 group,
 rating,
 name,
 comment,
 date:new Date().toISOString()
 };

 await redis('LPUSH','ngbc:reviews',JSON.stringify(review));
 return res.status(201).json(review);

 }catch(e){
 return res.status(503).json({error:'Recenzije trenutno nisu dostupne.'});
 }
};
