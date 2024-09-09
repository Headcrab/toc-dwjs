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
const modeSelectContainer = controlsContainer.createEl('div', { cls: 'toc-select-container' });
modeSelectContainer.createEl('label', { text: 'Режим:', for: 'mode-select' });
const modeSelect = modeSelectContainer.createEl('select', { id: 'mode-select' });
['По папкам', 'По файлам', 'Текущий файл'].forEach(mode => {
    modeSelect.createEl('option', { value: mode, text: mode });
});

// Создаем контейнер для выбора папки
const folderSelectContainer = controlsContainer.createEl('div', { cls: 'toc-select-container hidden' });
const folderInput = createAutocompleteInput(folderSelectContainer, 'Выберите папку:', 'folder-select', 'Введите путь к папке', getFolders);

// Создаем контейнер для выбора файлов
const fileSelectContainer = controlsContainer.createEl('div', { cls: 'toc-select-container hidden' });
const fileInput = createAutocompleteInput(fileSelectContainer, 'Выберите файлы:', 'file-select', 'Введите пути к файлам через запятую', getFiles);

// Функция для получения списка файлов
function getFiles() {
    return app.vault.getMarkdownFiles().map(f => f.path);
}

// Создаем контейнер для отображения текущего файла
const currentFileContainer = controlsContainer.createEl('div', { cls: 'toc-select-container hidden' });
currentFileContainer.createEl('label', { text: 'Текущий файл:', for: 'current-file' });
const currentFileDisplay = currentFileContainer.createEl('div', { cls: 'current-file-display', id: 'current-file' });

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
        max-width: 100%;
        margin: 0 auto;
        padding: 24px;
        background-color: var(--background-primary);
        border-radius: 12px;
        box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
    }
    .toc-controls {
        display: flex;
        flex-wrap: wrap;
        gap: 16px;
        margin-bottom: 24px;
        align-items: flex-end;
    }
    .toc-select-container {
        flex: 1;
        min-width: 200px;
    }
    .toc-select-container label {
        display: block;
        margin-bottom: 8px;
        color: var(--text-muted);
        font-size: 14px;
    }
    .toc-select-container select,
    .toc-select-container input[type="text"] {
        width: 100%;
        padding: 10px 12px;
        border-radius: 8px;
        background-color: var(--background-secondary);
        color: var(--text-normal);
        border: 1px solid var(--background-modifier-border);
        height: 40px;
        font-size: 14px;
        transition: all 0.3s ease;
    }
    .toc-select-container select:focus,
    .toc-select-container input[type="text"]:focus {
        outline: none;
        border-color: var(--interactive-accent);
        box-shadow: 0 0 0 2px var(--interactive-accent-hover);
    }
    .toc-generate-button {
        padding: 10px 16px;
        height: 40px;
        font-size: 14px;
        background-color: var(--interactive-accent);
        color: var(--text-on-accent);
        border: none;
        border-radius: 8px;
        cursor: pointer;
        transition: background-color 0.3s ease;
        align-self: flex-end;
    }
    .toc-generate-button:hover {
        background-color: var(--interactive-accent-hover);
    }
    .toc-output {
        background-color: var(--background-secondary);
        border-radius: 8px;
        padding: 16px;
        color: var(--text-normal);
        max-height: 600px;
        overflow-y: auto;
    }
    .toc-output p {
        margin: 0;
        padding: 0;
    }
    .toc-output .file-link {
        font-weight: bold;
        margin-top: 12px;
    }
    .toc-output .header-link {
        display: block;
        padding: 4px 0;
    }
    .toc-output .header-level-1 { margin-left: 0; }
    .toc-output .header-level-2 { margin-left: 16px; }
    .toc-output .header-level-3 { margin-left: 32px; }
    .toc-output .header-level-4 { margin-left: 48px; }
    .toc-output .header-level-5 { margin-left: 64px; }
    .toc-output .header-level-6 { margin-left: 80px; }
    .toc-output a {
        color: var(--text-accent);
        text-decoration: none;
        transition: color 0.2s ease;
    }
    .toc-output a:hover {
        color: var(--text-accent-hover);
    }
    .file-content {
        margin-left: 16px;
    }
    .toggle-button {
        background: none;
        border: none;
        color: var(--text-accent);
        cursor: pointer;
        font-size: 16px;
        padding: 0 8px;
        transition: color 0.2s ease;
    }
    .toggle-button:hover {
        color: var(--text-accent-hover);
    }
    .file-header {
        display: flex;
        align-items: center;
        margin-bottom: 8px;
    }
    .file-header a {
        flex-grow: 1;
    }
    .broken-link {
        color: var(--text-error);
        text-decoration: line-through;
    }
    .toc-progress-container {
        margin-top: 16px;
        background-color: var(--background-modifier-border);
        border-radius: 8px;
        padding: 4px;
        display: none;
    }
    .toc-progress-bar {
        height: 8px;
        background-color: var(--interactive-accent);
        border-radius: 4px;
        width: 0%;
        transition: width 0.3s ease-in-out;
    }
    .toc-progress-text {
        text-align: center;
        margin-top: 8px;
        color: var(--text-muted);
        font-size: 12px;
    }
    .hidden {
        display: none;
    }
    .current-file-display {
        width: 100%;
        padding: 10px 12px;
        border-radius: 8px;
        background-color: var(--background-secondary);
        color: var(--text-normal);
        border: 1px solid var(--background-modifier-border);
        height: 40px;
        line-height: 20px;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
        font-size: 14px;
    }
    .autocomplete-container {
        position: relative;
    }
    .autocomplete-dropdown {
        position: absolute;
        top: 100%;
        left: 0;
        right: 0;
        background-color: var(--background-secondary);
        border: 1px solid var(--background-modifier-border);
        border-top: none;
        max-height: 200px;
        overflow-y: auto;
        z-index: 1000;
        border-radius: 0 0 8px 8px;
    }
    .autocomplete-dropdown li {
        padding: 8px 12px;
        cursor: pointer;
        transition: background-color 0.2s ease;
    }
    .autocomplete-dropdown li:hover {
        background-color: var(--background-modifier-hover);
    }
    .autocomplete-active {
        background-color: var(--interactive-accent) !important;
        color: var(--text-on-accent);
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
    folderSelectContainer.classList.toggle('hidden', mode !== 'По папкам');
    fileSelectContainer.classList.toggle('hidden', mode !== 'По файлам');
    currentFileContainer.classList.toggle('hidden', mode !== 'Текущий файл');
    
    if (mode === 'Текущий файл') {
        const activeLeaf = app.workspace.getMostRecentLeaf();
        if (activeLeaf && activeLeaf.view && activeLeaf.view.getViewType() === 'markdown' && activeLeaf.view.file) {
            currentFileDisplay.textContent = activeLeaf.view.file.name;
        } else {
            currentFileDisplay.textContent = 'Нет активного Markdown-файла';
        }
        // Перемещаем currentFileContainer перед headerLevelContainer
        controlsContainer.insertBefore(currentFileContainer, headerLevelContainer);
    } else {
        // Возвращаем currentFileContainer в конец controlsContainer
        controlsContainer.appendChild(currentFileContainer);
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
                    currentFileDisplay.textContent = activeFile.name;
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

                for (const line of lines) {
                    if (line.trim().startsWith('```')) {
                        insideCodeBlock = !insideCodeBlock;
                        continue;
                    }
                    
                    if (insideCodeBlock) continue;
                    
                    if (line.trim() === '') continue;

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
                    }
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
            noResultsMessage.textContent = 'Не найдено подходящих заголовков в выбранной папке.';
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

function processInternalLinks() {
    const links = outputContainer.querySelectorAll('.internal-link');
    links.forEach(link => {
        const linkHref = link.getAttribute('data-href');
        if (linkHref) {
            const [filePath, heading] = linkHref.split('#');
            const file = app.vault.getAbstractFileByPath(filePath);
            if (file && typeof file.path === 'string') {
                link.dataset.linktext = linkHref;
            } else {
                link.classList.add('broken-link');
            }
        } else {
            link.classList.add('broken-link');
        }
    });
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
        generateButton.textContent = 'Остановить';
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

// Функция для создания строки ввода с автодополнением
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
        if (e.key === 'ArrowDown') {
            currentFocus++;
            addActive(dropdown.children);
            e.preventDefault();
        } else if (e.key === 'ArrowUp') {
            currentFocus--;
            addActive(dropdown.children);
            e.preventDefault();
        } else if (e.key === 'Enter') {
            e.preventDefault();
            if (currentFocus > -1) {
                if (dropdown.children[currentFocus]) {
                    dropdown.children[currentFocus].click();
                }
            }
        }
    });

    document.addEventListener('click', (e) => {
        if (!inputContainer.contains(e.target)) {
            dropdown.classList.add('hidden');
        }
    });

    function addActive(items) {
        if (!items) return false;
        removeActive(items);
        if (currentFocus >= items.length) currentFocus = 0;
        if (currentFocus < 0) currentFocus = (items.length - 1);
        items[currentFocus].classList.add('autocomplete-active');
    }

    function removeActive(items) {
        for (let i = 0; i < items.length; i++) {
            items[i].classList.remove('autocomplete-active');
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
            li.addEventListener('click', () => {
                input.value = item;
                dropdown.classList.add('hidden');
            });
        });
        dropdown.classList.remove('hidden');
    } else {
        dropdown.classList.add('hidden');
    }
}
```
