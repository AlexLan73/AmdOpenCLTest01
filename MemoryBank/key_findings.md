# 🧠 Key Findings - MemoryBank

## 📊 Ключевые результаты исследований

### **FFT Optimization (2024)**
- **Среднее ускорение**: 91.87x vs cuFFT
- **FFT16**: 318x speedup
- **FFT32**: 220x speedup
- **Пиковая производительность**: 2.23 TFLOP/s при 262k-524k окон

### **GPU Occupancy Analysis**
- **Оптимальный batch size**: 8k-64k окон
- **Пик производительности**: 262k-524k окон
- **Критический фактор**: Memory bandwidth для больших batch'ей

### **Correlation Architecture**
- **Рекомендуемый подход**: BATCHED MODE
- **Ускорение**: 38.5x vs последовательный подход
- **Kernel launches**: 2 вместо 40
- **Memory efficiency**: Предварительно вычисленные FFT опор

### **Sliding FFT16 Analysis (2025-10-14)**
- **Лучший подход**: одно батч-ядро «gather → FFT16 → fftshift → запись B» за один проход
- **Не запускать 4 смещённых потока**: усложняют индексацию, профит минимален для N=16
- **Индексация**: s=2·w; x[t]=(s+t<S)?X[L][s+t]:0; FFT16(x)=y; yshift[k]=y[(k+8) mod 16]; B[L][w][k]=yshift[k]
- **Для N≤64 фьюзинг быстрее cuFFT** и схемы с несколькими ядрами

## 🎯 Архитектурные принципы

### **1. Batch Processing**
- Всегда предпочитать batched операции
- Минимизировать количество kernel launch'ов
- Использовать оптимальные размеры batch'ей

### **2. Memory Optimization**
- Предварительно вычислять константные данные
- Использовать shared memory для частых операций
- Минимизировать обмены с глобальной памятью

### **3. GPU Utilization**
- Стремиться к максимальной загрузке SM
- Использовать occupancy analysis для оптимизации
- Контролировать memory bandwidth

### **4. OpenCL/AMD Specific**
- Исследовать AMD GPU архитектуру
- Сравнивать OpenCL vs CUDA подходы
- Оптимизировать под AMD memory hierarchy

## 🔬 Методология исследований

### **Sequential Thinking Process**
1. **Problem Definition**: Четкое определение задачи
2. **Information Gathering**: Сбор данных и контекста
3. **Problem Decomposition**: Разбиение на подзадачи
4. **Solution Generation**: Генерация вариантов
5. **Solution Evaluation**: Анализ и сравнение
6. **Implementation Planning**: Планирование реализации
7. **Documentation**: Документирование результатов

### **Quality Standards**
- Все выводы обоснованы конкретными данными
- Сравнительный анализ различных подходов
- Документирование ограничений и рисков
- Измеряемые метрики производительности

## 📈 Тренды и паттерны

### **Успешные подходы**
- **Batched operations**: Всегда показывают лучшие результаты
- **Pre-computed constants**: Значительное ускорение
- **Memory optimization**: Критично для производительности
- **Occupancy analysis**: Ключ к оптимизации GPU
- **Fused kernels**: gather+FFT+fftshift в одном ядре

### **Проблемные области**
- **Tensor Cores**: Не подходят для FFT butterfly операций
- **Matrix multiplication**: Медленнее butterfly для FFT
- **Small batch sizes**: Низкая эффективность GPU
- **Excessive kernel launches**: Высокий overhead
- **Multiple offset streams**: Усложняют индексацию без выгоды

## 🚀 Рекомендации для будущих проектов

### **1. Начинать с анализа**
- Провести occupancy sweep
- Определить оптимальные размеры batch'ей
- Проанализировать memory requirements

### **2. Использовать batched подходы**
- Минимизировать kernel launch'ы
- Максимизировать параллелизм
- Использовать предварительно вычисленные данные

### **3. Документировать все**
- Записывать все этапы анализа
- Сохранять результаты benchmark'ов
- Создавать перекрестные ссылки

### **4. OpenCL/AMD исследования**
- Изучать AMD GPU архитектуру
- Сравнивать производительность с NVIDIA
- Оптимизировать под AMD специфику

## 🔧 GitHub MCP Integration

### **Работа с GitHub через MCP**
- **Аутентификация**: Использовать Personal Access Token (PAT) с правами repo, workflow, write:packages, delete:packages, admin:org, gist, notifications, user, delete_repo
- **Основные инструменты**: mcp_github_get_me(), mcp_github_create_repository(), mcp_github_create_or_update_file(), mcp_github_push_files()
- **Workflow**: 1) get_me() → 2) create_repository() → 3) create_or_update_file() для первого файла → 4) push_files() для остальных
- **Важно**: push_files() не работает с пустыми репозиториями - сначала создать файл через create_or_update_file()
- **Обновление файлов**: Обязательно указывать параметр sha, полученный через get_file_contents()

### **Стандартные команды для AI Assistant**
```
"Используй MCP GitHub для работы с репозиторием. 
Сначала проверь токен через get_me, 
потом создай/обнови файлы через create_or_update_file или push_files"
```

### **Troubleshooting**
- "Git Repository is empty" → Сначала создать файл через create_or_update_file
- "Resource not accessible by integration" → Проверить права токена
- "Not Found" → Проверить правильность owner/repo параметров
- "Validation Failed" → Проверить формат данных, обязательные поля

---

*Обновлено: 2025-10-17*  
*Версия: 1.2*