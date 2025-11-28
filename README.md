<!doctype html>
<html lang="es">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width,initial-scale=1">
  <title>Revisor: Marco Teórico — Psicología Clínica</title>
  <style>
    /* Estilos generales (Times New Roman por defecto según lo solicitado) */
    body{font-family: 'Times New Roman', Times, serif; margin:20px; background:#f7f7fb; color:#111}
    h1{font-size:24px; margin-bottom:4px}
    label{display:block; margin-top:12px; font-weight:600}
    .controls{display:flex; gap:12px; flex-wrap:wrap; align-items:center}
    textarea, input[type=text], input[type=file]{width:100%; padding:10px; border-radius:8px; border:1px solid #ccc; box-sizing:border-box}
    button{background:#2b6cb0;color:white;border:none;padding:8px 12px;border-radius:8px;cursor:pointer}
    .row{display:grid; grid-template-columns:1fr 1fr; gap:16px; margin-top:12px}
    .panel{background:white;border-radius:10px;padding:14px; box-shadow:0 2px 6px rgba(0,0,0,0.06)}
    .preview{border:1px dashed #cbd5e0; padding:18px; min-height:240px}
    .report{white-space:pre-wrap; font-family:monospace; font-size:13px}
    .ok{color:green} .warn{color:orange} .err{color:red}
    small{color:#666}
    .badge{display:inline-block; background:#edf2f7; padding:4px 8px; border-radius:6px; font-size:13px; margin-right:6px}
    footer{margin-top:18px; font-size:13px; color:#555}
  </style>
</head>
<body>
  <h1>Revisor automático - Marco Teórico (Psicología Clínica)</h1>
  <p>Este <strong>index.html</strong> puede publicarse en GitHub Pages. Actúa como tutor/revisor automático que revisa los requisitos solicitados para un MARCO TEÓRICO. Pegue el texto del marco teórico o suba un archivo .txt. (Para .docx, convierta a .txt o pegue el contenido.)</p>

  <div class="panel">
    <label>Título de la investigación (será usado para comparar coherencia)</label>
    <input id="paperTitle" type="text" placeholder="Ej. Influencia del apego en la ansiedad infantil">

    <label>Texto del Marco Teórico (pegue aquí)</label>
    <textarea id="inputText" rows="12" spellcheck="true" placeholder="Pegue el marco teórico..." ></textarea>

    <div class="controls">
      <button id="runBtn">Ejecutar revisión</button>
      <button id="previewBtn">Mostrar vista previa (Times New Roman, fuente 12, títulos 14)</button>
      <button id="downloadReport">Descargar informe (JSON)</button>
      <small>Recomendación: use Word para confirmar <em>interlineado 1.5</em> y tamaño exacto. Esta herramienta ofrece comprobaciones automáticas y sugerencias.</small>
    </div>
  </div>

  <div class="row">
    <div class="panel">
      <h2>Informe de revisión</h2>
      <div id="report" class="report">Sin revisión aún. Presione "Ejecutar revisión".</div>
    </div>

    <div class="panel">
      <h2>Vista previa formateada</h2>
      <div class="preview" id="previewArea" style="font-size:12pt; line-height:1.5; text-align:justify">
        <h3 id="previewTitle" style="font-size:14pt; margin-bottom:6px; text-align:center"></h3>
        <div id="previewText"></div>
      </div>
      <small>La vista previa aplica: Times New Roman, tamaño 12 para contenido y 14 para títulos; texto justificado y espaciado 1.5.</small>
    </div>
  </div>

  <footer>
    <div class="badge">Comprobaciones automáticas:</div>
    <span>Fuente / tamaño / interlineado / cita (Apellido (AAAA),) / antigüedad <= 10 años / voz: tercera persona / tiempo: presente / puntuación / ortografía básica / conectores / plagio simple (duplicación interna)</span>
  </footer>

  <script>
    // Fecha de referencia (según el sistema) -> 2025-11-28. Vamos a usar el año actual real para lógica: 2025
    const CURRENT_YEAR = 2025; // si se publica después, asegúrese de actualizar
    const MAX_AGE = 10; // años
    const MIN_YEAR = CURRENT_YEAR - MAX_AGE; // >= 2015

    function runChecks(title, text){
      const report = [];

      // Normalizaciones
      const trimmed = text.trim();
      if(!trimmed) return {ok:false, report:['El texto está vacío.']};

      // 1) Tamaño de texto y fuente: no se puede inspeccionar un .docx desde aquí, pero podemos confirmar vista previa
      report.push('1) Fuente y tamaño: La vista previa aplica Times New Roman; confirmar en Word que el cuerpo está en 12pt y títulos en 14pt.');

      // 2) Interlineado
      report.push('2) Interlineado: La vista previa aplica line-height 1.5. Confirme en Word: Formato -> Párrafo -> Interlineado 1.5.');

      // 3) Citas: buscar coincidencias del patrón Apellido (YYYY),
      const citaRegex = /([A-ZÁÉÍÓÚÑ][a-záéíóúñ]+)\s*\(\s*(19|20)\d{2}\s*\),/g;
      const citas = [];
      let m;
      while((m = citaRegex.exec(trimmed))){
        citas.push({apellido:m[1], year:parseInt(m[2]+m[0].slice(m[0].match(/\d{2}$/)?-2: -2))});
      }
      // Above attempt to parse year is messy. Simpler: extract year via different regex
      const citaYearRegex = /\(([0-9]{4})\)\,/g;
      const citaFullRegex = /([A-ZÁÉÍÓÚÑ][a-záéíóúñ]+)\s*\(\s*([0-9]{4})\s*\),/g;
      const citasOk = [];
      while((m = citaFullRegex.exec(trimmed))){
        citasOk.push({apellido:m[1], year:parseInt(m[2])});
      }

      if(citasOk.length===0){
        report.push('3) Citas: No se encontraron citas que cumplan el patrón exacto "Apellido (AAAA),". Verifique formato. Ejemplo correcto: Sampieri (2000),');
      } else {
        report.push(`3) Citas encontradas: ${citasOk.length}.`);
        // verificar antigüedad
        const añosFuera = citasOk.filter(c=>c.year < MIN_YEAR);
        if(añosFuera.length>0){
          report.push(`   - Fuera de rango (más antiguas que ${MIN_YEAR}): ${añosFuera.map(c=>c.apellido+' ('+c.year+')').join(', ')}. Deben preferirse fuentes publicadas a partir de ${MIN_YEAR} inclusive.`);
        } else {
          report.push(`   - Todas las citas detectadas cumplen la restricción de antigüedad (>= ${MIN_YEAR}).`);
        }
      }

      // 4) Coherencia con el título: buscar palabras clave del título dentro del texto
      const titleWords = title.split(/\s+/).filter(w=>w.length>3).map(w=>w.toLowerCase());
      const textLower = trimmed.toLowerCase();
      const foundTitleWords = titleWords.filter(w=> textLower.includes(w));
      if(titleWords.length===0){
        report.push('4) Coherencia con el título: No se proporcionó título para comparar.');
      } else {
        const ratio = (foundTitleWords.length / titleWords.length);
        report.push(`4) Coherencia con el título: se encontraron ${foundTitleWords.length} de ${titleWords.length} palabras clave largas del título (${Math.round(ratio*100)}%). Palabras encontradas: ${foundTitleWords.join(', ')}`);
        if(ratio<0.5) report.push('   - Advertencia: el texto parece poco relacionado con el título (menor al 50% de coincidencia en palabras claves).');
      }

      // 5) Puntuación básica: detectar comas pegadas (incorrectas) o espacios antes de coma, espacios dobles, puntos seguidos sin espacio
      const issuesPunt = [];
      if(/\s+,/.test(trimmed)) issuesPunt.push('Espacio antes de coma detectado.');
      if(/,{2,}/.test(trimmed)) issuesPunt.push('Comas consecutivas detectadas.');
      if(/\.\s*\./.test(trimmed)) issuesPunt.push('Puntos suspensivos mal formados detectados.');
      if(/\s{2,}/.test(trimmed)) issuesPunt.push('Espacios múltiples detectados.');
      if(issuesPunt.length===0) report.push('5) Puntuación: No se detectaron problemas puntuales simples (espacios antes de comas, comas dobles, espacios múltiples).');
      else report.push('5) Puntuación: '+issuesPunt.join(' | '));

      // 6) Ortografía: no se puede hacer un corrector completo sin servicio externo; sugerimos usar el corrector de Word y el spellcheck del navegador.
      report.push('6) Ortografía: Recomendado ejecutar el corrector ortográfico de MS Word y revisar sugerencias. Este verificador no reemplaza al corrector profesional.');

      // 7) Tiempo verbal y persona: buscar pronombres en primera persona (yo, nosotros) y formas en pasado frecuentes
      const primera = /(\byo\b|\bnosotros\b|\bnosotras\b|\bnos|\bmi\b|\bnuestro\b|\bmi\b)/i.test(trimmed);
      if(primera) report.push('7) Persona/tiempo: Se detectaron pronombres de primera persona (ej. "yo", "nosotros"). El texto debe presentarse en tercera persona.');
      else report.push('7) Persona: No se detectaron pronombres de primera persona (OK).');

      // Detección simple de pasado: palabras comunes en pasado
      const pasadoRegex = /\b(fue|fueron|era|eran|realizó|realizaron|realizado|se realizó|tuvo|hubo|había|hicieron|hizo)\b/gi;
      const pasados = (trimmed.match(pasadoRegex)||[]).length;
      if(pasados>0) report.push(`   - Tiempo verbal: Se encontraron ${pasados} ocurrencias de marcadores de pasado. Se recomienda redactar en tiempo presente.`);
      else report.push('   - Tiempo verbal: No se detectaron marcadores claros de pasado (OK).');

      // 8) Uso de conectores: contar conectores frecuentes
      const conectores = ['además','sin embargo','por lo tanto','por tanto','en consecuencia','no obstante','aunque','por otra parte','asimismo','mientras tanto','debido a','causa de'];
      const conectoresEncontrados = conectores.filter(c=> textLower.includes(c));
      report.push(`8) Conectores: se encontraron ${conectoresEncontrados.length} conectores comunes: ${conectoresEncontrados.join(', ')}`);
      if(conectoresEncontrados.length<2) report.push('   - Sugerencia: aumentar uso de conectores para cohesión textual.');

      // 9) Parafraseo / plagio interno: detectar oraciones repetidas exactamente
      const sentences = trimmed.split(/[\.\n\?\!]+/).map(s=>s.trim()).filter(Boolean);
      const duplicates = [];
      const seen = {};
      sentences.forEach(s=>{
        const key = s.toLowerCase();
        if(seen[key]) seen[key]++;
        else seen[key]=1;
      });
      for(const s in seen) if(seen[s]>1) duplicates.push({text:s, count:seen[s]});
      if(duplicates.length>0) report.push(`9) Duplicación interna: se encontraron ${duplicates.length} oraciones repetidas exactamente. Ejemplos: "${duplicates.slice(0,3).map(d=>d.text).join('" | "')}"`);
      else report.push('9) Parafraseo / plagio (interna): No se detectaron oraciones idénticas repetidas.' );

      // 10) Proporción de citas vs texto: contar citas detectadas
      const numCitas = citasOk.length;
      const numSent = sentences.length;
      const citaRatio = (numSent===0)?0:(numCitas/numSent);
      report.push(`10) Referencias en texto: ${numCitas} citas en el cuerpo; oraciones ${numSent}; ratio citas/oración = ${citaRatio.toFixed(2)} (evaluar si es suficiente para respaldar afirmaciones).`);

      // 11) Recomendaciones finales
      report.push('\nRECOMENDACIONES:');
      report.push('- Asegurar que todas las fuentes principales sean de los últimos 10 años (>= '+MIN_YEAR+').');
      report.push('- Confirmar formato exacto de citas: "Apellido (AAAA),". Revise también la lista de referencias al final (formato APA u otro requerido).');
      report.push('- Usar tercera persona y tiempo presente. Evitar pronombres en primera persona y marcadores de pasado.');
      report.push('- Ejecutar corrector ortográfico y revisar puntuación manualmente.');
      report.push('- Mejorar cohesión con conectores; usar transiciones entre párrafos.');
      report.push('- Revisar posibles duplicaciones textuales con software de detección de plagio para obtener porcentaje real frente a fuentes externas.');

      return {ok:true, report};
    }

    document.getElementById('runBtn').addEventListener('click', ()=>{
      const title = document.getElementById('paperTitle').value || '';
      const text = document.getElementById('inputText').value || '';
      const out = runChecks(title, text);
      const rEl = document.getElementById('report');
      if(!out.ok){ rEl.textContent = out.report.join('\n'); return; }
      rEl.textContent = out.report.join('\n');
      // prepare downloadable report data
      window.latestReport = {title, text, generatedAt: new Date().toISOString(), report: out.report};
    });

    document.getElementById('previewBtn').addEventListener('click', ()=>{
      const t = document.getElementById('paperTitle').value || '';
      const txt = document.getElementById('inputText').value || '';
      document.getElementById('previewTitle').textContent = t;
      // simple paragraph splitting
      const paras = txt.split(/\n\n+/).map(p=>p.trim()).filter(Boolean);
      const previewDiv = document.getElementById('previewText');
      previewDiv.innerHTML = '';
      paras.forEach(p=>{
        const pEl = document.createElement('p');
        pEl.style.marginBottom = '12px';
        pEl.style.fontSize = '12pt';
        pEl.textContent = p;
        previewDiv.appendChild(pEl);
      });
      // apply styles: justification and line-height already in CSS
    });

    document.getElementById('downloadReport').addEventListener('click', ()=>{
      if(!window.latestReport){ alert('No hay informe generado. Presione "Ejecutar revisión" primero.'); return; }
      const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(window.latestReport, null, 2));
      const dl = document.createElement('a'); dl.setAttribute('href', dataStr); dl.setAttribute('download', 'reporte_marco_teorico.json'); document.body.appendChild(dl); dl.click(); dl.remove();
    });

    // Optional: enable dropping a .txt file
    document.addEventListener('dragover', e=>e.preventDefault());
    document.addEventListener('drop', e=>e.preventDefault());

    // Basic file input (txt)
    const fileInput = document.createElement('input');
    fileInput.type = 'file'; fileInput.accept = '.txt, .docx';
    fileInput.style.display='none';
    document.body.appendChild(fileInput);
    const fileBtn = document.createElement('button');
    fileBtn.textContent = 'Cargar .txt'; fileBtn.style.marginLeft='6px';
    document.querySelector('.controls').appendChild(fileBtn);
    fileBtn.addEventListener('click', ()=>fileInput.click());
    fileInput.addEventListener('change', async (e)=>{
      const f = e.target.files[0]; if(!f) return;
      const ext = f.name.split('.').pop().toLowerCase();
      if(ext === 'txt'){
        const reader = new FileReader();
        reader.onload = ()=>{ document.getElementById('inputText').value = reader.result; alert('Archivo .txt cargado. Presione Ejecutar revisión.'); };
        reader.readAsText(f, 'UTF-8');
      } else if(ext === 'docx'){
        if(!window.mammoth){ await new Promise(res=>{
          const s=document.createElement('script'); s.src='https://unpkg.com/mammoth/mammoth.browser.min.js'; s.onload=res; document.body.appendChild(s);
        }); }
        const arrayBuffer = await f.arrayBuffer();
        mammoth.extractRawText({arrayBuffer}).then(result=>{
          document.getElementById('inputText').value = result.value;
          alert('Archivo .docx cargado. Presione Ejecutar revisión.');
        });
      }
    });(f, 'UTF-8');
    });

  </script>
</body>
</html>
