<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Sawti World</title>
  <script src="https://cdn.jsdelivr.net/npm/tesseract.js@4/dist/tesseract.min.js"></script>
  <style>
    :root{
      --bg:#0b0b0f;
      --panel:#12141b;
      --card:#181b25;
      --soft:#1e2330;
      --line:#2b3242;
      --text:#f5f7ff;
      --muted:#b9c0d1;
      --pink:#ff2a6d;
      --pink2:#ff5c97;
      --blue:#4fb3ff;
      --green:#32d296;
      --red:#ff5a5a;
      --shadow:0 18px 40px rgba(0,0,0,.28);
    }

    *{box-sizing:border-box}

    html,body{
      margin:0;
      min-height:100%;
      background:radial-gradient(circle at top,#171b27,#0b0b0f 45%);
      color:var(--text);
      font-family:system-ui,-apple-system,"Segoe UI",Tahoma,sans-serif;
    }

    body{
      padding:24px 14px 50px;
      display:flex;
      align-items:flex-start;
      justify-content:center;
    }

    .app{
      width:100%;
      max-width:500px;
      background:rgba(18,20,27,.92);
      border:1px solid var(--line);
      border-radius:24px;
      box-shadow:var(--shadow);
      padding:18px 16px 20px;
      backdrop-filter:blur(8px);
    }

    h1{
      margin:6px 0 18px;
      font-size:clamp(2rem,5vw,2.8rem);
      text-align:center;
      letter-spacing:.5px;
    }

    .row{
      display:grid;
      grid-template-columns:1fr 1fr;
      gap:10px;
      margin-bottom:12px;
    }

    select,textarea,.box{
      width:100%;
      border:1px solid var(--line);
      background:var(--card);
      color:var(--text);
      border-radius:14px;
      padding:12px 14px;
      font-size:16px;
      outline:none;
      transition:.2s ease;
    }

    select:focus,textarea:focus{
      border-color:var(--pink);
      box-shadow:0 0 0 3px rgba(255,42,109,.18);
    }

    textarea{
      min-height:126px;
      resize:vertical;
      text-align:right;
      margin-top:10px;
    }

    .actions{
      display:flex;
      flex-wrap:wrap;
      gap:8px;
      justify-content:center;
      margin:10px 0;
    }

    button{
      border:none;
      border-radius:12px;
      padding:12px 16px;
      font-weight:800;
      cursor:pointer;
      font-size:15px;
      transition:transform .15s ease, opacity .15s ease, filter .15s ease;
      user-select:none;
    }

    button:hover{transform:translateY(-1px)}
    button:active{transform:translateY(0)}
    button:disabled{opacity:.6;cursor:wait}

    .primary{
      background:linear-gradient(135deg,var(--pink),var(--pink2));
      color:#fff;
      box-shadow:0 10px 18px rgba(255,42,109,.25);
    }

    .secondary{
      background:#f4f6fb;
      color:#111;
    }

    .ghost{
      background:var(--soft);
      color:var(--text);
      border:1px solid var(--line);
    }

    .box{
      min-height:72px;
      display:flex;
      align-items:center;
      justify-content:flex-start;
      background:#0f1117;
      border:1px dashed var(--line);
      white-space:pre-wrap;
      word-break:break-word;
      text-align:right;
      margin:12px 0 8px;
      line-height:1.7;
    }

    #in,#out{
      direction:rtl;
    }

    .status{
      min-height:24px;
      text-align:center;
      font-size:14px;
      color:var(--muted);
      margin-top:8px;
    }

    .status.success{color:var(--green)}
    .status.error{color:var(--red)}
    .status.info{color:var(--blue)}

    #fileInput{display:none}

    @media (max-width:420px){
      .row{
        grid-template-columns:1fr;
      }
    }
  </style>
