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

// Создаем контейнер для элементов управления
const controlsContainer = mainContainer.createEl('div', { cls: 'toc-controls' });

// Создаем выпадающий список для выбора режима
const modeSelectContainer = controlsContainer.createEl('div', { cls: 'toc-select-container mode-select' });
modeSelectContainer.createEl('label', { text: 'Режим:', for: 'mode-select' });
const modeSelect = modeSelectContainer.createEl('select', { id: 'mode-select' });
['По папкам', 'По файлам', 'Текущий файл'].forEach(mode => {
    modeSelect.createEl('option', { value: mode, text: mode });
});

// Создаем контейнер для выбора папки
const folderSelectContainer = controlsContainer.createEl('div', { cls: 'toc-select-container' });
const folderInput = createAutocompleteInput(folderSelectContainer, 'Выберите папку:', 'folder-select', 'Введите путь к папке', getFolders);

// Создаем контейнер для выбора файлов
const fileSelectContainer = controlsContainer.createEl('div', { cls: 'toc-select-container' });
const fileInput = createAutocompleteInput(fileSelectContainer, 'Выберите файлы:', 'file-select', 'Введите пути к файлам через запятую', getFiles);

// Функция для получения списка файлов
function getFiles() {
    return app.vault.getMarkdownFiles().map(f => f.path);
}

// Создаем контейнер для отображения текущего файла
const currentFileContainer = controlsContainer.createEl('div', { cls: 'toc-select-container' });
currentFileContainer.createEl('label', { text: 'Текущий файл:', for: 'current-file' });
const currentFileDisplay = currentFileContainer.createEl('div', { cls: 'current-file-display', id: 'current-file' });

// Создаем выпадающий список для выбора уровня заголовков
const headerLevelContainer = controlsContainer.createEl('div', { cls: 'toc-select-container header-level-select' });
headerLevelContainer.createEl('label', { text: 'Уровень заголовков:', for: 'header-level-select' });
const headerLevelSelect = headerLevelContainer.createEl('select', { id: 'header-level-select' });
for (let i = 1; i <= 6; i++) {
    headerLevelSelect.createEl('option', { value: i, text: `До ${i}` });
}

// Обновляем создание контейнеров для нумерованных и маркированных списков
const numberedListContainer = controlsContainer.createEl('div', { cls: 'toc-select-container list-container' });
const numberedListWrapper = numberedListContainer.createEl('div', { cls: 'checkbox-wrapper' });
const numberedListCheckbox = numberedListWrapper.createEl('input', { type: 'checkbox', id: 'numbered-list-checkbox' });
numberedListWrapper.createEl('label', { text: 'Нумерованные', for: 'numbered-list-checkbox' });
const numberedListLevelSelect = numberedListContainer.createEl('select', { id: 'numbered-list-level-select', cls: 'list-level-select' });
for (let i = 1; i <= 6; i++) {
    numberedListLevelSelect.createEl('option', { value: i, text: `До ${i}` });
}

const bulletListContainer = controlsContainer.createEl('div', { cls: 'toc-select-container list-container' });
const bulletListWrapper = bulletListContainer.createEl('div', { cls: 'checkbox-wrapper' });
const bulletListCheckbox = bulletListWrapper.createEl('input', { type: 'checkbox', id: 'bullet-list-checkbox' });
bulletListWrapper.createEl('label', { text: 'Маркированные', for: 'bullet-list-checkbox' });
const bulletListLevelSelect = bulletListContainer.createEl('select', { id: 'bullet-list-level-select', cls: 'list-level-select' });
for (let i = 1; i <= 6; i++) {
    bulletListLevelSelect.createEl('option', { value: i, text: `До ${i}` });
}

// Обновляем обработчики событий для чекбоксов и инициализируем видимость выпадающих списков
numberedListCheckbox.addEventListener('change', () => {
    numberedListLevelSelect.style.display = numberedListCheckbox.checked ? 'inline-block' : 'none';
});

bulletListCheckbox.addEventListener('change', () => {
    bulletListLevelSelect.style.display = bulletListCheckbox.checked ? 'inline-block' : 'none';
});

