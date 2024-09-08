---
language: Dataviewjs для Obsidian
---
# TOC

```dataviewjs
// Функция для получения списка папок
function getFolders() {
    return app.vault.getAllLoadedFiles()
        .filter(f => f.children)
        .map(f => f.path);
}

// Создаем элементы интерфейса
const container = this.container;
container.empty();

// Создаем основной контейнер
const mainContainer = container.createEl('div', { cls: 'toc-main-container' });

// Создаем заголовок
const header = mainContainer.createEl('h2', { text: 'Генератор оглавления', cls: 'toc-header' });

// Создаем контейнер для элементов управления
const controlsContainer = mainContainer.createEl('div', { cls: 'toc-controls' });

// Создаем выпадающий список для выбора папки
const folderSelectContainer = controlsContainer.createEl('div', { cls: 'toc-select-container' });
folderSelectContainer.createEl('label', { text: 'Выберите папку:', for: 'folder-select' });
const folderSelect = folderSelectContainer.createEl('select', { id: 'folder-select' });
getFolders().forEach(folder => {
    folderSelect.createEl('option', { value: folder, text: folder });
});

// Создаем выпадающий список для выбора уровня заголовков
const headerLevelContainer = controlsContainer.createEl('div', { cls: 'toc-select-container' });
headerLevelContainer.createEl('label', { text: 'Уровень заголовков:', for: 'header-level-select' });
const headerLevelSelect = headerLevelContainer.createEl('select', { id: 'header-level-select' });
for (let i = 1; i <= 6; i++) {
    headerLevelSelect.createEl('option', { value: i, text: `До уровня ${i}` });
}

// Создаем кнопку генерации
const generateButton = controlsContainer.createEl('button', { cls: 'toc-generate-button', text: 'Сгенерировать' });

// Создаем контейнер для вывода результата
const outputContainer = mainContainer.createEl('div', { cls: 'toc-output' });

// Изменяем стили
const style = container.createEl('style');
style.textContent = `
    .toc-main-container {
        max-width: 800px;
        margin: 0 auto;
        padding: 20px;
        background-color: #2c2c2c;
        border-radius: 8px;
        box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    }
    .toc-header {
        color: #e0e0e0;
        margin-bottom: 20px;
        text-align: center;
    }
    .toc-controls {
        display: flex;
        flex-wrap: wrap;
        gap: 15px;
        margin-bottom: 20px;
        align-items: flex-end;
    }
    .toc-select-container {
        flex: 1;
        min-width: 200px;
    }
    .toc-select-container label {
        display: block;
        margin-bottom: 5px;
        color: #b0b0b0;
    }
    .toc-select-container select {
        width: 100%;
        padding: 8px;
        border-radius: 4px;
        background-color: #3a3a3a;
        color: #e0e0e0;
        border: 1px solid #4a4a4a;
        height: 40px; /* Увеличиваем высоту выпадающих списков */
    }
    .toc-generate-button {
        padding: 6px 12px;
        height: 40px; /* Увеличиваем высоту кнопки для соответствия */
        font-size: 14px;
        background-color: #4a9eff;
        color: white;
        border: none;
        border-radius: 4px;
        cursor: pointer;
        transition: background-color 0.3s;
        align-self: flex-end;
    }
    .toc-output {
        background-color: #3a3a3a;
        border-radius: 4px;
        padding: 15px;
        color: #e0e0e0;
        max-height: 500px;
        overflow-y: auto;
    }
    .toc-output p {
        margin: 0;
        padding: 0;
    }
    .toc-output .file-link {
        font-weight: bold;
        margin-top: 10px;
    }
    .toc-output .header-link {
        display: block;
    }
    .toc-output .header-level-1 { margin-left: 0; }
    .toc-output .header-level-2 { margin-left: 20px; }
    .toc-output .header-level-3 { margin-left: 40px; }
    .toc-output .header-level-4 { margin-left: 60px; }
    .toc-output .header-level-5 { margin-left: 80px; }
    .toc-output .header-level-6 { margin-left: 100px; }
    .toc-output a {
        color: #4a9eff;
        text-decoration: none;
    }
    .toc-output a:hover {
        text-decoration: underline;
    }
