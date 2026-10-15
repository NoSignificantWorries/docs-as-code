# Диаграммы
## Диаграмма последовательности

```mermaid
sequenceDiagram
    actor User as Пользователь
    participant System as Система
    participant Model as ML-модель

    User->>System: Отправить текст/файл и требуемое время чтения

    alt Отправлен файл
        System ->> System: Попытка открыть файл
        alt Файл не удалось открыть
            System ->> User: Вернуть ошибку "Не удалось открыть файл"
        else Файл успешно открыт
            System->>System: Извлечь текст из файла
        end
    end

    System->>System: Проверка текста (символы, мин. длина)
    alt Текст не прошёл валидацию
        System-->>User: Вернуть ошибку "Недопустимые символы / текст слишком короткий"
    else Текст валиден
        System->>Model: Передать текст и время чтения
        Model-->>System: Вернуть пересказ текста
        System-->>User: Показать результат
    end
```


## Диаграмма классов

```mermaid
classDiagram
    class Summarizer {
        <<service>>
        +summarize(input: string | path, readingTimeMinutes: int) Summary
    }

    class InputValidator {
        <<validator>>
        #min_length: int
        #supported_formats: list~string~
        +validate(input: string | path) bool
        -_validate_file(file: path) bool
        -_validate_text(text: string) bool
    }
    note for InputValidator "Проверяет мин. длину,\nдопустимые символы и формат файла"

    class FileReader {
        <<adapter>>
        #file_types: list~FileType~
        +read(file: path) string
        -_check_type(file: path) FileType
        -_read_pdf(file: path) string
        -_read_str_md(file: path) string
        -_read_docx(file: path) string
        -_read_fb2_epub(file: path) string
    }
    note for FileReader "Поддерживает: PDF, TXT, MD, DOCX, FB2, EPUB"

    class ReadingTimeCalculator {
        <<util>>
        +toWordCount(minutes: float32) int
        +toMinutes(text_length: int) float32
    }

    class MLModel {
        <<gateway>>
        +generate(text: string, targetWords) string
    }

    class Summary {
        <<value object>>
        +text: string
        +readingTimeMinutes: int
    }

    Summarizer --> InputValidator : 1. валидирует вход
    Summarizer --> FileReader : 2. извлекает текст
    Summarizer --> ReadingTimeCalculator : 3. считает targetWords
    Summarizer --> MLModel : 4. генерирует
    Summarizer ..> Summary : 5. создаёт результат
```