// Инициализация видимости выпадающих списков
numberedListLevelSelect.style.display = 'none';
bulletListLevelSelect.style.display = 'none';

// Создаем кнопку генерации
const generateButton = controlsContainer.createEl('button', { cls: 'toc-generate-button', text: 'Сгенерировать' });

// Создаем контейнер для вывода результата
const outputContainer = mainContainer.createEl('div', { cls: 'toc-output' });

// Обновляем стили
const style = container.createEl('style');
style.textContent = `
    .toc-main-container {
        max-width: 100%;
        margin: 0 auto;
        padding: 24px;
        background-color: var(--background-primary);
        border-radius: 12px;
        box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
    }
    .toc-controls {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
        gap: 16px;
        margin-bottom: 24px;
    }
    .toc-select-container {
        display: flex;
        flex-direction: column;
        gap: 8px;
        margin-bottom: 16px;
    }
    .toc-select-container label {
        font-size: 14px;
        font-weight: 600;
        color: var(--text-normal);
    }
    .toc-select-container select,
    .toc-select-container input[type="text"],
    .current-file-display,
    .autocomplete-container input[type="text"] {
        width: 100%;
        padding: 10px 12px;
        border-radius: 6px;
        background-color: var(--background-secondary);
        color: var(--text-normal);
        border: 1px solid var(--background-modifier-border);
        font-size: 14px;
        transition: all 0.3s ease;
        height: 38px; // Устанавливаем фиксированную высоту
        line-height: 18px; // Устанавливаем line-height для вертикального центрирования текста
    }
    .current-file-display {
        display: flex;
        align-items: center; // Вертикальное центрирование текста
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }
    .list-container {
        display: flex;
        flex-direction: column;
        gap: 8px;
    }
    .list-container .checkbox-wrapper {
        display: flex;
        align-items: center;
        gap: 8px;
    }
    .list-container .checkbox-wrapper label {
        font-size: 14px;
        font-weight: normal;
    }
    .list-level-select {
        width: 100%;
    }
    .toc-generate-button {
        grid-column: 1 / -1;
        justify-self: start;
        padding: 10px 20px;
        font-size: 16px;
        background-color: var(--interactive-accent);
        color: var(--text-on-accent);
        border: none;
        border-radius: 6px;
        cursor: pointer;
        transition: background-color 0.3s ease;
        font-weight: 600;
    }
    .toc-generate-button:hover {
        background-color: var(--interactive-accent-hover);
    }
    .toc-output {
        background-color: var(--background-secondary);
        border-radius: 8px;
        padding: 20px;
        color: var(--text-normal);
        max-height: 600px;
        overflow-y: auto;
        font-size: 14px;
        line-height: 1.6;
    }
    .toc-output .file-header {
        display: flex;
        align-items: center;
        margin-bottom: 12px;
    }
    .toc-output .toggle-button {
        background: none;
        border: none;
        color: var(--text-muted);
        cursor: pointer;
        font-size: 18px;
        padding: 0 8px;
        transition: color 0.2s ease;
    }
    .toc-output .toggle-button:hover {
        color: var(--text-normal);
    }
    .toc-output .file-link {
        font-weight: 600;
        font-size: 16px;
        color: var(--text-normal);
        text-decoration: none;
        transition: color 0.2s ease;
    }
    .toc-output .file-link:hover {
        color: var(--text-accent);
    }
    .toc-output .header-link {
        display: block;
        padding: 4px 0;
        color: var(--text-muted);
        text-decoration: none;
        transition: color 0.2s ease;
    }
    .toc-output .header-link:hover {
        color: var(--text-accent);
    }
    .toc-output .header-level-1 { margin-left: 0; font-weight: 600; }
    .toc-output .header-level-2 { margin-left: 16px; }
    .toc-output .header-level-3 { margin-left: 32px; }
    .toc-output .header-level-4 { margin-left: 48px; }
    .toc-output .header-level-5 { margin-left: 64px; }
    .toc-output .header-level-6 { margin-left: 80px; }
    .toc-progress-container {
        margin-top: 16px;
        background-color: var(--background-modifier-border);
        border-radius: 6px;
        overflow: hidden;
    }
    .toc-progress-bar {
        height: 8px;
        background-color: var(--interactive-accent);
        transition: width 0.3s ease-in-out;
    }
    .toc-progress-text {
        text-align: center;
        margin-top: 8px;
        color: var(--text-muted);
        font-size: 14px;
    }
    @media (max-width: 768px) {
        .toc-controls {
            grid-template-columns: 1fr;
        }
    }
    .autocomplete-container {
        position: relative;
        width: 100%;
    }

    .autocomplete-dropdown {
        position: absolute;
        width: 100%;
        max-height: 200px;
        overflow-y: auto;
        background-color: var(--background-primary);
        border: 1px solid var(--background-modifier-border);
        border-radius: 6px;
        box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        z-index: 1000;
        top: 100%; // Располагаем дропдаун под полем ввода
    }

    .autocomplete-dropdown li {
        padding: 8px 12px;
        cursor: pointer;
        transition: background-color 0.2s ease;
    }

    .autocomplete-dropdown li:hover,
    .autocomplete-dropdown li.autocomplete-active {
        background-color: var(--background-modifier-hover);
    }

    .hidden {
        display: none !important;
    }
`;