`;

// Функция для очистки текста от тегов и лишних пробелов
function cleanText(text) {
    return text.replace(/#\S+/g, '').trim();
}

// Добавляем индикатор прогресса
const progressContainer = mainContainer.createEl('div', { cls: 'toc-progress-container' });
const progressBar = progressContainer.createEl('div', { cls: 'toc-progress-bar' });
const progressText = progressContainer.createEl('div', { cls: 'toc-progress-text' });

// Добавляем стили для индикатора прогресса
style.textContent += `
    .toc-progress-container {
        margin-top: 10px;
        background-color: #444;
        border-radius: 4px;
        padding: 3px;
        display: none;
    }
    .toc-progress-bar {
        height: 20px;
        background-color: #4a9eff;
        border-radius: 2px;
        width: 0%;
        transition: width 0.3s ease-in-out;
    }
    .toc-progress-text {
        text-align: center;
        margin-top: 5px;
        color: #e0e0e0;
    }
`;

// Глобальная переменная для отслеживания состояния генерации
let isGenerating = false;

// Оптимизированная функция генерации оглавления
async function generateTOC() {
    if (isGenerating) {
        isGenerating = false;
        generateButton.textContent = 'Сгенерировать';
        progressText.textContent = 'Генерация прервана';
        return;
    }

    isGenerating = true;
    generateButton.textContent = 'Остановить';
    const folderPath = folderSelect.value;
    const maxHeaderLevel = parseInt(headerLevelSelect.value);
    outputContainer.empty();
    const files = app.vault.getFiles().filter(file => file.path.startsWith(folderPath) && file.extension === 'md');
    
    progressContainer.style.display = 'block';
    progressBar.style.width = '0%';
    progressText.textContent = 'Подготовка...';

    const totalFiles = files.length;
    let processedFiles = 0;

    const updateProgress = () => {
        const progress = (processedFiles / totalFiles) * 100;
        progressBar.style.width = `${progress}%`;
        progressText.textContent = `Обработано ${processedFiles} из ${totalFiles} файлов`;
    };

    const processFile = async (file) => {
        if (!isGenerating) return null;

        const content = await app.vault.read(file);
        const lines = content.split("\n");
        
        let fileHasContent = false;
        let fileContent = '';
        
        let isFirstNonEmptyLine = true;
        let insideCodeBlock = false;
        
        for (const line of lines) {
            if (!isGenerating) return null;

            if (line.trim().startsWith('```')) {
                insideCodeBlock = !insideCodeBlock;
                continue;
            }
            
            if (insideCodeBlock) continue;
            
            if (line.trim() === '') continue;
            
            if (line.startsWith('#')) {
                const level = line.match(/^#+/)[0].length;
                if (level <= maxHeaderLevel) {
                    const headerText = line.replace(/^#+\s*/, '');
                    const cleanHeaderText = cleanText(headerText);
                    if (cleanHeaderText && !isFirstNonEmptyLine) {
                        fileContent += `<p class="header-link header-level-${level}"><a href="${file.path}#${headerText}" class="internal-link">${cleanHeaderText}</a></p>`;
                        fileHasContent = true;
                    }
                }
            }
            
            isFirstNonEmptyLine = false;
        }
        
        if (fileHasContent) {
            return {
                path: file.path,
                basename: file.basename,
                content: fileContent
            };
        }

        return null;
    };

    const batchSize = 10; // Размер пакета файлов для обработки
    const fragment = document.createDocumentFragment();

    for (let i = 0; i < totalFiles && isGenerating; i += batchSize) {
        const batch = files.slice(i, i + batchSize);
        const results = await Promise.all(batch.map(processFile));
        
        results.forEach(result => {
            if (result) {
                const fileContainer = fragment.appendChild(document.createElement('div'));
                const fileLink = fileContainer.appendChild(document.createElement('p'));
                fileLink.className = 'file-link';
                fileLink.innerHTML = `<a href="${result.path}" class="internal-link">${result.basename}</a>`;
                const contentDiv = fileContainer.appendChild(document.createElement('div'));
                contentDiv.innerHTML = result.content;
            }
        });

        processedFiles += batch.length;
        updateProgress();
        
        // Вставляем накопленные изменения в DOM
        outputContainer.appendChild(fragment);
        
        // Даем возможность обновиться интерфейсу
        await new Promise(resolve => setTimeout(resolve, 0));
    }

    progressContainer.style.display = 'none';
    generateButton.textContent = 'Сгенерировать';
    isGenerating = false;
}

// Обработчик события для кнопки генерации
generateButton.addEventListener('click', async () => {
    if (!isGenerating) {
        generateButton.textContent = 'Остановить';
    }
    await generateTOC();
});
```
