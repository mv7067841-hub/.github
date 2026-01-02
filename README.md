[1/1, 12:35 PM] ༺ℝ𝕆𝕐𝔸𝕃࿐Boy𒈞: <!DOCTYPE html>
<html lang="hi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI Storyboard Director (Final Stable)</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css">
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; background-color: #111827; }
        .main-card { background-color: #1f2937; border: 1px solid #374151; }
        .loader { border: 4px solid #4b5563; border-radius: 50%; border-top: 4px solid #8b5cf6; width: 30px; height: 30px; animation: spin 1.2s linear infinite; }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
        .prompt-list-item { background-color: #2d3748; border-left: 4px solid #4f46e5;}
        .disabled-button { opacity: 0.5; cursor: not-allowed; }
    </style>
</head>
<body class="text-gray-200">

    <div class="container mx-auto p-4 lg:p-8">
        <header class="text-center mb-8">
            <h1 class="text-4xl md:text-5xl font-extrabold text-transparent bg-clip-text bg-gradient-to-r from-violet-400 to-purple-500">AI Storyboard Director</h1>
            <p class="text-gray-400 mt-2 max-w-3xl mx-auto">Kahani likhein, AI se professional prompts banwayein, aur unhein tasveeron mein badlein.</p>
        </header>

        <div class="grid grid-cols-1 lg:grid-cols-2 gap-8">
            <!-- Left Column: Controls -->
            <div class="space-y-8">
                <!-- Step 1: Story Input -->
                <div class="main-card p-6 rounded-xl shadow-lg">
                     <h2 class="text-2xl font-bold text-white mb-4 flex items-center"><span class="bg-violet-600 text-white w-8 h-8 rounded-full flex items-center justify-center font-bold text-lg mr-3">1</span>Apni Kahani Likhein</h2>
                     <textarea id="story-input" rows="12" class="w-full bg-gray-800 border border-gray-600 rounded-lg p-3 text-white" placeholder="Yahan apni poori kahani likhein..."></textarea>
                     <button id="prepare-prompts-btn" class="mt-4 w-full bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-3 px-4 rounded-lg transition-colors flex items-center justify-center text-lg">
                        <i class="fas fa-robot mr-2"></i>Prepare Storyboard
                    </button>
                </div>
                
                <!-- Step 2: Prompt Review -->
                <div class="main-card p-6 rounded-xl shadow-lg">
                     <h2 class="text-2xl font-bold text-white mb-4 flex items-center"><span class="bg-violet-600 text-white w-8 h-8 rounded-full flex items-center justify-center font-bold text-lg mr-3">2</span>Prompts Ko Final Karein</h2>
                     <div id="prompt-list-container" class="hidden space-y-4 max-h-[28rem] overflow-y-auto p-1 pr-2"></div>
                     <p id="prompt-placeholder" class="text-center text-gray-500 py-8">Aapke prompts yahan dikhenge...</p>
                     <div id="generate-buttons-container" class="mt-4">
                        <button id="generate-all-btn" class="w-full bg-green-600 hover:bg-green-700 text-white font-bold py-3 px-4 rounded-lg transition-colors flex items-center justify-center text-lg disabled-button">
                            <i class="fas fa-play mr-2"></i>Sabhi Images Generate Karein
                        </button>
                    </div>
                </div>
            </div>

            <!-- Right Column: Gallery -->
            <div class="bg-gray-900 p-4 rounded-xl h-[90vh] lg:h-auto overflow-y-auto">
                <div id="gallery-status" class="h-full flex flex-col items-center justify-center text-gray-500 text-center">
                     <i class="fas fa-film text-6xl mb-4"></i>
                    <h2 class="text-2xl font-semibold">Aapki Kahani Ki Tasveerein</h2>
                </div>
                <div id="scene-loader" class="hidden flex-col items-center justify-center text-center">
                    <div class="loader"></div>
                    <p id="scene-loader-text" class="text-gray-400 mt-4 font-medium">Scenes ban rahe hain...</p>
                </div>
                <div id="image-gallery" class="hidden grid grid-cols-1 md:grid-cols-2 gap-6"></div>
            </div>
        </div>
        <div id="message-box" class="hidden fixed bottom-5 right-5 bg-red-600 text-white py-3 px-5 rounded-lg shadow-xl z-50"><p id="message-text"></p></div>
    </div>

    <script>
        // --- STATE & UI ELEMENTS ---
        let isGenerationCancelled = false;

        const ui = {
            storyInput: document.getElementById('story-input'),
            preparePromptsBtn: document.getElementById('prepare-prompts-btn'),
            promptListContainer: document.getElementById('prompt-list-container'),
            promptPlaceholder: document.getElementById('prompt-placeholder'),
            generateButtonsContainer: document.getElementById('generate-buttons-container'),
            galleryStatus: document.getElementById('gallery-status'),
            sceneLoader: document.getElementById('scene-loader'),
            sceneLoaderText: document.getElementById('scene-loader-text'),
            imageGallery: document.getElementById('image-gallery'),
            messageBox: document.getElementById('message-box'),
            messageText: document.getElementById('message-text'),
        };

        // --- HELPER FUNCTIONS ---
        function showMessage(message, isError = true) {
            ui.messageText.textContent = message;
            ui.messageBox.className = `fixed bottom-5 right-5 text-white py-3 px-5 rounded-lg shadow-xl z-50 ${isError ? 'bg-red-600' : 'bg-green-600'}`;
            ui.messageBox.classList.remove('hidden');
            setTimeout(() => ui.messageBox.classList.add('hidden'), 5000);
        }

        // --- API CALLS ---
        async function callGeminiAPI(prompt, isJson = false) {
            const payload = { 
                contents: [{ role: "user", parts: [{ text: prompt }] }],
                ...(isJson && { generationConfig: { responseMimeType: "application/json" } })
            };
            const apiKey = "";
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${apiKey}`;
            const response = await fetch(apiUrl, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(payload) });
            if (!response.ok) throw new Error(`Gemini API Error: ${response.status}`);
            const result = await response.json();
            const text = result.candidates[0].content.parts[0].text;
            return isJson ? JSON.parse(text) : text;
        }

        async function generateImageWithImagen(prompt) {
            const payload = { instances: [{ prompt }], parameters: { "sampleCount": 1, "aspectRatio": "16:9" } };
            const apiKey = "";
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/imagen-3.0-generate-002:predict?key=${apiKey}`;
            const response = await fetch(apiUrl, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(payload) });
            if (!response.ok) throw new Error(`Image API Error (${response.status})`);
            const result = await response.json();
            if (result.predictions && result.predictions[0].bytesBase64Encoded) return `data:image/png;base64,${result.predictions[0].bytesBase64Encoded}`;
            throw new Error('Image data not found. Prompt was likely blocked.');
        }

        // --- MAIN LOGIC ---
        ui.preparePromptsBtn.addEventListener('click', async () => {
            const storyText = ui.storyInput.value.trim();
            if (!storyText) return showMessage('Kripya pehle kahani likhein.');

            ui.preparePromptsBtn.disabled = true;
            ui.preparePromptsBtn.innerHTML = `<div class="loader"></div><span class="ml-2">Analyzing Story...</span>`;

            const systemPrompt = `You are an expert prompt engineer for an AI image generator. Your task is to convert a story into a series of safe, descriptive, and effective prompts.
            1. First, identify all unique characters in the story.
            2. For each character, create a detailed, consistent visual description based on their name and context (assume Mughal-era India unless specified otherwise).
            3. Then, for each line in the story, create a highly descriptive, cinematic, and safe image prompt in English.
            4. This prompt must include the detailed descriptions of any characters mentioned in that line.
            5. Rephrase actions safely (e.g., 'attack' becomes 'confronts', 'death' becomes 'defeat'). Focus on visual details like camera angles, lighting, and expressions instead of graphic actions.
            6. Return a JSON object with a single key "prompts", which is an array of objects, each with "original_scene" and "image_prompt" keys.`;
            
            try {
                const result = await callGeminiAPI(systemPrompt + "\n\nStory:\n" + storyText, true);
                ui.promptListContainer.innerHTML = '';
                result.prompts.forEach((p, index) => {
                    const promptItem = document.createElement('div');
                    promptItem.className = 'prompt-list-item p-3 rounded-lg';
                    promptItem.innerHTML = `
                        <p class="text-xs text-gray-400 mb-1">Original Scene ${index + 1}: ${p.original_scene}</p>
                        <textarea class="w-full bg-gray-700 text-sm p-2 rounded border border-gray-600" rows="4">${p.image_prompt}</textarea>
                    `;
                    ui.promptListContainer.appendChild(promptItem);
                });
                ui.promptPlaceholder.classList.add('hidden');
                ui.promptListContainer.classList.remove('hidden');
                document.getElementById('generate-all-btn').classList.remove('disabled-button');
                showMessage('Prompts taiyaar hain! Aap unhein check karke badal sakte hain.', false);
            } catch (error) {
                showMessage(error.message || 'Prompts banane mein samasya aayi.');
            } finally {
                ui.preparePromptsBtn.disabled = false;
                ui.preparePromptsBtn.innerHTML = `<i class="fas fa-robot mr-2"></i>Prepare Storyboard`;
            }
        });
        
        function resetGenerateButton() {
            const btnHtml = `<button id="generate-all-btn" class="w-full bg-green-600 hover:bg-green-700 text-white font-bold py-3 px-4 rounded-lg transition-colors flex items-center justify-center text-lg"><i class="fas fa-play mr-2"></i>Sabhi Images Generate Karein</button>`;
            ui.generateButtonsContainer.innerHTML = btnHtml;
            document.getElementById('generate-all-btn').addEventListener('click', generateScenes);
        }

        function showCancelButton() {
            const btnHtml = `<button id="cancel-btn" class="w-full bg-red-600 hover:bg-red-700 text-white font-bold py-3 px-4 rounded-lg shadow-md transition-all duration-300 flex items-center justify-center"><i class="fas fa-times mr-2"></i>Cancel Karein</button>`;
            ui.generateButtonsContainer.innerHTML = btnHtml;
            document.getElementById('cancel-btn').addEventListener('click', () => { isGenerationCancelled = true; });
        }

        async function generateScenes() {
            const promptTextareas = ui.promptListContainer.querySelectorAll('textarea');
            if (promptTextareas.length === 0) return showMessage('Generate karne ke liye koi prompt nahi hai.');
            const prompts = Array.from(promptTextareas).map(t => t.value.trim());

            isGenerationCancelled = false;
            ui.galleryStatus.classList.add('hidden');
            ui.sceneLoader.classList.remove('hidden');
            ui.imageGallery.innerHTML = '';
            ui.imageGallery.classList.add('hidden');
            showCancelButton();

            let sceneCount = 0;
            for (const prompt of prompts) {
                if(isGenerationCancelled) { showMessage("Generation rok di gayi.", false); break; }
                sceneCount++;
                ui.sceneLoaderText.textContent = `Scene ${sceneCount} of ${prompts.length} ban raha hai...`;
                
                try {
                    const imageUrl = await generateImageWithImagen(prompt);
                    if (!ui.sceneLoader.classList.contains('hidden')) {
                        ui.imageGallery.classList.remove('hidden');
                        ui.sceneLoader.classList.add('hidden');
                    }
                    const card = document.createElement('div');
                    card.className = 'bg-gray-800 rounded-lg overflow-hidden shadow-lg group';
                    card.innerHTML = `<div class="relative"><img src="${imageUrl}" class="w-full h-auto object-cover"><div class="absolute inset-0 bg-black bg-opacity-50 flex items-center justify-center opacity-0 group-hover:opacity-100 transition-opacity"><a href="${imageUrl}" download="scene-${sceneCount}.png" class="bg-indigo-600 text-white font-bold py-2 px-4 rounded-lg"><i class="fas fa-download mr-2"></i>Download</a></div></div>`;
                    ui.imageGallery.appendChild(card);
                } catch (error) {
                    const errorCard = document.createElement('div');
                    errorCard.className = 'bg-gray-800 rounded-lg p-4 text-center';
                    errorCard.innerHTML = `<p class="text-red-400 font-semibold">Scene ${sceneCount} nahi ban saka.</p><p class="text-xs text-gray-500">${error.message}</p>`;
                    ui.imageGallery.appendChild(errorCard);
                }
            }
            
            if (ui.imageGallery.children.length === 0 && !isGenerationCancelled) {
                ui.galleryStatus.classList.remove('hidden');
                ui.galleryStatus.innerHTML = `<i class="fas fa-exclamation-triangle text-6xl mb-4 text-red-500"></i><h2 class="text-2xl font-semibold">Koi Scene Nahi Bana</h2><p>Kripya dobara koshish karein.</p>`;
            }
            ui.sceneLoader.classList.add('hidden');
            resetGenerateButton();
        }

        // --- INITIALIZATION ---
        function initialize() {
            ui.storyInput.value = `Ek din Akbar बादशाह दरबार में बैठे थे, तभी एक रहस्यमयी साधु दरबार में आया। उसकी आँखों में अजीब-सी चमक थी, और हाथ में एक काले रंग का बंद डिब्बा।
साधु बोला – "जहाँपनाह, इस डिब्बे में वह चीज़ है, जो आपके पूरे साम्राज्य को हिला सकती है… पर इसे खोलने की हिम्मत सिर्फ़ सच्चे ज्ञानी में है।"
अकबर ने बीरबल की तरफ देखा – "बीरबल, आज तुम्हारी बुद्धि की सबसे बड़ी परीक्षा है।"
बीरबल मुस्कुराए और बोले – "जहाँपनाह, बीरबल की बुद्धि किसी से कम नहीं।"
उन्होंने डिब्बा हाथ में लिया… पर जैसे ही खोला, उसमें से एक रहस्यमयी धुंध निकली और पूरा दरबार अंधकार में डूब गया।
साधु की आवाज गूँजी – "यदि बीरबल असली ज्ञानी हैं तो इस रहस्य को सुलझाएँ, वरना आज ही हार मान लें।"
बीरबल ने कई तरकीबें आजमाईं – तर्क, हँसी-मजाक, चतुराई… पर धुंध और गाढ़ी होती गई।
दरबारियों ने पहली बार देखा कि बीरबल चुप थे, जैसे उनके पास कोई जवाब ही न हो।
साधु मुस्कुराया – "तो महान बीरबल भी हार सकते हैं? लगता है तुम्हारी बुद्धि का अंत यहीं है।"
अकबर ने हैरानी से बीरबल को देखा, और बीरबल ने पहली बार सिर झुका लिया।
पर तभी… धुंध के बीच एक सोने जैसा चमकता तोता उड़ता हुआ आया और डिब्बे पर बैठ गया।
डिब्बा अचानक खुद-ब-खुद बंद हो गया। साधु चौंका – "ये कैसे संभव है? यह तोते को कौन बुला सकता है?"
बीरबल ने हैरानी से तोते को देखा, जैसे उसे पहचानते हों… पर कुछ समझ नहीं पाए।
साधु अचानक बोला – "अगर तुम इस तोते का राज़ नहीं समझ पाए, तो बीरबल… तुम्हारी असली हार यहीं से शुरू होती है।"
धुंध छंटते ही साधु गायब हो गया, और पीछे सिर्फ़ वह सोने का तोता रह गया, जो बीरबल को घूर रहा था… जैसे कोई रहस्य कह रहा हो।`;
            resetGenerateButton();
        }

        initialize();
    </script>
</body>
</html>
[1/1, 12:37 PM] ༺ℝ𝕆𝕐𝔸𝕃࿐Boy𒈞: Mujhe is app ki puri design ek bar mein aur saral tarike se aur acche tarike se pura design Karke do sem ISI tarike se