</head>
<body>
  <div class="app">
    <h1>🌍 Sawti World</h1>

    <div class="row">
      <select id="from" aria-label="لغة المصدر">
        <option value="ar">العربية</option>
        <option value="en">English</option>
        <option value="fr">Français</option>
        <option value="es">Español</option>
        <option value="tr">Türkçe</option>
        <option value="hi">हिन्दी</option>
        <option value="zh">中文</option>
        <option value="sw">Kiswahili</option>
        <option value="ha">Hausa</option>
        <option value="am">አማርኛ</option>
      </select>

      <select id="to" aria-label="لغة الهدف">
        <option value="fr">Français</option>
        <option value="ar">العربية</option>
        <option value="en">English</option>
        <option value="es">Español</option>
      </select>
    </div>

    <div class="actions">
      <button class="primary" id="voiceBtn" onclick="startVoice()">🎙️ دوس وتكلم</button>
      <button class="secondary" onclick="document.getElementById('fileInput').click()">📸 صور نص</button>
    </div>

    <input id="fileInput" type="file" accept="image/*" capture="environment" onchange="readImage(event)" />

    <textarea id="txt" placeholder="اكتب النص هنا..." aria-label="نص للترجمة"></textarea>

    <div class="box" id="in">النص هنا</div>

    <div class="actions">
      <button class="primary" id="translateBtn" onclick="translateText()">🌍 ترجم الآن</button>
      <button class="ghost" onclick="copyOutput()">📋 نسخ</button>
      <button class="ghost" onclick="speakOutput()">🔊 نطق</button>
    </div>

    <div class="box" id="out">الترجمة هنا</div>
    <div id="status" class="status" aria-live="polite"></div>
  </div>

  <script>
    const fromSelect = document.getElementById('from');
    const toSelect = document.getElementById('to');
    const txtInput = document.getElementById('txt');
    const inBox = document.getElementById('in');
    const outBox = document.getElementById('out');
    const statusBox = document.getElementById('status');
    const translateBtn = document.getElementById('translateBtn');
    const voiceBtn = document.getElementById('voiceBtn');

    const TESSERACT_LANG = {
      ar: 'ara',
      en: 'eng',
      fr: 'fra',
      es: 'spa',
      tr: 'tur',
      hi: 'hin',
      zh: 'chi_sim',
      sw: 'swa',
      ha: 'hau',
      am: 'amh'
    };

    function setStatus(message, type = 'info') {
      statusBox.textContent = message;
      statusBox.className = 'status ' + type;
    }

    function setBusy(isBusy) {
      translateBtn.disabled = isBusy;
      voiceBtn.disabled = isBusy;
    }

    function getCurrentText() {
      return (txtInput.value || inBox.textContent || '').trim();
    }

    function setInputText(value) {
      const cleaned = value || '';
      txtInput.value = cleaned;
      inBox.textContent = cleaned || 'النص هنا';
    }

    function startVoice() {
      const SR = window.SpeechRecognition || window.webkitSpeechRecognition;
      if (!SR) {
        setStatus('متصفحك لا يدعم التعرف الصوتي.', 'error');
        return;
      }

      try {
        const recognition = new SR();
        recognition.lang = fromSelect.value;
        recognition.interimResults = false;
        recognition.maxAlternatives = 1;

        recognition.onstart = () => setStatus('جارٍ الاستماع...', 'info');
        recognition.onresult = (event) => {
          const transcript = event.results[0][0].transcript;
          setInputText(transcript);
          setStatus('تم التعرف على النص بنجاح.', 'success');
        };
        recognition.onerror = (event) => {
          setStatus('تعذّر التعرف الصوتي: ' + (event.error || 'غير معروف'), 'error');
        };
        recognition.onend = () => setStatus('انتهى الاستماع.', 'success');

        recognition.start();
      } catch (err) {
        setStatus('تعذّر بدء التعرف الصوتي.', 'error');
      }
    }

    async function translateText() {
      const text = getCurrentText();
      const from = fromSelect.value;
      const to = toSelect.value;

      if (!text) {
        outBox.textContent = 'يرجى إدخال نص أولاً.';
        setStatus('لا يوجد نص للترجمة.', 'error');
        return;
      }

      setBusy(true);
      outBox.textContent = '⏳ جارٍ الترجمة...';
      setStatus('جارٍ ترجمة النص...', 'info');

      try {
        const url = `https://api.mymemory.translated.net/get?q=${encodeURIComponent(text)}&langpair=${from}|${to}`;
        const res = await fetch(url);

        if (!res.ok) {
          throw new Error(`HTTP ${res.status}`);
        }

        const data = await res.json();
        const translated = data?.responseData?.translatedText;

        if (!translated) {
          throw new Error('لم يتم إرجاع ترجمة.');
        }

        outBox.textContent = translated;
        setStatus('تمت ترجمة النص بنجاح.', 'success');
      } catch (err) {
        console.error(err);
        outBox.textContent = 'تعذرت الترجمة. حاول مرة أخرى.';
        setStatus('فشل في ترجمة النص. تحقق من الإنترنت.', 'error');
      } finally {
        setBusy(false);
      }
    }

    async function readImage(event) {
      const file = event.target.files?.[0];
      if (!file) return;

      const lang = TESSERACT_LANG[fromSelect.value] || 'eng';

      setBusy(true);
      setStatus('جارٍ قراءة الصورة...', 'info');
      outBox.textContent = '⏳ جاري التعرف على النص...';

      try {
        const result = await Tesseract.recognize(file, lang, {
          logger: (m) => {
            if (m && m.status === 'recognizing text') {
              const percent = Math.round(m.progress * 100);
              setStatus(`جارٍ التعرف على النص: ${percent}%`, 'info');
            }
          }
        });

        const text = result?.data?.text?.trim();
        if (!text) {
          throw new Error('لا يوجد نص في الصورة.');
        }

        setInputText(text);
        setStatus('تم استخراج النص من الصورة بنجاح.', 'success');
      } catch (err) {
        console.error(err);
        setStatus('فشل في قراءة الصورة أو التعرف على النص.', 'error');
        outBox.textContent = 'تعذرت قراءة الصورة.';
      } finally {
        setBusy(false);
      }
    }

    async function copyOutput() {
      const value = outBox.textContent.trim();

      if (!value || value === 'الترجمة هنا') {
        setStatus('لا يوجد نص للنسخ.', 'error');
        return;
      }

      try {
        await navigator.clipboard.writeText(value);
        setStatus('تم نسخ النص.', 'success');
      } catch (err) {
        setStatus('تعذّر نسخ النص.', 'error');
      }
    }

    function speakOutput() {
      const value = outBox.textContent.trim();

      if (!value || value === 'الترجمة هنا') {
        setStatus('لا يوجد نص للنطق.', 'error');
        return;
      }

      if (!('speechSynthesis' in window)) {
        setStatus('المتصفح لا يدعم النطق.', 'error');
        return;
      }

      const utterance = new SpeechSynthesisUtterance(value);
      const target = toSelect.value;

      if (target === 'ar') utterance.lang = 'ar-SA';
      else if (target === 'en') utterance.lang = 'en-US';
      else if (target === 'fr') utterance.lang = 'fr-FR';
      else if (target === 'es') utterance.lang = 'es-ES';
      else if (target === 'tr') utterance.lang = 'tr-TR';
      else if (target === 'hi') utterance.lang = 'hi-IN';
      else if (target === 'zh') utterance.lang = 'zh-CN';
      else utterance.lang = 'en-US';

      utterance.onstart = () => setStatus('جارٍ نطق النص...', 'info');
      utterance.onend = () => setStatus('تم النطق.', 'success');
      utterance.onerror = () => setStatus('تعذّر نطق النص.', 'error');

      window.speechSynthesis.cancel();
      window.speechSynthesis.speak(utterance);
    }

    txtInput.addEventListener('input', () => {
      const value = txtInput.value;
      inBox.textContent = value || 'النص هنا';
    });

    fromSelect.addEventListener('change', () => {
      if (fromSelect.value === 'ar') {
        toSelect.value = 'fr';
      }
    });
  </script>
</body>
</html>