// Функция для очистки текста от тегов и лишних пробелов
function cleanText(text) {
    return text
        .replace(/#\s*/, '')
        .replace(/\[([^\]]+)\]\([^\)]+\)/g, '$1')
        .replace(/[*_`~]/g, '')
        .trim();
}

// Добавляем индикатор прогресса
const progressContainer = mainContainer.createEl('div', { cls: 'toc-progress-container' });
const progressBar = progressContainer.createEl('div', { cls: 'toc-progress-bar' });
const progressText = progressContainer.createEl('div', { cls: 'toc-progress-text' });

// Глобальная переменная для отслеживания состояния генерации
let isGenerating = false;

// Функция для обновления видимости элементов управления
function updateControlsVisibility() {
    const mode = modeSelect.value;
    folderSelectContainer.style.display = mode === 'По папкам' ? 'flex' : 'none';
    fileSelectContainer.style.display = mode === 'По файлам' ? 'flex' : 'none';
    currentFileContainer.style.display = mode === 'Текущий файл' ? 'flex' : 'none';
    
    if (mode === 'Текущий файл') {
        const activeLeaf = app.workspace.getMostRecentLeaf();
        if (activeLeaf && activeLeaf.view && activeLeaf.view.getViewType() === 'markdown' && activeLeaf.view.file) {
            currentFileDisplay.textContent = activeLeaf.view.file.path;
        } else {
            currentFileDisplay.textContent = 'Нет активного Markdown-файла';
        }
    }
}

// Обработчик изменения режима
modeSelect.addEventListener('change', updateControlsVisibility);

// Инициализация видимости элементов управления
updateControlsVisibility();

// Оптимизированная функция генерации оглавления
async function generateTOC() {
    try {
        if (isGenerating) {
            isGenerating = false;
            generateButton.textContent = 'Сгенерировать';
            progressText.textContent = 'Генерация прервана';
            return;
        }

        isGenerating = true;
        generateButton.textContent = 'Остановить';
        const mode = modeSelect.value;
        const maxHeaderLevel = parseInt(headerLevelSelect.value);
        const includeNumberedLists = numberedListCheckbox.checked;
        const maxNumberedListLevel = includeNumberedLists ? parseInt(numberedListLevelSelect.value) : 0;
        const includeBulletLists = bulletListCheckbox.checked;
        const maxBulletListLevel = includeBulletLists ? parseInt(bulletListLevelSelect.value) : 0;
        
        let files;
        switch (mode) {
            case 'По папкам':
                const folderPath = folderInput.value;
                files = app.vault.getFiles().filter(file => {
                    const isInFolder = file.path.startsWith(folderPath);
                    const isMarkdown = file.extension === 'md';
                    return isInFolder && isMarkdown;
                });
                break;
            case 'По файлам':
                const selectedFiles = fileInput.value.split(',').map(f => f.trim());
                files = selectedFiles.map(path => app.vault.getAbstractFileByPath(path)).filter(file => file && file.extension === 'md');
                break;
            case 'Текущий файл':
                const activeLeaf = app.workspace.getMostRecentLeaf();
                if (activeLeaf && activeLeaf.view && activeLeaf.view.getViewType() === 'markdown' && activeLeaf.view.file) {
                    const activeFile = activeLeaf.view.file;
                    files = [activeFile];
                    currentFileDisplay.textContent = activeFile.path;
                } else {
                    files = [];
                    currentFileDisplay.textContent = 'Нет активного Markdown-файла';
                    new Notice('На главной панели не открыт Markdown-файл');
                }
                break;
        }
        
        if (files.length === 0) {
            isGenerating = false;
            generateButton.textContent = 'Сгенерировать';
            new Notice('Не выбрано ни одного подходящего файла для генерации оглавления');
            return;
        }
        
        outputContainer.empty();
        
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
            try {
                const fileContent = [];
                let fileHasContent = false;

                const content = await app.vault.read(file);
                const lines = content.split('\n');
                let insideCodeBlock = false;
                let currentList = null;
                let currentListLevel = 0;

                for (const line of lines) {
                    if (line.trim().startsWith('```')) {
                        insideCodeBlock = !insideCodeBlock;
                        continue;
                    }
                    
                    if (insideCodeBlock) continue;
                    
                    if (line.trim() === '') {
                        if (currentList) {
                            fileContent.push(`</${currentList}>`);
                            currentList = null;
                            currentListLevel = 0;
                        }
                        continue;
                    }

                    if (line.match(/^#+\s/)) {
                        const level = line.match(/^#+/)[0].length;
                        if (level <= maxHeaderLevel) {
                            const headerText = line.replace(/^#+\s*/, '');
                            const cleanHeaderText = cleanText(headerText);
                            if (cleanHeaderText) {
                                const fullLink = `${file.path}#${headerText}`;
                                fileContent.push(`<p class="header-link header-level-${level}"><a class="internal-link" data-href="${fullLink}">${cleanHeaderText}</a></p>`);
                                fileHasContent = true;
                            }
                        }
                        if (currentList) {
                            fileContent.push(`</${currentList}>`);
                            currentList = null;
                            currentListLevel = 0;
                        }
                    } else if (line.match(/^(\s*)([-*+]|\d+\.)\s/)) {
                        const match = line.match(/^(\s*)([-*+]|\d+\.)\s/);
                        const indent = match[1];
                        const listType = match[2].match(/\d+\./) ? 'ol' : 'ul';
                        
                        // Считаем уровень вложенности на основе количества пробелов
                        const indentLevel = Math.floor(indent.length / 2) + 1;
                        
                        const maxAllowedLevel = listType === 'ol' ? maxNumberedListLevel : maxBulletListLevel;
                        
                        if (indentLevel <= maxAllowedLevel && 
                            ((listType === 'ol' && includeNumberedLists) || 
                             (listType === 'ul' && includeBulletLists))) {
                            
                            // Закрываем с списки более глубокого уровня
                            while (currentListLevel > indentLevel) {
                                fileContent.push(`</${currentList}>`);
                                currentListLevel--;
                            }
                            
                            // Открываем новые списки, если необходимо
                            if (currentListLevel < indentLevel) {
                                while (currentListLevel < indentLevel) {
                                    currentList = listType;
                                    currentListLevel++;
                                    fileContent.push(`<${listType} class="list-level-${currentListLevel}">`);
                                }
                            } else if (currentList !== listType) {
                                // Если тип списка изменился на том же уровне, зарываем старый и открываем новый
                                fileContent.push(`</${currentList}>`);
                                currentList = listType;
                                fileContent.push(`<${listType} class="list-level-${currentListLevel}">`);
                            }
                            
                            const listItemText = cleanText(line.replace(/^(\s*)([-*+]|\d+\.)\s/, ''));
                            if (listItemText) {
                                fileContent.push(`<li>${listItemText}</li>`);
                                fileHasContent = true;
                            }
                        }
                    } else {
                        // Закрываем все открытые списки при встрече не-спичного элемента
                        while (currentListLevel > 0) {
                            fileContent.push(`</${currentList}>`);
                            currentListLevel--;
                        }
                        currentList = null;
                    }
                }

                // В конце обработки файла
                while (currentListLevel > 0) {
                    fileContent.push(`</${currentList}>`);
                    currentListLevel--;
                }
                
                if (fileContent.length > 0) {
                    return {
                        path: file.path,
                        basename: file.basename,
                        content: fileContent.join('')
                    };
                } else {
                    return null;
                }
            } catch (error) {
                console.error('Error processing file:', file.path, error);
                return null;
            }
        };

        const batchSize = 10;
        const fragment = document.createDocumentFragment();

        for (let i = 0; i < totalFiles && isGenerating; i += batchSize) {
            const batch = files.slice(i, i + batchSize);
            const results = await Promise.all(batch.map(processFile));
            
            results.forEach(result => {
                if (result) {
                    const fileContainer = fragment.appendChild(document.createElement('div'));
                    const fileHeader = fileContainer.appendChild(document.createElement('div'));
                    fileHeader.className = 'file-header';
                    const toggleButton = fileHeader.appendChild(document.createElement('button'));
                    toggleButton.textContent = '▼';
                    toggleButton.className = 'toggle-button';
                    const fileLink = fileHeader.appendChild(document.createElement('a'));
                    fileLink.href = result.path;
                    fileLink.className = 'internal-link';
                    fileLink.textContent = result.basename;
                    const contentDiv = fileContainer.appendChild(document.createElement('div'));
                    contentDiv.className = 'file-content';
                    contentDiv.innerHTML = result.content;

                    toggleButton.addEventListener('click', () => {
                        contentDiv.style.display = contentDiv.style.display === 'none' ? 'block' : 'none';
                        toggleButton.textContent = contentDiv.style.display === 'none' ? '▶' : '▼';
                    });
                }
            });

            processedFiles += batch.length;
            updateProgress();
            
            outputContainer.appendChild(fragment);
            
            await new Promise(resolve => setTimeout(resolve, 0));
        }

        if (fragment.children.length === 0) {
            const noResultsMessage = fragment.appendChild(document.createElement('p'));
            noResultsMessage.textContent = 'Н найдено подходящих заголовков в выбранной папке.';
        }

        progressContainer.style.display = 'none';
        generateButton.textContent = 'Сгенерировать';
        isGenerating = false;

        processInternalLinks();
        addInternalLinkClickHandler();
        setupHoverPreview();

    } catch (error) {
        isGenerating = false;
        generateButton.textContent = 'Сгенерировать';
        progressText.textContent = 'Произошла ошибка';
        console.error('Ошибка при генерации оглавления:', error);
    }
}

function addInternalLinkClickHandler() {
    outputContainer.addEventListener('click', async (event) => {
        const link = event.target.closest('.internal-link');
        if (link) {
            event.preventDefault();
            const linktext = link.dataset.linktext;
            if (linktext) {
                const [filePath, heading] = linktext.split('#');
                const file = app.vault.getAbstractFileByPath(filePath);
                if (file && typeof file.path === 'string') {
                    const headingExists = await checkHeadingExistence(filePath, heading);
                    if (headingExists) {
                        app.workspace.getLeaf().openFile(file).then(() => {
                            if (heading) {
                                const headingElement = app.workspace.activeLeaf.view.containerEl.querySelector(`[data-heading="${heading}"]`);
                                if (headingElement) {
                                    headingElement.scrollIntoView({ behavior: 'smooth', block: 'start' });
                                }
                            }
                        });
                    } else {
                        new Notice(`Заголовок не найден: ${decodeURIComponent(heading)}`);
                    }
                } else {
                    new Notice(`Файл не найден или недействителен: ${filePath}`);
                }
            }
        }
    });
}

async function checkHeadingExistence(filePath, heading) {
    const file = app.vault.getAbstractFileByPath(filePath);
    if (file && typeof file.path === 'string') {
        const content = await app.vault.read(file);
        const lines = content.split('\n');
        const decodedHeading = decodeURIComponent(heading).replace(/^#*\s*/, '');
        const headingRegex = new RegExp(`^#+\\s*${decodedHeading.replace(/[-]/g, '[\\s-]')}`, 'i');
        return lines.some(line => headingRegex.test(line));
    }
    return false;
}

function setupHoverPreview() {
    const workspace = app.workspace;
    outputContainer.addEventListener('mouseover', (event) => {
        const target = event.target.closest('.internal-link');
        if (target && !target.classList.contains('broken-link')) {
            const linktext = target.dataset.linktext;
            if (linktext) {
                const [filePath, heading] = linktext.split('#');
                workspace.trigger('hover-link:hover-link', {
                    event,
                    source: 'preview',
                    hoverParent: outputContainer,
                    targetEl: target,
                    linktext: heading,
                    sourcePath: filePath
                });
            }
        }
    });
}

// Обработчик события для кнопки генерации
generateButton.addEventListener('click', async () => {
    if (!isGenerating) {
        generateButton.textContent = 'Осановить';
        try {
            await generateTOC();
        } catch (error) {
            isGenerating = false;
            generateButton.textContent = 'Сгенерировать';
            progressText.textContent = 'Произошла ошибка';
        }
    } else {
        isGenerating = false;
    }
});

// Обновляем функцию createAutocompleteInput
function createAutocompleteInput(container, labelText, id, placeholder, getItems) {
    container.createEl('label', { text: labelText, for: id });
    const inputContainer = container.createEl('div', { cls: 'autocomplete-container' });
    const input = inputContainer.createEl('input', { type: 'text', id: id, placeholder: placeholder });
    const dropdown = inputContainer.createEl('ul', { cls: 'autocomplete-dropdown hidden' });

    let currentFocus = -1;

    input.addEventListener('input', () => {
        const items = getItems().filter(item => item.toLowerCase().includes(input.value.toLowerCase()));
        updateDropdown(dropdown, items, input);
        currentFocus = -1;
    });

    input.addEventListener('focus', () => {
        const items = getItems();
        updateDropdown(dropdown, items, input);
    });

    input.addEventListener('keydown', (e) => {
        const items = dropdown.children;
        if (e.key === 'ArrowDown') {
            currentFocus = (currentFocus + 1) % items.length;
            addActive(items);
            e.preventDefault();
        } else if (e.key === 'ArrowUp') {
            currentFocus = (currentFocus - 1 + items.length) % items.length;
            addActive(items);
            e.preventDefault();
        } else if (e.key === 'Enter') {
            e.preventDefault();
            if (currentFocus > -1 && items[currentFocus]) {
                items[currentFocus].click();
            }
            hideDropdown(dropdown);
        }
    });

    document.addEventListener('click', (e) => {
        if (!inputContainer.contains(e.target)) {
            hideDropdown(dropdown);
        }
    });

    function addActive(items) {
        removeActive(items);
        if (items[currentFocus]) {
            items[currentFocus].classList.add('autocomplete-active');
            items[currentFocus].scrollIntoView({ block: 'nearest' });
        }
    }

    function removeActive(items) {
        for (let item of items) {
            item.classList.remove('autocomplete-active');
        }
    }

    return input;
}

function updateDropdown(dropdown, items, input) {
    dropdown.empty();
    if (items.length > 0) {
        items.forEach(item => {
            const li = dropdown.createEl('li');
            li.textContent = item;
            li.addEventListener('mousedown', (e) => {
                e.preventDefault();
                input.value = item;
                hideDropdown(dropdown);
            });
        });
        showDropdown(dropdown);
    } else {
        hideDropdown(dropdown);
    }
}

function showDropdown(dropdown) {
    dropdown.classList.remove('hidden');
}

function hideDropdown(dropdown) {
    dropdown.classList.add('hidden');
}
